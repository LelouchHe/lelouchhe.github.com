---
layout: math
title: 算法之美第一章 习题
category: algorithm
tag: algorithms
---

## 1.1

b进制数最大n位数为$b ^ n - 1$,所以b进制最大的2位数为$b ^ 2 - 1$,最大1位数为$b - 1$,而
$$
b ^ 2 - 1 \geq 3 (b - 1)
$$
恒成立(移项后化简,且$b \geq 2$)

重点是怎么找到问题关键,并用数学的形式表达

## 1.2

因为$2 ^ 4 - 1 = 15 \gt 9$,所以4位2进制数能表达最大值大于1位10进制数表达的最大值,所以相同数字的2进制数位数不会大于10进制数的4倍(其实等同于比较$2 ^ 4 \gt 10$)

N在b进制下的位数大致为$\log _ b N$,所以2进制位数与10进制位数比大致为$\log _ 2 N / \log _ {10} N = \log _ 2 10$

## 1.3

当d叉树为满树的时候,深度最小(因为最多的节点都被压宽,而没有增加深度),所以有如下公式:
$$
1 + d + d ^ 2 + d ^ 3 + \cdots = \frac{d ^ H - 1}{d - 1} \geq n
$$
求解该方程,即可得到H(深度,为了避免和d冲突所以称为H)的结果$H \geq \log _ d [(d - 1) n + 1]$,即可得到结论

## 1.4

$$\log {n !} = O(\log {n ^ n}) = O(n \log n)$$

$$\log {n !} = \Omega (\log {(\frac{n}{2}) ^ \frac{n}{2}}) = \Omega (\frac{n}{2} \log {\frac{n}{2}}) = \Omega (n \log n)$$

根据Hint来非常容易,需要注意的是,如何省略来简化运算,比如2式中真正算下来会有一项$- n / 2$,但是这项不足以影响到阶($n \log n$),所以可以忽略而没有影响

同时,我们可以看到,考察渐进度的时候,如何进行简化和忽略.一个常见的方法,就是同时放大/缩小每一项,或者添加/删除一些项

## 1.5

设
$$
H(n) = 1 / 1 + 1 / 2 + 1 / 3 + 1 / 4 + \cdots = \sum _ {i = 1} ^ n 1 / i
$$
所以
$$H(n) \leq 1 / 1 + 1 / 2 + 1 / 2 + 1 / 4 + \cdots = \Theta (\log n)$$
$$H(n) \geq 1 / 1 + 1 / 2 + 1 / 4 + 1 / 4 + \cdots = \Theta (\log n)$$

不等式右边是根据Hint来的,这样就能很容易看出$H(n) = \Theta (\log n)$

二式最右边等号之所以成立,是我们把$H(n)$按照二进制位来进行了分解,可以尝试加和一下不等式右边的无限序列,就能明白了,最后的个数(所有元素都是1)就是二进制位数,即$\log n$

## 1.6

其实P23的乘法算法就是简单的模仿了我们平时进行的竖式乘法,只不过把下一个结果向左移(竖式乘法)变成了左移被乘数(乘法算法),本质上是一致的

## 1.7

根据先前的分析,迭代需要至少n次(每次减少1位,共n位),每次迭代中,可能有加法,这个需要O(max(m, n))次,所以总的时间是O(n max(m, n))

## 1.8

递归的算法一般要利用归纳法来证明.

1. 当$x = 0$时,$(q, r) = (0, 0)$是显而易见的

2. 当$x = 2 k$时,即x为偶数,我们可以看到
$$x = 2 k = y q + r$$
$$x ' = k = y q ' + r '$$
所以可以看到,$q = 2 q ', r = 2 r '$,得证

3. 当$x = 2 k + 1$时,即x为奇数,可以这样来看
$$x = 2 k + 1 = y q + r$$
$$x ' = k = y q ' + r '$$
$$2 x ' = 2 k = y \cdot 2 q ' + 2 r '$$
所以,$q = 2 q ', r = 2 r ' + 1$(注意加1补齐缺少的1,只能r来补).这样没错,但是会产生一个问题,就是$r$可能越来越大(因为只考虑了相等,而没有考虑余数),所以需要特别判断当$r \geq y$的时候,需要做调整

时间复杂度类似于乘法,递归n次,每次O(n),总时间O(n ^ 2)

可以看到,证明很容易,但是要凭空设想出这样精巧的方案,确实很难

递归,抓住结束条件,归纳而上

## 1.9

