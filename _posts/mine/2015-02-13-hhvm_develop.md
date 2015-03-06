---
layout: post
title: hhvm扩展开发笔记
category: mine
tag: c++ hhvm
---

## 简介

hhvm无法在运行时动态的加载模块,所有的扩展函数/类都必须在初始化时加载,因此类似python那样使用bridge进行动态绑定,是无法实现的

但php原生的支持include机制,再加上hhvm支持原生php和native代码的组合使用,可以变相的完成动态加载的功能.

因此,hhvm支持的代码分为2个部分:

* interop/hhvm: 这部分定义了hhvm扩展的基本操作接口和一些基本模块(包括config和exception),都是一些功能性的底层模块
* sofa/compiler/hhvm: 这部分用于动态的生成组件对应的php,通过使用1提供的模块,完成hhmv与sofa之间的交互

hhvm和sofa的通信主要是通过函数调用完成的.对于一些比较固定的通信,比如构造函数(都是默认构造函数),init(参数都只有config)或config/exception,可以使用固定的函数调用来完成.然而对于用户自定义的组件来说,由于接口是变化的,因此无法通过正常的函数调用来完成(参数/返回值都没有固定格式).因此,在实现上,采用了嵌入汇编的方式,手动搭建调用堆栈,来完成组件在hhvm与sofa之间的通信功能.这是hhvm扩展实现的重点
所以,下面主要分2个部分,第一部分是C++汇编的相关细节,第二部分则是在此基础上的hhvm扩展的开发

## C++汇编简介

### 调用规范

调用规范(Calling Conventions),主要是指编译器生成的汇编代码遵循的一套规范,规定了函数调用的一些细节要求,从而使得遵循同一套规范的不同代码可以相互调用.这些细节主要包括参数的传递,返回值的位置,寄存器的使用等.目前,在x86-64平台下,主要有gcc和ms两套规范,我们这里主要着眼于gcc的规范

#### 调用简介

从汇编角度来看,函数调用就是一个简单的跳转指令(jmp或call).调用方事先将参数/返回值之类的被调函数需要用到的东西,按照调用规范准备好,然后就直接跳转到被调函数那里进行执行了.而被调函数也按照调用规范从固定的位置来取得相应的数据,把返回值放到规范要求的位置,最后在跳转回来

常用的指令是jmp系列和call+ret.

* jmp系列: 主要完成的是无条件跳转,仅仅是简单的修改pc寄存器的值,没有任何额外的信息.可以用于简单的跳转,需要调用方自行保存返回地址之类的事情
* call+ret: 函数调用的主要指令.call指令的结果也是修改了pc寄存器的值,但call还会将当前的pc值入栈,这样,当调用结束后,ret指令就能取到当前栈顶的返回地址,直接跳转返回.这个指令主要就是简化返回地址的处理,不过带来的问题是,我们需要保证ret时,栈顶确实就是入栈的原来的返回地址.这是通过严格遵守调用规范来保证的

cpu通过一个隐式的stack,来保存我们的调用环境,我们通过push/pop指令来完成于该stack的交互.该stack内容包括调用参数,返回值,返回地址,局部变量等信息,它的使用也是要遵循调用规范的

一般来说,这都是由gcc来处理的,但由于此处我们要手动完成这个任务,所以这些都需要我们自己来完成

#### 参数的传递

参数的传递规范,指导我们在调用函数时,如果把相应的参数传给被调函数.

参数一般是按照声明顺序,从左到右来进行传递的.具体的位置:

* 如果参数的个数小于等于6个,则按照顺序存放在寄存器中: rdi, rsi, rdx, rcx, r8, r9
* 当参数个数大于6个时,多余的参数要存放到栈上,按照声明顺序,位置是从低到高

需要注意的是,栈是从高往低扩展的,因此多余参数不是简单的从左到右依次push即可,这样的顺序就反了.一般有2种方式:

* 对于多余的参数,直接从右到左,依次push
* 事先计算好数量,提前把栈的位置空出来,在按照从左到右的顺序,依次mov过去

比如,有如下函数:

    int test(int arg1, int arg2, int arg3, int arg4, int arg5, int arg6, int arg7, int arg8);

其调用的汇编伪代码,大致如下:

    # 先将前6个参数放到寄存器里
    movq arg1, %rdi
    movq arg2, %rsi
    movq arg3, %rdx
    movq arg4, %rcx
    movq arg5, %r8
    movq arg6, %r9
    
    # 再处理多余参数
    push arg8
    push arg7
    
    # 接着就是调用了
    # call test

此时,cpu的状态是:

