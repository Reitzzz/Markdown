# 响应字符

使用方法
```c
response.setContentType("text/html;charset=utf-8"); //告诉浏览器：“我发给你的是一个 HTML 网页文件 并指定字符编码为UTF-8

PrintWriter writer = response.getWriter();//获取字符输出流


writer.write( s: "你好");
writer.write( s: "<h1>aaa</h1>");
```

注意： 流不需要关闭

# 响应字节数据

使用方法：

```c
FileInputStream fis = new FileInputStream( name: "d://a.jpg"); //1.读取文件
ServletOutputStream os = resp.getOutputStream(); //2.通过Response对象获取字节输出流

IOUtils.copy(fis,os);
fis.close();
```