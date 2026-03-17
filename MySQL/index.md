# MySQL 学习笔记

## 目录
- [MySQL 学习笔记](#mysql-学习笔记)
  - [目录](#目录)
  - [一、 数据库基本操作](#一-数据库基本操作)
    - [1. 数据库管理](#1-数据库管理)
  - [二、 数据表与数据类型](#二-数据表与数据类型)
    - [1. 常用数据类型](#1-常用数据类型)
    - [2. 表的约束条件](#2-表的约束条件)
    - [3. 数据表的操作](#3-数据表的操作)
  - [三、 数据增删改 (CRUD)](#三-数据增删改-crud)
    - [1. 插入数据](#1-插入数据)
    - [2. 修改数据](#2-修改数据)
    - [3. 删除数据](#3-删除数据)
  - [四、 基础数据查询](#四-基础数据查询)
    - [1. 全表与特定字段查询](#1-全表与特定字段查询)
    - [2. 条件查询与去重](#2-条件查询与去重)
    - [3. 排序与限制数量](#3-排序与限制数量)
  - [五、 进阶查询](#五-进阶查询)
    - [1. 聚合函数](#1-聚合函数)
    - [2. 模糊查询与通配符](#2-模糊查询与通配符)
    - [3. 联合查询 (UNION)](#3-联合查询-union)
    - [4. 表连接 (JOIN)](#4-表连接-join)
    - [5. 子查询](#5-子查询)
  - [六、 外键约束的删除策略](#六-外键约束的删除策略)

---

## 一、 数据库基本操作

### 1. 数据库管理
用于创建、查看、选择和删除数据库。

创建数据库：
```sql
CREATE DATABASE sql_name;
```

查看所有数据库：
```sql
SHOW DATABASES;
```

选择使用某个数据库：
```sql
USE sql_name;
```

删除数据库：
```sql
DROP DATABASE sql_name;
```

## 二、 数据表与数据类型

### 1. 常用数据类型
* **INT**：整数类型。
* **DECIMAL(m, n)**：带有小数点的数值类型。m 表示总共有几位有效数字，n 表示小数部分占几位。
* **VARCHAR(n)**：可变长度字符串。n 表示能够存放的最大字符长度。
* **BLOB**：用于存放二进制数据，如图片、视频等文件。
* **DATE**：日期类型，格式为 YYYY-MM-DD。
* **TIMESTAMP**：时间戳类型，用于记录具体时间，格式为 YYYY-MM-DD HH-MM-SS。

### 2. 表的约束条件
* **PRIMARY KEY (主键)**：唯一标识表中的每一条数据。在同一个表中可以设置多个主键。
* **FOREIGN KEY (外键)**：用于关联另一张表的主键，建立表与表之间的关系。
* **NOT NULL**：限制该字段的值不能为空。
* **AUTO_INCREMENT**：设置整数列自动递增。
* **UNIQUE**：限制该字段的所有值必须唯一。
* **DEFAULT**：为字段设置默认值。

### 3. 数据表的操作
创建数据表，并定义字段的类型与约束：
```sql
CREATE TABLE student(
    student_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(20) UNIQUE,
    major VARCHAR(20) DEFAULT "历史",
    gpa DECIMAL(3,2)
);
```

删除整个数据表：
```sql
DROP TABLE student;
```

修改表结构，新增一个字段：
```sql
ALTER TABLE student ADD gpa DECIMAL(3, 2);
```

修改表结构，删除一个字段：
```sql
ALTER TABLE student DROP COLUMN gpa;
```

## 三、 数据增删改 (CRUD)

### 1. 插入数据
按照表创建时的字段顺序插入数据，也可以指定具体字段插入。

按默认顺序插入：
```sql
INSERT INTO student VALUES();
```

指定字段插入：
```sql
INSERT INTO student (name, major, student_id) VALUES();
```

### 2. 修改数据
根据指定条件更新表中的数据。修改多个字段时用逗号隔开。
```sql
UPDATE student SET major = "历史学", score = 81 WHERE student_id = 1;
```

### 3. 删除数据
根据指定条件删除表中的某条数据。SQL中的不等于符号使用 <>。
```sql
DELETE FROM student WHERE student_id = 1;
```

## 四、 基础数据查询

### 1. 全表与特定字段查询
查询表中的所有数据：
```sql
SELECT * FROM student;
```

查询表中的特定字段：
```sql
SELECT name, major FROM student;
```

### 2. 条件查询与去重
使用 WHERE 进行条件过滤，使用 IN 匹配多个可能的值：
```sql
SELECT * FROM student WHERE major IN("历史", "英语");
```

查询某个字段，并使用 DISTINCT 去除重复项：
```sql
SELECT DISTINCT sex FROM student;
```

### 3. 排序与限制数量
使用 ORDER BY 对结果排序，默认升序，DESC 为降序。多字段排序时用逗号隔开：
```sql
SELECT * FROM student ORDER BY score DESC, student_id;
```

使用 LIMIT 限制返回的数据条数：
```sql
SELECT * FROM student LIMIT 2;
```

## 五、 进阶查询

### 1. 聚合函数
用于对一组数据进行计算并返回单一的值。

计算表中数据的总行数：
```sql
SELECT COUNT(*) FROM employee WHERE birth_date > "1970-01-01";
```

计算某列的平均值：
```sql
SELECT AVG(salary) FROM employee;
```

计算某列的总和：
```sql
SELECT SUM(salary) FROM employee;
```

获取某列的最大值：
```sql
SELECT MAX(salary) FROM employee;
```

获取某列的最小值：
```sql
SELECT MIN(salary) FROM employee;
```

### 2. 模糊查询与通配符
使用 LIKE 配合通配符进行模糊匹配。
* %：表示任意数量的字符。
* _：表示单个字符。

匹配以 335 结尾的电话号码：
```sql
SELECT * FROM client WHERE phone LIKE "%335";
```

匹配以 335 开头的电话号码：
```sql
SELECT * FROM client WHERE phone LIKE "335%";
```

匹配包含 354 的电话号码：
```sql
SELECT * FROM client WHERE phone LIKE "%354%";
```

### 3. 联合查询 (UNION)
将多个 SELECT 语句的结果合并为一个结果集。合并时，各个查询的字段个数和数据类型必须保持一致。
```sql
SELECT name AS total_name FROM employee UNION SELECT client_name FROM client;
```

### 4. 表连接 (JOIN)
通过共同的字段将多张表的数据连接起来。

内连接：
```sql
SELECT name FROM employee JOIN branch ON employee.emp_id = manager_id;
```

左连接：
```sql
SELECT name FROM employee LEFT JOIN branch ON employee.emp_id = manager_id;
```

右连接：
```sql
SELECT name FROM employee RIGHT JOIN branch ON employee.emp_id = manager_id;
```

### 5. 子查询
在一个查询语句嵌套另一个查询语句。当内层查询返回的结果超过一个时，需要使用 IN 而不是 =。
```sql
SELECT name FROM employee WHERE emp_id = (SELECT manager_id FROM branch WHERE branch_name = "研发");
```

## 六、 外键约束的删除策略

在创建外键时，可以设定当主表中的数据被删除时，从表中关联数据的处理方式。

设置为空 (SET NULL)：如果主表关联的记录被删除，则将从表的外键字段设为 NULL。如果该外键同时也是从表的主键，则不能使用此设置。
```sql
FOREIGN KEY (manager_id) REFERENCES employee(emp_id) ON DELETE SET NULL;
```

级联删除 (CASCADE)：如果主表关联的记录被删除，则从表中对应的记录也会跟着一起被删除。
```sql
FOREIGN KEY (emp_id) REFERENCES employee(emp_id) ON DELETE CASCADE;
```