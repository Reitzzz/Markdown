# StreamAPI与Lambda表达式

## 目录
- [1. Lambda 表达式](#1-lambda-表达式)
  - [1.1 函数式编程](#11-函数式编程)
  - [1.2 格式](#12-格式)
  - [1.3 注意点](#13-注意点)
- [2. StreamAPI - 创建流](#2-streamapi---创建流)
  - [2.1 细节](#21-细节)
  - [2.2 创建操作](#22-创建操作)
- [3. StreamAPI - 中间操作](#3-streamapi---中间操作)
  - [3.1 特点](#31-特点)
  - [3.2 操作类型](#32-操作类型)
- [4. StreamAPI - 终端操作](#4-streamapi---终端操作)
  - [4.1 查找与匹配](#41-查找与匹配)
  - [4.2 聚合操作](#42-聚合操作)
  - [4.3 归约操作](#43-归约操作)
  - [4.4 收集操作](#44-收集操作)

## 1. Lambda 表达式

### 1.1 函数式编程
忽略面向对象的复杂语法，强调做什么，而不是谁去做。

### 1.2 格式
- **一般写法**：`(参数) -> { 方法体 }`  
  - `( )` 对应方法的形参；  
  - `->` 固定格式；  
  - `{ }` 对应方法体。

- **省略写法**（可推导即可省略）：  
  1. 参数类型可以省略。  
  2. 只有一个参数时，`()` 可省略。  
  3. 方法体只有一行时，大括号、分号、`return` 可同时省略。

### 1.3 注意点
- 只能用于简化**函数式接口**的匿名内部类写法。  
- 函数式接口：有且仅有一个抽象方法的接口（可用 `@FunctionalInterface` 验证）。

## 2. StreamAPI - 创建流

### 2.1 细节
Stream 本身不是数据结构，数据仍存储在原来的数据结构中。可从集合、数组、文件、IO 通道等多种数据源创建。

### 2.2 创建操作

- **从集合创建**  
  方法声明：`default Stream<E> stream()`
  ```java
  Stream<String> stream = list.stream();
  ```

- **从数组创建**  
  方法声明：`public static <T> Stream<T> stream(T[] array)`
  ```java
  Stream<String> stream1 = Arrays.stream(array);
  ```

- **从离散元素创建**  
  方法声明：`public static<T> Stream<T> of(T... values)`
  ```java
  Stream<String> stream2 = Stream.of("a", "b", "c");
  ```

- **通过构建器**  
  ```java
  Stream.Builder<String> builder = Stream.builder();
  builder.add("a").add("b").add("c");
  Stream<String> stream = builder.build();
  ```

- **从文件读取**  
  ```java
  Path path = Paths.get("file.txt");
  Stream<String> stream = Files.lines(path);
  ```

- **数值特化流（直接指定）**  
  ```java
  IntStream intStream = IntStream.of(1, 2, 3);
  ```

- **数值特化流（区间范围，不包含结束值）**  
  ```java
  IntStream rangeStream = IntStream.range(1, 4);
  ```

- **数值特化流（区间范围，包含结束值）**  
  ```java
  IntStream rangeClosedStream = IntStream.rangeClosed(1, 4);
  ```

- **数值特化流（随机数）**  
  ```java
  IntStream randomStream = new Random().ints(5);
  ```

- **动态无限流（生成器）**  
  ```java
  Stream<String> stringStream = Stream.generate(() -> "Albert").limit(5);
  Stream<Double> doubleStream = Stream.generate(Math::random).limit(5);
  ```

- **动态无限流（迭代器）**  
  ```java
  Stream<Integer> iterateStream = Stream.iterate(0, n -> n + 2).limit(5);
  Stream<Integer> limitedStream = Stream.iterate(0, n -> n <= 10, n -> n + 2).limit(10);
  ```

- **并行流（集合直接获取）**  
  ```java
  List<String> stringList = List.of("a", "b", "c", "d", "e");
  Stream<String> stringStream = stringList.parallelStream();
  ```

- **并行流（状态转换）**  
  ```java
  Stream<String> stream = Stream.of("a", "b", "c");
  Stream<String> parallelStream = stream.parallel();
  ```

## 3. StreamAPI - 中间操作

### 3.1 特点
对流进行处理（筛选、排序、映射等），每次操作返回新流，支持链式调用。

### 3.2 操作类型

- **条件过滤 `filter`**  
  方法声明：`Stream<T> filter(Predicate<? super T> predicate)`  
  保留断言结果为 `true` 的元素。

- **数据去重 `distinct`**  
  方法声明：`Stream<T> distinct()`  
  基于 `hashCode()` 和 `equals()` 去重（自定义类需重写）。

- **截断元素 `limit`**  
  方法声明：`Stream<T> limit(long maxSize)`  
  仅保留前 `maxSize` 个元素。

- **跳过元素 `skip`**  
  方法声明：`Stream<T> skip(long n)`  
  丢弃前 `n` 个元素，不足则返回空流。

- **单层映射 `map`**  
  方法声明：`<R> Stream<R> map(Function<? super T, ? extends R> mapper)`  
  将元素转换成新元素构成新流。

- **扁平化映射 `flatMap` 与拆箱映射 `mapToInt`**  
  方法声明：`<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper)`  
  将多层流合并为单层流。

- **自然排序 `sorted()`**  
  方法声明：`Stream<T> sorted()`  
  要求元素实现 `Comparable` 接口。

- **自定义排序 `sorted(Comparator)`**  
  方法声明：`Stream<T> sorted(Comparator<? super T> comparator)`  
  传入自定义比较器。

## 4. StreamAPI - 终端操作

### 4.1 查找与匹配

- **任意匹配 `anyMatch`**  
  方法声明：`boolean anyMatch(Predicate<? super T> predicate)`  
  ```java
  boolean anyMatch = people.stream().anyMatch(person -> person.getAge() >= 18);
  ```

- **全不匹配 `noneMatch`**  
  方法声明：`boolean noneMatch(Predicate<? super T> predicate)`  
  ```java
  boolean noneMatch = people.stream().noneMatch(person -> person.getAge() < 18);
  ```

- **全量匹配 `allMatch`**  
  方法声明：`boolean allMatch(Predicate<? super T> predicate)`  
  ```java
  boolean allMatch = people.stream().allMatch(person -> person.getAge() >= 18);
  ```

- **查找首个 `findFirst`**  
  方法声明：`Optional<T> findFirst()`  
  ```java
  Optional<Person> first = people.stream().findFirst();
  first.ifPresent(System.out::println);
  ```

- **随机查找 `findAny`**  
  方法声明：`Optional<T> findAny()`  
  ```java
  Optional<Person> any = people.stream().findAny();
  any.ifPresent(System.out::println);
  ```

### 4.2 聚合操作

- **统计个数 `count`**  
  方法声明：`long count()`
  ```java
  long count = people.stream().count();
  ```

- **最大值 `max` / 最小值 `min`**  
  方法声明：`Optional<T> max(Comparator<? super T> comparator)`  
  ```java
  Optional<Person> max = people.stream().max(Comparator.comparingInt(Person::getAge));
  Optional<Person> min = people.stream().min(Comparator.comparingInt(Person::getAge));
  ```

### 4.3 归约操作

方法声明：`T reduce(T identity, BinaryOperator<T> accumulator)`
```java
String joinedName = people.stream()
    .map(Person::getName)
    .reduce("", (a, b) -> a + b);
```

### 4.4 收集操作

方法声明：`<R, A> R collect(Collector<? super T, A, R> collector)`
```java
Map<String, List<Person>> groupedByCountry = people.stream()
    .collect(Collectors.groupingBy(Person::getCountry));

String nameString = people.stream()
    .map(Person::getName)
    .collect(Collectors.joining(", "));

IntSummaryStatistics stats = people.stream()
    .collect(Collectors.summarizingInt(Person::getAge));
```