---
layout: post
title:  "Ubuntu16.04部署Gitlab小记"
subtitle: "为了加速墙外repo我是煞费苦心呐！"
date:   "2019-01-20"
author: "cj"
tags:
    ubuntu
    git
    gitlab
    aliyun
    google
    gfw
---

# Ubuntu16.04 部署 Gitlab 小记

执行 `git clone -b $(curl -L https://grpc.io/release) https://github.com/grpc/grpc` 时看着那10几K/s 的破网速，忍不住哭出声。。。

有阿里云主机一台，且域名已备案，搭个 Gitlab 吧，以后再下载墙外 repo 也不用那么费劲了！

## 1. 安装
```bash
# 依赖包
sudo apt-get install curl openssh-server ca-certificates postfix
# 信任 GitLab 的 GPG 公钥
curl https://packages.gitlab.com/gpg.key 2> /dev/null | sudo apt-key add - &>/dev/null

# 使用清华镜像加速
# run as root: su root, then:
echo "deb https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/ubuntu xenial main" >> /etc/apt/sources.list.d/gitlab-ce.list

# run as normal user
sudo apt-get update
sudo apt-get install gitlab-ce

```

## 2. 配置

编辑 `/etc/gitlab/gitlab.rb`:

1. 外部链接

    `external_url 'http://git.aliyun-host.com'`
2. 修改时区

    `gitlab_rails['time_zone'] = 'Asia/Shanghai'`
3. 禁用 `bundled-web-server`

    - `nginx['enable'] = false`
    - `unicorn['enable'] = false`
    - `web_server['external_users'] = ['www-data']`

4. 使修改生效

    `sudo gitlab-ctl reconfigure`

## 3. 配置子域名

阿里云已备案域名一枚如 `aliyun-host.com`，添加A记录 `git.aliyun-host.com`。

新建 nginx 站点：

`sudo vi /etc/nginx/sites-available/gitlab-omnibus-nginx.conf`

在官网拷贝[推荐配置](https://gitlab.com/gitlab-org/gitlab-recipes/blob/master/web-server/nginx/gitlab-omnibus-nginx.conf)粘贴进去，下面列出修改的部分：

```diff
- listen 0.0.0.0:80 default_server;
- listen [::]:80 default_server;
- server_name YOUR_SERVER_FQDN; ## Replace this with something like gitlab.example.com

+ listen 0.0.0.0:80;
+ listen [::]:80;
+ server_name git.aliyun-host.com; ## Replace this with something like gitlab.example.com
```

创建软链接

```bash
cd ../sites-enabled
sudo ln -s ../sites-available/gitlab-omnibus-nginx.conf .
```

重启 nginx: `sudo systemctl restart nginx.service`

## 4. 配置 Gitlab

访问 `http://git.aliyun-host.com`，输入密码、注册用户、添加 SSH 秘钥、创建 google 组、创建 grpc 项目。。。网上教程不要太多，不提。

## 5. 浪吧

本地有费尽千辛万苦同步好的 gprc 源码，添加阿里云仓库并上传之：

```bash
git remote add ali git@git.aliyun-host.com:google/breakpad.git
git push -u ali --all
```

浪吧！

## 6. Reference

- [Ubuntu 16.x 安装Gitlab](http://galudisu.info/2017/05/01/gitlab/ubuntu-gitlab-install/)
- [Ubuntu 16.04搭建GitLab服务器](https://www.linuxidc.com/Linux/2018-01/150319.htm)
- [Ubuntu 16.04 x64搭建GitLab服务器操作笔记](https://www.zybuluo.com/lovemiffy/note/418758)
- [Forwarding to GitLab Subdomain with Existing Nginx Installation](https://stackoverflow.com/questions/29403212/forwarding-to-gitlab-subdomain-with-existing-nginx-installation)
- [Using a non-bundled web-server](https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/doc/settings/nginx.md#using-a-non-bundled-web-server)
- [gitlab-omnibus-nginx.conf](https://gitlab.com/gitlab-org/gitlab-recipes/blob/master/web-server/nginx/gitlab-omnibus-nginx.conf)
- []