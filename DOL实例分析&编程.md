#**第一次报告**   
<br/>
###一、DOL实例分析&编程
这次实验主要是通过修改example代码内容来达到对应的目的。
<br/>
<br/>
###二、实验截图
1.首先我们运行未修改的example1，得到的结果如下图。

![](https://github.com/Valesail1/ES2016_14353414/blob/master/dol1.png)

得到的.dot图如下图所示，可以看出其中的连接方式。

![](https://github.com/Valesail1/ES2016_14353414/blob/master/dol2.png)

然后我们修改square.c文件，将其中的平方编程三次方。

    i = i*i*i;

我们删掉build/bin/main/example1文件夹，重新编译运行example1，会得到如下的结果。

![](https://github.com/Valesail1/ES2016_14353414/blob/master/dol3.png)

所有的结果都由平方变成了三次方，example1修改成功。

2.我们再运行example2的结果，得到的记过如下图所示。

![](https://github.com/Valesail1/ES2016_14353414/blob/master/dol4.png)

所有的结果都是输入的八次方，这是为什么呢，我们看看.dot图

![](https://github.com/Valesail1/ES2016_14353414/blob/master/dol5.png)

可以看出square一共迭代了三次，也就是说每个square算了输入的平方，三个就是i的八次方。
我们修改example2.xml文件的迭代次数

    <variable value="2" name="N"/>

修改成2，之后就只迭代两次了，这时候我们再看输出结果。

![](https://github.com/Valesail1/ES2016_14353414/blob/master/dol6.png)

再看看.dot文件

![](https://github.com/Valesail1/ES2016_14353414/blob/master/dol7.png)

可以看到square的迭代次数由三次变成了两次，example2修改成功。
<br/>
<br/>
###三、实验心得
在本次实验中，重点是在对各部分以及各部分之间的链接的理解。我们输入作为一个部分，输出作为一个部分，中间还有一个负责将输入转化为输出的一个部分，然后这些部分之间要通过connection相连才能够使他们之间的数据进行流通，从而达到从输入到输出的一条龙服务。