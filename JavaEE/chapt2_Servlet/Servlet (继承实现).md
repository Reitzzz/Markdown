# Servlet（继承实现）

分为GenericServlet 和HttpServlet（最常用）

## HttpServlet

### 底层原理
通过继承HttpServlet类实现

协议转换：容器调用公共的 service(ServletRequest, ServletResponse) 方法时，HttpServlet 首先将这两个通用的请求和响应对象向下转型为 HTTP 专用的 HttpServletRequest 和 HttpServletResponse。

请求分发：转型后，它会调用自己内部重载的、受保护的 service(HttpServletRequest, HttpServletResponse) 方法。

精准路由：这个受保护的 service 方法会解析 HTTP 请求的方法类型（调用 request.getMethod()），如果发现是 GET 请求，就调用 doGet()；如果是 POST 请求，就调用 doPost()，依此类推。

### 组成

1. doGet()方法

doGet 专门用于处理 HTTP 的 GET 请求。在 HTTP 协议的设计语义中，GET 请求主要用于从服务器**获取数据**，而不会对服务器上的资源产生修改操作 

触发场景：

1. 用户在浏览器地址栏直接输入 URL 并回车。
2. 用户点击网页上的超链接。
3. HTML 表单中设置了 method="get" 时的提交。
4. 前端通过 AJAX/Fetch 发起 GET 请求 <br>

数据传递方式： 客户端传递给服务器的参数会直接附加在 URL 的末尾，以 ?key=value&key2=value2 的形式拼接

```java
@Override
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    String keyword = request.getParameter("keyword");
    
    response.setContentType("text/html;charset=UTF-8");
    response.getWriter().println("Search result for: " + keyword);
}
```

2.doPost()方法

doPost 专门用于处理 HTTP 的 POST 请求。在设计语义中，POST 请求主要用于向服务器**提交数据**，通常会触发服务器端的数据修改、新增记录等操作

触发场景：

1. HTML 表单中设置了 method="post" 时的提交。
2. 前端通过 AJAX/Fetch 发起 POST 请求（常用于提交 JSON 数据）。

```java
@Override
protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    String username = request.getParameter("username");
    
    response.setContentType("text/html;charset=UTF-8");
    response.getWriter().println("User created: " + username);
}
```

### 总体代码实现

```java
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

@WebServlet("/user")
public class UserServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        response.setContentType("text/html;charset=UTF-8");
        PrintWriter out = response.getWriter();
        out.println("<h1>GET Request Processed</h1>");
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws IOException {
        this.doGet(request, response);
    }
}
```