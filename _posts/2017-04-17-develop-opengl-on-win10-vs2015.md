---
layout: post
title:  "win10+vs2015下OpenGL开发环境搭建"
subtitle: "操蛋"
date:   "2017-04-17" 
author: "cj"
tags:
    opengl
    c++
    vs
    windows
    linux
---

之前在实验楼学了一个[C++实现太阳系行星系统](https://www.shiyanlou.com/courses/558)的课程，在他的web虚拟机中死活运行失败
![img](http://115.231.175.17/img/solar_system_execute_error.png)
所以试着在熟悉的环境下编译一次。
搜索了不少资料，发现opengl这玩意有点过时，而且不像其他开源库那样有源码，只能通过一些第三方的库实现。

首先记录一下ubuntu下开发环境搭建过程，so easy:

{% highlight Shell %}
sudo apt-get install freeglut3 freeglut3-dev
{% endhighlight %}

项目代码参见[solarsystem](https://github.com/captainwong/shiyanlou_cpp/tree/master/solarsystem)

Windows下环境搭建，有点扯淡，不过还好我身经百战，这种事见的多了。。。

1.新建一个文件夹存放所有所需的头文件和dll/lib文件
目录结构类似
![img](http://115.231.175.17/img/opengl_file_structure.png)

2.下载[glut](http://freeglut.sourceforge.net/index.php#download)
版本不要选3.0.0，那个没有vs的sln文件，下载2.8.1版本的。
在VisualStudio/2012文件夹内打开sln，按照提示迁移工程到vs2015版本，编译，将include内所有头文件拷贝到上述GL目录内，将生成的dll/lib文件拷贝到对应的Debug或Release目录内

3.下载[glew](http://glew.sourceforge.net/)
打开build/vc12/glew.sln，升级工程，编译，并拷贝头文件与dll/lib文件，同上。

4.当然这种事干一次就够了
可以到[opengl_win10_vs2015](https://github.com/captainwong/opengl_win10_vs2015)下载opengl.7z文件，已经包含了include和lib

5.运行效果
![img](http://115.231.175.17/img/solar_system.png)
动态图：
![img](http://115.231.175.17/img/solar_system.gif)

后来呢，发现在Ubuntu下不能运行的原因是这一句：

{% highlight c++ %}
glutInitDisplayMode(GLUT_RGBA | GLUT_DOUBLE);
{% endhighlight %}

我写成了

{% highlight c++ %}
glutInitDisplayMode(GL_RGBA | GL_DOUBLE);
{% endhighlight %}

这破问题断断续续折腾了好几个小时。。。。。。。。。。。。。。

