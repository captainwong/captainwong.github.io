---
layout: post
title:  "hellostreamingworld 示例实现"
subtitle: ""
date:   "2016-07-15" 
author: "cj"
tags:
    gRPC
---

 文件夹examples/protos 内有个 hellostreamingworld.proto，但是 examples/cpp 内没有对应的示例代码。我参考 route_guide 和 helloworld 两个示例，写出了这份程序。

# 1 生成cc 文件#
{% highlight Shell %}
protoc --cpp_out=. hellostreamingworld.proto 
{% endhighlight %}

{% highlight Shell %}
protoc --grpc_out=. --plugin=protoc-gen-grpc="path/to/cpp-plugin.exe" hellostreamingworld.proto
{% endhighlight %}

# 2 建立 grpc_streaming_greeter_client 工程#
添加第1步生成的4个文件、设置工程属性、添加引用库路径、包含库等设置步骤已在[Windows 下 gRPC 安装小记](http://wangyapeng.net/2016/07/12/install-gRPC-on-windows/)一文写出，不再赘述。
以下是 main.cpp 内容。

{% highlight c++ %}
#include <iostream>
#include <memory>
#include <string>
#include <grpc++/grpc++.h>
#include "hellostreamingworld.grpc.pb.h"

using grpc::Channel;
using grpc::ClientContext;
using grpc::Status;
using hellostreamingworld::HelloRequest;
using hellostreamingworld::HelloReply;
using hellostreamingworld::MultiGreeter;

class GreeterClient {
public:
	GreeterClient(std::shared_ptr<Channel> channel)
		: stub_(MultiGreeter::NewStub(channel)) {}

	// Assambles the client's payload, sends it and presents the response back
	// from the server.
	void SayHello(const std::string& user, const std::string& num_greetings) {
		// Data we are sending to the server.
		HelloRequest request;
		request.set_name(user);
		request.set_num_greetings(num_greetings);
		// Container for the data we expect from the server.
		//HelloReply reply;

		// Context for the client. It could be used to convey extra information to
		// the server and/or tweak certain RPC behaviors.
		ClientContext context;

		// The actual RPC.
		//Status status = stub_->AsyncsayHello(&context, request, &reply);

		auto reader = stub_->sayHello(&context, request);
		HelloReply reply;
		while (reader->Read(&reply)) {
			std::cout << "Greeter received: " << reply.message() << std::endl;
		}

		Status status = reader->Finish();
		if (status.ok()) {
			std::cout << "SayHello rpc succeeded." << std::endl;
		} else {
			std::cout << "SayHello rpc failed." << std::endl;
		}
	}

private:
	std::unique_ptr<MultiGreeter::Stub> stub_;
};



int main(int argc, char** argv) {
	// Instantiate the client. It requires a channel, out of which the actual RPCs
	// are created. This channel models a connection to an endpoint (in this case,
	// localhost at port 50051). We indicate that the channel isn't authenticated
	// (use of InsecureChannelCredentials()).
	GreeterClient greeter(grpc::CreateChannel("localhost:50051", grpc::InsecureChannelCredentials()));
	std::string user("world"), num("3");
	greeter.SayHello(user, num);
	std::system("pause");
	return 0;
}
{% endhighlight %}

# 3 建立 grpc_streaming_greeter_server 工程，项目设置同上。

{% highlight c++ %}
#include <iostream>
#include <memory>
#include <string>
#include <grpc++/grpc++.h>
#include "hellostreamingworld.grpc.pb.h"

class GreeterServiceImpl final : public hellostreamingworld::MultiGreeter::Service
{
	virtual ::grpc::Status sayHello(::grpc::ServerContext* context, 
									const ::hellostreamingworld::HelloRequest* request, 
									::grpc::ServerWriter< ::hellostreamingworld::HelloReply>* writer) {
		hellostreamingworld::HelloReply reply;
		std::string hello("hello ");
		auto num = std::stoi(request->num_greetings());
		while (num-- > 0) {
			reply.set_message(hello + request->name());
			if (!writer->Write(reply)) {
				break;
			}
		}
		return ::grpc::Status::OK;
	}
};


int main()
{
	std::string server_address("0.0.0.0:50051");
	GreeterServiceImpl service;
	grpc::ServerBuilder builder;
	builder.AddListeningPort(server_address, grpc::InsecureServerCredentials());
	builder.RegisterService(&service);
	std::unique_ptr<grpc::Server> server(builder.BuildAndStart());
	std::cout << "Server listening on " << server_address << std::endl;
	server->Wait();


    return 0;
}
{% endhighlight %}

# 4 测试 #
先后运行 server 与 client 程序，client 输出3行 "Greeter received: hello world"，验证无误。

