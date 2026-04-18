# 目录

- [目录](#目录)
- [1. 响应JSON化与接口完善](#1-响应json化与接口完善)
  - [1.1 业务接口返回JSON数据](#11-业务接口返回json数据)
  - [1.2 未登录与权限异常处理](#12-未登录与权限异常处理)
  - [1.3 全局统一异常处理配置](#13-全局统一异常处理配置)
  - [1.4 项目实战：数据脱敏与接口防护（三层保险模型）](#14-项目实战数据脱敏与接口防护三层保险模型)
    - [1. 序列化兜底：实体类屏蔽敏感字段](#1-序列化兜底实体类屏蔽敏感字段)
    - [2. 接口层规范：引入 DTO/VO 模式方案](#2-接口层规范引入-dtovo-模式方案)
      - [1 DTO 层：负责“进”（输入脱敏与校验）](#1-dto-层负责进输入脱敏与校验)
      - [2 VO 层：负责“出”（输出脱敏）](#2-vo-层负责出输出脱敏)
      - [3 Controller 层：业务中转](#3-controller-层业务中转)
      - [4 Service 层：对象流转与转换](#4-service-层对象流转与转换)
    - [3. 查询层收敛：按需 Select](#3-查询层收敛按需-select)

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

## 1.4 项目实战：数据脱敏与接口防护（三层保险模型）

在前后端分离项目中，直接将数据库实体类（Entity）返回给前端是非常危险的操作（容易泄露密码、盐值等敏感信息）。以 `vue-admin-template` 的用户列表接口为例，为了防止密码泄露，我们应当采取“数据库不出、接口不传、序列化再兜底”的三层保险规范。

### 1. 序列化兜底：实体类屏蔽敏感字段
通过 Jackson 提供的 `@JsonIgnore` 注解，在 JSON 序列化时自动忽略该字段。这样即使 Controller 误返回了包含密码的对象，前端也无法解析到密码字段。但@JsonIgnore 是“读写都忽略”,如果你以后用 @RequestBody sys_user 来接收前端密码，password 可能会接不到（变 null） **一般只用于Demo**

**文件：** `src/main/java/com/example/practice/entity/sys_user.java`
```java
import com.fasterxml.jackson.annotation.JsonIgnore;
import com.baomidou.mybatisplus.annotation.TableField;

// ... 其他属性
@JsonIgnore   
@TableField("password")
private String password;
```

### 2. 接口层规范：引入 DTO/VO 模式方案

在复杂的业务场景中，Controller 层不应直接操作数据库实体类（Entity）。我们通过引入 **DTO（Data Transfer Object）** 处理输入，**VO（View Object）** 处理输出，实现完全的“三层保险”防护。

#### 1 DTO 层：负责“进”（输入脱敏与校验）
创建 `UserAddDTO`，**专门用于接收前端传来的“新增”或“注册”参数**。它可以包含业务逻辑需要的字段（如“确认密码”），但这些字段不需要存在于数据库表中。

**文件：** `UserAddDTO.java`
```java
public class UserAddDTO {
    private String username;
    private String password;        // 正常接收，无需注解，方便后端加密处理
    private String confirmPassword; // 业务字段：用于校验两次密码是否一致
    private String role;
    private String avatar;

    // Getter and Setter...
}
```

#### 2 VO 层：负责“出”（输出脱敏）
创建 `UserListVO`，仅包含前端列表**展示**需要的字段。通过在类结构上彻底抹除 `password` 字段，确保数据在返回时绝无泄露风险。

**文件：** `UserListVO.java`
```java
public class UserListVO {
    private Integer id;
    private String username;
    private String avatar;
    private String role;

    // Getter and Setter...
}
```

#### 3 Controller 层：业务中转
Controller 不再直接引用 `sys_user`，而是作为 DTO 与 VO 的分发器，保持接口层的纯粹。

**文件：** `UserController.java`
```java
@PostMapping("/add")
public Result add(@RequestBody UserAddDTO userAddDTO) {
    // 1. 业务校验（如校验两次密码是否一致）
    if(!userAddDTO.getPassword().equals(userAddDTO.getConfirmPassword())){
        return Result.error("两次密码输入不一致");
    }
    userService.addUser(userAddDTO);
    return Result.success();
}

@GetMapping("/page")
public Result<Page<UserListVO>> pageQuery(
        @RequestParam(defaultValue = "1") Integer current,
        @RequestParam(defaultValue = "10") Integer size) {
    // 返回封装好的 VO 分页对象
    Page<UserListVO> voPage = userService.getUserVOPage(current, size);
    return Result.success(voPage);
}
```

#### 4 Service 层：对象流转与转换
在 Service 实现类中，利用 `BeanUtils` 进行属性拷贝，并在入库前对密码进行加密处理。

**文件：** `UserServiceImpl.java`
```java
@Override
public void addUser(UserAddDTO dto) {
    sys_user user = new sys_user();
    
    BeanUtils.copyProperties(dto, user);  // DTO -> Entity 拷贝
    
    user.setPassword(BCrypt.hashpw(dto.getPassword())); // 加密后再存入数据库
    this.save(user);
}

@Override
public Page<UserListVO> getUserVOPage(Integer current, Integer size) {
    
    Page<sys_user> entityPage = this.page(Page.of(current, size));  // 1. 查出数据库实体分页
    
    
    Page<UserListVO> voPage = new Page<>(current, size, entityPage.getTotal());  // 2. 将 Entity Page 映射转换为 VO Page
    
    List<UserListVO> voList = entityPage.getRecords().stream().map(user -> {
        UserListVO vo = new UserListVO();
        BeanUtils.copyProperties(user, vo); // 仅拷贝同名属性，password 自动被过滤
        return vo;
    }).collect(Collectors.toList());
    
    voPage.setRecords(voList);
    return voPage;
}
```

### 3. 查询层收敛：按需 Select
在 Service 层拼接查询条件时，避免 `SELECT *`，明确指定需要的字段，从数据库查询源头掐断敏感数据的流出。

**文件：** `src/main/java/com/example/practice/service/UserServiceImpl.java`
```java
LambdaQueryWrapper<sys_user> wrapper = Wrappers.lambdaQuery();
// 只查需要的字段，绝对不查 password
wrapper.select(sys_user::getId, sys_user::getUsername, sys_user::getAvatar, sys_user::getRole);
// 执行分页...
```
