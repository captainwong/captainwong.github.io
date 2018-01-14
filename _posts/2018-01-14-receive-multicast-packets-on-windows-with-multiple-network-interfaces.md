---
layout: post
title:  "Windows多网卡环境接收组播包"
subtitle: "跨过千山万水，踏遍海角天涯--I got it!"
date:   "2018-01-14" 
author: "cj"
tags:
    multicast
    windows
    multiple-interfaces
    c++
---

> 说明：本文仅针对IPv4


局域网发现技术有很多，常用组播（或称为多播）：一台设备发送组播包，其他设备加入组播组，接收到组播包时即可知晓发送端IP，接收端回应约定数据即可让发送端也得知这些接收端的IP。


# 组播地址与端口号

IPv4的D类地址（224.0.0.0至239.255.255.255）是IPv4多播地址。D类地址的低28位构成多播组ID（group ID），整个32位地址则成为组地址（group address）。

224.0.0.1为all-hosts组，224.0.0.2是all-routers组。
224.0.0.0~224.0.0.255之间的地址（224.0.0.0/24）称为链路局部(link-local)多播地址，是为低级拓扑发现和维护协议保留。

一般应用程序使用239.0.0.0~239.255.255.255之间的地址，称作可管理地划分范围的IPv4多播空间（administratively scoped IPv4 multicast space）（RFC2365）。

本文使用239.254.43.21:45454作为组播地址和端口号
``` c++
static const auto MULTICAST_GROUP_ADDRESS = "239.255.43.21";
static const unsigned short MULTICAST_GROUP_PORT = 45454;
static const auto LOCAL_IP = "192.168.1.222";
```

