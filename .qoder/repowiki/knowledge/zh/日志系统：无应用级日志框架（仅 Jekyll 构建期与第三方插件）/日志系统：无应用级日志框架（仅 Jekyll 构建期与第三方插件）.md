---
kind: logging_system
name: 日志系统：无应用级日志框架（仅 Jekyll 构建期与第三方插件）
category: logging_system
scope:
    - '**'
source_files:
    - _config.yml
    - Gemfile
---

本仓库是一个基于 AcadHomepage 主题的静态学术个人主页，由 Jekyll 在构建期生成 HTML。仓库中不存在面向运行时应用的日志系统：没有自定义的 logger 初始化、结构化日志字段、日志级别策略或日志输出通道。

证据如下：
- Ruby 侧：Gemfile 显式声明了 `gem "logger"`，但这是为兼容 Ruby 3.4 移除默认 gem 而引入的标准库依赖；整个 `_config.yml`、`_includes`、`_layouts`、`_pages` 等 Jekyll 源文件中没有任何 `require 'logger'` 或 `Logger.new(...)` 的使用，站点本身不产生任何应用日志。
- JavaScript 侧：`assets/js/_main.js` 中的 `console.log` 调用全部被注释掉；`assets/js/plugins/jquery.magnific-popup.js` 中的 `console.log` 属于第三方插件自带的调试代码，并非项目自身日志体系的一部分。
- 构建产物：`_config.yml` 的 `exclude` 列表显式排除了 `log` 目录，说明即使本地运行 `jekyll serve` 产生的构建期日志也不应进入版本控制。
- 博客文章（如 `blog/2025-01-01-k8s-1.35-cilium-kubevip-containerd-high-availability-cluster.md`）中出现 `log.Fatal(err)`、`audit-log-path` 等字样，但这些是文章中示例 Kubernetes YAML 和 Go 代码片段，不属于站点自身的可执行代码。

结论：该仓库不包含面向运行时应用的日志系统，因此 logging_system 类别对本仓库不适用。