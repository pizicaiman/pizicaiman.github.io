<!-- docs/devops/index.md -->
---
layout: docs-category
title: DevOps & 云原生文档
category: devops
description: 系统设计风格与最佳实践，涵盖 Kubernetes、Docker、CI/CD、GitOps 等云原生技术。
permalink: /docs/devops/
---

<div class="docs-category">
  <div class="docs-content-wrapper">
    <p>在这里你可以找到关于 DevOps 和云原生技术的详细文档，包括容器化、自动化部署、监控告警等方面的实践指南。</p>

    <h2>📖 文档概览</h2>
    <p>DevOps 和云原生技术正在改变我们构建、部署和运维应用程序的方式。这些文档将带你深入了解现代基础设施和自动化运维的核心概念与实践。</p>

    <h2>📚 文章列表</h2>
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

    <h2>⚙️ 核心技术栈</h2>
    <div class="tech-stack">
      <ul>
        <li><strong>容器编排</strong>: Kubernetes, Docker</li>
        <li><strong>持续集成/部署</strong>: Jenkins, ArgoCD, Tekton</li>
        <li><strong>服务网格</strong>: Istio, Linkerd</li>
        <li><strong>监控告警</strong>: Prometheus, Grafana</li>
        <li><strong>日志管理</strong>: ELK Stack</li>
      </ul>
    </div>

    <h2>🎯 学习路径建议</h2>
    <div class="learning-path">
      <ol>
        <li><strong>基础阶段</strong>: 学习 Docker 和容器化概念</li>
        <li><strong>进阶阶段</strong>: 掌握 Kubernetes 核心概念和操作</li>
        <li><strong>高级阶段</strong>: 实践 GitOps 和服务网格技术</li>
        <li><strong>专家阶段</strong>: 构建完整的云原生平台</li>
      </ol>
    </div>

    <h2>📊 统计信息</h2>
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
</div>