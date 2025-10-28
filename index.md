---
layout: default
title: Home
---

## My Blog Posts

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
      <span style="color: #666;">- {{ post.date | date: "%B %d, %Y" }}</span>
    </li>
  {% endfor %}
</ul>
