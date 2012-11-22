---
layout: page
title: 算法挑战
---

非常羡慕那些能在高中大学时代征战NOI或ACM的牛人们

这方面我比较弱,还需要多多积累,希望能慢慢成长

{% for post in site.categories.algorithm %}
*   [{{ post.title }}]({{ post.url }})
    {{ post.description }}
{% endfor %}

{{ site.copyright }}
