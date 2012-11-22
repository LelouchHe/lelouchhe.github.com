---
layout: page
title: 吐槽
---

## 杂烩之地

恭喜你来到了我的吐槽专场,愿意忍受我的唠唠叨叨.

当然,其实也没有啦,只是我不知道怎么分类,才这样分出来的.

{% for post in site.categories.voice %}
*   [{{ post.title }}]({{ post.url }})
    {{ post.description }}
{% endfor %}

{{ site.copyright }}
