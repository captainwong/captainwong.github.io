---
layout: post
title:  "ssh 端口转发"
subtitle: "迁移自 bandwagon vps"
date:   "2015-07-15" 
author: "cj"
tags:
    port-forward
    ssh
---

公司项目，服务器-客户端模式，当服务器与客户端跨运营商运行时（服务器使用移动专线，客户端使用电信宽带），容易出现频繁断线、断线后不能重连的问题。而我所在的分公司环境为，一个电信8M宽带，一个电信2M独立IP，一般情况下很难重现rst/retransmition等现象。

我想到自己有个VPS，是否可以利用一下呢？vps上有个shadowsocks服务器程序，平时用来翻墙，他就是用来转发的嘛，那么能不能自己搞一下，使用端口转发的方式，将客户端的链接通过vps转发到我的服务器上呢？

然后搜索端口转发，果然，已经有成熟的实现：ssh端口转发。

基本原理就是两个机子分别运行ssh客户端与sshd服务器，2者建立加密连接后转换为普通tcp连接给指定的端口，该端口为自己的服务端程序用来监听的端口。

ssh命令中 -L / -R两个选项就是用来干这个事的，不过一个是本地转发，一个是远程转发。看过一些资料后发现二者的区别为：本地转发时，本地为ssh客户端程序，sshd服务程序与服务端程序运行在远程主机，其他地方的客户端连接本地主机的端口A时，ssh客户端将该连接加密后转发给远程sshd服务，sshd服务程序解密后交给服务端程序。

而远程转发恰好相反：本地运行的时ssh客户端与服务器程序，远程主机运行的是sshd服务程序，并在监听端口A上的连接加密后转发给本地ssh客户端，本地ssh客户端解密后转交给服务器程序。

权衡后我选择了远程转发，这样可以免去先ssh登陆到vps，再在vps上执行本地转发的麻烦。

只要在我的win7上安装ssh客户端，执行远程转发命令即可。
{% highlight shell %}
ssh -R 12345:36.40.73.92:12345 root@wangyapeng.net -p xxxxx
{% endhighlight %}

36.40.73.92为电信宽带分配的IP，wangyapeng.net为我的vps。安全起见，我的vps监听端口号不是默认的22，而是xxxxx：）

上述命令的作用为：wangyapeng.net监听12345端口，接受到连接后加密并转发给36.40.73.92的12345端口。由于我的开发机在路由器下，还需要在路由器上设置端口映射，将12345映射到我开发机的12345端口上。

linux manpage上关于ssh -R的说明为：

>−R
>
>\[bind_address:\]port:host:hostport
Specifies that the given port on the remote \(server\) host is to be forwarded to the given host and port on the local side. This works by allocating a socket to listen to port on the remote side, and when\- ever a connection is made to this port, the connection is forwarded over the secure channel, and a connection is made to host port hostport from the local machine.
>
>Port forwardings can also be specified in the configuration file. Privileged ports can be forwarded only when logging in as root on the remote machine. IPv6 addresses can be specified by enclosing the address in square brackets.
>
>By default, the listening socket on the server will be bound to the loopback interface only. This may be overridden by specifying a bind_address. An empty bind_address, or the address ‘∗’, indicates that the remote socket should listen on all interfaces. Specifying a remote bind_address will only succeed if the server’s GatewayPorts option is enabled \(see sshd_config\(5\)\).
>
>If the port argument is ‘0’, the listen port will be dynamically allocated on the server and reported to the client at run time. When used together with -O forward the allocated port will be printed to the standard output.

sh 的-g选项表示绑定监听地址到ADDR_ANY，但我试过之后没有效果。在vps上运行

```bash
telnet localhost 12345
```

本地服务器程序确实收到了连接，说明ssh端口转发已经生效，只不过sshd只接受本机发起的连接而已。

再来看ssh -R的说明，需要sshd_config的选项GatewayPorts为yes才可以，相当于-g。修改之后，再在本机上执行

```bash
telnet wangyapeng.net 12345
```

成功！

参考资料：

[实战ssh端口转发](https://www.ibm.com/developerworks/cn/linux/l-cn-sshforward/)

[SSH 隧道转发实战](http://chenweiguang.blogspot.com/2009/03/ssh.html)

[SSH原理与运用（二）：远程操作与端口转发](http://www.ruanyifeng.com/blog/2011/12/ssh_port_forwarding.html)

[SSH 远程端口转发](http://lvii.github.io/system/2013/10/08/ssh-remote-port-forwarding/)

