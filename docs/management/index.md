---
layout: docs-category
title: 管理
category: management
description: 管理学的理论与实践，涵盖项目管理、团队管理、组织行为等关键知识点
permalink: /docs/management/
---

## 📖 文档概览

管理是组织成功的关键因素，无论是在企业运营还是个人发展中都起着重要作用。这些文档将帮助您掌握管理学的核心原理和实践技巧。

## 📚 文章列表

<div class="posts-list">
  {% assign sorted_pages = site.pages | where: "category", "management" | sort: "date" | reverse %}
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