---
layout: docs-category
title: 哲学文化
category: philosophy
description: 探索哲学思想与文化内涵，涵盖东西方哲学、文化比较、思辨方法等关键知识点
permalink: /docs/philosophy/
---

<div class="category-intro">
  <h1>🧠 哲学文化文档</h1>
  
  <p>
    哲学与文化是人类智慧的结晶，为我们理解世界和自身提供了深刻的视角。
    本系列文档旨在分享我在哲学和文化领域的学习心得与思辨过程。
  </p>
  
  <div class="category-highlights">
    <h2>📘 文档亮点</h2>
    <ul>
      <li><strong>思辨导向</strong> - 注重思维训练和批判性思考</li>
      <li><strong>跨文化</strong> - 融合东西方哲学思想与文化传统</li>
      <li><strong>古今结合</strong> - 古典哲学思想与现代问题相结合</li>
      <li><strong>实践应用</strong> - 将哲学思维应用于日常生活和工作</li>
    </ul>
  </div>
  
  <div class="tech-stack">
    <h2>📚 涉及领域</h2>
    <p>
      这些文档涵盖了以下核心哲学文化领域：
    </p>
    <ul>
      <li><strong>东西方哲学</strong> - 儒释道、希腊哲学、德国古典哲学等</li>
      <li><strong>文化比较</strong> - 不同文化背景下的思维模式与价值观念</li>
      <li><strong>思辨方法</strong> - 逻辑推理、批判性思维、辩证法</li>
      <li><strong>伦理学</strong> - 道德哲学、应用伦理、价值判断</li>
      <li><strong>美学</strong> - 艺术哲学、审美经验、美的本质</li>
      <li><strong>认知科学</strong> - 心灵哲学、意识研究、认知过程</li>
    </ul>
  </div>
</div>

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

<style>
.category-intro {
  margin-bottom: 2rem;
  padding: 1.5rem;
  background-color: #f6f8fa;
  border-radius: 8px;
  border-left: 4px solid #34495e;
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