# Session会话跟踪

Session（会话）：**一种在服务器端保存用户数据的技术**

在 Web 开发中，HTTP 协议是**无状态**的。这意味着每次请求对于服务器来说都是独立的，服务器默认记不住你上一次发过什么请求。

举例：你去一家常去的超市（发送请求给服务器），但收银员（HTTP协议）记性不好，每次都不认识你，也不记得你的积分。为了解决这个问题，超市给你办了一张 VIP会员卡（Session ID）。你以后每次去结账只要出示这张卡（浏览器携带 Cookie/Session ID），超市系统（服务器）一扫，就能从后台调出你的专属积分和购物记录（Session 数据）。

与 Request 共享数据的不同：Request 域的数据只能在**一次请求**（如请求转发）中共享。而 Session 域的数据可以在**多次请求**（比如重定向、或者先后访问不同的 Servlet）中共享，只要这些请求属于同一个会话。

## 特点
1. 数据存储在服务器端，相对安全
2. 每个浏览器对应一个独立的 Session 对象
3. 可以在同一次会话的多次请求之间共享数据

## 实现方式：

HttpSession session = req.getSession();

## Session 资源间共享数据：

void setAttribute(String name, Object value): 存储数据到 session 域中

Object getAttribute(String name): 根据 key，获取值

void removeAttribute(String name): 根据 key，删除该键值对

## Session 生命周期控制

void invalidate(): 销毁当前 Session

## 代码示例

**存储数据到 Session**

```java
@WebServlet("/sessionDemo1")
public class SessionDemo1 extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        HttpSession session = req.getSession();
        session.setAttribute("username", "zhangsan");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        this.doGet(req, resp);
    }
}
```

**从 Session 获取数据**

```java
@WebServlet("/sessionDemo2")
public class SessionDemo2 extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        HttpSession session = req.getSession();
        Object username = session.getAttribute("username");
        System.out.println(username);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        this.doGet(req, resp);
    }
}
```