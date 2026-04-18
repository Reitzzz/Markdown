# 1. 改进的 Stream API

- [1. 改进的 Stream API](#1-改进的-stream-api)
  - [1.1 Java 8 Stream 回顾](#11-java-8-stream-回顾)
  - [1.2 新增 ofNullable 方法](#12-新增-ofnullable-方法)
  - [1.3 增强的 iterate 方法](#13-增强的-iterate-方法)
  - [1.4 新增 takeWhile 与 dropWhile 方法](#14-新增-takewhile-与-dropwhile-方法)

---

还记得我们之前在JavaSE中学习的Stream流吗？当然这里不是指进行IO操作的流，而是JDK1.8新增的Stream API，通过它大大方便了我们的编程。

## 1.1 Java 8 Stream 回顾

```java
public static void main(String[] args) {
    Stream
            .of("A", "B", "B", "C")   //这里我们可以直接将一些元素封装到Stream中
            .filter(s -> s.equals("B"))   //通过过滤器过滤
            .distinct()   //去重
            .forEach(System.out::println);   //最后打印
}
```

自从有了Stream，我们对于集合的一些操作就大大地简化了，对集合中元素的批量处理，只需要在Stream中一气呵成（具体的详细操作请回顾JavaSE篇）

如此方便的框架，在Java 9得到了进一步的增强：

## 1.2 新增 ofNullable 方法

```java
public static void main(String[] args) {
    Stream
            .of(null)   //如果传入null会报错
            .forEach(System.out::println);

    Stream
            .ofNullable(null) //使用新增的ofNullable方法，这样就不会了，不过这样的话流里面就没东西了
            .forEach(System.out::println);
}
```

## 1.3 增强的 iterate 方法

还有，我们可以通过迭代快速生成一组数据（实际上Java 8就有了，这里新增的是允许结束迭代的）：

```java
public static void main(String[] args) {
    Stream
            .iterate(0, i -> i + 1)   

//Java8只能像这样生成无限的流，第一个参数是种子，就是后面的UnaryOperator的参数i一开始的值，最后会返回一个值作为i的新值
// 每一轮都会执行UnaryOperator并生成一个新值到流中，这个是源源不断的，如果不加 limit() 进行限制的话，将无限生成下去。

                .limit(20)   //这里限制生成20个
            .forEach(System.out::println); 
}
```

```java
public static void main(String[] args) {
    Stream
      //参考:for (int i = 0; i < 20; i++)
            .iterate(0, i -> i < 20, i -> i + 1)  //快速生成一组0~19的int数据，中间可以添加一个断言，表示什么时候结束生成
            .forEach(System.out::println);
}
```

## 1.4 新增 takeWhile 与 dropWhile 方法

Stream还新增了对数据的截断操作，比如我们希望在读取到某个元素时截断，不再继续操作后面的元素：

```java
public static void main(String[] args) {
    Stream
            .iterate(0, i -> i + 1)
            .limit(20)
            .takeWhile(i -> i < 10)   //满足条件（i<10)则正常通过，一旦大于等于10（不满足条件）直接截断
            .forEach(System.out::println);
}
```

```java
public static void main(String[] args) {
    Stream
            .iterate(0, i -> i + 1)
            .limit(20)
            .dropWhile(i -> i < 10)   //满足条件(i<10)则截断，只有当不满足条件时再开始通过
            .forEach(System.out::println);
}
```