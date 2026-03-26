# 常用的类API

## 目录
- [常用的类API](#常用的类api)
  - [目录](#目录)
  - [Math类](#math类)
    - [常用方法](#常用方法)
  - [System类](#system类)
    - [常用方法](#常用方法-1)
  - [Object类](#object类)
    - [概念](#概念)
    - [构造方法](#构造方法)
    - [成员方法](#成员方法)
  - [Date时间类](#date时间类)
    - [构造](#构造)
    - [方法](#方法)
  - [SimpleDateFormat](#simpledateformat)
    - [构造](#构造-1)
    - [方法](#方法-1)
  - [Calendar](#calendar)
    - [概念与获取对象](#概念与获取对象)
    - [常用方法](#常用方法-2)
  - [java.time包](#javatime包)
    - [常用类与方法](#常用类与方法)
  - [包装类](#包装类)
    - [概念与获取](#概念与获取)
    - [成员方法](#成员方法-1)

## Math类

### 常用方法

- 求绝对值
方法声明：public static int abs(int a)
```java
int result = Math.abs(-10);
```

- 向上取整
方法声明：public static double ceil(double a)
```java
double result = Math.ceil(10.1);
```

- 向下取整
方法声明：public static double floor(double a)
```java
double result = Math.floor(10.9);
```

- 四舍五入
方法声明：public static int round(float a)
```java
int result = Math.round(10.5f);
```

- 返回a的b次幂
方法声明：public static double pow(double a, double b)
```java
double result = Math.pow(2.0, 3.0);
```

- 返回double随机值，范围为[0.0, 1.0)
方法声明：public static double random()
```java
double result = Math.random();
```

## System类

### 常用方法

- 终止虚拟机
方法声明：public static void exit(int status)
```java
System.exit(0);
```

- 返回当前时间毫秒值
方法声明：public static long currentTimeMillis()
```java
long time = System.currentTimeMillis();
```

- 数组拷贝
方法声明：public static void arraycopy(Object src, int srcPos, Object dest, int destPos, int length)
```java
int[] arr1 = {1, 2, 3};
int[] arr2 = new int[3];
System.arraycopy(arr1, 0, arr2, 0, 3);
```

## Object类

### 概念
是java中的顶级父类，所有类都继承或间接继承于object。

### 构造方法
空参构造
方法声明：public Object()
```java
Object obj = new Object();
```

### 成员方法

- 返回对象的字符串表示形式
方法声明：public String toString()
```java
String str = obj.toString();
```

- 比较两个对象是否相等。默认比较地址值，使用重写来比较值。
方法声明：public boolean equals(Object obj)
```java
boolean isEqual = obj1.equals(obj2);
```

- 对象克隆
方法声明：protected Object clone()
```java
Object clonedObj = obj.clone();
```

## Date时间类

### 构造

- 无参构造调用System的时间方法，表示当前时间
方法声明：public Date()
```java
Date d1 = new Date();
```

- 有参构造直接赋值，表示指定的时间
方法声明：public Date(long date)
```java
Date d2 = new Date(1000L);
```

### 方法

- 修改时间。在标准时间基础上加t毫秒。
方法声明：public void setTime(long time)
```java
d1.setTime(2000L);
```

- 获取时间对象的毫秒值
方法声明：public long getTime()
```java
long t = d1.getTime();
```

## SimpleDateFormat

### 构造

- 使用默认格式
方法声明：public SimpleDateFormat()
```java
SimpleDateFormat sdf1 = new SimpleDateFormat();
```

- 使用指定格式
方法声明：public SimpleDateFormat(String pattern)
```java
SimpleDateFormat sdf2 = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
```

### 方法

- 将日期对象变为字符串
方法声明：public final String format(Date d)
```java
String timeStr = sdf2.format(new Date());
```

- 将字符串解析得到Date对象
方法声明：public Date parse(String str)
```java
Date parsedDate = sdf2.parse("2026-03-26 12:00:00");
```

## Calendar

### 概念与获取对象
代表了系统当前时间的日历对象，可以单独修改获取时间中的年月日。
是一个抽象类，不能直接创建对象。

- 获取对象
方法声明：public static Calendar getInstance()
```java
Calendar c = Calendar.getInstance();
```

### 常用方法

- 获得时间毫秒值
方法声明：public long getTimeInMillis()
```java
long timeInMillis = c.getTimeInMillis();
```

- 取日历中某个字段信息。索引：0纪元 1年 2月 3一年中的第几周 4一个月中的第几周 5日期。也可以通过Calendar.YEAR /MONTH /DATE。注意事项：星期数字1代表星期日；月份范围0-11。
方法声明：public int get(int field)
```java
int year = c.get(Calendar.YEAR);
```

- 修改日历的某个字段信息
方法声明：public void set(int field, int value)
```java
c.set(Calendar.YEAR, 2026);
```

- 为某个字段增加/减少指定的值
方法声明：public void add(int field, int amount)
```java
c.add(Calendar.MONTH, 1);
```

## java.time包

### 常用类与方法

- 获取当前时间
方法声明：public static LocalDateTime now()
```java
LocalDateTime now = LocalDateTime.now();
```

- 格式化时间
方法声明：public String format(DateTimeFormatter formatter)
```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
String str = now.format(formatter);
```

## 包装类

### 概念与获取
本质: 用一个对象，把基本数据类型给包起来。包括：Byte, Short, Character, Integer, Long, Float, Double。
如何获取: 直接赋值。

```java
Integer i = 10;
```

### 成员方法

- 转二进制
方法声明：public static String toBinaryString(int i)
```java
String bin = Integer.toBinaryString(10);
```

- 转八进制
方法声明：public static String toOctalString(int i)
```java
String oct = Integer.toOctalString(10);
```

- 转十六进制
方法声明：public static String toHexString(int i)
```java
String hex = Integer.toHexString(10);
```

- 将字符串类型的整数转为int类型的整数。8种包装类当中，除了character都有对应的方法进行类型转换。
方法声明：public static int parseInt(String s)
```java
int num = Integer.parseInt("100");
```