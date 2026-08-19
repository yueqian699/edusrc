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

2 htaccess 重写解析绕过上传  
.htaccess可以帮我们实现包括：文件夹密码保护、用户自动重定向、自定义错误页面、改变你的文件扩展名、封禁特定IP地址的用户、只允许特定IP地址的用户、禁止目录列表，以及使用其他文件作为index文件等一些功能。  
启用.htaccess，需要修改httpd.conf，启用AllowOverride，并可以用AllowOverride限制特定命令的使用。如果需要使用.htaccess以外的其他文件名，可以用AccessFileName指令来改变。例如，需要使用.config ，则可以在服  务器配置文件中按以下方法配置：AccessFileName .config   

利用条件：Apache开启重写模块

<FilesMatch ".jpg">  
SetHandler application/x-httpd-php  
</FilesMatch>  

3 .user.ini.  

auto_prepend_file=a.jpg

.user.ini 配置项中有两个配置可以起到一些作用  
方法一：  
auto_prepend_file = <filename>         //包含在文件头  
方法二：  
auto_append_file = <filename>          //包含在文件尾  

4 
