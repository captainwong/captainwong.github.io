---
layout: post
title:  "使用Google开源库breadpad实现错误报告功能"
subtitle: "Google大法好"
date:   "2017-08-10" 
author: "cj"
tags:
    breadpad
    c++
    linux
    google
    bugreport
---



前阵子写的微信公众号后台服务器自动崩溃重启了一次，看日志没任何头绪，看来需要core dump。
但是搜索一阵子发现，这玩意真难用，要`ulimit -c unlimited`后才会生成dump。又搜索一番，发现Google出品的breadpad，谷歌出品，必属精品，就它了！

1. 按照[官方教程](https://chromium.googlesource.com/breakpad/breakpad)，下载源码

```
git clone https://chromium.googlesource.com/breakpad/breakpad
```

2. 有几个第三方库默认情况没有下载，按需手动下载到breakpad/src/thirdparty中

```
cd breakpad
git clone https://chromium.googlesource.com/linux-syscall-support src/third_party/lss
```

3. 编译安装

```
./configure
make
make check
sudo make install
```

安装目录默认为/usr/local/include/breakpad，库目录/usr/local/lib/libbreakpad.a, libbreakpad_client.a

4. 应用

方便起见，写了一个自动生成symbol调试信息的脚本build_symbols.sh:


