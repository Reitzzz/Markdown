# 1. 新的switch语法

- [1. 新的switch语法](#1-新的switch语法)
  - [1.1 需求背景与传统写法](#11-需求背景与传统写法)
  - [1.2 增强版 switch 语法](#12-增强版-switch-语法)
  - [1.3 switch 表达式详细规则](#13-switch-表达式详细规则)
  - [1.4 yield 关键字的使用](#14-yield-关键字的使用)

---

## 1.1 需求背景与传统写法

在Java 12引入全新的switch语法，让我们使用switch语句更加的灵活，比如我们想要编写一个根据成绩得到等级的方法：

```java
/**
 * 传入分数（范围 0 - 100）返回对应的等级：
 *      100-90：优秀
 *      70-80：良好
 *      60-70：及格
 *      0-60：寄
 * @param score 分数
 * @return 等级
 */
public static String grade(int score){
    
}
```

现在我们想要使用switch来实现这个功能（不会吧不会吧，不会有人要想半天怎么用switch实现吧），之前的写法是：

```java
public static String grade(int score){
    score /= 10;  //既然分数段都是整数，那就直接整除10
    String res = null;
    switch (score) {
        case 10:
        case 9:
            res =  "优秀";   //不同的分数段就可以返回不同的等级了
                break;   //别忘了break，不然会贯穿到后面
        case 8:
        case 7:
            res = "良好";
                break;
        case 6:
            res = "及格";
                break;
        default:
            res = "不及格";
                break;
    }
    return res;
}
```

## 1.2 增强版 switch 语法

但是现在我们可以使用新的特性了：

```java
public static String grade(int score){
    score /= 10;  //既然分数段都是整数，那就直接整除10
    return switch (score) {   //增强版switch语法
        case 10, 9 -> "优秀";   //不需要我们自己考虑break或是return来结束switch了
        case 8, 7 -> "良好"; 
        case 6 -> "及格";
        default -> "不及格";
    };
}
```

不过最后编译出来的样子，貌似还是和之前是一样的：

![image-20230306180750556](https://s2.loli.net/2023/03/06/ZcAmGyCQrD4uSMR.png)

## 1.3 switch 表达式详细规则

这种全新的switch语法称为`switch表达式`，它的意义不仅仅体现在语法的精简上，我们来看看它的详细规则：

```java
var res = switch (obj) {   //这里和之前的switch语句是一样的，但是注意这样的switch是有返回值的，所以可以被变量接收
    case [匹配值, ...] -> "优秀";   //case后直接添加匹配值，匹配值可以存在多个，需要使用逗号隔开，使用 -> 来返回如果匹配此case语句的结果
    case ...   //根据不同的分支，可以存在多个case
    default -> "不及格";   //注意，表达式要求必须涵盖所有的可能，所以是需要添加default的
};
```

## 1.4 yield 关键字的使用

那么如果我们并不是能够马上返回，而是需要做点什么其他的工作才能返回结果呢？

```java
var res = switch (obj) {   //增强版switch语法
    case [匹配值, ...] -> "优秀";
    default -> {   //我们可以使用花括号来将整套逻辑括起来
        //... 我是其他要做的事情
        yield  "不及格";  //注意处理完成后需要返回最终结果，但是这样并不是使用return，而是yield关键字
    }
};
```

当然，也可以像这样：

```java
var res = switch (args.length) {   //增强版switch语法
    case [匹配值, ...]:
        yield "AAA";   //传统的:写法，通过yield指定返回结果，同样不需要break
    default:
            System.out.println("默认情况");
        yield "BBB";
};
```

这种全新的语法，可以说极大地方便了我们的编码，不仅代码简短，而且语义明确。唯一遗憾的是依然不支持区间匹配。

**注意：** switch表达式在Java 14才正式开放使用，所以我们项目的代码级别需要调整到14以上。