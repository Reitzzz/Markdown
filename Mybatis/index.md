# MyBatis 核心知识体系笔记

## 目录

- [MyBatis 核心知识体系笔记](#mybatis-核心知识体系笔记)
  - [目录](#目录)
  - [1. 总体开发步骤](#1-总体开发步骤)
    - [1.1 创建User表 添加数据](#11-创建user表-添加数据)
    - [1.2 编写Mybatis核心配置文件 替换连接信息](#12-编写mybatis核心配置文件-替换连接信息)
    - [1.3 编写SQL映射文件](#13-编写sql映射文件)
    - [1.4 编码（Mapper代理开发版）](#14-编码mapper代理开发版)
  - [2. Mybatis核心配置文件](#2-mybatis核心配置文件)
    - [2.1 environments](#21-environments)
    - [2.2 typeAliases类型别名](#22-typealiases类型别名)
  - [3. Mapper代理开发](#3-mapper代理开发)
    - [3.1 目的](#31-目的)
    - [3.2 步骤](#32-步骤)
  - [4. 参数传递](#4-参数传递)
    - [4.1 单个参数](#41-单个参数)
      - [4.1.1 POJO 类型（实体类）](#411-pojo-类型实体类)
      - [4.1.2 Map 集合](#412-map-集合)
      - [4.1.3 Collection 集合](#413-collection-集合)
      - [4.1.4 List 集合](#414-list-集合)
      - [4.1.5 Array 数组](#415-array-数组)
      - [4.1.6 其他类型](#416-其他类型)
    - [4.2 多个参数](#42-多个参数)
  - [5. 配置文件完成增删改查](#5-配置文件完成增删改查)
    - [5.1 查询](#51-查询)
      - [5.1.1 查询所有信息](#511-查询所有信息)
      - [5.1.2 查看详情](#512-查看详情)
      - [5.1.3 条件查询](#513-条件查询)
    - [5.2 添加](#52-添加)
      - [5.2.1 基础添加与事务提交](#521-基础添加与事务提交)
      - [5.2.2 获取id](#522-获取id)
    - [5.3 修改](#53-修改)
      - [5.3.1 修改全部字段](#531-修改全部字段)
      - [5.3.2 修改动态字段](#532-修改动态字段)
    - [5.4 删除](#54-删除)
      - [5.4.1 删除单个](#541-删除单个)
      - [5.4.2 批量删除](#542-批量删除)
  - [6. 动态SQL](#6-动态sql)
    - [6.1 if](#61-if)
    - [6.2 where](#62-where)
    - [6.3 set](#63-set)
  - [7. 注解完成增删改查](#7-注解完成增删改查)
    - [7.1 与XML开发的区别](#71-与xml开发的区别)
    - [7.2 使用](#72-使用)

---

## 1. 总体开发步骤

### 1.1 创建User表 添加数据

### 1.2 编写Mybatis核心配置文件 替换连接信息

### 1.3 编写SQL映射文件

### 1.4 编码（Mapper代理开发版）
- (1) 定义类
- (2) 加载核心配置文件
- (3) 获取SqlSessionFactory对象
- (4) 获取SqlSession对象
- (5) 获取Mapper接口的代理对象
- (6) 调用Mapper接口方法执行SQL
- (7) 释放资源

```java
String resource = "mybatis_config.xml"; //加载核心配置文件
InputStream inputStream = Resources.getResourceAsStream(resource); 
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);//获取SqlSessionFactory对象
SqlSession sqlSession = sqlSessionFactory.openSession(true); //获取SqlSession对象
BrandMapper mapper = sqlSession.getMapper(BrandMapper.class); //获取Mapper接口的代理对象
.......... //调用Mapper接口方法执行SQL
sqlSession.close(); //释放资源
```

需要特别注意的属性与细节：
- resource：字符串变量的值必须与项目资源目录下的核心配置文件名完全一致，否则加载时会报错。
- openSession(true)：传入true代表开启自动提交，仅适用于纯查询或单一且不涉及级联业务的增删改。如果是多步写操作，必须无参调用，并在所有业务完成后手动执行commit()。
- BrandMapper.class：getMapper方法接收具体业务接口的字节码对象。操作不同数据表或业务域时需替换为对应的Mapper接口。
- close()：SqlSession占用底层数据库连接，业务执行完毕后必须调用此方法归还连接，防止连接池耗尽。

---

## 2. Mybatis核心配置文件

### 2.1 environments
配置数据库连接环境信息，可以配置多个环境，通过default属性切换不同的环境。

```xml
<environments default="development">
    <environment id="development">
        <transactionManager type="JDBC"/>
        <dataSource type="POOLED">
            <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
            <property name="url" value="jdbc:mysql://localhost:3306/sql_test?serverTimezone=GMT%2B8"/>
            <property name="username" value="root"/>
            <property name="password" value="niuniu0626"/>
        </dataSource>
    </environment>
</environments>
```

### 2.2 typeAliases类型别名
需要遵循顺序，在configuration中第一个写。

设置缩写：
```xml
<typeAliases> 
    <typeAlias alias="Author" type="domain.blog.Author"/> 
    <typeAlias alias="Blog" type="domain.blog.Blog"/> 
    <typeAlias alias="Comment" type="domain.blog.Comment"/> 
    <typeAlias alias="Post" type="domain.blog.Post"/> 
    <typeAlias alias="Section" type="domain.blog.Section"/> 
    <typeAlias alias="Tag" type="domain.blog.Tag"/> 
</typeAliases>
```

设置包扫描（在指定包名下搜索需要的Javabean）：<br>
1. 扫描目标：**普通的 Java 实体类（POJO/JavaBean）。**
2. 核心作用: 
自动起别名。它告诉 MyBatis 去指定的包下扫描所有的实体类，并默认以类名首字母小写（或直接使用类名）作为别名。

3. 带来的好处：在编写后续的 SQL Mapper XML 文件时，你的 resultType 或 parameterType 只需要写 User 或 user，而不需要写冗长的全限定类名 domain.blog.User。
```xml
<typeAliases> 
    <package name="domain.blog"/> 
</typeAliases>
```

---

## 3. Mapper代理开发

### 3.1 目的
- 解决原生方式中的硬编码
- 简化后期执行SQL

### 3.2 步骤
1. 定义与SQL映射文件同名的Mapper接口，并放在同一目录。
   - 放在同一目录指的是放在同一名字的包下面：
     `src/main/java/Mybatis_demo/tb_brandMapper.java`
     `src/main/resources/Mybatis_demo/tb_brandMapper.xml`
   - 同一目录则可以在配置文件中使用包扫描来简化SQL映射文件的加载：
     1. 扫描目标：**Mapper 接口及其对应的 XML 映射文件。**

     2. 核心作用：批量加载映射器。它告诉 MyBatis 去指定的包（如 Mybatis_demo）下扫描所有的 Mapper 接口，并自动将它们注册到引擎中。

     3. 带来的好处：如果你的项目里有 50 个表对应的 50 个 Mapper 接口，不需要在核心配置文件中写 50 遍 <mapper resource="..."/>，只需一行包扫描即可全部加载。
     ```xml
     <mappers>
         <package name="Mybatis_demo"/>
     </mappers>
     ```
1. 设置SQL映射文件的namespace属性为接口的全限定名（包名.类名）。
2. 在Mapper接口中定义方法，方法名就是SQL映射文件中sql语句的id，并保持参数类型和返回值运行一致。
3. 编码：
   - 4(1) 通过 SqlSession 的 getMapper 方法获取 Mapper 接口的代理对象：
     ```java
     UserMapper mapper = sqlSession.getMapper(UserMapper.class);
     ```
   - 4(2) 调用对应方法完成 sql 的执行：
     ```java
     List<User> users = mapper.selectAll();
     ```

---

## 4. 参数传递

### 4.1 单个参数

#### 4.1.1 POJO 类型（实体类）
底层逻辑：直接使用，XML 中的占位符名称必须和实体类中的属性名完全一致（MyBatis 会通过反射调用 getter 方法获取值）。
```java
void addBrand(Brand brand);
```
```xml
<insert id="addBrand">
    insert into tb_brand (brand_name, company_name)
    values (#{brandName}, #{companyName});
</insert>
```

#### 4.1.2 Map 集合
底层逻辑：直接使用，XML 中的占位符名称必须和 Map 中的键（Key）名一致。
```java
void addBrand(Map<String, Object> map);
```
```xml
<insert id="addBrand">
    insert into tb_brand (brand_name, company_name)
    values (#{brandName}, #{companyName});
</insert>
```

#### 4.1.3 Collection 集合
底层逻辑：MyBatis 默认将其封装在一个 Map 中，键名为 collection 或 arg0。
```java
List<Brand> selectByIds(Collection<Integer> ids);
```
```xml
<select id="selectByIds" resultType="brand">
    select * from tb_brand where id in
    <foreach collection="collection" item="id" separator="," open="(" close=")">
        #{id}
    </foreach>
</select>
```

#### 4.1.4 List 集合
底层逻辑：List 是 Collection 的子接口，MyBatis 对其做了特殊照顾，除了 collection 和 arg0，还可以使用键名 list。
```java
List<Brand> selectByIds(List<Integer> ids);
```
```xml
<select id="selectByIds" resultType="brand">
    select * from tb_brand where id in
    <foreach collection="list" item="id" separator="," open="(" close=")">
        #{id}
    </foreach>
</select>
```

#### 4.1.5 Array 数组
底层逻辑：MyBatis 默认将其封装在 Map 中，键名为 array 或 arg0。
```java
List<Brand> selectByIds(int[] ids);
```
```xml
<select id="selectByIds" resultType="brand">
    select * from tb_brand where id in
    <foreach collection="array" item="id" separator="," open="(" close=")">
        #{id}
    </foreach>
</select>
```

#### 4.1.6 其他类型
底层逻辑：直接使用。由于只有一个简单参数，MyBatis 不挑剔，在 #{} 里随便写什么名字都能取到值，但为了可读性，建议写和参数名一样的名字。
```java
Brand selectById(int id);
```
```xml
<select id="selectById" resultType="brand">
    select * from tb_brand where id = #{id};
</select>
```

### 4.2 多个参数
当有多个参数时，MyBatis 的默认处理方式会让代码变得非常难看，通常配合 @Param 注解使用。
```java
List<Brand> selectByCondition(@Param("status") int status, @Param("companyName") String companyName);
```
```xml
<select id="selectByCondition" resultType="brand">
    select * from tb_brand where status = #{status} and company_name like #{companyName};
</select>
```

---

## 5. 配置文件完成增删改查

### 5.1 查询

#### 5.1.1 查询所有信息
实体类中属性名字与数据库中不同时，查询结果为null。

1. 使用SQL片段解决(起别名)，缺点是不灵活。
```xml
<sql id="brand_column">
    id, brand_name as brandName, company_name as companyName, ordered, description, status
</sql>

<select id="selectAll" resultType="brand">
    select
        <include refid="brand_column" />
    from tb_brand;
</select>
```

2. 使用resultMap解决(映射)，较为常用。column为表的列名，property为实体类的变量名。
```xml
<resultMap id="brandResultMap" type="brand">
    <result column="brand_name" property="brandName" />
    <result column="company_name" property="companyName" />
</resultMap>

<select id="selectAll" resultMap="brandResultMap">
    select * from tb_brand;
</select>
```

#### 5.1.2 查看详情
占位符类型：
- #{}：替换为？，防止SQL注入，首选。
- ${}：字符拼接，表名或列名不固定时使用。

参数类型：parameterType (一般省略不写)。

特殊字符处理：
- 转义字符：如 `<` 写为 `&lt;`
- CDATA区：输入大写CD后回车，再在里面输入。
  ```xml
  <![CDATA[
  <
  ]]>
  ```
```xml
<select id="selectById" resultMap="brandResultMap1">
        select
            *
        from tb_brand
        where id = #{id}
    </select>
```

#### 5.1.3 条件查询

**多条件静态查询** (传递参数，写在接口里)：**仅了解 几乎不使用**
- 散装参数：如果方法中有多个参数，需要使用 @Param("SQL参数占位符名称")。
  ```java
  List<Brand> selectByCondition(@Param("status")int status, @Param("companyName") String companyName, @Param("brandName") String brandName);
  ```
- 对象参数：需要保证SQL中的参数名和实体类属性完全一致。
  ```java
  List<Brand> selectByCondition(Brand brand);
  ```
- Map参数：需要保证SQL中的参数名和Map中的键的名称完全一致。
  ```java
  List<Brand> selectByCondition(Map map);
  ```



**多条件动态查询**：
- if 使用：
  int直接判断是否=null
  ```xml
  <if test="status!=null">
      status = #{status}
  </if>
  ```
  String需要同时判断是否=null和是否=''
  ```xml
  <if test="companyName != null and companyName != ''">
      and company_name like #{companyName}
  </if>
  ```
- 问题：多条件时 如果第一个条件未输入则报错。
- 解决：
  1. 每个条件都使用and 再配合恒等式
  2. 通过 `<where>` 替换where关键字
```xml
<select id="selectByCondition" resultMap="brandResultMap">
    select *
    from tb_brand
    <where>
        <if test="status!=null">
            status = #{status}
        </if>
        <if test="companyName != null and companyName != ''">
            and company_name like #{companyName}
        </if>
        <if test="brandName != null and brandName != ''">
            and brand_name like #{brandName}
        </if>
    </where>
</select>
```

单条件动态查询：
- choose-switch (when-case otherwise-default)
```xml
<select id="selectbysingleCondition" resultMap="brandResultMap">
    select *
    from tb_brand
    <where>
    <choose>
        <when test="status!=null">
            status = #{status}
        </when>

        <when test="companyName != null and companyName != ''">
            company_name like #{companyName}
        </when>

        <when test="brandName != null and brandName != ''">
            brand_name like #{brandName}
        </when>
        <otherwise>
            1=1
        </otherwise>
    </choose>
    </where>
</select>
```
其中otherwise标签可写可不写，`<where>` 标签已经代替其功能。

### 5.2 添加

#### 5.2.1 基础添加与事务提交
SQL映射文件：
```xml
<insert id="add">
    insert into tb_brand(brand_name,company_name,ordered,description,status)
    values(#{brandName},#{companyName},#{ordered},#{description},#{status})
</insert>
```
测试类commit：
自动commit：
```java
SqlSession sqlSession = sqlSessionFactory.openSession(true);
```
手动commit：
```java
sqlSession.commit();
```

#### 5.2.2 获取id
useGeneratedKeys="true"：这表示开启主键自增回填功能。MyBatis 会在执行插入 SQL 后，获取数据库自动生成的 ID。
keyProperty="id"：这指定了将获取到的主键值存回到实体对象的哪个属性中。
```xml
<insert id="addOrder" useGeneratedKeys="true" keyProperty="id">
    insert into tb_order (payment, payment_type, status)
    values (#{payment},#{paymentType},#{status});
</insert>
```

### 5.3 修改

#### 5.3.1 修改全部字段
如果你传入的对象中某个属性为 null或只传入部分数据，数据库中对应的字段也会被更新为 null。
```xml
<update id="update">
    update tb_brand
    set brand_name = #{brandName},
        company_name = #{companyName},
        ordered = #{ordered},
        description = #{description},
        status = #{status}
    where id = #{id};
</update>
```

#### 5.3.2 修改动态字段
```xml
<update id="update">
    update tb_brand
    <set>
        <if test="brandName != null and brandName != ''">
            brand_name = #{brandName},
        </if>
        <if test="companyName != null and companyName != ''">
            company_name = #{companyName},
        </if>
        <if test="ordered != null">
            ordered = #{ordered},
        </if>
        <if test="description != null and description != ''">
            description = #{description},
        </if>
        <if test="status != null">
            status = #{status}
        </if>
    </set>
    where id = #{id}
</update>
```

### 5.4 删除

#### 5.4.1 删除单个
```xml
<delete id="deleteById">
    delete from tb_brand
    where id = #{id}
</delete>
```

#### 5.4.2 批量删除
接口方法：
```java
void deleteByIds(@Param("ids") int[] ids);
```
SQL映射：
```xml
<delete id="deleteByIds">
    delete from tb_brand where id
    in (
        <foreach collection="ids" item="id" separator=",">
            #{id}
        </foreach>
    );
</delete>
```

---

## 6. 动态SQL

### 6.1 if
用于判断参数是否为空，从而动态拼装SQL片段。
```xml
<select id="selectByCondition" resultType="brand">
    select * from tb_brand where 1=1
    <if test="status != null">
        and status = #{status}
    </if>
</select>
```

### 6.2 where
用于替换 WHERE 关键字，并能自动去除第一个条件前面的 and 或 or。
```xml
<select id="selectByCondition" resultType="brand">
    select * from tb_brand
    <where>
        <if test="status != null">
            status = #{status}
        </if>
        <if test="companyName != null and companyName != ''">
            and company_name like #{companyName}
        </if>
    </where>
</select>
```

### 6.3 set
用于更新操作，能自动在片段前加上 SET 关键字，并去除多余的逗号。
```xml
<update id="updateBrand">
    update tb_brand
    <set>
        <if test="brandName != null and brandName != ''">
            brand_name = #{brandName},
        </if>
        <if test="companyName != null and companyName != ''">
            company_name = #{companyName},
        </if>
    </set>
    where id = #{id}
</update>
```

---

## 7. 注解完成增删改查

### 7.1 与XML开发的区别
- 1. 功能：注解用于完成简单功能；XML文件用于完成复杂功能。
- 2. 编写：注解开发中SQL语句位于Mapper接口；XML开发中位于XML文件。

### 7.2 使用
```java
@Select("select * from tb_user where id = #{id}")
public User selectById(int id);
```
包含：@Select, @Insert, @Update, @Delete。