---
kind: dependency_management
name: Ruby/Gem 依赖管理（Jekyll 站点）
category: dependency_management
scope:
    - '**'
source_files:
    - Gemfile
    - Gemfile.lock
    - _config.yml
---

本仓库是一个基于 AcadHomepage 主题的 Jekyll 学术主页，依赖管理完全采用 Ruby Bundler 体系，通过 Gemfile 声明、Gemfile.lock 锁定版本，并在 _config.yml 中同步启用同名插件。

1. 使用的系统与工具
- 包管理器：Bundler（由 Gemfile + Gemfile.lock 体现），RubyGems 官方源 https://rubygems.org。
- 构建运行时：Jekyll ~> 4.3，配合 kramdown/kramdown-parser-gfm 作为 Markdown 引擎，rouge 做语法高亮，sass-embedded 编译 SCSS。
- 部署环境：GitHub Pages / Vercel（见 vercel.json），均通过 bundle exec jekyll build 驱动。

2. 关键文件与分组
- Gemfile：集中声明所有 gem 依赖，分为三类
  - 核心运行时：jekyll、kramdown、kramdown-parser-gfm、rouge、minima、jekyll-watch、jekyll-sass-converter、tzinfo/tzinfo-data、webrick 等；
  - Ruby 3.4 移除的内置库显式声明：csv、base64、logger、mutex_m、bigdecimal、drb、timeout；
  - :jekyll_plugins 组：jekyll-feed、jekyll-sitemap、jekyll-seo-tag、jekyll-paginate、jekyll-gist、jekyll-redirect-from。
- Gemfile.lock：固定所有直接/传递依赖的具体版本号及 x64-mingw-ucrt 平台信息，确保本地与 CI 一致。
- _config.yml 的 plugins 与 whitelist 字段与 Gemfile 中的 jekyll_plugins 保持一一对应，避免 GitHub Pages safe 模式下的黑名单问题。

3. 架构与约定
- 版本约束策略：对主框架使用“~>”宽松约束（如 jekyll ~> 4.3），对功能插件不写版本号，交由 Bundler 在首次安装时解析并写入 lock 文件，升级时通过 bundle update 统一更新。
- 平台隔离：lock 文件中出现 x64-mingw-ucrt 原生扩展（ffi、sass-embedded、google-protobuf），说明开发机为 Windows；CI/部署环境需匹配相同平台或提供对应预编译二进制。
- 插件与配置双写：新增插件必须同时出现在 Gemfile 的 group :jekyll_plugins 和 _config.yml 的 plugins/whitelist 两处，否则本地可运行而 GitHub Pages 构建失败。
- 静态资源与第三方 JS/CSS：assets/js/plugins、assets/fonts 下直接存放 jQuery 插件与 FontAwesome/Academicons 字体，未使用 npm/yarn 管理前端依赖，属于“手动下载 + 版本化提交”的轻量方式。

4. 开发者应遵循的规则
- 添加新 gem：只在 Gemfile 中声明，然后执行 bundle install 生成锁文件，不要手写 Gemfile.lock。
- 升级依赖：优先用 bundle update <gem> 精确升级单个 gem，再检查 _config.yml 是否需要调整插件列表或白名单。
- 新增 Jekyll 插件：务必同步修改 Gemfile 的 :jekyll_plugins 组和 _config.yml 的 plugins/whitelist，保证本地与 GitHub Pages 行为一致。
- 跨平台注意：若引入带 C 扩展的 gem（如 sass-embedded、ffi），需确保 CI 的 Ruby 版本与本机一致，或使用 Docker 固化构建环境。
- 前端资源：如需引入新的 JS/CSS 库，建议沿用 assets/js/plugins、assets/css 的手动放置方式，并将来源链接与版本注释写在对应文件头部，便于后续替换。