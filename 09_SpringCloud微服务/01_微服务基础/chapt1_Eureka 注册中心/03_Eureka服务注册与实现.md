# 1. 服务注册与发现

- [1. 服务注册与发现](#1-服务注册与发现)
  - [1.1 服务发现的需求背景与 Eureka 简介](#11-服务发现的需求背景与-eureka-简介)
  - [1.2 搭建 Eureka 服务端](#12-搭建-eureka-服务端)
  - [1.3 配置并注册微服务客户端](#13-配置并注册微服务客户端)
  - [1.4 基于 Eureka 的服务发现与调用](#14-基于-eureka-的服务发现与调用)
  - [1.5 负载均衡演示与测试](#15-负载均衡演示与测试)

## 1.1 服务发现的需求背景与 Eureka 简介

前面我们了解了如何对单体应用进行拆分，并且也学习了如何进行服务之间的相互调用，但是存在一个问题，**就是虽然服务拆分完成，但是没有一个比较合理的管理机制，如果单纯只是这样编写，在部署和维护起来，肯定是很麻烦的**。可以想象一下，如果某一天这些微服务的端口或是地址大规模地发生改变，我们就不得不将服务之间的调用路径大规模的同步进行修改，这是多么可怕的事情。**我们需要削弱这种服务之间的强关联性，因此我们需要一个集中管理微服务的平台，这时就要借助我们这一部分的主角了**。

**Eureka能够自动注册并发现微服务，然后对服务的状态、信息进行集中管理，这样当我们需要获取其他服务的信息时，我们只需要向Eureka进行查询就可以了**。

![image-20230306225607857](https://s2.loli.net/2023/03/06/A2mxhZ5jBkPrOdc.png)

像这样的话，服务之间的强关联性就会被进一步削弱。

## 1.2 搭建 Eureka 服务端

那么现在我们就来搭建一个Eureka服务器，只需要创建一个新的Maven项目即可，然后我们需要在父工程中添加一下SpringCloud的依赖，这里选用`2021.0.1`版本（Spring Cloud 最新的版本命名方式变更了，现在是 ***YEAR.x*** 这种命名方式，具体可以在官网查看：https://spring.io/projects/spring-cloud#learn）：
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-dependencies</artifactId>
    <!-- 对于SpringBoot 3.x，需使用2022.0.x或2023.0.x之后的版本 -->
    <version>2023.0.0</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

接着我们为新创建的项目添加依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>
</dependencies>
```

下载内容有点多，首次导入请耐心等待一下。

接着我们来创建主类，还是一样的操作：
```java
@EnableEurekaServer
@SpringBootApplication
public class EurekaServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

别着急启动！！！接着我们需要修改一下配置文件：

```yaml
server:
  port: 8888
eureka:
  # 开启之前需要修改一下客户端设置（虽然是服务端
  client:
    # 由于我们是作为服务端角色，所以不需要获取服务端，改为false，默认为true
    fetch-registry: false
    # 暂时不需要将自己也注册到Eureka
    register-with-eureka: false
    # 将eureka服务端指向自己
    service-url:
      defaultZone: http://localhost:8888/eureka
```

好了，现在差不多可以启动了，启动完成后，直接输入地址+端口即可访问Eureka的管理后台：

![image-20230306225619725](https://s2.loli.net/2023/03/06/4S9e3gN6ZFTuPbU.png)

## 1.3 配置并注册微服务客户端

可以看到目前还没有任何的服务注册到Eureka，我们接着来配置一下我们的三个微服务，首先还是需要导入Eureka依赖（注意别导错了，名称里面有个starter的才是）：
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

然后修改配置文件：

```yaml
eureka:
  client:
    # 跟上面一样，需要指向Eureka服务端地址，这样才能进行注册
    service-url:
      defaultZone: http://localhost:8888/eureka
```

OK，无需在启动类添加注解，直接启动就可以了，然后打开Eureka的服务管理页面，可以看到我们刚刚开启的服务：

![image-20230306225630173](https://s2.loli.net/2023/03/06/rXnWBA1zo4OlUSt.png)

可以看到`8082`端口上的服务器，已经成功注册到Eureka了，但是这个服务名称怎么会显示为UNKNOWN，我们需要修改一下：
```yaml
spring:
  application:
    name: userservice
```

![image-20230306225638109](https://s2.loli.net/2023/03/06/1o8pmzBXCtixhKu.png)

当我们的服务启动之后，会每隔一段时间跟Eureka发送一次心跳包，这样Eureka就能够感知到我们的服务是否处于正常运行状态。

现在我们用同样的方法，将另外两个微服务也注册进来：

![image-20230306225648063](https://s2.loli.net/2023/03/06/gkenG9bT4aMIUio.png)

## 1.4 基于 Eureka 的服务发现与调用

那么，现在我们怎么实现服务发现呢？

也就是说，**我们之前如果需要对其他微服务进行远程调用，那么就必须要知道其他服务的地址**：
```java
User user = template.getForObject("http://localhost:8082/user/"+uid, User.class);
```

而现在有了Eureka之后，我们可以直接向其进行查询，得到对应的微服务地址，这里直接将服务名称替换即可：

```java
@Service
public class BorrowServiceImpl implements BorrowService {

    @Resource
    BorrowMapper mapper;

    @Resource
    RestTemplate template;

    @Override
    public UserBorrowDetail getUserBorrowDetailByUid(int uid) {
        List<Borrow> borrow = mapper.getBorrowsByUid(uid);

        //这里不用再写IP，直接写服务名称userservice
        User user = template.getForObject("http://userservice/user/"+uid, User.class);
        //这里不用再写IP，直接写服务名称bookservice
        List<Book> bookList = borrow
                .stream()
                .map(b -> template.getForObject("http://bookservice/book/"+b.getBid(), Book.class))
                .toList(); // 在基于Java17的SpringBoot 3环境中，优先使用.toList()
        return new UserBorrowDetail(user, bookList);
    }
}
```

接着我们手动将RestTemplate声明为一个Bean，然后添加`@LoadBalanced`注解，这样**Eureka就会对服务的调用进行自动发现，并提供负载均衡**：
```java
@Configuration
public class BeanConfig {
    @Bean
    @LoadBalanced
    RestTemplate template(){
        return new RestTemplate();
    }
}
```

现在我们就可以正常调用了：

![image-20230306225713484](https://s2.loli.net/2023/03/06/1SHLTwmIK4ChdaD.png)

## 1.5 负载均衡演示与测试

不对啊，不是说有负载均衡的能力吗，怎么个负载均衡呢？

我们先来看看，同一个服务器实际上是可以注册很多个的，但是它们的端口不同，比如我们这里创建多个用户查询服务，我们现在将原有的端口配置修改一下，由IDEA中设定启动参数来决定，这样就可以多创建几个不同端口的启动项了：

![image-20230306225723916](https://s2.loli.net/2023/03/06/ZzmGK4CuSwLVhva.png)

![image-20230306225741406](https://s2.loli.net/2023/03/06/9lPAbiYntQDGNzy.png)

可以看到，在Eureka中，同一个服务出现了两个实例：

![image-20230306225750928](https://s2.loli.net/2023/03/06/eXGS3wdvL4RAys6.png)

现在我们稍微修改一下用户查询，然后进行远程调用，看看请求是不是均匀地分配到这两个服务端：
```java
@RestController
public class UserController {

    @Resource
    UserService service;
    
    @RequestMapping("/user/{uid}")
    public User findUserById(@PathVariable("uid") int uid){
        System.out.println("我被调用拉！");
        return service.getUserById(uid);
    }
}
```

![image-20230306225801442](https://s2.loli.net/2023/03/06/BlRf7T5iVMksb1J.png)

**可以看到，两个实例都能够均匀地被分配请求：**

![image-20230306225808889](https://s2.loli.net/2023/03/06/hysYFAeoLrnRWdX.png)

![image-20230306225818918](https://s2.loli.net/2023/03/06/nDpXICZHc4L5EQm.png)

这样，服务自动发现以及简单的负载均衡就实现完成了，并且，**如果某个微服务挂掉了，只要存在其他同样的微服务实例在运行，那么就不会导致整个微服务不可用，极大地保证了安全性**。
