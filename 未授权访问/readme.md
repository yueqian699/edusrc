未授权访问
从站点的js审计开始，有很多插件可提取js文件中api接口（有参数的要带参），只需复制下来，然后bp爆破  
get、post各跑一遍看200响应码  

有些有组件特征的如spring boot,ruoyi 可针对性的跑api接口  

但总的来说，ai可以很高效地审计js文件，fuzz未授权接口  

---
这里讲讲通过逻辑漏洞导致的未授权  
如  
false -> true  
500 -> 200  
-1 -> 0 / 1  
<img width="1174" height="522" alt="image" src="https://github.com/user-attachments/assets/3f96b293-0201-4c52-8b67-45c65dd5965f" />

删/伪造 cookie、auth鉴权头  

---
一个很厉害的测未授权的师傅  
未授权优秀文章推荐
Js在漏洞挖掘中的作用-接口篇  
https://mp.weixin.qq.com/s?__biz=Mzg3Mzg3OTU4OQ==&mid=2247491624&idx=1&sn=1259ef981b762e0778eb5af6c26bda25&chksm=cf0949eb302236c3763d2622265b463850d93686f815be2652273a20d826a1753675c6661ac6&scene=126&sessionid=1774797702#rd

把小程序当Web测 || 实战案例深度拆解路由跳转中的权限漏洞挖掘  
https://mp.weixin.qq.com/s?__biz=Mzg3Mzg3OTU4OQ==&mid=2247493381&idx=1&sn=3adfd4b5af08ce906534d72407a8cab7&chksm=cfa8c4d52871114df62964a63c63dc30dfd1ddf2522b1496cd4b92f398f27038cd131e977066&scene=126&sessionid=1774797666#rd

你以为的未授权漏洞VS我挖的未授权漏洞 | 一场与开发博弈的头脑风暴  
https://mp.weixin.qq.com/s?__biz=Mzg3Mzg3OTU4OQ==&mid=2247493148&idx=1&sn=a0c9b24413bec09b5757dd7b1a63a421&chksm=cf6dc3545e1a851bb2f63449d4ca25f7f0dc3e756921f755398f1774f122440aed77f8fdffef&scene=126&sessionid=1774797666#rd
