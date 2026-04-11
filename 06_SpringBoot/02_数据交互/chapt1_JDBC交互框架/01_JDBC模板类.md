**目录**
- [1. JDBC交互框架](#1-jdbc交互框架)
  - [1.1 JDBC模版类](#11-jdbc模版类)

# 1. JDBC交互框架

除了我们前面一直认识的Mybatis之外，实际上Spring官方也提供了一个非常方便的JDBC操作工具，它同样可以快速进行增删改查。首先我们还是通过starter将依赖导入：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
```

导入完成之后就可以轻松使用了。

## 1.1 JDBC模版类

Spring JDBC为我们提供了一个非常方便的`JdbcTemplate`类，它封装了常用的JDBC操作，我们可以快速使用这些方法来实现增删改查，这里我们还是配置一下MySQL数据源信息：

```xml
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
</dependency>
```

```yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
```

我们要操作数据库，最简单直接的方法就是使用JdbcTemplate来完成：

```java
@Resource
JdbcTemplate template;
```

它给我们封装了很多方法使用，比如我们要查询数据库中的一条记录：

![image-20230716000431492](https://s2.loli.net/2023/07/16/ygRp98mDKafXkw1.png)

我们可以使用`queryForMap`快速以Map为结果的形式查询一行数据：

```java
@Test
void contextLoads() {
    Map<String, Object> map = template.queryForMap("select * from user where id = ?", 1);
    System.out.println(map);
}
```

非常方便：

![image-20230720215124918](https://s2.loli.net/2023/07/20/ijczpNxh4fXoQKv.png)

我们也可以编写自定义的Mapper用于直接得到查询结果：

```java
@Data
@AllArgsConstructor // 添加这个注解以支持带参数的构造函数
@NoArgsConstructor  // 补充一个无参构造，避免因全参构造覆盖而报错
public class User {
    int id;
    String name;
    String email;
    String password;
}
```

```java
@Test
void contextLoads() {
    User user = template.queryForObject("select * from user where id = ?",
        (r, i) -> new User(r.getInt(1), r.getString(2), r.getString(3), r.getString(4)), 1);
                                                                                     //最后的1是id的参数
    System.out.println(user);
}
```

当然除了这些之外，它还提供了`update`方法适用于各种情况的查询、更新、删除操作：

```java
@Test
void contextLoads() {
    int update = template.update("insert into user values(2, 'admin', '654321@qq.com', '123456')");
    System.out.println("更新了 "+update+" 行");
}
```

这样，如果是那种非常小型的项目，甚至是测试用例的话，都可以快速使用JdbcTemplate快速进行各种操作。