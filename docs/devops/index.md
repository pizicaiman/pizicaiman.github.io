<!-- docs/devops/index.md -->
---
layout: docs-category
title: DevOps & 云原生文档
category: devops
description: 涵盖 Kubernetes、Docker、CI/CD、GitOps 等云原生技术的最佳实践和经验总结。
permalink: /docs/devops/
---

<style>
/* 现代化页面设计 */
.docs-category {
  max-width: 2400px !important;
  padding: 3rem 2.5rem;
  margin: 0 auto;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  min-height: 100vh;
  position: relative;
  overflow: hidden;
}

.docs-category::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: 
    radial-gradient(circle at 20% 80%, rgba(120, 119, 198, 0.3) 0%, transparent 50%),
    radial-gradient(circle at 80% 20%, rgba(255, 255, 255, 0.1) 0%, transparent 50%),
    radial-gradient(circle at 40% 40%, rgba(120, 119, 198, 0.2) 0%, transparent 50%);
  pointer-events: none;
}

.docs-content-wrapper {
  position: relative;
  z-index: 1;
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(20px);
  border-radius: 24px;
  padding: 3rem;
  box-shadow: 
    0 20px 40px rgba(0, 0, 0, 0.1),
    0 0 0 1px rgba(255, 255, 255, 0.2);
  margin: 2rem 0;
}

/* 标题样式 - 现代化设计 */
.docs-category h2 {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  font-size: 1.8rem;
  font-weight: 700;
  margin: 3rem 0 2rem 0;
  position: relative;
  padding-left: 1rem;
}

.docs-category h2::before {
  content: '';
  position: absolute;
  left: 0;
  top: 50%;
  transform: translateY(-50%);
  width: 4px;
  height: 2rem;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 2px;
}

.docs-category h2:first-of-type {
  margin-top: 0;
}

/* 段落样式 */
.docs-category > p {
  font-size: 1.1rem;
  color: #4a5568;
  margin-bottom: 2rem;
  line-height: 1.8;
  font-weight: 400;
}

@media (max-width: 1200px) {
  .docs-category {
    padding: 2rem 1rem;
  }
  
  .docs-content-wrapper {
    padding: 2rem;
    border-radius: 16px;
  }
}

/* 现代化文章列表设计 */
.devops-posts-list {
  margin: 2rem 0;
  padding: 0;
  list-style: none;
  display: grid;
  gap: 1.5rem;
}

.devops-post-item {
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.9) 0%, rgba(255, 255, 255, 0.7) 100%);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.3);
  border-radius: 16px;
  padding: 2rem;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  position: relative;
  overflow: hidden;
  box-shadow: 
    0 4px 6px rgba(0, 0, 0, 0.05),
    0 1px 3px rgba(0, 0, 0, 0.1);
}

.devops-post-item::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 3px;
  background: linear-gradient(90deg, #667eea 0%, #764ba2 100%);
  transform: scaleX(0);
  transition: transform 0.3s ease;
}

.devops-post-item:hover {
  transform: translateY(-4px);
  box-shadow: 
    0 20px 25px rgba(0, 0, 0, 0.1),
    0 10px 10px rgba(0, 0, 0, 0.04);
  border-color: rgba(102, 126, 234, 0.3);
}

.devops-post-item:hover::before {
  transform: scaleX(1);
}

.devops-post-header {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 1.5rem;
  margin-bottom: 1rem;
}

.devops-post-title {
  flex: 1;
  margin: 0;
  font-size: 1.3rem;
  font-weight: 700;
  line-height: 1.3;
}

.devops-post-title a {
  background: linear-gradient(135deg, #2d3748 0%, #4a5568 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  text-decoration: none;
  transition: all 0.3s ease;
}

.devops-post-title a:hover {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.devops-post-meta {
  display: flex;
  align-items: center;
  gap: 1rem;
  font-size: 0.9rem;
  color: #718096;
  white-space: nowrap;
  background: rgba(102, 126, 234, 0.1);
  padding: 0.5rem 1rem;
  border-radius: 20px;
  font-weight: 500;
}

.devops-post-excerpt {
  color: #4a5568;
  font-size: 1rem;
  line-height: 1.6;
  margin-top: 0.8rem;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

/* 现代化统计区域设计 */
.stats {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 2rem;
  margin: 3rem 0;
  padding: 0;
}

.stat-item {
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.9) 0%, rgba(255, 255, 255, 0.7) 100%);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.3);
  border-radius: 20px;
  padding: 2.5rem 2rem;
  text-align: center;
  position: relative;
  overflow: hidden;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  box-shadow: 
    0 4px 6px rgba(0, 0, 0, 0.05),
    0 1px 3px rgba(0, 0, 0, 0.1);
}

.stat-item::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(135deg, rgba(102, 126, 234, 0.05) 0%, rgba(118, 75, 162, 0.05) 100%);
  opacity: 0;
  transition: opacity 0.3s ease;
}

.stat-item:hover {
  transform: translateY(-8px);
  box-shadow: 
    0 20px 25px rgba(0, 0, 0, 0.1),
    0 10px 10px rgba(0, 0, 0, 0.04);
}

.stat-item:hover::before {
  opacity: 1;
}

.stat-number {
  font-size: 3rem;
  font-weight: 800;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  margin-bottom: 0.5rem;
  position: relative;
  z-index: 1;
}

.stat-label {
  color: #4a5568;
  font-size: 1rem;
  font-weight: 600;
  position: relative;
  z-index: 1;
}

