---
layout: post
title: Go中switch和select的区别
category: mine
tag: Go
---

## 相似点

switch和select是Go中容易混淆的一对关键字,主要是因为它们都是用了case结构,都有如下的形式:

    switch {
    case a:
    case b:
    default:
    }

    select {
    case a:
    case b:
    default:
    }

刚刚在学习time.Ticker的使用时,我就错误的把select写成了switch,结果在编译的时候go就不断的提醒我说"invalid case <-ticker.C in switch (mismatched types time.Time and bool)"

## 不同点

其实从上面的错误信息就能看得出来它们的一个区别,即对于没有参数的switch来说,在case处希望获得的是bool变量,这也是我们能用swtich替代复杂的if-else的原因

    if a {
        doA()
    } else if b {
        doB()
    } else if c {
        doC()
    }

    switch {
    case a: doA()
    case b: doB()
    case c: doC()
    }

上面的两种写法是等价的,而且Go还鼓励我们把复杂的条件判断化简为switch来操作

但select在这种情况下就有很多限制,case里必须是通过channel得到的值,可以是明确的值(`v := <-ch`),或者只是判断有无(`<-ch`);不能在case中使用fallthrough,因为select本来是用于通信的,基本不可能有模棱两可的情形(这样只能说是设计的bug)

其次,二者对于case的求值也是不同的,在switch中,case是从上向下依次求值的,只要某次为真,就执行语句,然后跳出(不是用fallthrough),但是在select中,case是同时去**求值**,但不一定去**赋值**,也就是说,所有的case都会进行判断是否有新值到达channel,然后分开不同的情况讨论:

1. 如果只有一个case有值,那么就简单直接的选择这个case
2. 如果有多个case有值,那么随机选择一个case执行
3. 如果没有case有值,那么就直接block住(在没有default的情况下)

比如如下的代码:

    package main

    import (
        "fmt"
        "time"
    )

    func main() {
        ch := make(chan int)

        go func() {
            for {
                select {
                case <-ch: fmt.Println(0)
                case <-ch: fmt.Println(1)
                }
            }
        }()

        go func() {
            for {
                ch <- 0
            }
        }()

        time.Sleep(2 * 1e9)
    }

输出的结果是很经典的随意0/1串,看到了吧,当ch有值的时候,所有的case都会进行求值,发现所有case都有值,那么select就随机选择一个case,从而产生了随机0/1串.要注意的是,虽然同时求值了,但是ch中的值还是只能赋值一次,因为赋值是原子操作,一次就没有了

再次,二者对于default的解释不一样

1. switch中的default是默认的意思,当所有case不满足的时候,就会执行default
2. select中的default是当select发现没有case满足,要block时的选择

其实本质都是一样的,一种所有case不满足时的选择,不过要充分理解两者对于case解释的不同,这样才会理解default,而不会犯错

