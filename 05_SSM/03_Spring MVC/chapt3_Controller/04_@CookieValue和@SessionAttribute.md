# 2.4 @CookieValue 和 @SessionAttribute 详解

在 Spring MVC 开发中，Cookie 与 Session 是处理客户端与服务端状态同步的核心手段。本篇笔记将详细解析两者的使用方法、参数含义及其背后的实现逻辑。

---

## 1.1 Cookie 的使用

Cookie 是由服务器发送并存储在客户端浏览器上的小文本文件，常用于识别用户身份或保存用户偏好。

### 1.1.1 创建与设置
由于 Cookie 需要从服务器写入客户端，必须借助 `HttpServletResponse` 对象。

**代码演示：**
```java
@RequestMapping("/setCookie")
public void setCookie(HttpServletResponse response) {
    // 1. 创建 Cookie 对象，参数为 (Key, Value)
    Cookie cookie = new Cookie("theme", "dark");

    // 2. setMaxAge：设置 Cookie 的有效期（单位：秒）
    // 意义：决定 Cookie 是临时存在（会话级）还是持久化存储在磁盘。
    cookie.setMaxAge(7 * 24 * 60 * 60); // 存活 7 天

    // 3. setPath：设置 Cookie 的有效路径
    // 意义：指定哪些 URL 路径下的请求会携带此 Cookie。设置 "/" 表示全站通用。
    cookie.setPath("/");

    // 4. setHttpOnly：安全加固
    // 意义：设置为 true 后，前端 JavaScript 无法通过 document.cookie 获取该值，
    // 能有效防止跨站脚本攻击（XSS）窃取用户信息。
    cookie.setHttpOnly(true);

    // 5. 将 Cookie 添加到响应头
    response.addCookie(cookie);
}
```

### 1.1.2 自动携带
设置成功后，浏览器在下一次发起请求时，会根据 `Path` 规则自动在 HTTP 请求头的 `Cookie` 字段中携带这些数据，服务器随后即可读取。



### 1.1.3 使用 @CookieValue 读取
`@CookieValue` 注解可以将请求中指定的 Cookie 值直接注入到方法参数中。

**参数解释：**
*   **value / name**：要获取的 Cookie 的键名。
*   **required**：布尔值，默认为 `true`。若请求中缺失该 Cookie 且未设默认值，系统会抛出异常。在不确定 Cookie 是否存在时，建议设为 `false`。
*   **defaultValue**：当指定的 Cookie 不存在时的降级方案，避免程序报错。

**代码演示：**
```java
@RequestMapping("/getCookie")
public String getCookieDetail(
    // 绑定字符串类型的 Cookie 值
    @CookieValue(value = "theme", defaultValue = "light") String theme,
    // 也可以直接绑定 Cookie 对象，以便获取更多元数据（如过期时间）
    @CookieValue(value = "JSESSIONID") Cookie cookie) {

    System.out.println("当前主题设置: " + theme);
    System.out.println("Session 标识符名称: " + cookie.getName());
    return "success";
}
```

---

## 1.2 Session 的使用

Session 存储在服务端，通过客户端传递的 `JSESSIONID` 来区分不同的用户，安全性高于 Cookie。

### 1.2.1 使用 @SessionAttributes 同步数据
该注解标注在**类**上，其核心意义在于实现 Model 数据与 Session 存储的自动同步，常用于多步骤表单或购物车场景。

**方法与对象解释：**
*   **@SessionAttributes("key")**：指定 Model 中哪些属性需要放入 Session。
*   **SessionStatus**：用于手动清理通过该注解存入 Session 的属性。

**代码演示：**
```java
@Controller
@SessionAttributes("cart") // 声明：模型中名为 "cart" 的数据需存入 Session
public class CartController {

    // 意义：初始化数据。若 Session 中没有 "cart"，则执行此方法
    @ModelAttribute("cart")
    public List<String> initCart() {
        return new ArrayList<>();
    }

    @RequestMapping("/add")
    public String addToCart(@ModelAttribute("cart") List<String> cart) {
        cart.add("Spring MVC 实战书籍"); // 数据会自动同步回 Session
        return "cart_view";
    }

    @RequestMapping("/complete")
    public String finish(SessionStatus status) {
        // 意义：显式清除 @SessionAttributes 维护的属性，防止内存溢出或数据残留
        status.setComplete();
        return "order_success";
    }
}
```

### 1.2.2 JSESSIONID 的机制
当服务端首次调用 `request.getSession()` 时，会创建一个 Session 对象并生成唯一的 `JSESSIONID`。服务器通过 `Set-Cookie: JSESSIONID=xxx` 响应头告知浏览器。浏览器后续请求会自动带上这个 ID，服务器根据 ID 找到内存中对应的 Session 空间。

### 1.2.3 使用 @SessionAttribute 读取
与上面的类级别注解不同，`@SessionAttribute`（无 s）标注在**方法参数**上，专门用于**读取**已存在的 Session 数据。

**意义：** 简化了从 `HttpSession` 中通过 `getAttribute` 手动转型取值的繁琐过程。

**代码演示：**
```java
@RequestMapping("/user/info")
public String getUserInfo(
    
    @SessionAttribute(value = "loginUser", required = false) User user) { //required=false 
                                                                          //意义：防止用户未登录（Session无此属性）时访问导致 400 错误   
    if (user == null) {
        return "redirect:/login"; // 未登录则重定向
    }
    System.out.println("当前登录用户: " + user.getUsername());
    return "user_detail";
}
```

---

## 1.3 Cookie 与 Session 的对比总结

了解两者的差异有助于在实际开发中做出正确的架构选择。

| 特性 | Cookie | Session |
| :--- | :--- | :--- |
| **存储位置** | 客户端（浏览器） | 服务端（内存/Redis/数据库） |
| **安全性** | 较低，数据可被篡改或截获 | 较高，数据不离开服务器 |
| **数据类型** | 仅支持 **String** | 支持 **任何 Java 对象** |
| **有效期** | 可长达数年（设置 MaxAge） | 默认较短（如30分钟），随会话结束 |
| **服务器负担** | 极低（压力在客户端） | 较高（大量在线用户会占用内存） |

> **开发实践建议：**
> 1. **敏感数据**：如支付状态、用户权限，必须存储在 Session 中。
> 2. **无害数据**：如“记住用户名”、夜间模式切换，可存储在 Cookie 中。
> 3. **分布式环境**：在多台服务器环境下，原生 Session 无法共享，建议配合 Spring Session 或 Redis 实现 Session 集中化管理。