---
layout: post
title: 机器学习实战1 - k最近邻
category: science
tag: ml
---

## k最近邻(k-Nearest Neighbors)

[chapter02][chapter02]

k最近邻应该是一种较为简单的分类算法.俗语有云,近朱者赤,近墨者黑,表达的就是这个意思.它的基本假设是数据分类是聚集的,同一个分类下的数据大体上会聚在一起(这个思想和无监督里的聚类算法类似).在既有数据集中选择k个与目标最近的数据,他们的分类就是目标的分类,具体步骤如下:

1. 计算目标与数据集中所有数据的距离
2. 将距离排序,并选取最近的前k个
3. 统计前k个的分类数量,目标分类是数量最多的分类

这里有2个需要注意的问题:

### 如何计算距离

每条数据有很多的属性(feature/attribute),在机器学习中通常用一个向量(vector)来表示,所有属性经过数字化,成为该向量的值.这样的话,问题就变成计算向量之间的距离.通常我们会选择标准欧式距离,因为这个看起来比较直接,而且在使用中效果不错,但还是有很多选择,可以参考[这里][distance]

我们要根据具体的数据情况,选择合适的距离计算方法.其中,一个重要的思考角度是属性的分布.一些简单的距离计算方法,并没有考虑到不同属性分布带来的属性不均衡问题.比如说,我们使用简单的欧式距离,即计算向量间差的平方和,在这种情况下,如果某个属性的范围比其余属性大若干数量级,别的属性都是0到1之间的,而该属性则是0到10^3之间,那么简单欧式距离的计算结果,基本就依赖于这一个属性了,而这往往不是我们想的.所以[这里][distance]提到的标准欧式距离,首先会将属性正则化(normalization),然后在计算.

但也是有这样的情况,比如我们事先知道某些属性比其他属性重要,那么将所有属性统一正则化,则无法凸显这个特点,此时,我们就要考虑一些带有加权性质的算法,在正则化之后,再对属性进行加权处理.

关键还是要分析和理解数据.通常而言,正则化是必须的,加权是可选的.

### 如何选择k

k的选择也是需要考量的,太小的值会带来太多的随机性(想象k=1),太大的值会影响结果的正确性(想象k=n,即全部数据).所以还是要观察数据,分析数据,并结合过往经验来处理.

一个比较好的方式是计算比较不同k值的错误率,选择最小错误率的k值.这就是[机器学习][mitchell]书上介绍的"评估假设".即使我们有一些经验上的猜测,但建议还是要做一轮这样的评估.将既有数据集分成训练集和测试集两部分,对不同的k,用训练集来预测测试集的分类,并计算错误率,然后选择最小的k值.

过度拟合(overfitting)对于k最近邻来说不是很大的问题(因为我们本质只有一个参数,怎么也over不了多少),而且也是可以通过随机划分来规避的

## 总结

### 优点

* 实现简单
* 原理直接

### 缺点

* 依赖于全部既有数据
* 计算量大 因为每次都必须把同所有数据的距离都计算一遍

### 其他选择

一个更naive的想法是,计算以目标为圆心,r为半径的圆所覆盖的数据的分类情况.这个想法和k最近邻类似,只不过k最近邻依赖的是k个邻居,即使k+1邻居本身也很近,但也不会考虑,而圆覆盖的情况,考虑的更多的是距离本身,有可能这个范围内是全部数据,也有可能是几个(通常也会涉及到r的选择),这样计算距离和r就有了直接的联系,个人感觉比单纯强制k个,要符合实际的多.

不过本人并没有比较过二者的优缺点,在我看来,应该是大体类似,圆覆盖效果可能会略略好一点点

[chapter02]: https://github.com/LelouchHe/machine_learning_in_action_code/tree/master/chapter02
[distance]: http://www.cnblogs.com/heaad/archive/2011/03/08/1977733.html
[mitchell]: https://book.douban.com/subject/1102235/
