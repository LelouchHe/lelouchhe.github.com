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

从线性代数的角度来看,就是已知$m \times n$矩阵D,$m$的列向量$\vec {o}$,求满足下面等式的$m$的列向量$\vec {w}$:

$$
D \vec {w} = \vec {o}
$$






[chapter05]: https://github.com/LelouchHe/machine_learning_in_action_code/tree/master/chapter05
[chapter08]: https://github.com/LelouchHe/machine_learning_in_action_code/tree/master/chapter08