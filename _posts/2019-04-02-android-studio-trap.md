---
layout: post
title:  "Android Studio 升级陷阱"
subtitle: "搞什么鬼"
date:   "2019-04-02"
author: "cj"
tags:
    android-studio
    android
    kotlin
    gradle
---

# Android Studio 升级陷阱

今天把 `Android Studio` 升级到了 `3.3.2`，完了提示我 `com.android.tools.build:gradle:3.3.1` 可以升级到 `com.android.tools.build:gradle:3.3.2`，我点了升级，然后就出鬼了，项目明明并没有使用 `kotlin`，却一直提示 `Gradle sync failed Could not download kotlin-stdlib-3.3.2.jar` 或 `Gradle sync failed Could not download kotlin-reflect-3.3.2.jar`。

我试了下手动下载，可以下载啊，就不到2M的文件！

重启无数次，无奈 `git diff` 看一下哪里动了，就找到了是 `com.android.tools.build:gradle:3.3.1` 这里被升级了，改回去之后一切OK。

搞什么鬼！~
