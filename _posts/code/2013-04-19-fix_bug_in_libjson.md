---
layout: post
title: 修复libjson不支持utf8的bug
category: code
tags: json hack
---

## libjson的苦恼

libjson是我使用过最快最爽的json解析库了.当然,用c++写的,使用起来肯定不如php之类的方便,但在速度上绝对更胜一筹.当年写的服务解析json耗时甚多(用的是jsoncpp),然后换了一下libjson,立马就达标了,解析时间从原先的50ms下降到不到1ms,真是厉害

不过好虽好,但有些问题非常的不爽

首先是C++版本怎么编译都有问题,google,论坛,文档,翻了无数遍,但却找不到任何原因.幸亏libjson的C接口深合我意,所以也无大碍.不过感到遗憾的是,libjson提供C接口,但本身是C++的库,如果我们要使用的话,在纯C的环境下是无法编译通过的.

其次是无法处理中文.按文档说明,处理unicode的话,需要在JSONOptions.h中定义UNICODE的一个宏,然后底层统一使用wchar_t作为字符存储.这个是个大问题,很多服务都在使用char,如果单独为这个来个转换,耗时不说,整个流程就复杂难看了(尤其没有那么多服务需要超高性能的json解析库).大名鼎鼎的libjson居然不支持utf8这种可以用char来处理的方式,还真是荒谬

## 问题的所在

问题出在json_as_string()这个函数上,本质来讲,libjson一般不同原来的字符串,只有当需要的时候,才会进行解析,而且一旦解析就会cache住,下次解析就会快.但是在json_as_string中将内部的internal转换成json_string时,处理转义字符出现了问题,参见JSONWorker.cpp中的如下代码:

    JSON_ASSERT(*(pos + 1) == JSON_TEXT('0'), JSON_TEXT("wide utf character (hihi)"));
    JSON_ASSERT(*(pos + 2) == JSON_TEXT('0'), JSON_TEXT("wide utf character (hilo)"));
    pos += 3;
    return Hex(pos);

可以很明显的看到,libjson内部将"\uxxxx"中的前2位忽略掉了,这在ASCII的情况下是正确的,但是对于非ASCII来说,就是有问题的.libjson在这个的处理上,还真是够差劲的.

不过幸好unicode的utf8编码比较简单,我准备hach一下修复这个问题,让libjson可以在更多场景下使用

## unicode的utf8编码

由于绝大部分的unicode都是4字节表示的(即"\uxxxx",当然还有其他,不过基本可以说没有),转换规则相对简单,按照下表即可:

* 0000-007F: 0xxxxxxx
* 0080-07FF: 110xxxxx 10xxxxxx
* 0800-FFFF: 1110xxxx 10xxxxxx 10xxxxxx

总的来说,首字节第一个0前的1的个数,表示这个utf8编码有几个字节,剩下的"x"就是原先unicode码的二进制表示,高位不足补零即可

详见[utf-8编码][utf8]

## 修改

代码很简单,不过有几个坑需要注意,一个是优先级,勤快点加括号就好,另一个是位操作的单位对结果有很大的影响,包括取值,扩展格式等,都要自己的考虑清楚再写

       pos++;
       int len = 1;
       if (json_likely(pos[0] > '0' || pos[1] >= '8')) {
           len = 3;
       } else if (pos[1] > 0 || pos[2] >= '8') {
           len = 2;
       }   
       json_uchar first = Hex(pos);
       pos++;
       json_uchar second = Hex(pos);
       uint16_t code = ((uint16_t)first << 8) | second;   // pos位置为最后一个字节,下一次循环自增即可
       result += (json_uchar)(0xFF << (8 - len)) | (code >> ((len - 1) * 6)); 
       while (--len > 0) {
           result += (json_uchar)(0x80 | ((code >> ((len - 1) * 6) & 0x3F)));
       }   

       return '\0';  // 丢弃的结果

这个仅仅是修改从原始串转换到UTF8的过程,还有一个逆过程没有实现,ToUTF8,不过这个转换只有在开启JSON_ESCAPE_WRITES时才有用,我们基本不会这样做(JSON_ESCAPE_WRITES会将一些特殊字符增加转义,对调试有些帮助,但对程序不大)

## 总结

第一次修改开源库,libjson效率虽然高,但是代码还是比较复杂,我通过gdb的方式才最终将bug找到并排除.不知道我是否应该提交给上游,但是目前就这样吧.

哈哈,我居然也修改了开源库,恩,想想很有趣

[utf8]: http://zh.wikipedia.org/zh-cn/UTF-8 "utf-8编码"
