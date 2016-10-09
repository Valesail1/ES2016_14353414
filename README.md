#**第一次报告**   
<br/>
###一、DOL 框架描述
####通过实验分布式操作层（DOL）是一个框架，基本上由三部分组成：
####1.应用程序编程
DOL定义了一组计算和通信程序，使对形状分布平台，并行应用程序的编程。使用这些程序，应用程序员可以编写程序，而无需了解底层架构的详细知识。事实上，这些程序都受限于硬件相关的软件（HDS）层进一步完善。
####2.仿真功能
提供程序员可能性测试其应用程序，功能仿真框架已经形成。除了应用功能验证，这个框架是用来在应用程序级获得的性能参数。
####3.映射优化
DOL映射优化的目标是计算一组应用程序的最佳映射到形状架构平台。在第一步骤中，基于XML规范格式已定义允许以描述一个抽象级别的应用程序和体系结构。尽管如此，所有必要的，以获得准确的性能估计包含信息。
<br/>
<br/>
###二、DOL安装心得
1.实验环境

我的电脑系统为windows8.1的64位系统，所用来跑虚拟机的软件为VMware Workstation 12 Player。

2.下载需要的文件并且解压到指定文件夹

3.编译systemc

进入systemc-2.3.1的目录下

    cd systemc-2.3.1

建立一个新建文件夹objdir

    mkdir objdir

进入该文件夹objdir

    cd objdir

运行configure

    ../configure CXX=g++ --disable-async-updates

![](https://github.com/Valesail1/ES2016_14353414/blob/master/1.png)

编译文件

    sudo make install

![](https://github.com/Valesail1/ES2016_14353414/blob/master/2.png)

然后查看所有的文件夹

    ls

![](https://github.com/Valesail1/ES2016_14353414/blob/master/3.png)

记录当前的工作路径，为home/zmm/systemc-2.3.1

![](https://github.com/Valesail1/ES2016_14353414/blob/master/4.png)

4.编译dol

这时候我们需要找到dol文件夹下面的build_zip.xml文件，将其中对应的地址改成上面我们所存储下来的地址

    cd ../dol

编译

    ant -f build_zip.xml all

![](https://github.com/Valesail1/ES2016_14353414/blob/master/5.png)

试运行第一个例子

进入build/bin/mian路径下

    cd build/bin/main
	ant -f runexample.xml -Dnumber=1

![](https://github.com/Valesail1/ES2016_14353414/blob/master/6.png)

<br/>
<br/>
###三、实验心得
在本次实验中，遇到了这么一个问题，在执行最后一步的时候，怎么都通不过，不知道具体原因，在网上查询过许多资料之后都不明白原因，换了很多台电脑，创建了很多新的虚拟机，最后终于在一台电脑上跑完PPT上所有的步骤之后成功的配置成功，感觉心特别累。
