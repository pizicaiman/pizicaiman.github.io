---
kind: configuration_system
name: Jekyll 站点根配置与数据分层
category: configuration_system
scope:
    - '**'
source_files:
    - _config.yml
    - Gemfile
    - _data/navigation.yml
    - vercel.json
---

本仓库是一个基于 AcadHomepage 主题的 Jekyll 静态站点，其配置系统由 Jekyll 原生机制驱动，采用 YAML + Gemfile 双轨方式组织：运行时站点元信息集中在 _config.yml，构建期 Ruby/Jekyll 插件依赖通过 Gemfile 声明，页面导航等可编辑内容以 _data/*.yml 形式注入到模板。

1. 使用的系统与工具
- Jekyll 4.3（gem jekyll ~> 4.3），Markdown 引擎为 kramdown（GFM 模式），代码高亮使用 rouge。
- 插件体系通过 _config.yml 的 plugins/whitelist 与 Gemfile 的 group :jekyll_plugins 双重声明，兼顾 GitHub Pages 安全白名单。
- Vercel SPA 路由重写通过根目录 vercel.json 实现单页回退。

2. 关键文件与包
- _config.yml：站点全局配置（标题、作者、SEO、include/exclude、permalink、timezone、Sass、defaults、压缩等）。
- Gemfile / Gemfile.lock：锁定 Jekyll 版本及插件（jekyll-feed、jekyll-sitemap、jekyll-seo-tag、jekyll-paginate、jekyll-gist、jekyll-redirect-from 等）。
- _data/navigation.yml：导航菜单数据，被 masthead/sidebar 等 include 模板消费。
- vercel.json：Vercel 部署时的 SPA 重定向规则。
- run_server.sh：本地开发启动脚本（封装 bundle exec jekyll serve）。

3. 架构与约定
- 配置分层：
  - 站点元数据与构建选项 -> _config.yml
  - 可编辑页面级数据 -> _data/*.yml（如导航）
  - 构建期依赖 -> Gemfile
  - 部署期行为 -> vercel.json
- 默认值集中化：_config.yml 中 defaults 块为 _pages 下所有页面统一设置 layout: default 和 author_profile: true，避免在每个 Markdown 文件中重复声明。
- 插件白名单：plugins 与 whitelist 同步列出，确保在 GitHub Pages 与本地 --safe 模式下行为一致。
- 资源排除策略：exclude 列表显式剔除 docs、node_modules、vendor、assets/js/plugins 等，防止污染 _site。

4. 开发者应遵循的规则
- 新增站点元信息或构建开关时，优先写入 _config.yml 对应段落，保持注释分组不变。
- 需要跨页面共享的数据（导航、侧边栏条目等）放入 _data/*.yml，并通过 Liquid 在 include 模板中读取，不要硬编码进模板。
- 新增 Jekyll 插件必须同时出现在 Gemfile 的 group :jekyll_plugins 与 _config.yml 的 plugins/whitelist 三处，保证本地与 GitHub Pages 一致性。
- 修改 _config.yml 后需重启 jekyll serve，因为该文件不会被热重载。
- 部署到 Vercel 时，SPA 路由重写已内置于 vercel.json，无需额外改动。