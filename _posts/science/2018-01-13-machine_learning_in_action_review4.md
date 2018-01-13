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

假设数据实例是由不同的可数字化的数据属性构成的,即$d_i = (x_1, x_2, \cdots, x_n)$,其对应的值也是可数字化的$o_i$,试图求权值$w = (w_1, w_2, \cdots, w_n)$,对既有数据集中的所有数据实例,满足$o_i = \prod _ {i = 1} ^ {n} w_i * x_i$

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

## 机器学习上的解答

数学上的解答都有或多或少的局限(当然,可参看[chapter08][chapter08]那些改进的数学解法),因为从数学上来看,归根到底是求方程的解.而从工程上来看,这个可不一定.在工程上,我们追求的是误差,只要误差在一定的可控范围之内,我们都会认为是正确的解答.就好象前文提到的朴素贝叶斯一样,数据属性相互独立虽然不符合现实,但在工程上结果足够好,就是可以接受的.

### 误差

先定义误差.最常见的误差,是[最小均方差(Minimum Mean-Square Error)][MMSE],即针对当前的$\vec {w}$,其误差为:

$$
E(\vec {w}) \equiv \frac {1} {2} \sum _ {d \in D} (t _ d - o _ d) ^ {2}
$$

其中$t_d$是根据当前$\vec {w}$和数据实例$d$计算出来的值,即$t_d = \vec {d} * \vec {w}$

我们的最终目的,就是要求得一个$\vec {w}$,使得$E(\vec {w})$最小,即:

$$
\vec {w} \equiv argmin E(\vec {w})
         = \frac {1} {2} \sum _ {d \in D} (t _ d - o _ d) ^ {2}
$$

### 梯度下降(Gradient Descent)

最优化问题在数学上已经研究的很多了,对于线性回归而言,一个非常通用的方法是[梯度下降][gradient descent].它在n维的解空间内,给我们提供了一种贪心的方法,让我们能很快速的找到一个局部最优解.

梯度就是二维平面中斜率的通用高维版本,沿着梯度,可以最快速的上升或下降.其表示如下:

$$
\nabla E(\vec {w}) \equiv [\frac {\partial E} {\partial w_1}, \frac {\partial E} {\partial w_2}, \cdots, \frac {\partial E} {\partial w_n}]
$$

当我们已经找到解空间的某个$\vec {w}$时,通过计算当前点的梯度,我们就能知道该如何更新当前的$\vec {w}$,使得$E(\vec {w})$减小.其更新如下:

$$
\vec {w} \gets \vec {w} + \Delta \vec {w}
         = \vec {w} + ( - \eta \nabla E(\vec {w})) 
$$

其中,$\eta$被称作学习速率,控制着更新$\vec {w}$的幅度.以分量表示则为:

$$
w_i \gets w_i + \Delta w_i
    = w_i + ( - \eta \frac {\partial E} {\partial w_i})
$$

然后根据$E$和梯度的定义,我们可以进一步变形为:

$$
w_i \gets w_i + ( - \eta \frac {\partial E} {\partial w_i})
    = w_i - \eta ( - \sum _ {d \in D} (t_d - o_d) x_{id})
    = w_i + \eta \sum _ {d \in D} (t_d - o_d) x_{id}
$$

这样,我们就可以从任意一个$\vec {w}$出发,通过梯度下降算法,逐步更新到使$E(\vec {w})$最小的局部最优$\vec {w}$

### 随机梯度下降(Stochastic Gradient Descent)

梯度下降算法好虽好,但对于每一个$w_i$的更新,都需要把所有的数据实例计算一遍,在某些大数据集中,性能会比较差,所以就有一种对梯度下降的改进算法,[随机梯度下降][Stochastic Gradient Descent],来解决这个问题

在随机梯度下降算法中,$\Delta w_i$的计算不再需要所有数据实例,而是随机的从数据集中挑选一个即可,即:

$$
w_i \gets w_i + \Delta w_i
    = w_i + ( - \eta \frac {\partial E} {\partial w_i})
    = w_i + \eta (t_j - o_j) x_{ij}
$$ 

$t_j$,$o_j$和$x_{ij}$都是随机从既有数据集中挑选的一个实例.此处虽然没有考虑到整体,但在实践中,效果和之前的基本一致.

注意,虽然在工程上我们经常省略这个忽略那个,但这并不代表工程问题是可以蒙混过关的.一来这些省略与折中背后其实都有数学上的证明为依靠,并不是拍脑袋的结论,二来这些都是经典做法,业界使用良久,不论从深度还是广度,都经过的大量的打磨与改善,也并不是随意而为.我们自己在处理问题时,首先应该是查找有没有标准的工具,其次是通用的思路与解法,最后才是自己的发明创造.独创的自然很好,但对于没有经过严格训练的咱们程序员们,还是安心用别人的轮子或点子的好.谨记.









































[chapter05]: https://github.com/LelouchHe/machine_learning_in_action_code/tree/master/chapter05
[chapter08]: https://github.com/LelouchHe/machine_learning_in_action_code/tree/master/chapter08
[MMSE]: https://en.wikipedia.org/wiki/Minimum_mean_square_error
[gradient descent]: https://en.wikipedia.org/wiki/Gradient_descent
[Stochastic Gradient Descent]: https://en.wikipedia.org/wiki/Stochastic_gradient_descent