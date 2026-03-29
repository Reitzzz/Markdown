# XHR动态更新

## 1. 使用 XHR 请求数据 (动态更新)

若希望在不刷新整个网页的情况下动态更新局部内容（例如点击按钮刷新当前时间），可以使用 JavaScript 发起异步请求。

首先在前端编写 JavaScript，定义发起 XHR 请求及处理响应结果的逻辑：

```javascript
function updateTime() {
    let xhr = new XMLHttpRequest();
    xhr.onreadystatechange = function() {
        if (xhr.readyState === 4 && xhr.status === 200) {
            document.getElementById("time").innerText = xhr.responseText
        }
    };
    xhr.open('GET', 'time', true);
    xhr.send();
}
```

准备前端的 HTML 结构，包含数据显示区域和触发更新的按钮：

```html
<hr>
<div id="time"></div>
<br>
<button onclick="updateTime()">更新数据</button>
<script>
    updateTime()
</script>
```

最后，编写后端的 Servlet，用于接收请求并将当前时间格式化后作为响应文本返回：

```java
@WebServlet("/time")
public class TimeServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy年MM月dd日 HH:mm:ss");
        String date = dateFormat.format(new Date());
        resp.setContentType("text/html;charset=UTF-8");
        resp.getWriter().write(date);
    }
}
```

## 2. GET 请求传递参数

GET请求也能传递参数，这里做一下演示。