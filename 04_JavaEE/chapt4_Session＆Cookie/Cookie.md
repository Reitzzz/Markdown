# Cookie 状态管理

## 1. 什么是 Cookie？

Cookie 可以在浏览器中保存部分信息，并且在下次发起请求时，浏览器的请求头中会自动携带这些信息。

我们可以通过简单的测试用例来观察其工作原理：

```java
// 创建一个名为"test"，值为"yyds"的Cookie对象
Cookie cookie = new Cookie("test", "yyds");
// 将Cookie添加到响应头中，返回给浏览器保存
resp.addCookie(cookie);
// 重定向到"time"页面
resp.sendRedirect("time");
```

```java
// 从请求中获取所有的Cookie数组并遍历
for (Cookie cookie : req.getCookies()) {
    // 打印每个Cookie的名称和对应的值
    System.out.println(cookie.getName() + ": " + cookie.getValue());
}
```

在 `HttpServletResponse` 中添加 Cookie 之后，浏览器的响应头中会包含一个 `Set-Cookie` 属性。重定向之后，新的请求头中会携带此 Cookie，同时我们可以直接通过 `HttpServletRequest` 快速获取客户端传来的 Cookie 信息。

## 2. Cookie 的核心属性

一个 Cookie 包含以下详细信息，其中最关键的是 **name**、**value**、**maxAge** 和 **domain**：

* **name**: Cookie 的名称。一旦创建，名称便不可更改。
* **value**: Cookie 的值。如果值为 Unicode 字符，需要进行字符编码；如果为二进制数据，则需要使用 BASE64 编码。
* **maxAge**: Cookie 失效的时间（单位：秒）。
    * 正数：在设定的秒数后失效。
    * 负数：临时 Cookie，关闭浏览器即失效，浏览器不会将其持久化保存（默认值为 -1）。
    * 0：表示立即删除该 Cookie。
* **secure**: 该 Cookie 是否仅通过安全协议（如 HTTPS、SSL）传输，默认为 false。
* **path**: Cookie 的有效路径。例如设置为 `/sessionWeb/`，则只有 contextPath 为此路径的程序才能访问；设置为 `/` 则表示本域名下所有 contextPath 均可访问。注意必须以 `/` 结尾。
* **domain**: 可以访问该 Cookie 的域名。例如设置为 `.google.com`，则所有以该后缀结尾的子域名均可访问。注意必须以 `.` 开头。
* **comment**: 该 Cookie 的用途说明。
* **version**: Cookie 遵循的规范版本号（0 代表 Netscape 规范，1 代表 W3C 的 RFC 2109 规范）。

### 修改失效时间示例

```java
// 设置Cookie的有效时间为20秒
cookie.setMaxAge(20);
```

设定为 20 秒后，响应头会下发 20 秒的过期时间。20 秒内发起访问均会携带此 Cookie，超时后浏览器会自动清除。

## 3. 实战：使用 Cookie 实现“记住我”功能

利用 Cookie 可以实现用户的免密码登录。如果用户在登录时勾选了“记住我”，我们可以将凭证保存在 Cookie 中，下次访问首页时提取并自动校验。

### 步骤一：前端添加勾选框

在登录表单中增加记住我选项：

```html
<div>
    <label>
        <input type="checkbox" placeholder="记住我" name="remember-me">
        记住我
    </label>
</div>
```

### 步骤二：后端处理登录成功逻辑

判断是否勾选了“记住我”，如果勾选则下发 Cookie：

```java
// 判断前端提交的表单数据中是否包含"remember-me"（即是否勾选了记住我）
if(map.containsKey("remember-me")){
    // 将用户名存入Cookie，并设置30秒的有效期
    Cookie cookie_username = new Cookie("username", username);
    cookie_username.setMaxAge(30);
    
    // 将密码存入Cookie，同样设置30秒的有效期
    Cookie cookie_password = new Cookie("password", password);
    cookie_password.setMaxAge(30);
    
    // 将这两个Cookie下发给浏览器保存
    resp.addCookie(cookie_username);
    resp.addCookie(cookie_password);
}
```

### 步骤三：自动登录拦截与校验

修改默认请求地址（如统一切换到 `/login`），在 GET 请求中提取 Cookie 并进行校验：

```java
@Override
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    // 获取客户端携带的所有Cookie
    Cookie[] cookies = req.getCookies();
    if(cookies != null){
        String username = null;
        String password = null;
        
        // 遍历寻找保存的用户名和密码信息
        for (Cookie cookie : cookies) {
            if(cookie.getName().equals("username")) username = cookie.getValue();
            if(cookie.getName().equals("password")) password = cookie.getValue();
        }
        
        // 如果成功获取到了用户名和密码，尝试进行自动登录拦截与校验
        if(username != null && password != null){
            // 开启数据库会话进行校验
            try (SqlSession sqlSession = factory.openSession(true)){
                UserMapper mapper = sqlSession.getMapper(UserMapper.class);
                User user = mapper.getUser(username, password);
                
                // 如果数据库中存在该用户，说明凭证有效，直接重定向到目标页面
                if(user != null){
                    resp.sendRedirect("time");
                    return;   // 校验成功，直接返回，不再执行后面的页面转发
                }
            }
        }
    }
    // 如果没有Cookie或校验失败，正常转发给默认的Servlet展示登录静态页面
    req.getRequestDispatcher("/").forward(req, resp);   
}
```

通过上述配置，30 秒内用户访问登录页面将被自动拦截并重定向到目标页面（如 `time` 页面）。

## 4. 遗留问题与引申

当前实现仍然存在一个问题：目标页面（首页）缺乏权限校验，无论用户是否登录都可以直接通过 URL 访问。为了实现“仅限登录后访问”的安全控制，就需要引入新的状态管理机制。