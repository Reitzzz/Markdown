## 1.2 使用接口实现AOP

前面我们介绍了如何使用xml配置一个AOP操作，这节课我们来看看如何使用Advice实现AOP。

它与我们之前学习的动态代理更接近一些，比如在方法开始执行之前或是执行之后会去调用我们实现的接口，首先我们需要将一个类实现Advice接口，只有实现此接口，才可以被通知，比如我们这里使用`MethodBeforeAdvice`表示是一个在方法执行之前的动作：

```java
public class StudentAOP implements MethodBeforeAdvice {
    @Override
    public void before(Method method, Object[] args, Object target) throws Throwable {
        System.out.println("通过Advice实现AOP");
    }
}
```

我们发现，方法中包括了很多的参数，其中args代表的是方法执行前得到的实参列表，还有target表示执行此方法的实例对象。运行之后，效果和之前是一样的，但是在这里我们就可以快速获取到更多信息。还是以简单的study方法为例：

```java
public class Student {
    public void study(){
        System.out.println("我是学习方法！");
    }
}
```

```xml
<bean id="student" class="org.example.entity.Student"/>
<bean id="studentAOP" class="org.example.entity.StudentAOP"/>
<aop:config>
    <aop:pointcut id="test" expression="execution(* org.example.entity.Student.study())"/>
    <aop:advisor advice-ref="studentAOP" pointcut-ref="test"/>
</aop:config>
```

我们来测试一下吧：

![image-20221216164110367](https://s2.loli.net/2022/12/16/ofducpb2mLh9XHi.png)

除了此接口以外，还有其他的接口，比如`AfterReturningAdvice`就需要实现一个方法执行之后的操作：

```java
public class StudentAOP implements MethodBeforeAdvice, AfterReturningAdvice {
    @Override
    public void before(Method method, Object[] args, Object target) throws Throwable {
        System.out.println("通过Advice实现AOP");
    }

    @Override
    public void afterReturning(Object returnValue, Method method, Object[] args, Object target) throws Throwable {
        System.out.println("我是方法执行之后的结果，方法返回值为："+returnValue);
    }
}
```

因为使用的是接口，就非常方便，直接写一起，配置文件都不需要改了：

![image-20221216164242506](https://s2.loli.net/2022/12/16/DUZzqaBSiJKNv8j.png)

我们也可以使用MethodInterceptor（同样也是Advice的子接口）进行更加环绕那样的自定义的增强，它用起来就真的像代理一样，例子如下：

```java
public class Student {
    public String study(){
        System.out.println("我是学习方法！");
        return "lbwnb";
    }
}
```

```java
public class StudentAOP implements MethodInterceptor {   //实现MethodInterceptor接口
    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {  //invoke方法就是代理方法
        Object value = invocation.proceed();   //跟之前一样，需要手动proceed()才能调用原方法
        return value+"增强";
    }
}
```

我们来看看结果吧：

![image-20221216173211310](https://s2.loli.net/2022/12/16/ARcUW2mJrn7Y6f9.png)

使用起来还是挺简单的。