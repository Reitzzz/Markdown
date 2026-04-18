# 集合Collection

## 目录
- [集合Collection](#集合collection)
  - [目录](#目录)
  - [1. Collection 接口](#1-collection-接口)
    - [1.1 常用方法](#11-常用方法)
  - [2. ArrayList](#2-arraylist)
    - [2.1 特点](#21-特点)
    - [2.2 与数组对比](#22-与数组对比)
    - [2.3 创建](#23-创建)
  - [3. 双列集合概述](#3-双列集合概述)
  - [4. Map 接口](#4-map-接口)
    - [4.1 常用 API](#41-常用-api)
    - [4.2 遍历方式](#42-遍历方式)
    - [4.3 HashMap 注意点](#43-hashmap-注意点)
  - [5. Set 接口](#5-set-接口)
    - [5.1 特点](#51-特点)
    - [5.2 实现类](#52-实现类)
  - [6. 遍历方式](#6-遍历方式)
    - [6.1 增强 for 循环](#61-增强-for-循环)
    - [6.2 Lambda 表达式遍历](#62-lambda-表达式遍历)
  - [7. Queue 与 Deque](#7-queue-与-deque)
    - [7.1 Queue 接口](#71-queue-接口)
    - [7.2 Deque 接口](#72-deque-接口)
    - [7.3 PriorityQueue](#73-priorityqueue)
  - [8. Collections 工具类](#8-collections-工具类)
  - [9. 集合工厂方法（Java 9+）](#9-集合工厂方法java-9)
  - [10. 有序集合增强（Java 21+）](#10-有序集合增强java-21)
  - [11. 底层原理与源码分析](#11-底层原理与源码分析)
    - [11.1 ArrayList 底层实现](#111-arraylist-底层实现)
    - [11.2 LinkedList 底层实现](#112-linkedlist-底层实现)
    - [11.3 HashMap 底层实现](#113-hashmap-底层实现)
    - [11.4 HashSet 与 TreeSet](#114-hashset-与-treeset)
    - [11.5 ConcurrentHashMap 关键机制](#115-concurrenthashmap-关键机制)

## 1. Collection 接口
单列集合的顶层接口。

### 1.1 常用方法
| 方法 | 说明 |
|------|------|
| `boolean add(E e)` | 添加元素 |
| `boolean remove(Object o)` | 删除元素 |
| `boolean isEmpty()` | 判断是否为空 |
| `int size()` | 获取元素个数 |

## 2. ArrayList

### 2.1 特点
有序、可重复、有索引。继承自 `Collection`，方法基本无差异。

### 2.2 与数组对比
| | 集合 | 数组 |
|---|------|------|
| 长度 | 可变 | 不可变 |
| 基本类型 | 需包装 | 可直接存储 |
| 引用类型 | 可存储 | 可存储 |

### 2.3 创建
基本类型需使用包装类（`char`→`Character`，`int`→`Integer`，其余首字母大写）。
```java
ArrayList<String> list = new ArrayList<>();
ArrayList<Integer> numList = new ArrayList<>();
```

## 3. 双列集合概述
键值对结构。键不可重复，值可重复，无索引。键重复则覆盖原值。

## 4. Map 接口

### 4.1 常用 API
| 方法 | 说明 |
|------|------|
| `V put(K key, V value)` | 添加/覆盖，返回被覆盖的值（无则返回 `null`） |
| `V remove(Object key)` | 删除并返回值 |
| `void clear()` | 清空 |
| `boolean containsKey(Object key)` | 是否包含键 |
| `boolean containsValue(Object value)` | 是否包含值 |
| `boolean isEmpty()` | 是否为空 |
| `int size()` | 键值对数量 |

```java
Map<String, String> m = new HashMap<>();
m.put("key1", "value1");
```

### 4.2 遍历方式

- **键找值**
  ```java
  Set<String> keys = m.keySet();
  for (String key : keys) {
      String value = m.get(key);
  }
  ```

- **键值对**
  ```java
  Set<Map.Entry<String, String>> entries = m.entrySet();
  for (Map.Entry<String, String> entry : entries) {
      String key = entry.getKey();
      String value = entry.getValue();
  }
  ```

### 4.3 HashMap 注意点
键存储自定义对象时，需重写 `hashCode()` 和 `equals()` 方法；值存储自定义对象则无需重写。

## 5. Set 接口

### 5.1 特点
无序、不重复、无索引。继承自 `Collection`，方法基本无差异。

### 5.2 实现类

- **HashSet**  
  去重场景。无序，不重复，无索引。自定义对象必须重写 `hashCode()` 和 `equals()`。  
  哈希碰撞：不同对象可能计算出相同哈希值。

- **LinkedHashSet**  
  去重且保证存取顺序。有序，不重复，无索引。

- **TreeSet**  
  排序场景。可排序，不重复，无索引。  
  两种比较方式：
  1. 自然排序：实现 `Comparable` 接口，重写 `compareTo()`。
     ```java
     public class Student implements Comparable<Student> {
         public int compareTo(Student o) {
             return this.age - o.age;
         }
     }
     ```
  2. 比较器排序：创建时传入 `Comparator`（优先级更高）。
     ```java
     TreeSet<String> ts = new TreeSet<>((o1, o2) -> o1.compareTo(o2));
     ```

## 6. 遍历方式

### 6.1 增强 for 循环
适用于单列集合和数组。变量修改不影响集合数据。
```java
for (String s : list) {
    System.out.println(s);
}
```

### 6.2 Lambda 表达式遍历
方法声明：`default void forEach(Consumer<? super T> action)`
```java
list.forEach(s -> System.out.println(s));
```

## 7. Queue 与 Deque

### 7.1 Queue 接口
先进先出（FIFO）。

| 方法 | 作用 | 异常处理 |
|------|------|----------|
| `add(e)` | 入队 | 失败抛异常 |
| `offer(e)` | 入队 | 失败返回 `false` |
| `remove()` | 出队 | 空抛异常 |
| `poll()` | 出队 | 空返回 `null` |
| `element()` | 查看队首 | 空抛异常 |
| `peek()` | 查看队首 | 空返回 `null` |

```java
Queue<String> queue = new LinkedList<>();
queue.offer("A");
System.out.println(queue.poll()); // A
```

### 7.2 Deque 接口
双端队列，支持两端入队/出队，可作为栈使用。

| 栈方法 | 等效 Deque 方法 |
|--------|----------------|
| `push(e)` | `addFirst(e)` |
| `pop()` | `removeFirst()` |

```java
Deque<String> stack = new LinkedList<>();
stack.push("A");
stack.push("B");
System.out.println(stack.pop()); // B
```

### 7.3 PriorityQueue
优先级队列，元素按优先级出队。默认自然顺序（升序），可传入 `Comparator`。底层为二叉堆，入队/出队 O(log n)。

```java
PriorityQueue<Integer> pq = new PriorityQueue<>((a, b) -> b - a); // 大顶堆
pq.offer(5);
pq.offer(1);
System.out.println(pq.poll()); // 5
```

## 8. Collections 工具类

| 方法 | 说明 |
|------|------|
| `sort(list)` | 排序（元素实现 `Comparable`） |
| `sort(list, comparator)` | 按比较器排序 |
| `binarySearch(list, key)` | 二分查找（需有序） |
| `reverse(list)` | 反转 |
| `shuffle(list)` | 随机打乱 |
| `max(c)` / `min(c)` | 最大/最小值 |
| `synchronizedXxx()` | 返回线程安全包装 |
| `unmodifiableXxx()` | 返回只读集合 |
| `emptyXxx()` | 返回不可变空集合 |

## 9. 集合工厂方法（Java 9+）
快速创建不可变集合：
```java
List<String> list = List.of("A", "B", "C");
Set<String> set = Set.of("A", "B", "C");
Map<String, Integer> map = Map.of("A", 1, "B", 2);
Map<String, Integer> map2 = Map.ofEntries(Map.entry("A", 1));
```
注意：通过 `of()` 创建的集合不可变。

## 10. 有序集合增强（Java 21+）
新增 `SequencedCollection`、`SequencedSet`、`SequencedMap` 接口，统一首尾操作。

```java
List<String> list = new ArrayList<>(List.of("A", "B", "C"));
list.addFirst("X");
list.addLast("Y");
System.out.println(list.getFirst()); // X
System.out.println(list.getLast());  // Y
List<String> reversed = list.reversed(); // 反转视图
```

## 11. 底层原理与源码分析

### 11.1 ArrayList 底层实现
- **数据结构**：`transient Object[] elementData`。
- **初始化**：无参构造赋空数组，首次 `add` 扩容至 **10**。
- **扩容**：`newCapacity = oldCapacity + (oldCapacity >> 1)`（约 1.5 倍），最大 `Integer.MAX_VALUE - 8`。
- **性能**：查询 O(1)，插入/删除非尾部 O(n)。
- **Fail-Fast**：迭代时非迭代器修改集合抛 `ConcurrentModificationException`。

### 11.2 LinkedList 底层实现
- **数据结构**：双向链表，`Node` 含 `prev`、`next`、`item`。
- **头尾操作**：O(1)。
- **查找**：`get(index)` 需遍历，优化：判断距头尾远近决定遍历方向。
- **性能**：增删 O(1)（已知节点），查询 O(n)。

### 11.3 HashMap 底层实现
- **数据结构**：数组 + 链表 + 红黑树。
- **寻址**：`(n - 1) & hash`（n 为 2 的幂）。
- **扰动函数**：`hash = key.hashCode() ^ (key.hashCode() >>> 16)`。
- **扩容**：默认容量 16，负载因子 0.75。扩容至 2 倍，元素位置在 `原索引 i` 或 `i + 原数组长度`。
- **树化**：链表长度 ≥ 8 且数组长度 ≥ 64 时转为红黑树。
- **退化**：树节点 ≤ 6 时转回链表。

### 11.4 HashSet 与 TreeSet
- **HashSet**：基于 `HashMap`，`add(e)` 调用 `map.put(e, PRESENT)`。
- **TreeSet**：基于 `TreeMap`，利用红黑树排序，要求元素实现 `Comparable` 或传入 `Comparator`。

### 11.5 ConcurrentHashMap 关键机制
- **JDK 1.7**：分段锁（Segment）。
- **JDK 1.8**：CAS + `synchronized`，锁粒度细化到单个槽位。
- **扩容**：支持多线程并发扩容。
- **size 统计**：使用 `CounterCell` 数组分散热点。