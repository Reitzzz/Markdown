# 目录

- [目录](#目录)
- [1. 响应JSON化与接口完善](#1-响应json化与接口完善)
  - [1.1 业务接口返回JSON数据](#11-业务接口返回json数据)
  - [1.2 未登录与权限异常处理](#12-未登录与权限异常处理)
  - [1.3 全局统一异常处理配置](#13-全局统一异常处理配置)

---

# 1. 响应JSON化与接口完善

## 1.1 业务接口返回JSON数据

前面我们完成了前后端分离的登录模式，我们来看看一般的业务接口该如何去实现，比如这里我们写一个非常简单的的用户名称获取接口：

```java
@RestController   //为了方便，我们一律使用RestController，这样每个请求默认都返回JSON对象
@RequestMapping("/api/user")   //用户相关的接口，路径可以设置为/api/user/xxxx
public class UserController {

    @GetMapping("/name")
    public RestBean<String> username() {
        User user = (User) SecurityContextHolder.getContext().getAuthentication().getPrincipal();
        return RestBean.success(user.getUsername());
    }
}
```

这样前端就可以在登录之后获取到这个接口的结果了。<br>
注意在index.html里编写时一定要在请求时携带Cookie，否则服务端无法识别身份，会直接被拦截并重定向：

```html
<script>
    axios.get('http://localhost:8081/api/user/name', {
        withCredentials: true  //携带Cookie访问，不然服务器不认识我们
    }).then(({data}) => {
        document.getElementById('username').innerText = data.data
    })
</script>
```

注意一定要登录之后再请求，成功的请求结果如下：

![image-20230724000237828](https://s2.loli.net/2023/07/24/L4PcVKpO2nmHG7e.png)

## 1.2 未登录与权限异常处理

不过我们发现，我们的一些响应还是不完善，比如用户没有登录，默认还是会302重定向，但是实际上我们只需要告诉前端没有登录就行了，所以说我们需要先在SecurityConfiguation里配置异常处理器:

```java
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
                ...
                .exceptionHandling(conf -> {
                    
                    conf.accessDeniedHandler(this::onAccessDeny);//配置授权相关异常处理器
                    
                    conf.authenticationEntryPoint(this::onAuthenticationFailure);//配置验证相关异常的处理器
                })
                .build();
    }
```

再在SecurityConfiguration里修改一下未登录状态下返回的结果：现在有三个方法，但是实际上功能都是一样的，我们可以把它们整合为同一个方法：

```java
private void handleProcess(HttpServletRequest request,
                           HttpServletResponse response,
                           Object exceptionOrAuthentication) throws IOException {

        response.setContentType("application/json;charset=utf-8");

        PrintWriter writer = response.getWriter();

        if(exceptionOrAuthentication instanceof AccessDeniedException exception) {
            writer.write(RestBean.failure(403, exception.getMessage()).asJsonString()); //如果是 AccessDeniedException -> 说明是权限不足，向前端输出 403 的 JSON 失败信息
        } 

        else if(exceptionOrAuthentication instanceof Exception exception) {
            writer.write(RestBean.failure(401, exception.getMessage()).asJsonString());
        } //如果是普通 Exception -> 说明是未登录或认证失败，向前端输出 401 的 JSON 失败信息。
        
        else if(exceptionOrAuthentication instanceof Authentication authentication){
            writer.write(RestBean.success(authentication.getName()).asJsonString());
        } //如果是 Authentication -> 说明是认证成功的凭据对象，向前端输出带有用户名的成功 JSON 信息。
    }
```

这样，用户在没有登录的情况下，请求接口就会返回我们的自定义JSON信息了：

![image-20230724002459523](https://s2.loli.net/2023/07/24/Rf9BSVLvih1lOE2.png)

## 1.3 全局统一异常处理配置

对于我们页面中的一些常见的异常，我们也可以编写异常处理器来将其规范化返回，比如404页面，我们可以直接配置让其抛出异常：

```yml
spring:
  mvc:
    throw-exception-if-no-handler-found: true  # 开启“找不到处理器时抛出异常(NoHandlerFoundException)”
  web:
    resources:
      add-mappings: false  # 关闭默认的静态资源映射功能 使得404可以被抛出
```

然后编写异常处理器：

```java
@RestController
@ControllerAdvice
public class ExceptionController {

    @ExceptionHandler(Exception.class)
    public RestBean<String> error(Exception e){
        if(e instanceof NoHandlerFoundException exception)  //这里就大概处理一下404就行
            return RestBean.failure(404, e.getMessage());  
        else if (e instanceof ServletException exception)  //其他的Servlet异常就返回400状态码
            return RestBean.failure(400, e.getMessage());
        else
            return RestBean.failure(500, e.getMessage());  //其他异常直接返回500
    }
}
```

这样我们的后端就返回的是非常统一的JSON格式数据了，前端开发人员只需要根据我们返回的数据编写统一的处理即可，基于Session的前后端分离实现起来也是最简单的，几乎没有多少的学习成本，跟我们之前的使用是一样的，只是现在前端单独编写了而已。