---
layout: post
title: printf的缓冲区
category: mine
---

## 场景

今天在写[mroutine](/mroutine)的时候,写了些toy代码熟悉ucontext,代码如下:

    #include <stdio.h>
    #include <unistd.h>
    #include <ucontext.h>

    int main()
    {
        ucontext_t uc;

        getcontext(&uc);
        printf("hello world");
        sleep(1);
        setcontext(&uc);

        return 0;
    }

结果死活就是不出结果,我以为ucontext难道和`printf`不兼容么?显然不应该.我就把`printf`替换成了`puts`,然后就有结果了.

然后我猛然想到难道是`printf`的缓冲区问题?就试着在`printf`里打入了`'\n'`,果然,有结果了

## 总结

其实这只是一个小点,使用流模式的输入输出,我们一定要记住缓冲区的存在,必要的时候要flush,或者简单的输出行(其实还是flush)

ps: 其实我还无聊的做了下面的实验来着

    #include <stdio.h>
    #include <unistd.h>
    #include <ucontext.h>

    int main()
    {
        ucontext_t uc;
        int i;

        getcontext(&uc);
        for (i = 0; i < 10000; i++)
            printf("hello world");
        sleep(1);
        setcontext(&uc);

        return 0;
    }

其实设个变量应该就能更明显的凸显出缓冲区的作用了,不过这个东西还是留给有兴趣的人做吧.

