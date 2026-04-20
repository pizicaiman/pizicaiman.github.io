# 博客样式使用示例

本文档展示了博客系统中可用的各种特殊样式及其使用方法。

## 🎨 徽章样式

### 基础徽章

```markdown
<div class="badge">默认徽章</div>
```

### 技术栈徽章

```markdown
<div class="badge badge-kubernetes">Kubernetes</div>
<div class="badge badge-devops">DevOps</div>
<div class="badge badge-monitoring">Monitoring</div>
<div class="badge badge-security">Security</div>
<div class="badge badge-cloud">Cloud</div>
<div class="badge badge-python">Python</div>
<div class="badge badge-go">Go</div>
<div class="badge badge-javascript">JavaScript</div>
```

### 实际效果

<div class="badge badge-kubernetes">Kubernetes</div>
<div class="badge badge-devops">DevOps</div>
<div class="badge badge-monitoring">Monitoring</div>
<div class="badge badge-security">Security</div>
<div class="badge badge-cloud">Cloud</div>
<div class="badge badge-python">Python</div>
<div class="badge badge-go">Go</div>
<div class="badge badge-javascript">JavaScript</div>

## 📋 荣誉列表样式

### 使用方法

```markdown
<div class="honors-list">
  <ol>
    <li>
      <strong>荣誉标题：</strong><br>
      详细描述这个荣誉或成就的具体内容和意义。
      可以包含技术细节、量化指标等。
    </li>
    <li>
      <strong>另一个荣誉：</strong><br>
      描述内容...
    </li>
  </ol>
</div>
```

### 实际效果

<div class="honors-list">
  <ol>
    <li>
      <strong>显著提升应用部署效率：</strong><br>
      通过标准化容器镜像构建、Helm Chart 模板化部署及 GitOps 自动化流水线，将新服务上线周期从数小时缩短至 10 分钟以内；结合资源请求（<code>requests</code>）与限制（<code>limits</code>）的精细化配置，集群整体 CPU 与内存利用率提升 40% 以上。
    </li>
    <li>
      <strong>实现分钟级弹性扩缩容：</strong><br>
      基于 Kubernetes HPA 及 KEDA，构建多维度弹性伸缩机制，支持 CPU/内存、自定义业务指标等触发条件；在大促、秒杀等高并发场景下，服务可在 2–3 分钟内完成自动扩容，峰值承载能力提升 3 倍。
    </li>
  </ol>
</div>

## 🚨 警告框样式

### 信息提示

```markdown
<div class="alert-box alert-info">
  <strong>提示：</strong> 这是一个信息提示框，用于提供一般性信息。
</div>
```

<div class="alert-box alert-info">
  <strong>提示：</strong> 这是一个信息提示框，用于提供一般性信息。
</div>

### 警告提示

```markdown
<div class="alert-box alert-warning">
  <strong>警告：</strong> 这是一个警告提示框，用于提醒用户注意潜在问题。
</div>
```

<div class="alert-box alert-warning">
  <strong>警告：</strong> 这是一个警告提示框，用于提醒用户注意潜在问题。
</div>

### 错误提示

```markdown
<div class="alert-box alert-error">
  <strong>错误：</strong> 这是一个错误提示框，用于显示错误信息。
</div>
```

<div class="alert-box alert-error">
  <strong>错误：</strong> 这是一个错误提示框，用于显示错误信息。
</div>

### 成功提示

```markdown
<div class="alert-box alert-success">
  <strong>成功：</strong> 这是一个成功提示框，用于显示操作成功的信息。
</div>
```

<div class="alert-box alert-success">
  <strong>成功：</strong> 这是一个成功提示框，用于显示操作成功的信息。
</div>

## 🎯 功能网格样式

### 使用方法

```markdown
<div class="feature-grid">
  <div class="feature-item">
    <h4>功能标题 1</h4>
    <p>功能描述内容...</p>
  </div>
  <div class="feature-item">
    <h4>功能标题 2</h4>
    <p>功能描述内容...</p>
  </div>
  <div class="feature-item">
    <h4>功能标题 3</h4>
    <p>功能描述内容...</p>
  </div>
</div>
```

### 实际效果

<div class="feature-grid">
  <div class="feature-item">
    <h4>容器编排</h4>
    <p>基于 Kubernetes 的容器编排和管理，支持自动扩缩容和负载均衡。</p>
  </div>
  <div class="feature-item">
    <h4>CI/CD 流水线</h4>
    <p>自动化的持续集成和持续部署流程，支持 GitOps 工作模式。</p>
  </div>
  <div class="feature-item">
    <h4>监控告警</h4>
    <p>全栈监控体系，包括指标、日志和链路追踪的统一管理。</p>
  </div>
  <div class="feature-item">
    <h4>安全防护</h4>
    <p>多层次安全防护，包括网络安全、镜像安全和运行时安全。</p>
  </div>
</div>

## 💻 技术栈标签样式

### 使用方法

```markdown
<div class="tech-stack">
  <span class="tech-item">Kubernetes</span>
  <span class="tech-item">Docker</span>
  <span class="tech-item">Prometheus</span>
  <span class="tech-item">Grafana</span>
  <span class="tech-item">Jenkins</span>
</div>
```

### 实际效果

<div class="tech-stack">
  <span class="tech-item">Kubernetes</span>
  <span class="tech-item">Docker</span>
  <span class="tech-item">Prometheus</span>
  <span class="tech-item">Grafana</span>
  <span class="tech-item">Jenkins</span>
  <span class="tech-item">GitLab CI</span>
  <span class="tech-item">ArgoCD</span>
  <span class="tech-item">Helm</span>
</div>

## 📝 教程步骤样式

