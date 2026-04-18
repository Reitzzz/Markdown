# 1. 针对于String类的方法增强

- [1. 针对于String类的方法增强](#1-针对于string类的方法增强)
  - [1.1 isBlank 与 lines 方法](#11-isblank-与-lines-方法)
  - [1.2 repeat 方法](#12-repeat-方法)
  - [1.3 strip 相关方法](#13-strip-相关方法)

---

在Java 11为String新增一些更加方便的操作：

## 1.1 isBlank 与 lines 方法

```java
public static void main(String[] args) {
    var str = "AB\nC\nD";
    System.out.println(str.isBlank());    //isBlank方法用于判断是否字符串为空或者是仅包含空格
    str
            .lines()   //根据字符串中的\n换行符进行切割，分为多个字符串，并转换为Stream进行操作
            .forEach(System.out::println);
}
```

## 1.2 repeat 方法

我们还可以通过`repeat()`方法来让字符串重复拼接：

```java
public static void main(String[] args) {
    String str = "ABCD";   //比如现在我们有一个ABCD，但是现在我们想要一个ABCDABCD这样的基于原本字符串的重复字符串
    System.out.println(str.repeat(2));  //  .repeat(2)表示打印两次
}
```

## 1.3 strip 相关方法

我们也可以快速地进行空格去除操作：

```java
public static void main(String[] args) {
    String str = " A B C D ";
    System.out.println(str.strip());   //去除首尾空格
    System.out.println(str.stripLeading());  //去除首部空格
    System.out.println(str.stripTrailing());   //去除尾部空格
}
```