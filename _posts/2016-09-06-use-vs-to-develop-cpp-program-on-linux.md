---
layout: post
title:  "使用Visual Studio开发Linux程序"
subtitle: "福音啊！"
date:   "2016-09-06" 
author: "cj"
tags:
    c++
    vs
    linux
    gdb
    debug
    知乎
    微软
---

前两天逛知乎发现一个问题：[微软有必要开发 Visual Studio for Linux 吗，如有必要，那么会产生什么影响？](https://www.zhihu.com/question/48553142)，[孙明琦](https://www.zhihu.com/people/sun-ming-qi-34)说：

> 只要你的linux装了ssh-server，gdb-server，g++，然后把usr/include下的文件拷贝到windows下，那么就可以在windows下的vs上编写linux程序，vs替你ssh到linux上用g++编译，用gdb-server debug，gdb各项功能完美映射到vs的debug窗口上，无比舒爽
>
> 作者：孙明琦
>
> 链接：[https://www.zhihu.com/question/48553142/answer/120663820](https://www.zhihu.com/question/48553142/answer/120663820)
>
> 来源：知乎
>
> 著作权归作者所有，转载请联系作者获得授权。

这是什么感觉？非一般的感觉啊！赶紧上手试试。

1. 按照[微软官方文档](https://blogs.msdn.microsoft.com/vcblog/2016/03/30/visual-c-for-linux-development/)
    先在Linux上安装几个必须的库。
    ```bash
    sudo apt-get install openssh-server g++ gdb gdbserver
    ```

2. 下载[Visual C++ for Linux Development extension](https://blogs.msdn.microsoft.com/vcblog/2016/03/30/visual-c-for-linux-development/)
    安装前提：Visual Studio版本为Update3。下载后按照提示安装之。

3. 打开Visual Studio，新建工程
    模板为：`Templates > Visual C++ > Cross Platform > Linux > Empty Project (Linux)`。也可以调试UI程序的，但我只需要console程序就够了。
    ![img](https://wangyapeng-me-1251421422.cos.ap-shanghai.myqcloud.com/new-linux-blank-project.png)

4. 写个HelloWorld程序，编译一下，会弹出连接远程Linux主机的提示，填写它！
    ![img](https://wangyapeng-me-1251421422.cos.ap-shanghai.myqcloud.com/HelloWorlCpp.png)
    ![img](https://wangyapeng-me-1251421422.cos.ap-shanghai.myqcloud.com/Connect-to-Linux-first-connection.png)

5. 点Remote GDB Debugger
    开始远程调试吧，不要太爽！Output窗口完美映射GDB，还可以在右上角搜索Linux Console，将程序的输出也显示出来，简直 perfect ！
    ![img](https://wangyapeng-me-1251421422.cos.ap-shanghai.myqcloud.com/Output-and-Linux-Console.png)

6. 总结
    微软大法好，Coders Use VS 保平安！哈哈，还差一步，插件没有自动引用Linux的头文件，需要手动从Linux主机的/usr/include拷贝到本机并添加到vs的c++ path中。Enjoy it！

# 参考资料

* [微软有必要开发 Visual Studio for Linux 吗，如有必要，那么会产生什么影响？](https://www.zhihu.com/question/48553142)
* [微软官方文档](https://blogs.msdn.microsoft.com/vcblog/2016/03/30/visual-c-for-linux-development/)
* [Visual C++ for Linux Development extension](https://blogs.msdn.microsoft.com/vcblog/2016/03/30/visual-c-for-linux-development/)
