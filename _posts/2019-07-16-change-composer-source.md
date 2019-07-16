---
layout: post
title:  "Composer源更换为阿里云镜像"
subtitle: "哈死老子了！"
date:   "2019-07-16"
author: "cj"
tags:
    composer
    aliyun
---

# Composer源更换为阿里云镜像

一直使用的 `Laravel-China` 镜像将要功成身退：[Laravel China 镜像完成历史使命，将于两个月后停用](https://learnku.com/articles/30758) ——感谢一路陪伴！

同时贴心地给出了 [Composer 国内加速：可用镜像列表](https://learnku.com/composer/wikis/30594)，既然有阿里：[阿里云 Composer 全量镜像](https://developer.aliyun.com/composer)，就用阿里了：

`composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/`

更新线上项目同时修复 `lodash` 漏洞（[Lodash库爆出严重安全漏洞，波及400万+项目](https://mp.weixin.qq.com/s?src=11&timestamp=1563266500&ver=1731&signature=qcHtck-YfjAaelLElxAg1C*lc1zcjAnT5iuY4fA*NfwD5FVhhysLoYFsIxCKDR1k*BRan0t5ty79ONDQ0*ZRC*4d8LDFmEuwzj-y0XMSmlmfyHfu1-e4i5VAeaIhMZmW&new=1)） 时却出了问题，哈死老子了！

问题是这样滴：

```bash
sudowww 'composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/'
sudowww 'composer install'
```

然后居然出现了 [Composer 安装依赖时让输入用户名和密码](https://learnku.com/laravel/t/30378) 这个问题，赶紧按照 [Laravel 安装和开发环境：修改项目依赖为新的镜像地址](https://learnku.com/laravel/wikis/16722) ：

```bash
sudowww 'composer update nothing'
```

居然不行！一堆错误，吓得老子赶紧 `rm -rf vendor/ && sudowww 'composer install -vvv'`，还是要输入用户名和密码！

回到本地开发环境，手动把 `composer.lock` 文件内的 `dl.laravel-china.org` 全部替换为 `mirrors.aliyun.com/composer/dists`，紧急 `git push`，再到线上环境 `sudowww 'rm -f composer.lock && rm -rf vendor/ && git pull && composer install -vvv'`搞定！

哈死老子了！！！


注：`sudowww` 是一个别名：`alias sudowww='sudo -H -u www-data sh -c'`