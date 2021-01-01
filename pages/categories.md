---
title: categories
permalink: /categories/
---

# 类别索引

<!-- {{ category | last | size }} 类别数量 -->
{% assign sorted_categories = site.categories | sort %}
{% for category in sorted_categories %}
<!-- 添加ID 方便跳转 -->
<h2 id="{{ category | first }}">{{ category | first }}</h2>
<ul class="c-archives__list">
    {% for post in category.last %}
        <li class="c-archives__item">
          {{ post.date | date: "%b %-d, %Y" }}: <a href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>
        </li>
    {% endfor %}
</ul>
{% endfor %}


# some tags
<div>
{% for tag in site.tags %}
    {% for post in tag.last %}
      <span class="badge badge-success"><a style="cursor:pointer; color:white" href="/tags/#{{ tag | first }}">{{ tag | first }}</a></span>
    {% endfor %}
{% endfor %}
</div>




