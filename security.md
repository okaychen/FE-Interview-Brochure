# 安全篇

前端安全方面，应届生遇到的多为常识性问题，一般不具有太大难度，但是必须要了解

## 知道有哪些可能引起前端安全的问题

前端应用中常遇到的安全问题：

- 跨站脚本xss
- 跨站请求伪造 csrf
- 网络劫持攻击
- iframe滥用
- 恶意第三方库

## xss攻击的分类

根据攻击来源的不同，通常分为三种：

- 反射型
反射性通常发生在URL地址的参数中，常用来窃取客户端的cookie或进行钓鱼欺骗，经常在网站的搜索栏，跳转的地方被注入
```html
<!--比如-->
http://www.test.com/search.php?key="><script>alert("XSS")</script>
```
- 存储型
相比反射型 XSS 的恶意代码存在 URL ⾥，存储型 XSS 的恶意代码存在数据库⾥，不需要用户去点击URL进行触发，提前将恶意代码保存在了漏洞服务器或者客户端中，站点取出后会自动解析执行
- DOM型
较上面两种，DOM型取出和执行恶意代码都由浏览器端完成，属于前端自身安全漏洞

## 如何预防xss攻击

xss攻击有两大要素：攻击者提交恶意代码，浏览器执行恶意代码

- 针对cookie劫持，攻击者向漏洞页面写入恶意代码获取cookie，可以通过防止cookie会话劫持来防范，一般要在设置cookie时加HttpOnly，来禁止意外注入站点的恶意js代码操作Cookie造成xss攻击
```JavaScript
// node设置httponly
'Set-Cookie' : 'SSID=EqAc1D; Expires=Wed; HttpOnly'
```
- 提高攻击门槛：(XSS Filter)针对用户提交的数据进行有效的验证，只接受我们规定的长度或内容的提交，过滤掉其他的输入内容 或者 将特殊字符输出编码(xss Escape)
  ![xx](https://www.chenqaq.com/assets/images/xss-encode02.png)
- xss漏洞检测poc：通过漏洞检测代码，完成自测，最小化被攻击风险

```html
<!--比如img图片标记属性跨站攻击代码-->
<img src="javascript:alert(/xss/)"></img><img dynsrc="javascript:alert('xss')">

<!--无需 “<>”，利用 html 标记事件属性跨站-->
<img src="" onerror=alert("xss")>
```

## csrf简单介绍一下

CSRF（Cross-site request forgery）跨站请求伪造：攻击者诱导受害者进⼊第三⽅⽹站，在第三⽅⽹站中，向被攻击⽹站发送跨站请求。利⽤受害者在被攻击⽹站已经获取的注册凭证，绕过后台的⽤户验证，达到冒充⽤户对被攻击的⽹站执⾏某项操作的⽬的，

常规的过程如下：

1. 用户打开浏览器，访问受信任网站A，输入用户名和密码请求登录网站A
2. 在用户信息通过验证后，网站A产生Cookie信息并返回给浏览器
3. 未退出网站A之前，在同一浏览器中，攻击者引诱用户进入网站B
4. 网站B接收到用户请求后，返回一些攻击性代码，并发出一个请求要求访问第三方站点A
5. 浏览器在接收到这些攻击性代码后，根据网站B的请求，在用户不知情的情况下携带Cookie信息，向网站A发出请求。网站A根据用户的Cookie信息以及用户的权限处理该请求，导致来自网站B的恶意代码被执行

## 如何预防csrf攻击

CSRF通常从第三⽅⽹站发起，被攻击的⽹站⽆法防⽌攻击发⽣，只能通过增强⾃⼰⽹站针对CSRF的防护能⼒来提升安全性

CSRF有两个特点：

- 通常发生在第三方域名
- 攻击者不能获取到cookie等信息，只是使用

针对这两点，我们可以制定专门的防护策略：

- 阻止不明外域的访问
	- 同源检测
	- Samesite Cookie
- 提交时要求附加文本域验证身份才可以获取信息
	- CSRF Token
	- 双重cookie验证

### 同源检测

通过使用Orgin Header确定来源域名：在部分与CSRF有关的请求中，请求的Header中会携带Origin字段,如果Origin存在，那么直接使⽤Origin中的字段确认来源域名就可以

使用Referer Header确定域名来源：根据HTTP协议，HTTP头中有一个字段叫Referer，记录了该HTTP请求的来源地址

### Samesite Cookie属性

Google起草了⼀份草案来改进HTTP协议，那就是为Set-Cookie响应头新增Samesite属性，它⽤来标明这个 Cookie是个“同站 Cookie”，同站Cookie只能作为第⼀⽅Cookie，不能作为第三⽅Cookie，Samesite 有两个属性值:

- Samesite=Strict: 这种称为严格模式，表明这个 Cookie 在任何情况下都不可能作为第三⽅ Cookie
- Samesite=Lax: 这种称为宽松模式，⽐ Strict 放宽了点限制,假如这个请求是这种请求且同时是个GET请求，则这个Cookie可以作为第三⽅Cookie

###  CSRF Token

通过csrf token验证请求的身份，是够携带正确的token，来区分正常的请求和攻击的请求

CSRF Token的防护策略分为三个步骤：

- 将CSRF Token输出到⻚⾯中
- ⻚⾯提交的请求携带这个Token
- 服务器验证Token是否

### 双重cookie验证

利⽤CSRF攻击不能获取到⽤户Cookie的特点，我们可以要求Ajax和表单请求携带⼀个Cookie中的值

双重Cookie采⽤以下流程：

- 在⽤户访问⽹站⻚⾯时，向请求域名注⼊⼀个Cookie，内容为随机字符串
- 在前端向后端发起请求时，取出Cookie，并添加到URL的参数中
- 后端接⼝验证Cookie中的字段与URL参数中的字段是否⼀致，不⼀致则拒

## 网络劫持有哪几种

网络劫持一般分为两种：HTTP劫持和DNS劫持

DNS劫持

- DNS强制解析：通过修改运营商的本地DNS记录，来引导⽤户流量到缓存服务器
- 302跳转的⽅式：通过监控⽹络出⼝的流量，分析判断哪些内容是可以进⾏劫持处理的,再对劫持的内存发起跳转的回复，引导⽤户获取内容

HTTP劫持: 由于http明⽂传输，运营商会修改你的http响应内容(即加⼴告)

DNS劫持由于涉嫌违法,已经被监管；针对HTTP劫持，最有效的办法就是全站HTTPS，将HTTP加密，这使得运营商⽆法获取明⽂，就⽆法劫持你的响应内容

## 中间人攻击的了解

中间人(Man-in-the-middle attack, MITM)是指攻击者和通讯的两端分别创建独立的联系，并交换其得到的数据，攻击者可以拦截通信双方的通话并插入新的内容

一般的过程如下:

- 客户端发送请求到服务端，请求被中间⼈截获
- 服务器向客户端发送公钥
- 中间⼈截获公钥，保留在⾃⼰⼿上。然后⾃⼰⽣成⼀个【伪造的】公钥，发给客户端
- 客户端收到伪造的公钥后，⽣成加密hash值发给服务器
- 中间⼈获得加密hash值，⽤⾃⼰的私钥解密获得真秘钥,同时⽣成假的加密hash值，发给服务器
- 服务器⽤私钥解密获得假密钥,然后加密数据传输给客户端

> 推荐：[HTTPS中间人攻击实践](https://www.cnblogs.com/lulianqi/p/10558719.html)