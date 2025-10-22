---
layout: docs-category
title: DevOps & 云原生文档
category: cloudnative
description: 系统设计风格与最佳实践，涵盖 Kubernetes、Docker、CI/CD、GitOps 等云原生技术。
permalink: /docs/cloudnative/
---

<div class="category-intro">
  <h1>🚀 DevOps & 云原生技术文档</h1>
  
  <p>
    在当今快速发展的技术环境中，DevOps 和云原生技术已成为现代软件开发和运维的核心。
    本系列文档旨在分享我在实际项目中应用这些技术的经验和最佳实践。
  </p>
  
  <div class="category-highlights">
    <h2>📘 文档亮点</h2>
    <ul>
      <li><strong>实践经验</strong> - 基于真实项目案例的技术总结</li>
      <li><strong>系统全面</strong> - 涵盖从基础概念到高级应用的完整知识体系</li>
      <li><strong>与时俱进</strong> - 紧跟技术发展趋势，持续更新最新内容</li>
      <li><strong>易于理解</strong> - 通过图表和示例帮助读者快速掌握复杂概念</li>
    </ul>
  </div>
  
  <div class="tech-stack">
    <h2>🔧 涉及技术栈</h2>
    <p>
      这些文档涵盖了以下核心技术领域：
    </p>
    <ul>
      <li><strong>容器技术</strong> - Docker、Podman 等容器化解决方案</li>
      <li><strong>编排平台</strong> - Kubernetes、OpenShift 等容器编排工具</li>
      <li><strong>CI/CD 工具</strong> - Jenkins、GitLab CI、GitHub Actions 等持续集成/部署工具</li>
      <li><strong>监控告警</strong> - Prometheus、Grafana、ELK 等监控和日志分析系统</li>
      <li><strong>基础设施即代码</strong> - Terraform、Ansible 等自动化运维工具</li>
      <li><strong>服务网格</strong> - Istio、Linkerd 等微服务治理方案</li>
    </ul>
  </div>
</div>

在这里你可以找到关于 DevOps 和云原生技术的详细文档，包括容器化、自动化部署、监控告警等方面的实践指南。

## 📖 文档概览
DevOps 和云原生技术正在改变我们构建、部署和运维应用程序的方式。这些文档将带你深入了解现代基础设施和自动化运维的核心概念与实践。</p>

## 📚 文章列表

<div class="posts-list">
  {% assign sorted_pages = site.pages | where: "category", "cloudnative" | sort: "date" | reverse %}
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
  content: "⚙️";
}
</style>