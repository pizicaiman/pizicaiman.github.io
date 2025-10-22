---
layout: docs-category
title: 大语言模型文档
category: llm
description: 关于大语言模型应用、Prompt Engineering、向量数据库等 AI 技术的深入分析和实践指南。
permalink: /docs/llm/
---

<div class="category-intro">
  <h1>🤖 大语言模型技术文档</h1>
  
  <p>
    大语言模型（LLM）正以前所未有的速度改变着我们与技术交互的方式。
    本系列文档旨在分享我在 LLM 应用开发、提示工程和 AI 架构设计方面的实践经验和深度思考。
  </p>
  
  <div class="category-highlights">
    <h2>📘 文档亮点</h2>
    <ul>
      <li><strong>应用导向</strong> - 专注于 LLM 在实际业务场景中的应用实践</li>
      <li><strong>技术深度</strong> - 深入探讨提示工程、微调技术等核心技术</li>
      <li><strong>架构设计</strong> - 提供可扩展的 LLM 应用架构设计方案</li>
      <li><strong>案例丰富</strong> - 通过真实案例展示技术实现过程</li>
    </ul>
  </div>
  
  <div class="tech-stack">
    <h2>🔧 涉及技术栈</h2>
    <p>
      这些文档涵盖了以下核心技术领域：
    </p>
    <ul>
      <li><strong>主流模型</strong> - GPT、Claude、Gemini 等大语言模型的应用</li>
      <li><strong>提示工程</strong> - Chain-of-Thought、Few-shot 等提示技术</li>
      <li><strong>向量数据库</strong> - Pinecone、Weaviate、FAISS 等相似性搜索工具</li>
      <li><strong>应用框架</strong> - LangChain、LlamaIndex 等 LLM 应用开发框架</li>
      <li><strong>部署方案</strong> - 模型部署、API 网关、性能优化等</li>
      <li><strong>评估方法</strong> - 模型效果评估、基准测试等技术</li>
    </ul>
  </div>
</div>

大语言模型相关的技术文档，涵盖模型应用架构、提示词工程、向量检索等关键技术和实践案例。

## 📖 文档概览

大语言模型（LLM）正在引领新一轮的AI革命。这些文档将帮助你理解LLM的核心原理、应用架构和最佳实践。

## 📚 文章列表

<div class="posts-list">
  {% assign sorted_pages = site.pages | where: "category", "llm" | sort: "date" | reverse %}
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
  border-left: 4px solid #9b59b6;
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
  content: "⚙️";
}
</style>