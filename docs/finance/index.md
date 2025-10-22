---
layout: docs-category
title: 财经
category: finance
description: 探索财经领域的知识与实践，包括投资理财、经济分析、财务管理等
permalink: /docs/finance/
---

## 📖 文档概览

财经知识在现代社会中扮演着重要角色，无论是在个人理财还是企业经营中都有着广泛的应用。这些文档将帮助您理解财经领域的核心原理和实践方法。

## 📚 文章列表

<div class="posts-list">
  {% assign sorted_pages = site.pages | where: "category", "finance" | sort: "date" | reverse %}
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