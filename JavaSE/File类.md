# File类

## 目录
- [File类](#file类)
  - [目录](#目录)
  - [创建](#创建)
  - [常见成员方法](#常见成员方法)
    - [判断、获取](#判断获取)
    - [创建与删除](#创建与删除)
    - [获取并遍历](#获取并遍历)

## 创建

- 根据文件路径创建文件对象
方法声明：public File(String pathname)
```java
File f1 = new File("D:\\test.txt");
```

- 根据父路径名字符串和子路径名字符串创建文件对象
方法声明：public File(String parent, String child)
```java
File f2 = new File("D:\\", "test.txt");
```

- 根据父路径对应文件对象和子路径名字符串创建文件对象
方法声明：public File(File parent, String child)
```java
File parentFile = new File("D:\\");
File f3 = new File(parentFile, "test.txt");
```

## 常见成员方法

### 判断、获取

- 判断是否为文件夹
方法声明：public boolean isDirectory()
```java
boolean isDir = f1.isDirectory();
```

- 判断是否为文件
方法声明：public boolean isFile()
```java
boolean isFile = f1.isFile();
```

- 判断是否存在
方法声明：public boolean exists()
```java
boolean exists = f1.exists();
```

- 获取文件大小。只能获取文件的大小，单位是字节，无法获取文件夹的大小。
方法声明：public long length()
```java
long len = f1.length();
```

- 获取绝对路径
方法声明：public String getAbsolutePath()
```java
String absPath = f1.getAbsolutePath();
```

- 获取路径
方法声明：public String getPath()
```java
String path = f1.getPath();
```

- 获取名称
方法声明：public String getName()
```java
String name = f1.getName();
```

- 获取最后修改时间
方法声明：public long lastModified()
```java
long time = f1.lastModified();
```

### 创建与删除

  创建文件。<br>
- 1. 如果当前路径表示的文件不存在，则创建成功，返回true；
- 2. 如果父级路径不存在会有异常IO；如果路径中不包括后缀名，则创建一个没有后缀的问题。<br> 
  
  方法声明：public boolean createNewFile()
```java
boolean created = f1.createNewFile();
```

- 创建单级文件夹。只能创建单级文件夹。
方法声明：public boolean mkdir()
```java
boolean dirCreated = f1.mkdir();
```

- 创建多级文件夹。用于创建多级文件夹，也可以创建单级。
方法声明：public boolean mkdirs()
```java
boolean dirsCreated = f1.mkdirs();
```

- 删除文件或空文件夹。<br>
- 1. 如果是文件或空文件夹直接删除，不走回收站；
- 2. 如果是有内容的文件夹，则删除失败。
  
方法声明：public boolean delete()
```java
boolean deleted = f1.delete();
```

### 获取并遍历

- 获取当前路径下的所有文件和文件夹对象。<br>
  1. 表示路径不存在时返回null；表示路径为文件时返回null；
  2. 表示路径为空文件夹时返回长度为0的数组。
   
方法声明：public File[] listFiles()
```java
File[] files = f1.listFiles();
if (files != null) {
    for (File file : files) {
        System.out.println(file.getName());
    }
}
```