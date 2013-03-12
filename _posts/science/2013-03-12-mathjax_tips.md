---
layout: post
title: MathJax小知识
category: scienct
tag: mathjax
---

## 转义符号

在`$$`或者`$$$$`中,有很多命令是以`\`开头的,如果后面跟着命令,比如`\begin`等,这些是无需关心转义的,但后面如果跟着符号,比如`\\`(这个是Array的换行符),我们就必须进行转义,输入`\\\\`,类似的还有括号输入`\{`需要编程`\\{`等

## 等号

等号不能置于开头位置,否则就会被解释为分行符号

参看[这篇文章][algorithms_00]

[algorithms_00]: /algorithms_chap_00_problem
