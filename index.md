---
layout: default
title: Pizicai's Tech Blog
---

# 技术探索与实践

欢迎来到我的技术博客，这里记录了我在以下领域的学习与实践:

## 🚀 DevOps & 云原生
- [Kubernetes多集群管理实践](/devops/2024/kubernetes-multi-cluster.md)
- [GitOps持续部署方案](/devops/2024/gitops-deployment.md)
- [查看更多...](/categories/devops)

## 🤖 大语言模型
- [企业级LLM应用架构设计](/llm/2024/enterprise-llm-arch.md)
- [向量数据库选型对比](/llm/2024/vector-db-comparison.md)
- [查看更多...](/categories/llm)

## ⛓️ Web3
- [跨链桥技术原理解析](/web3/2024/cross-chain-bridge.md)
- [去中心化存储方案](/web3/2024/decentralized-storage.md)
- [查看更多...](/categories/web3)

## 📊 GitHub 统计
![GitHub统计](https://github-readme-stats.vercel.app/api?username=pizicaiman&show_icons=true&theme=radical)

## 🔍 最近更新
<ul>
  {% for post in site.posts limit:5 %}
    <li>
      <span>{{ post.date | date: "%Y-%m-%d" }}</span> &raquo; 
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>

## 🏷️ 标签云
{% for tag in site.tags %}
  <a href="/tags/{{ tag[0] }}" style="font-size: {{ tag[1].size | times: 4 | plus: 80 }}%">{{ tag[0] }}</a>
{% endfor %}