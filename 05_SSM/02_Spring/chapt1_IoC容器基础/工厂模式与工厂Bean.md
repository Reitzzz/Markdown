# Spring 工厂模式与工厂 Bean 学习笔记

在默认情况下，Spring IoC 容器会调用 Bean 对应类型的构造方法来进行对象的创建。但在某些场景下（如工厂模式），我们希望 Spring 不要直接利用反射机制调用构造方法，而是通过工厂类来生成需要的 Bean 对象。



## 1. 静态工厂方法实例化 Bean

当我们拥有一个提供静态方法的工厂类时，可以直接在配置文件中指定该工厂方法来创建 Bean。

### 代码示例
```java
public class Student {
    Student() {
        System.out.println("我被构造了");
    }
}

public class StudentFactory {
    // 静态工厂方法
    public static Student getStudent(){
        System.out.println("欢迎光临电子厂");
        return new Student();
    }
}
```

### XML 配置
使用 `factory-method` 属性指定静态工厂方法：
```xml
<bean class="com.test.bean.StudentFactory" factory-method="getStudent"/>
```

**⚠️ 核心注意点：**
1. **注册的真实类型**：虽然 `class` 属性填写的是 `StudentFactory`，但这只是为了告诉 Spring 工厂方法的位置。真正注册到容器中的是**工厂方法的返回类型**（即 `Student` 对象）。
2. **局限性**：采用这种静态工厂模式后，无法再通过配置文件对该 Bean 进行依赖注入操作，所有的初始化工作只能在工厂方法内部完成。

---

## 2. 实例工厂方法实例化 Bean (工厂 Bean)

如果工厂方法不是静态的，意味着需要先创建工厂类的对象，然后才能调用工厂方法获取目标 Bean。这种情况下，我们需要将工厂类本身注册为一个 Bean（即工厂 Bean）。

### 代码示例
```java
public class StudentFactory {
    // 实例工厂方法（非静态）
    public Student getStudent(){
        System.out.println("欢迎光临电子厂");
        return new Student();
    }
}
```

### XML 配置
1. 首先，将工厂类注册为一个普通的 Bean。
2. 然后，使用 `factory-bean` 指定工厂 Bean 的名称，并使用 `factory-method` 指定实例化方法。

```xml
<bean name="studentFactory" class="com.test.bean.StudentFactory"/>

<bean factory-bean="studentFactory" factory-method="getStudent"/>
```

**💡 优势**：由于工厂类本身被注册为了一个 Bean，此时我们就可以在 XML 配置文件中为这个工厂 Bean 进行依赖注入等配置了。

---

## 3. 细节：获取工厂 Bean 与目标 Bean 的实例

在 Spring 容器中，针对工厂 Bean 提供了一个特殊的名称前缀 `&`，用于区分获取的是“工厂生产的对象”还是“工厂对象本身”。

* **获取目标 Bean**：直接输入工厂 Bean 的名称，获取到的是工厂方法**生产出来的目标 Bean**实例。
  ```java
  Student bean = (Student) context.getBean("studentFactory");
  ```

* **获取工厂 Bean 本身**：在名称前面添加 `&` 符号，获取到的是**工厂类自身**的实例。
  ```java
  StudentFactory bean = (StudentFactory) context.getBean("&studentFactory");
  ```