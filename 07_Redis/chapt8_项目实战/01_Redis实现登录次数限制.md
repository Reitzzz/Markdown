# 实战：登录次数限制

在实际的Web应用中，为了防止恶意用户通过字典暴力破解密码，通常会限制用户的连续登录失败次数。当失败次数达到设定阈值时，会对该账号进行临时锁定。利用Redis的高性能读写、自增（`increment`）和过期时间（`expire`）特性，我们可以非常优雅地实现这一防刷机制。

## 核心实现思路

1. **缓存Key设计**：
   - **计数Key** (`login:fail:用户名`)：用于记录用户在指定时间窗口内的登录失败次数。
   - **锁定Key** (`login:lock:用户名`)：当失败次数达到上限时生成，用于标识该账号已被临时锁定。

2. **业务流程控制**：
   - **第一步：前置拦截**。每次请求登录时，优先检查Redis中是否存在该用户的“锁定Key”。如果存在，直接拒绝并提示剩余锁定时间。
   - **第二步：账号密码校验**。查询数据库比对密码。
   - **第三步：失败处理（核心）**。如果密码错误，利用 `opsForValue().increment()` 使错误计数器加一。如果是首次出错（计数值为1），则为该计数Key设置一个有效期（例如15分钟的统计窗口  **15分钟以内连续登录多次失败则暂停登录**）。
   - **第四步：触发锁定**。如果计数值达到最大允许失败次数（如5次），则向Redis写入“锁定Key”（设置锁定时间），并清除对应的“计数Key”。
   - **第五步：成功放行**。如果密码正确，且账号未被锁定，则登录成功，并主动清除Redis中的“计数Key”和“锁定Key”，颁发Token。

## 常量定义

首先定义好重试次数、锁定时间和统计窗口的常量，方便后续统一维护：

```java
private static final int MAX_FAIL = 5; // 最大允许失败次数
private static final long LOCK_MINUTES = 15; // 账号锁定时间（分钟）
private static final long FAIL_WINDOW_MINUTES = 15; // 失败统计的时间窗口（分钟）
```

## 完整代码实现

```java
@Autowired
private org.springframework.data.redis.core.StringRedisTemplate stringRedisTemplate;
//引入Redis的操作工具

@PostMapping("/user/login")
public Result login(@RequestBody @Validated LoginDTO loginData) {
    String username = loginData.getUsername();
    String password = loginData.getPassword();

    String failKey = "login:fail:" + username; //用来记录某个具体用户登录失败的次数
    String lockKey = "login:lock:" + username; //用来标记某个用户是否已经被锁定了 如果存在则该用户被锁定

    // 1) 检查是否已被锁定 (前置拦截)
    Boolean locked = stringRedisTemplate.hasKey(lockKey);
    if (Boolean.TRUE.equals(locked)) {  //防止空指针 也可以写成if (locked != null && locked)

        Long ttl = stringRedisTemplate.getExpire(lockKey, java.util.concurrent.TimeUnit.SECONDS); 
        //查询 Redis 中lockKey还剩多少过期时间  后面这个参数表示以秒为单位返回

        String msg = (ttl != null && ttl > 0)
                ? "账号已锁定，请 " + ttl + " 秒后重试"
                : "账号已锁定，请稍后重试";
        return Result.error(msg);
    }

    // 2) 查数据库比对用户信息
    QueryWrapper<sys_user> wrapper = new QueryWrapper<>();
    wrapper.eq("username", username);
    sys_user user = userMapper.selectOne(wrapper);

    // 3) 用户不存在或密码错误 -> 失败计数处理
    if (user == null || !passwordEncoder.matches(password, user.getPassword())) {
        
        Long failCount = stringRedisTemplate.opsForValue().increment(failKey);// 给failKey对应的计数器FailCount加1

        
        if (failCount != null && failCount == 1L) {   // 首次失败时，设置统计窗口的过期时间
                                                      // 1L表示告诉编译器这是个Long变量
            stringRedisTemplate.expire(failKey, FAIL_WINDOW_MINUTES, java.util.concurrent.TimeUnit.MINUTES);
            
        }

        // 达到阈值，触发锁定
        if (failCount != null && failCount >= MAX_FAIL) {
            //写一个锁定 key，值设为 1(只是为了lockKey有值 具体是什么不重要) ，并在 LOCK_MINUTES (15)分钟后自动删除 解除锁定
            stringRedisTemplate.opsForValue().set(lockKey, "1", LOCK_MINUTES, java.util.concurrent.TimeUnit.MINUTES);
            // 清除之前的失败计数
            stringRedisTemplate.delete(failKey);
            return Result.error("账号已锁定" + LOCK_MINUTES + "分钟，请稍后重试");
        }

        long remain = MAX_FAIL - (failCount == null ? 0 : failCount);
        return Result.error("用户名或密码不正确，剩余尝试次数：" + remain);
    }

    // 4) 登录成功 -> 清理缓存状态
    stringRedisTemplate.delete(failKey);
    stringRedisTemplate.delete(lockKey);

    // 5) 颁发Token并返回
    String token = JwtUtil.createToken(user.getUsername(), user.getRole());
    Map<String, String> data = new HashMap<>();
    data.put("token", token);
    return Result.success(data);
}
```