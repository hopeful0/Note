# 创建第一个Java命令行程序
---

write in *2016-06-01*

#### 概述
使用命令行编译并执行一个Java命令行程序

## 准备工作
---

使用自己的计算机做Java开发，首先要确保你的计算机正确的配置了Java开发环境，具体配置方法可以到网上找教程。

我会在空闲时间在[Java](../README.md)目录下更新一篇关于Java开发环境的笔记。

在配置好Java开发环境后，你还需要合适的文本编辑器与一些命令行知识。

在**Windows**环境下，自带的**Notepad**并不是一个合适的文本编辑器，推荐使用**Notepad++**，这是一款功能强大的文本编辑器，非常适合Windows环境下不使用IDE做开发。**CMD控制台**是进行编译与执行命令行工程的关键。

而在**Linux**环境下，**终端**+**Vim**可以认为是标配，当然不熟悉`Vim`的朋友可以使用**GNU nano**。

[Google Java编程风格指南](http://www.hawstein.com/posts/google-java-style.htm)

## 编写源代码文件
---

在工作目录下新建一个名为`Main.java`的文档，然后用你的文本编辑器打开。

首先需要在源代码文件中确定唯一一个public类，这个类必须与文件名同名。如果你的源代码文件没有public类，就不需要与文件名同名，但是这里必须要有这个public类。

```Java
public class Main {

}
```

然后，Java程序还需要一个入口函数——main函数，其结构形式如下。

```Java
public static void main(String[] args) {

}
```

将这个函数放到Main类里面，然后函数内写上要实现的功能（如：打印字符串），源代码文件就编写完成了。

```Java
public class Main {

  public static void main(String[] args) {
    System.out.println("hello java!");
  }
  
}
```

上面代码中`System.out.println(String msg)`是Java中打印字符串并换行的函数。你可以自己替换其中的字符串，也可以写多个该函数来打印更多字符

还有注意代码中的缩进与换行，这可以使你的代码具有更清晰的结构，便于查看。

到这里源代码文件就编写完成了，保存并关闭你的文本编辑器即可。

## 编译源代码文件

下一步工作就是将".java"为后缀的源代码文件编译为".class"为后缀的字节码文件。

确保你的Java环境已经配置好，可以在CMD命令行或终端使用`java -version`得到你的jdk版本。

在你的工作目录打开命令行或终端，使用`javac`命令来编译文件。

```
javac Main.java
```

执行之后，你会发现工作目录下多了一个名为"Main.class"的文件，这就是我们编译得到的字节码文件了，是可以通过Java的解释器执行的文件。

## 执行字节码文件

最后就是运行程序了，用Java的解释器来解释其字节码文件即可。

```
java Main
```

如上，Main是不需要加后缀的。事实上当你输入`java`并按下空格后，直接按下`Tab`键，就会自动补全出`Main`或者列出当前目录下可执行的Java程序。

敲下回车，你就会发现，命令行中打印出了你希望看到的字符串，这样第一个Java命令行程序就完成了。


# END

# 评论

点击右侧加号，在这里留下你的评论。