---
layout: post
title: 破解 NationalReview 的订阅限制
category: code
tag: python calibre
---

## 缘由

最近升级了最新的 kindle paperwhite 5,决定把之前落下的各种杂志都传到 kindle 里看.主要的工具还是 calibre.

calibre 里现成的资源虽多,但还是缺一些.比较重要的保守派杂志 [National Review][NR] 就没有,让我很是头疼

[National Review][NR] 的网站是可以免费访问的,calibre 里也有对应的 recipe 来获取.但它的杂志和存档是需要[付费订阅][Subscribe]的,大概 100 刀一年.如果没有付费,一个月貌似只能免费看2篇杂志文章

虽说我并不差钱,而且在 [Amazon][] 上订阅 kindle 版本也更便宜 (24 刀一年,只看杂志),但我还是很好奇它的限制机制,故而在 calibre 的帮助下,决定试探一番

## 分析

根据 [Calibre官方教程][Add News] 的说法,我们其实是通过一段 称为 recipe 的 python 代码来下载不同新闻源的文章的.如果 National Review 的杂志是结构化可分析的,那就大概率是可以通过此方法下载的

杂志其实分2类页面,一类是 [杂志首页][Magazine Homepage],另一类是 [文章页][Article Page]

## 杂志首页

杂志首页是可以免费访问的,没有任何限制.那么我们就可以通过下载解析首页,来制作 recipe 需要的 `index`,即实现如下的[接口][parse_index]

该接口如要是用来生成文章的索引,该索引会被后面的流程,用于提供各文章的的地址,从而完成文章的下载.所以主要是需要获得所有文章的标题/URL

        def parse_index(self):
        '''
        This method should be implemented in recipes that parse a website
        instead of feeds to generate a list of articles. Typical uses are for
        news sources that have a "Print Edition" webpage that lists all the
        articles in the current print edition. If this function is implemented,
        it will be used in preference to :meth:`BasicNewsRecipe.parse_feeds`.

        It must return a list. Each element of the list must be a 2-element tuple
        of the form ``('feed title', list of articles)``.

        Each list of articles must contain dictionaries of the form::

            {
            'title'       : article title,
            'url'         : URL of print version,
            'date'        : The publication date of the article as a string,
            'description' : A summary of the article
            'content'     : The full article (can be an empty string). Obsolete
                            do not use, instead save the content to a temporary
                            file and pass a file:///path/to/temp/file.html as
                            the URL.
            }

        For an example, see the recipe for downloading `The Atlantic`.
        In addition, you can add 'author' for the author of the article.

        If you want to abort processing for some reason and have
        calibre show the user a simple message instead of an error, call
        :meth:`abort_recipe_processing`.
        '''
        raise NotImplementedError()

通过查看下载到的[首页][Magazine Homepage],发现原来 National Review 并没有使用静态的网页,而是将网页数据保存成 `js` 的数据,然后通过 `js` 来动态生成该网页

这样的话,就十分方便了.我们只要得到该数据,并直接解析成 calibre 需要的 index 即可,省下了大把的网页解析时间

## 文章页

本来我还以为[文章页][Article Page]肯定别有用心,但一看,居然和首页一样,也是利用 `js` 数据来动态生成的.如果用户没有订阅,则也是通过 `js` 来阻止用户看到全文.我们只需要简单的从数据里把全文摘出来就可以,仍然比解析网页更简单

文章页的获取是通过实现另一个[接口][preprocess_html].Calibre 通过上面的索引得到文章的 URL, 然后下载该 URL 内容,将其转换成 `BeautifulSoup` 结构,再经过该接口处理

这里,我们需要返回新的 `soup`, 因为原来的网页完全没有,我们直接从 `js` 数据里,生成一个简单的文章页并返回对应的 `soup`即可

        def preprocess_html(self, soup):
        '''
        This method is called with the source of each downloaded :term:`HTML` file, before
        it is parsed for links and images. It is called after the cleanup as
        specified by remove_tags etc.
        It can be used to do arbitrarily powerful pre-processing on the :term:`HTML`.
        It should return `soup` after processing it.

        `soup`: A `BeautifulSoup <https://www.crummy.com/software/BeautifulSoup/bs4/doc/>`__
        instance containing the downloaded :term:`HTML`.
        '''
        return soup

## 整合 Recipe

实现完上述2个接口,Recipe基本就完成了,就可以加到 Calibre的下载序列里,每2周一次的下载 National Review 的最新杂志内容

鉴于版权原因,我在这里就不提供代码了.感兴趣的话,完全可以从上面的分析写出来,非常的简单

## 问题

在写的时候,需要2个小问题

### SSL 过期

一开始我的 Calibre 的版本过低,很多 API 没法使用,所以直接升级到了最新的 6.11.结果一下子就出现了 SSL 失败的错误.

查来查去都没查头绪,我甚至都想把 SSL 验证关掉.但这样肯定是危险的.后来发现,原来是 National Review 网站证书的中间层的证书过期了,我根据这个 [SO帖子][SO] 安装了最新的,就解决了这个问题

### Tag没法删除

recipe 有一个预定义结构 `remove_tags`,我本来是用它去掉新生成的文章网页中的没意义的标签.结果发现,这个 `remove_tags` 居然是在 `preprocess_html` 之前操作的.所以我只能在生成网页的时候,手动去除这些标签了

    #: List of tags to be removed. Specified tags are removed from downloaded HTML.
    #: A tag is specified as a dictionary of the form::
    #:
    #:    {
    #:     name      : 'tag name',   #e.g. 'div'
    #:     attrs     : a dictionary, #e.g. {'class': 'advertisment'}
    #:    }
    #:
    #: All keys are optional. For a full explanation of the search criteria, see
    #: `Beautiful Soup <https://www.crummy.com/software/BeautifulSoup/bs4/doc/#searching-the-tree>`__
    #: A common example::
    #:
    #:   remove_tags = [dict(name='div', class_='advert')]
    #:
    #: This will remove all `<div class="advert">` tags and all
    #: their children from the downloaded :term:`HTML`.
    remove_tags           = []

## 总结

总的来看, National Review 的订阅限制实在过于简陋,完全没法阻止非付费用户的下载.虽然使用了动态生成网页的小技巧,但还是很容易被发现破绽

Calibre 的 recipe 不是特别直观,只能用调试的办法摸索各个 API 的使用.不过这类代码都很固定,一旦写完,应该是可以使用很久了

我的初版 recipe 还有很多不足,比如图片就没有设置好,还有文章样式有待打磨.但对我这种自力更生的阅读者,已经很满意了

接下来,我就可以照猫画虎的抓别的网站了

[NR]: https://www.nationalreview.com/
[Subscribe]: https://www.nationalreview.com/subscribe-50-off/
[Amazon]: https://www.amazon.com/National-Review/dp/B004QGYDWA/ref=tmm_kin_swatch_0
[Add News]: https://manual.calibre-ebook.com/news.html
[Magazine Homepage]: https://www.nationalreview.com/magazine/
[Article Page]:　https://www.nationalreview.com/magazine/2023/01/23/puerto-rico-libre/
[parse_index]: https://manual.calibre-ebook.com/_modules/calibre/web/feeds/news.html#BasicNewsRecipe.parse_index
[preprocess_html]: https://manual.calibre-ebook.com/_modules/calibre/web/feeds/news.html#BasicNewsRecipe.preprocess_html
[SO]: https://stackoverflow.com/questions/34812787/python-ssl-requests-and-lets-encrypt-certs/51332150#51332150