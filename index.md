---
layout: default
title: Pizicai's Tech Blog
---
<style>
.section { 
  margin: 2rem 0; 
  max-width: 800px; 
  margin-left: auto; 
  margin-right: auto; 
}
.section h2 { 
  margin: 0 0 0.75rem 0; 
  font-size: 1.25rem; 
  text-align: center; 
}
.links ul { 
  margin: 0.5rem 0 0 0; 
  text-align: center; 
}
.links li { 
  margin: 0.35rem 0; 
  display: inline-block; 
  margin: 0.5rem 1rem; 
}
.links a { 
  color: #0366d6; 
  text-decoration: none; 
  padding: 0.5rem 1rem; 
  border: 1px solid #e1e4e8; 
  border-radius: 6px; 
  display: inline-block; 
  transition: all 0.2s ease; 
}
.links a:hover { 
  text-decoration: none; 
  background-color: #f6f8fa; 
  border-color: #0366d6; 
}
.grid { 
  display: grid; 
  grid-template-columns: 2fr 1fr; 
  gap: 1.25rem; 
  max-width: 1000px; 
  margin: 0 auto; 
}
.panel { 
  border: 1px solid #e1e4e8; 
  border-radius: 8px; 
  padding: 0.75rem 1rem; 
  background: #fff; 
}
.panel h3 { 
  margin: 0 0 0.5rem 0; 
  font-size: 1.05rem; 
  text-align: center; 
}
.recent-list { 
  list-style: none; 
  margin: 0; 
  padding: 0; 
}
.recent-list li { 
  display: flex; 
  gap: 0.5rem; 
  align-items: baseline; 
  padding: 0.45rem 0; 
  border-bottom: 1px solid #f1f1f1; 
}
.recent-list li:last-child { 
  border-bottom: 0; 
}
.recent-list time { 
  color: #6a737d; 
  font-size: 0.9rem; 
}
.recent-list a { 
  color: #24292e; 
  text-decoration: none; 
}
.recent-list a:hover { 
  color: #0366d6; 
  text-decoration: underline; 
}
.tag-cloud { 
  text-align: center; 
}
.tag-cloud a { 
  display: inline-block; 
  margin: 0.25rem 0.5rem 0 0; 
  color: #586069; 
  text-decoration: none; 
}
.tag-cloud a:hover { 
  text-decoration: underline; 
}
.github-stats { 
  text-align: center; 
  margin: 2rem 0; 
}
@media (max-width: 900px) { 
  .grid { 
    grid-template-columns: 1fr; 
  } 
  .links li { 
    display: block; 
    margin: 0.5rem 0; 
  }
}
</style>

<div class="home">
  <!-- <div class="hero">
    <div class="hero-inner">
      <h1>技术探索与实践</h1>
      <p class="lead">记录我在 DevOps、LLM 与 Web3 等方向的系统化学习与最佳实践。</p>
    </div>
  </div> -->

  <div class="section links">
    <h2>🚀 DevOps & 云原生</h2>
    <ul>
      <li><a href="/docs/devops/2024/kubernetes-multi-cluster/">Kubernetes 多集群管理实践</a></li>
      <li><a href="/docs/devops/2024/gitops-deployment/">GitOps 持续部署方案</a></li>
      <li><a href="/docs/devops/">查看全部文档</a></li>
    </ul>
  </div>

  <div class="section links">
    <h2>🤖 大语言模型</h2>
    <ul>
      <li><a href="/docs/llm/2024/enterprise-llm-arch/">企业级 LLM 应用架构设计</a></li>
      <li><a href="/docs/llm/2024/vector-db-comparison/">向量数据库选型对比</a></li>
      <li><a href="/docs/llm/">查看全部文档</a></li>
    </ul>
  </div>

  <div class="section links">
    <h2>⛓️ Web3</h2>
    <ul>
      <li><a href="/docs/web3/2024/cross-chain-bridge/">跨链桥技术原理解析</a></li>
      <li><a href="/docs/web3/2024/decentralized-storage/">去中心化存储方案</a></li>
      <li><a href="/docs/web3/">查看全部文档</a></li>
    </ul>
  </div>

  <div class="grid section">
    <div class="panel">
      <h3>🔍 最近更新</h3>
      <ul class="recent-list">
        {% for post in site.posts limit:5 %}
          <li>
            <time>{{ post.date | date: "%Y-%m-%d" }}</time>
            <a href="{{ post.url }}">{{ post.title }}</a>
          </li>
        {% endfor %}
      </ul>
    </div>
    <div class="panel">
      <h3>🏷️ 标签云</h3>
      <div class="tag-cloud">
        {% for tag in site.tags %}
          <a href="/tags/{{ tag[0] }}" style="font-size: {{ tag[1].size | times: 4 | plus: 80 }}%">{{ tag[0] }}</a>
        {% endfor %}
      </div>
    </div>
  </div>

  <div class="section github-stats">
    <h2>📊 GitHub 统计</h2>
    <img alt="GitHub统计" src="https://github-readme-stats.vercel.app/api?username=pizicaiman&show_icons=true" />
  </div>
</div>