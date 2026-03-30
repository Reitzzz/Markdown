# Spring IoC 学习指南目录

## 理论基础 (建议理解)
* [IoC理论介绍](IoC理论介绍.md)

## 传统 XML 配置时代 (开发中已淘汰，了解概念即可)
* [创建一个Spring项目](创建一个Spring项目.md)
* [Bean的注册与配置](Bean的注册与配置.md)
* [依赖注入](依赖注入.md)
* [自动装配](自动装配.md)
* [生命周期与继承](生命周期与继承.md)
* [工厂模式与工厂Bean](工厂模式与工厂Bean.md)

## 现代注解开发 (核心重点，开发主流)
* [使用注解开发](注解开发.md)
    * [1. 注解开发基础配置](注解开发.md#1-注解开发基础配置)
    * [2. 使用 @Bean 注册 Bean](注解开发.md#2-使用-bean-注册-bean)
    * [3. Bean 属性与生命周期配置](注解开发.md#3-bean-属性与生命周期配置)
    * [4. 自动装配详解](注解开发.md#4-自动装配详解)
    * [5. 组件扫描与 @Component](注解开发.md#5-组件扫描与-component)
    * [6. 进阶与总结](注解开发.md#6-进阶与总结)

---

## 附录: Spring IoC 注解开发基本用法总结

现代企业开发已经全面拥抱注解配置。以下是按照实际开发顺序总结的基本用法：

### 第一步：引入依赖
在 Maven 项目中引入 Spring 核心容器依赖。Spring 6 要求使用的 Java 版本为 17 及以上。

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>6.0.10</version>
</dependency>
```

### 第二步：创建配置类并开启扫描
使用 `@Configuration` 标识这是一个配置类，以此来替代过去的 XML 文件。同时使用 `@ComponentScan` 指定 IoC 容器需要查找 Bean 的包路径。

```java
@Configuration
@ComponentScan("com.test.bean")
public class MainConfiguration {
}
```

### 第三步：将对象注册为 Bean 

#### 3.1 自动注册（组件扫描）
在自己编写的业务类上添加 `@Component` 注解，配合第二步的包扫描，Spring 容器会自动发现并将其交由 IoC 容器管理。

```java
@Component
public class Teacher {
}
```

#### 3.2 手动注册（@Bean 方法）
对于无法添加注解的第三方库类，或者需要手动干预实例化过程的对象，可以在配置类中编写返回该对象实例的方法，并加上 `@Bean` 注解。

```java
@Configuration
public class BeanConfiguration {
    
    @Bean
    public DataSource dataSource() {
        return new MysqlDataSource();
    }
    
}
```

### 第四步：实现自动依赖注入
在需要依赖其他对象的类中，声明对应的成员变量，并使用 `@Autowired` 或 `@Resource` 注解。Spring 容器会自动寻找匹配的 Bean 并完成赋值，从而实现对象之间的解耦。

```java
@Component
public class Student {
    
    @Autowired
    private Teacher teacher;
    
}
```

### 第五步：初始化容器并获取对象
在程序的启动入口处，通过 `AnnotationConfigApplicationContext` 加载编写好的配置类来启动 IoC 容器。随后即可向容器获取装配完毕的 Bean 实例。

```java
public class Main {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(MainConfiguration.class);
        Student student = context.getBean(Student.class);
    }
}
```