### 使用方法

```markdown
<div class="tutorial-steps">
  <div class="step">
    <h4>步骤一：准备工作</h4>
    <p>描述第一步的具体操作...</p>
  </div>
  <div class="step">
    <h4>步骤二：执行操作</h4>
    <p>描述第二步的具体操作...</p>
  </div>
  <div class="step">
    <h4>步骤三：验证结果</h4>
    <p>描述第三步的具体操作...</p>
  </div>
</div>
```

### 实际效果

<div class="tutorial-steps">
  <div class="step">
    <h4>步骤一：准备工作</h4>
    <p>首先确保已经安装了必要的工具和依赖，包括 Docker、kubectl 和 Helm。</p>
  </div>
  <div class="step">
    <h4>步骤二：执行操作</h4>
    <p>运行部署命令，将应用部署到 Kubernetes 集群中，并配置相应的服务和路由。</p>
  </div>
  <div class="step">
    <h4>步骤三：验证结果</h4>
    <p>检查应用状态，确认所有 Pod 都正常运行，并且可以通过服务访问应用。</p>
  </div>
</div>

## 📊 对比表格样式

### 使用方法

```markdown
<table class="comparison-table">
  <thead>
    <tr>
      <th>特性</th>
      <th>方案 A</th>
      <th>方案 B</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>性能</td>
      <td>高</td>
      <td>中</td>
    </tr>
    <tr>
      <td>复杂度</td>
      <td>中</td>
      <td>低</td>
    </tr>
  </tbody>
</table>
```

### 实际效果

<table class="comparison-table">
  <thead>
    <tr>
      <th>特性</th>
      <th>Kubernetes</th>
      <th>Docker Swarm</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>集群管理</td>
      <td>原生支持，功能强大</td>
      <td>基础支持，功能有限</td>
    </tr>
    <tr>
      <td>服务发现</td>
      <td>内置 DNS 和 Service</td>
      <td>基于 VIP 的服务发现</td>
    </tr>
    <tr>
      <td>负载均衡</td>
      <td>多种负载均衡策略</td>
      <td>简单的轮询负载均衡</td>
    </tr>
    <tr>
      <td>存储管理</td>
      <td>支持多种存储类型</td>
      <td>支持卷挂载</td>
    </tr>
    <tr>
      <td>学习曲线</td>
      <td>较陡峭</td>
      <td>相对平缓</td>
    </tr>
  </tbody>
</table>

## 📄 博客元数据样式

### 使用方法

```markdown
<div class="blog-meta">
  <div class="meta-item">📅 2024-01-15</div>
  <div class="meta-item">🏷️ Kubernetes, DevOps</div>
  <div class="meta-item">⏱️ 10 分钟阅读</div>
  <div class="meta-item">👁️ 1,234 次浏览</div>
</div>
```

### 实际效果

<div class="blog-meta">
  <div class="meta-item">📅 2024-01-15</div>
  <div class="meta-item">🏷️ Kubernetes, DevOps</div>
  <div class="meta-item">⏱️ 10 分钟阅读</div>
  <div class="meta-item">👁️ 1,234 次浏览</div>
</div>

## 🎓 论文展示样式

### 使用方法

```markdown
<div class='paper-box'>
  <div class='paper-box-image'>
    <div>
      <div class="badge">CVPR 2024</div>
      <img src='images/500x300.png' alt="论文缩略图" width="100%">
    </div>
  </div>
  <div class='paper-box-text' markdown="1">
[论文标题](https://arxiv.org/abs/xxxx.xxxxx)

**作者姓名**, 合作者姓名

[**项目链接**](https://github.com/username/project) <strong><span class='show_paper_citations' data='paper-id'></span></strong>
  - 论文摘要或主要贡献
  - 第二个要点
</div>
</div>
```

### 实际效果

<div class='paper-box'>
  <div class='paper-box-image'>
    <div>
      <div class="badge">技术文章</div>
      <img src='images/500x300.png' alt="示例图片" width="100%">
    </div>
  </div>
  <div class='paper-box-text' markdown="1">
[Kubernetes 集群管理最佳实践](/blog/2024-01-15-kubernetes-cluster-management/)

**Pizicaiaman**

[**阅读全文**](/blog/2024-01-15-kubernetes-cluster-management/) <strong><span class='show_paper_citations' data='k8s-best-practices'></span></strong>
  - 深入探讨 Kubernetes 集群的规划、部署和运维最佳实践
  - 涵盖生产环境实战经验和故障排查指南
</div>
</div>

## 🔗 锚点样式

### 使用方法

```markdown
# 章节标题
<span class='anchor' id='section-id'></span>

内容...
```

### 实际效果

# 示例章节
<span class='anchor' id='example-section'></span>

这是一个示例章节，使用了锚点样式，便于页面内导航。

## 💡 使用建议

1. **徽章样式**：用于标记技术栈、文章类型或状态
2. **荣誉列表**：用于展示成就、奖项或重要里程碑
3. **警告框**：用于重要提示、警告或错误信息
4. **功能网格**：用于展示功能特性或产品特点
5. **技术栈标签**：用于展示项目使用的技术
6. **教程步骤**：用于分步骤的教程或指南
7. **对比表格**：用于方案对比或特性比较
8. **博客元数据**：用于显示文章的发布信息
9. **论文展示**：用于展示学术论文或技术报告
10. **锚点样式**：用于创建页面内导航锚点

## 📚 相关文档

- [博客文档使用指南](BLOG_USAGE_GUIDE.md)
- [Jekyll 文档](https://jekyllrb.com/docs/)
- [Markdown 语法指南](https://www.markdownguide.org/)

---

**最后更新：** 2024-01-15