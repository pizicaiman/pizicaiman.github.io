---
layout: docs-category
title: 哲学文化
category: philosophy
description: 探索哲学思想与文化内涵，涵盖东西方哲学、文化比较、思辨方法等关键知识点
permalink: /docs/philosophy/
---

## 📖 文档概览

哲学与文化是人类智慧的结晶，无论是在个人思维提升还是社会文明发展中都起着重要作用。这些文档将帮助您理解哲学文化的深层内涵和思辨方法。

## 📚 文章列表

<div class="posts-list">
  {% assign sorted_pages = site.pages | where: "category", "philosophy" | sort: "date" | reverse %}
  {% for page in sorted_pages %}
    <div class="post-item">
      <h3><a href="{{ page.url | relative_url }}">{{ page.title }}</a></h3>
      <p class="post-meta">
        <span class="post-date">{{ page.date | date: "%Y-%m-%d" }}</span>
        {% if page.author %}
          <span class="post-author">by {{ page.author }}</span>
        {% endif %}
      </p>
      <p class="post-excerpt">{{ page.excerpt | strip_html | truncate: 150 }}</p>
    </div>
  {% endfor %}
</div>