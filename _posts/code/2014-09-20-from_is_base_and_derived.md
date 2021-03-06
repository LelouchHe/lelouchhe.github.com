---
layout: post
title: 从is_base_and_derived出发
category: code
tag: c++
---

## 缘由

[is_base_and_derived][is_base_and_derived]是[boost][boost]提供的一个用于判断继承关系的工具类,利用is_base_and_derived<B, D>::value的真假,即可判断B/D是否有继承关系

如果不看代码,打死我也是想不出要怎样判断的;看了代码之后,也觉得很模糊.

虽然人家boost体贴的写了很长的注释,但看的云里雾里的,所以查了些资料,试图搞懂它

## 代码

简化一下实现,大体上如下

    template<typename B, typename D>
    struct is_base_and_derived {
        static char check(const B *, int); // B版本

        template<typename T>
        static int check(const D *, T);    // D版本

        struct host {
            operator const B *() const;
            operator const D *();
        };

        enum {
            value = (sizeof (int) == sizeof (check(host(), 0)))
        };
    };

## 解释

从最后的结果来看,如果`check(host(), 0)`调用的是B的版本,就返回false,调用的是D的版本,则返回true.

host()返回一个临时的host对象,直接作为check的参数.check的第一个参数类型是`const B *`或`const D *`,而host提供了host到这些类型的自定义转换(那两个operator).那么此时,在不同关系下,编译器究竟是如何判断调用哪个函数的呢?

### class D : class B

如果此时要调用B版本,我们要经过的类型转换(转换1)大致是:

* `host -> const host -> const B *`: 转换成功

此时D是继承与B的,所以还有另一个版本的类型转换(转换2):

* `host -> const D * -> const B *`: 转换成功

这两个转换都是针对B版本调用的,初看感觉二者类似,但实际上,根据[C++标准][c++11],二者还是有优劣

#### 自定义转换 和 标准转换

这2种转换涉及到了2种C++的类型转换方式:

* **标准转换** Standard Conversion,即C++自行完成的所有隐式类型转换,包括5类: 精确匹配,左值转换(包括左值转右值,数组/函数转指针),修饰符调整(const/volatile,俗称cv),类型提升(当需要时,整型/浮点型都可以自动提升至宽度更大的相关类型,比如char->int,重点是值域不会有损),类型转换(不同类型的转换,值域一般会有损,比如int->unsigned int.还包括指针类型转换和bool的转换).这四个的优先关系从上到下依次降低(这个在选择重载函数时会用到)
* **自定义转换** User-Defined Conversion,即我们自己通过类型转换操作符的定义来完成的(就像host那样)

2种转换的基础上,就有了类似上面分析的B版本调用的转换序列:

* **标准转换序列** 即都是由标准转换构成的(四个类别最多分别取一个)
* **自定义转换序列** 即 标准转换序列 + 自定义转换 + 标准转换序列

转换序列的比较(比如重载时的选择),一般是根据标准转换的级别来比较的,对于自定义转换,一般要前后添加必要的"精确匹配"来帮助判断.

序列的等级,以该序列中最高的等级为准

更详细的说明,可见[C++标准][c++11]的13.3.3.1.1和13.3.3.1.2,都有更为精确的说明

#### B版本调用的选择

此时,再来看B版本调用的2中转换:

* 转换1: cv调整(`host -> const host`, rank = 3), 自定义转换(`const host -> const B *`)
* 转换2: 自定义转换(`host -> const D *`), 指针转换(`const D * -> const B *`, rank = 5)

看样子好像是转换1更高,但其实不然,转换2的完整顺序其实是:

* 转换2: 精确匹配(`host -> host`, rank = 1), 自定义转换(`host -> const D *`), 指针转换(`const D * -> const B *`, rank = 5)

自定义转换序列中,自定义转换的前后是必须有标准转换的(当然,一旦转型成功,自然就没有了),所以转换1的后面没有精确匹配,但转换2却有

也可以看出来,如果调用B版本的话,是需要用转换2的,即:

* `host -> host -> const D * -> const B *`

如果要调用D版本,我们的转换序列为:

* `host -> const D *`: 转换3(这个压根不需要标准转换,因为一次自定义就够了,不像转换2,必须标准转换才行,所以此处前不加host->host)

这两个的级别相同(都是1,因为都有精确匹配),但转换次数不一,根据规则,选择较短的

所以当D : B时,实际调用了D版本,check返回的int,自然value=true

### D不是B的派生类

此时,转换2是不可行的(最后的指针转换无法成功),所以我们只有:

* 转换1: `host -> const host -> const B *`
* 转换3: `host -> const D *`

一个是自定义转换序列,一个是自定义转换,二者是不可比较的(或者说一样好),此时会判断函数类型,普通函数优先于模版函数

所以转换1被选择,调用了B版本,check返回char,value=false

### const的重要性

试想,如果host的const B *转换去掉了后面的host,此时:

* 转换1: `host -> const B *`
* 转化2: `host -> const D * -> const B *`
* 转换3: `host -> const D *`

当D : B时,显然选择转换1,此时value=false

当D不派生自B时,二者一样好,优先普通,还是转换1,此时value=false

有兴趣的同学可以自行实验下

## 总结

其实这个是C++的一个大坑,即重载函数的选择,除了完全精确匹配非常好认外,其他的都是很坑爹的

我们大致上有这么几步:

1. 写出每个重载的类型转换序列
2. 自定义转换序列前后添加适当的精确匹配
3. 如果都是序列,则比较标准转换的等级,选择较高的
4. 如果级别一样,则选择较短的
5. 如果无法比较级别,或级别/长短都一样,则普通函数优于模版函数
6. 如果到了这里,一般就编译错误吧

我不敢说这个顺序完全对,但大致是可以分析出来的,如果有疑惑,最好还是code一下,用编译器来说话

不过话说,如果你的重载函数真得这么复杂才能分析,我倒是觉得你的设计有问题

## 后话

貌似"较短"原则不是看长短的,具体的参看[C++标准][c++11]的13.3.3/1,里面有提到过一丝丝.下次如果我有碰到类似的概念,再记录下

不过常见情况下,等级相同看长短也就够了

另.我比照的[C++标准][c++11]是2011年的ISO/IEC 14882:2011,也许和即将出版的C++14的节数不同,大家意会即可

再另.boost的开发者是怎样的脑子啊,这些稀奇古怪的边边角角都能利用到,真乃神人.我等凡人,还是好好的利用大神们提供的工具吧

[is_base_and_derived]: http://www.boost.org/doc/libs/1_56_0/boost/type_traits/is_base_and_derived.hpp
[boost]: http://www.boost.org/
[c++11]: http://www.open-std.org/jtc1/sc22/wg21/docs/standards#14882 "2011的标准草案"
