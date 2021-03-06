---
layout: post
title: C语言中的生存期问题
category: code
tag: c
---

## 问题的来源

前些日子在实现自己的[JSON库][mjson],使用的是惯用的C语言.但写到最后,发现越写越像C++的风格了.虽然这并不是我的初衷,但可能这反应了一些我平时没有注意到的问题

现在看来,其中之一就是生存期的问题

## 什么是生存期

生存期,就是指一个对象从生到死的这段时间.生,意味着给运行系统给这个对象分配内存,初始化值;死,意味着系统收回了对象的内存,对象不再有意义

有GC的语言,比较不关心这个问题,使用者一般只负责对象的创建,然后就可以随意的使用,运行系统的GC保证了只有当对象无法被使用时,才会进行释放和销毁.

C++虽然没有GC,但是RAII惯用法提供了比GC效率更高,更优雅的解决问题的方法,配合标准库里的智能指针,就同某位牛人说过的那样,"再也不会有内存问题的困扰了"

反观C,缺少运行系统和编译器的辅助,试图很方便的解决对象生存死亡的问题,确实有些难度

## 难点在哪里

C是比较初级的语言了(谁说它是高级语言来着!!),代码和运行时候机器码基本就是一一对应的了(忽略编译器的优化和CPU的执行),也就是说,**除非我们写出来,否则它就不会运行**

以前不知道在哪里看到这句话时,觉得很平常,但在实践过很长时间之后,终于发现这点确实点出了C语言处理问题的难点了,生存期问题只是这个问题的一个表现而已

比如,在[skip_list库][skip_list]的实现中,需要通用类型key/value,在C语言中,只能是void *类型的指针,通过api接口调用时,指向调用者给予的内存位置.但由谁来保证这些指针指向的对象,在skip_list还在使用的时候,一直存活的呢?库的内部实现是否要针对key/value的存活与否进行额外的校验和判断呢?如何进行校验和判断呢?

其实概括来说,难点有2个方向:如果试图完全控制对象的生存期,我们就面临着对象类型的不可知;如果试图完成对象的共享,我们就面临如何共享和一个大lib的负担

## 完全控制生存期

完全控制生存期,只是将带操作的对象完全置于我们的控制范围之内,类似于实现了对象的值语义.当对象进入操作之后,首先是复制对象,然后后续的操作都在这个复制的对象之上,当整个操作结束后,再将新建对象释放掉.此时,原先对象的死活就和我们没有任何关系了

但这样做有几个问题:

* 首先,操作的情景是不是适用于值语义.很多时候,操作情景还是希望保留C中原始的引用语义的,比如还是试图通过我们提供的接口直接操作原来的对象,那么此时,这样做就无法达成了
* 其次,待操作的对象能否用于值语义.C++中比较好说,对于一个对象,直接"operator="或者"cc"就行了,如果对象没有提供类似的语义,编译器就不会让你通过的.但C中却没有如此通用的值操作手段.往往我们需要根据不同的操作对象调用不同的操作来完成这一目的
* 最后,值语义是否满足了trade off.即使对象**能**完成值的转化,并不代表就**得**完成值语义.很多时候,对象的大小和操作的频率,也决定了我们能否这样来操作

关于第二个问题,是下面几点缺乏造成的:

* 泛型的缺乏 在C中,表达一个通用对象的概念,大部分时候只能用void *.如果我们知道对象只有确切的若干可枚举的类型的话,还可以选择union(虽然我很不喜欢这个概念).如果不试图实现一个类似继承树的东西,很难知道对象的确切类型.这给我们的操作带来的负担.C++中的泛型管理是交给编译器的,编译的时候,一切信息都是了解的,能否操作,怎么操作,都会合理的安排.可以就直接编译通过了,否则,就直接debug编译信息.但C中却没有这一点,我们如果不手工的建立一套结构体系,是很难达到这点的
* 类型操作的缺乏 紧跟这上一个问题.没有确切的类型,自然我们无法进行控制;就算有了类型的支持,对象的生存管理也有问题.比如,我们试图赋值时,单纯的指针赋值就不可以了,因为这并没有很好的管理起对象来.要不然操作的对象的内容是透明的,我们一个一个的进行复制,要不然就是待操作对象提供了类似的接口,让我们来进行操作.但是对于很多类型而言,这两点都达不到,我们也就只能望洋兴叹了

## 完全共享对象

