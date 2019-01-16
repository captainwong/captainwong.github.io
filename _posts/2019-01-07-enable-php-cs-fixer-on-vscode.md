---
layout: post
title:  "VS Code 使用 php-cs-fixer 优化代码格式"
subtitle: "WEB圈的技术栈实在是不熟悉啊，我还是觉得C/C++用着更爽。。。"
date:   "2019-01-07"
author: "cj"
tags:
    web
    vscode
    php-cs-fixer
    php
---

# `VS Code` 使用 `php-cs-fixer` 优化代码格式

## Env

* Win10 Pro 1809 64bit
* VS Code 1.30.1
* PHP 7.2.13-Win32-VC15-x64

## Setup

1. 下载 [php-cs-fixer.phar](https://github.com/FriendsOfPHP/PHP-CS-Fixer/releases/download/v2.13.3/php-cs-fixer.phar) 到php目录 (`E:\local_program\php-7.2.13-Win32-VC15-x64`)

2. `VS Code` 商店内搜索安装 `php-cs-fixer.phar` 插件

3. `VS Code` 配置
    ``` json
    {

        "php.validate.executablePath": "E:\\local_program\\php-7.2.13-Win32-VC15-x64\\php.exe",

        "php-cs-fixer.executablePath": "E:\\local_program\\php-7.2.13-Win32-VC15-x64\\php-cs-fixer.phar",

        "php-cs-fixer.lastDownload": 1546848191171,

        "php-cs-fixer.autoFixBySemicolon": true,

        "php-cs-fixer.onsave": true,

    }
    ```
4. 快捷键 <kbd>Shift</kbd> + <kbd>Alt</kbd> + <kbd>F</kbd> 进行格式化 PHP 代码