![参数传递][param]

#### 函数的入口

将参数按照如上的位置放好之后,就可以调用函数了.执行call指令,首先将返回地址入栈,然后将pc设置为call的参数值,就去执行被调函数的代码了.此时,一个典型的调用状态如图:

![函数调用][call]

注意此处一个特殊的bp寄存器

被调函数的第一个指令一般是

    # 保存旧的bp
    push %bp
    
    # 保存基准
    movq %sp, %bp
    
    # 空出局部变量空间
    subq %sp, 20

这个的作用主要是给当前调用栈一个基准(base),因为不论最后被调函数怎么变动,bp是恒定的,那么栈上的参数和栈内的局部变量相对于bp的偏移也是恒定的,这样就便于我们使用bp来获取参数或局部变量了

当然,这只是一种可能的实现.gcc在优化情况下,或者使用一个编译选项,可以不生成类似代码

#### 返回值

test函数有一个int类型的返回值.返回值一般是通过rax返回的.函数的结束一般都是这样的:

    # 恢复栈
    addq %sp, 20
    
    # 恢复bp
    pop %bp
    
    # 返回
    ret

可以看到,这个顺序和刚才的入口代码正好的反着的,从而保证了各个数据可以恰好对的上

对于返回值,后面还会详细说明

#### 资源维护

一个函数调用会产生很多资源的占用,比如占用了一些寄存器,占用了一些栈空间,函数执行中和执行完毕之后,都有一些资源维护工作要完成,否则有可能调用方和被调用方相互覆盖,导致最后出错.

由某方维护的意思是,该方可以直接使用该空间资源而不用担心被覆盖占用.对于那些不是该方维护的空间资源,使用前则必须保存,使用完之后,必须恢复原值.

大致来讲,双方的责任是:

* 调用方: 调用参数, rbp, rbx, r12-15
* 被调用方: 局部变量,除上面寄存器外的其他寄存器

所以,刚才我们看到,为什么bp寄存器需要先保存起来,才能被赋值,以及最后为什么必须得恢复原值了.

#### 浮点数的处理

为了提高处理浮点数的效率,cpu使用特殊的指令来操作浮点数.在默认情况下,gcc(3.4.5和4.8.2,其他版本没有尝试)会优先选择SSE指令和xmm寄存器.此时,和上面的普通的调用规范就有所区别了,而且对于单精度和双精度的指令也是不一样的

xmm寄存器有8个,从xmm0到xmm7.当参数中有浮点数时,要优先按顺序使用者8个寄存器,其余的参数,同普通参数一起,也是按照顺序放到栈上.如果返回值是浮点数的话,通过xmm0返回

具体指令的话,可以参看SSE的手册,此处只说明2个:

* movlpd: 将内存某双精度浮点数移动到xmm寄存器中
* movss: 将内存某单精度浮点数移动到xmm寄存器中

因此,处理参数/返回值时,需要区分普通/单精度浮点/双精度浮点三种类型分别进行处理

### 返回值优化

当函数以值的方式,返回一个较复杂的局部变量时,C++标准规定,必须通过返回值优化(RVO)的方式,来避免不必要的复制操作

经过探索,对于gcc(3.4.5和4.8.2)来说,处理返回值有几种情况:

* 返回值是浮点数,参照上面提到的,利用xmm进行返回
* 返回值的size小于等于16字节,且是POD时,利用rax+rdx进行返回.
* 其他情况(即大于16字节,或者不是POD),需要在调用方的栈空间中临时分配空间,并把该地址作为函数的第一个参数传入

第三种情况,则是RVO尽量避免复制的情况.此时,调用方分配裸空间,大小为返回值类型大小,但不进行构造或初始化,直接把该空间地址(相当于指针)当作第一个参数传给被调函数,该地址的构造初始化和其他操作,均有被调函数来完成,当函数执行完毕后,返回值不需任何操作,即可使用

所以,当手动生成调用栈时,不仅要处理不同的参数类型,还要判断函数返回值的大小和类型,从而调整整个调用栈的内容.在调用结束使用返回值时,也需要从正确的位置获取返回的值

### 异常处理
被调函数有可能抛出异常,因此在手动搭建调用栈时,需要能够处理这种情况

此处请参看[嵌入汇编如何捕获异常][exception]

### 虚函数表

sofa的Struct和Service的接口都是以虚函数的形式暴露的,在汇编层面,我们需要取得该虚函数实际指向的函数地址,才能调用该函数

