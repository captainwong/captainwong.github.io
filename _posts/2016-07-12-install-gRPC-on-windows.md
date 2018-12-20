---
layout: post
title:  "Windows下gRPC安装小记"
subtitle: "在坑里 再次打滚的经历。。。"
date:   "2016-07-12" 
author: "cj"
tags:
    proto3
    protobuf
    gRPC
    windows
---

 搞定了 proto3后，开始搞 gRPC。又是一个巨坑。

从 github clone 项目之后，在 vsprojects 中打开 grpc.sln，编译 gpr, gprc, gprc_unsecure, gprc++_unsecure 几个项目，生成 lib 库。因为用不了 boringsll，只能用未加密的版本。期间小坑无数，每个项目配置都得是 Mt 或 Mtd。
再打开 grpc_protoc_plugin.sln，编译 grpc_plugin_support, gprc_cpp_plugin 两个项目。会生成 grpc_cpp_plugin.exe 文件，可以放在 protoc 的路径下，并加到系统路径中，方便使用。

搞定前提条件后，到 example/protos 文件夹中，先生成 protoc 的 pb：

```cmd
protoc --cpp_out=. helloworld.proto
```

会生成helloworld.pb.h 和 helloworld.pb.cc两个文件。

再使用 grpc 插件生成 gprc 的 pb：

```cmd
protoc --grpc_out=. --plugin=protoc-gen-grpc="your-grpc-cpp-plugin-exe-path" helloworld.proto
```

生成 helloworld.grpc.pb.h 和 helloworld.grpc.pb.cc 两个文件。

将上述4个文件添加到新建的示例工程中，在工程属性的 c/c++ Additional Include Directories 中添加 protobuf3库中的 src 文件夹路径和 grpc 库中 include 文件夹路径。在 Additional Library Directories 中添加 protobuf 和 grpc 中 vsprojects 的输出路径，或 debug 或 release 。在 Linker->Input 的 Additional Dependencies 中添加如下 lib 库：

```
libprotobuf.lib
z.lib
gpr.lib
grpc_unsecure.lib
grpc++_unsecure.lib
```

编译，我去又是一堆链接错误。不怕！至少说明代码没问题就是配置的事（废话，Google 的示例代码有什么问题）。分析示例项目的链接错误，一般都是 unresolved external symbol，意思就是头文件里有声明，但找不到实现。按照我上一篇配置 proto3的经验，继续找没加到工程中的 c 或 cc 文件。 

这些都搞定后，还有两个坑。。。
一个是示例项目的属性必须设置为 Mtd 或 Mt，一个是如果项目使用了预编译头，要在生成的 pb.cc 文件的第一行非注释行添加

```c
#include "stdafx.h"
```

OK，我什么大风大浪没见过，美国的华莱士，我跟他谈笑风生啊！这点小困难怎么能难住我咧！继续研究另一个示例项目 route_guid 去也。
