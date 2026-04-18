# File类

## 目录
- [1. 创建](#1-创建)
  - [1.1 根据文件路径创建](#11-根据文件路径创建)
  - [1.2 根据父路径和子路径创建](#12-根据父路径和子路径创建)
  - [1.3 根据父文件对象和子路径创建](#13-根据父文件对象和子路径创建)
- [2. 常见成员方法](#2-常见成员方法)
  - [2.1 判断与获取](#21-判断与获取)
  - [2.2 创建与删除](#22-创建与删除)
  - [2.3 获取并遍历](#23-获取并遍历)

## 1. 创建

### 1.1 根据文件路径创建
方法声明：`public File(String pathname)`
```java
File f1 = new File("D:\\test.txt");
```

### 1.2 根据父路径和子路径创建
方法声明：`public File(String parent, String child)`
```java
File f2 = new File("D:\\", "test.txt");
```

### 1.3 根据父文件对象和子路径创建
方法声明：`public File(File parent, String child)`
```java
File parentFile = new File("D:\\");
File f3 = new File(parentFile, "test.txt");
```

## 2. 常见成员方法

### 2.1 判断与获取

- **判断是否为文件夹**  
  方法声明：`public boolean isDirectory()`
  ```java
  boolean isDir = f1.isDirectory();
  ```

- **判断是否为文件**  
  方法声明：`public boolean isFile()`
  ```java
  boolean isFile = f1.isFile();
  ```

- **判断是否存在**  
  方法声明：`public boolean exists()`
  ```java
  boolean exists = f1.exists();
  ```

- **获取文件大小（字节）**  
  方法声明：`public long length()`（只能获取文件大小，无法获取文件夹大小）
  ```java
  long len = f1.length();
  ```

- **获取绝对路径**  
  方法声明：`public String getAbsolutePath()`
  ```java
  String absPath = f1.getAbsolutePath();
  ```

- **获取路径**  
  方法声明：`public String getPath()`
  ```java
  String path = f1.getPath();
  ```

- **获取名称**  
  方法声明：`public String getName()`
  ```java
  String name = f1.getName();
  ```

- **获取最后修改时间**  
  方法声明：`public long lastModified()`
  ```java
  long time = f1.lastModified();
  ```

### 2.2 创建与删除

- **创建文件**  
  方法声明：`public boolean createNewFile()`
  - 文件不存在则创建成功返回 `true`。
  - 父路径不存在抛出 `IOException`；路径不含后缀则创建无后缀文件。
  ```java
  boolean created = f1.createNewFile();
  ```

- **创建单级文件夹**  
  方法声明：`public boolean mkdir()`
  ```java
  boolean dirCreated = f1.mkdir();
  ```

- **创建多级文件夹**  
  方法声明：`public boolean mkdirs()`（可创建单级或多级）
  ```java
  boolean dirsCreated = f1.mkdirs();
  ```

- **删除文件或空文件夹**  
  方法声明：`public boolean delete()`
  - 直接删除，不走回收站。
  - 若文件夹非空则删除失败。
  ```java
  boolean deleted = f1.delete();
  ```

### 2.3 获取并遍历

- **获取当前路径下所有文件和文件夹对象**  
  方法声明：`public File[] listFiles()`
  - 路径不存在或为文件时返回 `null`。
  - 空文件夹返回长度为 0 的数组。
  ```java
  File[] files = f1.listFiles();
  if (files != null) {
      for (File file : files) {
          System.out.println(file.getName());
      }
  }
  ```