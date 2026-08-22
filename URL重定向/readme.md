URL跳转也叫做重定向，301和302状态码都表示重定向，浏览器在拿到服务器返回的这个状态码后会自动跳转到一个新的URL地址，  
这个地址可以从响应的Location首部中获取。 301跳转是指页面永久性移走，通常叫做301跳转，也叫301重定向（转向） 302重定向又称之为暂时性转移，  
也被称为是暂时重定向。 http://www.aaa.com/?url=http://www.bbb.com  

url会暴露诸如此类：

redirect
oauth_callback
request
toUrl
jump
callback
redirectUrl
fromUrl
target
redUrl
goto
domain
jump_to
to
url
return_url
redirect_url
linkto
ReturnUrl
link
redirect_to

---
直接改参大概率被拦截  
单斜线 '/ ' 绕过   
http://www.xxx.com/redirect.php?url=/www.evil.com

利用反斜杠 ' \ ' 绕过
http://www.xxx.com/redirect.php?url=https://www.evil.com\www.evil.com

缺少协议绕过 
http://www.xxx.com/redirect.php?url=//www.evil.com

多斜线 ' / ' 前缀绕过
http://www.xxx.com/redirect.php?url=////www.evil.com

利用 ' \\ ' 绕过
http://www.xxx.com/redirect.php?url=https://www.evil.com\\www.evil.com

利用 ' @ ' 符号绕过
http://www.xxx.com/redirect.php?url=https://www.evil.com@www.evil.com

利用井号 ' # ' 符号绕过
http://www.xxx.com/redirect.php?url=https://www.evil.com#www.evil.com

利用问号 ' ? ' 符号绕过
http://www.xxx.com/redirect.php?url=https://www.evil.com?www.evil.com

利用小数点 ' . ' 符号绕过
http://www.xxx.com/redirect.php?url=.eval.com

重复特殊字符绕过
http://www.xxx.com/redirect.php?url=////www.eval.com//..

比如匹配规则是必须跳转，xiaozhupeiqi.com 域名下，?#等都不行的时候，如果匹配规则为xiaozhupeiqi.com，可以尝试百度inurl:xiaozhupeiqi.com的域名，比如
aaaxiaozhupeiqi.com，这样同样可以绕过。接下来实战中会用到，

xip.io绕过

http://127.0.0.1/url.php?username=1&password=1&password=1&redict=http://www.xiaozhupeiqi.com.220.181.57.217.xip.io
会跳转到百度

xss跳转

这种就没啥说的了，就是XSS造成的跳转，下面也有案例，在有些情况下XSS只能造成跳转的危害。  

<meta  content="1;url=http://www.baidu.com" http-equiv="refresh">

//%2F/attacker.com

6、更改url形式：ip、进制数、更换/缺失协议

-----
1.最常见的登陆跳转
登陆跳转我认为是最常见的跳转类型，几乎百分之七八十网站都会url里设置跳转，所以在登陆的时候建议多观察url参数，通常都会存在跳转至于存不存在漏洞需要自己测试。
上面的类型四,
漏洞url ：
https://xx.xxx.com/User/Login?redirect=http://xxx.com/
为登陆页面，如果登陆成功那么跳转http://xxx.com/，但是所有方式都无法绕过，但是发现可以跳转aaxxx.com，也就是匹配规则为必须为xxx.com的网址，但是aaxxx.com同样也可以。
方法：
百度 inurl:xxx.com，即可百度到很多域名包含xxx.com的url，即可实现跳转,不小心百度到一个黄域，正好证明危害登陆跳XX网。
还有一些花里胡哨的base64加码了的跳转，解码就是需要跳转的url,其实本质都一样
2.@绕过
@是最常见的一种绕过。
漏洞url
https://xx.xxx.com/user/goToLogin?toUrl=https://xx.xxx.com@www.baidu.com
这种跳转在chrome浏览器可以直接跳转，但是在火狐会弹框询问，但是并不影响它的危害。
火狐下@的跳转。
还有一些是跳转目录的，
如：
https://xx.xxx.com.cn/?redirect=/user/info.php
修改为
https://xx.xxx.com.cn/?redirect=@www.baidu.com
这种情况通常@也可以跳转，大胆的去尝试
3.充值接口跳转
通常充值接口都会进行跳转，如充值成功会跳转到充值前访问的页面，因为充值接口需要充值才能知道到底存不存在漏洞，所以测试的人相对少一些，大胆去尝试，单车变摩托，充值完成后还可以提现其实并不影响，不嫌麻烦就多测测。这些都是跳转成功的案例。
4.xss造成的url跳转
在一次渗透测试中，碰到一个站点，对<>"这些字符都是进行了过滤。且没有其他姿势配合，基本上告别了XSS等漏洞。如下](https://xzfil
可以发现我输入了xsstest<>"，但是<>被直接删除过滤掉了，但是发现双引号还在，先看下源码是怎么处理的。
乍一看双引号也被转义了，输入的xsstest 搜索有十七处，大部分被实体化了，还有一部分双引号被url编码了，但是此时突然发现我箭头指的一处并未对双引号进行转义或者过滤，虽然<>已经完全被过滤掉了。
此时构造meta的url跳转。
payload：
http://xxx.com/search?w=1;url=http://www.baidu.com" http-equiv="refresh&fsearch=yes
其中输入
1;url=http://www.baidu.com" http-equiv="refresh
最终闭合掉得到的源码为。
最终点击payload会跳转百度页面，其实这个严格意义上来说算XSS造成的跳转，构造应该也可以XSS。
5.业务完成后跳转
这可以归结为一类跳转，比如修改密码，修改完成后跳转登陆页面，绑定银行卡，绑定成功后返回银行卡充值等页面，或者说给定一个链接办理VIP，但是你需要认证身份才能访问这个业务，这个时候通常会给定一个链接，认证之后跳转到刚刚要办理VIP的页面。
通常这个点都会存在跳转至于存不存在任意跳转，师傅们自测,有些跳转业务不好码就不发了。
6.用户交互
在一些用户交互页面也会出现跳转，如请填写对客服评价，评价成功跳转主页，填写问卷，等等业务，注意观察url。
问卷调查提交跳转。
/nweixin框架重定向漏洞
一种nweixin框架的重定向漏洞多用于公众号信用卡服务大厅等地方
poc
/nweixin/pub/redirectOpenId.do?code=&state=0459!https%3A%2F%2Fwww.baidu.com/

/nweixin/pub/redirectOpenId.do?code=&state=0459!https%3A%2F%2Fwww.baidu.com/?ccwb.gsrcu.com%2Fnweixin%2F0459%2F%23%2FserviceHall%2FserviceHall 

/nweixin/pub/redirectOpenId.do?state=!https://www.baidu.com
分享功能重定向
其他地方看看没找到,就跑去分享功能看看.
二维码的URL跳转
在我们进行支付时一般会出现一个二维码，还有进行扫码登录，微信分享篇文章时都会出现！很多生成这个二维码链接后面都会有一个URL地址，根据这个URL地址来生成二维码，你看下列案例。

<img width="1236" height="1314" alt="image" src="https://github.com/user-attachments/assets/a630c546-db28-425e-9ad6-1ecc743a56f4" />

