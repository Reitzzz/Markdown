# 集合Collection

## 目录
- [Collection](#collection)
- [ArrayList](#arraylist)
- [双列集合](#双列集合)
- [Map](#map)
- [Set](#set)
- [遍历方式](#遍历方式)

## Collection

单列集合的祖宗接口。

### 使用

- 增加元素
方法声明：public boolean add(E e)
```java
boolean added = collection.add("Element");
```

- 删除元素
方法声明：public boolean remove(Object o)
```java
boolean removed = collection.remove("Element");
```

- 判断是否为空
方法声明：public boolean isEmpty()
```java
boolean empty = collection.isEmpty();
```

- 大小
方法声明：public int size()
```java
int size = collection.size();
```

## ArrayList

### 使用场景与特点
想要集合中的元素可重复。继承自Collection接口，方法基本无差异。添加的元素是有序，可重复，有索引。

### 和数组的对比
集合：长度可变，基本数据类型需要包装，可以存引用数据类型。
数组：长度不可变，可以存基本数据类型，可以存引用数据类型。

### 创建
使用基本数据类型时，需要使用其包装类（char -> Character, int -> Integer, 其他首字母大写）。
```java
ArrayList<String> list = new ArrayList<>();
ArrayList<Integer> numList = new ArrayList<>();
```

## 双列集合

### 结构
键-值对。键不可以重复，值可以重复，无索引。

### 特点
双列集合一次需要存一对数据，分别为键和值。如果键出现重复，则覆盖。键和值一对应，每个键只能找到自己对应的值。

## Map

Map是双列集合的顶层接口，它的功能是全部双列集合都可以继承使用的。

### 常见API

- 创建对象
```java
Map<String, String> m = new HashMap<>();
```

- 添加/覆盖元素。如果键不存在直接写入，返回为空；如果键存在则覆盖原值，并返回被覆盖的原值。
方法声明：public V put(K key, V value)
```java
m.put("key1", "value1");
```

- 删除元素并返回值
方法声明：public V remove(Object key)
```java
String removedValue = m.remove("key1");
```

- 清空集合
方法声明：public void clear()
```java
m.clear();
```

- 判断是否包含键/值
方法声明：public boolean containsKey(Object key)
方法声明：public boolean containsValue(Object value)
```java
boolean hasKey = m.containsKey("key1");
boolean hasValue = m.containsValue("value1");
```

- 判断是否为空 / 获取大小
方法声明：public boolean isEmpty()
方法声明：public int size()
```java
boolean empty = m.isEmpty();
int size = m.size();
```

### 遍历

1. 键找值
方法声明：public Set<K> keySet()
```java
Set<String> keys = m.keySet();
for (String key : keys) {
    String value = m.get(key);
}
```

2. 键值对
方法声明：public Set<Map.Entry<K, V>> entrySet()
```java
Set<Map.Entry<String, String>> entries = m.entrySet();
for (Map.Entry<String, String> entry : entries) {
    String key = entry.getKey();
    String value = entry.getValue();
}
```

### HashMap
是Map的实现类。如果键存储的是自定义对象，需要重写hashCode和equals方法（可以通过源代码操作自动生成）。如果值存储的是自定义对象，则不需要重写。

## Set

### 特点
添加的元素是无序，不重复，无索引。继承自Collection接口，方法基本无差异。

### 三个实现类

- **HashSet**
使用场景: 对集合中的元素去重。无序，不重复，无索引。如果集合中存储的是自定义对象，必须重写hashCode和equals方法。
哈希值：如果没有重写hashCode的方法，不同对象计算出的哈希值是不同的。如果已经重写hashCode的方法，不同的对象只要属性值相同，计算出来的哈希值是一样的。在小部分情况下，不同的属性值或者不同的地址值，计算出来的哈希值也有可能一样，称为哈希碰撞。

- **LinkedHashSet**
使用场景: 元素去重且保证存取顺序。有序，不重复，无索引。

- **TreeSet**
使用场景: 对集合中的元素进行排序。可排序，不重复，无索引。
两种比较方式：
1. 通过JavaBean类实现Comparable接口指定比较规则(自然排序)。重写compareTo()方法。返回结果为负数添加到左边，为正数添加到右边。
```java
public class Student implements Comparable<Student> {
    @Override
    public int compareTo(Student o) {
        return this.getAge() - o.getAge();
    }
}
```
2. 创建TreeSet的对象时，传递比较器Comparator指定规则。需要降序排序或者其他时使用。同时存在时，比较器优先。
```java
TreeSet<String> ts = new TreeSet<>(new Comparator<String>() {
    @Override
    public int compare(String o1, String o2) {
        return o1.compareTo(o2);
    }
});
```

## 遍历方式

### 增强for遍历
使用限制：单列集合，数组。第三方变量s修改不改变集合数据。
```java
for (String s : list) {
    System.out.println(s);
}
```

### Lambda表达式遍历
方法声明：default void forEach(Consumer<? super T> action)
```java
list.forEach((String s) -> {
    System.out.println(s);
});
```