---
layout: math
title: 算法之美第三章
category: algorithm
tag: algorithms
---

## 图论引论

从本章开始,我们正式进入一个比较复杂的章节.其实单就算法而言,并没有很艰难的实现或者很复杂深奥的计算,相反,图论提供给人的,更多的是一种观察和思考问题的方式

此处,我们不一定会按照书本内容来说,图论这部分更多的参考了[算法笔记][jeff]和[algorithms 4th][algorithms4]这两本书,前者提供了很多新的思考方式,而后者的插图很棒很丰富,代码也很明了,让人看的非常舒适

## 讲解顺序

通过我个人的理解,图论其实大体上分为5个部分,下面分别进行阐述

### 基本概念

认识一个问题,第一步肯定是了解各种概念.所以第一部分我们首先介绍一些关于图论的基本概念.

当然,很可能有些概念不明所以,我尽量会穿插些后面的初级内容,尽量说的明白一些

### 基本操作

这个部分,首先是将基本概念代码化,通过实地的代码,来了解图的各种表示和优缺点;然后就是图的基本操作--遍历,除去代码外,会着重讲解遍历框架和V/E的各种属性区分;最后,则是基本操作的实际应用,即拓扑排序(topo)和连通分量(cc)的求解.

虽然第三块内容可能稍微复杂些,但topo和cc可以给基本操作(图的遍历)一个最简单的应用场景,有助于理解遍历的特性和使用范围

### 图的结构

这部分主要考察和研究图结构方面的性质.我们忽略图中边的权值(也就是无权值图),着重考察节点之间的内在关系.

图的结构有很多有趣的问题,我这里可能提到的有最短路径问题,独立集问题,覆盖问题,二分图等.很多问题可能我还没有掌握,会慢慢的进行介绍

### 图的权值

在结构的基础上,添加边的权值,考察有权值图的性质.这给我们增加了新的有趣的问题,同样也是一般图论书介绍的最多的问题,比如最短路径问题(SSSP和APSP),最小生成树(MST),最大流(Max Flow)等,这些问题人们研究的比较透彻,也有很多非常精妙的算法解决.我们着重考察why和how,当然,还是会有相应的代码实现的

### 图的推广

最后一部分就是图算法的推广.实际问题肯定不会像算法问题一样明确的指明要用图论知识来求解.如何判断和适当的引入图就是一个问题.

比如以后会提到的"job schedule"问题,每个job有工作起始时间,要求求可完成的最大工作数.这个问题可以看做一个图论问题,节点为job,边为相互冲突的job,然后求解最大独立集即可.但这个问题有比较快速的贪心解法(这个后说),所以此处引入图论,只是增加复杂度而已.判断很重要,图毕竟是很复杂的概念,有简化的算法还是简化的好

当然,另一方面,当无法简化时,要注意联想到图.[ADM][adm]中有句话说的好,是"不要试图发明图算法,而是把问题规约为经典图问题".图本身就够复杂的了,图算法更加隐晦,所以在不可避免尝试图论算法时,一定不要重造轮子,一来是造不出来,二来是正确性和复杂度很难保证,所以问题的主攻方向还是**规约**到经典图问题上,然后利用现有的优化算法搞定

当然,这只是其中的一个大的方向,很多东西等到时候再讨论

## 真实顺序

虽然上面的顺序是美好的,但现实总是残酷的,图的结构这部分的资料找的还不是很全面,所以这部分可能会延后,我们先着重把其他4个部分搞定再说

## 基本概念

### 图

图(G)是由节点(V)和连接节点的边(E)构成的,即**G = (V, E)**,V和E可以是任意的集合,只要E中的元素连接V中的两两元素即可

一个泛化的说法是,V是对象,E是关系.只要处理的问题中出现了类似的样式,都可以认为是图的一种

图是广泛存在的,甚至一些我们认为不是的地方,下面举一些例子:

1. 上文提到过的"job schedule",V是job的集合,E是相互冲突的job(的关系的集合)
2. LIS(最长增长子串),V是串中的元素,E是元素与其后不低于其值的元素的集合
3. 编辑距离,V是source可能变换得到的串,E是可能的变换关系
4. 族谱,V是家族中的成员,E是各自之间的亲缘关系

