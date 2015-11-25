---
layout: post
title: 安装Shadowsocks
category: voice
tag: shadowsocks gfw
---

## 缘起

最近由于工作的关系,很久也没有更新博客,也很久没有尝试fq了.想入手nexus 6p了,才发现,原先的vpn已经被墙了,又卖了一个,才发现原来的教程也有些问题.悲剧.

不过也发现了Shadowsocks,所以顺手也使用了下,效果不错.至少在我的若干vps上,高峰时候也是速度不错的.

## 安装server

首先,你得有一个境外可连的vps服务器,这里推荐[linode][linode],这个是我的[ref][ref].

再安装完ubuntu之后,我们需要安装`python-pip`,从而安装shadowsocks

    sudo apt-get install python-pip
    sudo pip install shadowsocks

这样,shadowsocks的程序就已经安装完毕.最后把服务启动即可.

    sudo ssserver -p 443 -k password -m rc4-md5 --user nobody -d start

**-p**是指定端口号,**-k**是密码(需要明文输入),**-m**是加密方式.这三个是需要客户端连接的凭证,一定要完全一致.加密的话,其实哪个都可以,最好选择一个比较强的,`rc4-md5`是最基本的了.

## 安装client

我现在的常用系统是win10了,所以只能安windows版的客户端.到[这里][client]下载最新版安装即可.

在配置中,填入刚才启动server的ip/端口/密码/加密方式,就可以了.其他的配置其实和其他翻墙没太大不同,就自己看看即可.

我比较不喜欢的一个功能是,不选择"system proxy"时,无法让shadowsocks不进行pac匹配.这导致我无法使用chrome里的omega来方便的添加一个临时的规则,让我只能去hack配置文件.

## 结语

这就完成了shadowsocks的搭建.很期待我的nexus 6p赶紧回来,在手机上看看外面的世界.

[linode]: https://www.linode.com
[ref]: https://www.linode.com/?r=d6576fd029602fa1e9c123591db885fa863e4ce0
[client]: https://github.com/shadowsocks/shadowsocks-windows/releases