以下面的类结构为例:

    class A {
    public:
        A() {}
        virtual ~A() {}
        virtual void a(){}
    };
    
    class B {
    public:
        B() {}
        virtual ~B() {}
        virtual void b(){}
    };
    
    class C : public A {
    public:
        C() {}
        virtual ~C() {}
        virtual void a() {}
    };
    
    class D : public A, public B {
    public:
        D() {}
        virtual ~D() {}
        virtual void a() {}
        virtual void b() {}
    };

使用-fdump-class-hierarchy选项编译,可以生成其类结构信息,可以看到如下的结构:
 
    Vtable for A
    A::_ZTV1A: 5u entries
    0     0u
    8     (int (*)(...))(&_ZTI1A)
    16    A::~A
    24    A::~A
    32    A::a
    
    Class A
        size=8 align=8
        base size=8 base align=8
    A (0x7f55d89e3880) 0 nearly-empty
        vptr=((&A::_ZTV1A) + 16u)
    
    
    Vtable for B
    B::_ZTV1B: 5u entries
    0     0u
    8     (int (*)(...))(&_ZTI1B)
    16    B::~B
    24    B::~B
    32    B::b
    
    Class B
        size=8 align=8
        base size=8 base align=8
    B (0x7f55d89d0780) 0 nearly-empty
        vptr=((&B::_ZTV1B) + 16u)
    
    
    Vtable for C
    C::_ZTV1C: 5u entries
    0     0u
    8     (int (*)(...))(&_ZTI1C)
    16    C::~C
    24    C::~C
    32    C::a
    
    Class C
        size=8 align=8
        base size=8 base align=8
    C (0x7f55d89c5000) 0 nearly-empty
        vptr=((&C::_ZTV1C) + 16u)
    A (0x7f55d89c5080) 0 nearly-empty
        primary-for C (0x7f55d89c5000)
    
    
    Vtable for D
    D::_ZTV1D: 11u entries
    0     0u
    8     (int (*)(...))(&_ZTI1D)
    16    D::~D
    24    D::~D
    32    D::a
    40    D::b
    48    -8u
    56    (int (*)(...))(&_ZTI1D)
    64    D::_ZThn8_N1DD1Ev
    72    D::_ZThn8_N1DD0Ev
    80    D::_ZThn8_N1D1bEv
    
    Class D
        size=16 align=8
        base size=16 base align=8
    D (0x7f55d89b3880) 0
        vptr=((&D::_ZTV1D) + 16u)
    A (0x7f55d89b3900) 0 nearly-empty
        primary-for D (0x7f55d89b3880)
    B (0x7f55d89b3980) 8 nearly-empty
        vptr=((&D::_ZTV1D) + 64u)

其结构如图:

![虚函数表][virtual]

可以看到,虚函数表分为4个部分:

* 真实对象的偏移.这个主要是当你使用基类指针指向派生类对象时,真实对象的地址与该基类地址之差.可以看到,A于D的差是0,表示二者是重合的;但B与D之差是-8,表示如果是B的地址需要加上-8,才能得到D的地址.这一点从图上可以看到
* 类型信息
* 析构函数.虚函数表总有2个析构函数.这个具体原因不明
* 虚函数列表.虚函数的顺序严格按照继承顺序.这样,当使用A* ad = new D;和A* aa = new A,我们使用ad/aa的代码是一致的,因为他们对应的虚函数表,在A的范围内是完全一致的

这种构造,可以保证,从任何一个合法的位置分开,都是一个完整正确的虚函数对象.从上图看,不论是用A,还是用B,面对的对象和虚函数表的结构是一致的,所以处理代码完全不用变动.

虚函数表的这个特性,使得我们可以不依赖于具体实现,而只依靠接口(sofa中由idl生成的纯虚类),就能得到其任意虚函数在表内的偏移,这一点在后面动态生成php代码中非常关键,因为那个时候,我们只有idl,没有任何实现

### 其他细节

嵌入汇编的写法此处不赘述,有一些细节需要注意

* 嵌入汇编中一般不进行函数调用,所以gcc生成的汇编代码里没有给局部变量分配空间,直接使用了sp以下的栈(反正不会调用函数,也没人用),但由于我们要调用函数,所以必须手动调整sp,给gcc空出来它自己分配的局部变量
* C++中,引用即指针,二者的区别只存在于C++语法中,在汇编层面,都是一样的,可以一致分析
* 参数/返回值分析时,引用/指针也是类型的一部分,不要直接忽略了

## hhvm扩展

hhvm是C++开发的,所以也是以C++的形式来开发其扩展

### 扩展框架

