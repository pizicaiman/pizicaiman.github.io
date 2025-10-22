---
layout: docs-category
title: Web3 文档
category: web3
description: 区块链、智能合约、去中心化应用等 Web3 技术的原理分析和开发指南。
permalink: /docs/web3/
---

<div class="category-intro">
  <h1>⛓️ Web3 技术文档</h1>
  
  <p>
    Web3 代表了互联网的去中心化未来，通过区块链技术重新定义了数字所有权和价值交换。
    本系列文档旨在分享我在区块链开发、智能合约和去中心化应用方面的实践经验。
  </p>
  
  <div class="category-highlights">
    <h2>📘 文档亮点</h2>
    <ul>
      <li><strong>技术全面</strong> - 涵盖从区块链基础到高级 DeFi 应用的完整知识体系</li>
      <li><strong>实践导向</strong> - 通过实际项目案例展示技术实现过程</li>
      <li><strong>前沿探索</strong> - 紧跟 Web3 技术发展趋势，涵盖最新技术动态</li>
      <li><strong>开发指南</strong> - 提供详细的开发步骤和代码示例</li>
    </ul>
  </div>
  
  <div class="tech-stack">
    <h2>🔧 涉及技术栈</h2>
    <p>
      这些文档涵盖了以下核心技术领域：
    </p>
    <ul>
      <li><strong>区块链基础</strong> - 共识机制、密码学基础、分布式账本技术</li>
      <li><strong>智能合约</strong> - Solidity、Rust 等智能合约开发语言</li>
      <li><strong>开发框架</strong> - Hardhat、Truffle、Foundry 等开发工具</li>
      <li><strong>DeFi 应用</strong> - 去中心化交易所、借贷协议、稳定币等</li>
      <li><strong>钱包技术</strong> - MetaMask、WalletConnect 等钱包集成方案</li>
      <li><strong>跨链技术</strong> - 跨链桥接、互操作性协议等</li>
    </ul>
  </div>
</div>

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
        <span class="post-date">{{ page.date | date: "%Y-%m-%d" }}</span>
        {% if page.author %}
          <span class="post-author">by {{ page.author }}</span>
        {% endif %}
      </p>
      <p class="post-excerpt">{{ page.excerpt | strip_html | truncate: 150 }}</p>
    </div>
  {% endfor %}
</div>

<style>
.category-intro {
  margin-bottom: 2rem;
  padding: 1.5rem;
  background-color: #f6f8fa;
  border-radius: 8px;
  border-left: 4px solid #2ecc71;
}

.category-intro h1 {
  margin-top: 0;
  color: #24292e;
}

.category-intro p {
  line-height: 1.6;
  color: #586069;
}

.category-highlights,
.tech-stack {
  margin: 1.5rem 0;
}

.category-highlights h2,
.tech-stack h2 {
  color: #24292e;
  border-bottom: 1px solid #e1e4e8;
  padding-bottom: 0.5rem;
}

.category-highlights ul,
.tech-stack ul {
  list-style-type: none;
  padding-left: 0;
}

.category-highlights li,
.tech-stack li {
  padding: 0.5rem 0;
  border-bottom: 1px solid #eaecef;
}

.category-highlights li:last-child,
.tech-stack li:last-child {
  border-bottom: none;
}

.category-highlights li::before,
.tech-stack li::before {
  content: "✓";
  color: #28a745;
  font-weight: bold;
  display: inline-block;
  width: 1.5em;
  margin-right: 0.5em;
}

.tech-stack li::before {
  content: "⚙️";
}
</style>