---
layout: post
title: 如何升级gcc
category: mine
tag: gcc
---

## 缘起

最近在努力学习UnitTest,使用的是[gmock][gmock]框架,参考的书籍是[Modern C++ Programming with Test-Driven Development][cpp book].

其中涉及到一些C++11的语法.自然,我也可以修改下代码来兼容,但考虑到C++11迟早是要碰到的,所以干脆这次一并的弄好就好了

## 版本

我现在使用的OS是Linux Mint 13,gcc的版本是4.6,比较早了,只支持-std=c++0x

当前的gcc最新版本是4.8(其实貌似4.7以上都可以了)

## 简单的升级方法

我们自然可以下载代码和依赖库,然后手动编译升级.但很多人已经帮忙做了这些事情,我们可以简单的拿来使用一下,此处使用的方法就是使用ppa更新

### 添加ppa

    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update

可以通过[这个网址][ppa]来查看当前支持的各种工具,目前只关心里面的gcc,有4.8的版本

### 安装新版gcc

    sudo apt-get install gcc-4.8
    sudo apt-get install g++-4.8

这样,系统内部就有2份gcc了,可以到*/usr/bin/gcc*来参观下

### 配置系统gcc

此处,使用*update-alternatives*,来统一更新gcc和g++

    sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.6 60 --slave /usr/bin/g++ g++ /usr/bin/g++-4.6
    sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.8 80 --slave /usr/bin/g++ g++ /usr/bin/g++-4.8
    sudo update-alternatives --config gcc

具体的使用方法可以参考man,简而言之,前两条命令就是给不同版本的gcc赋了一个优先级(4.6是60,4.8是80),并且通过*--slave*将对应版本的g++和gcc绑定在一起了.理论上是优先级高的会被系统选择,但为了保险起见,第三条命令就是来手动配置系统的gcc,此处按照提示,选择4.8即可

### 完成

这样就升级完成了.我们后续也是可以通过上面的第三条命令,重新进行配置的

## 总结

源码编译肯定是个好方法,但做事情要抓住重点,现在我们并不是要学习编译gcc,这个只是工具而已,所以选择ppa升级是最好的了

以前我总是犯这样的毛病,以后要切记切记

[gmock]: https://code.google.com/p/googlemock/ "gmock"
[cpp book]: http://pragprog.com/book/lotdd/modern-c-programming-with-test-driven-development
[ppa]: https://launchpad.net/~ubuntu-toolchain-r/+archive/test
