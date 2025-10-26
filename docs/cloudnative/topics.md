---
layout: docs-category
title: CloudNative 专题
category: cloudnative
description: CloudNative 相关的核心专题，包括 DevOps、SRE 和 Linux 等关键技术领域
permalink: /docs/cloudnative/topics/
author: Pizicai
---

<div class="category-intro">
  <h1>📘 CloudNative 核心专题</h1>
  
  <p>
    CloudNative 技术栈涵盖多个关键领域，其中 DevOps、SRE 和 Linux 是构建和维护现代云原生系统的基石。
    本页面汇总了这些核心专题，帮助您快速了解和掌握相关技术。
  </p>
  
  <div class="category-highlights">
    <h2>🎯 专题概览</h2>
    <ul>
      <li><strong>相互关联</strong> - 这些专题相互关联，共同构成云原生技术基础</li>
      <li><strong>实践导向</strong> - 基于真实项目案例的技术总结</li>
      <li><strong>系统全面</strong> - 涵盖从基础概念到高级应用的完整知识体系</li>
      <li><strong>持续更新</strong> - 紧跟技术发展趋势，持续更新最新内容</li>
    </ul>
  </div>
</div>

## 📚 核心专题

<div class="topics-grid">
  <div class="topic-card">
    <h3><a href="/docs/cloudnative/">🚀 DevOps & 云原生</a></h3>
    <p class="topic-description">
      涵盖容器技术、编排平台、CI/CD 工具、监控告警、基础设施即代码等云原生核心技术。
    </p>
    <ul class="topic-highlights">
      <li>容器化技术 (Docker、Podman)</li>
      <li>容器编排 (Kubernetes、OpenShift)</li>
      <li>持续集成/部署 (Jenkins、GitLab CI)</li>
      <li>监控和日志分析 (Prometheus、Grafana)</li>
    </ul>
    <a href="/docs/cloudnative/" class="topic-link">查看专题文档 →</a>
  </div>
  
  <div class="topic-card">
    <h3><a href="/docs/sre/">🎯 站点可靠性工程 (SRE)</a></h3>
    <p class="topic-description">
      专注于系统可靠性、监控告警、容量规划、故障处理等关键 SRE 实践方法。
    </p>
    <ul class="topic-highlights">
      <li>可靠性原则 (SLI/SLO/SLA)</li>
      <li>监控系统设计</li>
      <li>容量规划与性能优化</li>
      <li>故障响应与事后分析</li>
    </ul>
    <a href="/docs/sre/" class="topic-link">查看专题文档 →</a>
  </div>
  
  <div class="topic-card">
    <h3><a href="/docs/linux/">🐧 Linux 系统管理</a></h3>
    <p class="topic-description">
      涵盖 Linux 系统管理与运维实践，包括系统配置、性能优化、安全加固等。
    </p>
    <ul class="topic-highlights">
      <li>系统管理与配置</li>
      <li>网络配置与安全</li>
      <li>性能优化与监控</li>
      <li>Bash 脚本编程</li>
    </ul>
    <a href="/docs/linux/" class="topic-link">查看专题文档 →</a>
  </div>
</div>

<style>
.category-intro {
  margin-bottom: 2rem;
  padding: 1.5rem;
  background-color: #f6f8fa;
  border-radius: 8px;
  border-left: 4px solid #9b59b6;
}

.category-intro h1 {
  margin-top: 0;
  color: #24292e;
}

.category-intro p {
  line-height: 1.6;
  color: #586069;
}

.category-highlights h2 {
  color: #24292e;
  border-bottom: 1px solid #e1e4e8;
  padding-bottom: 0.5rem;
}

.category-highlights ul {
  list-style-type: none;
  padding-left: 0;
}

.category-highlights li {
  padding: 0.5rem 0;
  border-bottom: 1px solid #eaecef;
}

.category-highlights li:last-child {
  border-bottom: none;
}

.category-highlights li::before {
  content: "✓";
  color: #28a745;
  font-weight: bold;
  display: inline-block;
  width: 1.5em;
  margin-right: 0.5em;
}

.topics-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 2rem;
  margin: 2rem 0;
}

.topic-card {
  border: 1px solid #e1e4e8;
  border-radius: 8px;
  padding: 1.5rem;
  background-color: #fff;
  box-shadow: 0 1px 3px rgba(0,0,0,0.05);
  transition: box-shadow 0.3s ease, transform 0.3s ease;
}

.topic-card:hover {
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
  transform: translateY(-4px);
}

.topic-card h3 a {
  text-decoration: none;
  color: #0366d6;
}

.topic-card h3 a:hover {
  text-decoration: underline;
}

.topic-description {
  color: #586069;
  margin: 1rem 0;
  line-height: 1.5;
}

.topic-highlights {
  list-style-type: none;
  padding-left: 0;
  margin: 1rem 0;
}

.topic-highlights li {
  padding: 0.25rem 0;
  color: #586069;
  position: relative;
  padding-left: 1.2em;
}

.topic-highlights li::before {
  content: "•";
  color: #9b59b6;
  position: absolute;
  left: 0;
  top: 0.25em;
}

.topic-link {
  display: inline-block;
  margin-top: 1rem;
  color: #0366d6;
  text-decoration: none;
  font-weight: 500;
}

.topic-link:hover {
  text-decoration: underline;
}
</style>