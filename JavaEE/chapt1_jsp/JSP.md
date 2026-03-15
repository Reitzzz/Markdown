## JSP页面核心结构拆解

### 结构一：JSP页面指令
这是JSP文件的标准头部配置，用于定义页面的全局属性。

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
```
**编写指南：**
* 这行代码声明了当前页面使用Java语言。
* `contentType`和`charset`设置为`UTF-8`是为了确保浏览器正确解析中文字符，防止页面出现乱码。
* 这行代码必须放在JSP文件的最顶端。

### 结构二：HTML基本骨架
在页面指令下方，是标准的HTML文档结构。

```html
<html>
<head>
    <title>页面标题</title>
</head>
<body>

</body>
</html>
```
**编写指南：**
* `<title>`标签用于定义在浏览器标签页上显示的页面名称。
* 所有的页面内容都需要写在`<body>`标签内部。

### 结构三：表单结构与输入控件
表单是实现用户和后端服务器数据交互的核心结构，在注册、登录和功能输入页面中广泛使用。可以将它看作一个“填空套公式”的模板，只要填对属性和值，就能实现所需功能。

**1. 表单容器 (大外壳)**
```html
<form action="Servlet地址" method="post">

</form>
```
**编写指南与属性字典**：
* **`action` (去哪里)**：决定用户点击“提交”后，数据发给哪个后端地址（如特定的Servlet）。
* **`method` (怎么去)**：指定数据传输方式。默认无脑使用 `"post"`，数据隐藏在请求体中，安全且适合传大量数据或密码。
* **`enctype` (特殊打包)**：如果表单内部包含文件上传功能（`type="file"`），必须且只能在 `<form>` 标签中添加 `enctype="multipart/form-data"` 属性。

**2. 常见输入控件**

* **文本框与密码框**：
```html
用户名：<input type="text" name="username"><br>
密码：<input type="password" name="password"><br>
```
**编写指南**：`type="text"` 接收明文，`type="password"` 将输入转为黑点或星号。**核心规则**：`name` 属性（如 `"username"`）是极其重要的“接头暗号”，后端完全靠这个名字来抓取对应的值。

* **单选框 (互斥选择)**：
```html
性别：
<input type="radio" name="gender" value="男">男
<input type="radio" name="gender" value="女">女<br>
```
**编写指南**：`type="radio"` 实现单选。**核心规则**：同一组的所有选项，`name` 必须完全一致（如此处的 `"gender"`）。`value` 是用户选中该项后，真正发送给后端的数据。

* **下拉列表**：
```html
专业：
<select name="major">
    <option value="计算机科学与技术">计算机科学与技术</option>
    <option value="软件工程">软件工程</option>
</select><br>
```
**编写指南**：`<select>` 定义整个下拉框，并通过 `name` 设定后端接头暗号（如 `"major"`）。`<option>` 提供选项，其 `value` 属性是选中后发送给后端的真实值。

* **复选框 (多项选择)**：
```html
特长：
<input type="checkbox" name="hobby" value="阅读">阅读
<input type="checkbox" name="hobby" value="运动">运动
<input type="checkbox" name="hobby" value="编程">编程<br>
```
**编写指南**：`type="checkbox"` 允许勾选多个。同组选项的 `name` 保持一致，后端接收时会得到一个包含多个 `value` 的数组。

* **文件上传**：
```html
个人照片：<input type="file" name="photo"><br>
```
**编写指南**：`type="file"` 会自动生成文件选择器。`name` 是后端接收该文件的变量名。**注意**：使用此控件时，最外层的 `<form>` 必须包含 `enctype` 属性。

* **提交按钮**：
```html
<input type="submit" value="提交注册">
```
**编写指南**：`type="submit"` 是表单的“发射器”，点击后立刻将 `<form>` 内所有带 `name` 的数据发往 `action` 地址。`value` 决定按钮表面显示的文字。
### 结构四：超链接与列表导航
首页通常通过超链接将各个独立的功能页面串联起来。

```html
<ol>
    <li><a href="URL地址">链接显示文本</a></li>
</ol>
```
**编写指南：**
* 使用`<a>`标签的`href`属性指定点击后跳转的目的地。
* 配合`<ol>`和`<li>`标签，将多个超链接排列成导航菜单。

### 结构五：动态内容与交互逻辑
JSP页面中可以嵌入动态数据渲染和简单的JavaScript交互。

```html
<div>${info}</div>
<img src="Response03" onclick="this.src='Response03?'+Math.random()">
```
**编写指南：**
* `${info}`属于EL表达式，用于接收并显示后端传递过来的动态数据。
* 对于验证码图片，给`<img>`标签绑定`onclick`点击事件，通过拼接`Math.random()`随机数参数实现点击刷新功能。