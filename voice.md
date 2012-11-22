---
layout: page
title: 吐槽
---

## 杂烩之地

恭喜你来到了我的吐槽专场,愿意忍受我的唠唠叨叨.

当然,其实也没有啦,只是我不知道怎么分类,才这样分出来的.

{% if site.categories.voice.length == 0 %}
不过好像还没有值得吐嘈的诶,真是神奇的一天.
{% else%}
{% for post in site.categories.voice %}
*   [{{ post.title }}]({{ post.url }})
    {{ post.description }}
{% endfor %}
{% endif %}

{{ site.copyright }}
