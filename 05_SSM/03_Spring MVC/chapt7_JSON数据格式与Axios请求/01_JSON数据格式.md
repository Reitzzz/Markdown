**目录**

- [1. JSON数据格式与Axios请求](#1-json数据格式与axios请求)
  - [1.1 核心概念与前后端分离](#11-核心概念与前后端分离)
  - [1.2 JSON数据格式详解](#12-json数据格式详解)
  - [1.3 前端：JSON与JS对象的转换](#13-前端json与js对象的转换)
  - [1.4 后端：FastJSON框架与基础使用](#14-后端fastjson框架与基础使用)
  - [1.5 后端：SpringMVC直接响应JSON数据](#15-后端springmvc直接响应json数据)
    - [1.5.1 实体类转换json](#151-实体类转换json)
    - [1.5.2 FastJSON转换器实现转换](#152-fastjson转换器实现转换)

---

# 1. JSON数据格式与Axios请求

## 1.1 核心概念与前后端分离

JSON (JavaScript Object Notation, JS 对象简谱) 是一种轻量级的数据交换格式。

我们现在推崇的是前后端分离的开发模式，而不是所有的内容全部交给后端渲染再发送给浏览器，也就是说，整个Web页面的内容在一开始就编写完成了，而其中的数据由前端执行JS代码来向服务器动态获取，再到前端进行渲染（填充），这样可以大幅度减少后端的压力，**并且后端只需要传输关键数据即可**（在即将到来的SpringBoot阶段，我们将完全采用前后端分离的开发模式）

## 1.2 JSON数据格式详解

既然要实现前后端分离，那么我们就必须约定一种更加高效的数据传输模式，来向前端页面传输后端提供的数据。因此JSON横空出世，它非常容易理解，并且与前端的兼容性极好，因此现在比较主流的数据传输方式则是通过JSON格式承载的。

一个JSON格式的数据长这样，以学生对象为例：

```json
{"name": "杰哥", "age": 18}
```

多个学生可以以数组的形式表示：

```json
[{"name": "杰哥", "age": 18}, {"name": "阿伟", "age": 18}]
```

嵌套关系可以表示为：

```json
{"studentList": [{"name": "杰哥", "age": 18}, {"name": "阿伟", "age": 18}], "count": 2}
```

## 1.3 前端：JSON与JS对象的转换

它直接包括了属性的名称和属性的值，与JavaScript的对象极为相似，它到达前端后，可以直接转换为对象，以对象的形式进行操作和内容的读取，相当于以字符串形式表示了一个JS对象，我们可以直接在控制台窗口中测试：

```javascript
let obj = JSON.parse('{"studentList": [{"name": "杰哥", "age": 18}, {"name": "阿伟", "age": 18}], "count": 2}')
//将JSON格式字符串转换为JS对象
obj.studentList[0].name   //直接访问第一个学生的名称
```

我们也可以将JS对象转换为JSON字符串：

```javascript
JSON.stringify(obj)
```

我们后端就可以以JSON字符串的形式向前端返回数据，这样前端在拿到数据之后，就可以快速获取，非常方便。

## 1.4 后端：FastJSON框架与基础使用

那么后端如何快速创建一个JSON格式的数据呢？我们首先需要导入以下依赖：

```xml
<dependency>
      <groupId>com.alibaba.fastjson2</groupId>
      <artifactId>fastjson2</artifactId>
      <version>2.0.34</version>
</dependency>
```

JSON解析框架有很多种，比较常用的是Jackson和FastJSON，这里我们使用阿里巴巴的FastJSON进行解析，这是目前号称最快的JSON解析框架，并且现在已经强势推出FastJSON 2版本。

首先要介绍的是JSONObject，它和Map的使用方法一样，并且是有序的（实现了LinkedHashMap接口），比如我们向其中存放几个数据：

```java
@RequestMapping(value = "/index")
public String index(){
    JSONObject object = new JSONObject();
    object.put("name", "杰哥");
    object.put("age", 18);
    System.out.println(object.toJSONString());   //以JSON格式输出JSONObject字符串
    return "index";
}
```

最后我们得到的结果为：

```json
{"name": "杰哥", "age": 18}
```

实际上JSONObject就是对JSON数据的一种对象表示。同样的还有JSONArray，它表示一个数组，用法和List一样，数组中可以嵌套其他的JSONObject或是JSONArray：

```java
@RequestMapping(value = "/index")
public String index(){
    JSONObject object = new JSONObject();
    object.put("name", "杰哥");
    object.put("age", 18);
    JSONArray array = new JSONArray();
    array.add(object);
    System.out.println(array.toJSONString());
    return "index";
}
```

得到的结果为：

```json
[{"name": "杰哥", "age": 18}]
```

当出现循环引用时，会按照以下语法来解析：

![img](https://s2.loli.net/2023/08/14/MjO4awH3X1YnlmR.png)

## 1.5 后端：SpringMVC直接响应JSON数据

### 1.5.1 实体类转换json

我们可以也直接创建一个实体类，将实体类转换为JSON格式的数据：

```java
@RequestMapping(value = "/index", produces = "application/json")  
@ResponseBody
public String data(){
    Student student = new Student();
    student.setName("杰哥");
    student.setAge(18);
    return JSON.toJSONString(student);
}
```

这里我们修改了`produces`的值, 将返回的内容类型设定为`application/json`，**表示服务器端返回了一个JSON格式的数据**<u>（当然不设置也行，也能展示，这样是为了规范）</u>>然后我们在方法上添加一个`@ResponseBody`表示方法返回（也可以在类上添加`@RestController`表示此Controller默认返回的是字符串数据）的结果不是视图名称而是直接需要返回一个字符串作为页面数据，这样，返回给浏览器的就是我们直接返回的字符串内容。

接着我们使用JSON工具类将其转换为JSON格式的字符串，打开浏览器，得到JSON格式数据。

### 1.5.2 FastJSON转换器实现转换

SpringMVC非常智能，我们可以直接返回一个对象类型，它会被自动转换为JSON字符串格式：

在配置类WebConfiguration中添加一下FastJSON转换器，这里需要先添加一个依赖：

```xml
<dependency>
    <groupId>com.alibaba.fastjson2</groupId>
    <artifactId>fastjson2-extension-spring6</artifactId>
    <version>2.0.34</version>
</dependency>
```

然后编写配置：

```java
@Override
public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
    converters.add(new FastJsonHttpMessageConverter());
}
```

```java
@RequestMapping(value = "/data", produces = "application/json")
@ResponseBody
public Student data(){
    Student student = new Student();
    student.setName("杰哥");
    student.setAge(18);
    return student;
}
```



再次尝试，内容就会自动转换为JSON格式响应给客户端了。