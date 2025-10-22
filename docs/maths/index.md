---
layout: docs-category
title: 数学文档
category: maths
description: 数学基础理论、应用数学、算法与数据结构等相关内容的深入解析和实践指南。
permalink: /docs/maths/
---

数学相关的技术文档，涵盖基础数学理论、应用数学、算法与数据结构等关键知识点和实践案例。

## 📖 文档概览

数学是科学和技术的基础，无论是在计算机科学、人工智能还是工程领域，都有着广泛的应用。这些文档将帮助你理解数学在实际应用中的核心原理和方法。

## 📚 文章列表

<div class="posts-list">
  {% assign sorted_pages = site.pages | where: "category", "maths" | sort: "date" | reverse %}
  {% for page in sorted_pages %}
    <div class="post-item">
      <h3><a href="{{ page.url | relative_url }}">{{ page.title }}</a></h3>
      <p>{{ page.excerpt }}</p>
      <div class="post-meta">
        <time datetime="{{ page.date | date_to_xmlschema }}">{{ page.date | date: "%Y-%m-%d" }}</time>
      </div>
    </div>
  {% endfor %}
</div>