---
layout: docs-category
title: Linux
category: linux
description: Linux系统管理与运维实践，涵盖系统配置、性能优化、安全加固等关键知识点
permalink: /docs/linux/
---

<div class="category-intro">
  <h1>🐧 Linux 文档</h1>
  
  <p>
    Linux作为开源操作系统的核心，是系统管理员和开发人员必须掌握的重要技能。
    本系列文档旨在分享我在Linux系统管理与运维方面的实践经验。
  </p>
  
  <div class="category-highlights">
    <h2>📘 文档亮点</h2>
    <ul>
      <li><strong>实用性强</strong> - 提供可直接应用的命令和配置示例</li>
      <li><strong>系统全面</strong> - 从基础操作到高级系统管理</li>
      <li><strong>问题导向</strong> - 针对常见问题提供解决方案</li>
      <li><strong>版本兼容</strong> - 涵盖多个主流Linux发行版</li>
    </ul>
  </div>
  
  <div class="tech-stack">
    <h2>📚 涉及领域</h2>
    <p>
      这些文档涵盖了以下核心Linux领域：
    </p>
    <ul>
      <li><strong>系统管理</strong> - 用户管理、文件系统、进程管理</li>
      <li><strong>网络配置</strong> - 网络接口配置、防火墙、DNS</li>
      <li><strong>性能优化</strong> - 系统调优、资源监控、瓶颈分析</li>
      <li><strong>安全管理</strong> - 权限控制、安全加固、审计日志</li>
      <li><strong>服务管理</strong> - systemd、服务配置、自动启动</li>
      <li><strong>Shell脚本</strong> - Bash脚本编程、自动化任务</li>
    </ul>
  </div>
</div>

## 📖 文档概览

Linux作为最流行的开源操作系统之一，广泛应用于服务器、云计算和嵌入式设备等领域。掌握Linux系统管理和运维技能对于技术人员至关重要。

## 📚 文章列表

<div class="posts-list">
  {% assign sorted_pages = site.pages | where: "category", "linux" | sort: "date" | reverse %}
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