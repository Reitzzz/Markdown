# Response重定向

## 实现方式
1. 标准写法
```c
resp.setStatus(302); //设置响应状态
resp.setHeader("location","资源B的路径"); //设置响应头Location
```
2. 简化写法（推荐）

```c
   resp.sendRedirect("资源B的路径");
```
