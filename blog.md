---
title: Blog
layout: default
---

<ul class="blog-list">
{% assign items = site.articles | sort: "date" | reverse %}
{% for a in items %}
  <li class="blog-card">
    <h3><a href="{{ a.url | relative_url }}">{{ a.title }}</a></h3>
    <div class="excerpt">{{ a.excerpt | strip }}&nbsp;&hellip;</div>
  </li>
{% endfor %}
</ul>