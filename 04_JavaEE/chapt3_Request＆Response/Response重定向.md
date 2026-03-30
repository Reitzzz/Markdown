# Response重定向

## 实现方式
1. 标准写法
```java
resp.setStatus(302); 
resp.setHeader("location", "资源B的路径"); 
```
2. 简化写法（推荐）

```java
resp.sendRedirect("资源B的路径");
```