---
title: Windows系统下ignite坑，启动找不到主类
date: 2017-07-06 10:34:34
tags: ignite
---
最近一直没来更新，惭愧
正好刚从ignite一个大坑出来，必须来记录下了

<!--more-->

正如我题目所说的，就是在windows系统下启动找不到主类的坑
因为项目原因，我接触了Apache ignite这个东西，官方说它是
`内存数据组织是高性能的、集成化的以及分布式的内存平台`。
具体有哪些东西，让我引一张官方的图：
![img1](https://files.readme.io/8fed3b7-apache-ignite.png)

因为我也是第一次接触，所以就按照[官方文档](https://ignite.apache.org/)一步一步来安装，学习
这里不得不说它们的[中文文档](https://www.zybuluo.com/liyuj/note/230739)棒棒哒！

我下载了2.0版本的二进制版本，放在了`C:\Program Files`目录下，然后安装新手教程进行启动，
在命令行到`bin`目录下运行ignite.bat文件，文档说的很清楚，出现以下提示就说明成功了：

	[02:49:12] Ignite node started OK (id=ab5d18a6)
	[02:49:12] Topology snapshot [ver=1, nodes=1, CPUs=8, heap=1.0GB]

但是我这确报错了。。。得到下面的提示

![img2](http://img.wqzhang.top/ignite07064.png)

我就奇了怪了，这还能找不到主类，对比了文件大小，和网上的数据是一样的
仔细看了下，上面提示说考虑把ignite-spring路径放到classpath

对比文档，这种错误应该是配置了ignite-spring模块但是没有引入ignite-spring相关包
只需要去lib下加入ignite-spring的相关jar依赖就行了

但是！

![img3](http://img.wqzhang.top/ignite07063.png)

我的lib下已经加入了ignite-spring包仍然报错。。。。。这。。扎心了，老铁
无奈的我只能去加了几个QQ群问问大神。。。都表示没遇到过这个错误
这个时候有个朋友说了句会不会是空格的问题，刚开始我还没意识到他的意思，
我什么都还没开始，配置也没动过，哪里有空格呢？
后来才看到。。。。我的项目存放的路径 `C:\Program Files` 
妹的。。不会吧。。。。我试这把项目拷贝到C盘下，重新配置Path变量，再尝试

![img4](http://img.wqzhang.top/ignite07065.png)

终于正常了，果然问题是出在那个空格上，导致读不到相关的模块依赖而报错。。。
而我一直在这个问题上困扰了好久。。。知道真相后，我斯巴达了。。。
后面继续学习了，这个坑。。应该是我自己给自己挖的坑吧。。。。
好了，后面学习的其他问题也回来更新的，鼓捣拜