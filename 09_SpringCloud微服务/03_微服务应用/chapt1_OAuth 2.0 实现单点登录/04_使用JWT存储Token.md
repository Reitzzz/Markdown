# 1. 使用JWT存储Token

## 目录
- [1. 使用JWT存储Token](#1-使用jwt存储token)
  - [目录](#目录)
  - [1.1 JWT概念与组成](#11-jwt概念与组成)
  - [1.2 相关核心概念补充](#12-相关核心概念补充)
  - [1.3 验证服务器JWT配置（Spring Boot 3.2 + Spring Authorization Server）](#13-验证服务器jwt配置spring-boot-32--spring-authorization-server)
  - [1.4 资源服务器配置与测试（Spring Boot 3.2）](#14-资源服务器配置与测试spring-boot-32)

---

## 1.1 JWT概念与组成

官网：https://jwt.io

JSON Web Token令牌（JWT）是一个开放标准（[RFC 7519](https://tools.ietf.org/html/rfc7519)），它定义了一种紧凑和自成一体的方式，用于在各方之间作为JSON对象安全地传输信息。这些信息可以被验证和信任，因为它是数字签名的。JWT可以使用密钥（使用**HMAC**算法）或使用**RSA**或**ECDSA**进行公钥/私钥对进行签名。

实际上，我们之前都是携带Token向资源服务器发起请求后，资源服务器由于不知道我们Token的用户信息，所以需要向验证服务器询问此Token的认证信息，这样才能得到Token代表的用户信息，但是各位是否考虑过，如果每次用户请求都去查询用户信息，那么在大量请求下，验证服务器的压力可能会非常的大。而使用JWT之后，Token中会直接保存用户信息，这样资源服务器就不再需要询问验证服务器，自行就可以完成解析，我们的目标是不联系验证服务器就能直接完成验证。

JWT令牌的格式如下：

![image-20230307000004710](https://s2.loli.net/2023/03/07/Xu8lxYhKoJNr6it.png)

一个JWT令牌由3部分组成：标头(Header)、有效载荷(Payload)和签名(Signature)。在传输的时候，会将JWT的3部分分别进行Base64编码后用`.`进行连接形成最终需要传输的字符串。

* **标头**：包含一些元数据信息，比如JWT签名所使用的加密算法，还有类型，这里统一都是JWT。
* **有效载荷**：包括用户名称、令牌发布时间、过期时间、JWT ID等，当然我们也可以自定义添加字段，我们的用户信息一般都在这里存放。
* **签名**：首先需要指定一个密钥，该密钥仅仅保存在服务器中，保证不能让其他用户知道。然后使用Header中指定的算法对Header和Payload进行base64加密之后的结果通过密钥计算哈希值，然后就得出一个签名哈希。这个会用于之后验证内容是否被篡改。

## 1.2 相关核心概念补充

这里还是补充一下一些概念，因为很多东西都是我们之前没有接触过的：

* **Base64：** 就是包括小写字母a-z、大写字母A-Z、数字0-9、符号"+"、"/"一共64个字符的字符集（末尾还有1个或多个`=`用来凑够字节数），任何的符号都可以转换成这个字符集中的字符，这个转换过程就叫做Base64编码，编码之后会生成只包含上述64个字符的字符串。相反，如果需要原本的内容，我们也可以进行Base64解码，回到原有的样子。

```java
    public void test(){
        String str = "你们可能不知道只用20万赢到578万是什么概念";
        //Base64不只是可以对字符串进行编码，任何byte[]数据都可以，编码结果可以是byte[]，也可以是字符串
        String encodeStr = Base64.getEncoder().encodeToString(str.getBytes());
        System.out.println("Base64编码后的字符串："+encodeStr);

        System.out.println("解码后的字符串："+new String(Base64.getDecoder().decode(encodeStr)));
    }
```
    注意Base64不是加密算法，只是一种信息的编码方式而已。

* **加密算法：** 加密算法分为对称加密和非对称加密，其中**对称加密（Symmetric Cryptography）**比较好理解，就像一把锁配了两把钥匙一样，这两把钥匙你和别人都有一把，然后你们直接传递数据，都会把数据用锁给锁上，就算传递的途中有人把数据窃取了，也没办法解密，因为钥匙只有你和对方有，没有钥匙无法进行解密，但是这样有个问题，既然解密的关键在于钥匙本身，那么如果有人不仅窃取了数据，而且对方那边的治安也不好，于是顺手就偷走了钥匙，那你们之间发的数据不就凉凉了吗。

    因此，**非对称加密（Asymmetric Cryptography）**算法出现了，它并不是直接生成一把钥匙，而是生成一个公钥和一个私钥，私钥只能由你保管，而公钥交给对方或是你要发送的任何人都行，现在你需要把数据传给对方，那么就需要使用私钥进行加密，但是，这个数据只能使用对应的公钥进行解密，相反，如果对方需要给你发送数据，那么就需要用公钥进行加密，而数据只能使用私钥进行解密，这样的话就算对方的公钥被窃取，那么别人发给你的数据也没办法解密出来，因为需要私钥才能解密，而只有你才有私钥。

    因此，非对称加密的安全性会更高一些，包括HTTPS的隐私信息正是使用非对称加密来保障传输数据的安全（当然HTTPS并不是单纯地使用非对称加密完成的，感兴趣的可以去了解一下）

    对称加密和非对称加密都有很多的算法，比如对称加密，就有：DES、IDEA、RC2，非对称加密有：RSA、DAS、ECC

* **不可逆加密算法：** 常见的不可逆加密算法有MD5, HMAC, SHA-1, SHA-224, SHA-256, SHA-384, 和SHA-512, 其中SHA-224、SHA-256、SHA-384，和SHA-512我们可以统称为SHA2加密算法，SHA加密算法的安全性要比MD5更高，而SHA2加密算法比SHA1的要高，其中SHA后面的数字表示的是加密后的字符串长度，SHA1默认会产生一个160位的信息摘要。经过不可逆加密算法得到的加密结果，是无法解密回去的，也就是说加密出来是什么就是什么了。本质上，其就是一种哈希函数，用于对一段信息产生摘要，以**防止被篡改**。

    实际上这种算法就常常被用作信息摘要计算，同样的数据通过同样的算法计算得到的结果肯定也一样，而如果数据被修改，那么计算的结果肯定就不一样了。

## 1.3 验证服务器JWT配置（Spring Boot 3.2 + Spring Authorization Server）

当前项目使用的是 **Spring Boot 3.2** 和 **Spring Authorization Server**，不是旧版 `spring-security-oauth2`。因此这里不再使用 `JwtAccessTokenConverter`、`JwtTokenStore`、`DefaultTokenServices` 这些旧 API。

新版 Authorization Server 默认也是签发 JWT，只是默认示例通常使用 RSA 非对称密钥，并通过 `jwk-set-uri` 让资源服务器远程获取公钥。这里为了和本节目标一致，我们改成最简单的 **对称密钥 HS256**：认证服务器用密钥签发 JWT，资源服务器用同一个密钥校验 JWT。

先在认证服务 `auth_service/src/main/resources/application.yml` 中配置签名密钥：

```yaml
jwt:
  signing-key: lbwnb
```

然后修改认证服务的 `AuthorizationServerConfiguration`。核心是提供一个对称密钥 `JWKSource`，并指定 JWT 使用 `HS256` 算法签名：

```java
package com.example.authservice.config;



@Configuration
public class AuthorizationServerConfiguration {

    @Value("${jwt.signing-key:lbwnb}")
    private String signingKey;

    @Bean
    @Order(1)
    public SecurityFilterChain authorizationServerSecurityFilterChain(HttpSecurity http) throws Exception {
        OAuth2AuthorizationServerConfiguration.applyDefaultSecurity(http);

        http.getConfigurer(OAuth2AuthorizationServerConfigurer.class)
                .oidc(Customizer.withDefaults());

        http.exceptionHandling(exceptions -> exceptions
                .authenticationEntryPoint((request, response, authException) ->
                        response.sendRedirect(request.getContextPath() + "/login")));

        return http.build();
    }

    @Bean
    public JWKSource<SecurityContext> jwkSource() {
        SecretKey secretKey = new SecretKeySpec(signingKeyBytes(), "HmacSHA256");
        OctetSequenceKey hmacKey = new OctetSequenceKey.Builder(secretKey)
                .keyID("auth-service-hs256")
                .algorithm(JWSAlgorithm.HS256)
                .build();

        JWKSet jwkSet = new JWKSet(hmacKey);
        return new ImmutableJWKSet<>(jwkSet);
    }

    @Bean
    public OAuth2TokenCustomizer<JwtEncodingContext> jwtCustomizer() {
        return context -> context.getJwsHeader().algorithm(MacAlgorithm.HS256);
    }

    private byte[] signingKeyBytes() {
        try {
            return MessageDigest.getInstance("SHA-256")
                    .digest(signingKey.getBytes(StandardCharsets.UTF_8));
        } catch (Exception e) {
            throw new IllegalStateException(e);
        }
    }

    @Bean
    public AuthorizationServerSettings authorizationServerSettings() {
        return AuthorizationServerSettings.builder().build();
    }
}
```

注意：HS256 要求密钥长度至少 256 bit，而 `lbwnb` 本身太短。这里为了保持配置简单，将配置值通过 SHA-256 派生成 256 bit 的 HMAC 密钥。资源服务器也必须使用同样的派生方式，否则签名校验无法通过。

`RegisteredClientRepository`、登录用户配置等内容保持原来的 Spring Authorization Server 写法即可，不需要再配置 `TokenStore`。

## 1.4 资源服务器配置与测试（Spring Boot 3.2）

资源服务器不再配置 `jwk-set-uri`，而是配置和认证服务器一致的密钥：

```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          secret-key: lbwnb
```

然后在资源服务的 `ResourceServerConfiguration` 中提供一个本地 `JwtDecoder`：

```java
package com.example.bookservice.config;


@Configuration
public class ResourceServerConfiguration {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .csrf(AbstractHttpConfigurer::disable)
                .authorizeHttpRequests(authorize -> authorize
                        .anyRequest().hasAuthority("SCOPE_book"))
                .oauth2ResourceServer(oauth2 -> oauth2.jwt(Customizer.withDefaults()));

        return http.build();
    }

    @Bean
    public JwtDecoder jwtDecoder(@Value("${spring.security.oauth2.resourceserver.jwt.secret-key}") String secretKey) {
        SecretKeySpec secretKeySpec = new SecretKeySpec(signingKeyBytes(secretKey), "HmacSHA256");
        return NimbusJwtDecoder.withSecretKey(secretKeySpec)
                .macAlgorithm(MacAlgorithm.HS256)
                .build();
    }

    private byte[] signingKeyBytes(String secretKey) {
        try {
            return MessageDigest.getInstance("SHA-256")
                    .digest(secretKey.getBytes(StandardCharsets.UTF_8));
        } catch (Exception e) {
            throw new IllegalStateException(e);
        }
    }
}
```

如果是 `user_service`，权限可以写成：

```java
.anyRequest().hasAuthority("SCOPE_user")
```

如果是 `borrow_service`，权限可以写成：

```java
.requestMatchers("/borrow/blocked").permitAll()
.anyRequest().hasAuthority("SCOPE_borrow")
```

这样资源服务器拿到请求中的 Bearer Token 后，会直接在本地用 `lbwnb` 派生出的 HMAC 密钥验证签名并解析用户信息，不需要再请求认证服务器的 `/oauth2/jwks` 接口。

启动认证服务器和资源服务器后，重新获取 Token，可以看到 `access_token` 是三段式的 JWT 字符串：

```text
header.payload.signature
```

把前两段 Base64URL 解码后，可以看到 JWT 标头和载荷中的信息，例如签名算法、用户名、scope、过期时间等。

如果 Token 被篡改，或者资源服务器配置的密钥与认证服务器不一致，请求资源接口时就会返回 401。