/* 现代化技术栈设计 */
.tech-stack {
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.9) 0%, rgba(255, 255, 255, 0.7) 100%);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.3);
  border-radius: 20px;
  padding: 2.5rem;
  margin: 2rem 0;
  position: relative;
  overflow: hidden;
  box-shadow: 
    0 4px 6px rgba(0, 0, 0, 0.05),
    0 1px 3px rgba(0, 0, 0, 0.1);
}

.tech-stack::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  width: 4px;
  height: 100%;
  background: linear-gradient(180deg, #667eea 0%, #764ba2 100%);
}

.tech-stack ul {
  margin: 0;
  padding-left: 1.5rem;
  position: relative;
  z-index: 1;
}

.tech-stack li {
  margin-bottom: 0.8rem;
  line-height: 1.7;
  font-size: 1.05rem;
  color: #4a5568;
  font-weight: 500;
}

/* 现代化学习路径设计 */
.learning-path {
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.9) 0%, rgba(255, 255, 255, 0.7) 100%);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.3);
  border-radius: 20px;
  padding: 2.5rem;
  margin: 2rem 0;
  position: relative;
  overflow: hidden;
  box-shadow: 
    0 4px 6px rgba(0, 0, 0, 0.05),
    0 1px 3px rgba(0, 0, 0, 0.1);
}

.learning-path ol {
  margin: 0;
  padding-left: 1.5rem;
  position: relative;
  z-index: 1;
}

.learning-path li {
  margin-bottom: 1rem;
  line-height: 1.7;
  font-size: 1.05rem;
  color: #4a5568;
  font-weight: 500;
  position: relative;
}

.learning-path li::marker {
  color: #667eea;
  font-weight: 700;
}

/* 响应式优化 */
@media (max-width: 768px) {
  .docs-category {
    padding: 1.5rem 1rem;
  }
  
  .docs-content-wrapper {
    padding: 1.5rem;
    border-radius: 16px;
  }
  
  .devops-post-item {
    padding: 1.5rem;
  }
  
  .devops-post-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 0.8rem;
  }
  
  .devops-post-title {
    font-size: 1.2rem;
  }
  
  .devops-post-meta {
    margin-top: 0;
    font-size: 0.85rem;
    padding: 0.4rem 0.8rem;
  }
  
  .stats {
    grid-template-columns: 1fr;
    gap: 1.5rem;
  }
  
  .stat-item {
    padding: 2rem 1.5rem;
  }
  
  .stat-number {
    font-size: 2.5rem;
  }
  
  .tech-stack,
  .learning-path {
    padding: 1.5rem;
  }
  
  .docs-category h2 {
    font-size: 1.5rem;
    margin: 2rem 0 1.5rem 0;
  }
}
</style>

<div class="docs-category">
  <div class="docs-content-wrapper">
    <p>在这里你可以找到关于 DevOps 和云原生技术的详细文档，包括容器化、自动化部署、监控告警等方面的实践指南。</p>

    <h2>📖 文档概览</h2>
    <p>DevOps 和云原生技术正在改变我们构建、部署和运维应用程序的方式。这些文档将带你深入了解现代基础设施和自动化运维的核心概念与实践。</p>

    <h2>📚 文章列表</h2>
    <ul class="devops-posts-list">
      {% assign sorted_pages = site.pages | where: "category", "devops" | sort: "date" | reverse %}
      {% for page in sorted_pages %}
        <li class="devops-post-item">
          <div class="devops-post-header">
            <h3 class="devops-post-title">
              <a href="{{ page.url | relative_url }}">{{ page.title }}</a>
            </h3>
            <div class="devops-post-meta">
              <span class="post-date">📅 {{ page.date | date: "%Y-%m-%d" }}</span>
            </div>
          </div>
          {% if page.excerpt %}
            <div class="devops-post-excerpt">{{ page.excerpt | strip_html | truncate: 200 }}</div>
          {% endif %}
        </li>
      {% endfor %}
    </ul>

    <h2>⚙️ 核心技术栈</h2>
    <div class="tech-stack">
      <ul>
        <li><strong>容器编排</strong>: Kubernetes, Docker</li>
        <li><strong>持续集成/部署</strong>: Jenkins, ArgoCD, Tekton</li>
        <li><strong>服务网格</strong>: Istio, Linkerd</li>
        <li><strong>监控告警</strong>: Prometheus, Grafana</li>
        <li><strong>日志管理</strong>: ELK Stack</li>
      </ul>
    </div>

    <h2>🎯 学习路径建议</h2>
    <div class="learning-path">
      <ol>
        <li><strong>基础阶段</strong>: 学习 Docker 和容器化概念</li>
        <li><strong>进阶阶段</strong>: 掌握 Kubernetes 核心概念和操作</li>
        <li><strong>高级阶段</strong>: 实践 GitOps 和服务网格技术</li>
        <li><strong>专家阶段</strong>: 构建完整的云原生平台</li>
      </ol>
    </div>

    <h2>📊 统计信息</h2>
    <div class="stats">
      <div class="stat-item">
        <span class="stat-number">{{ site.pages | where: "category", "devops" | size }}</span>
        <span class="stat-label">篇文档</span>
      </div>
      <div class="stat-item">
        <span class="stat-number">{{ site.pages | where: "category", "devops" | where_exp: "page", "page.tags" | size }}</span>
        <span class="stat-label">个标签</span>
      </div>
    </div>
  </div>
</div>