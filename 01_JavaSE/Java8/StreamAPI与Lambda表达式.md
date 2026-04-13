# StreamAPI与Lambda表达式

## 目录
- [StreamAPI与Lambda表达式](#streamapi与lambda表达式)
  - [目录](#目录)
  - [Lambda](#lambda)
    - [函数式编程](#函数式编程)
    - [格式](#格式)
    - [注意点](#注意点)
  - [StreamAPI - 创建流](#streamapi---创建流)
    - [细节](#细节)
    - [操作](#操作)
  - [StreamAPI - 中间操作](#streamapi---中间操作)
    - [特点](#特点)
    - [类型](#类型)
  - [StreamAPI - 终端操作](#streamapi---终端操作)
    - [查找与匹配](#查找与匹配)
    - [聚合操作](#聚合操作)
    - [归约操作](#归约操作)
    - [收集操作](#收集操作)

## Lambda

### 函数式编程
忽略面向对象的复杂语法，强调做什么，而不是谁去做。

### 格式
- 一般写法：( ) -> { }
( ) 对应着方法的形参；-> 固定格式；{ } 对应着方法的方法体。

- 省略写法
核心思想: 可推导→可省略
规则：
1. 参数类型可以省略不写。
2. 如果只有一个参数，() 也可以省略。
3. 如果方法体只有一行，大括号，分号，return 可以省略不写，但需要同时省略。

### 注意点
只能用于简化函数式接口的匿名内部类的写法。
函数式接口: 有且仅有一个抽象方法的接口。
验证: 加@FunctionalInterface 看是否报错。

## StreamAPI - 创建流

### 细节
Stream本身不是数据结构，数据依然存储在原来的数据结构。
可以从多种数据类型创建，包括集合 数组 文件 IO通道等。

### 操作

- 从集合创建
方法声明：default Stream<E> stream()
```java
Stream<String> stream = list.stream();
```

- 从数组创建
方法声明：public static <T> Stream<T> stream(T[] array)
```java
Stream<String> stream1 = Arrays.stream(array);
```

- 从离散元素创建
方法声明：public static<T> Stream<T> of(T... values)
```java
Stream<String> stream2 = Stream.of("a", "b", "c");
```

- 通过构建器
```java
Stream.Builder<String> builder = Stream.builder();
builder.add("a");
builder.add("b");
builder.add("c");
Stream<String> stream = builder.build();
```

- 从文件读取
```java
Path path = Paths.get("file.txt");
Stream<String> stream = Files.lines(path);
```

- 数制特化流(基本类型)：直接指定
```java
IntStream intStream = IntStream.of(1, 2, 3);
```

- 数制特化流：通过区间范围（不包含结束边界值）
```java
IntStream rangeStream = IntStream.range(1, 4);
```

- 数制特化流：通过区间范围（包含结束边界值）
```java
IntStream rangeClosedStream = IntStream.rangeClosed(1, 4);
```

- 数制特化流：通过随机数
```java
IntStream randomStream = new Random().ints(5);
```

- 动态无限流创建：生成器模式
```java
Stream<String> stringStream = Stream.generate(() -> "Albert").limit(5);
Stream<Double> doubleStream = Stream.generate(Math::random).limit(5);
```

- 动态无限流创建：迭代器模式
```java
Stream<Integer> iterateStream = Stream.iterate(0, n -> n + 2).limit(5);
Stream<Integer> limitedStream = Stream.iterate(0, n -> n <= 10, n -> n + 2).limit(10);
```

- 并行流创建：集合直接获取
```java
List<String> stringList = List.of("a", "b", "c", "d", "e");
Stream<String> stringStream = stringList.parallelStream();
```

- 并行流创建：状态转换（将已有的普通顺序流转换为并行流）
```java
Stream<String> stream = Stream.of("a", "b", "c");
Stream<String> parallelStream = stream.parallel();
```

## StreamAPI - 中间操作

### 特点
用于对流进行处理，包括筛选 排序 映射等。
每次操作都会返回新的流 从而支持链式调用。

### 类型

- 条件过滤 filter：接收一个 Predicate 断言型函数式接口，保留运行结果为 true 的元素，拦截不符合要求的元素。
方法声明：Stream<T> filter(Predicate<? super T> predicate)

- 数据去重 distinct：筛选，通过流所生成元素的 hashCode() 和 equals() 方法去除重复元素。自定义类的对象必须先重写方法。
方法声明：Stream<T> distinct()

- 截断元素 limit：限制流的最大长度，仅截取排在最前面的 maxSize 个元素。
方法声明：Stream<T> limit(long maxSize)

- 跳过元素 skip：返回一个扔掉了前 n 个元素的流。若流中元素不足 n 个，则返回一个空流。
方法声明：Stream<T> skip(long n)

- 单层映射 map：假如我们要把包含人员对象的流转换成只包含人员姓名的流，可以使用map方法：将流中的每个元素转换成新的元素 最后生成一个新元素构成的流。
方法声明：<R> Stream<R> map(Function<? super T, ? extends R> mapper)

- 扁平化映射 flatmap 和拆箱映射 mapToInt等：对于嵌套的集合 数组或其他多层结构的数据 map处理不够灵活 需要使用flatMap 将多层流合并为单层流。
方法声明：<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper)

- 自然排序 sorted(无参)：当流中的元素类型实现comparable接口时 （比如字符串或包装类型的数字 Integer Long等）可以直接调用无参数的sorted方法 按照自然顺序排序。
方法声明：Stream<T> sorted()

- 自定义排序 sorted(有参)：未实现comparable接口则使用比较器Comparator。
方法声明：Stream<T> sorted(Comparator<? super T> comparator)

## StreamAPI - 终端操作

### 查找与匹配

- 任意匹配 (anyMatch)：任意元素满足条件就返回true。
方法声明：boolean anyMatch(Predicate<? super T> predicate)
```java
boolean anyMatch = people.stream().anyMatch(person -> person.getAge() >= 18);
```

- 全不匹配 (noneMatch)：没有元素满足条件才返回true。
方法声明：boolean noneMatch(Predicate<? super T> predicate)
```java
boolean noneMatch = people.stream().noneMatch(person -> person.getAge() < 18);
```

- 全量匹配 (allMatch)：所有元素都满足条件才返回true。
方法声明：boolean allMatch(Predicate<? super T> predicate)
```java
boolean allMatch = people.stream().allMatch(person -> person.getAge() >= 18);
```

- 查找首个 (findFirst)：查询第一个元素 顺序流下通常就是源数据的第一个。
方法声明：Optional<T> findFirst()
```java
Optional<Person> first = people.stream().findFirst();
first.ifPresent(System.out::println);
```

- 随机查找 (findAny)：并行流下可能返回任意元素 顺序流通常返回第一个。
方法声明：Optional<T> findAny()
```java
Optional<Person> any = people.stream().findAny();
any.ifPresent(System.out::println);
```

### 聚合操作

- 统计个数 (count)
方法声明：long count()
```java
long count = people.stream().count();
```

- 求最大值 (max) / 求最小值 (min)
方法声明：Optional<T> max(Comparator<? super T> comparator)
方法声明：Optional<T> min(Comparator<? super T> comparator)
```java
Optional<Person> max = people.stream().max(Comparator.comparingInt(Person::getAge));
Optional<Person> min = people.stream().min(Comparator.comparingInt(Person::getAge));
```

### 归约操作

方法声明：T reduce(T identity, BinaryOperator<T> accumulator)
```java
String joinedName = people.stream()
    .map(Person::getName)
    .reduce("", (a, b) -> a + b);
```

### 收集操作

方法声明：<R, A> R collect(Collector<? super T, A, R> collector)
```java
Map<String, List<Person>> groupedByCountry = people.stream().collect(Collectors.groupingBy(Person::getCountry));

String nameString = people.stream().map(Person::getName).collect(Collectors.joining(", "));

IntSummaryStatistics stats = people.stream().collect(Collectors.summarizingInt(Person::getAge));
```