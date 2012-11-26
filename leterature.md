---
layout: page
title: 文青
---

有的时候我也文青一把

写的不好,就当作是青春的纪念吧

{% for post in site.categories.leterature %}
*   [{{ post.title }}]({{ post.url }})
    {{ post.description }}
{% endfor %}

{{ site.copyright }}
