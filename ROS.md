#安装ROS
###一、实验概要
随着机器人领域的快速发展和复杂化，代码的复用性和模块化的需求原来越强烈，而已有的开源机器人系统又不能很好的适应需求。2010年Willow Garage公司发布了开源机器人操作系统ROS（robot operating system），很快在机器人研究领域展开了学习和使用ROS的热潮。

ROS(Robot Operating System）是一个机器人软件平台，它能为异质计算机集群提供类似操作系统的功能。ROS的前身是斯坦福人工智能实验室为了支持斯坦福智能机器人STAIR而建立的交换庭(switchyard）项目。到2008年，主要由威楼加拉吉继续该项目的研发。

ROS是开源的，是用于机器人的一种后操作系统，或者说次级操作系统。它提供类似操作系统所提供的功能，包含硬件抽象描述、底层驱动程序管理、共用功能的执行、程序间的消息传递、程序发行包管理，它也提供一些工具程序和库用于获取、建立、编写和运行多机整合的程序。
###二、实验过程
1.添加软件源到source.list

    sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'

2.设置密匙

    sudo apt-key adv --keyserver hkp://ha.pool.sks-keyservers.net:80 --recv-key 0xB01FA116

3.命令行安装ROS（这里有很多版本可以选择进行安装，这里选择桌面完整版）

    sudo apt-get update
    sudo apt-get install ros-jade-desktop-full

4.初始化rosdep

    sudo rosdep init
    rosdep update

5.设置环境

    echo "source /opt/ros/indigo/setup.bash" >> ~/.bashrc
    source ~/.bashrc

6.安装rosinstall（一个使用频繁非常方便的一个工具）

    sudo apt-get install python-rosinstall

###三、实验感想
ROS提供一些标准操作系统服务，例如硬件抽象，底层设备控制，常用功能实现，进程间消息以及数据包管理。ROS是基于一种图状架构，从而不同节点的进程能接受，发布，聚合各种信息（例如传感，控制，状态，规划等等）。目前ROS主要支持Ubuntu。

ROS的功能特别强大，主要用在机器人以及3D构建上，我们可以用于创建地图，机器人通过ROS发布数据，rviz订阅消息接收数据，然后显示，这些数据也是有一定的数据格式的。

![](https://github.com/Valesail1/ES2016_14353414/blob/master/ROS1.png)