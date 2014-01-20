---
layout: post
title: vector的resize和reserve
category: code
tag: tip
---

## 缘起

最近在刷leetcode,在切[word ladder][wl]的时候一直re,死活不知道问题在哪里

google了半天,也没有什么收获,大家好像都比较强,所以都是只有正确答案,或者tle,像re的小问题都木有人碰

最后查了半天,有人指出re的[几个可能][re reason]:

* 数组越界
* 除0
* 栈溢出
* 访问空地址

好吧,这其实就是seg fault的几个原因(原来leetcode的re是这个意思).最后细细看代码,才发现问题

## resize vs reserve

resize和reserve长的很像,但功能差距很大

* vec.reserve(size)只是预留size的空间,但这些空间在真正写之前,都是不能访问的.比如下面的例子,虽然不是每次都会sf,但在严格的条件下(如leetcode),也确实sf没错
    
    vector<int> vec;
    vec.reserve(10);
    vec[0] = 1; // sf

* vec.resize(size)这个才是真正分配size的使用空间,我们可以随意的在size范围之内读写,都没有问题.上面那个例子,换成resize就万事大吉了

由于这个细节的问题,[word ladder][wl]和[word ladder II][wl II]都被我快刷爆了,不过这也侧面反映出来,c++基本功太差了

## 教训

* 基本的细节还是要弄清楚的,不能指望拿着手册就好.像我每次开题都备一本stl手册,但这些细节问题匆匆看一眼,哪能有印象
* c++功底荒废了许多,标准库还是非常赞的,抓紧时间好好看看
* 野蛮的刷不能解决问题,多思考,多探索

加油吧,少年,M$,美国

[wl]: http://oj.leetcode.com/problems/word-ladder/
[wl II]: http://oj.leetcode.com/problems/word-ladder-ii/
[re reason]: http://www.1point3acres.com/bbs/thread-67220-1-1.html