hhvm提供了一个扩展类 ::HPHP::Extension,我们只需要继承该类,并override其中三个虚函数即可

* virtual void moduleLoad(::HPHP::Hdf config): 扩展加载时,执行的函数.config是启动hhvm时指定的配置,类似于::sofa::Config.目前,在这里,完成sofa的初始化
* virtual void moduleInit(): 模块加载完毕,进行初始化.目前,基本模块和操作原语就是在这里被加载到hhvm中的,这个之后,就可以在hhvm中使用这些了
* virtual void moduleShutdown(): 程序结束卸载模块时执行.目前的操作是执行sofa的清理操作

### 注册基本模块和操作

由于涉及到和native代码的交互,所以初始化阶段,基本模块/操作的注册分为2步

#### 注册接口

扩展继承的::HPHP::Extension中,有一个接口用于注册php代码形式的接口,并提供一种特殊的标注语法,区别开php代码和native代码.如下:

    std::stringstream ss;
    ss << "<?hh namespace sofa; "
       << "abstract class Object { "
       << "    protected \\int $handle; "
       << "    public function handle() : \\int { return $this->handle; } "
       << "    public function __destruct() : void { "
       << "        if ($this->handle != 0) { "
       << "            self::release($this->handle);"
       << "            $this->handle = 0;"
       << "        } "
       << "    } "
       << "    <<__Native>> protected static function release(\\int $handle) : \\int; "
       << "    <<__Native>> protected static function add_ref(\\int $handle) : \\int; "
       << "};"
       << "abstract class Struct extends Object { "
       << "};"
       << "abstract class Service extends Object { "
       << "};";
    ext->CompileSystemlib(ss.str().c_str(), "systemlib.php.sofa.object");

CompileSystemlib接受2个参数,第一个参数是php代码形式的接口样式,第二个参数是注册接口的名称

注册的php代码,可以是任意的hhvm接受的php代码.可以看到,此处定义了三个有继承关系的抽象类,还有一些标注"<<__Native>>"的接口

"<<__Native>>"表示该接口是以native代码实现的,hhvm需要从扩展中,而不是在该php代码中查找其对应的实现

注意,完全php代码的接口也是可以的,就好像此处的"handle()"和"__destruct()"一样,hhvm接收纯php的扩展

#### 注册实现

由于我们用到了native实现,所以还必须对标注为"<<__Native>>"的接口的实现进行注册.如下:

    ::HPHP::Native::registerBuiltinFunction(::HPHP::makeStaticString("sofa\\Object::release"), reinterpret_cast<void(*)()>(do_release));
    ::HPHP::Native::registerBuiltinFunction(::HPHP::makeStaticString("sofa\\Object::add_ref"), reinterpret_cast<void(*)()>(do_add_ref));

registerBuiltinFunction接收2个参数,第一个参数是Native接口的名称,必须是全称(包含命名空间),第二个参数则是对应的native实现.此处的do_release和do_add_ref都是C++函数

注册实现时,需要统一转型为void(*)().不用担心参数类型问题.hhvm会根据我们注册接口时的php代码标注的类型帮我们检查类型.只要保证注册的接口和此处注册的实现的参数类型是兼容的即可,否则调用时就是参数类型错误

### 类型关系
hhvm将php中的类型和C++的类型做了对应,参考[hhvm扩展文档][extension]:

hhvm的文档基本等同于没有,所以建议在开发时,手边常开一个hhvm的源码窗口,便于及时的找到源文件来查看各个类型的接口和实现

在调用native代码之前,hhvm会根据接口的类型进行校验,如果不匹配,就会报参数类型错误.但这个错误在一般情况下不会输出.所以建议:

* 调试时,通过hhvm初始化配置,把日志级别调到最低,这样才能看到所有日志
* php接口和native实现的类型必须按照上表完全匹配,因为一旦php接口类型无误,hhvm就不再检查native的类型了(因为已经转义为void(*)())了

### 类型转换

上文提到过,hhvm与sofa之间的交互,主要是由嵌入汇编完成的.在这个层面,我们需要保证hhvm的类型和sofa的类型是兼容的.目前的转换,主要是将hhvm类型打包成汇编层面的sofa类型值,以便作为参数,直接传递给sofa组件的实现

这些转换,底层操作是由interop/hhvm提供,sofa/compiler/hhvm中根据反射信息,提前动态生成好的

#### hhvm到sofa(参数)

