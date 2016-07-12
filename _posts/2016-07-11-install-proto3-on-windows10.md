---
layout: post
title:  "Windows下protobuf3安装小记"
subtitle: "在坑里打滚的经历。。。"
date:   "2016-07-08" 
author: "cj"
tags:
    -proto3
    -protobuf
    -windows
---






一开始搞的是2，可惜我要用的gRPC只能用proto3.在vsprojects文件夹内打开sln即可。
使用visual studio 2015，需在config.h添加

'''c++
#define _SILENCE_STDEXT_HASH_DEPRECATION_WARNINGS
#define _SCL_SECURE_NO_WARNINGS
'''

protoc还没有正式支持windows下编译，下载了编译好的[protoc3.0.0-beta-3-win32.zip](https://github.com/google/protobuf/releases/download/v3.0.0-beta-3/protoc-3.0.0-beta-3-win32.zip)。
编译示例addressbook.proto：

'''
protoc --cpp_out=. adressbook.proto
'''

成功生成了addressbook.pb.h和addressbook.pb.cc两个文件，但是需要的libprotobuf.lib还是得自己编译。

从github下载的项目没有了vsprojects文件夹，只能自己想办法。

我将proto2的vsprojects文件夹下的sln/vcxproj/filters等vs需要的文件拷贝过来，成功编译出了libprotobuf.lib。

接着再回来编译示例程序，提示一堆链接错误。仔细看那些链接错误，原来是proto2的vcxproj项目配置里没有proto3新增的一些文件。

按照build output，一个一个地将需要的cc文件添加到libprotobuf工程中，最终成功编译了示例程序。
