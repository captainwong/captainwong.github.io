---
layout: post
title:  "Ubuntu16.04.5环境编译grpc小记"
subtitle: "都是坑"
date:   "2018-12-19" 
author: "cj"
tags:
    ubuntu
    gRPC
    protobuf
    php
---

# Ubuntu16.04.5环境编译grpc小记

## 1. 初始化环境

执行[脚本](https://github.com/captainwong/sh/blob/master/ubuntu16.04/1.init.sh)

## 2. 编译

### 2.1 按照[官方教程](https://grpc.io/docs/quickstart/php.html)编译

```bash
git clone -b $(curl -L https://grpc.io/release) https://github.com/grpc/grpc
cd grpc
git submodule update --init
make
sudo make install
```

其中在执行`make`时报错`protobuf configure.ac:104: error: possibly undefined macro: AC_PROG_LIBTOOL`，需要安装`libtool`：

```bash
sudo apt-get install libtool-bin -y
```

我记得中间还有一次报错，找不到`autoreconf`，安装`autoconf`解决：

```bash
sudo apt-get install autoconf -y
```

好像之后再`make`也会出错，`cd`到`src/thirdparty/protobuf`目录后执行`./autogen.sh && ./configure && make && sudo make install`后再返回`grpc`根目录再次`make`成功，然后执行`sudo make install`成功。

### 2.2 Build and install gRPC PHP extension

会提示`phpize`未安装，先安装：

```bash
sudo apt-get install php7.2-dev -y
```

再编译

```bash
cd grpc/src/php/ext/grpc
phpize
./configure
make
sudo make install
```

## Reference

* [error in autogen.sh? #2604](https://github.com/protocolbuffers/protobuf/issues/2604)
* [Cannot build and install: Missing configure #945](https://github.com/protocolbuffers/protobuf/issues/945)