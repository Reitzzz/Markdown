# Request & Response

常用的三种方法**同时适用**于doGet()和doPost()

GET 和 POST 并**不是在做“相反”的事**。

它们都是客户端（比如浏览器）向服务器发送请求的方式。它们的本质区别在于语义和传递数据的位置：

GET 的语义通常是向服务器“索要/查询”数据，它携带参数的方式是直接拼接在 URL 后面。

POST 的语义通常是向服务器“提交/新增”数据，它携带参数的方式是放在 HTTP 的请求体（Body）内部。

可以看出，虽然它们的目的不同，数据存放的位置不同，但它们都在把客户端的参数传递给服务器。既然都有参数传递，就需要有方法把参数取出来。

## Map<String, String[]> getParameterMap() 
**获取所有参数的Map集合**

Map<String, String[]> map = req.getParameterMap();

```c
@WebServlet("/reqDemo")
public class RequestDemoServlet extends HttpServlet { //extends了HttpServlet 则下面都使用Servlet

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("get....");

        Map<String, String[]> map = req.getParameterMap();

        for (String key : map.keySet()) {
            System.out.print(key + ":");

            String[] values = map.get(key);
            for (String value : values) {
                System.out.print(value + " ");
            }

            System.out.println();
        }
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        this.doGet(req, resp);
    }
}

```

## String[] getParameterValues(String name)

根据名称获取参数值的**数组**

String[] hobbies = req.getParameterValues("hobby");

```c
@WebServlet("/hobbyServlet")
public class HobbyServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String[] hobbies = req.getParameterValues("hobby");

        if (hobbies != null) {
            for (String hobby : hobbies) {
                System.out.println(hobby);
            }
        }
    }
}
```
## String getParameter(String name)
根据名称获取**单个**参数值

String hobby=req.getParameter("hobby");

```c
@WebServlet("/hobbyServlet")
public class HobbyServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
       String hobby=req.getParameter("hobby");
       String username=req.getParameter("username");
    }
}
```
