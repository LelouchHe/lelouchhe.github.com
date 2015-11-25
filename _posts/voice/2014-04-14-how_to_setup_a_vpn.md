---
layout: post
title: 如何设置vpn
category: voice
tag: vpn gfw
---

## 缘起

最近和同学合伙买了了vps,尝试了下在vps上搭建可以翻墙的vpn服务.虽然搭建过程有惊无险,最后顺利搞定,但是还是有很多坑的地方需要记住

## 添加vpn用户

我们肯定不希望因为vpn的问题,把自己的主用户名/密码暴露出去,所以要新建一个专门用来连接vpn的用户

    # root或sudo
    useradd vpn

然后修改其密码,并在sshd_config中,限制其通过ssh登录(只是用来做vpn的,不可以登录机器)

    # /etc/ssh/sshd_config中修改
    DenyUsers vpn

当然,如果用户本身数量较少的话,建议还是使用AllowUsers,这样的限制更强一些

## 安装服务程序

我们此处搭建的是pptp服务(各种vpn协议服务的区别,见[这里][vpn]),操作系统是"Ubuntu 13.04 64bit",可能其他操作系统有些许差别

需要的后台程序有pptpd和iptables,统统使用apt-get来安装

    # root或sudo
    apt-get install pptpd iptables

## 配置pptp

需要配置三个东西

### ip限制

直接修改pptpd.conf即可

    # /etc/pptpd.conf
    # 去掉localip和remoteip的注释即可
    localip 192.168.0.1
    remoteip 192.168.0.234-238,192.168.0.245

### 用户限制

添加可以使用该vpn服务的用户

    # /etc/ppp/chap-secrets
    # 用户名 服务名(pptpd) 密码 ip范围('*'表示全部)
    vpn pptpd $passwd *

### dns修改

为了避免将来可能的dns解析故障,最好此处把dns服务器也修改了.当然,这个不修改,也是可以的

    # /etc/ppp/options
    # 推荐Google的DNS服务器
    ms-dns 8.8.8.8
    ms-dns 8.8.4.4

### 重启服务

    /etc/init.d/pptpd restart

这样,pptpd就在vps上正式跑起来了

## 配置网络转发

只有pptpd还是不行的,我们需要允许vps可以转发数据才行,否则vps无法作为代理机器,帮助我们翻墙

### 开启系统的ip转发

    # /etc/sysctl.conf
    # 去掉ip_forward=1的注释即可
    net.ipv4.ip_forward=1
    # 还有ipv6,自己酌情

### 配置iptables

在上一步的基础上,通过iptables做进一步的权限控制

    # root或sudo
    iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -o venet0 -j MASQUERADE

命令大体的意思是针对nat协议,开启转发,来源于192.168.0.0/24网段的请求,都发送至venet0处

当然,我这里的出口是venet0,需要根据自己的vps配置来设置出口端口名称

如果不知道的话,先"ifconfig"或"route -n"看一下自己的路由表,找到对应名称

### 保存iptables设置

iptables是一次性的,机器重启之后就没有了,所以需要配置开机启动的东西.此处,我们把它配置为网络接口启动时才设置

先保存当前配置,需要root权限

    iptables-save > /etc/iptables.pptp

然后在网络接口启动的目录下,新建执行脚本

    # /etc/network/if-up.d/ 下,新建iptables脚本
    #!/bin/bash
    iptables-restore < /etc/iptables.pptp

最后给脚本加上可执行权限即可

当然,这样变得复杂了,但是确保了只有在网络接口正常启动时,才会进行对应的配置.如果没有这种要求的话,直接把原始命令写死在"/etc/rc.local"也是可以的

## 一些后话

配置的过程肯定没有这里写的如此一帆风顺,有几个坑需要注意:

* vps是否开启了多pptp的支持 可以通过查看"/etc/ppp"是否存在来判断,如果没有的话,铁定是没戏的.像我的vps,一开始没有开启(我也不知道啊),弄了很久都不行,最后从控制面板找到了ppp支持的选项,才解决了(如果你的没有的话,可能需要联系客服)
* 网络端口名称 一开始pptp正常连接,但死活无法连通数据.最后发现时iptables写错了网络端口,教条主义的写成了"etho0"和"wlan0",而不是vps提供的那个.所以一开始需要先确定下
* 调试日志 vpn无法正常运行的时候,查看日志是必须的,主要的日志是"/var/log/syslog",还有一个pptpd的日志,但是目前还没有发现.总之,日志很重要,便于发现问题

总的来说,感觉还是不错,终于不用依赖于switchy/chrome/gae来进行有限的翻墙了

[vpn]: http://igfw.net/archives/5156 "各种vpn协议的比较"
