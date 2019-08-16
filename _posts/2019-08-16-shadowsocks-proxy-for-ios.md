---
layout: post
title:  "在iOS上使用shadowsocks代理"
subtitle: "gfw go to hell"
date:   "2019-08-16"
author: "cj"
tags:
    shadowsocks
    privoxy
    iOS
    ubuntu
---

# 在iOS上使用shadowsocks代理

我以前写过一篇 [梯子制作教程](http://wangyapeng.me/2018/02/18/ladder-tutorial/)，当时没有搞定 `iOS` 的代理，直接走的 `openwrt` 路由器代理，现在极路由坏了换了个 `Tenda` 路由器没这功能了，好久都没在果机上收过 `Gmail` 邮件了，开网页又嫌麻烦，每次查看邮箱都几十封在那躺着。。。

今天再试一下，我 `Windows PC` 上有 `shadowsocks` 客户端在运行，IP `192.168.1.90`，开放局域网连接端口 `8080` ，在苹果的 `设置`-`无线局域网`-`WiFi名` 上点感叹号，配置 `HTTP` 代理为手动，服务器、端口分别填写 `192.168.1.90`、`8080`，然后正准备打开浏览器上 `google.com` 测试呢，呼啦啦几十封 `Gmail` 未读邮件过来了。。。

证明手动 `HTTP` 代理，让流量走 `PC` 是可行的。但也不能24小时开着电脑啊，刚好有台 `Ubuntu 16.04.6 Server` 一直在运行，搞起来。

## 一、安装 shadowsocks

1. 安装

    ```bash
    pip install shadowsocks
    ```

2. 配置

    写 `sslocal` 的配置文件：`/etc/sslocal.json`

    ```json
    {
            "server":"your-ssserver-ip",
            "server_port":your-ssserver-port,
            "password":"your-sserver-password",
            "method":"aes-256-cfb",
            "timeout":300,
            "local_address":"0.0.0.0",
            "local_port":10443
    }
    ```

    是的，前提是你得有自己的 `your-ssserver` 。。。不提。

    关键是设置 `local_address` 和 `local_port` ，相当于 `Windows PC` 上 `shadowsocks` 客户端的 `允许来自局域网的连接` 选项。

3. 开机启动 optional

    写 `supervisor` 脚本 `/etc/supervisor/conf.d/sslocal.conf`

    ```conf
    [program:sslocal]
    process_name=%(program_name)s
    command=sslocal -c /etc/sslocal.json start
    autostart=true
    autorestart=true
    user=root
    redirect_stderr=true
    stdout_logfile=/home/jack/sslocal.log
    ```

    `supervisorctl update` 使之生效。

4. 失败

    果机尝试 `HTTP` 代理，手动、自动都不行，查了一下，`shadowsocks` 提供的是 `SOCKS5` 代理，果机要求的是 `HTTP` 代理，不通是正常滴。再次搜索，发现可以用 `privoxy` 将 `SOCKS5` 代理转换为 `HTTP` 代理。

## 二、安装 `privoxy`

1. 安装

    `apt install privoxy`

2. 配置

    `vi` `/etc/privoxy/config` 文件，善用 `?keyword` 进行搜索。。。

    找到 `listen-address` 修改为 `listen-address  0.0.0.0:18118`。

    找到 `actionsfile`，追加一行 `actionsfile gfw.action`，等下创建 `gfw.action`。

3. 生成 `gfw.action`

    先安装 `gfwlist2privoxy`，它可以用来将 `gfwlist` 文件转换为 `privoxy` 的 `action` 文件格式。

    ```bash
    pip install gfwlist2privoxy
    wget https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt
    gfwlist2privoxy -i gfwlist.txt -f gfw.action -p 127.0.0.1:10443 -t socks5
    systemctl restart privoxy.service
    ```

4. 测试

    果机设置 `HTTP` 手动代理，`IP` 为 `Ubuntu` 的 `IP` `192.168.1.168`，端口 `18118` 。打开谷歌，开了！

## 总结

果机流量路线：

果机 `HTTP` 代理 -> `privoxy` -> `sslocal` -> `ssserver` -> `service` 

其中，走到 `privoxy` 时，`privoxy` 会根据 `gfw.action` 自动过滤，在其中的转发，不在其中的我看是没有转发的，直接放行了，因为打开微信网页什么的速度很快，比打开谷歌快多了。。。

我只求能收个 `Gmail` 邮件，更详细的就不研究了。总的来说，这个方法仅适合在自己的 `WiFi` 下使用，且需要局域网内有服务器一直运行，并不适合出门在外时通过移动流量翻墙，使用 `Android` 手机更方便。

## Reference

* [shadowsocks账号配置局域网的全局代理方案。可以给局域网内的pc，ios，android使用](https://my.oschina.net/angyr/blog/685372)
* [Linux 使用 ShadowSocks + Privoxy 实现 PAC 代理](https://huangweitong.com/229.html)
