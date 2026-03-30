# Response设置数据响应

## 响应行
```http
HTTP/1.1 200 OK
```

1. void setStatus(int sc)：设置响应状态码

## 响应头
```http
Content-Type: text/html
```

1. void setHeader(String name, String value)：设置响应头键值对

## 响应体：
```html
<html><head></head><body></body></html>
```

1. PrintWriter getWriter()：获取字符输出流
2. ServletOutputStream getOutputStream()：获取字节输出流