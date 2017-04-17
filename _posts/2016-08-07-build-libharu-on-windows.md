---
layout: post
title:  "Windows下libharu编译小记"
subtitle: "PDF神器"
date:   "2016-08-07" 
author: "cj"
tags:
    libharu
    libpng
    pdf
    windows
---


c++操作pdf，搜了很多资料，有很多库可以用，但最终决定使用libharu。

libharu依赖libpng，首先编译之，生成所需libpng16.lib文件。

在libhuru的工程属性中添加libpng的库目录与lib目录，编译即可。

libharu使用示例可参考[Player](https://github.com/captainwong/Player/blob/master/Player/printermgr.cpp)
