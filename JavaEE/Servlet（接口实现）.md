# Servlet (接口实现)

其本质是一个接口，由以下几大方法组成

## public void init(略)   初始化Servlet类

1. 调用时机： 默认情况下，Servlet第一次被访问时调用
   
2. 调用次数：一次

3. loadOnStartup:用来控制启动顺序 <br>
 负整数为第一次被访问时创建对象 <br>0或者正整数为服务器启动时创建对象 数字越小优先级越高

```c 
 public void init(ServletConfig config) throws ServletException {
    this.servletConfig = config;
    System.out.println("Servlet initialized");
}
```

## public void service(略)  请求处理 提供具体服务

1. 调用时机：每一次Servlet被访问时调用
2. 调用次数：多次

```c
@Override
public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
    res.setContentType("text/html;charset=UTF-8");
    PrintWriter out = res.getWriter();
    out.println("<h1>Hello World</h1>");
}
```

## public void destroy()  销毁方法

1. 调用时机：内存释放或者服务器关闭 Servlet对象被销毁
2. 调用次数：一次

```c
@Override
public void destroy() {
    System.out.println("Servlet destroyed");
}
```

## public ServletConfig getServletConfig() 获取配置对象方法

1. 作用：返回 Servlet 容器调用 init() 方法时传递给当前 Servlet 的 ServletConfig 对象，包含 Servlet 的初始化参数。

2. 调用时机：在 Servlet 初始化后，任何需要获取配置信息的时候

```c
@Override
public ServletConfig getServletConfig() {
    return this.servletConfig;
}
```

## public String getServletInfo() 获取附加信息方法

1. 作用：返回一段字符串，包含关于该 Servlet 的基本信息，例如作者、版本、版权等。

2. 调用时机：通常由 Servlet 容器或服务器管理工具在需要展示 Servlet 详情时调用。

```c
@Override
public String getServletInfo() {
    return "Version: 1.0, Author: Admin";
}
```


## 总体代码实现

```c
import javax.servlet.Servlet;
import javax.servlet.ServletConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

public class DemoServlet implements Servlet {

    private ServletConfig servletConfig;

    @Override
    public void init(ServletConfig config) throws ServletException {
        this.servletConfig = config;
        System.out.println("Servlet initialized");
    }

    @Override
    public ServletConfig getServletConfig() {
        return this.servletConfig;
    }

    @Override
    public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
        res.setContentType("text/html;charset=UTF-8");
        PrintWriter out = res.getWriter();
        out.println("<h1>Hello World</h1>");
    }

    @Override
    public String getServletInfo() {
        return "Version: 1.0, Author: Admin";
    }

    @Override
    public void destroy() {
        System.out.println("Servlet destroyed");
    }
}
```