---
layout: default
title: Blog
description: 개발 관련 글들을 정리한 블로그입니다.
---

## 최근 포스트

{% for post in site.posts %}
  <article class="post-preview">
    <h3><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h3>
    <p class="post-meta">{{ post.date | date: "%Y년 %m월 %d일" }}</p>
    {% if post.description %}
      <p>{{ post.description }}</p>
    {% else %}
      <p>{{ post.excerpt | strip_html | truncatewords: 30 }}</p>
    {% endif %}
  </article>
  <hr>
{% endfor %}