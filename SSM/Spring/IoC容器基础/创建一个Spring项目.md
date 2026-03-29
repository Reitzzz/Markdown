# 第一个Spring项目

首先一定要明确，使用 Spring 首要目的是为了**使得软件项目进行解耦**，而不是为了去简化代码！通过它，我们可以更好地对我们的 Bean 进行管理。这一部分我们来体验一下 Spring 的基本使用。

Spring 并不是一个独立的框架，它实际上包含了很多的模块：



而我们首先要去学习的就是 **Core Container**，也就是核心容器模块。只有了解了 Spring 的核心技术，我们才能真正认识这个框架为我们带来的便捷之处。

## 1. 导入 Maven 依赖

Spring 是一个非入侵式的框架，就像一个工具库一样，它可以很简单地加入到我们已有的项目中。因此，我们只需要直接导入其依赖就可以使用了。Spring 核心框架的 Maven 依赖坐标如下：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>6.0.10</version>
</dependency>
```

**注意：** 与旧版本教程不同的是，Spring 6 要求使用的 Java 版本为 17 及以上，包括后续学习 Spring MVC 时，要求 Tomcat 版本必须为 10 以上。这个依赖中包含了 Spring 核心相关的内容，如 Beans、Core、Context、SpEL 以及非常关键的 AOP 框架。


## 2. 创建 Spring 配置文件

这里我们就来尝试编写一个最简的 Spring 项目。Spring 会给我们提供 IoC 容器用于管理 Bean，但是我们需要先为这个容器编写一个配置文件，通过配置文件告诉容器需要管理哪些 Bean 以及 Bean 的属性、依赖关系等。

首先，我们需要在 `resource` 目录中创建一个 Spring 配置文件（在 resource 中创建的文件，会在编译时被一起放到类路径下），命名为 `test.xml`，直接右键点击即可创建：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="[http://www.springframework.org/schema/beans](http://www.springframework.org/schema/beans)"
       xmlns:xsi="[http://www.w3.org/2001/XMLSchema-instance](http://www.w3.org/2001/XMLSchema-instance)"
       xsi:schemaLocation="[http://www.springframework.org/schema/beans](http://www.springframework.org/schema/beans)
        [https://www.springframework.org/schema/beans/spring-beans.xsd](https://www.springframework.org/schema/beans/spring-beans.xsd)">
        
</beans>
```
*(注：如果在 IDEA 中提示没有为此文件配置应用程序上下文，直接指定成当前项目或按默认配置确定即可。这只是为了代码提示和依赖关系快速查看，不配置也不会影响程序运行。)*

## 3. 使用 IoC 容器管理 Bean

Spring 为我们提供了一个 IoC 容器，用于存放我们需要使用的对象。我们可以将对象交给 IoC 容器进行管理，当我们需要使用对象时，就可以向 IoC 容器去索要，并由它来决定给我们哪一个对象。

如果我们需要使用 Spring 提供的 IoC 容器，就需要创建一个应用程序上下文（它代表的就是 IoC 容器），它会负责实例化、配置和组装 Bean：

```java
public static void main(String[] args) {
    // ApplicationContext是应用程序上下文的顶层接口，它有很多种实现，这里我们先介绍第一种
    // 因为这里使用的是XML配置文件，所以说我们就使用 ClassPathXmlApplicationContext 这个实现类
    ApplicationContext context = new ClassPathXmlApplicationContext("test.xml");  // 这里写上刚刚配置文件的名字
}
```

比如现在我们要让 IoC 容器帮助我们管理一个 `Student` 对象（Bean），当我们需要这个对象时再申请。首先定义出 `Student` 类：

```java
package com.test.bean;

public class Student {
    public void hello(){
        System.out.println("Hello World!");
    }
}
```

既然现在要让别人帮忙管理对象，那么就不能再由我们自己去 `new` 这个对象了，而是编写对应的配置。我们打开刚刚创建的 `test.xml` 文件进行编辑，添加 Bean 的配置信息：

```xml
<bean name="student" class="com.test.bean.Student"/>
```
这里我们在配置文件中编写好了对应 Bean 的信息，之后容器就会根据这里的配置进行处理。

## 4. 从容器获取 Bean

现在，这个对象不需要我们再去手动创建了，而是由 IoC 容器自动进行创建并提供。我们可以直接从上下文中获取到它为我们创建的对象。Spring 提供了多种获取 Bean 的方式，最常用的有以下两种：

### 方式一：通过 Bean 的名称获取

使用 `getBean(String name)` 方法，传入我们在 XML 配置文件中定义的 `name` 属性值。由于该方法返回的是 `Object` 类型，因此需要手动进行强制类型转换。

```java
public static void main(String[] args) {
    ApplicationContext context = new ClassPathXmlApplicationContext("test.xml");
    Student student = (Student) context.getBean("student");   // 使用getBean方法来获取名字对应的对象（Bean）
    student.hello();
}
```

### 方式二：通过 Bean 的类型获取（面向接口编程）

使用 `getBean(Class<T> requiredType)` 方法，直接传入我们需要的类或接口的 `Class` 对象。这种方式无需进行类型转换，更加安全，且完美契合面向接口编程的思想。

```java
public static void main(String[] args) {
    ApplicationContext context = new ClassPathXmlApplicationContext("test.xml");
    Service service = context.getBean(Service.class);   // 使用getBean方法来获取实现Service接口的对象（Bean）
}
```

**为什么推荐通过接口类型来获取？**

在实际项目开发中，`Service` 通常是一个抽象接口，而我们在 XML 中配置的是它的具体实现类（例如 `Aservice` 或 `Bservice`）。这种结合 IoC 容器的写法具有极大的实战价值：
* **极致的解耦：** 调用方 Java 代码只依赖 `Service` 接口，完全屏蔽了底层的具体实现。如果后续业务需求变更，需要将实现类从 `Aservice` 换成全新的 `Bservice`，我们**完全不需要修改这部分 Java 源代码**，只需在 XML 配置中将 `<bean>` 的 `class` 路径替换掉即可，实现了业务的无缝切换。
* **隐藏实现细节：** 团队协作时，开发者们可以完全基于接口进行独立开发。调用者只需关注接口定义了哪些方法，而不需要关心具体的业务逻辑是如何写的，大幅降低了模块间的耦合度与沟通成本。

实际上，这里得到的 `Student` 或 `Service` 对象，都是由 Spring 通过底层的反射机制帮助我们创建的。初学者可能会疑惑：直接 `new` 一个对象不好吗？为什么要交给 IoC 容器管理呢？

通过上述面向接口结合容器动态获取的例子，相信你已经初步体会到 IoC 解耦带来的巨大优势。在后面的学习中我们会发现，一旦对象交由 Spring 管理，它就拥有了完整的生命周期，Spring 还可以通过动态代理等技术为其附加声明式事务、AOP 日志拦截等强大的企业级功能，这都是直接 `new` 对象无法做到的。