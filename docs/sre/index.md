---
layout: docs-category
title: SRE
category: sre
description: 站点可靠性工程（SRE）实践，涵盖可靠性原则、监控告警、容量规划、故障处理等关键知识点
permalink: /docs/sre/
---

<div class="category-intro">
  <h1>🎯 SRE 文档</h1>
  
  <p>
    站点可靠性工程（SRE）是确保大规模系统稳定运行的关键实践方法。
    本系列文档旨在分享我在SRE领域的学习心得和实践经验。
  </p>
  
  <div class="category-highlights">
    <h2>📘 文档亮点</h2>
    <ul>
      <li><strong>实践导向</strong> - 基于真实生产环境的SRE实践总结</li>
      <li><strong>系统全面</strong> - 涵盖从基础概念到高级实践的完整知识体系</li>
      <li><strong>案例丰富</strong> - 通过真实案例分析SRE工作方法</li>
      <li><strong>工具介绍</strong> - 介绍SRE常用的工具和技术栈</li>
    </ul>
  </div>
  
  <div class="tech-stack">
    <h2>📚 涉及领域</h2>
    <p>
      这些文档涵盖了以下核心SRE领域：
    </p>
    <ul>
      <li><strong>可靠性原则</strong> - SLI/SLO/SLA定义与管理</li>
      <li><strong>监控告警</strong> - 监控系统设计、告警策略优化</li>
      <li><strong>容量规划</strong> - 性能测试、容量评估与扩展策略</li>
      <li><strong>故障处理</strong> - 故障响应流程、事后分析与改进</li>
      <li><strong>自动化运维</strong> - 部署自动化、配置管理、自愈系统</li>
      <li><strong>混沌工程</strong> - 系统韧性测试与验证</li>
    </ul>
  </div>
</div>

## 📖 文档概览

站点可靠性工程（SRE）是Google提出的一套运维理念和方法论，它将软件工程的方法应用于运维问题，以创建可扩展和高可用的系统。

## 📚 文章列表

<div class="posts-list">
  {% assign sorted_pages = site.pages | where: "category", "sre" | sort: "date" | reverse %}
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
  border-left: 4px solid #e67e22;
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