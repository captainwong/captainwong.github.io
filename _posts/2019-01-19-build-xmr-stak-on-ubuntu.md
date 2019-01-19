---
layout: post
title:  "Ubuntu16.04编译xmr-stak小记"
subtitle: "不记下来下次还得查资料。。。"
date:   "2019-01-19"
author: "cj"
tags:
    ubuntu
    xmr-stak
    nvidia
---

# Ubuntu16.04编译xmr-stak小记

CUDA版

1. 禁用 `nouveau kernel driver`

    参考 [How to disable Nouveau kernel driver](https://askubuntu.com/questions/841876/how-to-disable-nouveau-kernel-driver)

    编辑：
    `sudo vi /etc/modprobe.d/blacklist-nouveau.conf`
    
    写入：
    ```conf
    blacklist nouveau
    options nouveau modeset=0
    ```

    Regenerate the kernel initramfs:

    `sudo update-initramfs -u`

    重启：

    `sudo reboot`


2. 下载 [CUDA官方驱动](https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&target_distro=Ubuntu&target_version=1604&target_type=runfilelocal)

    执行：

    `sudo sh cuda_10.0.130_410.48_linux.run`

    根据提示一路进行，可以不选择 `samples`。

    导出路径：

    ```bash
    export PATH=$PATH:/usr/loca/cuda-10.0/bin
    source ~/.profile
    export LC_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/loca/cuda-10.0/lib64
    sudo ldconfig
    ```

3. 根据 [`xmr-stak` 文档](https://github.com/fireice-uk/xmr-stak/blob/master/doc/compile_Linux.md) 执行即可

    ```bash
    sudo apt install libmicrohttpd-dev libssl-dev cmake build-essential libhwloc-dev
    git clone https://github.com/fireice-uk/xmr-stak.git
    mkdir xmr-stak/build
    cd xmr-stak/build
    cmake ..
    make install
    ```