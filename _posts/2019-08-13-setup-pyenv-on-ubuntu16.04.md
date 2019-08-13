---
layout: post
title:  "Ubuntu16.04安装pyenv小记"
subtitle: "。。。"
date:   "2019-08-13"
author: "cj"
tags:
    python
    pyenv
    ubuntu
---

# `Ubuntu16.04` 安装 `pyenv` 小记

## 安装 `pyenv`

1. 安装

    ```bash
    sudo apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev xz-utils tk-dev libffi-dev liblzma-dev python-openssl git

    git clone https://github.com/yyuu/pyenv.git ~/.pyenv
    echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
    echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
    echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.bashrc

    echo "Dont't forget to RESTART SHELL"
    ```

2. OpenSSL问题

    执行 [dl_from_sohu_to_pyenv_cache_install.sh](https://github.com/captainwong/sh/blob/master/python/dl_from_sohu_to_pyenv_cache_install.sh)

    ```txt
    root@ubuntuAmd64:~# /home/jack/sh/python/dl_from_sohu_to_pyenv_cache_install.sh 3.5.2
    --2019-08-13 00:58:11--  http://mirrors.sohu.com/python/3.5.2/Python-3.5.2.tar.xz
    Resolving mirrors.sohu.com (mirrors.sohu.com)... 123.125.123.141
    Connecting to mirrors.sohu.com (mirrors.sohu.com)|123.125.123.141|:80... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 15222676 (15M) [application/octet-stream]
    Saving to: ‘/root/.pyenv/cache/Python-3.5.2.tar.xz’

    Python-3.5.2.tar.xz 100%[===================>]  14.52M   516KB/s    in 29s

    2019-08-13 00:58:41 (513 KB/s) - ‘/root/.pyenv/cache/Python-3.5.2.tar.xz’ saved [15222676/15222676]

    Installing Python-3.5.2...
    patching file Lib/venv/scripts/posix/activate.fish
    ERROR: The Python ssl extension was not compiled. Missing the OpenSSL lib?

    Please consult to the Wiki page to fix the problem.
    https://github.com/pyenv/pyenv/wiki/Common-build-problems


    BUILD FAILED (Ubuntu 16.04 using python-build 1.2.13)

    Inspect or clean up the working tree at /tmp/python-build.20190813005841.27201
    Results logged to /tmp/python-build.20190813005841.27201.log

    Last 10 log lines:
    (cd /root/.pyenv/versions/3.5.2/share/man/man1; ln -s python3.5.1 python3.1)
    if test "xupgrade" != "xno"  ; then \
            case upgrade in \
                    upgrade) ensurepip="--upgrade" ;; \
                    install|*) ensurepip="" ;; \
            esac; \
            ./python -E -m ensurepip \
                    $ensurepip --root=/ ; \
    fi
    Ignoring ensurepip failure: pip 8.1.1 requires SSL/TLS
    ```

    执行 `openssl version` 结果：
    `OpenSSL 1.1.0h  27 Mar 2018 (Library: OpenSSL 1.1.1c  28 May 2019)`

    算了，`python` 对我来说不是重度需求，捡个可以安装的版本用就好了。`2.7.13, 3.5.3 and 3.6.0` 都可以安装成功。

    ```
    root@ubuntuAmd64:~# pyenv versions
    * system (set by /root/.pyenv/version)
    2.7.13
    3.5.3
    root@ubuntuAmd64:~# pyenv global 3.5.3
    root@ubuntuAmd64:~# python --version
    Python 3.5.3
    root@ubuntuAmd64:~# pyenv global 2.7.13
    root@ubuntuAmd64:~# python --version
    Python 2.7.13
    ```

3. 总结脚本

    可以在我的 [github](https://github.com/captainwong/sh/tree/master/python) 里找到。

## Reference
* [pyenv](https://github.com/pyenv/pyenv)
* [Common build problems](https://github.com/pyenv/pyenv/wiki/Common-build-problems)
* [使用pyenv管理工作环境](https://zhuanlan.zhihu.com/p/27294128)
* [pyenv使用国内镜像安装指定的Python版本](https://www.jianshu.com/p/cb7a128b284b)
* [pip/pip3更换国内镜像源](https://www.jianshu.com/p/8192df46f4e3)
* [ERROR: The Python ssl extension was not compiled. Missing the OpenSSL lib? (installing python 2.7 on ubuntu 18.04)](https://stackoverflow.com/questions/52873193/error-the-python-ssl-extension-was-not-compiled-missing-the-openssl-lib-inst)