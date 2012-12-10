---
layout: post
title: Go中method和interface的区别
category: mine
---

Go是一个非常不错的新语言,除了语法上面可能怪了点,结合了C的简练和高级的语言级原语,让编程,尤其是在网络环境多并发下的编程变得更加容易

这里谈一谈两个很容易混淆的东东,method和interface

## 什么是method

Go使用了很类似C的一种用法,不知道各位是怎样封装C模块的,本人是这样做的

    struct sample_t;

    struct sample_t *s_ini();
    void s_fini(struct sample_t *);

    int s_do_a(struct sample_t *);
    int s_do_b(struct sample_t *);

如果你也是这样做的,那么你就能很快的适应Go的method语法.其实就本质而言,我们上面的这种C的写法,亦或者C++/Java内部加以成员函数的方式,都是要将**数据**和**方法**联系起来(第一个参数不就相当于this指针么?)

Go的method使用的时候更像C++,但生命和定义的时候却更加的C了,比如下面这样:

    type Sample struct {}

    func (s Sample) DoA() {}
    func (s *Sample) DoB() {}

然后我们使用的时候这样来使用

    var s Sample
    s.DoA()
    var sp *Sample
    sp.DoB()

    s.DoB()

在Go中,s和sp之后的那个"."很类似C++中的成员的操作符,但这里它的名称是**选择符**,而且这个**选择符**是不区分指针还是值的.从上面的例子也能看到,我们定义了`*Sample`指针的方法,但是我们仍然可以使用在值上面,这也是我们定义的时候没有区分接收名称的原因(它们都是s)

但我们仍然要注意区分指针和值的调用,因为指针调用可以改变指针指向的对象,但值却不可以.当我们需要method调用的时候,一定要注意这样的情况,避免出现迷糊的bug

## method的好处

表面看来,这只是一种不同的表达方式,使用的时候除了不区分指针与值外,和别的语言差不多.但实际上,这是一种不同的思维方式.尤其和下面的interface结合起来看的时候

## 什么是interface

interface是一种类型,用来定义一种规格或者规范的类型,约束满足条件的最小限制,完成泛化的操作.如果接触过Java,那么一定了解这个关键字的作用,实际使用中,也和Java类似,不过也有不同的地方

    type Sampler interface {
        DoA()
    }

使用`type`定义一个新的interface,这就定义了一种新的约束,我们可以随意的使用`Sampler`类型的变量,只要这个变量的真实类型实现了我们需要`DoA`就可以,比如上面提到过的`Sample`

    func Do(si Sampler) {
        si.DoA()
    }

    var s Sample
    Do(s)

!!我们根本不需要**继承**`Sampler`!!

对,我们不需要继承派生等复杂的概念,我们只要记得要给我们的类型实现这样一个method,就完成了对interface的使用,剩下的时候交给编译器来处理,如果我们定义错了method,或者压根就没有这样的接口,没有关系,编译器会在编译期就告诉我们的,而Go的编译是**飞快**的

同样的,我们也看到了method和interface的共通.interface提供一种约定,method实现这种约定,但不需要与这个interface绑定起来.其实如果没有`Sampler`的interface,我们的`DoA`还是可以像正常method一样使用.

## interface的好处

Go的interface给了我们一个更扁平的结构,我们不需要为了实现一个接口从一棵派生的大树上进行继承,我们仅仅需要实现一个method即可,剩下的那些我们不需要的统统可以不要,这样,我们就可以在使用接口的地方来调用我们的类型了.

1. 
















