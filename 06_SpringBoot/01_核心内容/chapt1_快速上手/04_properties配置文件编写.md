**目录**

- [1 配置文件介绍](#1-配置文件介绍)
  - [1.1 properties 格式配置文件与自定义配置](#11-properties-格式配置文件与自定义配置)
  - [1.2 YAML 格式配置文件](#12-yaml-格式配置文件)
  - [1.3 数据源配置](#13-数据源配置)
  - [1.4 MyBatis 整合与配置](#14-mybatis-整合与配置)
  - [1.5 其他常见配置](#15-其他常见配置)

---

# 1 配置文件介绍

## 1.1 properties 格式配置文件与自定义配置

前面我们已经体验了SpringBoot带来的快捷开发体验，不过我们发现有些东西还是需要我们自己来编写配置才可以，不然SpringBoot项目无法正常启动，我们来看看如何编写配置。我们可以直接在`application.properties`中进行配置编写，它是整个SpringBoot的配置文件，比如要修改服务器的默认端口：

![image-20230715232124133](https://s2.loli.net/2023/07/15/E3nsZG7DcaSzOBY.png)

这些配置其实都是各种Starter提供的，部分配置在Starter中具有默认值，我们即使不配置也会使用默认值，比如这里的8080就是我们服务器的默认端口，我们也可以手动修改它，来变成我们需要的。

除了配置已经存在的选项，我们也可以添加自定义的配置，来方便我们程序中使用，比如我们这里创建一个测试数据：

![image-20230715234130924](https://s2.loli.net/2023/07/15/HJWz7PIl6Sgk1nx.png)

我们可以直接在程序中通过`@Value`来访问到（跟我们之前Spring基础篇讲的是一样的）

```java
@Controller
public class TestController {
    @Value("${test.data}")
    int data;   //直接从配置中去取
}
```

## 1.2 YAML 格式配置文件

配置文件除了使用`properties`格式以外，还有一种叫做`yaml`格式，它的语法如下：

```yaml
一级目录:
    二级目录:
      三级目录1: 值
      三级目录2: 值
      三级目录List: 
      - 元素1
      - 元素2
      - 元素3
```

我们可以看到，每一级目录都是通过缩进（不能使用Tab，只能使用空格）区分，并且键和值之间需要添加冒号+空格来表示。

SpringBoot也支持这种格式的配置文件，我们可以将`application.properties`修改为`application.yml`或是`application.yaml`来使用YAML语法编写配置：

```yaml
server:
  port: 80
```

## 1.3 数据源配置

现在我们来尝试为之前的数据源进行一下配置，这样才能正常启动我们的服务器：

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
```

配置完成后，我们就可以正常启动服务器了。

## 1.4 MyBatis 整合与配置

这里我们接续来测试一下MyBatis的配置，想要在SpringBoot中使用Mybatis也很简单，不需要进行任何配置，我们直接编写Mapper即可，这里我们随便创建一个表试试看：

![image-20230716000431492](https://s2.loli.net/2023/07/16/ygRp98mDKafXkw1.png)

```java
@Data
public class User {
    int id;
    String name;
    String email;
    String password;
}
```

注意，在SpringBoot整合之后，我们只需要直接在配置类上添加`@MapperScan`注解即可，跟我们之前的使用方法是一样的：

```java
@Configuration
@MapperScan("com.example.mapper")
public class WebConfiguration implements WebMvcConfigurer {
  ...
```

不过，为了方便，我们也可以直接为需要注册为Mapper的接口添加`@Mapper`注解，来表示这个接口作为Mapper使用：

![image-20230716000755756](https://s2.loli.net/2023/07/16/lTrXepw1c38IdSv.png)

这样，即使不配置MapperScan也能直接注册为Mapper正常使用，是不是感觉特别方便？

```java
@Mapper
public interface UserMapper {
    @Select("select * from user where id = #{id}")
    User findUserById(int id);
}
```

```java
@ResponseBody
@GetMapping("/test")
public User test(){
        return mapper.findUserById(1);
}
```

访问接口测试一下：

![image-20230716001311316](https://s2.loli.net/2023/07/16/PSfpylWGCs3bzZj.png)

## 1.5 其他常见配置

最后，我们再来介绍一下常见的配置项，比如SpringSecurity和SpringBootMvc配置：

```yaml
spring:  
  #  Spring Mvc相关配置
  mvc:
    static-path-pattern: /static/**   #静态资源解析地址
  # Spring Security 相关配置
  security:
    filter:
      order: -100 #Spring Security 过滤器优先级
    user:
      name: 'admin'   #默认登录用户名
      password: '123456'   #默认登录密码
      roles:    #默认用户的角色
        - admin
        - user
```

更多的配置我们可以在后续的学习中继续认识，这些配置其实都是由Starter提供的，确实极大程度简化了我们对于框架的使用。