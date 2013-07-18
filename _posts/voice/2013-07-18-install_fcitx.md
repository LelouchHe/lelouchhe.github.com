---
layout: post
title: 安装fcitx输入法
category: voice
tag: input
---

## 不能忍了

在linux下存活了三年左右吧,一直使用的是ibus-sunpinyin.

好不好呢?摔..显然,linux下没有一个完美的输入法.

不过好在码农也不用中文,所以暂时就忍了

最近频繁的更新笔记和博文,中文输入明显增多了,结果就无法忍了

安装搜狗的linux变种--fcitx-sougou

## 安装过程

零碎的步骤参见[本文][install]

卸载ibus

    sudo apt-get remove ibus

添加fcitx源.ubuntu自带源有点问题,最好使用nightly build版本的

    sudo add-apt-repository ppa:fcitx-team/nightly
    sudo apt-get update

然后就是安装了,可以全装,或者选择的

    sudo apt-get install fcitx fcitx-config-gtk
    sudo apt-get install fcitx-sunpinyin
    sudo apt-get install fcitx-googlepinyin
    sudo apt-get install fcitx-module-cloudpinyin  fcitx-sogoupinyin
    sudo apt-get install fcitx-table-all

设置默认输入法

    im-switch -s fcitx -z default

## 坑

一定要把ibus全部卸载才行,要不然系统死活默认ibus,不管是从Language Support那里,或者输入法管理器那里都没有用处

一定要从软件管理中心把ibus全部搞定才行.然后重启就可以了

## 使用感觉

生活质量确实提高了,不过还需要时间检验啊

[install]: http://blog.ubuntusoft.com/ubuntu12-10-sougou-pinyin.html#.UegFoFQW1KY
