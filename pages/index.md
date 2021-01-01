---
layout: page
title: liangddyy's blog
permalink: /
---

# Welcome to my blog

{% assign all_posts = "" | split: "" %}
{% for post in site.posts %}
    {% assign all_posts = all_posts | push: post %}
{% endfor %}
{% assign all_posts = all_posts | sort: date | reverse %}

{% for post in all_posts limit:10 %}
   <div class="post-preview">
   <h2><a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a></h2>
   <span class="post-date">{{ post.date | date: "%B %d, %Y" }}</span><br>
   {% if post.badges %}{% for badge in post.badges %}<span class="badge badge-{{ badge.type }}">{{ badge.tag }}</span>{% endfor %}{% endif %}
   {{ post.excerpt | strip_html | strip }}
      &nbsp;&nbsp;<a text-align:right href="{{ site.baseurl }}{{ post.url }}">read more</a>
   </div>
   <hr>
{% endfor %}

<a href="{{ site.baseurl }}/archive/">更多历史文章</a>