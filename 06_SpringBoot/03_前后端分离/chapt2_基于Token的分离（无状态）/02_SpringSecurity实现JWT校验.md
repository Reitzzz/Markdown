# 目录

- [目录](#目录)
- [1. SpringSecurity实现JWT校验](#1-springsecurity实现jwt校验)
  - [1.1 JWT校验机制与Authorization头](#11-jwt校验机制与authorization头)
  - [1.2 编写JWT工具类](#12-编写jwt工具类)
  - [1.3 实现JWT校验过滤器](#13-实现jwt校验过滤器)
  - [1.4 配置Security安全策略](#14-配置security安全策略)
  - [1.5 接口测试与前端集成](#15-接口测试与前端集成)

---

# 1. SpringSecurity实现JWT校验

## 1.1 JWT校验机制与Authorization头

前面我们介绍了JWT的基本原理以及后端的基本校验流程，那么我们现在就来看看如何实现这样的流程。

SpringSecurity中并没有为我们提供预设的JWT校验模块（只有OAuth2模块才有，但是知识太超前了）这里我们只能手动进行整合，JWT可以存放在Cookie或是请求头中，不过不管哪种方式，我们都可以通过Request获取到对应的JWT令牌，**这里我们使用比较常见的请求头携带JWT的方案，客户端发起的请求中会携带这样的的特殊请求头**：

```text
Authorization: Bearer eyJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJzZWxmIiwic3ViIjoidXNlciIsImV4cCI6MTY5MDIxODE2NCwiaWF0IjoxNjkwMTgyMTY0LCJzY29wZSI6ImFwcCJ9.Z5-WMeulZyx60WeNxrQg2z2GiVquEHrsBl9V4dixbRkAD6rFp-6gCrcAXWkebs0i-we4xTQ7TZW0ltuhGYZ1GmEaj4F6BP9VN8fLq2aT7GhCJDgjikaTs-w5BbbOD2PN_vTAK_KeVGvYhWU4_l81cvilJWVXAhzMtwgPsz1Dkd04cWTCpI7ZZi-RQaBGYlullXtUrehYcjprla8N-bSpmeb3CBVM3kpAdehzfRpAGWXotN27PIKyAbtiJ0rqdvRmvlSztNY0_1IoO4TprMTUr-wjilGbJ5QTQaYUKRHcK3OJrProz9m8ztClSq0GRvFIB7HuMlYWNYwf7lkKpGvKDg
```

**这里的Authorization请求头就是携带JWT的专用属性，值的格式为"Bearer Token"，前面的Bearer代表身份验证方式**，默认情况下有两种：

> Basic 和 Bearer 是两种不同的身份验证方式。
>
> Basic 是一种基本的身份验证方式，它将用户名和密码进行base64编码后，放在 Authorization 请求头中，用于向服务器验证用户身份。这种方式不够安全，因为它将密码以明文的形式传输，容易受到中间人攻击。
>
> Bearer 是一种更安全的身份验证方式，它基于令牌（Token）来验证用户身份。Bearer 令牌是由身份验证服务器颁发给客户端的，客户端在每个请求中将令牌放在 Authorization 请求头的 Bearer 字段中。服务器会验证令牌的有效性和权限，以确定用户的身份。Bearer 令牌通常使用 JSON Web Token (JWT) 的形式进行传递和验证。

一会我们会自行编写JWT校验拦截器来处理这些信息。

## 1.2 编写JWT工具类

首先需要导入JWT依赖
```xml
<dependency>
     <groupId>com.auth0</groupId>
     <artifactId>java-jwt</artifactId>
     <version>4.3.0</version>
</dependency>
```

再把用于处理JWT令牌的工具类完成一下：

```java
public class JwtUtils {
    
    private static final String key = "abcdefghijklmn"; //Jwt秘钥

    
    public static String createJwt(UserDetails user){  //根据用户信息创建Jwt令牌
        Algorithm algorithm = Algorithm.HMAC256(key);

        Calendar calendar = Calendar.getInstance();
        Date now = calendar.getTime();
        calendar.add(Calendar.SECOND, 3600 * 24 * 7); //首先获取当前时间 再根据当前时间+7天

        return JWT.create()
                .withClaim("name", user.getUsername())  
                .withClaim("authorities", user.getAuthorities().stream().map(GrantedAuthority::getAuthority).toList())   //配置JWT自定义信息

                .withExpiresAt(calendar.getTime())  //设置过期时间
                .withIssuedAt(now)    //设置创建创建时间
                .sign(algorithm);   //最终签名
    }

    
    public static UserDetails resolveJwt(String token){  //根据Jwt验证并解析用户信息
        Algorithm algorithm = Algorithm.HMAC256(key);
        JWTVerifier jwtVerifier = JWT.require(algorithm).build();
        try {
            DecodedJWT verify = jwtVerifier.verify(token);  //对JWT令牌进行验证，看看是否被修改
            Map<String, Claim> claims = verify.getClaims();  //获取令牌中内容
            if(new Date().after(claims.get("exp").asDate())) //如果是过期令牌则返回null
                return null;
            else
                
                return User  //重新组装为UserDetails对象，包括用户名、授权信息等
                        .withUsername(claims.get("name").asString())
                        .password("")
                        .authorities(claims.get("authorities").asArray(String.class))
                        .build();
        } catch (JWTVerificationException e) {
            return null;
        }
    }
}
```

## 1.3 实现JWT校验过滤器

接着我们需要自行实现一个JwtAuthenticationFilter加入到SpringSecurity默认提供的过滤器链中，用于处理请求头中携带的JWT令牌，并配置登录状态：

```java
public class JwtAuthenticationFilter extends OncePerRequestFilter {  
//继承OncePerRequestFilter表示每次请求过滤一次，用于快速编写JWT校验规则

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        
        String authorization = request.getHeader("Authorization"); //首先从Header中取出JWT
                                                                   //固定步骤 照抄即可
        
        if (authorization != null && authorization.startsWith("Bearer ")) {  //判断是否包含JWT且格式为Bearer开头

            String token = authorization.substring(7);  //截取七位以后的token

            UserDetails user = JwtUtils.resolveJwt(token);  //开始解析成UserDetails对象

            if(user != null) {  //如果得到的是null说明解析失败，JWT有问题
                
                UsernamePasswordAuthenticationToken authentication =
                        new UsernamePasswordAuthenticationToken(user, //user：认证主体，也就是当前登录用户(即通过token解析出来的user)

                        null, //凭证信息。用户名密码登录时这里通常放密码，但 JWT 场景下已经不靠密码校验了，所以一般直接传 null

                        user.getAuthorities() 
                        //对应的权限模型。如果你的项目是菜单权限、资源权限，或者权限字符串格式不同，就要改 JWT 里存的内容和解析后的 GrantedAuthority 组装方式
                        );

                authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));

                //验证没有问题，那么就可以开始创建Authentication了，这里我们跟默认情况保持一致
                //使用UsernamePasswordAuthenticationToken作为实体，填写相关用户信息进去
                
                SecurityContextHolder.getContext().setAuthentication(authentication); 
                //然后直接把配置好的Authentication塞给SecurityContext表示已经完成验证
            }
        }
        
        filterChain.doFilter(request, response);

        //最后放行，继续下一个过滤器
        //可能各位小伙伴会好奇，要是没验证成功不是应该拦截吗？这个其实没有关系的
        //因为如果没有验证失败上面是不会给SecurityContext设置Authentication的，后面直接就被拦截掉了
        //而且有可能用户发起的是用户名密码登录请求，这种情况也要放行的，不然怎么登录，所以说直接放行就好
    }
}
```

## 1.4 配置Security安全策略

最后我们来配置一下SecurityConfiguration配置类，其实配置方法跟之前还是差不多，用户依然可以使用表单进行登录，并且登录方式也是一样的，就是有两个新增的部分需要我们注意一下：<br>
RestBean具体代码:[实现登录授权和跨域处理](../chapt1_基于Session的分离（有状态）/02_实现登录授权和跨域处理.md#1.2-实现登录授权和跨域处理)

```java
@Configuration
public class SecurityConfiguration {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
                    
                ...  
                
                .sessionManagement(conf -> {
                    conf.sessionCreationPolicy(SessionCreationPolicy.STATELESS);
                }) //将Session管理创建策略改成无状态，这样SpringSecurity就不会创建会话了，也不会采用之前那套机制记录用户，因为现在我们可以直接从JWT中获取信息
                    
                .addFilterBefore(new JwtAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class) 
                //添加我们用于处理JWT的过滤器到Security过滤器链中，注意要放在UsernamePasswordAuthenticationFilter之前

                .build();  
    }

    
    private void handleProcess(HttpServletRequest request,
                               HttpServletResponse response,
                               Object exceptionOrAuthentication) throws IOException {  //这个跟之前一样的写法，整合到一起处理，统一返回JSON格式
        response.setContentType("application/json;charset=utf-8");
        PrintWriter writer = response.getWriter();
        if(exceptionOrAuthentication instanceof AccessDeniedException exception) {

            writer.write(RestBean.failure(403, exception.getMessage()).asJsonString());   
            //AccessDeniedException 属于授权异常。当用户已经登录成功，但是尝试访问其角色或权限不足的接口时，Spring Security 会抛出此异常。
            // 内部代码将其捕获，并封装为一个状态码为 403 的自定义JSON对象返回

        } else if(exceptionOrAuthentication instanceof AuthenticationException exception) {
            writer.write(RestBean.failure(401, exception.getMessage()).asJsonString());
            //AuthenticationException 属于认证异常的顶层父类。当用户尚未登录试图访问受保护资源，或者在登录过程中密码错误、账号锁定等情况下会抛出。
            // 内部代码将其封装为状态码为 401 的自定义JSON对象返回，提示前端该用户未认证。

        } else if(exceptionOrAuthentication instanceof Authentication authentication){
            
            writer.write(RestBean.success(JwtUtils.createJwt((User) authentication.getPrincipal())).asJsonString());
            //这里需要注意，在登录成功的时候需要返回我们生成的JWT令牌，这样客户端下次访问就可以携带这个令牌了，令牌过期之后就需要重新登录才可以
        }
    }
}
```

## 1.5 接口测试与前端集成

最后我们创建一个测试使用的Controller来看看效果：

```java
@RestController
public class TestController {

    @GetMapping("/test")
    public String test(){
        return "HelloWorld";
    }
}
```

那么现在采用JWT之后，我们要怎么使用呢？首先我们还是使用工具来测试一下：

![image-20230724200235358](https://s2.loli.net/2023/07/24/L1O8m6auYc2IFWR.png)

登录成功之后，可以看到现在返回给我们了一个JWT令牌，接着我们就可以使用这个令牌了。比如现在我们要访问某个接口获取数据，那么就可以携带这个令牌进行访问：

![image-20230724200341917](https://s2.loli.net/2023/07/24/Hn7X5qeDf9htk6P.png)

注意需要在请求头中添加：

```text
Authorization: Bearer 刚刚获取的Token
```

如果以后没有登录或者携带一个错误的JWT访问服务器，都会返回401错误：

![image-20230724200533964](https://s2.loli.net/2023/07/24/ID96yY7lkr5VsPS.png)

我们现在来模拟一下前端操作：

```html
<script>
    //其他都是跟之前一样的
    function getInfo() {
        axios.post('http://localhost:8081/api/auth/login', {
            username: document.getElementById('username').value,
            password: document.getElementById('password').value
        }, {
            headers: {
                'Content-Type': 'application/x-www-form-urlencoded'
            }
        }).then(({data}) => {
            if(data.code === 200) {
                //将得到的JWT令牌存到sessionStorage用于本次会话
                sessionStorage.setItem("access_token", data.data)
                window.location.href = '/index.html'
            } else {
                alert('登录失败：'+data.message)
            }
        })
    }
</script>
```

接着是首页，获取信息的时候携带上JWT即可，不需要依赖Cookie了：

```html
<script>
    axios.get('http://localhost:8081/api/user/name', {
        headers: {
            'Authorization': "Bearer "+sessionStorage.getItem("access_token")
        }
    }).then(({data}) => {
        document.getElementById('username').innerText = data.data
    })
</script>
```

这样我们就实现了基于SpringSecurity的JWT校验，整个流程还是非常清晰的。