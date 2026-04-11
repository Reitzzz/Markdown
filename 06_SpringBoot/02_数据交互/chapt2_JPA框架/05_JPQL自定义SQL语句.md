# 1. JPQL自定义SQL语句

**目录**
- [1. JPQL自定义SQL语句](#1-jpql自定义sql语句)
  - [1.1 概述](#11-概述)
  - [1.2 使用JPQL更新指定ID用户密码](#12-使用jpql更新指定id用户密码)
  - [1.3 使用原生SQL根据用户名修改密码](#13-使用原生sql根据用户名修改密码)
  - [1.4 总结与使用场景分析](#14-总结与使用场景分析)

---

## 1.1 概述

虽然SpringDataJPA能够简化大部分数据获取场景，但是难免会有一些特殊的场景，需要使用复杂查询才能够去完成，这时你又会发现，如果要实现，只能用回Mybatis了，因为我们需要自己手动编写SQL语句，过度依赖SpringDataJPA会使得SQL语句不可控。

使用JPA，我们也可以像Mybatis那样，直接编写SQL语句，不过它是JPQL语言，与原生SQL语句很类似，但是它是面向对象的，当然我们也可以编写原生SQL语句。

## 1.2 使用JPQL更新指定ID用户密码

比如我们要更新用户表中指定ID用户的密码：

```java
@Repository
public interface AccountRepository extends JpaRepository<Account, Integer> {

    @Transactional    //DML操作需要事务环境，可以不在这里声明，但是调用时一定要处于事务环境下
    @Modifying     //表示这是一个DML操作
    @Query("update Account set password = ?2 where id = ?1") //这里操作的是一个实体类对应的表，参数使用?代表，后面接第n个参数
    int updatePasswordById(int id, String newPassword);
}
```

```java
@Test
void updateAccount(){
    repository.updatePasswordById(1, "654321");
}
```

## 1.3 使用原生SQL根据用户名修改密码

现在我想使用原生SQL来实现根据用户名称修改密码：

```java
@Transactional
@Modifying
@Query(value = "update users set password = :pwd where username = :name", nativeQuery = true) //nativeQuery = true表示使用原生SQL
                                                                                              //和Mybatis一样，这里使用 :名称 表示参数，也可以继续用上面那种方式
int updatePasswordByUsername(@Param("name") String username,   //我们可以使用@Param指定名称
                             @Param("pwd") String newPassword);
```

```java
@Test
void updateAccount(){
    repository.updatePasswordByUsername("Admin", "654321");
}
```

## 1.4 总结与使用场景分析

通过编写原生SQL，在一定程度上弥补了SQL不可控的问题。

虽然JPA能够为我们带来非常便捷的开发体验，但是正是因为太便捷了，保姆级的体验有时也会适得其反，尤其是一些国内用到复杂查询业务的项目，**可能开发到后期特别庞大时，就只能从底层SQL语句开始进行优化，而由于JPA尽可能地在屏蔽我们对SQL语句的编写，所以后期优化是个大问题**，并且Hibernate相对于Mybatis来说，更加重量级。不过，在微服务的时代，单体项目一般不会太大，JPA的劣势并没有太明显地体现出来。