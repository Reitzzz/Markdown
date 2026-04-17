# 目录

- [目录](#目录)
- [1. 自动续签JWT令牌与方案对比](#1-自动续签jwt令牌与方案对比)
    - [1.1 自动续签JWT的需求与实现](#11-自动续签jwt的需求与实现)
  - [1.2 令牌刷新频率控制建议](#12-令牌刷新频率控制建议)
  - [1.3 JWT与传统Session校验方案对比](#13-jwt与传统session校验方案对比)

---

# 1. 自动续签JWT令牌与方案对比

### 1.1 自动续签JWT的需求与实现

在有些时候，我们可能希望用户能够一直使用我们的网站，而不是JWT令牌到期之后就需要重新登录。这种情况下前端就可以配置JWT自动续签：在发起请求时如果发现令牌即将到期，就向后端发起续签请求，从而得到一个新的JWT令牌。

在实现续签接口时，除了生成新令牌，**更重要的是做好安全校验与防御性编程**，防止伪造凭证或系统异常导致服务崩溃。

以下是一个健壮的令牌刷新接口实现：

```java
@RestController
@RequestMapping("/api/auth")
public class AuthorizeController {

    @GetMapping("/refresh")
    public Result<Map<String, String>> refreshToken() {
        // 1. 获取当前安全上下文中的认证信息
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();

        // 2. 防御性拦截：检查认证对象是否存在且已合法认证
        if (authentication == null || !authentication.isAuthenticated()) {
            return Result.error("未登录或token无效");
        }

        // 3. 安全类型转换：使用 instanceof 模式匹配，彻底杜绝 ClassCastException
        Object principal = authentication.getPrincipal();  
        // 注意：这里取什么类型，严格取决于你的 JwtAuthenticationFilter 中存入了什么类型
        if (!(principal instanceof String username) || !StringUtils.hasText(username)) {
            // (1) 如果 principal 的实际类型确实是 String，它就会自动将其强转，并赋值给后面声明的新变量 username
            // (2) 如果取出来的用户名是空的、null、或者全是空格，那么说明数据不合法，同样进入 if 报错
            return Result.error("认证信息异常");
        }

        // 4. 验证用户最新状态：回查数据库，确保用户未被注销或角色未被修改
        QueryWrapper<sys_user> wrapper = new QueryWrapper<>();
        wrapper.eq("username", username);                //SELECT * FROM sys_user WHERE username = ? 
        sys_user user = userMapper.selectOne(wrapper);

        if (user == null || !StringUtils.hasText(user.getRole())) {
            return Result.error("用户不存在或角色异常");
        }

        // 5. 签发新令牌并保持与登录接口一致的返回结构
        String jwt = JwtUtil.createToken(user.getUsername(), user.getRole());
        Map<String, String> data = new HashMap<>();
        data.put("token", jwt); // 与 /login 返回结构保持一致，方便前端统一处理
        
        return Result.success(data);
    }
}
```

> **核心设计考量（避坑指南）：**
> * **严谨的类型推断**：切忌直接对 `getPrincipal()` 进行对象强转。如果不清楚前置过滤器塞入了什么，极易在运行期引发 `ClassCastException` 导致接口 500 崩溃。
> * **数据实时性验证**：仅凭旧 Token 里的负载信息直接签发新 Token 是不安全的。必须拿解析出的唯一标识（如 `username`）去数据库查最新状态，这是保证权限动态控制的关键。
> * **接口结构一致性**：将返回值封装为 `{"token": "xxx"}`，与登录接口对接标准统一，是非常好的接口设计习惯。

这样，前端在发现令牌可用时间不足时，就会先发起一个请求自动完成续期，得到一个新的Token：

![image-20230724232152613](https://s2.loli.net/2023/07/24/cqEgnQOZtFp1w7o.png)

## 1.2 令牌刷新频率控制建议

我们可能还需要**配置一下这种方案的请求频率，不然用户疯狂请求刷新Token就不太好了，我们同样可以借助Redis进行限流等操作**，防止频繁请求，这里就不详细编写了，各位小伙伴可以自行实现。

## 1.3 JWT与传统Session校验方案对比

我们最后可以来对比一下两种前后端分离方式的优缺点如何：

**JWT校验方案的优点：** 1. 无状态: JWT是无状态的，服务器不需要在后端维护用户的会话信息，可以在分布式系统中进行水平扩展，减轻服务器的负担。
2. 基于Token: JWT使用token作为身份认证信息，该token可以存储用户相关的信息和权限。这样可以减少与数据库的频繁交互，提高性能。
3. 安全性: JWT使用数字签名或加密算法保证token的完整性和安全性。每次请求都会验证token的合法性，防止伪造或篡改。
4. 跨域支持: JWT可以在不同域之间进行数据传输，适合前后端分离的架构。

**JWT校验方案的缺点：** 1. 无法做到即时失效: JWT中的token通常具有较长的有效期，一旦签发，就无法立即失效。如果需要即时失效，需要在服务端进行额外的处理。
2. 信息无法撤销: JWT中的token一旦签发，除非到期或者客户端清除，无法撤销。无法中途取消和修改权限。
3. Token增大的问题: JWT中包含了用户信息和权限等，token的体积较大，每次请求都需要携带，增加了网络传输的开销。
4. 动态权限管理问题: JWT无法处理动态权限管理，一旦签发的token权限发生变化，仍然有效，需要其他手段进行处理。

**传统Session校验方案的优点：** 1. 即时失效: Session在服务器端管理，可以通过设置过期时间或手动删除实现即时失效，保护会话的安全性。
2. 信息即时撤销: 服务器端可以随时撤销或修改Session的信息和权限。
3. 灵活的权限管理: Session方案可以更灵活地处理动态权限管理，可以根据具体场景进行即时调整。

**传统Session校验方案的缺点：** 1. 状态维护: 传统Session需要在服务器端维护会话状态信息，增加了服务器的负担，不利于系统的横向扩展。
2. 性能开销: 每次请求都需要在服务器端进行会话状态的校验和读写操作，增加了性能开销。
3. 跨域问题: Session方案在跨域时存在一些问题，需要进行额外的处理。
4. 无法分布式共享: 传统Session方案不适用于多个服务器之间共享会话信息的场景，需要额外的管理和同步机制。

综上所述，<u>**JWT校验方案适用于无状态、分布式系统，几乎所有常见的前后端分离的架构都可以采用这种方案**</u>。而传统Session校验方案适用于需要即时失效、即时撤销和灵活权限管理的场景，适合传统的服务器端渲染应用，以及客户端支持Cookie功能的前后端分离架构。在选择校验方案时，需要根据具体的业务需求和技术场景进行选择。