由$x \equiv x ' \mod N$可知$x - x ' = a N$,同理$y - y ' = b N$,二式相加,得$(x + y) - (x ' + y ') = (a + b) N$,这样就能得到题目$x + y \equiv x ' + y ' \mod N$

乘法也是类似的,只不过为了方便计算,我们把等式左边的量单独拿出来,而不是像上面一样做成差的形式

$$
x y = (x ' + a N) (y ' + b N) = x ' y ' + N(x ' b + y ' a + a b N) \equiv x ' y ' mod N
$$

## 1.10

因为$x \equiv y \mod N$, 所以$x - y = a N$, 又因为$N = b M$, 二式比较得$x - y = a b M$, 所以$a \equiv b \mod M$

## 1.11

刚才看错了,以为是$\mod 5$,我就说怎么这么简单

$$4 ^ {1536} = (4 ^ 3) ^ {512} = 64 ^ {512} \equiv (-6) ^ {512} = (6 ^ 2) ^ {256} = 36 ^ {256} \equiv 1 \mod 35$$

$$9 ^ {4824} = (9 ^ 3) ^ {1608} = 729 ^ {1608} \equiv (-6) ^ {1608} = (6 ^ 2) ^ {804} = 36 ^ {804} \equiv 1 \mod 35$$

所以$4 ^ {1536} \equiv 9 ^ {4824} \mod 35$

ps:这种问题怎么凑?我们利用的性质是
$$a \equiv b \mod N, c \equiv d \mod N \Rightarrow a + c \equiv b + d \mod N, a c \equiv b d \mod N$$
这个性质的主要利用点是找到同余项,比如此处的$4 ^ {1536}$和$9 ^ {4824}$

如果是我之前看到的$\mod 5$,那么4和9本身就满足条件(都余4),但$\mod 35$,我们就需要其他的项,一般是试错性质的比较次幂的同余,逐步来得到最终的结果

pss:隔了很久再来看上面一段,方法是正确的,但结论是错误的,此处能找同余项么?显然不能,比如就算是$\mod 5$,找到的4的同余$4 ^ {1536}$和$4 ^ {4824}$同样不能认为是同余的,因为二者的次幂不一样

其实我们可以总结出这样的规律

$$
a ^ x \equiv b \mod N \Rightarrow a ^ y \equiv b ^ {\lfloor y / x \rfloor} a ^ {y \ mod x} \mod N
$$

可以简单的利用乘法替换原则来解释.这个规律的意义在于指导我们进行同余变换(比如本题),更重要的是,由于$b$的存在,我们倾向于使$b = 1$来简化计算,这也是为什么很多和$\mod$相关的题目,都在找1的原因(此处是第一处找1的原因)

## 1.12

$$
2 ^ {2 ^ {2006}} = (2 ^ 2) ^ {2 ^ {2005}} = 4 ^ {2 ^ {2005}} \equiv 1 ^ {2 ^ {2005}} = 1 \mod 3
$$

从解这两道题,我们可以看到,1在$\mod$运算中的地位,因为1不论怎么计算,最终结果都是1,所以在求解类似题目的时候,要首先往1方向靠

ps:我们直接借鉴1.11的公式来计算下

$$
2 ^ 2 \equiv 1 \mod 3 \Rightarrow 2 ^ {2 ^ {2006}} \equiv 2 ^ {2 ^ {2006} \mod 2} \equiv 2 ^ 0 \equiv 1 \mod 3
$$

这样计算更加直观和快速.我们固然是可以从乘法替换原则入手解答,但如果有更高层的公式来使用,解决也就更直观

## 1.13

$$5 ^ {30000} = (5 ^ 3) ^ {10000} = 125 ^ {10000} \equiv 1 \mod 31$$

$$6 ^ {123456} = (6 ^ 3) ^ {41152} = 216 ^ {41152} \equiv (-1) ^ {41152} = 1 ^ {41151} = 1 \mod 31$$

所以二者之差$\mod 31$为0,即可以整除31

看,仍然是凑1战术,1非常有特殊性,0的话是整除,这个比较好看,对于不好看的取模,找1是比较妥当的

ps:可以同样直接利用公式来进行尝试

## 1.14

根据hint来看,时间复杂度为$O(M (n) \log m)$,其中$M(n)$为每次乘法操作的时间复杂度,一般是$O(n ^ 2)$,n为乘数的位数.

再看本题,由于进行取模运算,所以每次的乘数最大不会超过p,所以是时间复杂度为$O((\log p) ^ 2 \log n)$,如果p为常数,则简化为$O(\log n)$

