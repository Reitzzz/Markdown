# Servlet 核心技术与生命周期

本章核心：掌握 Servlet 的底层实现方式、生命周期管理、访问路由配置，以及 Web 应用全局对象和初始化参数的使用。

## 1. Servlet 基础与生命周期
* [01-Servlet (接口实现) ＆生命周期](Servlet（接口实现）＆生命周期.md)
  * Servlet 接口的五大核心方法 (`init`, `service`, `destroy`, `getServletConfig`, `getServletInfo`)
  * Servlet 的完整生命周期与对应调用时机
* [02-Servlet (继承实现)](Servlet%20(继承实现).md)
  * HttpServlet 的底层原理 (协议转换、请求分发、精准路由)
  * `doGet` 与 `doPost` 方法的使用场景与代码实现

## 2. Servlet 路由与参数配置
* [03-@WebServlet注解详解](@WebServlet注解详解.md)
  * 访问路径配置 (目录匹配、扩展名匹配、默认 Servlet、多路径)
  * 启动加载优先级配置 (`loadOnStartup`)
* [04-初始化参数](初始化参数.md)
  * 局部初始化参数配置 (`@WebInitParam`) 及获取
  * 全局初始化参数配置 (`web.xml` 的 `<context-param>`) 及获取

## 3. 全局上下文管理
* [05-了解ServletContext对象](了解ServletContext对象.md)
  * 获取 ServletContext 全局唯一对象
  * 使用 ServletContext 设置与获取附加值 (跨请求数据传递)
  * 使用 ServletContext 实现请求转发
  * 获取 WebApp 根目录下的资源文件