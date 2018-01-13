---
layout: math
title: 机器学习实战4 - 线性回归
category: science
tag: ml
---

## 线性回归(Linear Regression)

[chapter05][chapter05]
[chapter08][chapter08]

个人感觉,线性回归是机器学习中**非常**重要的工具.之所以说是工具,而不再是算法,是因为很多更加复杂更加强大的学习算法,就是建立在线性回归的使用上,只不过通过这样那样的手段和组合方式,完成普通线性回归难以处理的问题.

由于逻辑回归(Logistic Regression)和线性回归本质上非常类似,所以我就贸然的把这个分类算法也拉过来说一下.如果懂了线性回归,逻辑回归基本不说也懂了

## 问题

线性回归要解决的问题,和前文提到的朴素贝叶斯试图解决的问题类似.

假设数据实例是由不同的可数字化的数据属性构成的,即$d_i = (x_1, x_2, \cdots, x_n)$,其对应的值也是可数字化的$o_i$,试图求权值$w = (w_1, w_2, \cdots, w_n)$,对既有数据集中的所有数据实例,满足或近似满足$o_i = \prod _ {i = 1} ^ {n} w_i * x_i$

从线性代数的角度来看,就是已知$m \times n$矩阵D,$m$的列向量$\vec {o}$,求满足下面等式的$n$的列向量$\vec {w}$:

$$
D \vec {w} = \vec {o}
$$

## 数学上的解答

如果单纯的把这个当作数学问题来求解的话,事实上已经有很多求解的方法了.

### naive解法

最naive的解法,应该是把这个等式当作$m$个关于$n$个未知$w_i$的方程组来解方程.这种方法我们很小的时候就已经很熟悉了,此处不赘述

从以前解方程的经验,可以直观的知道2点:

1. 当$m < n$时,没有固定解.换成机器学习的术语来说,就是既有数据集比数据属性数还少,是没法求解的.这就是所谓的**欠拟合(underfitting)**或**高bias**
2. 当$m > n$时,不一定有解.换成机器学习的术语来说,就是既有数据集太大或者数据属性太少时,是没法完全拟合的,即必然有错误项

这2点都是以往解方程时要注意的

### 线性代数解法

如果使用线性代数的话,可以非常快速的直接给出答案:

$$
\vec {w} = (D ^ { \mathrm{ T } } D) ^ {-1} D ^ { \mathrm{ T } } \vec {o}
$$

同样的问题,如果$D ^ { \mathrm{ T } } D$没有对应的逆矩阵的话(一种情况就是$m < n$),是没有办法求解的














































[chapter05]: https://github.com/LelouchHe/machine_learning_in_action_code/tree/master/chapter05
[chapter08]: https://github.com/LelouchHe/machine_learning_in_action_code/tree/master/chapter08