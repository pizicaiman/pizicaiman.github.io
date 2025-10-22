---
layout: docs-category
title: 财经
category: finance
description: 探索财经领域的知识与实践，包括投资理财、经济分析、财务管理等
permalink: /docs/finance/
---

<div class="category-intro">
  <h1>💰 财经文档</h1>
  
  <p>
    财经知识在现代社会中扮演着重要角色，无论是在个人理财还是企业经营中都有着广泛的应用。
    本系列文档旨在分享我在财经领域的学习心得和实践经验。
  </p>
  
  <div class="category-highlights">
    <h2>📘 文档亮点</h2>
    <ul>
      <li><strong>实用性</strong> - 注重实际应用，提供可操作的建议</li>
      <li><strong>系统性</strong> - 从基础概念到高级策略的完整知识体系</li>
      <li><strong>案例丰富</strong> - 通过真实案例分析帮助理解</li>
      <li><strong>与时俱进</strong> - 紧跟财经领域最新发展趋势</li>
    </ul>
  </div>
  
  <div class="tech-stack">
    <h2>📚 涉及领域</h2>
    <p>
      这些文档涵盖了以下核心财经领域：
    </p>
    <ul>
      <li><strong>投资理财</strong> - 股票、债券、基金等投资工具分析</li>
      <li><strong>经济分析</strong> - 宏观经济、微观经济理论与实践</li>
      <li><strong>财务管理</strong> - 企业财务分析、预算管理、风险控制</li>
      <li><strong>金融市场</strong> - 金融市场运作机制、交易策略</li>
      <li><strong>金融科技</strong> - 区块链在金融领域的应用、数字货币</li>
      <li><strong>行为金融</strong> - 投资者心理、市场情绪分析</li>
    </ul>
  </div>
</div>

## 📖 文档概览

财经知识在现代社会中扮演着重要角色，无论是在个人理财还是企业经营中都起着重要作用。这些文档将帮助您理解财经领域的核心原理和实践方法。

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

<style>
.category-intro {
  margin-bottom: 2rem;
  padding: 1.5rem;
  background-color: #f6f8fa;
  border-radius: 8px;
  border-left: 4px solid #27ae60;
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