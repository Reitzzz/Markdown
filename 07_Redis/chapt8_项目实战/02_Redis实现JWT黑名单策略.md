# 1. 实战：Redis 实现 JWT 退出登录黑名单

## 目录
- [1. 实战：Redis 实现 JWT 退出登录黑名单](#1-实战redis-实现-jwt-退出登录黑名单)
  - [目录](#目录)
  - [1.1 业务痛点与方案概述](#11-业务痛点与方案概述)
  - [1.2 依赖与配置](#12-依赖与配置)
  - [1.3 核心代码实现](#13-核心代码实现)
    - [1.3.1 改造 JwtUtil（新增 jti 与过期时间支持）](#131-改造-jwtutil新增-jti-与过期时间支持)
    - [1.3.2 编写 Token 黑名单服务](#132-编写-token-黑名单服务)
    - [1.3.3 改造 JwtAuthenticationFilter（增加黑名单校验）](#133-改造-jwtauthenticationfilter增加黑名单校验)
    - [1.3.4 配置 SecurityConfig 注册过滤器](#134-配置-securityconfig-注册过滤器)
    - [1.3.5 改造 UserController 的 Logout 接口](#135-改造-usercontroller-的-logout-接口)
  - [1.4 测试流程](#14-测试流程)

---

## 1.1 业务痛点与方案概述

在使用 SpringBoot + JWT 构建前后端分离项目时，JWT 存在一个致命缺陷：**Token 是无状态的，一旦签发，在过期之前即使前端清除了本地存储，该 Token 被黑客截获后依然可以访问接口。**

为了实现真正的“退出登录（即时失效）”，我们需要引入 Redis 作为黑名单中心。

**核心流程：**
1. **签发阶段**：生成 Token 时，利用 `UUID` 塞入唯一标识 `jti` (JWT ID)，并确保有 `exp` (过期时间)。
2. **退出阶段**：用户调用 `/logout` 时，后端解析当前 Token，将其 `jti` 存入 Redis，并将 Redis 的过期时间（TTL）设置为 Token 的剩余存活时间。
3. **校验阶段**：每次请求经过 SpringSecurity 的 `JwtAuthenticationFilter` 时，先去 Redis 查一下该 Token 的 `jti` 是否在黑名单内。

---

## 1.2 依赖与配置

确保项目中已整合 Redis：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

---

## 1.3 核心代码实现

### 1.3.1 改造 JwtUtil（新增 jti 与过期时间支持）

相比基础版，增加了对 `jti` 的注入以及解析方法：

```java
public class JwtUtil {
    // ... SECRET 与 EXPIRE_TIME 配置 ...

    public static String createToken(String username, String role) {
        Date now = new Date();
        Date exp_time = new Date(now.getTime() + EXPIRE_TIME);
        String jid = UUID.randomUUID().toString(); // 【新增】生成唯一ID
        return JWT.create()
                .withSubject(username)
                .withJWTId(jid)  // 【新增】设置 JWT ID
                .withClaim("role", role)
                .withIssuedAt(now)
                .withExpiresAt(exp_time)
                .sign(Algorithm.HMAC256(SECRET));
    }

    // 【新增】解析 ID
    public static String getJti(String token){
        return JWT.decode(token).getId();
    }

    // 【新增】解析过期时间
    public static Date getExpireTime(String token){
        return JWT.decode(token).getExpiresAt();
    }
    
    // ... 其他 verify, getUsername 方法 ...
}
```

### 1.3.2 编写 Token 黑名单服务

实现类利用 `StringRedisTemplate` 进行持久化校验。此处直接使用 `JWT.decode(token)` 提取载荷信息，提高处理效率。

```java
@Service
public class TokenBlacklistServiceImpl implements TokenBlacklistService {
    private static final String PREFIX = "jwt:blacklist:";
    private final StringRedisTemplate redisTemplate;

    public TokenBlacklistServiceImpl(StringRedisTemplate redisTemplate) {
        this.redisTemplate = redisTemplate;
    }

    @Override
    public void blacklist(String token) {
        if (token == null) return;

        String tokenID = JWT.decode(token).getId();
        Date exp_time = JWT.decode(token).getExpiresAt();

        if (exp_time == null || tokenID == null) return;

        long remain_time = exp_time.getTime() - System.currentTimeMillis();
        if (remain_time <= 0) return;

        // 将 jti 存入 Redis，TTL 设为 Token 剩余存活时间
        redisTemplate.opsForValue().set(PREFIX + tokenID, "1", remain_time, TimeUnit.MILLISECONDS);
    }

    @Override
    public boolean isBlacklisted(String token) {
        if (token == null) return false;
        String tokenID = JWT.decode(token).getId();
        return tokenID != null && Boolean.TRUE.equals(redisTemplate.hasKey(PREFIX + tokenID));
    }
}
```

### 1.3.3 改造 JwtAuthenticationFilter（增加黑名单校验）

在执行 `JwtUtil.verify` 之前，必须先进行黑名单拦截：

```java
@Override
protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
    String token = resolveToken(request);

    // 【新增】黑名单预检
    if (StringUtils.hasText(token) && tokenBlacklistService.isBlacklisted(token)) {
        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        response.setCharacterEncoding("UTF-8");
        response.setContentType("application/json;charset=UTF-8");

        String json = """
        {
          "code": 60204,
          "message": "Token已被禁用，请重新登录",
          "data": null
        }
        """;

        response.getWriter().write(json);
        response.getWriter().flush();
        return;
    }

    // 原有的 Token 校验与 SecurityContext 注入逻辑...
    if (StringUtils.hasText(token) && JwtUtil.verify(token)) {
        // ...
    }

    filterChain.doFilter(request, response);
}

// resolveToken 方法逻辑...
```

### 1.3.4 配置 SecurityConfig 注册过滤器

黑名单功能的实现依赖于 `JwtAuthenticationFilter` 被正确插入安全过滤链。

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public JwtAuthenticationFilter jwtAuthenticationFilter(TokenBlacklistService tokenBlacklistService) {
        return new JwtAuthenticationFilter(tokenBlacklistService);
    }

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http, JwtAuthenticationFilter jwtAuthenticationFilter) throws Exception {
        http
                .csrf(csrf -> csrf.disable())
                .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
                .authorizeHttpRequests(auth -> {
                    auth.requestMatchers("/vue-admin-template/user/login").permitAll();
                    auth.requestMatchers("/vue-admin-template/user/logout").hasRole("ADMIN");
                    auth.anyRequest().permitAll();
                })
                // 【关键】将自定义过滤器放在 UsernamePasswordAuthenticationFilter 之前
                .addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }
}
```

### 1.3.5 改造 UserController 的 Logout 接口

```java
@PostMapping("/user/logout")
public Result logout(HttpServletRequest request) {
    // 此处需根据前端实际传参方式提取 Token（例如 X-Token 或 Authorization）
    String token = request.getHeader("X-Token"); 
    if (token != null) {
        // 加入黑名单，实现即时失效
        tokenBlacklistService.blacklist(token);
    }
    return Result.success("退出成功");
}
```

---

## 1.4 测试流程

1. **正常访问**：登录获取 Token A，访问业务接口，返回数据。
2. **执行退出**：调用 `/logout` 接口，Redis 中出现 `jwt:blacklist:{jti}` 记录。
3. **失效校验**：再次使用 Token A 访问，Filter 拦截并返回 `60204` 错误码。
4. **自动清理**：待 Token A 的原始过期时间到达后，Redis 中的黑名单记录自动消失。