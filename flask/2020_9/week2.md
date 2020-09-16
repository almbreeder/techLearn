## 概念
- **DOM（Document Object Model）**
百度百科的定义：DOM即文档对象模型，是W3C制定的**标准接口规范**，是一种**处理HTML和XML文件的标准API**。
DOM将文档解析为一个由节点和对象（包含属性和方法的对象）组成的结构集合。简言之，它会将web页面和脚本或程序语言连接起来。
<center> 
API (web 或 XML 页面) = DOM + JS (脚本语言)
</center>

- **CSRF/XSRF(Cross-site request forgery)**
跨站请求伪造利用了web身份中的一个漏洞，简单的身份验证只能保证请求是来自某个用户的，不能保证是用户自愿发的。当用户访问一个网址的同时又访问了别的网站，由于这时前一个session并没有结束后，如果第二个网站存在恶意链接就可能直接访问或者向服务器发送一些非用户本意的操作。
百度百科的例子：
> 假如一家银行用以运行转账操作的URL地址如下：http://www.examplebank.com/withdraw?account=AccoutName&amount=1000&for=PayeeName
那么，一个恶意攻击者可以在另一个网站上放置如下代码： <img src="http://www.examplebank.com/withdraw?account=Alice&amount=1000&for=Badman"
如果有账户名为Alice的用户访问了恶意站点，而她之前刚访问过银行不久，登录信息尚未过期，那么她就会损失1000资金。

&emsp;&emsp;&ensp;&ensp;**防御措施**：1.检查Referer字段  2.添加校验token

- **PRG(Post/Redirect/Get)模式**
是一种用来防止重复提交表单的技术，即通过对提交表单的POST请求返回重定向响应将最后一个请求转换为GET请求。

- **XSS(Cross Site Scripting)**
跨站点攻击，黑客的目的就是想尽一切方法，将一段脚本内容放到目标网站的目标浏览器上解释执行!!

---
## 笔记