---
layout: post
title:  "PHP下编写grpc客户端小记"
subtitle: "好嗨哟，感觉人生已经到达了高潮"
date:   "2018-12-26"
author: "cj"
tags:
    Laravel
    php
    gRPC
    protobuf
    c++
    ubuntu
    linux
---

# PHP下编写grpc客户端

## 目录

1. [环境](#环境)
2. [C++程序添加grpc服务端](#C++程序添加grpc服务端)
    1. [编写proto文件](#编写proto文件)
    2. [编译proto生成所需c++文件](#编译proto生成所需c++文件)
    3. [编写调用代码](#编写调用代码)
    4. [编写测试端](#编写测试端)
    5. [编写Makefile](编写Makefile)
    6. [启动测试](启动测试)
3. [编写PHP客户端](编写PHP客户端)
    1. [编译proto生成所需PHP文件](编译proto生成所需PHP文件)
    2. [编写客户端代码](编写客户端代码)
    3. [新增一个命令](新增一个命令)
    4. [添加计划任务](添加计划任务)
    5. [问题解决](问题解决)

## 1. 环境

参考[Ubuntu16.04.5环境编译grpc小记](http://wangyapeng.me/2018/12/19/build-grpc-on-ubuntu-16.04.5/)。

由c++程序作为grpc服务端，php作为grpc客户端。背景为c++程序作为报警信息来源，php定时向服务端发起询问：有没有状态发生变化的报警主机？如果有就获取更新的主机，信息中包含主机的报警信息。

## 2. C++程序添加grpc服务端

### 2.1 编写proto文件

`alarm_server.proto`

```protobuf
syntax = "proto3";

package alarm_server;

message GetMachineRequest {
    // machine account. empty means get all updated machines.
    string account=1;
}

message AlarmMessage {
    bool valid=1;
    int32 zone=2;
    int32 event=3;
    int64 timestamp=4;
    int32 source=5;
}

message MachineReply {
    string account=1;
    bool online=2;
    int32 type=3;
    int32 status=4;
    string password=5;
    AlarmMessage alarmMessage=6;
}

message IsNeedSyncRequest {
    string empty=1;
}

message IsNeedSyncReply {
    int32 changedMachineCount=1;
}

service AlarmServer {
    rpc IsNeedSync(IsNeedSyncRequest) returns (IsNeedSyncReply) {}
    rpc Sync(GetMachineRequest) returns (stream MachineReply) {}
}
```

### 2.2 编译proto生成所需c++文件

`gen_grpc.sh`

```bash
#!/bin/bash
CURRENT_DIR=$( cd "$(dirname "${BASH_SOURCE[0]}")" ; pwd -P )
GRPC_CPP_PLUGIN_PATH=`which grpc_cpp_plugin`

PROTOS_PATH=${CURRENT_DIR}/.
OUT_PATH=${CURRENT_DIR}/../rpc

protoc -I ${PROTOS_PATH} --grpc_out=${OUT_PATH} --plugin=protoc-gen-grpc=${GRPC_CPP_PLUGIN_PATH} ${CURRENT_DIR}/alarm_server.proto
protoc -I ${PROTOS_PATH} --cpp_out=${OUT_PATH} ${CURRENT_DIR}/alarm_server.proto
```

生成了`alarm_server.grpc.pb.cc`, `alarm_server.grpc.pb.h`, `alarm_server.pb.cc`, `alarm_server.pb.h`等文件。

### 2.3 编写调用代码

`rpc_server.h`

```cpp
#pragma once

#include <jlib/dp.h>
#include <memory>
#include <thread>
#include <mutex>
#include <condition_variable>

namespace grpc {
class Server;
}

class RpcServer : public jlib::dp::singleton<RpcServer>
{
protected:
    RpcServer();

public:
    void startup();
    void shutdown();

private:
    class AlarmServerImpl;
    std::shared_ptr<grpc::Server> server_ = {};

    bool running_ = false;
    bool shutdown_ok_ = false;

    std::thread thread_ = {};
    std::mutex mutex_ = {};
    std::condition_variable cv_ = {};
};
```

`rpc_server.cpp`

```cpp
#include "rpc_server.h"
#include <grpcpp/grpcpp.h>
#include "alarm_server.grpc.pb.h"
#include "../core/machine.h"
#include <jlib/log2.h>

class RpcServer::AlarmServerImpl final : public ::alarm_server::AlarmServer::Service
{
protected:
virtual ::grpc::Status IsNeedSync(::grpc::ServerContext* context,
            const ::alarm_server::IsNeedSyncRequest* request,
            ::alarm_server::IsNeedSyncReply* reply) override {
    reply->set_changedmachinecount(core::machine_mgr::get_instance()    ->getUpdatedMachineCount());
    return ::grpc::Status::OK;
}

virtual ::grpc::Status Sync(::grpc::ServerContext* context,
            const ::alarm_server::GetMachineRequest* request,
            ::grpc::ServerWriter<::alarm_server::MachineReply>* writer) override {
        static auto machine2reply = [](const core::machine_ptr& machine) {
        static auto machine2alarm_msg = [](const core::machine_ptr& machine, alarm_server::AlarmMessage* alarmMsg) {
        if (machine->has_alarm()) {
            alarmMsg->set_valid(true);
            alarmMsg->set_zone(machine->get_prev_alarm_exception_zone());
            alarmMsg->set_event(machine->get_prev_alarm_exception_event());
            alarmMsg->set_timestamp(std::chrono::system_clock::to_time_t(machine->get_prev_alarm_time()));

            core::machine_mgr::get_instance()->clear_machine_alarm(machine);
            } else {
                alarmMsg->set_valid(false);
            }
        };

        alarm_server::MachineReply reply;
        reply.set_account(machine->get_account());
        reply.set_online(machine->is_online());
        reply.set_type(machine->get_type());
        reply.set_status(machine->get_status());
        reply.set_password(machine->get_psw());
        machine2alarm_msg(machine, reply.mutable_alarmmessage());
        return reply;
    };

    if (request->account().empty()) {
        for (auto machine : core::machine_mgr::get_instance()->getUpdatedMachines()) {
        if (!writer->Write(machine2reply(machine))) {
            JLOG_ERRO("AlarmServerImpl::Sync failed");
            break;
        }
        }
    } else {
        auto machine = core::machine_mgr::get_instance()->get(request->account());
        if (machine) {
            if (!writer->Write(machine2reply(machine))) {
                JLOG_ERRO("AlarmServerImpl::Sync failed");
            }
        }
    }

    return ::grpc::Status::OK;
}

public:

};



RpcServer::RpcServer()
{

}

void RpcServer::startup()
{
    running_ = true;

    thread_ = std::thread([this]() {
        AUTO_LOG_FUNCTION;
        try {
            AlarmServerImpl service;
            std::string server_address("0.0.0.0:50051");
            ::grpc::ServerBuilder builder;
            builder.AddListeningPort(server_address, ::grpc::InsecureServerCredentials());
            builder.RegisterService(&service);
            std::shared_ptr<grpc::Server> server = builder.BuildAndStart();
            server_ = server;

            JLOG_INFO("RpcServer::AlarmServerImpl started on {}", server_address);

            server->Wait();

            {
                std::unique_lock<std::mutex> ul(mutex_);
                cv_.wait(ul, [this]() {return shutdown_ok_; });
            }
        } catch (...) {
            return;
        }
    });
}

void RpcServer::shutdown()
{
    AUTO_LOG_FUNCTION;
    running_ = false;

    {
        std::lock_guard<std::mutex> lg(mutex_);
        server_->Shutdown();
        server_ = nullptr;
        shutdown_ok_ = true;

        JLOG_INFO("server_ done shutdown");
    }

    cv_.notify_one();
    thread_.join();
    JLOG_INFO("thread_ done joined, RpcServer::shutdown OK");
}
```

然后在`main`函数中启动`rpc_server`：`RpcServer::get_instance()->startup();`

### 2.4 编写测试端

`test_grpc.cpp`

```cpp
#include <iostream>
#include <memory>
#include <string>
#include <chrono>
#include <ctime>
#include <thread>
#include <grpcpp/grpcpp.h>
#include "../wechat_server/rpc/alarm_server.grpc.pb.h"
#include "../wechat_server/core/machine.h"

using grpc::Channel;
using grpc::ClientContext;
using grpc::Status;
using namespace alarm_server;

class AlarmClient {
public:
    AlarmClient(std::shared_ptr<Channel> channel)
        : stub_(AlarmServer::NewStub(channel)) {}

    size_t IsNeedSync() {
        IsNeedSyncRequest request;
        IsNeedSyncReply reply;
        ClientContext context;

        Status status = stub_->IsNeedSync(&context, request, &reply);

        if(status.ok()){
            return reply.changedmachinecount();
        }else{
            std::cout << "IsNeedSync: " << status.error_code() << ": " << status.error_message() << std::endl;
            return 0;
        }
    }

    void sync(){
        GetMachineRequest request;
        ClientContext context;
        MachineReply reply;
        auto reader = stub_->Sync(&context, request);
        int i = 0;
        //std::cout << "--------------------sync started" << std::endl;
        while(reader->Read(&reply)){
            std::cout << "#" << ++i << " "
                << reply.account() << " "
                << (reply.online() ? "online " : "offline ")
                << core::machine_info::get_type_string(core::machine_info::integer2type(reply.type())) << " "
                << core::machine_info::get_status_string(core::machine_info::integer2status(reply.status())) << " "
                << reply.password() << " ";

            const auto& alarmMsg = reply.alarmmessage();
            if(alarmMsg.valid()){
                std::cout << "alarm message:" << " "
                    << alarmMsg.zone()  << " "
                    << alarmMsg.event()  << " ";
                auto t = static_cast<time_t>(alarmMsg.timestamp());
                std::tm * ptm = std::localtime(&t);
                char buffer[32];
                // Format: 2009-12-31 20:20:00
                std::strftime(buffer, 32, "%Y-%m-%d %H:%M:%S", ptm);
                std::cout << buffer << std::endl;
            }else{
                std::cout << "no alarm message" << std::endl;
            }
        }

        std::cout << std::endl;
        //std::cout << "sync finished-------------------" << std::endl << std::endl;
    }

    void sync(const std::string& account){
        GetMachineRequest request;
        request.set_account(account);
        ClientContext context;
        MachineReply reply;
        auto reader = stub_->Sync(&context, request);
        int i = 0;
        //std::cout << "--------------------sync started" << std::endl;
        while(reader->Read(&reply)){
            std::cout << "#" << ++i << " "
                << reply.account() << " "
                << (reply.online() ? "online " : "offline ")
                << core::machine_info::get_type_string(core::machine_info::integer2type(reply.type())) << " "
                << core::machine_info::get_status_string(core::machine_info::integer2status(reply.status())) << " "
                << reply.password() << " ";

            const auto& alarmMsg = reply.alarmmessage();
            if(alarmMsg.valid()){
                std::cout << "alarm message:" << " "
                    << alarmMsg.zone()  << " "
                    << alarmMsg.event()  << " ";
                auto t = static_cast<time_t>(alarmMsg.timestamp());
                std::tm * ptm = std::localtime(&t);
                char buffer[32];
                // Format: 2009-12-31 20:20:00
                std::strftime(buffer, 32, "%Y-%m-%d %H:%M:%S", ptm);
                std::cout << buffer << std::endl;
            }else{
                std::cout << "no alarm message" << std::endl;
            }
        }

        std::cout << std::endl;
    }

private:
    std::unique_ptr<AlarmServer::Stub> stub_;
};

int main()
{
    AlarmClient client(grpc::CreateChannel("localhost:50051", grpc::InsecureChannelCredentials()));

    std::cout << "Testing sync with account..." << std::endl;

    std::cout << "sync 70801150121796" << std::endl;
    client.sync("70801150121796");

    std::cout << "sync 70801151915015" << std::endl;
    client.sync("70801151915015");

    std::cout << "sync 70801163150828" << std::endl;
    client.sync("70801163150828");

    std::cout << "Testing sync without account..." << std::endl;
    while(true){
        auto cnt = client.IsNeedSync();
        if(cnt > 0){
            //std::cout << std::endl << cnt << " machines updated, syncing..."<< std::endl;
            client.sync();
        }else{
            //std::cout << "no machines updated"<< std::endl;
        }

        std::this_thread::sleep_for(std::chrono::seconds(1));
    }

    return 0;
}
```

### 2.5 编写Makefile

`Makefile`中与`grpc`编译相关部分：

```makefile
CPPFLAGS += `pkg-config --cflags protobuf grpc`
CXXFLAGS += -gdwarf -g -Wall -Wno-unused-variable -Wno-unused-function -std=c++11
CXX = g++

HOST_SYSTEM = $(shell uname | cut -f 1 -d_)
SYSTEM ?= $(HOST_SYSTEM)
ifeq ($(SYSTEM),Darwin)
LDFLAGS += -L/usr/local/lib `pkg-config --libs protobuf grpc++ grpc`\
            -lgrpc++_reflection \
            -ldl
else
LDFLAGS += -L/usr/local/lib `pkg-config --libs protobuf grpc++ grpc` \
            -Wl,--no-as-needed -lgrpc++_reflection -Wl,--as-needed \
            -ldl
endif

vpath %.proto $(PROTOS_PATH)

GRPC_SOURCES= $(PROJDIR)/wechat_server/rpc/alarm_server.pb.cc \
                $(PROJDIR)/wechat_server/rpc/alarm_server.grpc.pb.cc

GRPC_OBJS= $(patsubst %.cc,%.o,$(GRPC_SOURCES) )

grpc: system-check
    bash ../wechat_server/protos/gen_grpc.sh

%.o: %.cpp
    $(CXX) $(CXXFLAGS) -c $< -o $@ $(INCLUDES)

%.o: %.cc
    $(CXX) $(CXXFLAGS) -c $< -o $@ $(INCLUDES)

# The following is to test your system and ensure a smoother experience.
# They are by no means necessary to actually compile a grpc-enabled software.

PROTOC = protoc
GRPC_CPP_PLUGIN = grpc_cpp_plugin
GRPC_CPP_PLUGIN_PATH ?= `which $(GRPC_CPP_PLUGIN)`

PROTOC_CMD = which $(PROTOC)
PROTOC_CHECK_CMD = $(PROTOC) --version | grep -q libprotoc.3
PLUGIN_CHECK_CMD = which $(GRPC_CPP_PLUGIN)
HAS_PROTOC = $(shell $(PROTOC_CMD) > /dev/null && echo true || echo false)
ifeq ($(HAS_PROTOC),true)
HAS_VALID_PROTOC = $(shell $(PROTOC_CHECK_CMD) 2> /dev/null && echo true || echo false)
endif
HAS_PLUGIN = $(shell $(PLUGIN_CHECK_CMD) > /dev/null && echo true || echo false)

SYSTEM_OK = false
ifeq ($(HAS_VALID_PROTOC),true)
ifeq ($(HAS_PLUGIN),true)
SYSTEM_OK = true
endif
endif

system-check:
ifneq ($(HAS_VALID_PROTOC),true)
    @echo " DEPENDENCY ERROR"
    @echo
    @echo "You don't have protoc 3.0.0 installed in your path."
    @echo "Please install Google protocol buffers 3.0.0 and its compiler."
    @echo "You can find it here:"
    @echo
    @echo "   https://github.com/google/protobuf/releases/tag/v3.0.0"
    @echo
    @echo "Here is what I get when trying to evaluate your version of protoc:"
    @echo
    -$(PROTOC) --version
    @echo
    @echo
endif
ifneq ($(HAS_PLUGIN),true)
    @echo " DEPENDENCY ERROR"
    @echo
    @echo "You don't have the grpc c++ protobuf plugin installed in your path."
    @echo "Please install grpc. You can find it here:"
    @echo
    @echo "   https://github.com/grpc/grpc"
    @echo
    @echo "Here is what I get when trying to detect if you have the plugin:"
    @echo
    -which $(GRPC_CPP_PLUGIN)
    @echo
    @echo
endif
ifneq ($(SYSTEM_OK),true)
    @false
endif
```

`Makefile`中与`test_grpc`项目相关部分：

```makefile
test_grpc: $(GRPC_OBJS) test_grpc.o
    $(CXX) $^ $(LDFLAGS) -o $@
```

先执行`make grpc`编译proto，再执行`make test_grpc`编译测试客户端，再执行`make release`编译服务端。

### 2.6 启动测试

1. 启动服务端
    ````bash
    ./wechat_server --log_level=2
    ```
2. 启动测试客户端
    ```bash
    ./test_grpc
    ```
3. `test_grpc`部分输出：
    ```txt
    jack@ubuntu-hyperv:~/wechat_server/tools$ ./test_grpc
    Testing sync with account...
    sync 70801150121796
    #1 70801150121796 online wire disarm ****** no alarm message

    sync 70801151915015
    #1 70801151915015 online wire disarm ****** no alarm message

    sync 70801163150828
    #1 70801163150828 online wire disarm ****** no alarm message

    Testing sync without account...
    #1 863977037135212 offline unknown arm ****** alarm message: 3 1130 2018-12-26 15:04:47

    #1 863977037135212 online unknown arm ****** no alarm message

    ^C
    jack@ubuntu-hyperv:~/wechat_server/tools$ 
    ```
测试成功。

## 3. 编写PHP客户端

### 3.1 编译proto生成所需PHP文件

`gen_grpc.sh`

```bash
#!/bin/bash
protoc --proto_path=/home/jack/wechat_server/wechat_server/protos --php_out=./ --grpc_out=./ --plugin=protoc-gen-grpc=`which grpc_php_plugin` /home/jack/wechat_server/wechat_server/protos/alarm_server.proto
```

生成了`Alarm_server`与`GPBMetadata`两个文件夹，在项目根目录浏览：

```bash
jack@ubuntu-hyperv:/var/www/wx-server$ ll
total 624
drwxr-xr-x  17 www-data www-data   4096 Dec 25 17:18 ./
drwxr-xr-x  11 www-data www-data   4096 Dec 24 21:34 ../
drwxr-xr-x   2 www-data www-data   4096 Dec 24 21:41 Alarm_server/
-rwxr-xr-x   1 www-data www-data     36 Dec 24 21:31 amp.sh*
drwxr-xr-x   9 www-data www-data   4096 Dec 12 15:30 app/
-rw-r--r--   1 www-data www-data   1686 Dec 12 15:30 artisan
drwxr-xr-x   3 www-data www-data   4096 Dec 12 15:30 bootstrap/
-rw-r--r--   1 www-data www-data   1774 Dec 25 17:18 composer.json
-rw-r--r--   1 www-data www-data 212478 Dec 25 16:47 composer.lock
drwxr-xr-x   2 www-data www-data   4096 Dec 17 16:03 config/
-rwxr-xr-x   1 www-data www-data    192 Dec 25 17:11 cron_job.sh*
drwxr-xr-x   5 www-data www-data   4096 Dec 12 15:30 database/
-rw-r--r--   1 www-data www-data    656 Dec 12 15:37 .env
-rw-r--r--   1 www-data www-data    712 Dec 17 16:03 .env.example
-rwxr-xr-x   1 www-data www-data    220 Dec 24 21:41 gen_grpc.sh*
drwxr-xr-x   8 www-data www-data   4096 Dec 25 17:25 .git/
-rw-r--r--   1 www-data www-data    111 Dec 12 15:30 .gitattributes
-rw-r--r--   1 www-data www-data    173 Dec 24 21:44 .gitignore
drwxr-xr-x   2 www-data www-data   4096 Dec 24 21:41 GPBMetadata/
drwxr-xr-x 981 www-data www-data  36864 Dec 12 15:35 node_modules/
-rw-r--r--   1 www-data www-data   1085 Dec 12 15:30 package.json
-rw-r--r--   1 www-data www-data   1040 Dec 12 15:30 phpunit.xml
drwxr-xr-x   6 www-data www-data   4096 Dec 13 10:30 public/
-rwxr-xr-x   1 www-data www-data     23 Dec 24 21:31 pull.sh*
-rw-r--r--   1 www-data www-data   3550 Dec 12 15:30 readme.md
drwxr-xr-x   5 www-data www-data   4096 Dec 12 15:30 resources/
drwxr-xr-x   2 www-data www-data   4096 Dec 13 10:30 routes/
-rw-r--r--   1 www-data www-data    563 Dec 12 15:30 server.php
drwxr-xr-x   6 www-data www-data   4096 Dec 12 15:30 storage/
drwxr-xr-x   4 www-data www-data   4096 Dec 12 15:30 tests/
drwxr-xr-x  48 www-data www-data   4096 Dec 25 17:22 vendor/
drwxr-xr-x   2 www-data www-data   4096 Dec 24 21:44 .vscode/
-rw-r--r--   1 www-data www-data    549 Dec 12 15:30 webpack.mix.js
-rw-r--r--   1 www-data www-data 255508 Dec 12 15:30 yarn.lock
```

查看`AlarmServer`文件夹：

```bash
jack@ubuntu-hyperv:/var/www/wx-server$ ll Alarm_server/
total 36
drwxr-xr-x  2 www-data www-data 4096 Dec 24 21:41 ./
drwxr-xr-x 17 www-data www-data 4096 Dec 25 17:18 ../
-rw-r--r--  1 www-data www-data 3595 Dec 25 16:43 AlarmMessage.php
-rw-r--r--  1 www-data www-data 1352 Dec 25 16:43 AlarmServerClient.php
-rw-r--r--  1 www-data www-data 1574 Dec 25 16:43 GetMachineRequest.php
-rw-r--r--  1 www-data www-data 1379 Dec 25 16:43 IsNeedSyncReply.php
-rw-r--r--  1 www-data www-data 1277 Dec 25 16:43 IsNeedSyncRequest.php
-rw-r--r--  1 www-data www-data 4439 Dec 25 16:43 MachineReply.php
```

查看`GPBMetadata`文件夹：

```bash
jack@ubuntu-hyperv:/var/www/wx-server$ ll GPBMetadata/
total 12
drwxr-xr-x  2 www-data www-data 4096 Dec 24 21:41 ./
drwxr-xr-x 17 www-data www-data 4096 Dec 25 17:18 ../
-rw-r--r--  1 www-data www-data 1984 Dec 25 16:43 AlarmServer.php
```

### 3.2 编写客户端代码

在`app/Handlers`文件夹内新增`GrpcClient.php`

```php
<?php

namespace App\Handlers;

use Alarm_server\AlarmServerClient;
use Alarm_server\AlarmMessage;
use Alarm_server\GetMachineRequest;
use Alarm_server\MachineReply;
use Alarm_server\IsNeedSyncRequest;
use Alarm_server\IsNeedSyncReply;
use App\Models\Machine;
use Illuminate\Support\Facades\Log;

class GrpcClient
{
    public function sync(){
        $client = new AlarmServerClient('localhost:50051', [
            'credentials' => \Grpc\ChannelCredentials::createInsecure()
        ]);

        $request = new IsNeedSyncRequest();
        list($reply, $status) = $client->IsNeedSync($request)->wait();
        if($reply->getChangedMachineCount() > 0){
            $getMachineRequest = new GetMachineRequest();
            $call = $client->Sync($getMachineRequest);
            $machineReplies = $call->responses();
            foreach($machineReplies as $machineReply){
                $this->updateMachine($machineReply);
            }
        }else{
            Log::info('GrpcClient::sync no need to update');
        }
    }

    private function updateMachine(MachineReply $machineReply){
        Log::info('GrpcClient::updateMachine: ' . $machineReply->getAccount());
        // 执行更新数据库、推送报警信息给微信用户等操作
    }
}
```

### 3.3 新增一个命令

执行`php artisan make:command SyncWithGrpc --command=wx-server:sync-grpc`后编辑`SyncWithGrpc.php`文件：

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Support\Facades\Log;

use App\Handlers\GrpcClient;

class SyncWithGrpc extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'wx-server:sync-grpc';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = '从grpc服务端同步数据';

    /**
     * Create a new command instance.
     *
     * @return void
     */
    public function __construct()
    {
        parent::__construct();
    }

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->info('SyncWithGrpc running...');
        Log::info("SyncWithGrpc running...");
        $client = new GrpcClient();
        $client->sync();
    }
}
```

### 3.4 添加计划任务

```bash
export EDITOR=vi && crontab -e
```

添加如下一行：

```cron
* * * * * /bin/bash /var/www/wx-server/cron_job.sh >> /dev/null 2>&1
```

`cron_job.sh`内容：

```bash
#!/bin/bash

step=1 #间隔的秒数

for (( i = 0; i < 60; i=(i+step) )); do
    /usr/bin/php -d extension=grpc.so /var/www/wx-server/artisan wx-server:sync-grpc
    sleep $step
done

exit 0
```

实现了每秒执行一次与grpc服务端同步的功能。

### 3.5 问题解决

* 解决`Alarm_server`找不到问题

    问题描述：

    `Class 'Alarm_server\AlarmServerClient' not found {"exception":"[object] (Symfony\\Component\\Debug\\Exception\\FatalThrowableError(code: 0): Class 'Alarm_server\\AlarmServerClient' not found at /var/www/wx-server/app/Handlers/GrpcClient.php:17)`

    在`composer.json`中修改：

    ```json
    "autoload": {
        .
        .
        .
        "psr-4": {
            "App\\": "app/",
            "Alarm_server\\": "Alarm_server/",
            "GPBMetadata\\": "GPBMetadata/"
        }
    },
    ```

    修改后执行`composer update`即可。

* 解决`Grpc\ChannelCredentials`找不到问题

    问题描述：

    `Class 'Grpc\ChannelCredentials' not found {"exception":"[object] (Symfony\\Component\\Debug\\Exception\\FatalThrowableError(code: 0): Class 'Grpc\\ChannelCredentials' not found at /var/www/wx-server/app/Handlers/GrpcClient.php:18)`

    原因是虽然在`/etc/php/7.2/fpm/php.ini`中追加了`extensions=grpc.so`，但这只能保证`php-fpm`可以加载`grpc.so`，而`cron_job`内执行的是`/usr/bin/php /var/www/wx-server/artisan wx-server:sync-grpc`，需要补上参数`-d extension=grpc.so`才能解决。
