# @WebServlet 注解详解

可以直接使用 `@WebServlet` 注解来快速注册一个 Servlet。下面详细介绍此注解的各种配置项。

## 1. 访问路径配置

`name` 属性用于指定 Servlet 的名称。`urlPatterns` 和 `value` 属性功能相同，用于配置当前 Servlet 的访问路径。访问路径不仅可以是固定值，还支持多种通配符匹配方式：

### 目录匹配

```java
@WebServlet("/test/*")
```

上面的路径表示，所有匹配 `/test/` 加上任意后续名称的路径，都可以访问此 Servlet。

### 扩展名匹配

```java
@WebServlet("*.js")
```

获取任何以 `.js` 结尾的文件请求，都会由当前定义的 Servlet 进行处理。

### 默认 Servlet 匹配

```java
@WebServlet("/")
```

此路径和 Tomcat 默认为我们提供的 Servlet 冲突，会直接替换掉默认的 Servlet。其含义为：如果当前访问路径没有找到任何匹配的 Servlet，就会使用此 Servlet 进行兜底处理。

### 多路径匹配

可以为一个 Servlet 同时配置多个访问路径：

```java
@WebServlet({"/test1", "/test2"})
```

## 2. 启动加载配置 (loadOnStartup)

`loadOnStartup` 属性决定了是否在 Tomcat 启动时就加载此 Servlet。

* 默认情况下，Servlet 只有在被访问时才会加载，其默认值为 -1，表示不在启动时加载。
* 可以将其修改为大于等于 0 的整数，从而开启启动时加载功能。
* 填入数字的大小决定了此 Servlet 的启动优先级，数字越小优先级越高。

```java
@Log
@WebServlet(value = "/test", loadOnStartup = 1)
public class TestServlet extends HttpServlet {

    @Override
    public void init() throws ServletException {
        super.init();
        log.info("我被初始化了！");
    }

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html;charset=UTF-8");
        resp.getWriter().write("<h1>恭喜你解锁了全新玩法</h1>");
    }
}
```

其他内容属于 Servlet 的基本配置，此处不再详细展开。