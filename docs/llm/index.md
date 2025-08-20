<!-- docs/llm/index.md -->
---
layout: docs-category
title: 大语言模型文档
category: llm
description: 关于大语言模型应用、Prompt Engineering、向量数据库等 AI 技术的深入分析和实践指南。
permalink: /docs/llm/
---

本分类包含大语言模型相关的技术文档，涵盖模型应用架构、提示词工程、向量检索等关键技术和实践案例。

# 大语言模型 (LLM)

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