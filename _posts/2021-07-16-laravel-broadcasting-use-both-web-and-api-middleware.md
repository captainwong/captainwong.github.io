---
layout: post
title:  "Laravel Broadcasting 同时启用 web 和 api 两个 Middleware"
subtitle: "妙啊"
date:   "2021-07-16"
author: "cj"
tags:
    php
    laravel
    broadcasting
    websocket
    laravel-echo
    socket.io
---

# Laravel Broadcasting 同时启用 web 和 api 两个 Middleware

最近在给项目做 `API` 接口，大体搞定了，但是缺少实时推送功能。既然 `web` 版可以用 `websocket` 接收推送，`api` 为何不可呢？

核心在 `app/Providers/BroadcastServiceProvider.php`:

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\Broadcast;
use Illuminate\Support\Facades\Route;
use Illuminate\Http\Request;

class BroadcastServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        //Broadcast::routes();

        Route::post('broadcasting/auth', function () {
            $url = request()->hasHeader('Authorization') ? '/api' : '/web';         
            $proxy = Request::create($url . '/broadcasting/auth', 'POST', request()->toArray());
            $proxy->headers = request()->headers;
         
            return app()->handle($proxy)->content();
         });
         
        Broadcast::routes(['prefix' => 'web', 'middleware' => ['web']]);
        Broadcast::routes(["prefix" => "api", "middleware" => ['api', 'jwt.auth']]);

        require base_path('routes/channels.php');
    }
}

```

如果请求头中带有 `Authorization`，则走 `auth:api` 中间件，不带就走 `auth:web`。


`laravel-echo-server.json` 中 `authEndpoint` 字段仍然保持不变：`/broadcasting/auth`。

新增一个 `resources/js/test.js` 用来测试 `api`：

```js
window.$ = window.jQuery = require('jquery');

import Echo from "laravel-echo"

window.io = require('socket.io-client');

window.Echo = new Echo({
    broadcaster: 'socket.io',
    host: window.location.hostname,
    auth:
    {
        headers:
        {
            'Authorization': 'Bearer {ACCESS_TOKEN_GOES_HERE}'
        }
    }
});
```

暂时把 `access_toekn` 写死了，我目的并不是在网页上这样用，而是要在 `APP`里 或者 `C++` 客户端里使用 `websocket` 接收推送，所以这里写死了做个验证。

不要忘了修改 `webpack.mix.js` 然后 `yarn dev` 编译一下:

```js
mix.js('resources/js/app.js', 'public/js')
   .sass('resources/sass/app.scss', 'public/css')
   .sass('resources/sass/wechat.scss', 'public/css')
   .js('resources/js/test.js', 'public/js')
   .version();
```

新增一个 `web` 页面 `resources/views/pages/test_websocket.blade.php`:

```php
<html>
<body>
<p>     
    Websocket Test
</p>

<script src="{{ mix('js/test.js') }}"></script>
<script src="https://{{Request::getHost()}}/socket.io/socket.io.js"></script>
<script>
  
$(document).ready(function(){
  Echo.private('some-private-channel')      
        .listen('.some-event', (e) => {
            console.log(e);     
        });
});
    
</script>
</body>
</html>
```


打开这个网页，进入控制台，可以接收推送；打开原有的走 `auth:web` 的带有推送功能的网页，可以接收推送，大功告成。

TODO：C++ 客户端调用 `api` 获取 `token` 后加入 `websocket channel` 接收推送

## Reference

* [Laravel-echo/server 结合 JWT 配置方式](https://learnku.com/laravel/t/17293) 这个方案只能启用 `auth:api`，不过是个好的开始

* [Laravel Broadcast - Combining multiple middleware (web, auth:api)](https://stackoverflow.com/questions/48934892/laravel-broadcast-combining-multiple-middleware-web-authapi/48941457#48941457) `stackoverflow` 上也有人提出了同样的问题，吾道不孤

* [HOWTO: broadcasting, laravel-echo, laravel-echo-server and JWT](https://laravel.io/forum/10-09-2016-howto-broadcasting-laravel-echo-laravel-echo-server-and-jwt) 很好的资料，同样是仅使用 `auth:api` 一个中间件

* [Using both 'web' & 'auth:api' middlewares at the same time for Broadcast Events](https://github.com/laravel/framework/issues/23268#issuecomment-542890314) 最终的解决方案来了，`gayhub` yyds 23333