共享对象,就是不进行值语义的转换,简单的使用指针引用对象.这样的操作代价最小,但问题同样很多:

* 首先,如何判断对象生存与否.单靠指针是无法完成这点的,我们需要更厚的间接层来保留对象信息.这意味着更加复杂的中间层,以及随之而来的中间层对象的生存期问题
* 其次,如何同对象真正的控制者沟通.作为库的开发者,往往不能掌握使用到对象的控制权(当然,有时候那个人就是你自己,但难免你会觉得你的库很可能别人用).如果要做到完全共享,就涉及到了和系统其他部分的沟通问题.系统的其他模块是否也是共享模式呢?它们会释放对象资源么?释放了对象之后,你的"共享"还在么?如何判断在不在呢?

## 解决的思路

可以看到,除去系统实现本身要满足的各种规格,值语义的问题主要在于C语言的类型体系,而共享,则涉及到了整个系统的实现问题(即我们无法单独实现共享,而不告知其他人)

解决的思路就是从这两点上入手.先是判断好系统要求的规格限制和要求,然后按照这两点进行突破.下面提供一些我能想到的思路

### 值语义

C的类型体系是没有多少办法绕过去的,所以值语义的问题很难解决:

* 如果可能的话,尽量限制好待操作对象的类型.因为我们有的时候并不需要一个非常通用的库,也许针对目前的系统,特定的类型可以运行的很好.比如[mjson][mjson]中的hashmap,本来是想做成通用的库,但发现,其实key的类型只要str就好了(json的key只能是str),这样就简化了key值的维护,我们单纯的strdup就好了(当然,真正实现使用了另一种类型)
* 如果可能的话,尽量让对象透明.这样我们要处理时,可以直接操作对象了,要复制时,也有个目标.更进一步,我们甚至可以直接把对象嵌套到我们的对象内部(隐藏对象是不能做到这点的),既简化了操作,又保证了生存期不会有问题
* 如果可能的话,使用C++吧.其实到了后来,我就发现对象的实现越发的和C++一样了,但缺乏编译器的支持,只能很简陋的搭个样子而已.如果能转到C++来实现,自然很多问题就不复存在了.实在对C有要求,最后在用C封个接口就行了

可以看到,其实STL就是实现的值语义的库,操作类型的最低要求就是可复制.

有的时候我们自作聪明,将裸指针传进去来避免复制开销,最后出了bug,反过来埋怨STL真难用.真是,too young, too simple, sometimes naive

### 共享对象

共享对象涉及到的是系统问题,一旦决定共享,一定预先设计好,这个共享层面到哪个层次结束.避免底层的实现机制干扰了整个系统的设计实现

* 不使用裸指针,使用智能指针.当然,在没有标准库的支持下,我们只能亲力亲为的实现了.[mjson库][mjson]就实现了一个初级版本,并且把共享的层面控制在了共享字符串(ref_str_t类型)的接口之内
* 使用内存池技术.即在我们的接口范围内,使用内存池来分配和存储对象.这个的问题就是如何控制别人也来使用内存池,否则它们传入个普通对象,你以为在池子里,然后就core了.这个问题带来的系统涉及的更严重.我见过一个设计,因为使用内存池,程序开始前利用gcc的trick覆盖了libc的malloc/free,一开始的时候没有注意到,还在想,为什么这里里面不停的malloc,不担心性能问题么.
* 最后的建议,还是转到C++.不要使用C++的其他范式,只用STL和智能指针,生活会美好很多的

## 关于C++

上面提了很多建议,最后都是说要转向C++.这里并没有说C++有多么的好,只是目前我们遇到的这个问题,C++方面已经解决的很不错了,语言层面和编译器层面,大量的牛人已经给出了非常多的辅助,如果单是因为语言层面的内心纠结而不去使用,真的是愧对牛人们的辛劳

在我看来,C++最赞的一点就是,各个功能的正交分离.如果没有用到某个东西,它很少能成为你的负担.

像我,就愿意把C++看做是个大的功能包,里面有RAII,有STL,有智能指针,有function/bind(比函数指针更通用化),我觉得完全足够我平时的开发了.这些大部分都属于泛型一边,只有RAII涉及到最初级的面向对象,而此处也是当做工具来使用的

用C++来开发,如果对接口有洁癖(比如我),最后用C封装一下,不就两全其美了么?

[mjson]: https://github.com/LelouchHe/mjson "mjson库"
[skip_list]: https://github.com/LelouchHe/skip_list "跳表库"
