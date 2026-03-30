# Filter 过滤器

## 1. 为什么需要过滤器？

有了 Session 之后，我们可以很好地控制用户的登录验证。但是，如果针对每个页面单独配置权限校验会极其繁琐。过滤器（Filter）相当于在所有访问路径前加了一堵墙。来自浏览器的所有请求都会首先经过过滤器，只有过滤器允许通过的请求，才能顺利到达对应的 Servlet。对于不允许通过的请求，我们可以自由控制是进行重定向还是请求转发。

并且，应用程序中可以添加多个过滤器，请求需要穿过层层过滤器链才能最终到达 Servlet。

## 2. 添加过滤器的基本方式

添加一个过滤器非常简单，只需实现 `Filter` 接口，并添加 `@WebFilter` 注解：

```java
// 路径的匹配规则和Servlet一致，"/*"表示拦截所有请求
@WebFilter("/*")   
public class TestFilter implements Filter {
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) servletRequest;
        // 打印请求的URL，验证所有请求是否都经过了此过滤器
        System.out.println(request.getRequestURL());
    }
}
```

此时发起请求，会发现所有的请求都经过了此过滤器，但都没有得到任何响应内容。这是因为请求被拦截后，默认没有被放行。

## 3. 请求放行与 FilterChain

要让请求顺利到达对应的 Servlet，需要在 `doFilter` 方法的最后调用：

```java
// 将请求继续传递给下一个过滤器，如果没有下一个过滤器，则到达对应的Servlet
filterChain.doFilter(servletRequest, servletResponse);
```

如果有多个过滤器，它们的执行顺序类似于递归结构。当 `doFilter` 被调用时，请求会一直向下传递直到 Servlet 处理完成，然后又依次逆向返回到最外层的 Filter。过滤器的执行顺序默认按照类名的自然排序进行。

```java
@WebFilter("/*")
public class TestFilter1 implements Filter {
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("我是1号过滤器");
        filterChain.doFilter(servletRequest, servletResponse);
        System.out.println("我是1号过滤器，处理后");
    }
}

@WebFilter("/*")
public class TestFilter2 implements Filter {
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("我是2号过滤器");
        filterChain.doFilter(servletRequest, servletResponse);
        System.out.println("我是2号过滤器，处理后");
    }
}
```

执行结果的顺序会是：1号 -> 2号 -> Servlet处理 -> 2号处理后 -> 1号处理后。

## 4. HttpFilter 专用类

同 Servlet 一样，Filter 也有针对 HTTP 请求进行专门处理的父类 `HttpFilter`，我们可以直接继承它来简化类型转换的代码：

```java
public abstract class HttpFilter extends GenericFilter {
    // ... 原码省略部分细节
    
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) throws IOException, ServletException {
        if (req instanceof HttpServletRequest && res instanceof HttpServletResponse) {
            // 内部自动帮我们向下转型为 HttpServletRequest 和 HttpServletResponse
            this.doFilter((HttpServletRequest)req, (HttpServletResponse)res, chain);
        } else {
            throw new ServletException("non-HTTP request or response");
        }
    }

    protected void doFilter(HttpServletRequest req, HttpServletResponse res, FilterChain chain) throws IOException, ServletException {
        chain.doFilter(req, res);
    }
}
```

## 5. 综合实战：登录拦截过滤器

利用 `HttpFilter`，我们可以为应用程序添加一个全局的权限校验过滤器：在未登录情况下，只允许静态资源和登录页面的请求通过；登录之后则畅行无阻。

```java
@WebFilter("/*")
public class MainFilter extends HttpFilter {
    @Override
    protected void doFilter(HttpServletRequest req, HttpServletResponse res, FilterChain chain) throws IOException, ServletException {
        String url = req.getRequestURL().toString();
        
        // 1. 判断是否为静态资源，若是则直接放行，不拦截
        if(!url.endsWith(".js") && !url.endsWith(".css") && !url.endsWith(".png")){
            HttpSession session = req.getSession();
            // 尝试从 Session 中获取用户凭证
            User user = (User) session.getAttribute("user");
            
            // 2. 判断用户是否未登录，且请求的不是登录页面
            if(user == null && !url.endsWith("login")){
                // 未登录则重定向到登录页面
                res.sendRedirect("login");
                return; // 拦截请求，不再向下传递
            }
        }
        // 3. 校验通过或属于放行的资源，交给过滤链继续处理
        chain.doFilter(req, res);
    }
}
```