---
layout: post
title:  "梯子制作方法"
subtitle: "越过长城，走向世界"
date:   "2018-2-18" 
author: "cj"
tags:
    gfw
    vps
    shadowsocks
---

> 转载请保留[原文链接](http://wangyapeng.me/2018/02/18/ladder-tutorial/)

总结一下梯子制作方法以便交流。

# 1. 概述

购买物理坐标位于国外的虚拟主机，安装shadowsocks服务。本地计算机或手机或路由器安装shadowsocks客户端，即可越过长城，走向世界。

# 2. 选购虚拟主机

- 2.1 访问[50vz vps](https://www.50vz.net/aff.php?aff=853)(点这个链接有10块钱优惠哦~)

![img](https://wangyapeng-me-1251421422.cos.ap-shanghai.myqcloud.com/ladder/step1.webp)

- 2.2 点马上选购，分类选LowEnd-VZ

![img](https://wangyapeng-me-1251421422.cos.ap-shanghai.myqcloud.com/ladder/step2.webp)

- 2.3 选择*凤凰城 - N1*，下一步

![img](https://wangyapeng-me-1251421422.cos.ap-shanghai.myqcloud.com/ladder/step3.jpg)

- 2.4 继续

![img](https://wangyapeng-me-1251421422.cos.ap-shanghai.myqcloud.com/ladder/step4.jpg)

- 2.5 同意条款

![img](https://wangyapeng-me-1251421422.cos.ap-shanghai.myqcloud.com/ladder/step5.jpg)

- 2.6 结算

![img](https://wangyapeng-me-1251421422.cos.ap-shanghai.myqcloud.com/ladder/step6.webp)

- 2.7 填写账户信息，注意邮箱，稍后会将登陆凭据发送到该邮箱

![img](https://wangyapeng-me-1251421422.cos.ap-shanghai.myqcloud.com/ladder/step7.jpg)

- 2.8 支付宝扫码付款

![img](https://wangyapeng-me-1251421422.cos.ap-shanghai.myqcloud.com/ladder/step8.jpg)

- 下面这几步不重要
  - 2.9* 付款完成后返回会员中心
    ![img](https://wangyapeng-me-1251421422.cos.ap-shanghai.myqcloud.com/ladder/step9.webp)
  - 2.10* 会员中心里点击刚才买到的虚拟主机
    ![img](https://wangyapeng-me-1251421422.cos.ap-shanghai.myqcloud.com/ladder/step10.jpg)
  - 2.11* 查看主机信息
    ![img](https://wangyapeng-me-1251421422.cos.ap-shanghai.myqcloud.com/ladder/step11.jpg)

- 2.12 查看邮件（重要信息在这）IP、用户名、密码记住，等下要用

![img](https://wangyapeng-me-1251421422.cos.ap-shanghai.myqcloud.com/ladder/check_mail.jpg)

# 3. 安装远程连接工具putty

## 3.1 下载

点击下载[putty](https://the.earth.li/~sgtatham/putty/latest/w32/putty-0.70-installer.msi)

## 3.2 安装

一路next，到这一步可以选择是否创建桌面快捷方式

![img](https://wangyapeng-me-1251421422.cos.ap-shanghai.myqcloud.com/ladder/putty_create_dsktp_ico.png)

# 4. 使用putty连接虚拟主机

## 4.1 运行putty

![img](https://wangyapeng-me-1251421422.cos.ap-shanghai.myqcloud.com/ladder/putty_open.webp)

填写邮件中登录凭据显示的IP，默认端口22，点击open

## 4.2 忽略安全警告

![img](https://wangyapeng-me-1251421422.cos.ap-shanghai.myqcloud.com/ladder/putty_secure_alert.png)

## 4.3 输入用户名root

![img](https://wangyapeng-me-1251421422.cos.ap-shanghai.myqcloud.com/ladder/putty_login_as.png)

## 4.4 输入密码

先在邮件内复制好密码，在putty界面直接右键单击即可粘贴密码，密码不会显示，回车

![img](https://wangyapeng-me-1251421422.cos.ap-shanghai.myqcloud.com/ladder/putty_login_psw.png)

## 4.5 修改默认端口

为防止网络攻击，修改默认的远程登录端口为其他数字，1~65535之间，如8888

![img](https://wangyapeng-me-1251421422.cos.ap-shanghai.myqcloud.com/ladder/crt_change_ssh_port.png)

## 4.6 等待虚拟主机初始化并重启

![img](https://wangyapeng-me-1251421422.cos.ap-shanghai.myqcloud.com/ladder/crt_init_done.png)

## 4.7 重新登录

关闭putty，重新运行。修改默认端口为刚才填写的端口号如8888，重新登录，登录成功后如下图：

![img](https://wangyapeng-me-1251421422.cos.ap-shanghai.myqcloud.com/ladder/putty_login_ok.png)

# 5. 开启shadowsocks服务

## 5.1 安装

在putty内输入以下命令，回车运行

``` shell
yum install python-setuptools -y && easy_install pip && pip install shadowsocks
```

## 5.2 运行

在putty内输入以下命令，回车运行

``` shell
sudo ssserver -p 443 -k password -m aes-256-cfb --user nobody -d start
```

其中443为代理端口，password为密码，可修改，稍后运行的shadowsocks客户端要使用。

## 5.3 完成

可以关闭putty了

# 6. Windows安装shadowsocks客户端

## 6.1 下载

[下载地址](https://github.com/shadowsocks/shadowsocks-windows/releases/download/4.0.8/Shadowsocks-4.0.8.zip)

下载下来是一个zip压缩包，解压并运行

![img](https://wangyapeng-me-1251421422.cos.ap-shanghai.myqcloud.com/ladder/ss_run_client.png)

## 6.2 配置

运行后没有主界面，在系统托盘区找到它的小飞机图标，右键单击，点击 服务器--编辑服务器

![img](https://wangyapeng-me-1251421422.cos.ap-shanghai.myqcloud.com/ladder/ss_config.png)

服务器地址填写虚拟主机的IP，端口、密码填写5.2设置的内容，如443、password，点击确定。

勾选启用系统代理；可选开机启动；点击 PAC--从GFWList更新本地PAC。

![img](https://wangyapeng-me-1251421422.cos.ap-shanghai.myqcloud.com/ladder/ss_config2.webp)

## 6.3 测试

访问墙外的网站测试，如[谷歌](www.google.com)

![img](https://wangyapeng-me-1251421422.cos.ap-shanghai.myqcloud.com/ladder/google.webp)

# 7. Android安装shadowsocks客户端影梭

## 7.1 下载

[官方下载](https://github.com/shadowsocks/shadowsocks-android/releases/download/v4.4.6/shadowsocks--universal-4.4.6.apk)

[百度网盘](https://pan.baidu.com/s/1pMpXwW7) 密码: q1hd

## 7.2 运行

![img](https://wangyapeng-me-1251421422.cos.ap-shanghai.myqcloud.com/ladder/android-1.webp)

## 7.3 配置

点击上图中的铅笔按钮进行配置

![img](https://wangyapeng-me-1251421422.cos.ap-shanghai.myqcloud.com/ladder/android-2.webp)

服务器、端口、密码填写5.2设置的内容；路由设置为“绕过局域网及中国大陆地址”，也可按需选择其他选项；保存。

## 7.3 开启代理

点击7.2图中的小飞机按钮进行开启和关闭代理。

![img](https://wangyapeng-me-1251421422.cos.ap-shanghai.myqcloud.com/ladder/android-3.webp)

## 7.4 测试

![img](https://wangyapeng-me-1251421422.cos.ap-shanghai.myqcloud.com/ladder/android-4.webp)

# 8. 苹果设备

暂未发现支持方法。可通过9方法开启WiFi供苹果设备使用梯子。

# 9. OpenWrt内核路由器安装shadowssocks客户端插件

openwrt内核的路由器可以支持安装插件，安装shadowsocks客户端插件后可以向局域网内直连、WiFi连接的设备提供梯子。我仅以极路由3（HiWiFi HC5861）测试成功过，其他品牌、其他型号的路由器需自行搜索刷机、配置方法。

附上[极路由刷机教程](https://github.com/rssnsj/openwrt-hc5x61)