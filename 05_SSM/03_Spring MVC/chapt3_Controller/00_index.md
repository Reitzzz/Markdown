# SpringMVC 控制器核心知识目录索引

* [2. Controller控制器](01_配置视图解析器和控制器.md#2-controller控制器)
  * [2.1 配置视图解析器和控制器](01_配置视图解析器和控制器.md#21-配置视图解析器和控制器)
* [2.2 @RequestMapping详解](02_@RequestMapping详解.md#22-requestmapping详解)
  * [2.2.1 路径映射与通配符](02_@RequestMapping详解.md#221-路径映射与通配符)
  * [2.2.2 请求方法限定](02_@RequestMapping详解.md#222-请求方法限定)
  * [2.2.3 请求参数与请求头限定](02_@RequestMapping详解.md#223-请求参数与请求头限定)
* [2.3 @RequestParam和@RequestHeader详解](03_@RequestParam和@RequestHeader讲解.md#23-requestparam和requestheader详解)
  * [2.3.1 @RequestParam基本使用](03_@RequestParam和@RequestHeader讲解.md#231-requestparam基本使用)
  * [2.3.2 引入Servlet原生API对象](03_@RequestParam和@RequestHeader讲解.md#232-引入servlet原生api对象)
  * [2.3.3 参数绑定到实体类](03_@RequestParam和@RequestHeader讲解.md#233-参数绑定到实体类)
  * [2.3.4 @RequestHeader注解](03_@RequestParam和@RequestHeader讲解.md#234-requestheader注解)
* [2.4 @CookieValue和@SessionAttribute](04_@CookieValue和@SessionAttribute.md#24-cookievalue和sessionattribute)
  * [2.4.1 @CookieValue注解](04_@CookieValue和@SessionAttribute.md#241-cookievalue注解)
  * [2.4.2 @SessionAttribute注解](04_@CookieValue和@SessionAttribute.md#242-sessionattribute注解)
* [2.5 重定向和请求转发](05_重定向和请求转发.md#25-重定向和请求转发)
  * [2.5.1 重定向](05_重定向和请求转发.md#251-重定向)
  * [2.5.2 请求转发](05_重定向和请求转发.md#252-请求转发)
* [2.6 Bean的Web作用域](06_Bean的作用域.md#26-bean的web作用域)

> 注：关于“Bean的Web作用域”（包含request、session等作用域的细分），在现代SpringMVC开发中主要以无状态（Stateless）单例Controller为主，该部分在实际开发中较少涉及，故此处不作展开的详细目录跳转。