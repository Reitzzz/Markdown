# 1. CSRF跨站请求伪造攻击

## 目录
- [1. CSRF跨站请求伪造攻击](#1-csrf跨站请求伪造攻击)
  - [目录](#目录)
  - [1.1 攻击简介](#11-攻击简介)
  - [1.2 会话与凭据机制](#12-会话与凭据机制)
  - [1.3 钓鱼网站模拟](#13-钓鱼网站模拟)
  - [1.4 保护机制与SameSite](#14-保护机制与samesite)

---

## 1.1 攻击简介

CSRF是我们要介绍的第一种攻击形式，这种攻击方式非常好理解。

![image-20230701181244150](https://s2.loli.net/2023/07/01/4ibrwFIPnSE81lx.png)

我们时常会在QQ上收到别人发送的钓鱼网站链接，只要你在上面登陆了你的QQ账号，那么不出意外，你的号已经在别人手中了。实际上这一类网站都属于恶意网站，专门用于盗取他人信息，执行非法操作，甚至获取他人账户中的财产，非法转账等。而这里，我们需要了解一种比较容易发生的恶意操作，从不法分子的角度去了解整个流程。

## 1.2 会话与凭据机制

我们在JavaWeb阶段已经了解了Session和Cookie的机制，在一开始的时候，服务端会给浏览器一个名为JSESSIONID的Cookie信息作为会话的唯一凭据，只要用户携带此Cookie访问我们的网站，那么我们就可以认定此会话属于哪个浏览器用户。因此，只要此会话的用户执行了登录操作，那么就可以随意访问个人信息等内容。

## 1.3 钓鱼网站模拟

我们来尝试模拟一下这种操作，来编写一个钓鱼网站：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>坤坤炒粉放鸡精视频在线观看</title>
    <script src="[https://unpkg.com/axios@1.1.2/dist/axios.min.js](https://unpkg.com/axios@1.1.2/dist/axios.min.js)"></script>
</head>
<body>
<iframe name="hiddenIframe" hidden></iframe>
<form action="http://localhost:8080/mvc/pay" method="post" target="hiddenIframe">
    <input type="text" name="account" value="黑客" hidden>
    <button type="submit">点击下载全套视频</button>
</form>
</body>
</html>
```

这个页面并不是我们官方提供的页面，而是不法分子搭建的恶意网站。我们发现此页面中有一个表单，但是表单中的输入框被隐藏了，而我们看到的只有一个按钮，我们不知道这是一个表单，也不知道表单会提交给那个地址，这时整个页面就非常有迷惑性了。如果我们点击此按钮，那么整个表单的数据会以POST的形式发送给我们的服务端（会携带之前登陆我们网站的Cookie信息），但是这里很明显是另一个网站跳转，通过这样的方式，恶意网站就成功地在我们毫不知情的情况下引导我们执行了转账操作，当你发现上当受骗时，钱已经被转走了。

我们首先作为用户，先在正常的网站进行登录：

![image-20230701233127287](https://s2.loli.net/2023/07/01/4wQ2iB5uhcLMJHa.png)

接着我们假装自己上当，进入到我们的钓鱼网站：

![image-20230701233202016](https://s2.loli.net/2023/07/01/OstNZg4doCz6S5G.png)

现在我们毫不知情，如果是正常人思维的话，就会直接点击下载全套视频，恭喜，此时后台已经转账成功了，留下一脸懵逼的你：

![image-20230701233317019](https://s2.loli.net/2023/07/01/mtGLhuNHxPUr6Os.png)

而这种构建恶意页面，引导用户访问对应网站执行操作的方式称为：**跨站请求伪造**（CSRF，Cross Site Request Forgery）

## 1.4 保护机制与SameSite

显然，我们之前编写的图书管理系统就存在这样的安全漏洞，而SpringSecurity就解决了这样的问题。

当然，除了通过我们自己SpringSecurity使用去解决之外，随着现在的浏览器不断发展，安全性越来越受到重视，很多浏览器都有SameSite保护机制，当用户在两个不同域名的站点操作时，默认情况下Cookie就会被自动屏蔽：

![a3421f74c3cee6b67d214809adb743a5](https://s2.loli.net/2023/07/01/qiLDnrFyQxpt3UB.png)

SameSite是一种安全机制，旨在防止跨站点请求伪造（CSRF）攻击，它通过限制第三方Cookie的使用来实现这一目的。在Chrome浏览器中，SameSite默认为Lax，这意味着第三方Cookie只能在用户导航到与原始站点相同的站点时发送。这同样大大提升了用户的安全性，让黑客少了许多可乘之机，不过这个机制对做实施的同学（可能）不太友好。