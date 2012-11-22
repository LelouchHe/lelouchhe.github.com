---
layout: page
title: 源码研究
---

有些开源的项目真的非常棒,要是有时间把这些全都搞懂就赞透了

不过现在工作了,时间紧的很,所以我就列出来目前我在看的几个项目,争取抽时间把这些弄清楚

* [zlog](http://github.com/HardySimpson/zlog)
* [lua](http://www.lua.org/manual/5.2/manual.html)
* [unix v6 / xv6](http://pdos.csail.mit.edu/6.828/2012/xv6.html)

{% for post in site.categories.code %}
*   [{{ post.title }}]({{ post.url }})
    {{ post.description }}
{% endfor %}

{{ site.copyright }}
