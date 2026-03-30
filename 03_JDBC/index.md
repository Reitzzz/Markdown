# JDBC 核心架构笔记：搭积木式开发指南

## 目录导航
- [JDBC 核心架构笔记：搭积木式开发指南](#jdbc-核心架构笔记搭积木式开发指南)
  - [目录导航](#目录导航)
  - [基石：核心概念](#基石核心概念)
  - [第一层：注册驱动](#第一层注册驱动)
  - [第二层：获取连接](#第二层获取连接)
    - [1. 基础方式 (DriverManager)](#1-基础方式-drivermanager)
    - [2. 进阶方式 (数据连接池)](#2-进阶方式-数据连接池)
  - [第三层：定义SQL语句](#第三层定义sql语句)
  - [第四层：获取执行SQL对象](#第四层获取执行sql对象)
  - [第五层：执行SQL与处理返回结果](#第五层执行sql与处理返回结果)
    - [1. 增删改操作 (执行更新)](#1-增删改操作-执行更新)
    - [2. 查询操作 (获取结果集)](#2-查询操作-获取结果集)
  - [进阶扩展：事务管理](#进阶扩展事务管理)
  - [顶层收尾：释放资源](#顶层收尾释放资源)

---

## 基石：核心概念
* JDBC全称Java DataBase Connectivity。
* 其核心优势在于使用同一套JAVA代码即可操作不同的关系数据库。
* JDBC实际是一个接口，不同软件有不同实现类。

## 第一层：注册驱动
* 注册驱动是建立数据库通信的第一步，但目前在大多数应用中，这一步可写可不写。

```java
Class.forName("com.mysql.cj.jdbc.Driver");
```

## 第二层：获取连接
这一层是建立程序与数据库之间通道的关键，主要分为基础的`DriverManager`方式和进阶的`数据连接池`方式。

### 1. 基础方式 (DriverManager)
* 通过DriverManager获取连接需要提供：url、user、password。
* url语法细节：如果连接本机MySQL服务器且默认端口为3306，可以简写；也可以配置useSSL=false等参数。

```java
String url = "jdbc:mysql://localhost:3306/db_name?useSSL=false";
String user = "root";
String password = "password";
Connection conn = DriverManager.getConnection(url, user, password);
```

### 2. 进阶方式 (数据连接池)
* 数据连接池是一个容器，负责分配、管理数据库连接。
* 允许应用程序使用现有的数据库连接，而不是重新建立。
* 能够释放空闲时间超过最大空闲时间的数据库连接，来避免因为没有释放数据库连接而引起的数据库连接遗漏。
* 主要好处包括：资源重用、提升响应速度、避免数据库连接遗漏。
* 标准接口为DataSource，核心功能是获取连接（Connection getConnection）。
* 以Druid为例，实现流程分为定义配置文件、加载配置文件、获取数据连接池对象、获取连接四个步骤。

```java
Properties prop = new Properties();
prop.load(new FileInputStream("src/main/resources/druid.properties"));
DataSource dataSource = DruidDataSourceFactory.createDataSource(prop);
Connection connection = dataSource.getConnection();
```

## 第三层：定义SQL语句
* 根据业务需求预先准备好需要执行的SQL字符串。

```java
String sql = "select * from user where username = ? and password = ?";
```

## 第四层：获取执行SQL对象
通过上一层建立的`Connection`通道，获取能够真正向数据库发送SQL指令的对象。

* **普通执行SQL对象 (Statement)**
  * 通过 `createStatement()` 方法获取。

```java
Statement stmt = conn.createStatement();
```

* **预编译SQL的执行SQL对象 (PreparedStatement)**
  * 核心优势是防止SQL注入，即防止通过操作输入来修改事先定义好的SQL语句。
  * 通过 `prepareStatement(sql)` 传入SQL并获取对象。
  * 获取对象后，需要使用 `pstmt.setXxx(参数1, 参数2)` 设置参数值。参数1代表问号(?)的位置编号（从1开始），参数2代表问号的值。

```java
PreparedStatement pstmt = conn.prepareStatement(sql);
pstmt.setString(1, "admin");
pstmt.setString(2, "123456");
```

* **执行存储过程中的对象 (CallableStatement)**
  * 通过 `prepareCall(sql)` 获取。

```java
CallableStatement cstmt = conn.prepareCall(sql);
```

## 第五层：执行SQL与处理返回结果

### 1. 增删改操作 (执行更新)
* 使用 `int executeUpdate(sql)` 或者是预编译对象的 `executeUpdate()`（不再需要传递SQL）执行语句。
* 对于DML语句（数据操作语言，主要包括增删改），返回值代表语句影响的行数。
* 对于DDL语句（数据定义语言，主要包括创建删除选择数据库等），执行成功也可能返回0。

```java
int count = pstmt.executeUpdate();
```

### 2. 查询操作 (获取结果集)
* 使用 `ResultSet executeQuery(sql)` 封装DQL查询语句的结果。
* 解析ResultSet对象的步骤：
  * 使用 `boolean next()` 方法将光标从当前位置向下移动一行，并判断当前行是否为有效行并返回值。
  * 使用 `xxx getXxx(参数)` 获取具体数据，XXX代表int、String等数据类型。参数可以是列的编号（int类型，从1开始计算）或列的名称（String类型）。

```java
while(rs.next()){
    int id = rs.getInt(1);
    String name = rs.getString("username");
}
```

## 进阶扩展：事务管理
* **MySQL事务管理**：包含开启事务、提交事务、回滚事务。特点是MySQL默认自动提交事务。

```sql
BEGIN;
COMMIT;
ROLLBACK;
```

* **JDBC事务管理**：同样对应开启事务、提交事务、回滚事务三个步骤，通过代码控制业务逻辑的原子性。

```java
conn.setAutoCommit(false);
conn.commit();
conn.rollback();
```

## 顶层收尾：释放资源
* 在所有数据库交互结束后，必须显式释放相关资源，完成整个积木塔的闭环。

```java
rs.close();
pstmt.close();
conn.close();
```