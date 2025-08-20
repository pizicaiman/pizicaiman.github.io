<!-- docs/devops/index.md -->
---
layout: docs-category
title: DevOps & 云原生文档
category: devops
description: 涵盖 Kubernetes、Docker、CI/CD、GitOps 等云原生技术的最佳实践和经验总结。
permalink: /docs/devops/
---

在这里你可以找到关于 DevOps 和云原生技术的详细文档，包括容器化、自动化部署、监控告警等方面的实践指南。

# DevOps & 云原生

## 文章列表

<div class="posts-list">
  {% assign sorted_pages = site.pages | where: "category", "devops" | sort: "date" | reverse %}
  {% for page in sorted_pages %}
    <div class="post-item">
      <h3><a href="{{ page.url | relative_url }}">{{ page.title }}</a></h3>
      <p class="post-meta">
        <span class="post-date">{{ page.date | date: "%Y-%m-%d" }}</span>
      </p>
      {% if page.excerpt %}
        <p class="post-excerpt">{{ page.excerpt | strip_html | truncate: 160 }}</p>
      {% endif %}
    </div>
  {% endfor %}
</div>

## 技术栈

- 容器编排: Kubernetes, Docker
- 持续集成/部署: Jenkins, ArgoCD, Tekton
- 服务网格: Istio, Linkerd
- 监控告警: Prometheus, Grafana
- 日志管理: ELK Stack