# JShell交互式编程

Java 9为我们通过了一种交互式编程工具JShell，你还别说，真有Python那味。

![image-20230306180136996](https://s2.loli.net/2023/03/06/HhrVDqeOwPZ6lvS.png)

环境配置完成后，我们只需要输入`jshell`命令即可开启交互式编程了，它支持我们一条一条命令进行操作。

比如我们来做一个简单的计算：

![image-20230306180146794](https://s2.loli.net/2023/03/06/BYnUL5WmTgavrS6.png)

我们一次输入一行（可以不加分号），先定义一个a=10和b=10，然后定义c并得到a+b的结果，可以看到还是非常方便的，但是注意语法还是和Java是一样的。

![image-20230306180158288](https://s2.loli.net/2023/03/06/NM7ruqzwX34poG2.png)

我们也可以快速创建一个方法供后续的调用。当我们按下Tab键还可以进行自动补全：

![image-20230306180220301](https://s2.loli.net/2023/03/06/1Yy7DHoPdOjV8L5.png)

除了直接运行我们写进去的代码之外，它还支持使用命令，输入`help`来查看命令列表：

![image-20230306180228542](https://s2.loli.net/2023/03/06/k9aUe5QXbJfmZDr.png)

比如我们可以使用`/vars`命令来展示当前定义的变量列表：

![image-20230306180242109](https://s2.loli.net/2023/03/06/z7uTFCqdxgfHYb5.png)

当我们不想使用jshell时，直接输入`/exit`退出即可：

![image-20230306180252071](https://s2.loli.net/2023/03/06/i6gw4cRtvdufzGq.png)