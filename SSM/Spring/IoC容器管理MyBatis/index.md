# Spring 整合 MyBatis (全注解开发) 学习指南目录

## 1. 理论基础与核心组件
* [了解数据源](了解数据源.md#1-数据库框架整合)
    * [1.1 了解数据源](了解数据源.md#11-了解数据源)

## 2. Spring 整合 MyBatis 进阶 (纯注解核心)
* [整合Mybatis框架](整合流程与实现.md#1-整合mybatis框架)
    * [1.1 引入依赖与基本整合](整合流程与实现.md#11-引入依赖与基本整合)
    * [1.2 使用 @MapperScan 自动扫描](整合流程与实现.md#12-使用-mapperscan-自动扫描)
    * [1.3 全注解开发实现](整合流程与实现.md#13-全注解开发实现)
* [使用HikariCP连接池](使用HikariCP连接池.md#1-使用hikaricp连接池)
    * [1.1 导入依赖](使用HikariCP连接池.md#11-导入依赖)
    * [1.2 声明数据源Bean](使用HikariCP连接池.md#12-声明数据源bean)
    * [1.3 配置日志实现](使用HikariCP连接池.md#13-配置日志实现)
    * [1.4 结合Lombok使用](使用HikariCP连接池.md#14-结合lombok使用)

## 3. 事务管理
* [传统Mybatis事务管理 (开发中已淘汰，了解概念即可)](传统方式事务管理.md#1-mybatis事务管理)
* [使用Spring进行事务管理](使用Spring进行事务管理.md#1-使用spring事务管理)
    * [1.1 开启与配置声明式事务](使用Spring进行事务管理.md#11-开启与配置声明式事务)
    * [1.2 事务测试与回滚验证](使用Spring进行事务管理.md#12-事务测试与回滚验证)
    * [1.3 @Transactional注解核心参数](使用Spring进行事务管理.md#13-transactional注解核心参数)
    * [1.4 事务的传播行为](使用Spring进行事务管理.md#14-事务的传播行为)

---

## 附录：Spring 版 MyBatis 全注解开发标准使用步骤总结

现代主流开发中彻底抛弃了 `mybatis-config.xml` 等配置文件，采用全注解的方式进行整合。标准开发步骤总结如下：

### 步骤 1：引入必要的 Maven 依赖
在项目中必须引入 Spring 上下文、Spring JDBC、MyBatis 以及 MyBatis-Spring 整合包。同时，还需要引入 MySQL 驱动、HikariCP 连接池和必要的日志框架（SLF4J）。

### 步骤 2：创建数据表与对应的实体类 (Entity)
根据数据库中的表结构创建 Java 实体类，利用 Lombok 的 `@Data` 注解自动生成 Getter/Setter 及 toString 等方法，保持代码整洁。
```java
package org.example.springbootdemo.Entity;

import lombok.Data;

@Data
public class Student_MyBatis {
    int sid;
    String name;
    String sex;
}
```

### 步骤 3：编写 Mapper 接口
无需编写 XML 映射文件，直接使用 MyBatis 的注解（如 `@Select`, `@Insert` 等）在接口方法上定义 SQL 语句。
```java
package org.example.springbootdemo.mapper;

import org.apache.ibatis.annotations.Select;
import org.example.springbootdemo.Entity.Student_MyBatis;

public interface testmapper {
    @Select("select * from student where sid =1")
    Student_MyBatis getStudent();
}
```

### 步骤 4：编写纯注解配置类 (Configuration)
这是全注解整合的核心步骤。创建一个配置类替代原有的所有 XML 配置文件：
1. 添加 `@Configuration` 注解声明为配置类。
2. 添加 `@MapperScan("包路径")` 注解，让 Spring 自动扫描并代理 Mapper 接口。
3. 声明 `DataSource` 的 Bean，使用高性能的 `HikariDataSource` 并配置数据库连接参数。
4. 声明 `SqlSessionFactory` 的 Bean（使用 `SqlSessionFactoryBean`），并将上一步的数据源注入进去。
```java
package org.example.springbootdemo.config;

import com.zaxxer.hikari.HikariDataSource;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;

@Configuration
@MapperScan("org.example.springbootdemo.mapper")
public class MyBatisConfig {

    // 1. 配置数据源（HikariCP）
    @Bean
    public DataSource dataSource() {
        HikariDataSource ds = new HikariDataSource();
        ds.setJdbcUrl("jdbc:mysql://localhost:3306/study?useSSL=false&serverTimezone=Asia/Shanghai&characterEncoding=utf8");
        ds.setDriverClassName("com.mysql.cj.jdbc.Driver");
        ds.setUsername("root");
        ds.setPassword("niuniu0626");
        return ds;
    }

    // 2. 配置 SqlSessionFactory，消灭传统的 mybatis-config.xml
    @Bean
    public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(dataSource);
        return factoryBean.getObject();
    }
}
```
*(注：如果需要开启声明式事务，还需在此类上追加 `@EnableTransactionManagement` 注解，并注册 `DataSourceTransactionManager` Bean)*

### 步骤 5：启动容器并调用
通过 `AnnotationConfigApplicationContext` 传入我们写好的 `MyBatisConfig.class`，初始化 Spring 容器。此时 Mapper 对象已经被 Spring 作为 Bean 管理，可以直接获取并调用。
```java
package org.example.springbootdemo.MyBatisDemo;

import org.example.springbootdemo.config.MyBatisConfig;
import org.example.springbootdemo.mapper.testmapper;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class demo2 {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = 
                new AnnotationConfigApplicationContext(MyBatisConfig.class);
        
        // 直接从 Spring 容器中获取 Mapper 实例
        testmapper mapper = context.getBean(testmapper.class);
        System.out.println(mapper.getStudent());
        
        context.close();
    }
}
```