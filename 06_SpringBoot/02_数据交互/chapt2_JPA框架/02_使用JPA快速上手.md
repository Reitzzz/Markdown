# 1. 使用JPA快速上手

**目录**
- [1. 使用JPA快速上手](#1-使用jpa快速上手)
  - [1.1 引入依赖](#11-引入依赖)
  - [1.2 实体类配置](#12-实体类配置)
  - [1.3 配置文件与自动建表](#13-配置文件与自动建表)
  - [1.4 编写Repository接口](#14-编写repository接口)
  - [1.5 测试基本操作](#15-测试基本操作)
  - [1.6 JPA与Mybatis的对比](#16-jpa与mybatis的对比)

---

## 1.1 引入依赖

同样的，我们只需要导入stater依赖即可：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

## 1.2 实体类配置

接着我们可以直接创建一个类，比如用户类，我们只需要把一个账号对应的属性全部定义好即可：

```java
@Data
public class Account {
    int id;
    String username;
    String password;
}
```

接着，我们可以通过注解形式，在属性上添加数据库映射关系，这样就能够让JPA知道我们的实体类对应的数据库表长啥样，这里用到了很多注解：

```java
@Data
@Entity   //表示这个类是一个实体类
@Table(name = "account")    //对应的数据库中表名称
public class Account {

    @GeneratedValue(strategy = GenerationType.IDENTITY)   //生成策略，这里配置为自增
    @Column(name = "id")    //对应表中id这一列
    @Id     //此属性为主键
    int id;

    @Column(name = "username")   //对应表中username这一列
    String username;

    @Column(name = "password")   //对应表中password这一列
    String password;
}
```

## 1.3 配置文件与自动建表

接着我们来修改一下配置文件，把日志打印给打开：

```yaml
spring:
  jpa:
    #开启SQL语句执行日志信息
    show-sql: true
    hibernate:
      #配置为检查数据库表结构，没有时会自动创建
      ddl-auto: update
```

`ddl-auto`属性用于设置自动表定义，可以实现自动在数据库中为我们创建一个表，表的结构会根据我们定义的实体类决定，它有以下几种：

- `none`: 不执行任何操作，**数据库表结构需要手动创建**。
- `create`: 框架在每次运行时都会**删除所有表，并重新创建**。
- `create-drop`: 框架在每次运行时都会删除所有表，然后再创建，**但在程序结束时会再次删除所有表**。
- `update`: 框架会检查数据库表结构，**如果与实体类定义不匹配，则会做相应的修改，以保持它们的一致性**。
- `validate`: 框架会检查数据库表结构与实体类定义是否匹配，**如果不匹配，则会抛出异常**。

这个配置项的作用是为了避免手动管理数据库表结构，使开发者可以更方便地进行开发和测试，但在生产环境中，更推荐使用数据库迁移工具来管理表结构的变更。

我们可以在日志中发现，在启动时执行了如下SQL语句：

![image-20230720235136506](https://s2.loli.net/2023/07/20/kABZVhJ8vjKSqzT.png)

我们的数据库中对应的表已经自动创建好了。

## 1.4 编写Repository接口

我们接着来看如何访问我们的表，我们需要创建一个Repository实现类：

```java
@Repository
public interface AccountRepository extends JpaRepository<Account, Integer> {
}
```

注意JpaRepository有两个泛型，前者是具体操作的对象实体，也就是对应的表，后者是ID的类型，接口中已经定义了比较常用的数据库操作。

## 1.5 测试基本操作

编写接口继承即可，我们可以直接注入此接口获得实现类：

```java
@Resource
AccountRepository repository;

@Test
void contextLoads() {
    Account account = new Account(); // 也可以调用repository下的多个方法
                                     //如deleteAll() findAll()等
    account.setUsername("小红");
    account.setPassword("1234567");
    System.out.println(repository.save(account).getId());   //使用save来快速插入数据，并且会返回插入的对象，如果存在自增ID，对象的自增id属性会自动被赋值，这就很方便了
}
```

执行结果如下：

![image-20230720235640148](https://s2.loli.net/2023/07/20/ksI3J5eidzTrvyL.png)

同时，查询操作也很方便：

```java
@Test
void contextLoads() {
  	//默认通过通过ID查找的方法，并且返回的结果是Optional包装的对象，非常人性化
    repository.findById(1).ifPresent(System.out::println);
}
```

得到结果为：

![image-20230720235949290](https://s2.loli.net/2023/07/20/TRHOWbop267Al4Q.png)

包括常见的一些计数、删除操作等都包含在里面，仅仅配置应该接口就能完美实现增删改查：

![image-20230721000050875](https://s2.loli.net/2023/07/21/uIBciLqFsH5tdDR.png)

## 1.6 JPA与Mybatis的对比

我们发现，使用了JPA之后，整个项目的代码中没有出现任何的SQL语句，可以说是非常方便了，JPA依靠我们提供的注解信息自动完成了所有信息的映射和关联。

相比Mybatis，JPA几乎就是一个全自动的ORM框架，而Mybatis则顶多算是半自动ORM框架。