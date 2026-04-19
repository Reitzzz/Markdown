# 1. Token持久化存储

**目录**
- [1. Token持久化存储](#1-token持久化存储)
  - [1.1 手动实现Token持久化](#11-手动实现token持久化)
  - [1.2 实现验证Service](#12-实现验证service)
  - [1.3 编写实体类与Mapper](#13-编写实体类与mapper)
  - [1.4 编写Security配置类](#14-编写security配置类)
  - [1.5 启动验证](#15-启动验证)

---

## 1.1 手动实现Token持久化

我们之前使用SpringSecurity时，remember-me的Token是支持持久化存储的，而我们当时是存储在数据库中，那么Token信息能否存储在缓存中呢，当然也是可以的，我们可以手动实现一个：

```java
//实现PersistentTokenRepository接口
@Component
public class RedisTokenRepository implements PersistentTokenRepository {

    // ================== 0. 准备工作 ==================
    // Key名称前缀，用于区分
    private final static String REMEMBER_ME_KEY = "spring:security:rememberMe:";
    @Resource
    RedisTemplate<Object, Object> template;


    // ================== 1. 底层工具：解决对象无法序列化的问题 ==================
    
    // 存入：把完整的Token对象拆成Hash存进去
    private void setToken(PersistentRememberMeToken token){
        Map<String, String> map = new HashMap<>();
        map.put("username", token.getUsername());
        map.put("series", token.getSeries());
        map.put("tokenValue", token.getTokenValue());
        map.put("date", ""+token.getDate().getTime());
        template.opsForHash().putAll(REMEMBER_ME_KEY+token.getSeries(), map);
        template.expire(REMEMBER_ME_KEY+token.getSeries(), 1, TimeUnit.DAYS);
    }

    // 取出：把Hash拿出来拼回成Token对象
    private PersistentRememberMeToken getToken(String series){
        Map<Object, Object> map = template.opsForHash().entries(REMEMBER_ME_KEY+series);
        if(map.isEmpty()) return null;
        return new PersistentRememberMeToken(
                (String) map.get("username"),
                (String) map.get("series"),
                (String) map.get("tokenValue"),
                new Date(Long.parseLong((String) map.get("date"))));
    }


    // ================== 2. 登录发证：创建新的Token并保存两份映射 ==================
    
    @Override
    public void createNewToken(PersistentRememberMeToken token) {
        // 第一步：存一个 username -> seriesId 的“路标”（为了以后能通过用户名找到对应的凭证删除）
        template.opsForValue().set(REMEMBER_ME_KEY+"username:"+token.getUsername(), token.getSeries());
        template.expire(REMEMBER_ME_KEY+"username:"+token.getUsername(), 1, TimeUnit.DAYS);
        // 第二步：存真正的凭证 seriesId -> Token详细信息 (调用底层存入工具)
        this.setToken(token);
    }


    // ================== 3. 带着Token免密访问：查询与刷新时间 ==================
    
    @Override
    public PersistentRememberMeToken getTokenForSeries(String seriesId) {
        // 直接调用底层取出工具拿数据
        return this.getToken(seriesId);
    }

    @Override
    public void updateToken(String series, String tokenValue, Date lastUsed) {
        // 先获取，然后修改创建一个新的，再放入覆盖
        PersistentRememberMeToken token = this.getToken(series);
        if(token != null)
           this.setToken(new PersistentRememberMeToken(token.getUsername(), series, tokenValue, lastUsed));
    }


    // ================== 4. 注销或安全重置：吊销Token ==================
    
    @Override
    public void removeUserTokens(String username) {
        // 第一步：通过“路标” (username) 查出真正的凭证ID (seriesId)
        String series = (String) template.opsForValue().get(REMEMBER_ME_KEY+"username:"+username);
        // 第二步：根据查出来的ID，销毁真正的凭证
        template.delete(REMEMBER_ME_KEY+series);
        // 第三步：把“路标”也一起删掉
        template.delete(REMEMBER_ME_KEY+"username:"+username);
    }
}
```

## 1.2 实现验证Service

接着把验证Service实现了：

```java
@Service
public class AuthService implements UserDetailsService {

    @Resource
    UserMapper mapper;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        Account account = mapper.getAccountByUsername(username);
        if(account == null) throw new UsernameNotFoundException("");
        return User
                .withUsername(username)
                .password(account.getPassword())
                .roles(account.getRole())
                .build();
    }
}
```

## 1.3 编写实体类与Mapper

Mapper也安排上：

```java
@Data
public class Account implements Serializable {
    int id;
    String username;
    String password;
    String role;
}
```

```java
@CacheNamespace(implementation = MybatisRedisCache.class)
@Mapper
public interface UserMapper {

    @Select("select * from users where username = #{username}")
    Account getAccountByUsername(String username);
}
```

## 1.4 编写Security配置类

最后配置文件配一波：

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http
            .authorizeRequests()
            .anyRequest().authenticated()
            .and()
            .formLogin()
            .and()
            .rememberMe()
            .tokenRepository(repository);
}

@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth
            .userDetailsService(service)
            .passwordEncoder(new BCryptPasswordEncoder());
}
```

## 1.5 启动验证

OK，启动服务器验证一下吧。