ps: 再次回顾0.4,看到了当初为什么以$M(n)$而不是常数来进行判断,固然矩阵算法对于求解$F_n$的复杂度为$O(\log n)$,但是每次乘法的运算是不一样的,此处正好是$\mod p$,所以得到如上的结论

## 1.15

本题是对加法/乘法替换准则的反向思考,在和/积确定的时候,什么时候可以拆分

先是正向思考,如何得到原问题推理的结论呢?显然,如果存在$x ^ {-1} \mod c$的话,根据乘法替换准则,可以得到结论.而$x ^ {-1} \mod c$存在的前提是$(c, x) = 1$,即互质

换一种思路,

$$
a x \equiv b x \mod c \Rightarrow a x - b x = c k \Rightarrow a - b = c \frac{k}{c}
$$

由于$a - b$是整数,所以$c (k / x)$必须也是,如果c和x互质的话,那$k / x$必然是整数,所以$a - b \equiv 0 \mod c$.如果c和x不是互质,右边则不一定是整数了,也就不一定能整除,从而证明结论了

所以充要条件是互质,这也是唯一可以对乘法进行拆分的条件

ps:还有一种类似的规律,p和q互质的时候,$p | a 且 q | a \Leftrightarrow p q | a$.这是一个比较基本的规律了,但经常容易遗忘,而且此时拆分的不是被模项,而是模数,需要仔细辨别

## 1.16

首先尝试计算下连续平方法的乘法使用次数,以11为例,由于$(11) _ {10} = (1011) _ 2$,而且,除最高位外,其余位为0时,进行1次乘法(平方),否则进行2次乘法(平方+自乘).设b的二进制位数为n,其中1的个数为m,则乘法次数为$(n - 1) + (m - 1)$

可见,如果没有1的阻碍(需要进行自乘),平方法应该是最快的了,当b为2次方幂时,就是这样的情况.当1逐渐增多,甚至$b = 2 ^ n - 1$,全都是1时,自乘带来的额外乘法就比较大了

比如$b = (15) _ {10} = (1111) _ 2$,单纯的平方法需要6次乘法,但如果尝试更高次数的乘法,$(b (b ^ 2) ^ 2) ^ 3$,只需要5次即可.更高次幂同样带来的更多的乘法(k次幂带来$k - 1$次乘法),但个人感觉这中间有一个边界,一旦超过这个边界,高次幂的快速增长,可以弥补带来的更多的乘法

## 1.17

迭代算法,第i次时,$x ^ i$同$x$相乘,复杂度为$O(i \log x \log x)$,共迭代$y - 1$次,所以总的复杂度为:

$$
\sum _ {i = 1} ^ {y - 1} O(i (\log x) ^ 2) = O(y ^ 2 (\log x) ^ 2) 
$$

递归算法,第i次时,$x ^ (2 ^ i)$自乘,复杂度为$O((2 ^ i \log x) ^ 2)$,递归深度为$O(\log y)$,所以复杂度为:

$$
\sum _ {i = 1} ^ {\log y} O((2 ^ i \log x) ^ 2) = O(y (\log x) ^ 2)
$$

通过计算也能看到,复杂度降低了一个数量级

## 1.18 

需要注意gcd_ext的退出条件,不要弄混淆了,代码如下:

    int gcd_ext(int a, int b, int& x, int& y) {
        if (a < b) {
            return gcd_ext(b, a, y, x);
        }

        if (b == 0) {
            x = 1;
            y = 0;
            return a;
        }

        int nx = 0;
        int ny = 0;
        int d = gcd_ext(b, a % b, nx, ny);
        
        x = ny;
        y = nx - a / b * ny;
        return d;
    }

## 1.19

思路怎么出来?考虑这样的变换:

$$
gcd(F _ {n + 1}, F _ n) = gcd(F _ {n} + F _ {n - 1}, F _ n) = gcd(F _ {n - 1}, F _ n)
$$

同时,初始状态下$F _ 1 = 1, F _ 2 = 1$,所以利用数学归纳法就可以证明了

怎么想到这样处理的?在证明gcd算法正确时,曾经证明过$gcd(a, b) = gcd(a + b, b)$,如果当时用心并熟悉的话,应该很快就能看到这个pattern的

## 1.20

求逆的话,利用gcd_ext就能比较容易的得到,这里要重点说明的是第2个重要的1,即

$$x x ^ {-1} \equiv 1 \mod N$$

