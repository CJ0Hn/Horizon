---
layout: default
title: Home
---

# Horizon Daily Summaries

Welcome to the daily digest of [Horizon](https://github.com/thysrael/horizon).

## Latest Summaries

<ul>
  {% for post in site.posts limit:20 %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
      <small style="color: #666;">- {{ post.date | date: "%Y-%m-%d" }}</small>
    </li>
  {% endfor %}
</ul>
