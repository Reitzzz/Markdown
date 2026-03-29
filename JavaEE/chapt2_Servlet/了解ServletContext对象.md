# 了解 ServletContext 对象

`ServletContext` 全局唯一，它是属于整个 Web 应用程序的，可以通过 `getServletContext()` 方法来获取该对象。

## 1. 设置和获取附加值

此对象可以用来设置附加值，实现跨请求的数据传递：

```java
ServletContext context = getServletContext();
context.setAttribute("test", "我是重定向之前的数据");
resp.sendRedirect("time");
```

因为无论在哪里、什么时间，获取到的 `ServletContext` 始终是同一个对象，因此可以随时随地获取已添加的属性：

```java
System.out.println(getServletContext().getAttribute("test"));
```

## 2. 请求转发

除了数据传递，`ServletContext` 还可以用于执行请求转发：

```java
context.getRequestDispatcher("/time").forward(req, resp);
```

## 3. 获取资源文件

它还可以获取 Web 应用程序根目录下的资源文件（注意：是 webapp 根目录下的资源，而不是 resource 目录中的资源）。有关 `ServletContext` 的其他高级特性，将在后续学习中进一步探讨。