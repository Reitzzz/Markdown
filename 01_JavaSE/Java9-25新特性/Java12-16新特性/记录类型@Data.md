# 记录类型

继类、接口、枚举、注解之后的又一新类型来了，它的名字叫"记录"，在Java 14中首次出场，这一出场，Lombok的噩梦来了。

在实际开发中，很多的类仅仅只是充当一个实体类罢了，保存的是一些不可变数据，比如我们从数据库中查询的账户信息，最后会被映射为一个实体类：

```java
@Data
public class Account {   //使用Lombok，一个注解就搞定了
    String username;
    String password;
}
```

Lombok可以说是简化代码的神器了，他能在编译时自动生成getter和setter、构造方法、toString()方法等实现，在编写这些实体类时，简直不要太好用，而这一波，官方也是看不下去了，于是自己也搞了一个记录类型。

记录类型本质上也是一个普通的类，不过是final类型且继承自java.lang.Record抽象类的，它会在编译时，会自动编译出 `public get` `hashcode` 、`equals`、`toString` 等方法，好家伙，这是要逼死Lombok啊。

```java
public record Account(String username, String password) {  //直接把字段写在括号中

}
```

使用起来也是非常方便，自动生成了构造方法和成员字段的公共get方法：

![image-20230306180856852](https://s2.loli.net/2023/03/06/iopGnBQf6v3VS7Y.png)

并且toString也是被重写了的：

![image-20230306180905242](https://s2.loli.net/2023/03/06/cvOXQklZybzHPfs.png)

`equals()`方法仅做成员字段之间的值比较，也是帮助我们实现好了的：

```java
Account account0 = new Account("Admin", "123456");
Account account1 = new Account("Admin", "123456");   //两个属性都是一模一样的
System.out.println(account0.equals(account1));  //得到true
```

是不是感觉这种类型就是专门为这种实体类而生的。

```java
public record Account(String username, String password) implements Runnable {  //支持实现接口，但是不支持继承，因为继承的坑位已经默认被占了

    @Override
    public void run() {
        
    }
}
```

**注意：** 记录类型在Java 16才正式开放使用，所以我们项目的代码级别需要调整到16以上。