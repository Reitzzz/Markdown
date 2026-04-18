# 1. Optional类

**目录**
- [1. Optional类](#1-optional类)
  - [1.1 传统空指针处理方式](#11-传统空指针处理方式)
  - [1.2 Optional类基本使用](#12-optional类基本使用)
  - [1.3 Optional的获取与替代方案](#13-optional的获取与替代方案)

---

## 1.1 传统空指针处理方式

Java 8中新引入了Optional特性，来让我们更优雅的处理空指针异常。我们先来看看下面这个例子：

```java
public static void hello(String str){   //现在我们要实现一个方法，将传入的字符串转换为小写并打印
    System.out.println(str.toLowerCase());  //那太简单了吧，直接转换打印一气呵成
}
```

但是这样实现的话，我们少考虑了一个问题，万一给进来的`str`是`null`呢？如果是`null`的话，在调用`toLowerCase`方法时岂不是直接空指针异常了？所以我们还得判空一下：

```java
public static void hello(String str){
    if(str != null) {
        System.out.println(str.toLowerCase());
    }
}
```

## 1.2 Optional类基本使用

但是这样写着就不能一气呵成了，我现在又有强迫症，我就想一行解决，这时，Optional来了，我们可以将任何的变量包装进Optional类中使用：

```java
public static void hello(String str){
    Optional
            .ofNullable(str)   //将str包装进Optional
            .ifPresent(s -> {   //ifPresent表示只有对象不为null才会执行里面的逻辑，实现一个Consumer（接受一个参数，返回值为void）
                System.out.println(s);   
            });
}
```

由于这里只有一句打印，所以我们来优化一下：

```java
public static void hello(String str){
    Optional
            .ofNullable(str)   //将str包装进Optional
            .ifPresent(System.out::println);  
    //println也是接受一个String参数，返回void，所以这里使用我们前面提到的方法引用的写法
}
```

这样，我们就又可以一气呵成了，是不是感觉比之前的写法更优雅。

## 1.3 Optional的获取与替代方案

除了在不为空时执行的操作外，还可以直接从Optional中获取被包装的对象：

```java
System.out.println(Optional.ofNullable(str).get());
```

不过此时当被包装的对象为null时会直接抛出异常，当然，我们还可以指定如果get的对象为null的替代方案：

```java
System.out.println(Optional.ofNullable(str).orElse("VVV"));   //orElse表示如果为空就返回"VVV"
```

其他操作还请回顾JavaSE篇视频教程。