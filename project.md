---
layout: page
title: 我的开源
---

虽然我还是土人,但是也有一两个想分享的东东

可能是代码,可能是机械,或者其他乱七八糟的什么

{% if site.categories.project.length == 0 %}
我还没有什么项目值得拿来分享,不过你可以来看看[我的github](http://github.com/LelouchHe)
{% else%}
{% for post in site.categories.project %}
*   [{{ post.title }}]({{ post.url }})
    {{ post.description }}
{% endfor %}
{% endif %}

{{ site.copyright }}