可以举的例子还有很多,一般来说,我们要养成一个好习惯,即遇到问题,要首先分析其中的关系,而图则是对关系建模的一个优秀的工具.从上面的例子可以看到,关系的种类是多种多样的,但只要可以起到关联的作用,那么就可以抽象为图.这一点,在日后图的推广中特别重要,有的问题不在于能否解答,而在于能否想到

#### 关系

关系是个比较抽象的概念,在简单的图问题上,节点之间的关系是明确的,就是显式的边而已,但对于更加复杂的问题,关系是比较难找的(比如上面的3个例子),有几个重要的点需要思考:

* **与对象的关系** 图问题究竟是先有关系E,还是先有对象V呢?个人看法是,关系在图中更为关键.对象是死的,关系的引入才带来的问题的可变.这也提醒我们,在处理问题时,不要一上来就追查什么是对象,而是要先分析有哪些可利用的关系,这关系的两头是同质还是异质(即是否同类对象),能否简化和转变,这样我们才能得到正确的对象概念,不至于陷于死胡同和人工构造的陷阱中
* **自关系** 对象能否和自己有关系呢?当然可以.状态机中经常会出现的自循环,就是自关系的一种.这种关系比较隐蔽,有时候会被我们主观无视掉
* **多关系** 对象之间的关系能否有多种呢?这里面涉及到2个方面,一个是对象间不同类型的关系,比如两个人既是父子又是朋友,这样的关系自然是可以存在多种的;二是对象间相同类型的关系,比如地图上两点的可达性,很有可能有多条不同的路径连接,这样的关系也是可以有多个的
* **对称** A和B有关系,能否说明B和A有关系呢?这涉及到对称问题,或者通俗来说,即方向.如果关系是没有方向的,比如朋友关系,那么这个关系是对称的,否则,像父子关系,就是单向的.当然,并不是说,单向关系中,对象间只能有一种关系,"多关系"和"对称"是不冲突的,比如暗恋关系,A暗恋B,B也暗恋A,暗恋肯定是单向的,但两对象间依然可以保持多关系
* **传递** A/B有关系,B/C有关系,那A/C有关系么?这个也是同关系的具体类型有关的,比如父子关系就不具有传递性,但class的继承关系就可以传递

(后面的讨论越发的类似数学中介绍的"自反/对称/传递"了,但搞计算机的,有个浅显的了解即可,其中涉及的数学理论,就可以忽略的)

关系是抽象图的最重要的特征,希望分析问题时多多注意.

下面在讨论问题时,就不适用抽象的关系/对象了,而是参照显式图的边/节点.这样比较容易理解些

### 无向图

图的一个经典分类即是按照边的方向性(对称),分成无向图和有向图.其中,无向图就是边没有方向的图,即边有对称性,通常使用(u, v)或(v, u)來表达连接u/v的同一条边.上面提到过的对称关系的图,都可以看作是无向图,比如朋友关系,互斥的job等.

#### 基本概念

* **自环(self loop)** 即上文提到的自关系在图中的表现
* **平行边(multiedge)** 即上文提到的多关系在图中的表现
* **路径(path)** 由u到v的路径,是指一系列边的集合,这些边首尾相接,且首边的起点是u,末边的终点是v.可以简单的想象成地图上两点之间的路线
* **环(cycle)** 一种特殊的路径,首尾相连,即u/v是同一个节点
* **简单图(simple)** 本系列处理的问题大部分属于简单图,即没有自环,没有平行边
* **简单路径** 不含有环的路经,称为简单路径,大部分时候考察的路径问题都属于这类(比如最短路径,很明显,去掉对应的环,只会减少路径长度而已)
* **简单环** 类似简单路径,不含有内环的环,称为简单环.复杂的环结构,可以想象称环环嵌套的形式,简单环中只有首尾的一个环而已
* **连通(connected)** u/v之间有路径,即表示u/v连通.连通的节点的集合,构成连通分量(connected component).无向图本身如果不是连通的,那么它肯定是一些连通分量的集合(单节点是最小的连通分量)
* **树(tree)** 即为无环连通图.无环表示内部没有环,连通表示内部节点相互之间有路径可达(连通)
* **森林(forest)** 不相交的树的集合
* **生成树(spanning tree)** 连通图的子图,包含了所有的节点和部分边,并且是树.其实可以看作将连通图中的某些环边删掉形成的树
* **生成森林(spanning forest)** 非连通图的各连通分量的生成树的集合
* **稀疏(sparse)**和**稠密(dense)** 在简单图中,节点数目确定時,边的数目变化的范围很大,没有边到两两节点都有边.一般来说,如果E=O(V)時,可以将图看作是稀疏的(可以想象,E=V-1且连通時,就是树了,非常稀疏),相反,当E=O(V^2)時,就称之为稠密(此时基本两两节点相连)

