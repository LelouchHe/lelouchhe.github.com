---
layout: math
title: 尝试使用MathJax
category: science
---

## 使用的工具

由于以后肯定要写公式(数学/物理),所以研究了下如何在pages上写

经过比对,比较好的方案是基于js的[MathJax](http://www.mathjax.org/),可能的缺点就是比较慢,经常性的能看到整个公式从原文转换到公式的整个过程

## 注意要点

先看个成功的例子:

$$x = {-b \pm \sqrt{b ^ 2 - 4ac} \over 2a}.$$

要输入的数学公式不能用`\t`开头写成代码那样,因为添加的`pre`会阻止MathJax的解析,比如下面这个就不显示:

    $$x = {-b \pm \sqrt{b ^ 2 - 4ac} \over 2a}.$$

其次,一些$LaTex$符号不能写到一起,否则就会解释成另外的东西,这个是和`markdown`的解释冲突,比如下面这个:

$$x = {-b \pm \sqrt{b^2 - 4ac} \over 2a}.$$

处理这种冲突需要谨记,[MathJax](http://www.mathjax.org/)是在对应html文档生成好之后才处理公式的.

再次,要行内显示数学公式,必须按照[MathJax](http://www.mathjax.org/)上面说的那样,因为行内公式默认没有开启

    <script type="text/x-mathjax-config">
    MathJax.Hub.Config({tex2jax: {inlineMath: [['$','$'], ['\\(','\\)']]}});
    </script>

## 感慨

基本就这些了,当然肯定还有很多**坑**在前面,慢慢学习吧

推荐看好心人翻译的[文档](http://mathjax-chinese-doc.readthedocs.org/en/latest/index.html)
