## 配置环境并搭建项目

这里我们继续使用之前的Tomcat10服务器，Spring6之后要求必须使用Tomcat10或更高版本，跟之前一样，我们直接创建一个新的JakartaEE项目。

![image-20230219162053172](https://s2.loli.net/2023/02/19/4IucyfBKsLzASNJ.png)

创建完成后会自动生成相关文件，但是还是请注意检查运行配置中的URL和应用程序上下文名称是否一致。

### 传统XML配置形式

SpringMvc项目依然支持多种配置形式，这里我们首先讲解最传统的XML配置形式。

首先我们需要添加Mvc相关依赖：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>6.0.10</version>
</dependency>
```

接着我们需要配置一下web.xml，将DispatcherServlet替换掉Tomcat自带的Servlet，这里url-pattern需要写为`/`，即可完成替换：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="[https://jakarta.ee/xml/ns/jakartaee](https://jakarta.ee/xml/ns/jakartaee)"
         xmlns:xsi="[http://www.w3.org/2001/XMLSchema-instance](http://www.w3.org/2001/XMLSchema-instance)"
         xsi:schemaLocation="[https://jakarta.ee/xml/ns/jakartaee](https://jakarta.ee/xml/ns/jakartaee) [https://jakarta.ee/xml/ns/jakartaee/web-app_5_0.xsd](https://jakarta.ee/xml/ns/jakartaee/web-app_5_0.xsd)"
         version="5.0">
    <servlet>
        <servlet-name>mvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>mvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

接着需要为整个Web应用程序配置一个Spring上下文环境（也就是容器），因为SpringMVC是基于Spring开发的，它直接利用Spring提供的容器来实现各种功能，那么第一步依然跟之前一样，需要编写一个配置文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="[http://www.springframework.org/schema/beans](http://www.springframework.org/schema/beans)"
       xmlns:xsi="[http://www.w3.org/2001/XMLSchema-instance](http://www.w3.org/2001/XMLSchema-instance)"
       xsi:schemaLocation="[http://www.springframework.org/schema/beans](http://www.springframework.org/schema/beans)
        [https://www.springframework.org/schema/beans/spring-beans.xsd](https://www.springframework.org/schema/beans/spring-beans.xsd)">
</beans>
```

接着我们需要为DispatcherServlet配置一些初始化参数来指定刚刚创建的配置文件：

```xml
<servlet>
    <servlet-name>mvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:application.xml</param-value>
    </init-param>
</servlet>
```

这样我们就完成了基本的配置，现在我们可以来测试一下是否配置正确，我们删除项目自带的Servlet类，创建一个Mvc中使用的Controller类，现在还没学没关系，跟着写就行了，这里我们只是测试一下：

```java
@Controller
public class HelloController {
    @ResponseBody
    @RequestMapping("/")
    public String hello(){
        return "HelloWorld!";
    }
}
```

接着我们需要将这个类注册为Bean才能正常使用，我们来编写一下Spring的配置文件，这里我们直接配置包扫描，XML下的包扫描需要这样开启：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="[http://www.springframework.org/schema/beans](http://www.springframework.org/schema/beans)"
       xmlns:xsi="[http://www.w3.org/2001/XMLSchema-instance](http://www.w3.org/2001/XMLSchema-instance)"
       xmlns:context="[http://www.springframework.org/schema/context](http://www.springframework.org/schema/context)"
       xsi:schemaLocation="[http://www.springframework.org/schema/beans](http://www.springframework.org/schema/beans)
        [https://www.springframework.org/schema/beans/spring-beans.xsd](https://www.springframework.org/schema/beans/spring-beans.xsd) [http://www.springframework.org/schema/context](http://www.springframework.org/schema/context) [https://www.springframework.org/schema/context/spring-context.xsd](https://www.springframework.org/schema/context/spring-context.xsd)">
    <context:component-scan base-package="com.example"/>
</beans>
```

如果可以成功在浏览器中出现HelloWorld则说明配置成功：

![image-20230219170637540](https://s2.loli.net/2023/02/19/D1sAFePzj7d49VL.png)

聪明的小伙伴可能已经发现了，实际上我们上面编写的Controller就是负责Servlet基本功能的，比如这里我们返回的是HelloWorld字符串，那么我们在访问这个地址的时候，得到的就是这里返回的字符串，可以看到写法非常简洁，至于这是怎么做到的，怎么使用，我们会在本章进行详细介绍。