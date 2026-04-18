# 用于Lambda的形参局部变量语法

在Java 10我们认识了`var`关键字，它能够直接让局部变量自动进行类型推断，不过它不支持在lambda中使用：

![image-20230306180413626](https://s2.loli.net/2023/03/06/uaNSkgeOUQTxoLl.png)

但是实际上这里是完全可以进行类型推断的，所以在Java 11，终于是支持了，这样编写就不会报错了：

![image-20230306180421523](https://s2.loli.net/2023/03/06/Nft9Csk6ac8AgY2.png)