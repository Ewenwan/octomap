OctoMap - An Efficient Probabilistic 3D Mapping Framework Based on Octrees.
===========================================================================

[论文](http://www2.informatik.uni-freiburg.de/~hornunga/pub/hornung13auro.pdf)


http://octomap.github.io

Originally developed by Kai M. Wurm and Armin Hornung, University of Freiburg, Copyright (C) 2009-2014.
Currently maintained by [Armin Hornung](https://github.com/ahornung).
See the [list of contributors](octomap/AUTHORS.txt) for further authors.

License: 
  * octomap: [New BSD License](octomap/LICENSE.txt)
  * octovis and related libraries: [GPL](octovis/LICENSE.txt)


Download the latest releases:
  https://github.com/octomap/octomap/releases

API documentation:
  http://octomap.github.com/octomap/doc/
  
Build status: 
  [![Build Status](https://travis-ci.org/OctoMap/octomap.png?branch=devel)](https://travis-ci.org/OctoMap/octomap)
  
Report bugs and request features in our tracker:
  https://github.com/OctoMap/octomap/issues

A list of changes is available in the [octomap changelog](octomap/CHANGELOG.txt)


##  octomap原理
	1. 八叉树
	有八个子节点的树！是不是很厉害呢？至于为什么要分成八个子节点，
	想象一下一个正方形的方块的三个面各切一刀，不就变成八块了嘛！
	如果你想象不出来，请看下图： 切一刀->2块--> 再切一刀->4块-->再切一刀->8块  8卦
![](https://images2015.cnblogs.com/blog/606958/201512/606958-20151212140710419-2029480818.png)
	
	实际的数据结构呢，就是一个树根不断地往下扩，每次分成八个枝，直到叶子为止。
	叶子节点代表了分辨率最高的情况。例如分辨率设成0.01m，那么每个叶子就是一个1cm见方的小方块了呢！
	每个小方块都有一个数描述它是否被占据。在最简单的情况下，可以用0－1两个数表示（太简单了所以没什么用）。
	通常还是用0～1之间的浮点数表示它被占据的概率。0.5表示未确定，越大则表示被占据的可能性越高，反之亦然。
	由于它是八叉树，那么一个节点的八个孩子都有一定的概率被占据或不被占据啦！（下图是一棵八叉树）。
![](https://images2015.cnblogs.com/blog/606958/201512/606958-20151212142153278-792679245.png)
	
	用树结构的好处时：当某个节点的子结点都“占据”或“不占据”或“未确定”时，就可以把它给剪掉！
	换句话说，如果没必要进一步描述更精细的结构（孩子节点）时，我们只要一个粗方块（父节点）的信息就够了。
	这可以省去很多的存储空间。因为我们不用存一个“全八叉树”呀！
	
	2.　八叉树的更新
	在八叉树中，我们用概率来表达一个叶子是否被占据。为什么不直接用0－1表达呢？
	因为在对环境的观测过程中，由于噪声的存在，某个方块有时可能被观测到是“占据”的，
	过了一会儿，在另一些方块中又是“不占据”的。有时“占据”的时候多，有时“不占据”的时候多。
	这一方面可能是由于环境本身有动态特征（例如桌子被挪走了），另一方面（多数时候）可能是由于噪声。
	根据八叉树的推导，假设t＝1，…,T时刻，观测的数据为z1,…,zT，那么第n个叶子节点记录的信息为：
	
	p（n|z1:zT） = [  1+ (1-p(n|zT))/p(n|ZT) * (1-p（n|z1:zT-1）)/p（n|z1:zT-1） * p(n)/(1-p(n)) ]^(-1)
	
	
	logit 变换 把 0～1概率 映射到 全实数R空间 -无穷大 ～ +无穷大
![](https://upload.wikimedia.org/wikipedia/commons/thumb/c/c8/Logit.svg/350px-Logit.svg.png)

	p = 0~1  上图中的x
	a = logit(p) = log(p/(1-p))  范围为 -无穷大 ～ +无穷大
	 反过来可以得到
	  exp(a) = p/(1-p) ===>
	  exp(a) = p(1+exp(a)) ====>
	  p = exp(a)/(1+exp(a)) = 1/(1+exp(-a))  // sigmod(a) 神经网络激活函数
	     -无穷大 ～ +无穷大  映射为  0～1
        
	 我们对 p() 取logit 变换得到
	 L(P) = L(n|z1:zT) = L(n|z1:zT-1) + L(n|zT)  每一次的logit变换值 只是前面观测的 + 当前次概率的logit值
	 然后我们再对 logit值 求反变换 得到其概率值p!!!!!!!!!!!!!方便计算=========================

	每新来一个就直接加到原来的上面 
        此外还要加一个最大最小值的限制。最后转换回原来的概率即可。
	
	
	八叉树中的父亲节点占据概率，可以根据孩子节点的数值进行计算。比较简单的是取平均值或最大值。
	如果把八叉树按照占据概率进行渲染，不确定的方块渲染成透明的，
	确定占据的渲染成不透明的，就能看到我们平时见到的那种东西啦！
         octomap本身的数学原理还是简单的。不过它的可视化做的比较好。
	 
	 
	 
	 下载
	 https://github.com/Ewenwan/octomap
	 api 文档
	 http://octomap.github.io/octomap/doc/
	 
	 安装
	  mkdir build
	  cd build
	  cmake ..
	  make
	  
	  make 出错 可能是版本冲突， 与之前ros安装的版本不符合
	  一开始装的1.9 编译不通过
	  后 改成装 1.8 编译通过

	事实上，octomap的代码主要含两个模块：本身的octomap和可视化工具octovis。
	octovis依赖于qt4和qglviewer，所以如果你没有装这两个依赖，
	请安装它们：sudo apt-get install libqt4-dev qt4-qmake libqglviewer-dev

	如果编译没有给出任何警告，恭喜你编译成功！
	
	使用octovis查看示例地图
	在bin/文件夹中，存放着编译出来可执行文件。为了直观起见，我们直接看一个示例地图：

	bin/octovis octomap/share/data/geb079.bt

	 octovis会打开这个地图并显示。它的UI是长这样的。你可以玩玩菜单里各种东西（虽然也不多，我就不一一介绍UI怎么玩了），
	 能看出这是一层楼的扫描图。octovis是一个比较实用的工具，你生成的各种octomap地图都可以用它来看。
	 （所以你可以把octovis放到/usr/local/bin/下，省得以后还要找。）
	 





OVERVIEW
--------

OctoMap consists of two separate libraries each in its own subfolder:
**octomap**, the actual library, and **octovis**, our visualization libraries and tools.
This README provides an overview of both, for details on compiling each please 
see [octomap/README.md](octomap/README.md) and [octovis/README.md](octovis/README.md) respectively.
See http://www.ros.org/wiki/octomap and http://www.ros.org/wiki/octovis if you 
want to use OctoMap in ROS; there are pre-compiled packages available.

You can build each library separately with CMake by running it from the subdirectories, 
or build octomap and octovis together from this top-level directory. E.g., to
only compile the library, run:

    cd octomap
    mkdir build
    cd build
    cmake ..
    make
  
To compile the complete package, run:

    cd build
    cmake ..
    make
  
Binaries and libs will end up in the directories `bin` and `lib` of the
top-level directory where you started the build.


See [octomap README](octomap/README.md) and [octovis README](octovis/README.md) for further
details and hints on compiling, especially under Windows.
