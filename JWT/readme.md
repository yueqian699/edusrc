1 定义    
JWT，即 JSON Web Token，定义了一种紧凑的、自包含的方式，用于在网络应用环境间以 JSON 对象安全地传输信息。JWT 一般被用来在身份提供者和服务提供者间传递被认证用户的身份信息，因为它经过了数字签名，相对比较安全，以便于更好的从资源服务器获取资源，也可以增加一些额外的业务逻辑所需的声明信息。
JWT 常用于代替 Session，用于识别用户身份。传统上使用 Session 机制区别用户身份有两个缺点：一是占用服务器的存储资源，二是在集群部署下设计会非常复杂。JWT 完全可以解决 Session 方案存在的问题。

2 组成  
01 header   
{
    "typ": "JWT",
    "alg": "HS256"
}

头部包含两部分：声明类型和使用的哈希算法，通常直接使用HMAC SHA256，就是HS256。

02 payload  
{
    "iss": "jwt.io",
    "exp": 1496199995458,
    "name": "sinwaj",
    "role": "admin"," 
}

03 JWT signature  
HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)  

其中：secret 为加密使用的盐，也可以认为是私钥，千万不能泄露。  

JWT 工作流程步骤  
用户携带用户名和密码请求访问；  
● 服务器校验用户凭据；  
● 应用提供一个 token 给客户端；  
● 客户端存储 token，并且在随后的每一次请求中都带着它；  
● 服务器校验 token 并返回数据。  
需要注意问题：  
● 每一次请求都需要 token；  
● Token 应该放在请求 header 中；  
● 需要将服务器设置为接受来自所有域的请求，用 Access-Control-Allow-Origin: *。  

JWT常用攻击手法  
1 签名为空  

JWT 第一部分含有 alg 字段，该字段指定生成签名采用哪种哈希算法，如某站使用的是 HS256，可将该字段篡改为none，某些 JWT 的实现，  
一旦发现 alg 为 none，将不再生成哈希签名，自然不存在校验签名一说，这里将payload里的username修改为admin即可 这样就能篡改用户信息。  
<img width="931" height="399" alt="image" src="https://github.com/user-attachments/assets/c4cf5989-758e-4de1-b4e7-c369d1e3d3e2" />

将头部alg的值修改为none，接着修改payload内容编码，加密后的内容除了最后蓝色字体的不用复制，其余的都需要进行复制  
<img width="970" height="402" alt="image" src="https://github.com/user-attachments/assets/65e7d3b8-44b2-4c84-878e-f80b752dc8c4" />

使用bp即可对内容进行修改  
<img width="976" height="391" alt="image" src="https://github.com/user-attachments/assets/fe32f90e-2b93-4834-bedb-8ba89a117fde" />

2 爆破密钥（资产方设置的secret）

通过爆出获取到对应的key以后，我们可以利用JWT进行漏洞测试，在不同的项目中的测试方法也不同，接下来我们模拟一些常见的测试场景  
水平越权  
把1234567890改为1234567891  

垂直越权  
把sysadmin的值N改为Y（NO改为YES或者false改为true）  
{
"sub": "1234567890",
"name": "0xShe",
"sysadmin": "Y",
"iat": 1690989833
}

SQL注入  
把data的值内插入SQL注入语句  
{
"sub": "1234567890",
"name": "0xShe",
"data": "1' UNION SELECT 'key';-- ",
"iat": 1690989833
}

命令执行  
{
"sub": "1234567890",
"name": "0xShe",
"echo": "yes||whoami",
"iat": 1690989833
}

文件读取  
把path的值替换为需要读取的文件地址  
{
"sub": "1234567890",
"name": "0xShe",
"path": "/etc/passwd",
"iat": 1690989833
}

SSRF  
把url的值替换为SSRF测试poc  
{
"url": "http://www.DNSlog.cn/",
"iat": 1690989833
}
 
XSS  
http://a.com?url=jwt的时候插入xss payload可以测xss  

其他  
jwt会声明使用一些比如hash算法，可以试试把hash那一块改成空，部分场景可以直接绕过。  

