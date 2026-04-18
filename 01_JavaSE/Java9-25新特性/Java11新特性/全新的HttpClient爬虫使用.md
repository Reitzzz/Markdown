# 1. 全新的HttpClient使用

- [1. 全新的HttpClient使用](#1-全新的httpclient使用)
  - [1.1 基本请求发送](#11-基本请求发送)
  - [1.2 编写简易爬虫获取网页](#12-编写简易爬虫获取网页)
  - [1.3 提取网页中的图片地址](#13-提取网页中的图片地址)
  - [1.4 下载图片并保存到本地](#14-下载图片并保存到本地)

---

## 1.1 基本请求发送

在Java 9的时候其实就已经引入了全新的Http Client API，用于取代之前比较老旧的HttpURLConnection类，新的API支持最新的HTTP2和WebSocket协议。

```java
public static void main(String[] args) throws URISyntaxException, IOException, InterruptedException {

    HttpClient client = HttpClient.newHttpClient();   
    HttpRequest request = HttpRequest.newBuilder().uri(new URI("https://www.baidu.com")).build();
    //直接创建一个新的HttpClient
    //现在我们只需要构造一个Http请求实体，就可以让客户端帮助我们发送出去了（实际上就跟浏览器访问类似）
    

    HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
    //现在我们就可以发送请求了
    // 注意send方法后面还需要一个响应体处理器BodyHandlers（内置了很多）这里我们选择ofString直接把响应实体转换为String字符串
    
    System.out.println(response.body());
}
```

## 1.2 编写简易爬虫获取网页

利用全新的客户端，我们甚至可以轻松地做一个爬虫（仅供学习使用，别去做违法的事情，爬虫玩得好，牢饭吃到饱），比如现在我们想去批量下载某个网站的壁纸：

网站地址：https://pic.netbian.com/4kmeinv/

我们随便点击一张壁纸，发现网站的URL格式为：

![image-20230306180458933](https://s2.loli.net/2023/03/06/BxUmcfP2d7F3Luy.png)

并且不同的壁纸似乎都是这样：https://pic.netbian.com/tupian/数字.html，好了差不多可以开始整活了：

```java
public static void main(String[] args) throws URISyntaxException, IOException, InterruptedException {
    HttpClient client = HttpClient.newHttpClient();
    for (int i = 0; i < 10; i++) {  
        HttpRequest request = HttpRequest.newBuilder().uri(new URI("https://pic.netbian.com/tupian/"+(29327 + i)+".html")).build();  //这里我们按照规律，批量获取
        HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
        System.out.println(response.body());  //这里打印一下看看网页
    }
}
```

可以看到，最后控制台成功获取到这些图片的网站页面了：

![image-20230306180509828](https://s2.loli.net/2023/03/06/gI1ker9wuKfWZ64.png)

## 1.3 提取网页中的图片地址

接着我们需要来观察一下网站的HTML具体怎么写的，把图片的地址提取出来：

![image-20230306180647473](https://s2.loli.net/2023/03/06/koQX2LCjhVU1EZt.png)

好了，知道图片在哪里就好办了，直接字符串截取：

```java
public static void main(String[] args) throws URISyntaxException, IOException, InterruptedException {
    HttpClient client = HttpClient.newHttpClient();
    for (int i = 0; i < 10; i++) {
        ...
        String html = response.body();
        
        String prefix = "<a href=\"\" id=\"img\"><img src=\"";  //先找好我们要截取的前面一段，作为前缀去匹配位置
        String suffix = "\" data-pic=";   //再找好我们要截取的屁股后面紧接着的位置，作为后缀去匹配位置
        //直接定位，然后前后截取，得到最终的图片地址
        html = html.substring(html.indexOf(prefix) + prefix.length());
        html = html.substring(0, html.indexOf(suffix));
        System.out.println(html);  //最终的图片地址就有了
    }
}
```

## 1.4 下载图片并保存到本地

好了，现在图片地址也可以批量拿到了，直接获取这些图片然后保存到本地吧：

```java
public static void main(String[] args) throws URISyntaxException, IOException, InterruptedException {
    HttpClient client = HttpClient.newHttpClient();
    for (int i = 0; i < 10; i++) {
        ...
                //创建请求，把图片取到
        HttpRequest imageRequest = HttpRequest.newBuilder().uri(new URI("[https://pic.netbian.com](https://pic.netbian.com)"+html)).build();
        //这里以输入流的方式获取，不过貌似可以直接下载文件，各位小伙伴可以单独试试看
        HttpResponse<InputStream> imageResponse = client.send(imageRequest, HttpResponse.BodyHandlers.ofInputStream());
        //拿到输入流和文件输出流
        InputStream imageInput = imageResponse.body();
        FileOutputStream stream = new FileOutputStream("images/"+i+".jpg"); //一会要保存的格式
        try (stream;imageInput){  //直接把要close的变量放进来就行，简洁一些了
            int size;   //下面具体保存过程的不用我多说了吧
            byte[] data = new byte[1024];  
            while ((size = imageInput.read(data)) > 0) {  
                stream.write(data, 0, size);
            }
        }
    }
}
```

我们现在来看看效果吧，美女的图片已经成功保存到本地了：

![image-20230306180720108](https://s2.loli.net/2023/03/06/AEV6Dpjogy2eInz.png)

当然，这仅仅是比较简单的爬虫，不过我们的最终目的还是希望各位能够学会使用新的HttpClient API。