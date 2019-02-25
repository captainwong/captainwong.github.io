---
layout: post
title:  "使用frp搭建微信开发环境"
subtitle: "学习一个"
date:   "2019-02-25"
author: "cj"
tags:
    wechat
    nginx
    reverse-proxy
    aliyun
    laravel
    php
    easywechat
---

# 使用`frp`搭建微信开发环境

这两天准备换 `PHP` 搞微信开发，用 `ngrok` 转发时设置微信接口永远失败。。。搞不懂什么原因。

刚好看了篇大神的文章[内网穿透：在公网访问你家的 NAS](https://zhuanlan.zhihu.com/p/57477087)，介绍了 `frp` 这个神器，搜索了下 [微信+frp](https://www.google.com/search?newwindow=1&biw=1918&bih=899&ei=wVVzXMj-M5mBk-4Pz8yX8As&q=frp+%E5%BE%AE%E4%BF%A1&oq=frp+%E5%BE%AE%E4%BF%A1&gs_l=psy-ab.3..33i160l2.13296496.13298428..13298904...0.0..0.275.2373.2-9....2..0....1..gws-wiz.......0i71j35i39j0i131j0i67j0j0i203j0i22i30.2pEwzUk5Oi8)，已经有不少人在用这种技术了，试了下感觉非常好用，记录下来备忘。

## 1. 准备

* 阿里云轻量级服务器一台，系统 `Ubuntu 16.04`，IP 假设为 `1.2.3.4`
* 已备案域名一个并指向 `1.2.3.4`，文章使用 `your-domain.com` 进行说明
* 域名新增一个 A记录：`wx.your-domain.com` 仍然指向阿里云服务器 IP `1.2.3.4`
* 本地 `Win10` 系统，使用 `Homestead` 虚拟机, IP 为 `192.168.10.10`，已安装 `Ubuntu 16.04`

开始吧！

## 2. 服务端

### 2.1 配置 `fprs` 

下载 [frp_0.24.1_linux_amd64.tar.gz](https://github.com/fatedier/frp/releases/download/v0.24.1/frp_0.24.1_linux_amd64.tar.gz) 并解压

```bash 
wget https://github.com/fatedier/frp/releases/download/v0.24.1/frp_0.24.1_linux_amd64.tar.gz
tar zxvf frp_0.24.1_linux_amd64.tar.gz
cd frp_0.24.1_linux_amd64

# 看下都有啥
root@aliyun:~/frp_0.24.1_linux_amd64$ ll
total 19431
drwxrwxrwx 1 root root     4096 Feb 25 15:21 ./
drwxrwxrwx 1 root root     4096 Feb 25 14:15 ../
-rwxrwxrwx 1 root root  9589632 Feb 25 14:15 frpc*
-rwxrwxrwx 1 root root     6376 Feb 25 14:15 frpc_full.ini*
-rwxrwxrwx 1 root root      168 Feb 25 14:54 frpc.ini*
-rwxrwxrwx 1 root root 10276544 Feb 25 14:15 frps*
-rwxrwxrwx 1 root root     2199 Feb 25 14:15 frps_full.ini*
-rwxrwxrwx 1 root root       26 Feb 25 14:15 frps.ini*
-rwxrwxrwx 1 root root    11358 Feb 25 14:15 LICENSE*
```

编辑 `frps.ini`

```ini
[common]
bind_port = 7000
auth_token = your-token
vhost_http_port = 8080
vhost_https_port = 8443

[web]
type = http
custom_domains = wx.your-domain.com
```

启动

```bash
root@aliyun:~/frp_0.24.1_linux_amd64$ ./frps -c ./frps.ini
2019/02/25 15:39:16 [I] [service.go:124] frps tcp listen on 0.0.0.0:7000
2019/02/25 15:39:16 [I] [service.go:166] http service listen on 0.0.0.0:8080
2019/02/25 15:39:16 [I] [service.go:187] https service listen on 0.0.0.0:8443
2019/02/25 15:39:16 [I] [root.go:204] Start frps success
```

### 2.2 设置 `nginx` 反向代理

`vi /etc/nginx/sites-available/wx.your-domain.com.conf`

```nginx
server {
    listen 80;
    server_name wx.your-domain.com;

    location / {
        # 8080 设置为 frps 的 vhost_http_port
        proxy_pass http://localhost:8080; 
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/wx.your-domain.com.conf /etc/nginx/sites-enabled/wx.your-domain.com.conf
sudo systemctl reload nginx
```

*记得配置下阿里云服务器的防火墙，放开7000端口。。*

## 3. 客户端

### 3.1 先做个测试

修改 win10 的 `hosts`:
`192.168.10.10 wx.your-domain.com`，验证下：

```bat
ping wx.your-domain.com
```

本地已经有 `Laravel` 项目存在，打开浏览器测试 `wx.your-domain.com`，浏览正常。

### 3.2 配置 `frpc`

先从服务器下载： `scp root@your-domain.com:/root/frp_0.24.1_linux_amd64.tar.gz .`，比从 `github` 下载快多了。。。

解压并编辑 `frpc.ini`:

```ini
[common]
server_addr = 1.2.3.4
server_port = 7000
auto_token = your-token

[web]
type = http
local_ip = 127.0.0.1
local_port = 80
custom_domains = wx.your-domain.com
```

启动

```bash
vagrant@homestead:~/code/frp_0.24.1_linux_amd64$ ./frpc -c ./frpc.ini
2019/02/25 15:54:27 [I] [service.go:214] login to server success, get run id [dacdb8f2e391b0f7], server udp port [0]
2019/02/25 15:54:27 [I] [proxy_manager.go:137] [dacdb8f2e391b0f7] proxy added: [web]
2019/02/25 15:54:27 [I] [control.go:143] [web] start proxy success
```

### 3.3 验证

修改 win10 的，删除之前添加的那行 `192.168.10.10 wx.your-domain.com`，验证下：

```bat
ping wx.your-domain.com
```
没问题，打开浏览器测试 `wx.your-domain.com`，浏览正常。

### 3.4 微信接口配置信息

使用了 `overtrue/laravel-wechat:~4.0`，路由为 `Route::any('/wechat', 'WeChatController@serve');`，看下 `WeChatController@serve`方法：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Log;

class WeChatController extends Controller
{
    public function serve(){
        app('debugbar')->disable();
        Log::info('wechat request arrived.');
        $app = app('wechat.official_account');
        return $app->server->serve();
    }
}
```

再到微信测试号修改接口配置信息:
URL: `wx.your-domain.com/wechat`
，总算配置成功了！

## Reference

* [内网穿透：在公网访问你家的 NAS](https://zhuanlan.zhihu.com/p/57477087)
* [frp](https://github.com/fatedier/frp)
* [frp_0.24.1_linux_amd64.tar.gz](https://github.com/fatedier/frp/releases/download/v0.24.1/frp_0.24.1_linux_amd64.tar.gz)
* [使用frp内网穿透实现微信本地调试](https://www.lwhweb.com/2017/11/02/use-frp-forward-wechat-message-local-machine/)
* [使用frp微信本地调试](https://www.jianshu.com/p/16d7ff75257a)
* [部署FRP服务 实现内网穿透](https://lfire.github.io/2017/11/15/deploy-the-frp-server/)
* [nginx + frp 实现内网穿透进行微信本地开发](https://blog.csdn.net/jackymvc/article/details/80594619)
