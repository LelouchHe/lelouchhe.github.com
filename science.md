---
layout: page
title: 科学吧
---

我很喜欢数学和物理,虽然我不会天真的以为我有那方面的天赋,但我觉得能在人生仅有的岁月里,**浅浅**的了解宇宙和上帝背后的故事,一定是非常赞的

以下是我准备的书单 **虽千万人吾往矣**

## 数学

* [什么是数学](http://book.douban.com/subject/1320282/)
* [微积分与数学分析引论](http://book.douban.com/subject/1281343/)
* [数学分析八讲](http://book.douban.com/subject/4825571/)
* [微积分学教程](http://book.douban.com/subject/1707158/)
* [线性代数及应用](http://book.douban.com/subject/1425950/)
* [概率论基础教程](http://book.douban.com/subject/2066458/)
* [具体数学](http://book.douban.com/subject/1231910/)

## 物理
* [费恩曼物理学教程](http://book.douban.com/subject/1437852/)
* [理论物理学教程](http://book.douban.com/subject/2059252/)

努力吧,少年

{% for post in site.categories.science %}
*   [{{ post.title }}]({{ post.url }})
    {{ post.description }}
{% endfor %}

{{ site.copyright }}
