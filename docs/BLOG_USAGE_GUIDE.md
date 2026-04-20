# 博客文档使用指南

本指南详细说明如何在项目中创建、管理和展示博客文档，包括新增文档流程、主页展示配置以及不同类型 Markdown 文件的特殊样式支持。

## 📋 目录

1. [项目结构概述](#项目结构概述)
2. [新增文档流程](#新增文档流程)
3. [文档在主页展示](#文档在主页展示)
4. [特殊样式支持](#特殊样式支持)
5. [最佳实践](#最佳实践)

## 🏗️ 项目结构概述

```
pizicaiman.github.io/
├── _pages/              # 页面文档（固定页面）
│   ├── about.md         # 关于页面（主页）
│   ├── projects.md      # 项目页面
│   └── internships.md   # 实习经历页面
├── _posts/              # 博客文章（按日期组织）
├── _layouts/            # 布局模板
│   └── default.html     # 默认布局
├── _includes/           # 可重用组件
├── _sass/               # 样式文件
└── _config.yml          # 站点配置
```

## 📝 新增文档流程

### 1. 创建页面文档（_pages）

页面文档用于创建固定页面，如关于、项目、博客等。

#### 步骤：

1. **创建 Markdown 文件**
   - 在 `_pages/` 目录下创建新的 `.md` 文件
   - 文件名将决定 URL 路径

2. **设置 Front Matter**
   ```yaml
   ---
   permalink: /your-page/          # URL 路径
   title: "页面标题"                # 页面标题
   excerpt: "页面简介"              # 页面描述
   author_profile: true             # 是否显示作者信息
   layout: default                  # 使用的布局
   ---
   ```

3. **编写内容**
   ```markdown
   # 页面主标题
   
   这里是页面内容，支持标准 Markdown 语法。
   
   ## 子标题
   
   - 列表项 1
   - 列表项 2
   ```

#### 示例：创建博客页面

```markdown
---
permalink: /blog/
title: "博客"
excerpt: "我的技术博客文章"
author_profile: true
layout: default
---

# 技术博客

欢迎来到我的技术博客，这里分享我的技术见解和经验。

## 最新文章

- [Kubernetes 最佳实践](/blog/kubernetes-best-practices/)
- [DevOps 自动化指南](/blog/devops-automation/)
```

### 2. 创建博客文章（_posts）

博客文章按日期组织，适合时间线式的内容展示。

#### 步骤：

1. **创建带日期的文件名**
   - 格式：`YYYY-MM-DD-title.md`
   - 示例：`2024-01-15-kubernetes-best-practices.md`

2. **设置 Front Matter**
   ```yaml
   ---
   title: "文章标题"
   date: 2024-01-15 10:00:00 +0800
   categories: [技术, Kubernetes]
   tags: [Kubernetes, DevOps, 最佳实践]
   excerpt: "文章摘要"
   author_profile: true
   ---
   ```

3. **编写内容**
   ```markdown
   # Kubernetes 最佳实践
   
   本文介绍 Kubernetes 集群管理的最佳实践。
   
   ## 集群规划
   
   ### 节点配置
   - 控制平面节点：3 个
   - 工作节点：根据负载动态调整
   ```

## 🏠 文档在主页展示

### 1. 在主页添加内容

主页文件是 `_pages/about.md`，可以通过以下方式添加内容：

#### 方式一：直接添加内容

在 `about.md` 文件中直接添加新的章节：

```markdown
# 🎉 About Me
<span class='anchor' id='about-me'></span>

现有内容...

# 📚 最新博客文章
<span class='anchor' id='latest-blogs'></span>

## 技术文章

- [Kubernetes 集群管理最佳实践](/blog/kubernetes-cluster-management/)
- [DevOps CI/CD 流水线设计](/blog/devops-cicd-pipeline/)
- [云原生架构设计模式](/blog/cloud-native-patterns/)

## 学习笔记

- [Go 语言并发编程](/blog/go-concurrency/)
- [Python 数据分析实战](/blog/python-data-analysis/)
```

#### 方式二：使用卡片样式

```markdown
# 📚 推荐阅读

<div class='paper-box'>
  <div class='paper-box-image'>
    <div>
      <div class="badge">技术文章</div>
      <img src='images/k8s-logo.png' alt="Kubernetes" width="100%">
    </div>
  </div>
  <div class='paper-box-text' markdown="1">
[Kubernetes 集群管理最佳实践](/blog/kubernetes-cluster-management/)

深入探讨 Kubernetes 集群的规划、部署和运维最佳实践。

[**阅读全文**](/blog/kubernetes-cluster-management/) <strong><span class='show_paper_citations' data='k8s-best-practices'></span></strong>
  - 涵盖集群架构设计、资源调度、安全配置等核心主题
  - 提供生产环境实战经验和故障排查指南
</div>
</div>
```

### 2. 创建导航链接

在 `_data/navigation.yml` 中添加导航链接：

```yaml
main:
  - title: "首页"
    url: /
  - title: "项目"
    url: /projects/
  - title: "博客"
    url: /blog/
  - title: "实习经历"
    url: /internships/
```

## 🎨 特殊样式支持

### 1. 论文展示样式

用于展示学术论文或技术报告：

```markdown
<div class='paper-box'>
  <div class='paper-box-image'>
    <div>
      <div class="badge">CVPR 2024</div>
      <img src='images/paper-thumb.jpg' alt="论文缩略图" width="100%">
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

### 2. 荣誉列表样式

用于展示荣誉、奖项或成就：

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

### 3. 锚点样式

用于创建页面内导航锚点：

```markdown
# 章节标题
<span class='anchor' id='section-id'></span>

内容...
```

### 4. 徽章样式

用于标记文章类型或状态：

```markdown
<div class="badge">Kubernetes</div>
<div class="badge">DevOps</div>
<div class="badge">原创</div>
```

### 5. 代码块样式

支持语法高亮的代码块：

```markdown
\```yaml
# Kubernetes 配置示例
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
\```
```

### 6. 表格样式

标准 Markdown 表格，自动应用样式：

```markdown
| 技术栈 | 用途 | 熟练度 |
|--------|------|--------|
| Kubernetes | 容器编排 | 精通 |
| Docker | 容器化 | 精通 |
| Jenkins | CI/CD | 熟练 |
| Prometheus | 监控 | 熟练 |
```

### 7. 警告框样式

用于重要提示或警告：

```markdown
> **注意：** 这是一个重要提示信息。
> 
> 可以包含多行内容。

> **警告：** 这是一个警告信息，需要特别注意。
```

### 8. 自定义 HTML 样式

如果需要更复杂的样式，可以直接使用 HTML：

```html
<div class="custom-section">
  <h3>自定义样式区域</h3>
  <p>这里可以放置任何 HTML 内容</p>
  <ul class="feature-list">
    <li>功能点 1</li>
    <li>功能点 2</li>
  </ul>
</div>
```

## 💡 最佳实践

### 1. 文件命名规范

- **页面文件**：使用小写字母和连字符，如 `blog-guide.md`
- **博客文章**：使用日期前缀，如 `2024-01-15-article-title.md`
- **图片文件**：使用描述性名称，如 `kubernetes-architecture.png`

### 2. Front Matter 规范

始终包含必要的 Front Matter：

```yaml
---
title: "必填：页面/文章标题"
permalink: /url-path/    # 页面文档必填
date: YYYY-MM-DD        # 博客文章必填
excerpt: "简短描述"
author_profile: true    # 是否显示作者信息
categories: []          # 分类
tags: []                # 标签
---
```

### 3. 内容组织

- 使用清晰的标题层级（H1-H6）
- 每个章节使用锚点便于导航
- 适当使用列表和表格提高可读性
- 为长文章添加目录导航

### 4. 图片管理

- 将图片放在 `images/` 目录
- 使用相对路径引用：`images/your-image.png`
- 为图片添加描述性 alt 文本
- 优化图片大小以提高加载速度

### 5. 链接管理

- 内部链接使用相对路径：`/blog/article-title/`
- 外部链接添加描述性文本
- 重要链接考虑使用按钮样式

### 6. SEO 优化

- 为每个页面设置有意义的 title 和 description
- 使用语义化的 HTML 标签
- 添加适当的 meta 标签

### 7. 版本控制

- 提交前检查 Markdown 语法
- 编写有意义的 commit 信息
- 定期备份重要内容

## 🔧 常见问题

### Q: 如何修改页面 URL？

A: 在 Front Matter 中修改 `permalink` 字段：

```yaml
---
permalink: /new-url-path/
---
```

### Q: 如何隐藏作者信息？

A: 在 Front Matter 中设置：

```yaml
---
author_profile: false
---
```

### Q: 如何添加自定义 CSS？

A: 在 `_includes/head/custom.html` 中添加：

```html
```

### Q: 如何调试页面渲染问题？

A: 检查以下几点：
1. Front Matter 格式是否正确
2. Markdown 语法是否有误
3. 文件编码是否为 UTF-8
4. 查看浏览器控制台错误信息

## 📚 相关资源

- [Jekyll 官方文档](https://jekyllrb.com/docs/)
- [Markdown 语法指南](https://www.markdownguide.org/)
- [Kramdown 文档](https://kramdown.gettalong.org/)
- [Liquid 模板语言](https://shopify.github.io/liquid/)

---

**最后更新：** 2024-01-15  
**维护者：** Pizicaiman