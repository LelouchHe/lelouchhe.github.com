---
layout: post
title: Go闭包的一个小点
category: mine
---

分析下如下代码的输出:

## 测试1
    
    package main

    import (
        "fmt"
        "time"
    )

    func main() {
        for i := 0; i < 100; i++ {
            func() {
                fmt.Println(i)
            }()
        }
        time.Sleep(1e9)
    }

## 分析1

答案显而易见,自然是0~99,因为闭包中的函数使用的是当前的i,而这个i是从0递增的

## 测试2

    package main

    import (
        "fmt"
        "time"
    )

    func main() {
        for i := 0; i < 100; i++ {
            func(i int) {
                fmt.Println(i)
            }(i)
        }
        time.Sleep(1e9)
    }

## 分析2

答案其实还是很好理解的,此时已经不是闭包了,已经成了带有一个参数的匿名参数,参数也是递增的,所以输出仍然是0~99

## 测试3

    package main

    import (
        "fmt"
        "time"
    )

    func main() {
        for i := 0; i < 100; i++ {
            go func(i int) {
                fmt.Println(i)
            }(i)
        }
        time.Sleep(1e9)
    }

## 分析3

这个看上去和2一样,其实结果也是一样的,因为`go`仅仅是开了个`goroutine`而已,匿名函数的参数仍然是递增的,所以输出不变

## 测试4

    package main

    import (
        "fmt"
        "time"
    )

    func main() {
        for i := 0; i < 100; i++ {
            go func() {
                fmt.Println(i)
            }()
        }
        time.Sleep(1e9)
    }

## 分析4

这个又回到了1的闭包结构,结果却迥然不同,输出的全是99.

为什么呢?因为这就是闭包的特点,闭包包含着外部的环境变量值(此处是i),但这个环境变量值并不像匿名函数那样作为参数副本,而是实实在在在的引用(或者指针,反正一个意思),当外部变量变化时,闭包能使用的值自然也就变化了,这个是特性(具体可以参见`Lua`的upvalue的实现)

为什么输出99呢?因为这个循环运行的比`goroutine`还快,`goroutine`还没有开始,for循环已经结束了,此时i的值就是99了,所以闭包内的i自然就是99

## 测试5

    package main

    import (
        "fmt"
        "time"
    )

    func main() {
        for i := 0; i < 100; i++ {
            go func() {
                fmt.Println(i)
            }()
            time.Sleep(1e7)
        }
        time.Sleep(1e9)
    }

## 分析5

我们给每次循环加一个延迟(10ms),这样是`goroutine`有时间来执行,结果和1一样,输出0~99

## 结论

闭包和匿名函数类似,不同的是闭包不止有函数的结构(即函数体),还有函数的环境(内部使用到的外部的变量),这个环境不是作为副本传入的(那该有多慢),而是在使用到的时候进行动态加载使用的,所以不要搞混了

闭包可以减少参数的传递,使得函数可以维持数据,但在有些情况下,注意环境的动态性,小心那些不经意的bug

ps: Go越来越好用了 ;)

## 再补一个测试

最后补一个从[DCCMX][dccmx]看来的例子,这个例子加深了对闭包的印象:

    package main
    import (
        "fmt"
        "time"
    )

    func main() {
        var fn [100]func()
        for i := 0; i < 100; i++ {
            fn[i] = func() {
                fmt.Println(i)
            }
        }

        for i := 0; i < 100; i++ {
            fn[i]()
        }
    }

请大家自己看看这个输出什么,并且和下面这个对比对比

    package main
    import (
        "fmt"
        "time"
    )

    func main() {
        var fn [100]func()
        var i int
        for i = 0; i < 100; i++ {
            fn[i] = func() {
                fmt.Println(i)
            }
        }

        for i = 0; i < 100; i++ {
            fn[i]()
        }
    }

说实话,闭包有趣,但是坑也很多的.

[dccmx]: http://blog.dccmx.com/2011/08/variable-in-clousre/ "再谈闭包"
