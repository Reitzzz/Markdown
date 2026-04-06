# 2.4 @CookieValue 和 @SessionAttribute

## 目录
- 1.1 [Cookie 的使用](#11-cookie-的使用顺序)
- 1.2 [Session 的使用](#12-session-的使用顺序)
- 1.3 [Cookie 与 Session 的对比总结](#13-cookie-与-session-对比总结)

---

## 1.1 Cookie 的使用

在 Web 开发中，状态管理是核心环节。Spring MVC 提供了便捷的注解来处理客户端（Cookie）的数据持久化。

### 1.1.1 创建与设置
由于 Cookie 是保存在客户端的，我们需要通过 `HttpServletResponse` 对象将其写入：

```java
@RequestMapping("/setCookie")
public void setCookie(HttpServletResponse response) {
    Cookie cookie = new Cookie("theme", "dark");
    cookie.setMaxAge(7 * 24 * 60 * 60); // 设置有效期为7天
    cookie.setPath("/"); // 设置有效路径
    cookie.setHttpOnly(true); // 防止 XSS 攻击获取 Cookie
    response.addCookie(cookie);
}
```

### 1.1.2 自动携带
当 Cookie 通过 `response.addCookie` 设置成功后，浏览器会在后续符合 `path` 规则的请求中，自动将该 Cookie 放入 Request Header 中携带发送给服务器。

### 1.1.3 读取与判断
`@CookieValue` 用于将请求中的某个 Cookie 值绑定到控制器的方法参数上。

**1. 基本参数解析**
*   **value / name**: 指定 Cookie 的名称。
*   **required**: 是否必须包含此 Cookie，默认为 true。如果请求中没有对应的 Cookie 且未设置默认值，会抛出异常。
*   **defaultValue**: 设置默认值。如果请求中没有对应的 Cookie，则使用该值。

**2. 进阶用法示例**
除了获取字符串，你还可以直接获取 Cookie 对象并进行逻辑判断：

```java
@RequestMapping("/getCookie")
public String getCookieDetail(@CookieValue(value = "user_id", defaultValue = "guest") String userId,
                             @CookieValue(value = "JSESSIONID") Cookie cookie) {
    // 判断过程
    System.out.println("用户ID: " + userId);
    System.out.println("Session实例名: " + cookie.getName());
    return "success";
}
```

---

## 1.2 Session 的使用

Spring MVC 提供了 `@SessionAttributes` 和 `@SessionAttribute` 来处理服务端（Session）的数据管理。

### 1.2.1 创建与存数据
如果你希望将模型数据 (Model) 自动同步到 Session 中，可以使用类级别的 `@SessionAttributes`。

*   **作用范围**：该注解声明在类上，指定哪些模型属性需要临时存储在 Session 中。
*   **清理时机**：通常配合 `SessionStatus` 对象手动清理。

```java
@Controller
@SessionAttributes("cart") // 将 Model 中名为 cart 的属性同步到 Session
public class CartController {

    @ModelAttribute("cart")
    public Cart initCart() {
        return new Cart(); // 如果 Session 中没有，则创建一个
    }

    @RequestMapping("/add")
    public String addToCart(@ModelAttribute("cart") Cart cart) {
        cart.addItem("Java核心技术"); // 存数据
        return "cart_view";
    }

    @RequestMapping("/complete")
    public String finishOrder(SessionStatus status) {
        status.setComplete(); // 清除 @SessionAttributes 标记的 Session 属性
        return "order_success";
    }
}
```

### 1.2.2 返回 sessionId
当 Session 创建后，服务器会自动生成一个唯一的 `JSESSIONID`，并通过响应头（Set-Cookie）返回给浏览器保存。

### 1.2.3 根据 sessionId 读取与判断
`@SessionAttribute` 主要用于 **读取** 已经存在于 Session 中的属性。

**注意事项**
*   该注解不会自动创建 Session。如果尝试获取一个不存在的属性且 `required=true`，则会报错。
*   适用于获取登录用户信息、权限令牌等长期驻留数据。

```java
@RequestMapping("/profile")
public ModelAndView userProfile(@SessionAttribute(value = "user", required = false) User user) {
    // 判断过程
    if (user == null) {
        return new ModelAndView("redirect:/login");
    }
    return new ModelAndView("profile", "user", user);
}
```

---

## 1.3 Cookie 与 Session 的对比总结

| 特性 | Cookie | Session |
| :--- | :--- | :--- |
| **存储位置** | 客户端浏览器 | 服务器内存/数据库 |
| **安全性** | 较低（易被伪造或窃取） | 较高 |
| **数据类型** | 仅支持 String | 支持任何 Java 对象 |
| **生命周期** | 可通过 `setMaxAge` 长期保存 | 随会话结束或超时失效 |
| **服务器负担** | 无 | 数据过多会消耗服务器内存 |

> **开发建议**：
> 敏感数据（如用户密码、余额）绝对不要放在 Cookie 中。
> Spring MVC 的注解极大地简化了代码量，但在复杂场景下（如分布式 Session 共享），通常会配合 Spring Session 框架一起使用。