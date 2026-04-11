## 1.2 JDBC简单封装

对于一些插入操作，Spring JDBC为我们提供了更方便的SimpleJdbcInsert工具，它可以实现更多高级的插入功能，比如我们的表主键采用的是自增ID，那么它支持插入后返回自动生成的ID，这就非常方便了：

```java
@Configuration
public class WebConfiguration {

    @Resource
    DataSource source;

    @Test
    void contextLoads() {
        //这个类需要自己创建对象
        SimpleJdbcInsert simple = new SimpleJdbcInsert(source)
                .withTableName("user")   //设置要操作的表名称
                .usingGeneratedKeyColumns("id");    //设置自增主键列
        Map<String, Object> user = new HashMap<>(2);  //插入操作需要传入一个Map作为数据
        user.put("name", "bob");
        user.put("email", "112233@qq.com");
        user.put("password", "123456");
        Number number = simple.executeAndReturnKey(user);   //最后得到的Numver就是得到的自增主键
        System.out.println(number);
    }
}
```

这样就可以快速进行插入操作并且返回自增主键了，还是挺方便的。

![image-20230720224314223](https://s2.loli.net/2023/07/20/xMeBEY3sdKVGmly.png)

当然，虽然SpringJDBC给我们提供了这些小工具，但是其实只适用于简单小项目，稍微复杂一点就不太适合了，下一部分我们将介绍JPA框架。

---
