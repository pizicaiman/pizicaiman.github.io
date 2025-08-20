<!-- docs/web3/index.md -->
---
layout: docs-category
title: Web3 文档
category: web3
description: 区块链、智能合约、去中心化应用等 Web3 技术的原理分析和开发指南。
permalink: /docs/web3/
---

探索 Web3 技术栈，包括区块链基础、智能合约开发、去中心化存储、跨链技术等前沿领域的技术文档。

# Web3

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