以上基本就是无向图的最基本常用的概念了,这些概念需要辅以应用才能有更加深入的了解.当然,这并不是全部,比如独立集覆盖或网络流之类的特殊概念,等到深入对应问题時再进行介绍

#### 图和树

树是一种特殊的图结构,spanning的过程,就是由连通图得到其树结构的过程.生成树不同于以前学习过的树,其每个节点都可以作为根节点的.一旦确定了根节点,整个树的方向和形状就基本确定的.

树极大的简化了图结构,有几个重要的性质:

1. V = E - 1 这个是树的基本性质,节点和边进行了对应
2. 无环
3. 连通

通常来讲,3个性质满足2个,剩下的那个就自然满足了(证明略去).当然,这些性质都很trival,我们并不要拿这些性质來证明什么,重要的一点是,当面对具有这些性质的图時,能意识到这是树,树的方向,树的父子关系,树的景点算法,就都可以用来解决问题了

比如最大独立集问题(简而言之,独立集是无边相连的节点的集合,最大独立集就是元素数量最多的独立集,后续会介绍),对于普通图来说,是NP问题,但在树形图上,就有非常快速的解法(利用了父子关系和分治法)

要能想起来

### 有向图

有向图的边是非对称的,有方向性,通常使用<u, v>来表示,u成为head,而v成为tail.这一性质,主要影响了以下几个概念:

* **可达(reachability)** 无向图中,u/v有路径就表示u/v是连通的,但在有向图中,路径是有方向的,所以存在u->v和v->u的路径,当探查到存在一条路经時,我们只能称之为可达,即存在u->v的路径,仅表示u可以到达v,或v在u的可达集合中
* **强连通(strong connected)** 双向可达,即u->v可达且v->u可达,同时存在二者相互的路径.强连通和无向图中的连通是类似的,都表达了节点之间相互有路径的情况(有向图需要双路经而已)
* **无环有向图(DAG)** 概念本身比较简单(就是没有环啊摔..),但DAG经常出现在问题中,比如一部分DP问题可以归结为DAG的遍历

其实就复杂度而言,并不能说有向图就一定比无向图复杂,二者在问题情景上应该是平级的,不同问题处理的复杂度各有不同而已

### 有权图

上面提到的2种图结构,其边都是无权值的(或权值都是1),还有一个大类,其中的边有着不同的权值(W = E -> R的一个函数),有3个比较重要的概念需要熟悉:

0. 负边 权值为负数的边
1. 负路经 权值和为负数的路经
2. 负环 权值和为负数的环

这三个都比较通俗易懂,之所以单独拿出来,是因为负权值的存在,对权值问题的处理有着很大的影响,这一点,在讨论最短路经時再仔细介绍

## 基本操作

介绍完图的一些基本概念之后,我们开始讲解一些图的基本操作,这些操作/算法都是图论问题处理的基础,作为其他复杂算法的基础步骤,需要我们好好理解和研究

### 图的表示

最常用的表示方式有2种,一种是矩阵,另一种是邻接表

#### 矩阵

矩阵的row和col代表的是图的节点,(i,j)表示的节点i和节点j之间的关系,true表示有边,false表示无边.这个关系可以是无向的(即(i,j)和(j,i)同时存在),也可以是有向的(非对称),同样也可以扩展來表示有权图,即(i,j)表示i/j间边的权值,当边不存在時,进行特殊标记(不存在的权值即可,比如INT_MAX/INT_MIN等)

这种表示方法的优点是:

1. 可以快速定位i/j之间的关系,直接判断(i,j)即可
2. 表达比较紧凑,整个图存储在二维数组中,在内存空间上是连续的

但这样的缺点也有:

1. 消耗的空间很大,不论图的边的情况,都要消耗O(V^2)的空间,尤其是对于稀疏图而言,很多位置都是空的(不过对于稠密图来讲,应该是可以接受的)
2. 遍历某节点的相邻节点的复杂度是O(V),但往往对于某些图来说,根本就没有这么多边.尤其是考虑遍历所有节点的相邻节点,总复杂度是O(V^2),但一般只有O(E)的访问是可能的,其他都也大多为空

