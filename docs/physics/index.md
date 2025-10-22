---
layout: docs-category
title: 物理学
category: physics
description: 探索物理学的奥秘，从经典力学到量子物理
permalink: /docs/physics/
---

<div class="category-intro">
  <h1>⚛️ 物理学文档</h1>
  
  <p>
    物理学是探索自然界基本规律的科学，从宏观的宇宙到微观的粒子，从经典力学到现代量子理论。
    本系列文档旨在分享我在物理学学习和研究中的理解和思考。
  </p>
  
  <div class="category-highlights">
    <h2>📘 文档亮点</h2>
    <ul>
      <li><strong>系统性强</strong> - 从基础概念到前沿理论的完整知识体系</li>
      <li><strong>理论结合</strong> - 理论推导与实际应用相结合</li>
      <li><strong>深入浅出</strong> - 通过图表和实例帮助理解复杂概念</li>
      <li><strong>跨学科</strong> - 展示物理学在其他领域的应用</li>
    </ul>
  </div>
  
  <div class="tech-stack">
    <h2>📚 涉及领域</h2>
    <p>
      这些文档涵盖了以下核心物理领域：
    </p>
    <ul>
      <li><strong>经典力学</strong> - 牛顿力学、拉格朗日力学、哈密顿力学</li>
      <li><strong>电磁学</strong> - 麦克斯韦方程组、电磁波理论</li>
      <li><strong>热力学</strong> - 热力学定律、统计物理基础</li>
      <li><strong>量子力学</strong> - 波函数、薛定谔方程、量子纠缠</li>
      <li><strong>相对论</strong> - 狭义相对论、广义相对论</li>
      <li><strong>计算物理</strong> - 数值方法在物理问题中的应用</li>
    </ul>
  </div>
</div>

物理相关的技术文档，涵盖基础物理理论、应用物理、算法与物理结构等关键知识点和实践案例。

## 📖 文档概览

物理是科学和技术的基础，无论是在计算机科学、人工智能还是工程领域，都有着广泛的应用。这些文档将帮助你理解数学在实际应用中的核心原理和方法。

## 📚 文章列表

<div class="posts-list">
  {% assign sorted_pages = site.pages | where: "category", "physics" | sort: "date" | reverse %}
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

<style>
.category-intro {
  margin-bottom: 2rem;
  padding: 1.5rem;
  background-color: #f6f8fa;
  border-radius: 8px;
  border-left: 4px solid #3498db;
}

.category-intro h1 {
  margin-top: 0;
  color: #24292e;
}

.category-intro p {
  line-height: 1.6;
  color: #586069;
}

.category-highlights,
.tech-stack {
  margin: 1.5rem 0;
}

.category-highlights h2,
.tech-stack h2 {
  color: #24292e;
  border-bottom: 1px solid #e1e4e8;
  padding-bottom: 0.5rem;
}

.category-highlights ul,
.tech-stack ul {
  list-style-type: none;
  padding-left: 0;
}

.category-highlights li,
.tech-stack li {
  padding: 0.5rem 0;
  border-bottom: 1px solid #eaecef;
}

.category-highlights li:last-child,
.tech-stack li:last-child {
  border-bottom: none;
}

.category-highlights li::before,
.tech-stack li::before {
  content: "✓";
  color: #28a745;
  font-weight: bold;
  display: inline-block;
  width: 1.5em;
  margin-right: 0.5em;
}

.tech-stack li::before {
  content: "📚";
}
</style>