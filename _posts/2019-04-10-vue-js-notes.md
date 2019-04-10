---
layout: post
title:  "Vue.JS 学习笔记"
subtitle: "学习一个"
date:   "2019-04-10"
author: "cj"
tags:
    vue.js
    homestead
---

# Vue.JS 学习笔记

为了快速上手，学习 [教程](https://learnku.com/courses/vuejs-essential)，中间踩坑无数，对我这个前端超级菜鸟来说好难啊！把踩过的坑记录下来，以备将来回来嘲笑自己。。。

## 安装 `vue-cli@2.9.3`

```bash
vagrant@homestead:~$ npm install --global vue-cli@2.9.3
npm WARN deprecated coffee-script@1.12.7: CoffeeScript on NPM has moved to "coffeescript" (no hyphen)
npm WARN checkPermissions Missing write access to /usr/lib/node_modules
npm ERR! path /usr/lib/node_modules
npm ERR! code EACCES
npm ERR! errno -13
npm ERR! syscall access
npm ERR! Error: EACCES: permission denied, access '/usr/lib/node_modules'
npm ERR!  { Error: EACCES: permission denied, access '/usr/lib/node_modules'
npm ERR!   stack: 'Error: EACCES: permission denied, access \'/usr/lib/node_modules\'',
npm ERR!   errno: -13,
npm ERR!   code: 'EACCES',
npm ERR!   syscall: 'access',
npm ERR!   path: '/usr/lib/node_modules' }
npm ERR!
npm ERR! The operation was rejected by your operating system.
npm ERR! It is likely you do not have the permissions to access this file as the current user
npm ERR!
npm ERR! If you believe this might be a permissions issue, please double-check the
npm ERR! permissions of the file and its containing directories, or try running
npm ERR! the command again as root/Administrator (though this is not recommended).

npm ERR! A complete log of this run can be found in:
npm ERR!     /home/vagrant/.npm/_logs/2019-04-10T01_54_11_323Z-debug.log
```

使用 `sudo` 解决

## 项目初始化

```bash
vagrant@homestead:~/code$ vue init webpack vuejs-essential

? Project name vuejs-essential
? Project description A Vue.js project
? Author jack-homestead <1281261856@qq.com>
? Vue build standalone
? Install vue-router? No
? Use ESLint to lint your code? No
? Set up unit tests No
? Setup e2e tests with Nightwatch? No
? Should we run `npm install` for you after the project has been created? (recom
mended) npm

vue-cli · Generated "vuejs-essential".


# Installing project dependencies ...
# ========================

npm WARN deprecated browserslist@2.11.3: Browserslist 2 could fail on reading Browserslist >3.0 config used in other tools.
npm WARN deprecated bfj-node4@5.3.1: Switch to the `bfj` package for fixes and new features!
npm WARN deprecated browserslist@1.7.7: Browserslist 2 could fail on reading Browserslist >3.0 config used in other tools.
Unhandled rejection Error: EACCES: permission denied, open '/home/vagrant/.npm/_cacache/index-v5/d0/4d/0d779b188af55718b4171d26d01a41d0c46ab27bc176dc227543a131f462'

Unhandled rejection Error: EACCES: permission denied, open '/home/vagrant/.npm/_cacache/index-v5/d0/4d/0d779b188af55718b4171d26d01a41d0c46ab27bc176dc227543a131f462'

Unhandled rejection Error: EACCES: permission denied, open '/home/vagrant/.npm/_cacache/index-v5/d0/4d/0d779b188af55718b4171d26d01a41d0c46ab27bc176dc227543a131f462'

npm ERR! cb() never called!

npm ERR! This is an error with npm itself. Please report this error at:
npm ERR!     <https://npm.community>

npm ERR! A complete log of this run can be found in:
npm ERR!     /home/vagrant/.npm/_logs/2019-04-10T01_58_29_310Z-debug.log

# Project initialization finished!
# ========================

To get started:

cd vuejs-essential
npm run dev

Documentation can be found at https://vuejs-templates.github.io/webpack
```

解决：

到 `Should we run 'npm install' for you after the project has been created?` 的时候选择稍后手动运行，结束当前命令后自己再运行 `yarn --no-bin-links`。

## 报 `webpack-dev-server` 命令未找到

```bash
vagrant@homestead:~/code/vuejs-essential$ npm run dev

> vuejs-essential@1.0.0 dev /home/vagrant/code/vuejs-essential
> webpack-dev-server --inline --progress --config build/webpack.dev.conf.js

sh: 1: webpack-dev-server: not found
npm ERR! file sh
npm ERR! code ELIFECYCLE
npm ERR! errno ENOENT
npm ERR! syscall spawn
npm ERR! vuejs-essential@1.0.0 dev: `webpack-dev-server --inline --progress --config build/webpack.dev.conf.js`
npm ERR! spawn ENOENT
npm ERR!
npm ERR! Failed at the vuejs-essential@1.0.0 dev script.
npm ERR! This is probably not a problem with npm. There is likely additional logging output above.

npm ERR! A complete log of this run can be found in:
npm ERR!     /home/vagrant/.npm/_logs/2019-04-10T02_11_25_868Z-debug.log
```

搜索了下，参考 [Bug with webpack-dev-server 2.10.0 - Unable to start webpack-dev-server](https://github.com/vuejs-templates/webpack/issues/1229), 执行 `yarn add -D webpack-dev-server@2.9.7 --no-bin-links` 安装后解决。

