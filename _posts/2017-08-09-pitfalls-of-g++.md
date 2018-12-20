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

1. 不能用~表示home路径，可以用`$HOME`代替

2. `$(CXX) $(LDFLAGS) -o $(EXEC) $^ $(INCLUDEDIR) $(LIBDIR)` 这种写法没问题，但是把`$(INCLUDEDIR) $(LIBDIR)`放到`-o`前面就会出现link失败，wtf
