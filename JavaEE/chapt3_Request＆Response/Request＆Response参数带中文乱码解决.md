# 解决参数带中文乱码

## POST请求解决

在获取参数之前增加一行 `request.setCharacterEncoding("UTF-8");`。这是因为 POST 请求的参数在请求体中，需要显式告诉服务器按照 UTF-8 来解析。

```java
@WebServlet("/hobbyServlet")
public class HobbyServlet extends HttpServlet {

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
       req.setCharacterEncoding("UTF-8");
       String hobby=req.getParameter("hobby");
       String username=req.getParameter("username");
    }
}

