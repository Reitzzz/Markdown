# 1. SpringBoot整合Redis

**目录**
- [1. SpringBoot整合Redis](#1-springboot整合redis)
  - [1.1 引入依赖](#11-引入依赖)
  - [1.2 配置文件](#12-配置文件)
  - [1.3 默认模板类](#13-默认模板类)
  - [1.4 模板类的使用](#14-模板类的使用)
    - [1.4.1 通用键操作与 String 类型 (opsForValue)](#141-通用键操作与-string-类型-opsforvalue)
    - [1.4.2 Hash 类型操作 (opsForHash)](#142-hash-类型操作-opsforhash)
    - [1.4.3 List 类型操作 (opsForList)](#143-list-类型操作-opsforlist)
    - [1.4.4 Set 类型操作 (opsForSet)](#144-set-类型操作-opsforset)
    - [1.4.5 ZSet (Sorted Set) 类型操作 (opsForZSet)](#145-zset-sorted-set-类型操作-opsforzset)
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

在 Spring Boot 中，所有的 Redis 操作都是通过模板类完成的。Spring Data Redis 为不同的数据结构封装了专门的 `Operations` 工具对象。我们可以直接注入 `StringRedisTemplate`（推荐）或 `RedisTemplate` 来使用。

### 1.4.1 通用键操作与 String 类型 (opsForValue)

`opsForValue()` 用于操作简单的 K-V 字符串。此外，模板类本身也提供了一些不依赖具体数据类型的全局通用方法（如删除、判断存在、设置过期时间）。

```java
@SpringBootTest
class RedisStringTest {

    @Autowired
    StringRedisTemplate template;

    @Test
    void testString() {
        // 1. 获取专门用于操作 String 类型的工具对象
        ValueOperations<String, String> operations = template.opsForValue();
        
        // 设置值与获取值
        operations.set("c", "xxxxx");
        System.out.println(operations.get("c"));

        // 2. 模板类提供的全局通用操作
        // 判断是否包含键
        System.out.println(template.hasKey("c"));
        
        // 设置过期时间
        template.expire("c", 60, TimeUnit.SECONDS);
        
        // 删除键
        template.delete("c");
    }
}
```

### 1.4.2 Hash 类型操作 (opsForHash)

Hash 结构类似于 Java 中的 `Map<String, Map<Object, Object>>`，非常适合存储对象（如用户信息）。

```java
@Test
void testHash() {
    HashOperations<String, Object, Object> hashOps = template.opsForHash();

    // 存入单个属性 (大Key, 小Key, 值)
    hashOps.put("user:1", "name", "张三");

    // 批量存入多个属性
    Map<String, String> map = new HashMap<>();
    map.put("age", "20");
    map.put("city", "杭州");
    hashOps.putAll("user:1", map);

    // 获取单个属性
    Object name = hashOps.get("user:1", "name");
    
    // 获取大Key下的所有属性和值
    Map<Object, Object> entries = hashOps.entries("user:1");
    
    // 删除某个小Key
    hashOps.delete("user:1", "city");
}
```

### 1.4.3 List 类型操作 (opsForList)

List 是双端列表，支持从头部（左）或尾部（右）进行插入和弹出，类似 Java 的 `LinkedList`。

```java
@Test
void testList() {
    ListOperations<String, String> listOps = template.opsForList();

    // 从左侧(头部)推入元素
    listOps.leftPush("myList", "java");
    listOps.leftPushAll("myList", "spring", "redis");

    // 获取指定范围的元素 (0 到 -1 代表全部)
    List<String> list = listOps.range("myList", 0, -1);

    // 从右侧(尾部)弹出元素
    String lastItem = listOps.rightPop("myList");
    
    // 获取列表长度
    Long size = listOps.size("myList");
}
```

### 1.4.4 Set 类型操作 (opsForSet)

Set 是无序且不可重复的集合，常用于去重统计、共同关注、标签系统。

```java
@Test
void testSet() {
    SetOperations<String, String> setOps = template.opsForSet();

    // 添加元素
    setOps.add("mySet", "A", "B", "C", "A"); // 自动去重

    // 判断元素是否存在
    Boolean isMember = setOps.isMember("mySet", "A");

    // 获取集合中所有元素
    Set<String> members = setOps.members("mySet");
    
    // 随机弹出一个元素
    String randomItem = setOps.pop("mySet");
}
```

### 1.4.5 ZSet (Sorted Set) 类型操作 (opsForZSet)

ZSet 是有序集合，每个元素关联一个分数（Score），Redis 会根据分数进行排序，非常适合排行榜功能。

```java
@Test
void testZSet() {
    ZSetOperations<String, String> zSetOps = template.opsForZSet();

    // 添加元素并指定分数
    zSetOps.add("rank", "user_A", 100.0);
    zSetOps.add("rank", "user_B", 250.5);

    // 给特定元素增加分数
    zSetOps.incrementScore("rank", "user_A", 50.0);

    // 按分数从大到小获取前两名 (0, 1)
    Set<String> top2 = zSetOps.reverseRange("rank", 0, 1);
    
    // 获取元素的排名 (分数从大到小排，排名从0开始)
    Long rank = zSetOps.reverseRank("rank", "user_B");
}
```

实际上，所有的值操作都被封装到了对应的 `Operations` 对象中，而普通的键操作（如 `delete`）直接通过模板对象即可使用。这种设计方式与我们在控制台操作命令的逻辑是一一对应的。

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