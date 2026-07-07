---
kind: build_system
name: Jekyll 静态站点构建与 Vercel 部署
category: build_system
scope:
    - '**'
source_files:
    - Gemfile
    - Gemfile.lock
    - _config.yml
    - vercel.json
    - run_server.sh
    - .gitignore
---

本项目基于 Jekyll（~> 4.3）构建学术个人主页，采用 Ruby Bundler 管理依赖并通过 Vercel 进行静态站点托管。

**构建系统组成**
- **依赖管理**：通过 Gemfile + Gemfile.lock 锁定 jekyll、kramdown、rouge、jekyll-sass-converter、jekyll-feed、jekyll-sitemap、jekyll-seo-tag、jekyll-paginate、jekyll-gist、jekyll-redirect-from 等插件；显式声明 csv/base64/logger/mutex_m/bigdecimal/drb/timeout/webrick 以兼容 Ruby 3.4 移除的默认 gem。
- **本地开发**：run_server.sh 提供一键启动脚本 bundle exec jekyll serve --livereload，启用热重载。
- **站点配置**：_config.yml 集中定义 site title/author、permalink 格式、markdown 引擎（kramdown+GFM）、Sass 目录与压缩输出、时区（Asia/Shanghai）、白名单插件等。
- **产物输出**：Jekyll 默认生成 _site/ 目录，已被 .gitignore 忽略。

**部署流程**
- 使用 Vercel 托管，vercel.json 配置 SPA 风格重定向 (.*) → /index.html，使 Jekyll 生成的多级 URL 在客户端路由下正常工作。
- 无 Dockerfile、Makefile、CI 流水线或跨平台编译脚本，部署完全由 Vercel 自动触发（推测基于 Git push）。

**约定与约束**
- 所有构建相关依赖必须通过 gem install / bundle install 安装，禁止全局安装 jekyll。
- 修改 _config.yml 后需重启本地服务（Jekyll 不会自动重载该文件）。
- Sass 源码位于 _sass/，额外加载路径包含 assets/css，编译输出为压缩 CSS。
- 博客文章按 blog/YYYY-MM-DD-title.md 命名，配合 jekyll-paginate 分页。
- 生产环境禁用 HTML 压缩中的 development 环境分支（compress_html.ignore.envs: development）。