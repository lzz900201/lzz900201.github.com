---
layout: page
title: Liu的博客
tagline: 分享学习技术的经验
---
{% include JB/setup %}

## posts
-----

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
