# 1. vim文本编辑器

## 目录
- [1. vim文本编辑器](#1-vim文本编辑器)
  - [目录](#目录)
  - [1.1 Vim编辑器简介与启动](#11-vim编辑器简介与启动)
  - [1.2 Vim的三种默认模式](#12-vim的三种默认模式)
  - [1.3 编辑模式详解](#13-编辑模式详解)
  - [1.4 末行模式保存与退出](#14-末行模式保存与退出)
  - [1.5 命令模式常用命令](#15-命令模式常用命令)
  - [1.6 末行模式复杂命令（搜索与替换）](#16-末行模式复杂命令搜索与替换)
  - [1.7 可视化模式](#17-可视化模式)

---

## 1.1 Vim编辑器简介与启动

和Windows中的记事本一样，Linux中也有文本编辑器，叫做Vi编辑器，Ubuntu中内置了Vi编辑器的升级版Vim，我们这里就讲解Vim编辑器的使用。

我们可以直接输入`vim 文件名称`来使用Vim编辑器对文本文件进行编辑：

```sh
test@ubuntu-server:~$ vim hello.txt
```

进入编辑器之后，我们发现界面变成了：

```text
~                                                                                                                                                                                         
~                                                                                                   
~                                                                                                   
~                                                                                                   
~                                                                                                   
~                                                                                                   
~                                                                                                   
~                                                                                                   
~                                                                                                   
~                                                                                                   
~                                                                                                   
~                                                                                                   
"hello.txt" [New File]                                                            0,0-1         All
```

## 1.2 Vim的三种默认模式

这时我们直接输入内容是无法完成的，因为默认进入之后为`命令模式`，Vim编辑器默认有三种模式：

![img](https://s2.loli.net/2023/03/06/b3EQehwpU51TBsa.jpg)

* **命令模式**：此模式下可以输入任意的命令进行操作，所有的输入都被看做是命令输入，而不是文本编辑输入。
* **编辑模式**：此模式下输入的任何内容都会以文本编辑方式写入到文件中，就像我们直接在Windows的记事本中写内容一样。
* **末行模式**：此模式下用于输入一些复杂命令，会在最后一行进行复杂命令的输入。

## 1.3 编辑模式详解

在命令模式下，我们可以直接按下键盘上的`i`，此命令表示进行插入操作，会自动切换到编辑模式，这时可以看到最下方变为：

```text
~                                                                                                   
~                                                                                                   
~                                                                                                   
~                                                                                                   
~                                                                                                   
~                                                                                                   
~                                                                                                   
-- INSERT --                                                                      1,1           All
```

而这时我们所有的输入内容都可以直接写到文件中了，如果我们想回到命令模式，按下`Esc`键即可。

除了`i`以外，我们也可以按下`a`表示从当前光标所在位置之后继续写，与`i`不同的是，`i`会在光标之前继续写，`o`会直接跳到下一行，而`A`表示在当前行的最后继续写入，`I`表示在当前行的最前面继续写入。

这里我们随便粘贴一段文本信息进去（不要用Ctrl+V，Linux中没这操作，XShell右键点粘贴）：

```text
I was hard on people sometimes, probably harder than I needed to be. 
I remember the time when Reed was six years old, coming home, and I had just fired somebody that day.
And I imagined what it was like for that person to tell his family and his young son that he had lost his job. 
It was hard. 
But　somebody’s got to do it. 
I figured that it was always my job to make sure that the team was excellent, and if I didn’t do it, nobody was going to do it.
You always have to keep pushing to innovate.
Dylan could have sung protest songs forever and probably made a lot of money, but he didn’t. 
He had to move on, and when he did, by going electric in 1965, he alienated a lot of people.
```

## 1.4 末行模式保存与退出

在我们编辑完成之后，需要进入到末行模式进行文件的保存并退出，按下`:`进入末行模式，再输入`wq`即可保存退出。

## 1.5 命令模式常用命令

接着我们来看一些比较常用的命令，首先是命令模式下的光标移动命令：

* `^`   直接调到本行最前面
* `$`   直接跳到本行最后面
* `gg`  直接跳到第一行
* `[N]G`    跳转到第N行
* `[N]方向键`    向一个方向跳转N个字符

## 1.6 末行模式复杂命令（搜索与替换）

在末行模式下，常用的复杂命令有：

* `:set number`    开启行号
* `:w`  保存
* `:wq`或`:x` 保存并关闭
* `:q`  关闭
* `:q!` 强制关闭

我们可以输入`/`或是`?`在末行模式中使用搜索功能，比如我们要搜索单词`it`：

```text
/it  
```

接着会在文本中出现高亮，按`n`跳转到下一个搜索结果，`?`是从后向前搜索，`/`是从前向后搜索。

它还支持替换功能，但是使用起来稍微比较复杂，语法如下：

```text
:[addr]s/源字符串/目的字符串/[option]
```

`addr`表示第几行或是一个范围，`option`表示操作类型：

* `g`: globe,表示全局替换
* `c`: confirm,表示进行确认
* `p`: 表示替代结果逐行显示(Ctrl + L恢复屏幕)
* `i`: ignore,不区分大小写

比如我们要将当前行中的`it`全部替换为`he`，那么可以这样写：

```text
:s/it/he/g
```

## 1.7 可视化模式

实际上除了以上三种模式外，还有一种模式叫做可视化模式，按下键盘上的`v`即可进入，它能够支持选取一段文本，选取后，我们可以对指定段落的文本内容快速进行复制、剪切、删除、插入等操作，非常方便。在此模式下，我们可以通过上下左右键进行选取，以进入可视化模式时的位置作为基本位置，通过移动另一端来进行选取。

我们可以使用以下命令来对选中区域进行各种操作：

* `y`   复制选中区域
* `d`/`x`   剪切（删除）选中区域
* `p`  粘贴
* `u`  撤销上一步

当然，这些命令在命令模式下也可以使用，但是可视化模式下使用更适合一些。