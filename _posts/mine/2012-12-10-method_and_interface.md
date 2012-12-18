---
layout: post
title: Go中method和interface的区别
category: mine
tag: Go
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

interface的重点是**隐式**,但这就带来一个问题,就是当我们真正需要实现或者继承什么东西的时候,除了编译器和文档之外,我们无法有直接的认识,即我们也不知道我们定义的method是不是就是interface.

当然,这不是大问题,但有时候会带来一些小困扰,比如在看Go的标准库的时候,一些东西比较繁杂,容易冲昏头脑.

## methode和interface的区别

method只是Go里函数的一种,带有接收(reveiver)的特殊函数而已,可以用来实现interface,也可以不实现,这个看使用的情况.而interface是Go真真正正的类型,我们可以定义interface的变量,可以拿它传参数,做返回值

指针和值在method上没有区别,唯一的不同是值无法改变(因为传递的是值的副本),指针可以改变所指对象(但指针本身还是副本不能改变).但在interface上就不同了.如果我们要对interface赋值(直接赋值,或者作为参数,返回值等赋值),必须保证该类型实现了接口.比如还是上面的例子,我把代码拿下来:

    type Sample struct {}

    func (s Sample) DoA() {}

    type Sampler interface {
        DoA()
    }

    var s Sample
    var sp *Sample
    var si Sampler = s   // valid
    var sip Sapmler = sp // invalid

因为指针类型没有实现Sampler的接口,所以就不能给对应的Sampler变量赋值,具体造成这样的原因,还在研究中,不过这个确实是要记住的

其实接口就是规范,规范应该严格遵守,而指针和值在method上的混用,应该是属于语法上的便利而已,因为在使用时,我们还是得分清指针和值的情况.一个在编译时,一个在编码时,时机不同尔.

## type switch

使用一种特殊的类型断言,我们可以在限定情况下判断变量的真正类型(当然,reflect也可以,这是后话)

类型断言的形式是v.(Type),v必须是interface,通过这个断言,我们就能获得实现该interface的真正类型的变量,这个断言还返回第二个值,一个bool表示转换是否成功(如果不获取这个值,转换失败的时候就会panic).不过我们还可以利用一个特殊的关键字,获取其类型,即类型断言,如下:

    switch t := v.(type) {
    case int:
        fmt.Println("type is int")
    // ...
    }

关键字是type,而且这种形式的断言只能在switch中使用,而且,更重要的一条是,必须确保case中的类型确实实现了v对应的接口,否则就会panic(也就是说,如果我们要获得任意对象的类型,还是用reflect吧)

## 吐槽

其实Go的语法很符合我心目中语言的,我喜欢C++,不过不太喜欢C++中的一些奇怪的点,比如struct居然和class一样,而不是简单的POD,比如没有显式的interface关键字(当然,这个字比C++要年轻吧应该),比如可控制的GC(比如一个开关,可开可关的那种),比如匿名函数(C++0x有了)等等.这样看来,Go就是一个理想的替代品,虽然我还是不满没有了真正的指针,也没有了GC的开关,但有了内建好的channel,这个东西我在开发的时候自己就造了无数的轮子,但没有一个如此通用和方便

但是,但是,Go的标准库相当不合我口味了,唉,只能慢慢看下去了.

ps: 我挺喜欢exported符号是大写首字母的,因为为了不大写字母(又难看,又难打),我只好把exported的东西控制的越少越好,等于本来应该我自己深思熟虑的东西,被编译器强行又加上了要求,恩,这个想法不错,编译器是不会抱怨工作太多的嘛.











