# 目录

- [目录](#目录)
- [1. OpenFeign实现负载均衡](#1-openfeign实现负载均衡)
  - [1.1 引入依赖](#11-引入依赖)
  - [1.2 开启Feign客户端](#12-开启feign客户端)
  - [1.3 声明Feign客户端接口](#13-声明feign客户端接口)
  - [1.4 注入并使用客户端](#14-注入并使用客户端)
  - [1.5 改造其他微服务调用](#15-改造其他微服务调用)
  - [1.6 其他配置说明](#16-其他配置说明)

---

# 1. OpenFeign实现负载均衡

官方文档：https://docs.spring.io/spring-cloud-openfeign/docs/current/reference/html/

Feign和RestTemplate一样，也是HTTP客户端请求工具，但是它的使用方式更加便捷。首先是依赖：

## 1.1 引入依赖
**OpenFeign 依赖一般加在“发起调用的那个微服务”里，也就是消费者/调用方**

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

## 1.2 开启Feign客户端

接着在启动类添加`@EnableFeignClients`注解：
```java
@SpringBootApplication
@EnableFeignClients
public class BorrowApplication {
    public static void main(String[] args) {
        SpringApplication.run(BorrowApplication.class, args);
    }
}
```

## 1.3 声明Feign客户端接口

那么现在我们需要调用其他微服务提供的接口，该怎么做呢？我们直接创建一个对应服务的接口类即可：
```java
@FeignClient("userservice")   //声明为userservice服务的HTTP请求客户端
public interface UserClient {
}
```

接着我们直接创建所需类型的方法，比如我们之前的：

```java
RestTemplate template = new RestTemplate();
User user = template.getForObject("http://userservice/user/"+uid, User.class);
```

现在可以直接写成这样：
```java
@FeignClient("userservice")
public interface UserClient {

    //路径保证和其他微服务提供的一致即可
    @RequestMapping("/user/{uid}")
    User getUserById(@PathVariable("uid") int uid);  //参数和返回值也保持一致
}
```

## 1.4 注入并使用客户端

接着我们直接注入使用（有Mybatis那味了）：
```java
@Resource
UserClient userClient;

@Override
public UserBorrowDetail getUserBorrowDetailByUid(int uid) {
    List<Borrow> borrow = mapper.getBorrowsByUid(uid);
    
    User user = userClient.getUserById(uid);
    //这里不用再写IP，直接写服务名称bookservice
    List<Book> bookList = borrow
            .stream()
            .map(b -> template.getForObject("http://bookservice/book/"+b.getBid(), Book.class))
            .collect(Collectors.toList());
    return new UserBorrowDetail(user, bookList);
}
```

访问，可以看到结果依然是正确的：

![image-20230306230054245](https://s2.loli.net/2023/03/06/koMOYnxtq8UPiac.png)

并且我们可以观察一下两个用户微服务的调用情况，也是以负载均衡的形式进行的。

## 1.5 改造其他微服务调用

按照同样的方法，我们接着将图书管理服务的调用也改成接口形式：

![image-20230306230101467](https://s2.loli.net/2023/03/06/GiBa7FzQsvkIpdS.png)

最后我们的Service代码就变成了：
```java
@Service
public class BorrowServiceImpl implements BorrowService {

    @Resource
    BorrowMapper mapper;

    @Resource
    UserClient userClient;
    
    @Resource
    BookClient bookClient;

    @Override
    public UserBorrowDetail getUserBorrowDetailByUid(int uid) {
        List<Borrow> borrow = mapper.getBorrowsByUid(uid);

        User user = userClient.getUserById(uid);
        List<Book> bookList = borrow
                .stream()
                .map(b -> bookClient.getBookById(b.getBid()))
                .collect(Collectors.toList());
        return new UserBorrowDetail(user, bookList);
    }
}
```

继续访问进行测试：

![image-20230306230112322](https://s2.loli.net/2023/03/06/V5fhk2xLo8bXmWA.png)

OK，正常。

## 1.6 其他配置说明

当然，Feign也有很多的其他配置选项，这里就不多做介绍了，详细请查阅官方文档。
````</Book></Borrow></Book></Borrow>