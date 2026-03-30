# 请求转发

请求转发（forward）：**一种在服务器内部的资源跳转方式** <br>
在 JavaEE 开发中，A 通常是一个负责处理数据的 Servlet，B 通常是一个负责显示界面的 JSP 或 HTML <br>

举例：你去政务大厅找 A 窗口办事（发送请求给 A）。A 拿到你的材料后，发现这事得归 B 管。于是，A 在后台自己跑去找 B，把材料交给 B 处理。B 处理完生成了结果，交还给 A，最后由 A 把结果递到你手上

与重定向的不同： 你去 A 窗口办事。A 看了看说：“这事我不办，你得去 B 窗口办，B 窗口在二楼左转。”。你听完后，自己乖乖跑到 B 窗口重新提交了申请，最后 B 把结果交给你。 **产生了新的Request对象** （即新的工单）
## 特点
1. 浏览器地址栏路径不发生变化
2. 只能转发到当前服务器的内部资源
3. 一次请求，可以在转发的资源间使用request共享数据
## 实现方式：
req.getRequestDispatcher("资源B路径").forward(req,resp);

## 请求转发资源间共享数据： 使用Request对象

void setAttribute(String name, Object o): 存储数据到 request域中

Object getAttribute(String name): 根据 key，获取值

void removeAttribute(String name): 根据 key，删除该键值对