# WinSock2编程须知
winsock.h与winsock2.h的一些宏定义如IP_ADD_MEMBERSHIP使用了不同的值，因此须特别注意。这破问题我是在join gruop(```setsockopt(sockfd, IPPROTO_IP, IP_ADD_MEMBERSHIP...)```)始终失败，后来检查错误码为10042 (WSAENOPROTOOPT)，搜索得知不可直接```#include <windows.h>```, 而需要
``` c++
#include <WinSock2.h>

#include <Ws2tcpip.h>

#include <stdio.h>

#pragma comment(lib, "ws2_32.lib")
```
具体可参考[INFO: Header and Library Requirement When Set/Get Socket Options at the IPPROTO_IP Level](https://support.microsoft.com/en-us/help/257460/info-header-and-library-requirement-when-set-get-socket-options-at-the)

# 发送端
无需多提，仅需创建UDP socket，将数据报发送至D类地址的某个约定好的端口即可。
``` c++
struct sockaddr_in addr = {};
addr.sin_family = AF_INET;
addr.sin_port = htons(MULTICAST_GROUP_PORT);
addr.sin_addr.s_addr = inet_addr(MULTICAST_GROUP_ADDRESS);
int addr_len = sizeof(addr);

char host[1024] = { 0 };
gethostname(host, 1024);

int msgNo = 0;
char msg[1024] = { 0 };
while (true) {
    sprintf(msg, "Groupcast Message %s No.%d", host, msgNo++);
    int ret = sendto(sockfd, msg, strlen(msg), 0, (struct sockaddr*)&addr, addr_len);
    if (ret < 0) {
        perror("sendto");
        return -1;
    } else {
        printf("Sent msg: %s\n", msg);
    }
    Sleep(1000);
}
```


# 接收端

## 单网卡环境

非常简单，网上的demo也大多针对这种情况。
创建UDP socket，绑定INADDR_ANY、约定的端口，加入组播组，接收即可。
``` c++


int ret = 0;

int sockfd = socket(AF_INET, SOCK_DGRAM, 0);
if (-1 == sockfd) {
    printf("socket error!!!\n");
    perror("socket:");
    return -1;
}

int reuse = 1;
if (setsockopt(sockfd, SOL_SOCKET, SO_REUSEADDR, (const char *)&reuse, sizeof(reuse)) < 0) {
    perror("Setting SO_REUSEADDR error");
    closesocket(sockfd);
    return -1;
}

struct sockaddr_in localaddr = { 0 };
localaddr.sin_family = AF_INET;
localaddr.sin_port = htons(MULTICAST_GROUP_PORT);
localaddr.sin_addr.s_addr = /*inet_addr(LOCAL_IP)*/ htonl(INADDR_ANY);
ret = bind(sockfd, (struct sockaddr*)&localaddr, sizeof(struct sockaddr));
if (-1 == ret) {
    printf("bind localaddr error!!!\n");
    perror("bind:");
    closesocket(sockfd);
    return -1;
}

/*设置是否支持本地回环接收*/
/*int loopBack = 1;
ret = setsockopt(sockfd, IPPROTO_IP, IP_MULTICAST_LOOP, (const char *)&loopBack, sizeof(loopBack));
if (-1 == ret) {
printf("setsockopt broadcaset error!!!\n");
perror("setsockopt:");
closesocket(sockfd);
return -1;
}*/

struct ip_mreq ipmr = { 0 };
ipmr.imr_interface.s_addr = /*inet_addr(LOCAL_IP)*/ (INADDR_ANY);
ipmr.imr_multiaddr.s_addr = inet_addr(MULTICAST_GROUP_ADDRESS);
int len = sizeof(ipmr);
ret = setsockopt(sockfd, IPPROTO_IP, IP_ADD_MEMBERSHIP, (char*)&ipmr, len);
if (-1 == ret) {
    printf("set error IP_ADD_MEMBERSHIP %d\n", WSAGetLastError());
    perror("setsockopt:");
    closesocket(sockfd);
    return -1;
}

/* now just enter a read-print loop */
char msgbuf[MSGBUFSIZE];
int nbytes = 0;
localaddr.sin_addr.s_addr = inet_addr(MULTICAST_GROUP_ADDRESS);
while (1) {
    int addrlen = sizeof(localaddr);
    if ((nbytes = recvfrom(sockfd, msgbuf, MSGBUFSIZE, 0, (struct sockaddr *) &localaddr, &addrlen)) < 0) {
        perror("recvfrom");
        return -1;
    }
    msgbuf[nbytes] = 0;
    puts(msgbuf);
}

```
## 多网卡环境

有些不同，加入组播组时若依然使用INADDR_ANY，则内核默认使用网络设备列表的第一个设备，有可能并不是该局域网。因此，需要将INADDR_ANY替换为LOCAL_IP，即本设备与其他互相发现的设备所在局域网的网卡IP。
``` c++
ipmr.imr_interface.s_addr = inet_addr(LOCAL_IP);
```









# Reference
* [局域网发现之UDP组播 - CSDN博客](http://blog.csdn.net/lixin88/article/details/55209630)
* [局域网发现设备代码实现：udp组播 - CSDN博客](http://blog.csdn.net/lixin88/article/details/56013014)
* [Multicast Example Programs](http://ntrg.cs.tcd.ie/undergrad/4ba2/multicast/antony/example.html)
* [sockets - UDP multicast from specific network card - Stack Overflow](https://stackoverflow.com/questions/4054238/udp-multicast-from-specific-network-card)
* [c++ - Qt: joinMulticastGroup for all interface - Stack Overflow](https://stackoverflow.com/questions/19218994/qt-joinmulticastgroup-for-all-interface)
* [sockets - Qt QUdpSocket fails to receive multicast udp packets on a specific interface - Stack Overflow](https://stackoverflow.com/questions/46303447/qt-qudpsocket-fails-to-receive-multicast-udp-packets-on-a-specific-interface)
* [[QTBUG-27641] Multicast joining fails on multiple interfaces - Qt Bug Tracker](https://bugreports.qt.io/browse/QTBUG-27641)
* [Multicast/Broadcast questions | Qt Forum](https://forum.qt.io/topic/27579/multicast-broadcast-questions/6)
* [How to fix the global broadcast address (255.255.255.255) behavior on Windows?](https://social.technet.microsoft.com/Forums/windows/en-US/72e7387a-9f2c-4bf4-a004-c89ddde1c8aa/how-to-fix-the-global-broadcast-address-255255255255-behavior-on-windows?forum=w7itpronetworking)
* [(Solved) UDP broadcast on multiple ethernet interfaces/adapters - CodeProject](https://www.codeproject.com/Questions/524739/UDPplusbroadcastplusonplusmultipleplusethernetplus)
* [qtbase/qnativesocketengine_win.cpp at 5.10 · qt/qtbase](https://github.com/qt/qtbase/blob/5.10/src/network/socket/qnativesocketengine_win.cpp)
* [DingHe/unpv13e: UNIX网络编程 卷1：套接字联网API（第3版）源代码](https://github.com/DingHe/unpv13e)
* [INFO: Header and Library Requirement When Set/Get Socket Options at the IPPROTO_IP Level](https://support.microsoft.com/en-us/help/257460/info-header-and-library-requirement-when-set-get-socket-options-at-the)