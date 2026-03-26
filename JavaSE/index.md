# Java SE 核心知识体系笔记目录

## 一、 面向对象与核心基础
* [面向对象程序设计](./面向对象程序设计.md)
  * 类与对象的概念、构造方法与 this 关键字
  * 封装、继承与多态机制详解
  * 抽象类与接口的定义与使用
  * 内部类（成员、静态、局部、匿名内部类）
* [字符串](./字符串.md)
  * String 对象的多种创建方式与比较规则
  * StringBuilder 与 StringBuffer 的链式编程与应用
  * 字符串的遍历、截取与替换操作
* [异常处理](./异常处理.md)
  * 异常体系结构（Error、受检 Exception 与非受检 Exception）
  * 异常处理核心关键字（try, catch, finally, throw, throws）

## 二、 常用 API 与集合框架
* [常用的类API](./常用的类API.md)
  * 基础工具类（Math, System, Object 顶级父类）
  * 传统时间类操作（Date, SimpleDateFormat, Calendar）
  * 八大基本数据类型的包装类机制
* [集合Collection](./集合Collection.md)
  * 单列集合体系（ArrayList, HashSet, LinkedHashSet, TreeSet 排序规则）
  * 双列集合体系（Map 接口核心 API, HashMap）
  * 集合的通用遍历方式（迭代器, 增强for, Lambda 表达式）
* [StreamAPI与Lambda表达式](./StreamAPI与Lambda表达式.md)
  * Lambda 表达式语法规则与函数式编程思想
  * Stream 流的创建与中间操作（filter, map, sorted 等）
  * Stream 终端操作（查找匹配、聚合统计、归约与 collect 收集）

## 三、 高级特性与底层机制
* [反射、泛型、注解](./反射、泛型、注解.md)
  * 泛型机制（泛型类/接口/方法，上下界与无界通配符）
  * 反射原理（获取 Class 对象，解剖与运行构造/变量/方法）
  * 注解的使用（内置注解，四大元注解，自定义注解结构）

## 四、 IO 流与文件操作
* [File类](./File类.md)
  * File 对象的创建与路径表示方法
  * 文件与目录的判断、获取、创建与删除操作
* [IO流](./IO流.md)
  * IO 流的分类与体系结构
  * 核心字节流操作（FileInputStream, FileOutputStream 及文件拷贝）
  * 字符流、数据流与对象流（序列化）的使用

## 五、 并发编程
* [多线程](./多线程.md)
  * 线程与进程概念，并发与并行的区别
  * 多线程的三种实现方式（Thread, Runnable, Callable & Future）
  * 线程安全与锁机制（synchronized 同步代码块/方法, Lock 接口）
  * 线程通信机制（等待-唤醒机制、阻塞队列 BlockingQueue）
  * 线程池原理与自定义配置（ThreadPoolExecutor 七大参数解析）