---
layout: post
title:  "升级到Laravel8及PHP8.0小记"
subtitle: "老了，跟不上时代了"
date:   "2021-05-18"
author: "cj"
tags:
    laravel
    php
    web
---

# 升级到Laravel8及PHP8.0小记

环境：
* Ubuntu16.04 64bit
* PHP 7.2.34-21+ubuntu16.04.1+deb.sury.org+1 (cli) (built: May  1 2021 11:52:36) ( NTS )
* nginx 1.16.1
* mysql Ver 14.14 Distrib 5.7.33, for Linux (x86_64) using  EditLine wrapper
* ...

## 1. 安装PHP8

```bash
apt-get install -y php8.0-bcmath php8.0-cli php8.0-curl php8.0-fpm php8.0-gd php8.0-mbstring php8.0-mysql php8.0-opcache php8.0-pgsql php8.0-readline php8.0-xml php8.0-zip php8.0-sqlite3 php8.0-redis
```

## 2. 设置默认PHP版本为7.2

由于有其他项目仍在使用 Laravel5.x/6.x 以及 PHP7.2，修改默认 PHP 版本为 7.2 以兼容之。

```bash
sudo update-alternatives --set php /usr/bin/php7.2
sudo update-alternatives --set phar /usr/bin/phar7.2
sudo update-alternatives --set phar.phar /usr/bin/phar.phar7.2
sudo update-alternatives --set phpize /usr/bin/phpize7.2
sudo update-alternatives --set php-config /usr/bin/php-config7.2
```

主要是 php，其他失败了也不要紧。

## 3. 升级 composer 到 2.*

性能提升不小，更新它

```bash
composer self-update
```

## 4. 升级 nodejs 到 v14.x

貌似一夜之间大家都升级了。。。

参考 [官方文档](https://github.com/nodesource/distributions/blob/master/README.md)

```bash
curl -fsSL https://deb.nodesource.com/setup_14.x | sudo -E bash -
sudo apt-get install -y nodejs
```

## 5. 部署 Laravel8 项目 （larabbs8）

需要考虑的地方有 nginx conf, cron job, supervisor conf, deployer 部署脚本等。

### 5.1 nginx conf 指定PHP版本

sites-enabled 里的 conf，之前是：
```conf
fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
```
修改为：
```conf
fastcgi_pass unix:/var/run/php/php8.0-fpm.sock;
```

### 5.2 deployer 部署脚本指定 php 及 composer 版本

参考 [stackoverflow](https://stackoverflow.com/questions/49049552/how-to-tell-deployer-to-use-different-php-version-once-sshed-to-my-shared-hosti)，在 `require 'recipe/laravel.php';` 下方增加：

```php
set('bin/php', function () {
    return '/usr/bin/php8.0';
});

set('bin/composer', function () {
    return '/usr/bin/php8.0 /usr/local/bin/composer';
});
```

### 5.3 cron job 指定 PHP 版本

```
* * * * * php8.0 /var/www/larabbs8-deployer/current/artisan schedule:run >> /dev/null 2>&1
```

### 5.4 supervisor conf 指定 PHP 版本

```conf
[program:bbs8]
process_name=%(program_name)s
command=php8.0 /var/www/larabbs8-deployer/current/artisan horizon
autostart=true
autorestart=true
user=www-data
redirect_stderr=true
stdout_logfile=/var/www/larabbs8-deployer/shared/storage/logs/horizon.log
```


