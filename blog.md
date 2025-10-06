---
title: Blog
layout: default
---

<ul class="blog-list">
{% assign items = site.articles | sort: "date" | reverse %}
{% for a in items %}
  <li class="blog-card">
    <h3><a href="{{ a.url | relative_url }}">{{ a.title }}</a></h3>
    <div class="excerpt" style="display:block">
        {{ a.excerpt }}
        <style>
            .excerpt p:last-of-type::after{content:" \2026";}
            .excerpt p:last-of-type{display:inline;}
        </style>
    </div>
  </li>
{% endfor %}
</ul>