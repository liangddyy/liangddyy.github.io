---
title: tags
permalink: /tags/
---

# TAGS

{% for tag in site.tags %}
<!-- 添加ID 方便跳转 -->
<h2 id="{{ tag | first }}">{{ tag | first }}</h2>
<ul class="c-archives__list">
    {% for post in tag.last %}
        <li class="c-archives__item">
          {{ post.date | date: "%b %-d, %Y" }}: <a href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>
        </li>
    {% endfor %}
</ul>
{% endfor %}


