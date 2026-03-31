**目录**

- [1. Axios异步请求](#1-axios异步请求)
  - [1.1 前端异步请求概念](#11-前端异步请求概念)
  - [1.2 Axios发起GET请求获取数据](#12-axios发起get请求获取数据)
  - [1.3 Axios发起POST请求提交表单数据](#13-axios发起post请求提交表单数据)
  - [1.4 Axios发起POST请求提交JSON数据](#14-axios发起post请求提交json数据)

---

# 1. Axios异步请求

前面我们讲解了如何向浏览器发送一个JSON格式的数据，那么我们现在来看看如何向服务器请求数据。

![img](https://s2.loli.net/2023/08/14/faYcVC6dpIOuJyA.png)

## 1.1 前端异步请求概念

这一部分，我们又要回到前端相关内容的介绍中，当然，我们依然是仅做了解，并不需要详细学习前端项目开发知识。

前端为什么需要用到异步请求，这是因为我们的网页是动态的（这里的动态不是指有动画效果，而是能够实时更新内容）比如我们点击一个按钮会弹出新的内容、或是跳转到新的页面、更新页面中的数据等等，这些都需要通过JS完成异步请求来实现。

> 前端异步请求指的是在前端中发送请求至服务器或其他资源，并且不阻塞用户界面或其他操作。在传统的同步请求中，当发送请求时，浏览器会等待服务器响应，期间用户无法进行其他操作。而异步请求通过将请求发送到后台，在等待响应的同时，允许用户继续进行其他操作。这种机制能够提升用户体验，并且允许页面进行实时更新。常见的前端异步请求方式包括使用XMLHttpRequest对象、Fetch API、以及使用jQuery库中的AJAX方法，以及目前最常用的Axios框架等。

## 1.2 Axios发起GET请求获取数据

假设我们后端有一个需要实时刷新的数据（随时间而变化）现在需要再前端实时更新展示，这里我们以axios框架的简单使用为例子，带各位小伙伴体验如何发起异步请求并更新我们页面中的数据。

首先是前端页面，直接抄作业就行：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>测试</title>
    <script src="https://unpkg.com/axios@1.1.2/dist/axios.min.js"></script>
</head>
<body>
<p>欢迎来到GayHub全球最大同性交友网站</p>
<p>用户名: <span id="username"></span></p>
<p>密码: <span id="password"></span></p>
<button onclick="getInfo()">获取用户名和密码</button>
</body>
</html>

<script>
    function getInfo() {
        axios.get('/mvc/test').then(({data}) => {
            document.getElementById('username').innerText = data.username
            document.getElementById('password').innerText = data.password
        })
    }
</script>
```

接着我们使用axios框架直接对后端请求JSON数据：

```html
<script>
    function getInfo() {
        axios.get('/mvc/test').then(({data}) => {
            document.getElementById('username').innerText = data.username
            document.getElementById('password').innerText = data.password
        })
    }
</script>
```
之后再在Controller里编写对应逻辑
```java
@Controller

public class HelloController {

    @GetMapping("/")
    public String index(){
        return "login";
    }


    @ResponseBody
    @RequestMapping(value = "/test")
    public User test(){
        return new User("test","123456");
    }
}
```
这样，我们就实现了从服务端获取数据并更新到页面中，前端开发者利用JS发起异步请求，可以实现各种各样的效果，而我们后端开发者只需要关心接口返回正确的数据即可，这就已经有前后端分离开发的雏形了（实际上之前，我们在JavaWeb阶段使用XHR请求也演示过，不过当时是纯粹的数据）

## 1.3 Axios发起POST请求提交表单数据

那么我们接着来看，如何向服务端发送一个JS对象数据并进行解析，这里以简单的登录为例：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>测试</title>
    <script src="[https://unpkg.com/axios@1.1.2/dist/axios.min.js](https://unpkg.com/axios@1.1.2/dist/axios.min.js)"></script>
</head>
<body>
  <p>欢迎来到GayHub全球最大同性交友网站</p>
  <button onclick="login()">立即登录</button>
</body>
</html>
```

这里依然使用axios发送POST请求：

```html
<script>
    function login() {
        axios.post('/mvc/test', {
            username: 'test',
            password: '123456'
        }, {
            headers: {
                'Content-Type': 'application/x-www-form-urlencoded'
            }
        }).then(({data}) => {
            if(data.success) {
                alert('登录成功')
            } else {
                alert('登录失败')
            }
        })
    }
</script>
```

服务器端只需要在请求参数位置添加一个对象接收即可（和前面是一样的，因为这里也是提交的表单数据）：

```java
@ResponseBody
@PostMapping(value = "/test", produces = "application/json")
public String hello(String username, String password){   //Spring默认解析表单数据 直接使用json会导致失败
    boolean success = "test".equals(username) && "123456".equals(password);
    JSONObject object = new JSONObject();
    object.put("success", success);
    return object.toString();
}
```

## 1.4 Axios发起POST请求提交JSON数据

我们也可以将js对象转换为JSON字符串的形式进行传输，这里需要使用ajax方法来处理：

```html
<script>
    function login() {
        axios.post('/mvc/test', {
            username: 'test',
            password: '123456'
        }).then(({data}) => {
            if(data.success) {
                alert('登录成功')
            } else {
                alert('登录失败')
            }
        })
    }
</script>
```

如果我们需要读取前端发送给我们的JSON格式数据，那么这个时候就需要添加`@RequestBody`注解：

```java
@ResponseBody
@PostMapping(value = "/test", produces = "application/json")
public String hello(@RequestBody User user){
    boolean success = "test".equals(user.getUsername()) && "123456".equals(user.getPassword());
    JSONObject object = new JSONObject();
    object.put("success", success);
    return object.toString();
}
```

这样，我们就实现了前后端使用JSON字符串进行通信。