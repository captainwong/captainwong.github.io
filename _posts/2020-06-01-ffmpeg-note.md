---
layout: post
title:  "FFmpeg 学习笔记"
subtitle: "杂记"
date:   "2020-06-01"
author: "cj"
tags:
    ffmpeg
---

# FFmpeg 学习笔记

## 编译

### 下载源码

* [官网](http://ffmpeg.org/download.html) 比较慢，没梯子是万万不行的
* [FFmpeg.club](FFmpeg.club) 夏曹俊维护的网站，缺点是更新不大及时
* [码云](gitee.com) 码云可以从 `github` 导入仓库，然后随便下载。我经常用他导入比较大的库比如 `protobuf`, `grpc`, `breakpad` 等等，对不住了，不过真香。。。

### Ubuntu 16.04 编译

#### 编译

```bash
tar jxvf ffmpeg-4.2.1-source.tar.bz2
cd ffmpeg-4.2.1-source
apt install pkg-config nasm -y
# Ubuntu Server 就不安装 SDL2 了，没有图形库。因此不会编译 `ffplay`。
# apt install sdl2-dev

# --prefix 安装路径
# --disable-static 不编译静态库
# --enable-shared 编译动态库# 如果 make 的时候报 `/usr/bin/ld: libavcodec/mqc.o: relocation R_X86_64_32 against `.rodata' can not be used when making a shared object; recompile with -fPIC` 错误
# ./configure 尾巴加上 --extra-cflags="-fPIC"
./configure --prefix=/usr/local/ffmpeg --disable-static --enable-shared
make -j 16
sudo make install
```

#### 设置环境变量

`vi ~/.bashrc`

添加 `export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/usr/local/ffmpeg/lib/pkgconfig`，让 `pkg-config` 可以找到 `ffmpeg` 的库。

添加 `export PATH=$PATH:/usr/local/ffmpeg/bin` 让 `ffmpeg` 可以在任意路径执行。
`source ~/.bashrc` 

测试：

`pkg-config --libs --cflags libavutil`

`ffmpeg`

如果输出正常则配置无误。

### macOS Catalina 编译

```bash
tar jxvf ffmpeg-4.2.1-source.tar.bz2
cd ffmpeg-4.2.1-source
brew install pkg-config nasm
./configure --prefix=/usr/local/ffmpeg --disable-static --enable-shared
make -j 16
sudo make install
```

除了 `apt` 换成了 `brew`，其他完全一样。配置环境变量也一样，只是配置文件是 `~/.bash_profile`，不再赘述。

### Windows 编译

#### cygwin 编译

`cygwin` 是个在 `windows` 下模拟 `linux` 的东东，所以步骤参照 `Ubuntu16.04`，只是它的包管理工具不是 `apt`，而是 `cyg-apt`。

1. 去 [官网](http://cygwin.org/) 下载安装包安装，除了获取源列表有点慢可能需要搭梯子，获取完源列表后，选择网易的源，下载速度就飞快了。安装时为了省事，全选 `Debug, Devel, Net`，还有 `Web/wget`。

2. 打开 `cygwin terminal`，安装 `cyg-apt`: 

    ```bash
    # 不用代理没法下载啊摔
    wget -e http_proxy=http://127.0.0.1:10809 https://raw.githubusercontent.com/transcode-open/apt-cyg/master/apt-cyg
    install apt-cyg /bin
    ```
3. 解压、编译，参考 `Ubuntu16.04`

#### msys2 编译

1. 去[官网](https://www.msys2.org/) 下载安装包安装
2. 第一件事：换源！参考 [清华大学MSYS2 镜像使用帮助](https://mirror.tuna.tsinghua.edu.cn/help/msys2/)
3. 然鹅没有 `vim` 没法换源，先龟速安装吧。。。 `pacman -S vim`
4. 换源~~
5. 不使用 `msys2.exe` 编译，而是打开 `mingw64.exe` 
6. 安装工具链 `pacman -S make diffutils pkg-config nasm mingw64/mingw-w64-x86_64-gcc`。之所以 `gcc` 要写那么长，是因为直接写 `gcc` 的话安装的是 `msys gcc`，也能成功编译，但执行 `ffmpeg.exe` 时会提示 `msys-2.0.dll` 找不到。
7. `pacman -S mingw64/mingw-w64-x86_64-fdk-aac`
8. `pacman -S mingw64/mingw-w64-x86_64-opus`
9. 安装 `libx264`

    ```bash
    wget https://code.videolan.org/videolan/x264/-/archive/master/x264-master.tar.bz2
    tar jxvf x264-master.tar.bz2
    cd x264-master.tar.bz2
    make && make install
    echo "export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/usr/local/lib/pkgconfig" >> ~/.bashrc
    source ~/.bashrc
    ```
10. 安装 `libx265`: `pacman -S mingw64/mingw-w64-x86_64-x265`
11. `./configure --prefix=/usr/local/ffmpeg --enable-shared --disable-static --enable-debug=3 --enable-libx264 --enable-libx265 --enable-filter=delogo --enable-gpl --enable-nonfree --enable-libfdk-aac --enable-libopus`
12. `make -j 12 && make install`


#### msys2 + vs2019 编译 (x86_64)

1. 安装 `msys2` 同上不提
2. 打开 `x64 Native Tools Command Prompt for VS 2019`
3. 切换路径到 `msys2` 安装路径如 `G:\msys64`
4. `msys2_shell.cmd -use-full-path -mingw64`
5. 切换到 `ffmpeg` 源码路径，执行 `./configure --prefix=../win64 --toolchain=msvc --arch=x86_64 --enable-cross-compile`
6. 执行 `make` 会报错

    `fftools/cmdutils.c(1149): error C2065: “slib”: 未声明的标识符`

    用 `vscode` 打开 `config.h`，`reopen with encoding` 选 `gb2312`，再 `save with encoding` 选 `utf8` 后再次执行 `make` 即可
7. `make install`