前提是$gcd(x, N) = 1$,即x和N互质.然后利用$x x ^ {-1} + N b = 1$和gcd_ext,就可以求的对应的值了

互质的前提是非常重要的条件,一旦碰到求逆,就要想到是否互质.比如题目中$21 \mod 91$的逆就不存在,你想到这点了么?

## 1.21

一提到逆元,肯定是互质,这就相当于求与$11 ^ 3$互质的且属于$[0, 11 ^ 3)$的整数个数

一些类似的求解个数的问题:

* 求$[0, n]$内,k的倍数($> 0$)的个数,可以简单的写为$\lfloor n / k \rfloor$,可以举一些例子帮助看清规律
* 求$[0, n]$内,a,b倍数的个数,这是上面的简单推广,利用集合的特性,结果为$\lfloor n / a \rfloor + \lfloor n / b \rfloor - \lfloor n / (a b) \rfloor$,更多的情况,也都是按照标准的集合方式处理

回到本题,就是找$[0, 11 ^ 3)$中不是11倍数的个数,需要注意的是0和$11 ^ 3$都不能选,最后的结果就是$(11 ^ 3 - 1) - (11 ^ 2 - 1)$,前面减1是排除0,后面减1是排除$11 ^ 3$

## 1.22

a对模b存在逆元,表示a,b互质,且$a a ^ {-1} \equiv 1 \mod b \Rightarrow a a ^ {-1} - 1 = b k$,两边同时对a取模,则有$b (-k) \equiv 1 \mod a$,结论成立

可以看到,在互质的前提下,被模和模数是相对的(当然,对应的逆元肯定是不同的啦)

## 1.23

唯一性的证明,往往是利用反证法,假设不唯一,然后推出矛盾

设a和N互质(存在逆元),且有$a i \equiv 1 \mod N$,假设存在$j \neq i$,且$a j \equiv 1 \mod N$

二式相减,得$a (i - j) \equiv 0 \mod N$,a和N互质,所以只能$N | (i - j)$了,但$i \in [0, N), j \in [0, N) \Rightarrow (i - j) \in (-N, N)$,所以只能$i = j$,与假设矛盾

ps:本章有几个证明,非常经典,需要仔细的理解

* gcd 包括正确性和复杂度
* gcd_ext 正确性
* 逆元 互质和唯一性
* 费马小定理 重排列的唯一性
* 费马测试 1/2概率证明
* rsa 解码正确性
* 通用散列函数族 正确性和概率

以上都是重要的推理过程,需要确保理解到位

## 1.24

1.21推广的内容,即$(p ^ n - 1) - (p ^ {n - 1} - 1)$

## 1.25

还是按照公式来做,因为$2 ^ 8 \equiv 1 \mod 127$,所以有$2 ^ {125} \equiv 2 ^ {125 \mod 8} \equiv 2 ^ 5 \equiv 64 \mod 127$

或者,我们可以根据提示(按照上面的公式,我们完全没有用到提示诶),质数有一个定理(费马小定理),即$a ^ {p - 1} \equiv 1 \mod p$,此处即是$2 ^ {126} \equiv 1 \mod 127 \Rightarrow 2 2 ^ {125} \equiv 128 \mod 127$,由于2与127互质,根据1.15,得$2 ^ {125} \equiv 64 \mod 127$.和第一种方法一致

ps:这里出现的就是第二个重要的1,即当p为质数时,$a \in [0, p) \Rightarrow a ^ {p - 1} \equiv 1 \mod p$.当需要1或者存在质数时,可以试着向这个方向思考下

## 1.26

求个位数,明显是要求$ \mod 10$的值

$10 = 2 \times 5$,且2和5均为质数,所以有$17 ^ {(2 - 1) (5 - 1)} \equiv 17 ^ 4 \equiv 1 \mod 10$,然后套进公式,得$17 ^ {17 ^ {17}} \equiv 17 ^ {17 ^ {17} \mod 4} \equiv 17 ^ 1 \equiv 7 \mod 10$

ps:这就引入了第3个重要的1.对于不同质数p,q,任意a,$a ^ {1 + (p - 1) (q - 1) \equiv a \mod p q}$,当$gcd(a, p q) = 1$时,有$a ^ {(p - 1) (q - 1)} \equiv 1 \mod p q$

我们现在以自己的理解来证明下这个问题:

