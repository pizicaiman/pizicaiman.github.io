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

{% for post in site.categories.devops %}
- [{{ post.title }}]({{ post.url }}) - {{ post.date | date: "%Y-%m-%d" }}
  {{ post.excerpt }}
{% endfor %}

## 技术栈

- 容器编排: Kubernetes, Docker
- 持续集成/部署: Jenkins, ArgoCD, Tekton
- 服务网格: Istio, Linkerd
- 监控告警: Prometheus, Grafana
- 日志管理: ELK Stack