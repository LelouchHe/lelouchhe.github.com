---
layout: post
title: memcached设置servers的坑
category: mine
---

## 缘由

今天准备给服务添加memcache服务来提高性能,抵御可能的高峰流量,并协调一部分分布式服务的同步问题,结果悲催的整了一个多小时,栽倒了一个小小的api上

## 经过

这次使用的是libmemcached提供的C++接口,非常简明畅快,添加servers的时候使用的是静态配置好的服务器列表,一开始是这样的"123.123.123.123:12345;12.12.12.12:12345",结果可以想象的到,memcache不可用,不同步的问题还是很夸张,而且性能还下降了.

跟踪了几条查询,发现所有的本机cache貌似可用,但后面那个的cache基本都没结果.自然,就想到了格式问题.去libmemcached的官网查了查,**居然**没有接口说明!!我使用的是`memcache::Memcache::setServers`成员函数,看代码调用了`memcached_servers_parse`,然后在官网上这个函数的参数格式竟然木有写!!

好吧,几经周折,终于在`MySQL`的网站上找到了memcached介绍性质的[文章][libmemcached],看到了下面的话:

    memcached_server_st *memcached_servers_parse (char *server_strings);
    Parses a string containing a list of servers, where individual servers are separated by a comma, space, or both, and where individual servers are of the form server[:port]. The return value is a server list structure.

感谢`MySQL`,然后我就把格式改成了用逗号分隔的形式.

接着重启服务,看到还是不同步.

很郁闷的查了查资料也没有结果,就开始比对配置,发现服务器的配置是不一样的,我想当然的把本机写在了最前头,后面跟着其他的几台机器,我就猜想估计memcache并没有像我想象的那样,根据ip来hash不同的cache,而是简单的根据添加服务器的顺序来hash,然后我就把配置改成一致,最后自然是成功了.

## 总结

首先是很惊讶,libmemcached居然不是根据ip来hash的,这个让人很匪夷所思,这样岂不是每个分布式的节点都必须按照唯一的顺序来添加cache服务器么?当然,也许这是正确的途径,但是还是会带来一些麻烦

其次,文档的重要性不言而喻,手足无措的时候手册绝对可以帮上大忙.最近在学Lua和Go,发现Manual绝对是最好的教材(顺便鄙视下万恶的GFW,连[Golang](http://golang.org)都墙)

最后,规范的重要性,我习惯了内部惯用的表达法,以";"隔开机器,以":"隔开ip和端口,但并不是所有人都这样想的,而此时我们要遵循对应的规则.这也是**标准**的重要性了.

不过,我还是弱爆了,唉,伤心

[libmemcached]: http://dev.mysql.com/doc/refman/5.0/en/ha-memcached-interfaces-libmemcached.html "mysql"
