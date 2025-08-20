<!-- docs/devops/index.md -->
---
layout: docs-category
title: DevOps & 云原生文档
category: devops
description: 涵盖 Kubernetes、Docker、CI/CD、GitOps 等云原生技术的最佳实践和经验总结。
permalink: /docs/devops/
---

在这里你可以找到关于 DevOps 和云原生技术的详细文档，包括容器化、自动化部署、监控告警等方面的实践指南。

## 📖 文档概览

DevOps 和云原生技术正在改变我们构建、部署和运维应用程序的方式。这些文档将带你深入了解现代基础设施和自动化运维的核心概念与实践。

## 📚 文章列表

<div class="posts-list">
  {% assign sorted_pages = site.pages | where: "category", "devops" | sort: "date" | reverse %}
  {% for page in sorted_pages %}
    <div class="post-item">
      <h3><a href="{{ page.url | relative_url }}">{{ page.title }}</a></h3>
      <p class="post-meta">
        <span class="post-date">📅 {{ page.date | date: "%Y-%m-%d" }}</span>
      </p>
      {% if page.excerpt %}
        <p class="post-excerpt">{{ page.excerpt | strip_html | truncate: 160 }}</p>
      {% endif %}
    </div>
  {% endfor %}
</div>

## ⚙️ 核心技术栈

- **容器编排**: Kubernetes, Docker
- **持续集成/部署**: Jenkins, ArgoCD, Tekton
- **服务网格**: Istio, Linkerd
- **监控告警**: Prometheus, Grafana
- **日志管理**: ELK Stack

## 🎯 学习路径建议

1. **基础阶段**: 学习 Docker 和容器化概念
2. **进阶阶段**: 掌握 Kubernetes 核心概念和操作
3. **高级阶段**: 实践 GitOps 和服务网格技术
4. **专家阶段**: 构建完整的云原生平台

## 📊 统计信息

<div class="stats">
  <div class="stat-item">
    <span class="stat-number">{{ site.pages | where: "category", "devops" | size }}</span>
    <span class="stat-label">篇文档</span>
  </div>
  <div class="stat-item">
    <span class="stat-number">{{ site.pages | where: "category", "devops" | where_exp: "page", "page.tags" | size }}</span>
    <span class="stat-label">个标签</span>
  </div>
</div>