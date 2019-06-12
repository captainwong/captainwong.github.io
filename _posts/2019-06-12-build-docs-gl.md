---
layout: post
title:  "docs.gl 搭建过程"
subtitle: "日有所得"
date:   "2019-06-12"
author: "cj"
tags:
    opengl
    docs.gl
    nginx
---

# docs.gl 搭建过程

最近在学习 `OpenGL`，如 [docs.gl](https://github.com/BSVino/docs.gl) 作者所言：

>The actual documentation website provided by Khronos is in frames and poorly formatted, difficult to navigate and search.

[官方文档](https://www.khronos.org/registry/OpenGL-Refpages/gl4/) 确实难看啊，幸好有 [docs.gl](https://github.com/BSVino/docs.gl) 救场。

1. `cd /var/www/data && sudo git clone https://github.com/BSVino/docs.gl`
2. `sudo python build.py --full`，生成 `htdocs` 文件夹存放了所有网站内容。由于生成的网页都不带 `html` 后缀名，直接使用 `ngxin` 站点，打开链接无法浏览，会下载文件。。因此使用 `ngxin` 代理到 `docs.gl` 提供的 `server.py` 简单 `python` 网站服务。
3. 使用 `nginx` 代理，配置如下：
    ```nginx
    server {
        listen 80;
        server_name gl.doc.********.com;

        location / {
            proxy_pass http://localhost:8001;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }

    }
    ```
4. 尝试运行：`cd htdocs && sudo python ../server.py`，打开浏览器可以正常访问、搜索
5. 一直开着 `putty` 也不是事，加个 `supervisor` 配置 `gldocs.conf`：
    ```conf
    [program:gldoc]
    process_name=%(program_name)s
    directory=/var/www/docs.gl/htdocs
    command=python /var/www/docs.gl/server.py
    autostart=true
    autorestart=true
    user=www-data
    redirect_strerr=true
    stdout_logfile=/var/www/docs.gl/server.log
    ```
    `sudo supervisorctl update` 搞定。
6. `server.log` 有问题。虽然生成了文件，但是是 `root` 用户的，而且大小一直是0。手动 `chown` 给 `www-data` 用户，依然如故。猜测是用户权限或 `python` 的标准输出有问题？今天很困，抽空研究研究。