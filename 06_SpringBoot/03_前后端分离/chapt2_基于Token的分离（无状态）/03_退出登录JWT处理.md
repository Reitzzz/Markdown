# 目录

- [目录](#目录)
- [1. 退出登录JWT处理](#1-退出登录jwt处理)
  - [1.1 JWT无状态带来的退出问题](#11-jwt无状态带来的退出问题)
  - [1.2 黑白名单方案介绍](#12-黑白名单方案介绍)
  - [1.3 JWT签发环节引入UUID](#13-jwt签发环节引入uuid)
  - [1.4 黑名单验证逻辑实现](#14-黑名单验证逻辑实现)
  - [1.5 Security退出登录配置](#15-security退出登录配置)
  - [1.6 机制总结与分布式架构改进](#16-机制总结与分布式架构改进)

---

# 1. 退出登录JWT处理

## 1.1 JWT无状态带来的退出问题

虽然我们使用JWT已经很方便了，但是有一个很严重的问题就是，我们没办法像Session那样去踢用户下线，什么意思呢？我们之前可以使用退出登录接口直接退出，用户Session中的验证信息也会被销毁，但是现在是无状态的，用户来管理Token令牌，服务端只认Token是否合法，那这个时候该怎么让用户正确退出登录呢？

首先我们从最简单的方案开始，我们可以直接让客户端删除自己的JWT令牌，这样不就相当于退出登录了吗，这样甚至不需要请求服务器，直接就退了：

```html
<script>
        ...
  
    function logout() {
        //直接删除存在sessionStorage中的JWT令牌
        sessionStorage.removeItem("access_token")
        //然后回到登录界面
        window.location.href = '/login.html'
    }
</script>
```

这样虽然是最简单粗暴的，但是存在一个问题，**用户可以自行保存这个Token拿来使用。虽然客户端已经删除掉了，但是这个令牌仍然是可用的，如果用户私自保存过，那么依然可以正常使用这个令牌，这显然是有问题的**。也就是说，这种方式只是“客户端退出”，不是“服务端失效”。

## 1.2 黑白名单方案介绍

目前有两种比较好的方案：

* 黑名单方案：所有黑名单中的JWT将不可使用。
* 白名单方案：不在白名单中的JWT将不可使用。**(用户较少时可以使用)**

## 1.3 JWT签发环节引入UUID

这里我们以黑名单机制为例，让用户退出登录之后，无法再次使用之前的JWT进行操作，首先我们需要给JWT额外添加一个用于判断的唯一标识符，这里就用UUID好了：

```java
public class JwtUtils {
    private static final String key = "abcdefghijklmn";

    public static String createJwt(UserDetails user){
        Algorithm algorithm = Algorithm.HMAC256(key);
        Calendar calendar = Calendar.getInstance();
        Date now = calendar.getTime();
        calendar.add(Calendar.SECOND, 3600 * 24 * 7);
        return JWT.create()
                    
                .withJWTId(UUID.randomUUID().toString()) //额外添加一个UUID用于记录黑名单，将其作为JWT的ID属性jti
                .withClaim("name", user.getUsername())
                .withClaim("authorities", user.getAuthorities().stream().map(GrantedAuthority::getAuthority).toList())
                .withExpiresAt(calendar.getTime())
                .withIssuedAt(now)
                .sign(algorithm);
    }
  
        ...
}
```

## 1.4 黑名单验证逻辑实现

这样我们发出去的所有令牌都会携带一个UUID作为唯一凭据，接着我们可以先用一个**内存黑名单集合**来保存已经退出登录的 token：

```java
public class JwtUtils {

  private static final HashSet<String> blackList = new HashSet<>();

  public static boolean invalidate(String token){  //加入黑名单方法
        Algorithm algorithm = Algorithm.HMAC256(key);
        JWTVerifier jwtVerifier = JWT.require(algorithm).build();
        try {
            DecodedJWT verify = jwtVerifier.verify(token);

            return blackList.add(verify.getId()); //取出UUID丢进黑名单中
        } catch (JWTVerificationException e) {
            return false;
        }
  }
  
  ...
  
    public static UserDetails resolveJwt(String token){
        Algorithm algorithm = Algorithm.HMAC256(key);
        JWTVerifier jwtVerifier = JWT.require(algorithm).build();
        try {
            DecodedJWT verify = jwtVerifier.verify(token);
            
            if(blackList.contains(verify.getId()))  //判断是否存在于黑名单中，如果存在，则返回null表示失效
                return null;
            Map<String, Claim> claims = verify.getClaims();
            if(new Date().after(claims.get("exp").asDate()))
                return null;
            return User
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

## 1.5 Security退出登录配置

接着我们来 `SecurityConfiguration` 中配置一下退出登录操作。<br>
注意，**只写 `onLogoutSuccess` 还不够，必须在 `filterChain` 里把 logout 真正接上**：

```java
.logout(conf -> conf
        .logoutUrl("/api/auth/logout")
        .logoutSuccessHandler(this::onLogoutSuccess)
        .permitAll())
```

这样请求 `/api/auth/logout` 时，Spring Security 才会进入退出登录流程，并调用下面这个方法：

```java
private void onLogoutSuccess(HttpServletRequest request,
                                 HttpServletResponse response,
                                 Authentication authentication) throws IOException {
        response.setContentType("application/json;charset=utf-8");
        PrintWriter writer = response.getWriter();
        String authorization = request.getHeader("Authorization");
        if(authorization != null && authorization.startsWith("Bearer ")) {
            String token = authorization.substring(7);
            
            if(JwtUtils.invalidate(token)) {  //将Token加入黑名单
                
                writer.write(RestBean.success("退出登录成功").asJsonString());  //只有成功加入黑名单才会退出成功
                return;
            }
        }
        writer.write(RestBean.failure(400, "退出登录失败").asJsonString());
}
```

## 1.6 机制总结与分布式架构改进

这样，我们就成功安排上了黑名单机制，即使用户提前保存，这个Token依然会因为 `jti` 被加入黑名单而失效：

![image-20230724214624046](https://s2.loli.net/2023/07/24/4o76q5yNHkabuip.png)

*当前实现使用的是应用内存中的 `HashSet`，优点是简单直观，便于理解；缺点是应用重启后黑名单会丢失，而且多个服务实例之间无法共享。后续如果进入微服务场景，可以把黑名单迁移到 Redis 中，并根据 JWT 的过期时间设置对应的失效时间，方便自动清理。*
