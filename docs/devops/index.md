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
  line-height: 1.7;
}

/* 标题样式优化 */
.docs-category h2 {
  color: #24292e;
  border-bottom: 2px solid #0366d6;
  padding-bottom: 0.5rem;
  margin-top: 2.5rem;
  margin-bottom: 1.5rem;
  font-size: 1.4rem;
  font-weight: 600;
}

.docs-category h2:first-of-type {
  margin-top: 1.5rem;
}

/* 段落样式优化 */
.docs-category > p {
  font-size: 1.05rem;
  color: #586069;
  margin-bottom: 1.5rem;
  line-height: 1.7;
}

@media (max-width: 1200px) {
  .docs-category {
    max-width: 98vw !important;
    padding-left: 1rem;
    padding-right: 1rem;
  }
}

.devops-posts-list {
  margin: 2rem 0;
  padding: 0;
  list-style: none;
}

.devops-post-item {
  padding: 1.5rem 0;
  border-bottom: 1px solid #f0f0f0;
  transition: all 0.2s ease;
  position: relative;
}

.devops-post-item:last-child {
  border-bottom: none;
}

.devops-post-item:hover {
  background-color: #fafbfc;
  margin: 0 -1rem;
  padding-left: 1rem;
  padding-right: 1rem;
  border-radius: 6px;
}

.devops-post-header {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 1rem;
  margin-bottom: 0.5rem;
}

.devops-post-title {
  flex: 1;
  margin: 0;
  font-size: 1.1rem;
  font-weight: 600;
  line-height: 1.4;
}

.devops-post-title a {
  color: #24292e;
  text-decoration: none;
  transition: color 0.2s ease;
}

.devops-post-title a:hover {
  color: #0366d6;
}

.devops-post-meta {
  display: flex;
  align-items: center;
  gap: 1rem;
  font-size: 0.85rem;
  color: #6a737d;
  white-space: nowrap;
  margin-top: 0.1rem;
}

.devops-post-excerpt {
  color: #6a737d;
  font-size: 0.9rem;
  line-height: 1.5;
  margin-top: 0.5rem;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

/* 统计区跟随宽度自适应并加大间距 */
.stats {
  display: flex;
  gap: 3rem;
  margin: 3rem 0;
  padding: 2rem;
  background: linear-gradient(135deg, #f8f9fa 0%, #e9ecef 100%);
  border-radius: 12px;
  border: 1px solid #e1e4e8;
  justify-content: center;
}

.stat-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
}

.stat-number {
  font-size: 2.2rem;
  font-weight: 700;
  color: #0366d6;
  margin-bottom: 0.3rem;
}

.stat-label {
  color: #6a737d;
  font-size: 0.9rem;
  font-weight: 500;
}

/* 技术栈样式优化 */
.tech-stack {
  background: #f8f9fa;
  padding: 1.5rem;
  border-radius: 8px;
  border-left: 4px solid #0366d6;
  margin: 1.5rem 0;
}

.tech-stack ul {
  margin: 0;
  padding-left: 1.2rem;
}

.tech-stack li {
  margin-bottom: 0.5rem;
  line-height: 1.6;
}

/* 学习路径样式优化 */
.learning-path {
  background: #fff;
  padding: 1.5rem;
  border-radius: 8px;
  border: 1px solid #e1e4e8;
  margin: 1.5rem 0;
}

.learning-path ol {
  margin: 0;
  padding-left: 1.2rem;
}

.learning-path li {
  margin-bottom: 0.8rem;
  line-height: 1.6;
}

/* 响应式优化 */
@media (max-width: 768px) {
  .devops-post-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 0.5rem;
  }
  
  .devops-post-meta {
    margin-top: 0;
  }
  
  .stats {
    flex-direction: column;
    gap: 1.5rem;
    padding: 1.5rem;
  }
  
  .stat-number {
    font-size: 1.8rem;
  }
}
</style>

<div class="docs-category">

在这里你可以找到关于 DevOps 和云原生技术的详细文档，包括容器化、自动化部署、监控告警等方面的实践指南。

## 📖 文档概览

DevOps 和云原生技术正在改变我们构建、部署和运维应用程序的方式。这些文档将带你深入了解现代基础设施和自动化运维的核心概念与实践。

## 📚 文章列表

<ul class="devops-posts-list">
  {% assign sorted_pages = site.pages | where: "category", "devops" | sort: "date" | reverse %}
  {% for page in sorted_pages %}
    <li class="devops-post-item">
      <div class="devops-post-header">
        <h3 class="devops-post-title">
          <a href="{{ page.url | relative_url }}">{{ page.title }}</a>
        </h3>
        <div class="devops-post-meta">
          <span class="post-date">📅 {{ page.date | date: "%Y-%m-%d" }}</span>
        </div>
      </div>
      {% if page.excerpt %}
        <div class="devops-post-excerpt">{{ page.excerpt | strip_html | truncate: 200 }}</div>
      {% endif %}
    </li>
  {% endfor %}
</ul>

## ⚙️ 核心技术栈

<div class="tech-stack">
- **容器编排**: Kubernetes, Docker
- **持续集成/部署**: Jenkins, ArgoCD, Tekton
- **服务网格**: Istio, Linkerd
- **监控告警**: Prometheus, Grafana
- **日志管理**: ELK Stack
</div>

## 🎯 学习路径建议

<div class="learning-path">
1. **基础阶段**: 学习 Docker 和容器化概念
2. **进阶阶段**: 掌握 Kubernetes 核心概念和操作
3. **高级阶段**: 实践 GitOps 和服务网格技术
4. **专家阶段**: 构建完整的云原生平台
</div>

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