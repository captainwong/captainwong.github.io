---
layout: post
title:  "博客环境搭建过程"
subtitle: "迁移自 bandwagon vps"
date:   "2015-05-17" 
author: "cj"
tags:
    bandwagon
    blog
    website
---

[搬瓦工](https://bandwagonhost.com/aff.php?aff=3224)的虚拟主机，开始买的micro64版，便宜，一年才24块，但内存才128M，不够用啊。后来升级到micro96，最终升级到micro128，够用了。
![img](/img/wangyapeng.net-micro128.snapshot.jpg)
其实买这个VPS的初衷不过是搭建shadowsocks服务器用来翻墙，24块一年可比[红杏](http://honx.in/_VTdyiokWGkmlTPDL)（一个月10块钱）便宜多了。插一句，更比[NydusVPN](http://www.share-nydus.com/s/HXoggott)便宜，那货一个月要20块——五一那天做活动买一年送一年，我一时鬼迷心窍冲动就买了。。。

言归正传，有了主机，再买域名。在godaddy上搜索wangyapeng，.com的需要1W多软妹币啊！简直我了个大擦。。捡便宜买了个.net域名花了60多，不错。然后在dnspod上添加记录，搞定！

固定链接设置：
{% highlight nginx %}
location / {
#root /usr/share/nginx/html;
root /var/www/html;
index index.html index.htm index.php;

if (-f $request_filename/index.html){
rewrite (.*) $1/index.html break;
}
if (-f $request_filename/index.php){
rewrite (.*) $1/index.php;
}
if (!-f $request_filename){
rewrite (.*) /index.php;
}
}
{% endhighlight %}

