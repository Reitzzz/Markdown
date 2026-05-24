## 分布式权限校验

虽然完成前面的部分，我们已经可以自己去编写一个比较中规中矩的微服务项目了，但是还有一个问题我们没有解决，登录问题。假如现在要求用户登录之后，才能进行图书的查询、借阅等操作，那么我们又该如何设计这个系统呢？

回顾我们之前进行权限校验的原理，服务器是如何判定一个请求是来自哪个用户的呢？

* 首先浏览器会向服务端发送请求，访问我们的网站。
* 服务端收到请求后，会创建一个SESSION ID，并暂时存储在服务端，然后会发送给浏览器作为Cookie保存。
* 之后浏览器会一直携带此Cookie访问服务器，这样在收到请求后，就能根据携带的Cookie中的SESSION ID判断是哪个用户了。
* 这样服务端和浏览器之间可以轻松地建立会话了。

但是我们想一下，**我们现在采用的是分布式的系统，那么在用户服务进行登录之后，其他服务比如图书服务和借阅服务，它们会知道用户登录了吗**？

![image-20230306234928969](https://s2.loli.net/2023/03/06/hV2JkERda4qKtjB.png)

实际上我们登录到用户服务之后，**Session中的用户数据只会在用户服务的应用中保存，而在其他服务中，并没有对应的信息**，但是我们现在希望的是，所有的服务都能够同步这些Session信息，这样我们才能实现在用户服务登录之后其他服务都能知道，那么我们该如何**实现Session的同步**呢？

1. 我们可以在每台服务器上都复制一份Session，但是这样显然是很浪费时间的，并且用户验证数据占用的内存会成倍的增加。
2. 将Session移出服务器，**用统一存储来存放，比如我们可以直接在Redis或是MySQL中存放用户的Session信息，这样所有的服务器在需要获取Session信息时，统一访问Redis或是MySQL即可**，这样就能保证所有服务都可以同步Session了（是不是越来越感觉只要有问题，没有什么是加一个中间件解决不了的）

![image-20230306234940672](https://s2.loli.net/2023/03/06/pqZolFN6eIPza52.png)

那么，我们就着重来研究一下，然后实现2号方案，这里我们就使用Redis作为Session统一存储，我们把一开始的压缩包重新解压一次，又来从头开始编写吧。

这里我们就只使用Nacos就行了，和之前一样，我们把Nacos的包导入一下，然后进行一些配置：

![image-20230306234948408](https://s2.loli.net/2023/03/06/FYcNvAuZ7z8rj2V.png)

现在我们需要为每个服务都添加验证机制，首先导入依赖：

```xml
<!--  SpringSession Redis支持  -->
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
</dependency>
<!--  添加Redis的Starter  -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

然后我们依然使用SpringSecurity框架作为权限校验框架：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

接着我们在每个服务都编写一下对应的配置文件：

```yaml
spring:
  session:
  	# 存储类型修改为redis
    store-type: redis
  redis:
  	# Redis服务器的信息，该咋写咋写
    host: host: 192.168.0.12
```

这样，默认情况下，每个服务的接口都会被SpringSecurity所保护，只有登录成功之后，才可以被访问。

我们来打开Nacos看看：

![image-20230306234959085](https://s2.loli.net/2023/03/06/SyCJXKgO3qGx8EL.png)

可以看到三个服务都正常注册了，接着我们去访问图书服务：

![image-20230306235007563](https://s2.loli.net/2023/03/06/gytqnZjTMvVEUm3.png)

可以看到，访问失败，直接把我们给重定向到登陆页面了，也就是说必须登陆之后才能访问，同样的方式去访问其他服务，也是一样的效果。

由于现在是统一Session存储，那么我们就可以在任意一个服务登录之后，其他服务都可以正常访问，现在我们在当前页面登录，登录之后可以看到图书服务能够正常访问了：

![image-20230306235015909](https://s2.loli.net/2023/03/06/xfV5oYGvc1jKqTM.png)

同时用户服务也能正常访问了：

![image-20230306235021694](https://s2.loli.net/2023/03/06/OH6wjLVreot4IiA.png)

我们可以查看一下Redis服务器中是不是存储了我们的Session信息：

![image-20230306235046117](https://s2.loli.net/2023/03/06/nNIkoXOAYuMH8aV.png)

虽然看起来好像确实没啥问题了，但是借阅服务炸了，我们来看看为什么：

![image-20230306235053840](https://s2.loli.net/2023/03/06/wls5vCajnuMBOkU.png)

在RestTemplate进行远程调用的时候，由于我们的请求没有携带对应SESSION的Cookie，所以导致验证失败，访问不成功，返回401，所以虽然这种方案看起来比较合理，但是在我们的实际使用中，还是存在一些不便的。

***