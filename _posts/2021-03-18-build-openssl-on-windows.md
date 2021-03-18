---
layout: post
title:  "Windows 环境编译 OpenSSL 小记"
subtitle: "到底没明白Perl有啥好的"
date:   "2021-03-18"
author: "cj"
tags:
    openssl
    perl
    windows
---

# Windows 环境编译 OpenSSL 小记

## 0. 环境

* Win10 20H2 19042.867
* Visual Studio 2019 Community 16.9.1

## 1. 下载源码

到 [openssl.org/source](https://www.openssl.org/source/) 下载源码，版本 [1.1.1j](https://www.openssl.org/source/openssl-1.1.1j.tar.gz)。
解压后阅读 README, INSTALL, NOTES.WIN, NOTES.PERL，得先安装 ActivePerl并安装 Text-Template 模块。

## 2. 准备 ActivePerl

烦的一批，直接到 [官网](https://www.activestate.com/products/perl/downloads/) 下载来是个半成品，还得联网下载莫名其妙的东西，开了代理费半天劲下载下来了，还总提示 
```
Something Went Wrong
────────────────────
 x Could not install dependencies
 x The project is currently building. Please visit
https://platform.activestate.com to see the progress.
```

到上面这个网址看看，用 github 账号授权登录后，发现居然像 github 仓库一样，还有 master 分支，还可以在线编译、增删模块，可以在线生成一个离线版的 Perl 安装包，我勒个去，城会玩。。。

OpenSSL 已经说了需要 Perl 的 Text-Template 模块，在 platform.activestate.com 里 Configuration 一下，添加这个模块，会在线编译，耗时30多分钟后生成了 ActivePerl-5.28.1.0000-MSWin32-x64-5a7917ca.msi，就问你高级不高级！吓得我直呼666。。。

下载安装不提。

## 3. 准备 nasm

到 [nasm.us](nasm.us) 官网下载 [nasm-2.15.05-win64.zip](https://www.nasm.us/pub/nasm/releasebuilds/2.15.05/win64/nasm-2.15.05-win64.zip)，解压并添加目录到 PATH 不提。

## 编译 OpenSSL

已经解压 OpenSSL 源码到 `C:\Users\Jack\Downloads\openssl-1.1.1j`，新建文件夹 `C:\Users\Jack\Downloads\openssl-1.1.1j-build`。
打开 `x64 Native Tools Command Prompt for VS 2019`，`cd` 到 `build`，执行 `perl ..\openssl-1.1.1j-build Configure VC-WIN64A`。

这里有几个选项可以用：

* VC-WIN32
* VC-WIN64A AMD64
* VC-WIN64I IA64
* VC-CE 不了解，应该是 Windows CE 的

我要的是 x64 的，所以用 VC-WIN64A。

完了执行 `nmake && name install`。`install` 需要管理员权限，默认安装路径`C:\Program Files\OpenSSL`，忘记开管理员权限就再打开一个命令行。

完
