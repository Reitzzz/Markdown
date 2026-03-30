# Response响应字符＆字节数据

## 响应字符 (HTML数据)

告诉浏览器发过去的是 HTML 网页文件，并指定字符编码为 UTF-8。

```java
response.setContentType("text/html;charset=utf-8"); 

PrintWriter writer = response.getWriter();

writer.write("你好");
writer.write("<h1>aaa</h1>");
```

## 响应字符 (JSON数据 - 现代开发核心重点)

在前后端分离的开发模式中，后端通常返回 JSON 格式的数据供前端解析，而不是返回 HTML 标签。必须设置正确的 Content-Type 响应头。

```java
response.setContentType("application/json;charset=utf-8");

PrintWriter writer = response.getWriter();
writer.write("{\"code\": 200, \"message\": \"success\"}");
```

## 响应字节数据

使用方法：

```java
FileInputStream fis = new FileInputStream("d://a.jpg"); 
ServletOutputStream os = resp.getOutputStream(); 

IOUtils.copy(fis, os);
fis.close();
```