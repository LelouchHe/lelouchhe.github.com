---
layout: post
title: 嵌入汇编如何捕获异常
category: mine
tag: trap
---

## 缘由

最近工作上遇到一个比较少见的问题,特地记录一下,以免下次被坑.

情景是这样的,我们要动态的调用任意函数,函数的类型是不定的,有可能是普通函数,也有复杂的比如成员函数或虚函数之类.参数倒是可以统一规划为64位的整数.

一个初步的想法就是利用嵌入汇编,手动的将函数调用栈构建好,直接跳转到对应方法.这个方法比较暴力,但效率非常高,基本等同于直接的函数调用,而且不需要添加额外的抽象层.

但这就带来一个问题: 我们的函数是C++函数,是有可能抛出异常的.而[gcc文档][gcc exception]中明确说到,这样的异常会让程序`abort`的.

那如何解决呢?

## C++异常处理机制简介

假定开发环境是**x86-64 Linux gcc 4.4.7(and above)**.我们先来看下gcc是如何处理异常的.

gcc有2套处理异常的方案:

1. `setjmp/longjmp`: 利用标准库中的远程跳转函数,`try...catch`时使用`setjmp`保存环境,`throw`时用`longjmp`完成跳转,顺便延调用栈执行需要的析构操作.这个方法的初级版本比较简单,很多人在C下实现的异常处理机制,就是这么做的.一个难点是如何执行需要的析构.当然,如果在C下,就不用考虑这个了.C++下的话,大体上都是在保存环境时,保存尚未析构的对象信息和显式的栈信息,从而可以在`longjmp`时获取,并最后真正调用.这个方法的缺点是成本太高.不论我们最后是否抛出异常,`setjmp`及之后都必须保存大量的信息.这也是最开始异常处理让人觉得性能低下的一个原因所在.
2. `dwarf`: 这是目前gcc的默认实现方案.在编译时,将异常信息保存在特定的section中,包括`try`的起止范围,对应的`catch`语句之类.在程序正常运行时,通过一个简单的`jmp`跳过`catch`的代码,因此当没有异常抛出时,完全没有性能问题.当抛出异常后,会从该section中得到异常处理需要的信息,从而完成最后的栈回滚,析构调用并回到`catch`语句中.

很明显,第2种方案比第1种方案好,但由于异常信息以特定的数据结构编码到特定的section,从而导致我们无法简单的进行操作.

下面都是以第2种方法为前提的讨论.`dwarf`是gcc的默认实现,一般都推荐用这个.

## 嵌入汇编的困难

嵌入汇编之所以无法处理异常,是因为就算我们在调用潜入汇编的代码外围加上`try`,但由于嵌入汇编这层上没有编译器添加的异常结构,导致异常回滚时,到这里就无法再前进了,就只能当作没有`try`块从而`abort`了

比如,我们的代码如下:

    extern "C" int add(int a, int b) {
        throw "in add";
    }

    int inline_add(int a, int b) {
        int r = 0;
        __asm__ __volatile__ (
            "movl %1, %%edi\n\t"
            "movl %2, %%esi\n\t"
            "call add\n\t"
            "movl %%eax, %0\n\t"
            : "=r"(r)
            : "r"(a), "r"(b)
            : "%eax"
        );
        return r;
    }

就算我们用`try`包住:

    try {
        inline_add(1, 1);
    } catch (...) {
        // ...
    }

由于`inline_add`这层调用上是没有任何异常处理信息的,所以程序会`abort`.

那能不能直接内联到`try`块里呢?这个我试过,是没有用处的.查看生成的汇编代码,和不内联的基本一样,没有异常信息.

## gcc如何生成异常信息

通过内联的实验可以看出,异常信息并不是以`try`为标志来生成的.相反,是以`throw`的出现来生成的.

当我们如下编码时:

    void just_throw() {
        throw 0;
    }

    try {
        just_throw();
    } catch (...) {
        // ...
    }

gcc会正常的生成异常信息.这给我们一个提示,异常信息对于`try`来说是一个整体(可以查看生成的汇编),那我们强行给嵌入汇编的`try`添加一个会`throw`的调用不就行了么?

    try {
        inline_add(1, 1);
        just_throw();
    } catch (...) {
        // ...
    }

这个样子确实可以处理异常了,因为`just_throw`的出现,使得gcc给这个`try`块增加了异常信息.

其实细说起来,并不是这样的.比如如下:

    try {
        just_throw();
        inline_add(1, 1);
    } catch (...) {
        // ...
    }
    
我们就又无法处理了.一是因为`just_throw`直接就抛出了,根本运行不到`inline_add`,二来通过生成的汇编来看,根本就没包含到我们的汇编代码.

应该说,`just_throw`放在后面能够生效,主要是`inline_add`之后没有其他任何的操作,从`inline_add`抛出的异常,其实可以等同于从`just_throw`抛出的,所以最后都能`catch`住.上下颠倒过来,自然就没这个等价了.

## 构建异常信息

通过上面的实验,我们需要一个**能**抛出异常,但不抛出异常的函数调用,紧跟在`inline_add`之后,帮助构建异常信息,使之可以被`catch`.

最后的代码是这样的:

    
    void build_exception_frame(bool b) __attribute__((optimize("O0")));
    void build_exception_frame(bool b) {
        if (b) {
            throw 0;
        }
    }

    try {
        inline_add(1, 1);
        build_exception_frame(false);
    } catch (...) {
        // ...
    }

解释2点:

* 加了`attribute`,主要是为了避免gcc优化把`build_exception_frame`直接内联或优化没了.经过测试,**O2**及以上的优化就会有问题
* 使用参数b,而不是直接`if (false)`,也是gcc优化的问题

我们可以通过查看生成的汇编代码来确定上述的正确性.此处不赘言了

## 结语

虽然我们得到了可以运行的代码,但gcc为什么会这样,我还没有彻底搞懂.毕竟,哪有像上文描述的那么感性的编译器.所以,解析来可能要深入gcc的代码一探究竟了.

另,这个问题是我第一个[stackoverflow]上回答的问题.没想到,我也可以解决别人的困惑了,感觉真不错.

[gcc exception]: https://gcc.gnu.org/onlinedocs/libstdc++/manual/using_exceptions.html
[stackoverflow]: http://stackoverflow.com/questions/2642385/throwing-a-c-exception-after-an-inline-asm-jump/27619955#27619955