由于常见问题处理的图大部分都是稀疏的,所以在讨论优缺点時,总要考虑这一点,回顾上面的基本概念,稀疏图中E=O(V),所以但凡涉及到O(V^2)的操作,大多访问到了根本不存在的边,是无效的访问

缺点2中提到一个有趣的操作,即**遍历相邻节点**,这个看似局部的操作,实际上是图论算法的最基本的操作,绝大部分的算法都是在这个的基础之上完成的.这大概是**整体与局部**关系的一个有力的证明吧

在C++中,通常使用双重的vector來代替原始的二维数组表示,节点采用整数來标记

    vector<vector<double> > g(V);
    for (int i = 0; i < V; i++) {
        g[i].resize(V);
    }

    // 取u/v关系
    double w = g[u][v];

#### 邻接表

邻接表的表示方法是将每个节点和其相邻节点绑定在一起,这样,当遍历相邻节点時,我们只需要遍历真正存在的边即可.如果进行所有节点的相邻节点遍历,也只是需要O(E)=O(V)次访问即可(同O(V^2)相比有很大进步)

但这样做的代价就是判断节点关系的复杂度不再是常数了,因为只能通过遍历相邻节点來判断二者是否相邻了,对于有向图,甚至需要分别遍历2个节点各自的相邻节点.同矩阵表示法相比,优缺点对调了

不过这并不是什么太大的问题,因为很少有算法需要明确的得到任意节点间的关系(因为是稀疏图,没有比较),所以在这样的前提下,邻接表有很大的优势.我们在讲解图论時,除非特别声明,一律使用邻接表來表示图结构

下面即是我们封装好的图结构,部分参考了[algorithms 4th][algorithms4]的代码

    struct GraphNode {
        int v;
        double w;
        GraphNode(int tv = -1, double tw = 0.0) : v(tv), w(tw) {
        }
    };

    class Graph {
    public:
        Graph(int V, bool is_directed = false)
            : m_vn(V), m_en(0), m_is_directed(is_directed),
              m_ns(V) {
        }
        // 默认的复制构造函数/operator=即可

        int vn() const {
            return m_vn;
        }
        int en() const {
            return m_en;
        }
        bool is_directed() const {
            return m_is_directed;
        }

        const vector<GraphNode>& ns(int u) const {
            assert(u >= 0 && u < m_vn);

            return m_ns[u];
        }

        void add(int u, int v, double w) {
            assert(u >= 0 && u < m_vn);
            assert(v >= 0 && v < m_vn);

            m_ns[u].push_back(GraphNode(v, w));
            m_en++;
            if (!m_is_directed) {
                m_ns[v].push_back(GraphNode(u, w));
                m_en++;
            }
        }

        Graph reverse() const {
            if (!m_is_directed) {
                return *this;
            }

            Graph g(m_vn, true);
            for (int i = 0; i < m_vn; i++) {
                int n = m_ns[i].size();
                for (int j = 0; j < n; j++) {
                    g.add(i, ns[i][j].v, ns[i][j].w);
                }
            }

            return g;
        }

    private:
        int m_vn;
        int m_en;
        bool m_is_directed;
        vector<vector<GraphNode> > m_ns;
    };

#### 邻接hash

(说好的不是两种么???)

如果我们真的想判断节点关系,但又不想付出邻接矩阵的空间复杂度呢?好吧,这个要求虽然有些过分,但前人们也研究出来了相应的对策,即邻接hash表示

正如名字所言,我们邻接的不再是vector(话说知道vector的push_back复杂度为O(1)么?),而是hash表,这样在查询节点关系時,可以以O(1)來查询关系是否存在,有向图最多也就2次查询.这样的代价就是空间稍微增大了一点,但其复杂度依然是O(E)的(常用hash的最大空间和实际数量是一个级别的)

但就像矩阵表示時所言,判断节点关系在图论问题中并没有想象当中那么重要(当然,除非你就要二节点的关系),所以还是优先选择简单的邻接表方式



[jeff]: http://www.cs.uiuc.edu/~jeffe/teaching/algorithms/
[algorithms4]: http://book.douban.com/subject/10432347/
[adm]: http://book.douban.com/subject/3072383/