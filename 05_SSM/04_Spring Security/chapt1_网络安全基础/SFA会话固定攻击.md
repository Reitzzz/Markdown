# 1. SFA会话固定攻击

## 目录
- [1. SFA会话固定攻击](#1-sfa会话固定攻击)
  - [目录](#目录)
  - [1.1 攻击简介](#11-攻击简介)
  - [1.2 攻击方式](#12-攻击方式)
  - [1.3 攻击模拟](#13-攻击模拟)
  - [1.4 保护机制与HttpOnly](#14-保护机制与httponly)

---

## 1.1 攻击简介

这同样是利用Cookie中相同的JSESSIONID进行的攻击，会话固定攻击（Session fixation attack）是一种针对Web应用程序的安全漏洞攻击，攻击者利用这种漏洞，将一个有效的会话ID分配给用户，并诱使用户在该会话中进行操作，然后攻击者可以利用该会话ID获取用户的权限，或者通过此会话继续进行其他攻击。

简单来说，就是黑客把他的JSESSIONID直接给你，你一旦使用这个ID登录，那么在后端这个ID就被认定为已登录状态，那么也就等同于他直接进入了已登录状态，从而直接访问你账号的任意内容，执行任意操作。

## 1.2 攻击方式

攻击者通常使用以下几种方式进行会话固定攻击：

1. 会话传递：攻击者通过URL参数、表单隐藏字段、cookie等方式将会话ID传递给用户。当用户使用该会话ID登录时，攻击者就能利用该会话ID获取用户的权限。
2. 会话劫持：攻击者利用劫持用户与服务器之间的通信流量，获取到用户的会话ID，然后利用该会话ID冒充用户进行操作。
3. 会话劫持：攻击者事先获取到会话ID，并将其分配给用户，之后通过其他方式欺骗用户登录该会话。这样，攻击者就可以利用会话ID获取用户的权限。

## 1.3 攻击模拟

这里我们来尝试一下第一种方案，这里我们首先用另一个浏览器访问目标网站，此时需要登录，开始之前记得先清理一下两个浏览器的缓存，否则可能无法生效：

![image-20230702001501063](https://s2.loli.net/2023/07/02/LFchsWkevUwb58E.png)

这里我们直接记录下这个JSESSIONID，然后将其编写到我们的诈骗网站中，这里有一个恶意脚本，会自动将对应用户的Cookie进行替换，变成我们的JSESSIONID值：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>冠希哥全套视频</title>
    <script src="[https://unpkg.com/axios@1.1.2/dist/axios.min.js](https://unpkg.com/axios@1.1.2/dist/axios.min.js)"></script>
</head>
<body>
<script>
    //第三方网站恶意脚本，自动修改Cookie信息
    document.cookie = "JSESSIONID=6AAF677EC2B630704A80D36311F08E01; path=/mvc; domain=localhost"
    //然后给你弄到原来的网站
    location.href = 'http://localhost:8080/mvc/'
</script>
</body>
</html>
```

接着我们访问这个恶意网站，然后再作为用户，去正常访问目标网站进行登录操作：

![image-20230702001831904](https://s2.loli.net/2023/07/02/d4YSeut3xQ7rRqh.png)

可以看到此时用户的浏览器JSESSIONID值为刚刚恶意网站伪造的值，现在我们来进行登录操作：

![image-20230702002007450](https://s2.loli.net/2023/07/02/aDwTcW6d58tPLBG.png)

此时我们回到一开始的浏览器，刷新之后，我们发现这个浏览器同样已经登录成功了，原理其实很简单，相当于让用户直接帮我们登录了，是不是感觉特别危险？

## 1.4 保护机制与HttpOnly

当然，现在的浏览器同样有着对应的保护机制，Tomcat发送的SESSIONID默认是勾选了HttpOnly选项的，一旦被设定是无法被随意修改的，当然前提是先得正常访问一次网站才行，否则仍然存在安全隐患。

> HttpOnly是Cookie中一个属性，用于防止客户端脚本通过document.cookie属性访问Cookie，有助于保护Cookie不被跨站脚本攻击窃取或篡改。但是，HttpOnly的应用仍存在局限性，一些浏览器可以阻止客户端脚本对Cookie的读操作，但允许写操作；此外大多数浏览器仍允许通过XMLHTTP对象读取HTTP响应中的Set-Cookie头。

![image-20230702002624508](https://s2.loli.net/2023/07/02/8IaHk2FEMwyoXem.png)

为了彻底杜绝这个问题，登录成功之后应该重新给用户分配一个新的JSESSIONID才行，而这些都由SpringSecurity帮我们实现了。