p是质数,所以$a ^ {p - 1} \equiv 1 \mod p$,有了1,根据公式,推出$a ^ {(p - 1) (q - 1)} \equiv a ^ {(p - 1) (q - 1) \mod (p - 1)} \equiv a ^ 0 \equiv 1 \mod p$,两边同乘以a,得$a ^ {1 + (p - 1) (q - 1)} \equiv a \mod p \Rightarrow a ^ {1 + (p - 1) (q - 1)} - a \equiv 0 \mod p$.同理对于q也是如此,$a ^ {1 + (p - 1) (q - 1)} - a \equiv 0 \mod q$.

pq都是质数,肯定互质,所以根据1.15的推广(模数的乘法原则,整除且互质即可相乘),有$a ^ {1 + (p - 1) (q - 1)} - a \equiv 0 \mod p q$,两边同时加a,定理即得证.当$gcd(a, p q) = 1$时,根据1.15,就得到了重要的1了

可以看到以下方面:

* 相同模数的加减,乘法是任意的
* 不同模数,只有整除且模数互质的情况下,模数可乘(可以用简单的倍数关系得证),被模数可以相同从而保持不变,也可以相乘

## 1.27

回顾下RSA的过程:

1. 取p,q为质数,设$N = p q$,$r = (p - 1) (q - 1)$
2. 取e,使得$gcd(e, r) = 1$,求d,满足$e d \equiv 1 \mod r$
3. 公钥即为$(N, e)$,私钥为d
4. 加密函数$encode(m) = m ^ e \mod N$
5. 解密函数$decode(c) = c ^ d \mod N \equiv m ^ {e d} \equiv m \mod N$(可以从1.26证明)

根据题目的条件,可以简单的算出$d = 235$,$encode(m) = m ^ e \mod N = 105$

## 1.28

有题目可得$N = p q = 77$,$r = (p - 1) (q - 1) = 60$,e的唯一硬性约束是与r互质,那么最小就是7,此时$d = 7 ^ {-1} \mod r = 43$

## 1.29

### a

证明类似与书上的证明,即假设存在相等的两项,当确定了$a _ 1$时,$a _ 2$有且只有1个取值满足条件,这样就保证了概率和期望

### b

$m = 2 ^ k$,这样上面的证明不可行了,因为其中关键一步(求逆元)是比如互质的,m的取值无法保证每个都满足条件,所以不可行

可以感性的思考下,最后的$\mod 2 ^ k$,其实就是取了末位的k位而已,前头直接被忽略了,这样就给取值留下了很多选择,必然就不是唯一的了

### c

$[m] \to [m - 1]$,表示必有2个映射到1个元素上,共有$C _ {m - 1} ^ 1 = m - 1$种选择,任意选择2个,恰好映射到该元素的概率是多少?$\frac{1}{m - 1} * \frac{1}{m - 1}$(二者都从$m - 1$个数中,映射到了那1个元素),所以总的概率是$1 / (m - 1)$,所以得证

ps:补充下通用散列函数族的定义,按照书上所讲,设函数集合的个数为H,函数值的个数为n(即最后能映射到的桶位置),对于任意不同的x和y,恰有$H / n$个函数使得二者的函数值相同

这样的意义在于,从函数集合中随机取一个,任意不同x和y映射的值冲突的概率是$H / n / H = 1 / n$.这一点在证明中,尤其重要,只要我们证明了不同x和y冲突的概率为$1 / n$即可证明这点,具体的过程可以参考书上和c的解法

## 1.30

先去wiki下什么叫做"进位前瞻回路加法",当然,这个翻译的名字很扯淡,原名叫做"Carry-lookahead adder",中文应该是"超前进位加法器"

简单来说,假设2个n位二进制数a和b相加,每位独立计算的,其中有2个数据,G表示当前位,P表示当前位向下一位的进位:

* $G _ i = (a _ i & b _ i) | (a _ i & P _ {i - 1}) | (b _ i) & P _ {i - 1}$
* $P _ i = a _ i \oplus b _ i \oplus P _ {i - 1}$

一开始觉得很夸张,但仔细的穷举所有可能的值之后,确实是这样的运算规则.名称中的"carry-lookahead"表示我们不进行逐位逐位的计算,而是直接把G/P展开,变成a和b的直接计算值,这样每位的计算都是独立的,不相互影响,可以并行计算

### a

把n划分为$\lfloor n / 2 \rfloor$,递归深度是$\log n$,每次加法的复杂度是$O(\log m)$,所以总的复杂度是$O(\log n \log m)$

### b

这个要求的方法就是上面说的"超前进位加法器":

* $r = (x & y) | (x & z) | (y & z)$
* $s = (x \oplus y \oplus z) << 1$

