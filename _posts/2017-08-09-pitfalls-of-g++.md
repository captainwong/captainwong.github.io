---
layout: post
title:  "Makefile要点"
subtitle: "那些年我们一起踩过的Makefile大坑"
date:   "2017-08-09" 
author: "cj"
tags:
    makefile
    c++
    g++
---

1. 不能用 `~` 表示 `home` 路径，可以用 `$HOME` 代替

2. `$(CXX) $(LDFLAGS) -o $(EXEC) $^ $(INCLUDEDIR) $(LIBDIR)` 这种写法没问题，但是把`$(INCLUDEDIR) $(LIBDIR)`放到 `-o` 前面就会出现 link 失败，wtf

3. 检测文件是否存在并执行命令
    ```makefile
    install:
        mkdir -p $(PROJDIR)/bin; \
        cp $(PROJECT_NAME) $(PROJDIR)/bin/; \
        ./build_symbols.sh $(PROJECT_NAME); 
    ifeq ("$(wildcard $(PROJDIR)/bin/$(PROJECT_NAME).config)","")
        cp $(PROJECT_NAME).config.example $(PROJDIR)/bin/$(PROJECT_NAME).config 
        @echo "You should edit $(PROJDIR)/$(PROJECT_NAME).config before running!"
    endif
    ```

    检测配置文件是否存在，不存在就用模板创建一份并提示用户修改。

    - 使用 `ifeq ... endif`，不能使用 `[ ! -e /path/to/file ] && some-command`，当文件存在时会导致 `make` 失败
    - `ifeq` 和 `endif` 前面不能有空白

4. 可以在 `echo some-message` 前面加上 `@` : `@echo some-message`，执行时不会重复输出
