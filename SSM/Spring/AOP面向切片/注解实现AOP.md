# 1. 使用注解实现AOP

## 1.1 开启AOP注解支持与Bean注册

接着我们来看看如何使用注解实现AOP操作，现在变回我们之前的注解开发，首先我们需要在主类添加`@EnableAspectJAutoProxy`注解，开启AOP注解支持：

```java
@EnableAspectJAutoProxy
@ComponentScan("org.example.entity")
@Configuration
public class MainConfiguration {
}
```

还是熟悉的玩法，类上直接添加`@Component`快速注册Bean：

```java
@Component
public class Student {
    public void study(){
        System.out.println("我是学习方法！");
    }
}
```

接着我们需要在定义AOP增强操作的类上添加`@Aspect`注解和`@Component`将其注册为Bean即可，就像我们之前在配置文件中也要将其注册为Bean那样：

```java
@Aspect
@Component
public class StudentAOP {

}
```

## 1.2 前置通知 (@Before)

接着，我们可以在定义AOP增强操作的类里编写增强方法，并将此方法添加到一个切点中，比如我们希望在Student的study方法执行之前执行我们的`before`方法：

```java
public void before(){
    System.out.println("我是之前执行的内容！");
}
```

那么只需要添加@Before注解即可：

```java
@Before("execution(* org.example.entity.Student.study())")  //execution写法跟之前一样
public void before(){
    System.out.println("我是之前执行的内容！");
}
```

这样，这个方法就会在指定方法执行之前执行了，是不是感觉比XML配置方便多了。我们来测试一下：

```java
public static void main(String[] args) {
    ApplicationContext context = new AnnotationConfigApplicationContext(MainConfiguration.class);
    Student bean = context.getBean(Student.class);
    bean.study();
}
```

![image-20221216165625372](https://s2.loli.net/2022/12/16/KpiXcdNt7BglYQh.png)

## 1.3 获取切入点信息 (JoinPoint)

同样的，我们可以为其添加`JoinPoint`参数来获取切入点信息，使用方法跟之前一样：

```java
@Before("execution(* org.example.entity.Student.study())")
public void before(JoinPoint point){
    System.out.println("参数列表："+ Arrays.toString(point.getArgs()));
    System.out.println("我是之前执行的内容！");
}
```

## 1.4 命名绑定模式获取参数

为了更方便，我们还可以直接将参数放入，比如：

```java
public void study(String str){
    System.out.println("我是学习方法！");
}
```

使用命名绑定模式，可以快速得到原方法的参数：

```java
@Before(value = "execution(* org.example.entity.Student.study(..)) && args(str)", argNames = "str")
//命名绑定模式就是根据下面的方法参数列表进行匹配
//这里args指明参数，注意需要跟原方法保持一致，然后在argNames中指明
public void before(String str){
    System.out.println(str);   //可以快速得到传入的参数
    System.out.println("我是之前执行的内容！");
}
```

## 1.5 其他通知注解 (@AfterReturning 等)

除了@Before，还有很多可以直接使用的注解，比如@AfterReturning、@AfterThrowing等，比如@AfterReturning：

```java
public String study(){
    System.out.println("我是学习方法！");
    return "lbwnb";
}
```

```java
@AfterReturning(value = "execution(* org.example.entity.Student.study())", argNames = "returnVal", returning = "returnVal")   //使用returning指定接收方法返回值的参数returnVal
public void afterReturn(Object returnVal){
    System.out.println("返回值是："+returnVal);
}
```

## 1.6 环绕通知 (@Around)

同样的，环绕也可以直接通过注解声明：

```java
@Around("execution(* com.test.bean.Student.test(..))")
public Object around(ProceedingJoinPoint point) throws Throwable {
    System.out.println("方法执行之前！");
    Object val = point.proceed();
    System.out.println("方法执行之后！");
    return val;
}
```

## 1.7 总结

实际上，无论是使用注解或是XML配置，我们要做的流程都是一样的，在之后的学习中，我们还会遇到更多需要使用AOP的地方。