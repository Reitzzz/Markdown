# 接口中的private方法

在Java 8中，接口中 的方法支持添加`default`关键字来添加默认实现：

```java
public interface Test {
    default void test(){
        System.out.println("我是test方法默认实现");
    }
}
```

而在Java 9中，接口再次得到强化，现在接口中可以存在私有方法了：

```java
public interface Test {
    default void test(){
        System.out.println("我是test方法默认实现");
        this.inner();   //接口中方法的默认实现可以直接调用接口中的私有方法
    }
    
    private void inner(){   //声明一个私有方法
        System.out.println("我是接口中的私有方法！");
    }
}
```

注意私有方法必须要提供方法体，因为权限为私有的，**也只有这里能进行方法的具体实现了，并且此方法只能被接口中的其他私有方法或是默认实现调用**。