s表示进位(所以需要左移一位),r表示当前位值

### c

参考b的解法,3个数的加法变成2个数的并行加法,可以想到,将n分成3组,然后3变2,这样一直划分下去,直到最后成为2个n位数相加

深度是$\log n$,最后加法的深度是$\log n$,所以总的深度是$O(\log n) + O(\log n) = O(\log n)$

## 1.31

### a

根据1.4,可以知道$\log (N !) = \Theta (N \log N)$,且$n = \log N$,所以位数近似为$\log (N !) = \Theta (n 2 ^ n)$

### b

按照常规的乘法进行计算即可,复杂度为$O (\sum _ {i = 0} ^ n i \log i \log i) = O (N ^ 2 (\log N) ^ 2)$

ps:感觉solutions给出的答案有些问题,$N !$应该和N相关,如果单论n位数的话,最小和最大之间还有很多乘法没有考虑到,所以感觉不是很对

## 1.32

### a

可以采用经典的二分查找,复杂度是$O(\log N)$

### b

分情况讨论:

* $N = 1$时,$q = 1$,成立
* $N \ge 2$时,如果q存在的话,$q \ge 2$.原等式变换一下,$N = q ^ k \Rightarrow \log N = k \log q \Rightarrow k = \log N / \log q \le \log N$,成立

所以原结论是成立的

### c

判断是否幂,即$N = q ^ k$,此时q和k都未知.但是根据b的结论,k是有范围的,$k \in [2, \log N]$,所以可以枚举k的值,来判断q是否存在.使用经典的二分查找,需要枚举$\log N$次,每次枚举中,二分查找q的值,需要$O(\log N)$,乘幂计算需要$O(\log k)$次,所以最后的复杂度为

$$
O(\sum _ {k = 2} ^ {\log N} \log N \log k) = O((\log N) ^ 2 \log (\log N))
$$

利用了1.4的结论$\log (n !) = \Theta (n \log n)$

## 1.33

根据$lcm(a, b) = \frac{a b}{gcd(a, b)}$,可知其复杂度为$O(n ^ 3)$

## 1.34

还记得期望的公式么?$E(x) = \sum i p(i)$,所以利用期望公式可以计算,即我们的提示1

$$
E = \sum i (1 - p) ^ {i - 1} p = \frac{1}{p}
$$

或者更加简单点,借用递归的说法

