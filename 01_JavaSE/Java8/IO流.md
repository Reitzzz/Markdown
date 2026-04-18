# IO流

## 目录
- [1. 与File的区别](#1-与file的区别)
- [2. 分类](#2-分类)
  - [2.1 流的方向](#21-流的方向)
  - [2.2 操作文件类型](#22-操作文件类型)
    - [2.2.1 字节流](#221-字节流)
    - [2.2.2 字符流](#222-字符流)
    - [2.2.3 数据流](#223-数据流)
    - [2.2.4 对象流](#224-对象流)
- [3. 拷贝文件](#3-拷贝文件)
  - [3.1 小文件拷贝](#31-小文件拷贝)
  - [3.2 大文件拷贝](#32-大文件拷贝)
- [4. 资源释放](#4-资源释放)
  - [4.1 try-catch-finally 方式](#41-try-catch-finally-方式)
  - [4.2 try-with-resources 语法](#42-try-with-resources-语法)
- [5. 缓冲流原理与装饰者模式](#5-缓冲流原理与装饰者模式)
  - [5.1 为什么需要缓冲流](#51-为什么需要缓冲流)
  - [5.2 BufferedInputStream 内部机制](#52-bufferedinputstream-内部机制)
  - [5.3 装饰者模式](#53-装饰者模式)
  - [5.4 flush() 的重要性](#54-flush-的重要性)
  - [5.5 字符流与编码](#55-字符流与编码)
- [6. 转换流](#6-转换流)
- [7. 打印流](#7-打印流)
- [8. 字节数组流](#8-字节数组流)
- [9. Files 工具类](#9-files-工具类)
- [10. 输入流快捷方法](#10-输入流快捷方法)

## 1. 与File的区别
File 对文件本身进行操作（新建、删除等）。  
IO 流则读写文件中的数据。

## 2. 分类

### 2.1 流的方向
- **输入流（Input）**：读取本地文件中的数据。
- **输出流（Output）**：把数据写到本地文件。

### 2.2 操作文件类型

#### 2.2.1 字节流
可操作所有类型文件。

- **InputStream（抽象类）→ FileInputStream**  
  创建对象：文件不存在则报错。  
  方法声明：`public FileInputStream(String name)`
  ```java
  FileInputStream fis = new FileInputStream("str");
  ```

  读取单字节：读到末尾返回 -1。  
  方法声明：`public int read()`
  ```java
  int b = fis.read();
  System.out.print((char) b);
  ```

  读取字节数组：返回实际读取长度 `len`。  
  方法声明：`public int read(byte[] buffer)`
  ```java
  byte[] buffer = new byte[1024];
  int len = fis.read(buffer);
  String str = new String(buffer, 0, len);
  ```

  释放资源：  
  方法声明：`public void close()`
  ```java
  fis.close();
  ```

- **OutputStream（抽象类）→ FileOutputStream**  
  创建对象：文件不存在则创建（父路径必须存在）；存在则清空文件。追加模式可在参数中传入 `true`。  
  方法声明：`public FileOutputStream(String name, boolean append)`
  ```java
  FileOutputStream fos = new FileOutputStream("str", true);
  ```

  写入数据：支持单字节、字节数组、部分字节数组。换行使用 `\r\n`。  
  ```java
  fos.write(97);
  byte[] bytes = {97, 98, 99};
  fos.write(bytes);
  fos.write(bytes, 0, 3);
  ```

  释放资源：  
  ```java
  fos.close();
  ```

#### 2.2.2 字符流
只能操作纯文本文件。  
常用抽象类：`Writer`、`Reader`，具体实现如 `FileReader`、`FileWriter`。

#### 2.2.3 数据流
- **DataInputStream**  
  方法声明：`public final String readUTF()`、`public final int readInt()`
  ```java
  DataInputStream dis = new DataInputStream(fis);
  String str = dis.readUTF();
  int num = dis.readInt();
  ```

- **DataOutputStream**  
  方法声明：`public final void writeInt(int v)`、`public final void writeUTF(String str)`
  ```java
  DataOutputStream dos = new DataOutputStream(fos);
  dos.writeInt(100);
  dos.writeUTF("str");
  ```

#### 2.2.4 对象流
实体类需实现 `Serializable` 接口。

- **ObjectOutputStream**  
  方法声明：`public final void writeObject(Object obj)`
  ```java
  ObjectOutputStream oos = new ObjectOutputStream(fos);
  oos.writeObject(student);
  ```

- **ObjectInputStream**  
  方法声明：`public final Object readObject()`
  ```java
  ObjectInputStream ois = new ObjectInputStream(fis);
  Object obj = ois.readObject();
  ```

## 3. 拷贝文件

### 3.1 小文件拷贝
边读边写，注意先创建的对象后释放。
```java
int b;
while ((b = fis.read()) != -1) {
    fos.write(b);
}
```

### 3.2 大文件拷贝
使用字节数组缓冲。
```java
int len;
byte[] bytes = new byte[1024];
while ((len = fis.read(bytes)) != -1) {
    fos.write(bytes, 0, len);
}
```

## 4. 资源释放

### 4.1 try-catch-finally 方式
先声明为 `null`，在 `try` 中赋值，`finally` 中判空后关闭。
```java
FileOutputStream fos = null;
try {
    fos = new FileOutputStream("str");
} catch (IOException e) {
    // 处理异常
} finally {
    if (fos != null) {
        try {
            fos.close();
        } catch (IOException e) {
            // 处理关闭异常
        }
    }
}
```

### 4.2 try-with-resources 语法
```java
try (FileInputStream fis = new FileInputStream("a.txt");
     FileOutputStream fos = new FileOutputStream("b.txt")) {
    int b;
    while ((b = fis.read()) != -1) {
        fos.write(b);
    }
} catch (IOException e) {
    // 处理异常
}
```

## 5. 缓冲流原理与装饰者模式

### 5.1 为什么需要缓冲流
磁盘/网络 IO 较慢，无缓冲流每次 `read()` 都触发系统调用。缓冲流预先读取一块数据到内存缓冲区，后续 `read()` 直接从内存取数据。

### 5.2 BufferedInputStream 内部机制
- **默认缓冲区大小**：8192 字节（8 KB）。
- **`read()` 流程**：先检查缓冲区是否有数据 → 若无则从底层流填充 → 返回缓冲区中的字节。
- **`mark(reset)` 支持**：允许回溯读取，受 `readlimit` 限制，超出则失效。

### 5.3 装饰者模式
Java IO 流体系是典型的**装饰者模式**：
- **被装饰者（Component）**：`FileInputStream`（核心功能：读文件）。
- **装饰者（Decorator）**：`BufferedInputStream`（附加功能：缓冲）。
- **关系**：`BufferedInputStream` 内部持有 `InputStream` 引用，构造时传入被包装对象。

```java
InputStream in = new BufferedInputStream(new FileInputStream("file.txt"));
```

### 5.4 flush() 的重要性
- `BufferedOutputStream` 写入数据时先存入内部缓冲区。
- 只有缓冲区满、手动调用 `flush()` 或关闭流（`close`）时，数据才真正写入硬盘。
- **忘记 `flush()` 可能导致数据丢失**。

### 5.5 字符流与编码
- `FileReader` 默认使用**平台默认编码**（Windows 常为 GBK，Linux/Mac 常为 UTF-8），跨平台易乱码。
- **推荐写法**：`new InputStreamReader(new FileInputStream("file"), StandardCharsets.UTF_8)`。

## 6. 转换流
`InputStreamReader` / `OutputStreamWriter` 实现字节流与字符流转换，**可指定编码**。

```java
// 读取时指定 UTF-8
try (BufferedReader reader = new BufferedReader(
        new InputStreamReader(new FileInputStream("test.txt"), StandardCharsets.UTF_8))) {
    String line;
    while ((line = reader.readLine()) != null) {
        System.out.println(line);
    }
}

// 写入时指定编码
try (BufferedWriter writer = new BufferedWriter(
        new OutputStreamWriter(new FileOutputStream("out.txt"), StandardCharsets.UTF_8))) {
    writer.write("你好，世界");
}
```

> 注意：`FileReader` 默认使用平台编码，推荐始终使用转换流并显式指定编码。

## 7. 打印流
- `PrintStream` / `PrintWriter` 提供 `print()`、`println()`、`printf()` 等便捷方法。
- `System.out` 即为 `PrintStream`。
- `PrintWriter` 支持自动刷新。

```java
try (PrintWriter pw = new PrintWriter(new FileWriter("log.txt"), true)) { // true 表示自动刷新
    pw.println("日志信息");
    pw.printf("格式化输出：%s %d%n", "张三", 25);
}
```

## 8. 字节数组流
数据源或目标为内存中的 `byte[]`，不涉及磁盘 I/O，常用于临时缓存或模拟流操作。

```java
// 写入内存
ByteArrayOutputStream baos = new ByteArrayOutputStream();
baos.write("Hello".getBytes());
byte[] data = baos.toByteArray();

// 从内存读取
ByteArrayInputStream bais = new ByteArrayInputStream(data);
int b;
while ((b = bais.read()) != -1) {
    System.out.print((char) b);
}
```

## 9. Files 工具类
位于 `java.nio.file`，提供便捷静态方法。

```java
Path path = Path.of("test.txt");

// 读写字符串（Java 11+）
String content = Files.readString(path);
Files.writeString(path, "Hello World");

// 读取所有行
List<String> lines = Files.readAllLines(path);

// 复制、移动、删除
Files.copy(path, Path.of("copy.txt"), StandardCopyOption.REPLACE_EXISTING);
Files.move(path, Path.of("moved.txt"));
Files.delete(path);
Files.deleteIfExists(path);

// 遍历目录
try (Stream<Path> walk = Files.walk(Path.of("."))) {
    walk.filter(Files::isRegularFile)
        .forEach(System.out::println);
}
```

## 10. 输入流快捷方法
Java 9+ 为 `InputStream` 新增便捷方法：

```java
try (InputStream is = new FileInputStream("test.txt")) {
    // 一次性读取所有字节
    byte[] allBytes = is.readAllBytes();
    
    // 读取指定数量字节
    byte[] buffer = is.readNBytes(10);
    
    // 精确跳过 n 个字节
    is.skipNBytes(5);
    
    // 直接将输入流传输到输出流
    try (OutputStream os = new FileOutputStream("copy.txt")) {
        long transferred = is.transferTo(os);
        System.out.println("传输字节数：" + transferred);
    }
}
```