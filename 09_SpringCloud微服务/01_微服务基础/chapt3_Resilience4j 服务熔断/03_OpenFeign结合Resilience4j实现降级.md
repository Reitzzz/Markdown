# 1. OpenFeign结合Resilience4j实现降级

## 目录
- [1. OpenFeign结合Resilience4j实现降级](#1-openfeign结合resilience4j实现降级)
  - [目录](#目录)
  - [1.1 OpenFeign结合Resilience4j降级概述](#11-openfeign结合resilience4j降级概述)
  - [1.2 实现替代方案](#12-实现替代方案)
  - [1.3 配置替代实现与Controller修改](#13-配置替代实现与controller修改)
  - [1.4 开启熔断支持与测试](#14-开启熔断支持与测试)

---

## 1.1 OpenFeign结合Resilience4j降级概述

Resilience4j 也可以配合 Feign 进行降级，我们可以对应接口中定义的远程调用单独进行降级操作。在现代的 Spring Cloud 版本中，Resilience4j 已经替代 Hystrix 成为默认的断路器实现。

比如我们还是以用户服务挂掉为例，那么这个时候肯定是会远程调用失败的，也就是说我们的 Controller 中的方法在执行过程中会直接抛出异常，进而被 Resilience4j 断路器监控到并进行服务降级。

## 1.2 实现替代方案

而实际上导致方法执行异常的根源就是远程调用失败，所以我们换个思路，既然用户服务调用失败，那么我就给这个远程调用添加一个替代方案，如果此远程调用失败，那么就直接上替代方案。那么怎么实现替代方案呢？我们知道 Feign 都是以接口的形式来声明远程调用，那么既然远程调用已经失效，我们就自行对其进行实现，创建一个实现类，对原有的接口方法进行替代方案实现：
```java
@Component   //注意，需要将其注册为Bean，Feign才能自动注入
public class UserFallbackClient implements UserClient{
    @Override
    public User getUserById(int uid) {   //这里我们自行对其进行实现，并返回我们的替代方案
        User user = new User();
        user.setName("我是替代方案");
        return user;
    }
}
```

## 1.3 配置替代实现与Controller修改

实现完成后，我们只需要在原有的接口中指定失败替代实现即可（得益于 OpenFeign 的抽象，这里的代码与底层使用何种熔断器无关）：
```java
//fallback参数指定为我们刚刚编写的实现类
@FeignClient(value = "userservice", fallback = UserFallbackClient.class)
public interface UserClient {

    @RequestMapping("/user/{uid}")
    User getUserById(@PathVariable("uid") int uid);
}
```

现在保持 `BorrowController` 处于干净的状态，不需要添加额外的断路器注解，也不需要备选方法：
```java
@RestController
public class BorrowController {

    @Resource
    BorrowService service;

    @RequestMapping("/borrow/{uid}")
    UserBorrowDetail findUserBorrows(@PathVariable("uid") int uid){
        return service.getUserBorrowDetailByUid(uid);
    }
}
```

## 1.4 开启熔断支持与测试

最后我们在配置文件中开启 Spring Cloud Circuit Breaker（底层由 Resilience4j 驱动）对 Feign 的支持：
```yaml
spring:
  cloud:
    openfeign:
      circuitbreaker:
        enabled: true
```
*(注：如果使用的是较老的 Spring Cloud 版本，配置可能是 `feign.circuitbreaker.enabled: true`，在新版本中统一规范为了上述格式)*

启动服务，调用接口试试看：

![image-20230306230301021](https://s2.loli.net/2023/03/06/bieclsNmpqOrdHB.png)

![image-20230306230310524](https://s2.loli.net/2023/03/06/3dn7AlkYGJCmUxw.png)

可以看到，现在已经采用我们的替代方案作为结果。