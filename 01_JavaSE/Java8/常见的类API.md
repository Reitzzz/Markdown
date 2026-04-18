# 常用的类API

## 目录
- [常用的类API](#常用的类api)
  - [目录](#目录)
  - [1. Math类](#1-math类)
    - [1.1 常用方法](#11-常用方法)
  - [2. System类](#2-system类)
    - [2.1 常用方法](#21-常用方法)
  - [3. Object类](#3-object类)
    - [3.1 概念](#31-概念)
    - [3.2 构造方法](#32-构造方法)
    - [3.3 成员方法](#33-成员方法)
  - [4. Date时间类](#4-date时间类)
    - [4.1 构造方法](#41-构造方法)
    - [4.2 成员方法](#42-成员方法)
  - [5. SimpleDateFormat类](#5-simpledateformat类)
    - [5.1 构造方法](#51-构造方法)
    - [5.2 成员方法](#52-成员方法)
  - [6. Calendar类](#6-calendar类)
    - [6.1 概念与获取对象](#61-概念与获取对象)
    - [6.2 常用方法](#62-常用方法)
  - [7. java.time包](#7-javatime包)
    - [7.1 常用类与方法](#71-常用类与方法)
  - [8. 包装类](#8-包装类)
    - [8.1 概念与获取](#81-概念与获取)
    - [8.2 成员方法](#82-成员方法)

## 1. Math类

### 1.1 常用方法

- **求绝对值**  
  方法声明：`public static int abs(int a)`
  ```java
  int result = Math.abs(-10);
  ```

- **向上取整**  
  方法声明：`public static double ceil(double a)`
  ```java
  double result = Math.ceil(10.1);
  ```

- **向下取整**  
  方法声明：`public static double floor(double a)`
  ```java
  double result = Math.floor(10.9);
  ```

- **四舍五入**  
  方法声明：`public static int round(float a)`
  ```java
  int result = Math.round(10.5f);
  ```

- **返回 a 的 b 次幂**  
  方法声明：`public static double pow(double a, double b)`
  ```java
  double result = Math.pow(2.0, 3.0);
  ```

- **返回随机值 [0.0, 1.0)**  
  方法声明：`public static double random()`
  ```java
  double result = Math.random();
  ```

## 2. System类

### 2.1 常用方法

- **终止虚拟机**  
  方法声明：`public static void exit(int status)`
  ```java
  System.exit(0);
  ```

- **返回当前时间毫秒值**  
  方法声明：`public static long currentTimeMillis()`
  ```java
  long time = System.currentTimeMillis();
  ```

- **数组拷贝**  
  方法声明：`public static void arraycopy(Object src, int srcPos, Object dest, int destPos, int length)`
  ```java
  int[] arr1 = {1, 2, 3};
  int[] arr2 = new int[3];
  System.arraycopy(arr1, 0, arr2, 0, 3);
  ```

## 3. Object类

### 3.1 概念
Java 中的顶级父类，所有类都直接或间接继承自 `Object`。

### 3.2 构造方法
空参构造：`public Object()`
```java
Object obj = new Object();
```

### 3.3 成员方法

- **返回对象的字符串表示**  
  方法声明：`public String toString()`
  ```java
  String str = obj.toString();
  ```

- **比较两个对象是否相等**  
  方法声明：`public boolean equals(Object obj)`  
  默认比较地址值，可重写来比较内容。
  ```java
  boolean isEqual = obj1.equals(obj2);
  ```

- **对象克隆**  
  方法声明：`protected Object clone()`
  ```java
  Object clonedObj = obj.clone();
  ```

## 4. Date时间类

### 4.1 构造方法

- **无参构造（当前时间）**  
  方法声明：`public Date()`
  ```java
  Date d1 = new Date();
  ```

- **有参构造（指定毫秒值）**  
  方法声明：`public Date(long date)`
  ```java
  Date d2 = new Date(1000L);
  ```

### 4.2 成员方法

- **修改时间**  
  方法声明：`public void setTime(long time)`
  ```java
  d1.setTime(2000L);
  ```

- **获取毫秒值**  
  方法声明：`public long getTime()`
  ```java
  long t = d1.getTime();
  ```

## 5. SimpleDateFormat类

### 5.1 构造方法

- **使用默认格式**  
  方法声明：`public SimpleDateFormat()`
  ```java
  SimpleDateFormat sdf1 = new SimpleDateFormat();
  ```

- **使用指定格式**  
  方法声明：`public SimpleDateFormat(String pattern)`
  ```java
  SimpleDateFormat sdf2 = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
  ```

### 5.2 成员方法

- **格式化（Date → String）**  
  方法声明：`public final String format(Date d)`
  ```java
  String timeStr = sdf2.format(new Date());
  ```

- **解析（String → Date）**  
  方法声明：`public Date parse(String str)`
  ```java
  Date parsedDate = sdf2.parse("2026-03-26 12:00:00");
  ```

## 6. Calendar类

### 6.1 概念与获取对象
代表系统当前时间的日历对象，可单独修改/获取年月日。抽象类，不能直接实例化。

- **获取对象**  
  方法声明：`public static Calendar getInstance()`
  ```java
  Calendar c = Calendar.getInstance();
  ```

### 6.2 常用方法

- **获取毫秒值**  
  方法声明：`public long getTimeInMillis()`
  ```java
  long timeInMillis = c.getTimeInMillis();
  ```

- **获取字段信息**  
  方法声明：`public int get(int field)`  
  常用字段：`Calendar.YEAR`、`Calendar.MONTH`（0-11）、`Calendar.DATE`。  
  注意：星期中 1 代表星期日。
  ```java
  int year = c.get(Calendar.YEAR);
  ```

- **修改字段信息**  
  方法声明：`public void set(int field, int value)`
  ```java
  c.set(Calendar.YEAR, 2026);
  ```

- **增减字段值**  
  方法声明：`public void add(int field, int amount)`
  ```java
  c.add(Calendar.MONTH, 1);
  ```

## 7. java.time包

### 7.1 常用类与方法

- **获取当前时间**  
  方法声明：`public static LocalDateTime now()`
  ```java
  LocalDateTime now = LocalDateTime.now();
  ```

- **格式化时间**  
  方法声明：`public String format(DateTimeFormatter formatter)`
  ```java
  DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
  String str = now.format(formatter);
  ```

## 8. 包装类

### 8.1 概念与获取
将基本数据类型包装为对象。包括：`Byte`、`Short`、`Character`、`Integer`、`Long`、`Float`、`Double`。  
获取方式：直接赋值（自动装箱）。
```java
Integer i = 10;
```

### 8.2 成员方法

- **转二进制**  
  方法声明：`public static String toBinaryString(int i)`
  ```java
  String bin = Integer.toBinaryString(10);
  ```

- **转八进制**  
  方法声明：`public static String toOctalString(int i)`
  ```java
  String oct = Integer.toOctalString(10);
  ```

- **转十六进制**  
  方法声明：`public static String toHexString(int i)`
  ```java
  String hex = Integer.toHexString(10);
  ```

- **字符串转整数**  
  方法声明：`public static int parseInt(String s)`  
  除 `Character` 外，其他包装类均有类似方法。
  ```java
  int num = Integer.parseInt("100");
  ```