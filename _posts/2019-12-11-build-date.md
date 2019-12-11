---
layout: post
title:  "编译Howard Hinnant的date库小记"
subtitle: "膜拜大神"
date:   "2019-12-11"
author: "cj"
tags:
    c++
    chrono
    date
    c++20
---

# 编译 `Howard Hinnant` 的 `date` 库小记

项目地址 [https://github.com/HowardHinnant/date](https://github.com/HowardHinnant/date)

## vcpkg

在 `PowerShell` 中执行
1. `git clone https://github.com/microsoft/vcpkg.git`
2. `cd vcpkg && .\bootstrap-vcpkg.bat`
3. `.\vcpkg install curl`

## date

`git clone https://github.com/HowardHinnant/date`

`cd date && mkdir build && cd build`

打开 `x86 Native Tools Command Prompt for VS 2019`，执行（前提已安装CMake）：

```
cmake .. -DCMAKE_TOOLCHAIN_FILE=G:\dev_libs\vcpkg\scripts\buildsystems\vcpkg.cmake -DENABLE_DATE_TESTING=1 -G "Visual Studio 16 2019" -A Win32 -DCMAKE_CXX_FLAGS="/MP"
```
