# 第3章：Request和Response对象及进阶交互

本章核心：了解 HTTP 请求与响应的本质，掌握如何获取前端传递的数据，如何向前端反馈数据，以及实现异步请求和文件传输等进阶功能。

## 1. Request 对象 (处理客户端请求)
* **获取请求数据**
  * [01-Request & Response的通用方式](Request%20&%20Response的通用方式.md)
    * 区分 GET 与 POST 的语义
    * 使用 `getParameterMap()`, `getParameterValues()`, `getParameter()` 提取参数
    * 使用 `getHeader()` 获取请求头信息 (如 Token、User-Agent)
* **中文乱码处理**
  * [02-Request＆Response参数带中文乱码解决](Request＆Response参数带中文乱码解决.md)
    * POST 请求体中文乱码的 `setCharacterEncoding` 解决方案
* **服务器内部跳转**
  * [03-Request请求转发逻辑与数据共享](Request请求转发.md)
    * 请求转发 (Forward) 的原理与特点
    * 使用 Request 域对象 (`setAttribute` / `getAttribute`) 共享数据

## 2. Response 对象 (向客户端响应数据)
* **响应基础设置**
  * [04-Response设置数据响应 (状态码、响应头、响应体)](Response设置数据响应.md)
    * 设置 HTTP 状态码与响应头
    * 获取字符/字节输出流
* **响应具体数据**
  * [05-Response响应字符＆字节数据 (输出HTML与文件流)](Response响应字符＆字节数据.md)
    * 响应 HTML 网页数据
    * 响应 JSON 格式数据 (前后端分离重点)
    * 响应字节流数据 (图片/文件读取)
* **客户端页面跳转**
  * [06-Response重定向](Response重定向.md)
    * 重定向 (Redirect) 的标准写法与简化写法 (`sendRedirect`)
    * 重定向与请求转发的区别

## 3. 进阶交互实战 (异步与IO)
* **局部动态更新**
  * [07-XHR动态更新](XHR动态更新.md)
    * 原生 JavaScript 发起 XMLHttpRequest 异步请求
    * Servlet 配合处理并返回局部数据
* **文件传输处理**
  * [08-上传与下载文件](上传和下载文件.md)
    * 使用 `commons-io` 实现文件下载
    * 前端 `multipart/form-data` 表单规范
    * 后端 `@MultipartConfig` 注解与 `Part` 对象解析上传文件