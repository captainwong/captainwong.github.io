---
layout: post
title:  "Laravel 广播"
subtitle: "使用 redis + socket.io 等进行 websocket 通信"
date:   "2019-03-30"
author: "cj"
tags:
    laravel
    redis
    websocket
    laravel-echo-server
    socket.io
    laravel-echo
---

# Laravel5.7 广播

环境为 `homestead/ubuntu18.04/laravel5.7/php7.2` 。。。

本想用 `pusher` ，他官网访问极慢，换 `redis` 。。。

## 大致步骤

1. 编辑 `config/app.php`，取消对 `BroadcastServiceProvider` 的注释
2. 编辑 `.env`，修改 `BROADCAST_DRIVER=redis`
3. `yarn global add laravel-echo-server --prefix /usr/local --no-bin-links`
4. `laravel-echo-server init`

    除了第一项 `development mode` 选 `yes` 其他一路回车默认
5. 创建 `event`, `app/Events/EventMachineOperated.php`，真实项目，摘录一些关键代码
    ```php
    class EventMachineOperated implements ShouldBroadcast
    {
        public $message; // public 的成员都将广播出去

        public function broadcastOn()
        {
            return new Channel('machineStatus');
        }
    }
    ```
6. 编辑 `routes/channels.php`，对这个频道暂时不做权限控制
    ```php
    Broadcast::channel('machineStatus', function () {
        return true;
    });
    ```
7. `yarn add socket.io-client`
8. `yarn add laravel-echo`
9. 编辑 `/resources/assets/js/bootstrap.js`，最底下取消注释稍作修改

    ```php
    import Echo from "laravel-echo"

    window.io = require('socket.io-client');

    window.Echo = new Echo({
        broadcaster: 'socket.io',
        host: window.location.hostname + ':6001'
    });
    ```
10. 编辑 `resources/views/pages/test_websocket.blade.php`
    ```xml
    <!DOCTYPE html>
    <html lang="{{ app()->getLocale() }}">
    <head>
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1">

        <!-- CSRF Token -->
        <meta name="csrf-token" content="{{ csrf_token() }}">

        <title>Websocket Test</title>

        <!-- Styles -->
        <link href="{{ mix('css/app.css') }}" rel="stylesheet">
    </head>
    <body>
        <div id="app">
            <div class="content">
                <div class="m-b-md">
                    Websocket Test
                </div>
            </div>
        </div>

        <!-- receive notifications -->
        
        <script src="http://{{ Request::getHost() }}:6001/socket.io/socket.io.js"></script>
        <script src="{{ mix('js/app.js') }}"></script>

        <script>
            //Pusher.logToConsole = true;

            Echo.channel('machineStatus')
                .listen('EventMachineOperated', (e) => {
                    console.log('triggered');
                    console.log(e);
                });
        </script>
        <!-- receive notifications -->
    </body>
    </html>
    ```

11. `npm run dev`
12. `php artisan horizon`
13. `laravel-echo-server start`
14. 创建了个测试 `api`，使用 `postman` 调用接口后触发 `EventMachineOperated` 事件
15. 创建相关路由，不提
16. 浏览器打开页面 `http://your-website/testWebsocket`，按下 <kbd>F12</kbd> 开调试模式，使用 `postman` 触发事件，有 `log` 产生，大功告成。
17. 创建私有频道时，需修改 `laravel-echo-server.json` 内 `authHost` 设置项为真实域名，我这边使用 `http://localhost` 失败鸟。

## frpc 反向代理设置

由于我的环境是域名指向公网阿里云服务器，并配置了 `frps` 做转发。`websocket` 通信用到的 6001 端口需要在防火墙放开，且需要本地 `homestead` 内修改 `frpc` 做相应设置。`frp` 相关资料参阅 [使用frp搭建微信开发环境](http://wangyapeng.me/2019/02/25/build-local-wechat-dev-env-with-frp/)。

`frpc.ini`

```ini
[websocket]
type = tcp
local_ip = 127.0.0.1
local_port = 6001
remote_port = 6001
```

测试时可以先执行 `laravel-echo-server start`，然后启动 `frpc`: `./frpc -c frpc.ini`，在浏览器内输入 `http://your-site:6001` 做测试，如果输出 `OK` 说明配置成功。


## 生产环境部署

*2019年4月3日21:04:19新增*

当在生产环境部署时，由于网站开启了全站 `HTTPS`, 客户端一直无法认证通过。

查看[官方文档](https://github.com/tlaverdure/laravel-echo-server)，可以让 `js` 代码不再显示走 `6001` 端口，而是配置 `nginx` 转发解决。

- 在网站的 `nginx conf` 中新增：

    ```conf
    location /socket.io {
        proxy_pass http://localhost:6001; #could be localhost if Echo and NginX are on the same box
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
    }
    ```

- 修改 `laravel-echo-server.conf` 中 `authHost` 设置项为真实域名如 `https://your-domain.com`

- 修改 `app/resources/assets/js/bootstrap.js`，去掉显示指定的 `6001` 端口：

    ```js
    window.Echo = new Echo({
        broadcaster: 'socket.io',
        host: window.location.hostname
    });        
    ```

- 修改引入 `socket.io.js` 的 `blade view`，同样去除 `6001` 端口：
    ```html
    <script src="https://\lbrace \lbrace  Request::getHost() }}{% endraw %}/socket.io/socket.io.js"></script>
    ```

如此一来，浏览器端都是访问的 `https://your-domain.com`，让 `nginx` 帮我们处理 `location` 为 `socket.io` 的情况，自动转发流量到 `laravel-echo-server` 的服务接口 `http://localhost:6001`，完美。

## Reference

- [广播系统](https://learnku.com/docs/laravel/5.7/broadcasting/2277)
- [由浅入深：基于 Laravel Broadcast 实现 WebSocket C/S 实时通信](https://laravelacademy.org/post/8559.html)
- [Laravel 5.7 中广播实践，使用websocket（Redis + socket.io） 技术接收](https://yq.aliyun.com/articles/667300#)
- [基于 Redis驱动的 Laravel 事件广播](https://www.ctolib.com/topics-130749.html)
- [内网穿透：在公网访问你家的 NAS](https://zhuanlan.zhihu.com/p/57477087)
- [tlaverdure/laravel-echo-server](https://github.com/tlaverdure/laravel-echo-server)