$$
E =
\left
\\\{
\begin{aligned}
1 & \, & 概率为p \\\\
1 + E & \, & 概率为1 - p
\end{aligned}
\right.
= 1 p + (1 - p) (1 + E) \Rightarrow E = \frac{1}{p}
$$

后者还是借用期望公式,只不过利用递归的方式简化了运算

## 1.35

又是质数,又是1,记住我们3个重要的1:

* $a ^ x \equiv b \mod N \Rightarrow a ^ y \equiv b ^ {\lfloor y / x \rfloor} a ^ {y \mod x} \mod N$,$b = 1$时尤其有用,无前提条件
* $x x ^ {-1} \equiv 1 \mod N$,前提是$gcd(x, N) = 1$,x和N互质
* $x ^ {e d} \equiv 1 \mod p q$,前提是p和q是质数,$gcd(e, (p - 1) (q - 1)) = 1,e d \equiv 1 \mod (p - 1) (q - 1)$

虽然不一定能用到,但是最好想到

### a

$$
x ^ 2 \equiv 1 \mod p \Rightarrow (x + 1) (x - 1) \equiv 0 \mod p \Rightarrow x \equiv 1 \mod p 或 x \equiv -1 \equiv p - 1 \mod p
$$

### b

因为p是质数,所以$\forall x \in [1, p), x x ^ {-1} \equiv 1 \mod p$,因为只有1和p-1是自身,其余的都不同,所以可以两两配对,得

$$
(p - 1) ! = 1 \times 2 \times 3 \cdots (p - 2) \times (p - 1) \equiv 1 (p - 1) \equiv -1 \mod p
$$

$ \mod p$是很强的分类准则,一下就限定了取值的范围

### c

还是反证法,不过要能从反证中推出矛盾,还是需要一定的琢磨和研究的

假设N为合数时,依然有$(N - 1) ! \equiv -1 \mod N$,两边同乘-1,得$(-1) (N - 1) ! \equiv 1 \mod N$,所以看的出来$(N - 1) !$有逆元(-1),所以$(N - 1) !$和$N$互质.但N为合数,$(N - 1) !$中必然有$N$的因数,二者必然不互质,所以矛盾了

ps:还是想说一句,推导的重点还是围绕3个1来展开,只能说,在我目前接触的最初等的这部分模运算中,3个1是重中之重

### d

从计算量上就能看出区别,费马小定理计算$a ^ (p - 1)$,a可以取的很小,但这个Wilson定理需要计算的是$(p - 1) !$,即使都要$ \mod p$,差距也是很大的

## 1.36

质数的性质其实我们知道的不多,总结下一般就这么几条:

* 费马小定理 p是质数,则$\forall a \in [1, p), a ^ {p - 1} \equiv 1 \mod p$
* 模数乘法原则 p和q互质,$p | a, q | b \Rightarrow p q | a b$
* 除法原则 $a c \equiv b c \mod N$,且$gcd(c, N) = 1$,则$a \equiv b \mod N$

(如果以后还有的话,需要继续补充)

### a

$$
p \equiv 3 \mod 4 \Rightarrow p = 4 k + 3 \Rightarrow p + 1 = 4 (k + 1)
$$

问题得证

### b

$$
(a ^ {(p + 1) / 4}) ^ 2 = a ^ {(p + 1) / 2} = a a ^ {(p - 1) / 2}
$$

如果要$a ^ {(p + 1) / 4}$是a的平方根,则$a ^ {(p - 1) / 2} \equiv 1 \mod p$

从质数能得出哪些性质呢?$a ^ {p - 1} \equiv 1 \mod p$,所以$a ^ {(p - 1) / 2} \equiv 1 \mod p$或者$a ^ {(p - 1) / 2} \equiv -1 \mod p$

如果是第一种情况,那么问题得证了

假设是第二种情况,再加上$a \equiv x ^ 2 \mod p$,得到$(x ^ 2) ^ {(p - 1) / 2} = x ^ {p - 1} \equiv -1 \mod p$,与费马小定理$x ^ {p - 1} \equiv 1 \mod p$矛盾,所以这个情况不成立

故结论成立

## 1.37

### a

一个明显的规律是二者以$3 \times 5$为周期重复

### b

假设存在相同的2个数,有$a \equiv j \equiv b \mod p, a \equiv k \equiv b \mod q$,二者相减,得$a - b \equiv 0 \mod p, a - b \equiv 0 \mod q$,再根据模数乘法原则,得$a - b \equiv 0 \mod p q$,又因为$a - b \in (-p q, p q)$,所以只能为0,与假设矛盾了

所以针对取模后的特定序列,只存在唯一的整数满足条件

### c

针对问题求解就好了,目前我们可以有以下的结论

$x \mod a b \equiv x \mod a$

可以把模运算变换成乘法运算即可,a和b任意

### d

推广看来,就是解如下的方程:

设$p _ i$为质数

$$
\left
\\\{
\begin{aligned}
x \equiv a _ 1 \mod p _ 1 \\\\
x \equiv a _ 2 \mod p _ 2 \\\\
\cdots \\\\
x \equiv a _ n \mod p _ n
\end{aligned}
\right.
$$

设$M = \prod p _ i, M _ i = M / p _ i, t _ i = M _ i ^ {-1} \mod p_i$,所以有如下性质:

$$a _ i M _ i t _ i \equiv a _ i \mod p _ i$$

$$a _ j M _ j t _ j \equiv 0 \mod p _ i$$

这样,令$x = \sum a _ i M _ i t _ i$,这样就能满足上述的方程了

现在的问题是,我们如何能找到这样答案呢?

从问题a可以感觉到,序列是按照$p q$为周期重复的,所以一个可能的思路是求这些所有质数的最小公倍数(就是$M$),然后再想方法凑一下相等的情况,x中肯定需要有个$a _ i$,同时需要一个数,在$\mod p _ i$时为1,在$\mod p _ j$时为0,可以看到,$M \mod p _ i$都是0,我们从M中把$p _ i$取出即可(就是$M _ i$),然后顺理成章的想到$t _ i M _ i \equiv 1 \mod p _ i$(重要的3个1)

当然,这个思路也不容易想到,如果没有事先了解,感觉可能一下子推导不出来的

## 1.38

## 1.39

又见质数,记住$a ^ {p - 1} \equiv 1 \mod p$和公式配合使用,得$a ^ {b ^ c} \equiv a ^ {(b ^ c) \mod (p - 1)} \mod p$,两次幂模运算,复杂度$O(n ^ 3)$

## 1.40

这个性质我们上面貌似有使用,但一直没有证明,汗

可以利用反证法,假设满足上述条件的,且p为质数.由$x ^ 2 \equiv 1 \mod p$得,$(x + 1) (x - 1) \equiv 0 \mod p$,因为p是质数,所以只可能$x + 1 \equiv 0 \mod p$或$x - 1 \equiv 0 \mod p$,这个和性质矛盾,所以原命题是成立的

也可以看到,这是另一个和质数相关的性质,在确定了满足质数条件之后,可以使用

## 1.41

### a

类比于平方操作,很可能是一正一负,所以当$a \equiv x ^ 2 \mod N$时,有$a \equiv (- x) ^ 2 \equiv (N - x)\mod N$,因为N为奇数,所以二者不相同,所以如果有一个x存在,那么肯定有另一个配对的$N - x$

有没有另一个相同的呢?假设$\exists y \neq x, y \neq N - x, y ^ 2 \equiv x ^ 2 \mod N$,变换得知$(y - x) (y + x) \equiv 0 \mod N$,由于N是质数,根据1.40,得出了矛盾

所以原问题得证

### b

x和$N - x$是配对的,所以除去0之外,共有$\frac{N - 1}{2} + 1$个

### c

这个情况下,N肯定是个合数,所以可以拼凑一个合数解出来

## 1.42

罗列下解码前提,p为质数,e与$p - 1$互质,$e d \equiv 1 \mod p - 1$,现在已知$m ^ e \mod p$,求m

首先从$e d \equiv 1 \mod p - 1$,可以以$O(n ^ 3)$计算出d,又有$m ^ {p - 1} \equiv 1 \mod p$,根据公式,$m ^ {e d} \equiv m ^ {e d \mod (p - 1)} \equiv m \mod p$,所以解码只需将$m ^ e$求d模幂即可,同样是$O(n ^ 3)$

可以看到,如果使用单质数,存在多项式时间完成解密,所以是很不安全的

## 1.43

目前有N,e(为3)和d,求N的因数分解p和q

根据RSA的算法可知,$e d \equiv 1 \mod (p - 1)(q - 1)$,所以有$e d = k (p - 1) (q - 1) + 1 \Rightarrow k = \frac{e d - 1}{(p - 1) (q - 1)}$,其中$e = 3, d \in [0, (p - 1) (q - 1))$,所以有$k = \frac{3 d - 1}{(p - 1) (q - 1)} < 3$,k是正整数,所以可能取值1和2,分别代入,就能求得$(p - 1) (q - 1)$,又同$p q = N$结合起来,就可以求出最后的p和q了

ps:在处理整数的时候,整数的各种性质是需要考虑的,比如模性质,比如正整数/负整数的范围,比如相邻差1等等,都是需要综合起来考量的

## 1.44

这个可以转换为1.37的同余方程,有$m ^ 3 \equiv a _ i \mod N _ i$,已知$a _ i, N _ i$,且$N _ i$之间以大概率的可能性互质,这样我们就可以借用1.37的结论来完成求解$m ^ 3$的值,再开立方即可

ps:此题也提醒我们,同余方程中,并不要求$N _ i$必须为质数,只要各方程之间$N _ i$互质即可

## 1.45

### b

$(M ^ d) ^ e \equiv M \mod N$,只需$O(n ^ 3)$的复杂度就可以验证了

### c

证明见b,同与RSA

### d

此处的N很容易分解,$N = 391 = 17 \times 23$,所以$p = 17, q = 23$,再根据$d = e ^ {-1} \mod (p - 1) (q - 1)  = 145$

## 1.46

### a

签名其实和解密是一致的过程,只要把Alice发给Bob的$m ^ e \mod N$发送给Bob进行签名,$(m ^ e) ^ d \equiv m \mod N$,就得到了原始信息

### b

那只能让给Bob的不像文本即可,加入随机的与N互质的r,求得$r ^ e$,然后把$r ^ e m ^ e$让Bob签名即可,签名后$(r ^ e m ^ e) ^ d \equiv r ^ {e d} m ^ {e d} \equiv r m \mod N$,我们知道了r,就可以通过$m r r ^ {-1} \equiv m \mod N$,就得到了原始信息了

这个巧妙的方法是如何想到的呢?感觉就是重复原始的加密过程,将m混淆成我们知道解法的随机消息,一旦解密,在Bob看来可能是乱码,但我们知道随机的r,所以可以正常解码

不论怎么说,这个思路很巧妙,值得借鉴
