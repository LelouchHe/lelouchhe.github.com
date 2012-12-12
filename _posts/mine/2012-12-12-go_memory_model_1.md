---
layout: post
title: Go内存模型1
category: mine
---

## 例子

看看如下的代码有怎么样的输出

    package main

    import "fmt"

    var a int
    var b int

    func main() {
        go func() {
            a = 1
            b = 2
        }()

        fmt.Println(a)
        fmt.Println(b)
    }

## 答案

其实正确的答案应该是不确定,比如在我的机子上打印出来的是"0 2",别的自己上等我回去验证下.

## 解释

其实这样说明个什么道理呢?在一个goroutine内部,我们可以严格的保证顺序,比如在上面那个闭包里,如果我们还在赋值之后使用a,那么a的值必然是1,因为在单一的goroutine中是顺序的,并没有并发之类的东西.然而在多个goroutine之间,这种顺序性是无法保证的,比如上面的a和b,两次赋值和两次读取,所以就无法保证执行的顺序,在我的机子上,就先执行了`b = 2`,然后才执行了`a = 1`

我们可以验证下单一goroutine的情况

    package main

    import (
        "fmt"
        "time"
    )

    var a int
    var b int

    func main() {
        go func() {
            a = 1
            b = 2
            fmt.Println(a)
            fmt.Println(b)
        }()

        time.Sleep(1)
    }

在这种情况下,只能打印出"1 2".在举个例子加深下印象

    package main

    import "fmt"

    var a int
    var b int

    func main() {
        go func() {
            a = 1
            b = a + 1
        }()

        fmt.Println(a)
        fmt.Println(b)
    }

能猜到答案么?呵呵,还是**不确定**.在我的机子上,打印出来的还是"0 2",虽然b非常的依赖于a,但在外部的goroutine眼中,这些都是浮云.

## 结论

这个例子告诉我们一个道理,即Go的内存模型不能保证不同goroutine之间的顺序性,这种顺序性只能保证存于单一的goroutine中.在我们的并发编程中,我们不能依赖于其他goroutine的顺序来做出判断,就如此例一样,如果有这样的想法,必然带来问题.

解决之道就是Go的那句格言:"不要通过共享内存来通信,而要通过通信来共享内存"

其实说的就是上面这个例子,而这只是Go内存模型的一个小点.

