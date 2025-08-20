---
layout: category
title: 大语言模型 (LLM) 文章
category: llm
permalink: /docs/llm/
---

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