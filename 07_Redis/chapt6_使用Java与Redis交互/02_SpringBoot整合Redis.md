# 1. SpringBoot整合Redis

**目录**
- [1. SpringBoot整合Redis](#1-springboot整合redis)
  - [1.1 引入依赖](#11-引入依赖)
  - [1.2 配置文件](#12-配置文件)
  - [1.3 默认模板类](#13-默认模板类)
  - [1.4 模板类的使用](#14-模板类的使用)
  - [1.5 事务操作](#15-事务操作)
  - [1.6 对象序列化](#16-对象序列化)

## 1.1 引入依赖

我们接着来看如何在SpringBoot项目中整合Redis操作框架，只需要一个starter即可，但是它底层没有用Jedis，而是Lettuce：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

## 1.2 配置文件

starter提供的默认配置会去连接本地的Redis服务器，并使用0号数据库，当然你也可以手动进行修改：

```yaml
spring:
    data:
      redis:
        host: 127.0.0.1
        database: 0
        port: 6379
```

## 1.3 默认模板类

starter已经给我们提供了两个默认的模板类：

```java
@Configuration(
    proxyBeanMethods = false
)
@ConditionalOnClass({RedisOperations.class})
@EnableConfigurationProperties({RedisProperties.class})
@Import({LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class})
public class RedisAutoConfiguration {
    public RedisAutoConfiguration() {
    }

    @Bean
    @ConditionalOnMissingBean(
        name = {"redisTemplate"}
    )

    // 第一个
    //当你把一个 Java 对象（如 User）丢进去时，它会利用 Java 原生的序列化机制将其转为二进制字节流。
    //副作用：在 Redis 终端查看时，你会看到类似 \xac\xed\x00\x05 这样的乱码。这对于调试和跨语言（如 Python/Go 读取）非常不友好

    @ConditionalOnSingleCandidate(RedisConnectionFactory.class)
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<Object, Object> template = new RedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }

    //第一个模板类与第二个模板类分界线

    @Bean
    @ConditionalOnMissingBean
    @ConditionalOnSingleCandidate(RedisConnectionFactory.class)

    //第二个模板类
    //默认序列化机制：它默认使用 StringRedisSerializer。它强制要求 Key 和 Value 都必须是字符串。
    //优势：存进去是什么字符，Redis 里存的就是什么字符。没有任何乱码，可读性极高，且存储空间更小。

    public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) {
        return new StringRedisTemplate(redisConnectionFactory);
    }
}
```

## 1.4 模板类的使用

那么如何去使用这两个模板类呢？我们可以直接注入`StringRedisTemplate`来使用模板：

```java
@SpringBootTest
class SpringBootTestApplicationTests {

    @Autowired
    StringRedisTemplate template;

    @Test
    void contextLoads() {
        ValueOperations<String, String> operations = template.opsForValue();
        //获取一个专门用于操作 Redis 中 String（字符串）类型数据的工具对象
        operations.set("c", "xxxxx");   //设置值
        System.out.println(operations.get("c"));   //获取值
        
        template.delete("c");    //删除键
        System.out.println(template.hasKey("c"));   //判断是否包含键
    }

}
```

实际上所有的值的操作都被封装到了`ValueOperations`对象中，而普通的键操作直接通过模板对象就可以使用了，大致使用方式其实和Jedis一致。

## 1.5 事务操作

我们接着来看看事务操作，由于Spring没有专门的Redis事务管理器，所以只能借用JDBC提供的，只不过无所谓，正常情况下反正我们也要用到这玩意：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

```java
@Service
public class RedisService {

    @Resource
    StringRedisTemplate template;

    @PostConstruct
    public void init(){
        template.setEnableTransactionSupport(true);   //需要开启事务
    }

    @Transactional    //需要添加此注解
    public void test(){
        template.multi();
        template.opsForValue().set("d", "xxxxx");
        template.exec();
    }
}
```

## 1.6 对象序列化

我们还可以为RedisTemplate对象配置一个Serializer来实现对象的JSON存储：

```java
@Test
void contextLoad2() {
    //注意Student需要实现序列化接口才能存入Redis
    template.opsForValue().set("student", new Student());
    System.out.println(template.opsForValue().get("student"));
}
```