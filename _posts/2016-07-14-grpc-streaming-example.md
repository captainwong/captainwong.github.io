---
layout: post
title:  "hellostreamingworld 示例实现"
subtitle: "test"
date:   "2016-07-15" 
author: "cj"
tags:
    gRPC
---

 文件夹examples/protos 内有个 hellostreamingworld.proto，但是 examples/cpp 内没有对应的示例代码。我参考 route_guide 和 helloworld 两个示例，写出了这份程序。

1. 生成cc 文件
{% highlight Shell %}
protoc --cpp_out=. hellostreamingworld.proto 
{% endhighlight %}

{% highlight Shell %}
protoc --grpc_out=. --plugin=protoc-gen-grpc="path/to/cpp-plugin.exe" hellostreamingworld.proto
{% endhighlight %}

2. 建立 grpc_streaming_greeter_client 工程
添加第1步生成的4个文件、设置工程属性、添加引用库路径、包含库等设置步骤已在[Windows 下 gRPC 安装小记](http://wangyapeng.net/2016/07/12/install-gRPC-on-windows/)一文写出，不再赘述。
[查看代码](https://github.com/captainwong/AlarmCenter/blob/ipc/grpc_streaming_greeter_client/gprc_streaming_greeter_client.cpp)

3. 建立 grpc_streaming_greeter_server 工程，项目设置同上。
[查看代码](https://github.com/captainwong/AlarmCenter/blob/ipc/grpc_streaming_greeter_server/grpc_streaming_greeter_server.cpp)

4. 测试
先后运行 server 与 client 程序，client 输出3行 "Greeter received: hello world"，验证无误。

