# 1. 方法名称拼接自定义SQL

**目录**
- [1. 方法名称拼接自定义SQL](#1-方法名称拼接自定义sql)
  - [1.1 支持的条件判断关键词](#11-支持的条件判断关键词)
  - [1.2 模糊匹配查询示例](#12-模糊匹配查询示例)
  - [1.3 多条件联合查询示例](#13-多条件联合查询示例)
  - [1.4 逻辑判断查询示例](#14-逻辑判断查询示例)
  - [1.5 方法命名规范注意事项](#15-方法命名规范注意事项)

---

虽然接口预置的方法使用起来非常方便，但是如果我们需要进行条件查询等操作或是一些判断，就需要自定义一些方法来实现，同样的，我们不需要编写SQL语句，而是通过方法名称的拼接来实现条件判断。

## 1.1 支持的条件判断关键词

这里列出了所有支持的条件判断名称：

| 属性 | 拼接方法名称示例 | 执行的语句 |
| :--- | :--- | :--- |
| **Distinct** | `findDistinctByLastnameAndFirstname` | `select distinct … where x.lastname = ?1 and x.firstname = ?2` |
| **And** | `findByLastnameAndFirstname` | `… where x.lastname = ?1 and x.firstname = ?2` |
| **Or** | `findByLastnameOrFirstname` | `… where x.lastname = ?1 or x.firstname = ?2` |
| **Is, Equals** | `findByFirstname`, `findByFirstnameIs`, `findByFirstnameEquals` | `… where x.firstname = ?1` |
| **Between** | `findByStartDateBetween` | `… where x.startDate between ?1 and ?2` |
| **LessThan** | `findByAgeLessThan` | `… where x.age < ?1` |
| **LessThanEqual** | `findByAgeLessThanEqual` | `… where x.age <= ?1` |
| **GreaterThan** | `findByAgeGreaterThan` | `… where x.age > ?1` |
| **GreaterThanEqual** | `findByAgeGreaterThanEqual` | `… where x.age >= ?1` |
| **After** | `findByStartDateAfter` | `… where x.startDate > ?1` |
| **Before** | `findByStartDateBefore` | `… where x.startDate < ?1` |
| **IsNull, Null** | `findByAge(Is)Null` | `… where x.age is null` |
| **IsNotNull, NotNull** | `findByAge(Is)NotNull` | `… where x.age not null` |
| **Like** | `findByFirstnameLike` | `… where x.firstname like ?1` |
| **NotLike** | `findByFirstnameNotLike` | `… where x.firstname not like ?1` |
| **StartingWith** | `findByFirstnameStartingWith` | `… where x.firstname like ?1`（参数与附加`%`绑定） |
| **EndingWith** | `findByFirstnameEndingWith` | `… where x.firstname like ?1`（参数与前缀`%`绑定） |
| **Containing** | `findByFirstnameContaining` | `… where x.firstname like ?1`（参数绑定以`%`包装） |
| **OrderBy** | `findByAgeOrderByLastnameDesc` | `… where x.age = ?1 order by x.lastname desc` |
| **Not** | `findByLastnameNot` | `… where x.lastname <> ?1` |
| **In** | `findByAgeIn(Collection<Age> ages)` | `… where x.age in ?1` |
| **NotIn** | `findByAgeNotIn(Collection<Age> ages)` | `… where x.age not in ?1` |
| **True** | `findByActiveTrue` | `… where x.active = true` |
| **False** | `findByActiveFalse` | `… where x.active = false` |
| **IgnoreCase** | `findByFirstnameIgnoreCase` | `… where UPPER(x.firstname) = UPPER(?1)` |

## 1.2 模糊匹配查询示例

比如我们想要实现根据用户名模糊匹配查找用户：

```java
@Repository
public interface AccountRepository extends JpaRepository<Account, Integer> {
    // 按照表中的规则进行名称拼接，不用刻意去记，IDEA会有提示
    List<Account> findAllByUsernameLike(String str);
}
```

我们来测试一下：

```java
@Test
void contextLoads() {
    repository.findAllByUsernameLike("%明%").forEach(System.out::println);
}
```

![image-20230721001035279](https://s2.loli.net/2023/07/21/mioZaUk7Yj3QDxb.png)

## 1.3 多条件联合查询示例

又比如我们想同时根据用户名和ID一起查询：

```java
@Repository
public interface AccountRepository extends JpaRepository<Account, Integer> {
    List<Account> findAllByUsernameLike(String str);

    Account findByIdAndUsername(int id, String username);
    // 也可以使用Optional类进行包装，Optional<Account> findByIdAndUsername(int id, String username);
}
```

```java
@Test
void contextLoads() {
    System.out.println(repository.findByIdAndUsername(1, "小明"));
}
```

## 1.4 逻辑判断查询示例

比如我们想判断数据库中是否存在某个ID的用户：

```java
@Repository
public interface AccountRepository extends JpaRepository<Account, Integer> {
    List<Account> findAllByUsernameLike(String str);
    Account findByIdAndUsername(int id, String username);
    // 使用exists判断是否存在
    boolean existsAccountById(int id);
}
```

## 1.5 方法命名规范注意事项

注意自定义条件操作的方法名称一定要遵循规则，不然会出现异常：

```sh
Caused by: org.springframework.data.repository.query.QueryCreationException: Could not create query for public abstract ...
```

有了这些操作，我们在编写一些简单SQL的时候就很方便了，用久了甚至直接忘记SQL怎么写。