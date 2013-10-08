---
layout: post
title: strtoul和stdlib
category: code
tag: tip
---

## 数值转换的问题

今天同事拿了一段代码來问我,如下:

    #include <stdio.h>
    
    int main()
    {
        char str[] = "209772416700";
        unsigned long uid = strtoul(str, NULL, 10);
        printf("%lu\t%u\n",uid, sizeof(uid));
        return 0;
    }

结果并不是如预想的那样,相反的,输出了'18446744073028570812'的一个大数字.但是看uid的size,确实也是64bit的,而strtoul也确实是转ul类型的

## 少打了一行

查看大数的二进制,突然觉得貌似了然了:大数完全是正确结果的32bit符号扩展结果.这就表明strtoul内部使用了signed int來转换数值.但这个应该不可能吧?(c库函数有bug是肯定不可能的)

仔细查看一番,发现文件并没有包含strtoul的头文件stdlib.h,但程序还可以正常编译,看来又是##自带函数##惹得祸.当我加上这个头文件后,一切正常了.

以前也有类似的经历,gcc自带的一些函数实现,可能由于某些原因幷不完善,所以出现了上面的问题.

## 多看一眼

以后遇到类似莫名其妙的问题,一定要把-Wall打开,任何时候的警告都要全,都要看,都不能偷懒.
