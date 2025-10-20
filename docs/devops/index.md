<!-- docs/devops/index.md -->
---
layout: docs-category
title: DevOps & 云原生文档
category: devops
description: 涵盖 Kubernetes、Docker、CI/CD、GitOps 等云原生技术的最佳实践和经验总结。
permalink: /docs/devops/
---

<style>
/* 全局页面宽度优化 */
.docs-category {
  max-width: 2400px !important;
  padding-left: 2.5rem;
  padding-right: 2.5rem;
  margin-left: auto;
  margin-right: auto;
}

@media (max-width: 1200px) {
  .docs-category {
    max-width: 98vw !important;
    padding-left: 1rem;
    padding-right: 1rem;
  }
}

.devops-posts-list {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(340px, 1fr));
  gap: 1.5rem 2rem;
  margin: 2rem 0;
}

.devops-post-item {
  background: #fff;
  border: 1px solid #e1e4e8;
  border-radius: 6px;
  box-sizing: border-box;
  padding: 1.2rem 1rem 1rem 1rem;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  min-width: 0;
}

.devops-post-meta-row {
  display: flex;
  align-items: center;
  margin-top: 0.4rem;
  gap: 1.5rem;
}
.devops-post-excerpt {
  color: #6a737d;
  font-size: 0.97rem;
  margin-top: 0.6rem;
}

/* 统计区跟随宽度自适应并加大间距 */
.stats {
  display: flex;
  gap: 2.5rem;
  margin: 2.5rem 0;
  font-size: 1rem;
}
.stat-item {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.stat-number {
  font-size: 1.8rem;
  font-weight: bold;
  color: #0057b8;
}
.stat-label {
  color: #6a737d;
  margin-top: 0.4rem;
}
</style>

<div class="docs-category">
在这里你可以找到关于 DevOps 和云原生技术的详细文档，包括容器化、自动化部署、监控告警等方面的实践指南。

## 📖 文档概览

DevOps 和云原生技术正在改变我们构建、部署和运维应用程序的方式。这些文档将带你深入了解现代基础设施和自动化运维的核心概念与实践。

## 📚 文章列表

<div class="devops-posts-list">
  {% assign sorted_pages = site.pages | where: "category", "devops" | sort: "date" | reverse %}
  {% for page in sorted_pages %}
    <div class="devops-post-item">
      <h3 style="margin-bottom:0.6rem;"><a href="{{ page.url | relative_url }}">{{ page.title }}</a></h3>
      <div class="devops-post-meta-row">
        <span class="post-date">📅 {{ page.date | date: "%Y-%m-%d" }}</span>
        <!-- 可扩展：比如作者、标签等同排显示 -->
      </div>
      {% if page.excerpt %}
        <div class="devops-post-excerpt">{{ page.excerpt | strip_html | truncate: 160 }}</div>
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
</div>