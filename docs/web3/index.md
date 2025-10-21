---
layout: docs-category
title: Web3 文档
category: web3
description: 区块链、智能合约、去中心化应用等 Web3 技术的原理分析和开发指南。
permalink: /docs/web3/
---

探索 Web3 技术栈，包括区块链基础、智能合约开发、去中心化存储、跨链技术等前沿领域的技术文档。

## 📖 文档概览

Web3 代表了互联网的下一个演进阶段，通过区块链技术实现去中心化和用户数据主权。这些文档将带你深入了解 Web3 的核心技术。

## 📚 文章列表

<div class="posts-list">
  {% assign sorted_pages = site.pages | where: "category", "web3" | sort: "date" | reverse %}
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

- **区块链基础**: Ethereum, Polygon, Solana
- **智能合约**: Solidity, Rust
- **去中心化应用**: Web3.js, Ethers.js
- **去中心化存储**: IPFS, Filecoin, Arweave
- **跨链技术**: Bridge, Relay Chain

## 🎯 学习路径建议

1. **基础阶段**: 学习区块链基本概念和加密货币原理
2. **进阶阶段**: 掌握智能合约开发和DApp构建
3. **高级阶段**: 实践去中心化存储和跨链技术
4. **专家阶段**: 构建完整的Web3生态系统

## 📊 统计信息

<div class="stats">
  <div class="stat-item">
    <span class="stat-number">{{ site.pages | where: "category", "web3" | size }}</span>
    <span class="stat-label">篇文档</span>
  </div>
  <div class="stat-item">
    <span class="stat-number">{{ site.pages | where: "category", "web3" | where_exp: "page", "page.tags" | size }}</span>
    <span class="stat-label">个标签</span>
  </div>
</div>