* bool: 转为0/1的int64_t即可.汇编代码中只检查LSB,所以没有问题
* int: 可以直接转为int64_t.因为整型在位数上是兼容的(8->16->32->64扩展即可),所以被调函数会取到正确的数据进行解释的.
* 浮点型: 浮点型不像整型那样,位数/表达/指令都是不兼容的,所以必须分别处理
* double: 因为位数正好是64,所以直接reinterpret_cast转成int64_t即可.有xmm就存放在xmm中,否则就放在栈上
* float: 由于无法区分,所以传入时打包成array类型.即array($float).这样在汇编处理时,可以很简单的同double区分.有xmm时,直接赋值到xmm,没有的话,只好进行特殊编码存放在栈上.由于float只有32位,所以可以将高32位置1,二者拼接为64位,再转型为int64_t存放.高32位为1的double是非法的,所以不用担心会误伤.而且被调函数有对应参数的类型,只会从低32位来取float,所以也是没问题的.
* String: 新建std::string对象,将String的值复制一份,并返回该std::string的地址.能这么做,主要是因为std::string在sofa中,一般就2种用法,const std::string&, 或 std::string*. 都是指针类型
* Array: Array有三种对应,vector,list和map,我们从sofa的反射信息中,能够知道是哪个类型,选择合适的转型函数
    * vector: 这里又分为2种,vector<bool>在sofa中是单独表示的,其他的vector是另一种.但转型方式类似,都是新建sofa::vector,遍历原数组进行赋值,最后返回地址
    * list: 新建sofa::list, 遍历原数组并进行赋值,最后返回地址
    * map: 新建sofa::map,遍历原map并进行赋值,最后返回地址
* Object: 注册接口时看到,所有的Object表示都有一个handle,这个handle就是sofa::Object的指针,因为sofa内的计数都放在Object里,所以可以放心的将这个指针传给被调函数,而不用担心生命期的问题

#### sofa到hhvm(返回值)

* bool: 直接按照0/1来解释即可
* int: 返回值在rax中,所以只能根据sofa反射信息,得到具体的整型类型,然后对rax进行截断处理
* 浮点型: 返回值在xmm0种.也只能根据sofa反射信息,选择正确的指令,从中取得正确的浮点值
* String: 作为返回值,有2种场景:
    * sofa::Struct: 返回的是std::string&, 只是一个简单的指针而已.可以直接返回,强制转型,再赋值给String即可
    * sofa::Service: 返回的是std::string.返回的返回值需要RVO,所以只能提前分配好sizeof (std::string)空间,然后把该地址强制转型为std::string*,再赋值给String返回
* Array: Array的三种类型处理类似上面的过程.不过有2种场景
    * sofa::struct: 此处返回的一般是容器的引用,也就是一个指针(rax),可以直接强制转型为对应的容器,然后遍历,赋值给新建的Array即可
    * sofa::service: 此处一般是按值返回容器,需要RVO,类似String,需要提前分配空间,再进行上面的操作
* Object: 返回的即是sofa::Object的指针,直接赋值到Object.handle即可

### 分工

#### interop/hhvm

提供的基本模块:

* sofa\Object
* sofa\Config
* sofa\Exception

提供的基本操作

* sofa\import: 引入某sofa模块
* sofa\cast: sofa之间类型转换
* sofa\export_service: 导出sofa服务

提供的基本原语.以下原语不建议用户直接使用,主要用于代码生成模块来完成hhvm/sofa交互的

* sofa\create_struct: 构建sofa::Struct
* sofa\create_service: 构建sofa::Service
* sofa\invoke: 汇编调用函数

#### sofa/compiler/hhvm

根据idl信息,使用上面提供的基本模块/操作,动态生成xx.yy.zz.ver_a_b_c.php.该文件是hhvm/sofa交互的主要文件,用户通过"sofa\import('xx.yy.zz.ver_a_b_c')"导入

这里的难点主要在于

* 虚函数位置的确定.代码生成主要发生在用户提交idl时,此时只有idl,因此只能通过sofa的反射信息,遍历对应Struct/Service的继承结构,来计算每个虚函数接口的位置
* 参数/返回值处理. 这个主要涉及到汇编调用参数/返回值的确定.上面对此有详细的说明.难点在于处理的情况非常多

生成的php文件于idl和sofa生成的纯虚类是一起打包的,因此对所有使用idl的用户都是可见的(因为至少需要idl,所以co时会获取到),这样也就解决了该php文件存放位置的问题.

[param]: /image/param.png "参数传递"
[call]: /image/call.png "函数调用"
[exception]: /exception_in_inline_assembly
[virtual]: /image/virtual.png "虚函数表"
[extension]: https://github.com/facebook/hhvm/wiki/Extension-API#php-and-c-extensions
