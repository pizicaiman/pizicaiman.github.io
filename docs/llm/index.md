---
layout: docs-category
title: 大语言模型文档
category: llm
description: 关于大语言模型应用、Prompt Engineering、向量数据库等 AI 技术的深入分析和实践指南。
permalink: /docs/llm/
---

大语言模型相关的技术文档，涵盖模型应用架构、提示词工程、向量检索等关键技术和实践案例。

## 📖 文档概览

大语言模型（LLM）正在引领新一轮的AI革命。这些文档将帮助你理解LLM的核心原理、应用架构和最佳实践。

## 📚 文章列表

<div class="posts-list">
  {% assign sorted_pages = site.pages | where: "category", "llm" | sort: "date" | reverse %}
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

- **大语言模型**: GPT, LLaMA, Claude
- **向量数据库**: Pinecone, Weaviate, Milvus
- **提示词工程**: Chain-of-Thought, Few-shot Learning
- **模型微调**: LoRA, QLoRA
- **部署方案**: HuggingFace, ONNX, TensorRT

## 🎯 学习路径建议

1. **基础阶段**: 理解LLM基本原理和应用场景
2. **进阶阶段**: 掌握提示词工程和向量检索技术
3. **高级阶段**: 学习模型微调和部署优化
4. **专家阶段**: 构建端到端的LLM应用系统

## 📊 统计信息

<div class="stats">
  <div class="stat-item">
    <span class="stat-number">{{ site.pages | where: "category", "llm" | size }}</span>
    <span class="stat-label">篇文档</span>
  </div>
  <div class="stat-item">
    <span class="stat-number">{{ site.pages | where: "category", "llm" | where_exp: "page", "page.tags" | size }}</span>
    <span class="stat-label">个标签</span>
  </div>
</div>