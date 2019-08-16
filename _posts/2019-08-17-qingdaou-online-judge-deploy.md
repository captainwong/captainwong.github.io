---
layout: post
title:  "青岛大学 Online Judge 系统部署小记"
subtitle: "不是我想要的"
date:   "2019-08-17"
author: "cj"
tags:
    qingdaou
    oj
    ubuntu
    docker
---

# 青岛大学 Online Judge 系统部署小记

本想找一个 `leetcode` 那样的开源项目，但是提供多种语言的在线编辑、编译、执行，尝试了下 `github` 上 2300 多个星的项目 [QingdaoU/OnlineJudge](https://github.com/QingdaoU/OnlineJudge)，没有题库，没法随意写代码而是只能先在后台添加题目，前台答题时也只能遵循固定的输入输出，限制太大了。还是记下来，毕竟第一次使用 `docker` 。。。

按照官方部署文档 [QingdaoU/OnlineJudgeDeploy](https://github.com/QingdaoU/OnlineJudgeDeploy/tree/2.0)执行，有错误，`python` 需要 3.X，且安装 `docker` 的脚本并不快，总结一下：

1. 修改全局 `python` 版本

    `pyenv global 3.5.3`

2. 安装 `docker-compose`

    `pip install docker-compose`

3. 安装 `docker`

    [docker.sh](https://github.com/captainwong/sh/blob/master/ubuntu16.04/docker.sh)

    ```bash
    apt update -y
    apt -y install apt-transport-https ca-certificates curl software-properties-common
    curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
    add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
    apt update -y
    apt install docker-ce -y
    ```

4. 部署 `oj`

    ```bash
    git clone -b 2.0 https://github.com/QingdaoU/OnlineJudgeDeploy.git && cd OnlineJudgeDeploy
    docker-compose up -d
    ```

    报错，因为80端口被占用了。根据 [官方文档](https://docs.onlinejudge.me/#/onlinejudge/faq)，修改 `docker-compose.yml`：

    >修改 docker-compose 中 ports 相关的配置，比如 0.0.0.0:80:8080 可以修改为 0.0.0.0:8020:8080，冒号后面的端口号不会冲突请勿改动。

    再次执行 `docker-compose up -d` 成功。
