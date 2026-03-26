# IO流

## 目录
- [IO流](#io流)
  - [目录](#目录)
  - [与File的区别](#与file的区别)
  - [分类](#分类)
    - [流的方向](#流的方向)
    - [操作文件类型](#操作文件类型)
      - [字节流](#字节流)
      - [字符流](#字符流)
      - [数据流](#数据流)
      - [对象流](#对象流)
  - [拷贝文件](#拷贝文件)
  - [资源释放](#资源释放)

## 与File的区别
File是对文件本身进行操作(新建 删除等)。
IO流则可以读写文件中的数据。

## 分类

### 流的方向
- Input流：读取本地文件中的数据。
- Output流：把数据写到本地文件当中。

### 操作文件类型

#### 字节流
可以操作所有类型文件。

- **InputStream(抽象类) -> FileInputStream**
创建对象：如果文件不存在会直接报错。
方法声明：public FileInputStream(String name)
```java
FileInputStream fis = new FileInputStream("str");
```

读取数据：每调用一次读一个字节，随后移动指针。读到文件末尾 read方法返回-1。
方法声明：public int read()
```java
int b = fis.read();
System.out.print((char)b);
```

读取数据：返回一次读取的长度len。
方法声明：public int read(byte[] buffer)
```java
byte[] buffer = new byte[1024];
int len = fis.read(buffer);
String str = new String(buffer, 0, len);
```

释放资源
方法声明：public void close()
```java
fis.close();
```

- **OutputStream(抽象类) -> FileOutputStream**
创建对象：str可以表示路径或File对象。如果文件不存在会创建一个新的文件，但要保证父路径存在。如果文件存在会清空文件。str后接一个true参数，打开续写开关。
方法声明：public FileOutputStream(String name, boolean append)
```java
FileOutputStream fos = new FileOutputStream("str", true);
```

写入数据：参数是整数，写入数据是ASCII对应字符。三种方式写入。换行: 输入换行符\r\n。
方法声明：public void write(int b)
方法声明：public void write(byte[] b)
方法声明：public void write(byte[] b, int off, int len)
```java
fos.write(97);
byte[] bytes = {97, 98, 99};
fos.write(bytes);
fos.write(bytes, 0, 3);
```

释放资源
方法声明：public void close()
```java
fos.close();
```

#### 字符流
只能操作纯文本文件。
Writer(抽象类)
FileReader(抽象类)

#### 数据流
- DataInputStream
方法声明：public final String readUTF()
方法声明：public final int readInt()
```java
DataInputStream dis = new DataInputStream(fis);
String str = dis.readUTF();
int num = dis.readInt();
```

- DataOutputStream
方法声明：public final void writeInt(int v)
方法声明：public final void writeUTF(String str)
```java
DataOutputStream dos = new DataOutputStream(fos);
dos.writeInt(100);
dos.writeUTF("str");
```

#### 对象流
Student类需要implements Serializable。

- ObjectOutputStream
方法声明：public final void writeObject(Object obj)
```java
ObjectOutputStream oos = new ObjectOutputStream(fos);
oos.writeObject(student);
```

- ObjectInputStream
方法声明：public final Object readObject()
```java
ObjectInputStream ois = new ObjectInputStream(fis);
Object obj = ois.readObject();
```

## 拷贝文件

- 小文件
核心思想: 边读边写。先创建的对象后释放资源。
```java
int b;
while ((b = fis.read()) != -1) {
    fos.write(b);
}
```

- 大文件
```java
int len;
byte[] bytes = new byte[1024];
while ((len = fis.read(bytes)) != -1) {
    fos.write(bytes, 0, len);
}
```

## 资源释放

- try catch finally
先创建对象 但赋值为null。try中再给对象具体赋值。finally中先判断对象是否为null，不为null再执行try{fos.close()} catch{}。
```java
FileOutputStream fos = null;
try {
    fos = new FileOutputStream("str");
} catch (IOException e) {
    
} finally {
    if (fos != null) {
        try {
            fos.close();
        } catch (IOException e) {
            
        }
    }
}
```

- 现代 try-with-resources 语法
```java
try (FileInputStream fis = new FileInputStream("a.txt");
     FileOutputStream fos = new FileOutputStream("b.txt")) {
    int b;
    while ((b = fis.read()) != -1) {
        fos.write(b);
    }
} catch (IOException e) {
    
}
```