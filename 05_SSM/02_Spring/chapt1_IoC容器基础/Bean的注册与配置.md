# Bean注册与配置

前面我们通过一个简单例子体验了一下如何使用 Spring 来管理我们的对象，并向 IoC 容器索要被管理的对象。这部分我们就来详细了解一下如何向 Spring 注册 Bean 以及 Bean 的相关配置。

## 1. 配置文件的导入

实际上我们的配置文件可以有很多个，并且这些配置文件是可以相互导入的。为了简单起见，我们通常从单配置文件开始讲起，但如果需要拆分配置，可以使用 `import` 标签：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans ...>
    <import resource="test.xml"/>    //类似于嵌套与合并
</beans>
```

## 2. Bean的基本注册与按类型获取

要配置一个 Bean，只需要添加 `<bean/>` 标签。但是如果不指定具体的类，Spring 无法得知我们要配置的 Bean 到底是哪一个，所以必须指定对应的 `class` 属性：

```xml
<bean class="com.test.bean.Student"/>
```

注册成功后，我们就可以根据类型向容器索要 Bean 实例对象了：

```java
public static void main(String[] args) {
    ApplicationContext context = new ClassPathXmlApplicationContext("test.xml");
    // getBean有多种形式，其中第一种就是根据类型获取对应的Bean
    // 容器中只要注册了对应类的Bean或是对应类型子类的Bean，都可以获取到
    Student student = context.getBean(Student.class);
    student.hello();
}
```

## 3. Bean的歧义性与按名称获取

在有些时候，按类型获取 Bean 可能会出现歧义。比如我们分别注册两个子类的 Bean：

```java
public class ArtStudent extends Student {
    public void art() {
        System.out.println("我爱画画");
    }
}

public class SportStudent extends Student {
    public void sport() {
        System.out.println("我爱运动");
    }
}
```

```xml
<bean class="com.test.bean.ArtStudent"/>
<bean class="com.test.bean.SportStudent"/>
```

此时如果我们在获取 Bean 时索要的是它们的父类 `Student.class`，会得到一个 **Bean定义不唯一异常**。因为容器发现有两个满足条件的子类 Bean，不知道该返回哪一个。

**解决方式一：精确指定子类类型**

```java
ArtStudent student = context.getBean(ArtStudent.class);
student.art();
```

**解决方式二：为 Bean 指定名称**

如果连两个 Bean 的类型都是完全一样的，那就无法使用 `Class` 来进行区分了：

```xml
<bean class="com.test.bean.Student"/>
<bean class="com.test.bean.Student"/>
```

这种情况下，我们可以为 Bean 指定一个 `name` 属性（或 `id` 属性，功能相同，但 `id` 会检查命名规范）用于区分。不同的 Bean 名字不能相同：

```xml
<bean name="art" class="com.test.bean.ArtStudent"/>
<bean name="sport" class="com.test.bean.SportStudent"/>
<bean name="a" class="com.test.bean.Student"/>
<bean name="b" class="com.test.bean.Student"/>
```

这样，我们就可以通过名称准确区分并获取指定的 Bean：

```java
Student student = (Student) context.getBean("a");
student.hello();
```

除了指定 `name`，我们还可以为 Bean 起别名（相当于小名）：

```xml
<bean name="a" class="com.test.bean.Student"/>
<alias name="a" alias="test"/>      //起别名为test
```

使用别名同样可以拿到对应的 Bean：

```java
Student student = (Student) context.getBean("test");
student.hello();
```

## 4. Bean的作用域 (Scope)

IoC 容器创建的 Bean 是只有一个，还是每次索要的时候都会给我们一个新的对象？我们可以在主方法中连续获取两次：

```java
Student student1 = context.getBean(Student.class);
Student student2 = context.getBean(Student.class);
System.out.println(student1 == student2);   // 默认为单例模式，对象始终为同一个，输出 true
```

默认情况下，通过 IoC 容器进行管理的 Bean 都是**单例模式 (singleton)** 的，这个对象只会被创建一次。只要容器没有被销毁，这个对象将一直存在并被复用。

如果我们希望每次拿到的对象都是一个新的，可以修改其作用域为**原型模式/多例模式 (prototype)**：
```xml
<bean name="student" class="org.example.springbootdemo.Entity.Student" scope="prototype" />
```

```java
Student student1 = context.getBean(Student.class);  // 原型模式下，对象不再始终是同一个了
Student student2 = context.getBean(Student.class);
System.out.println(student1 == student2);           // 输出 false
```
* **singleton (单例):** 容器加载配置时创建，一直保存在容器中。
* **prototype (原型):** 容器不保存实例，只有在每次调用 `getBean` 时才会直接 `new` 一个新对象。

## 5. 懒加载 (Lazy Initialization)

当 Bean 的作用域为单例模式时，默认在容器一开始加载配置时就被创建了。如果我们希望它延迟加载，等到真正第一次使用（如第一次调用 `getBean`）时再创建，可以开启懒加载：

```xml
<bean class="com.test.bean.Student" lazy-init="true"/>
```
开启后，该 Bean 依然会被容器存储为单例，但创建时机被延后了。

## 6. 加载顺序依赖 (depends-on)

单例模式下 Bean 是由 IoC 容器自动加载的，但默认加载顺序并不确定。如果我们需要维护特定的 Bean 加载顺序（例如某个 Bean 必须要在另一个 Bean 初始化之前创建），可以使用 `depends-on` 属性来设定前置依赖加载。

比如 `Teacher` 应该在 `Student` 之前加载：

```xml
<bean name="teacher" class="com.test.bean.Teacher"/>
<bean name="student" class="com.test.bean.Student" depends-on="teacher"/>
```
这样就可以严格保证 Bean 的加载顺序了。