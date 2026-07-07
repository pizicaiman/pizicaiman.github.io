---
kind: frontend_style
name: 基于 Minimal Mistakes 的 SCSS 主题样式体系
category: frontend_style
scope:
    - '**'
source_files:
    - assets/css/main.scss
    - _sass/_variables.scss
    - _sass/_mixins.scss
    - _sass/_base.scss
    - _sass/_utilities.scss
    - _sass/_page.scss
    - _sass/_sidebar.scss
    - _sass/_footer.scss
    - _sass/_animations.scss
---

本仓库采用 Jekyll + AcadHomepage（基于 Minimal Mistakes）主题的学术个人主页，前端样式完全由 SCSS 驱动，遵循模块化、变量集中、响应式优先的设计约定。

## 1. 系统与方法论
- 样式预处理：全部使用 Sass/SCSS，通过 _sass 目录按功能拆分为独立文件，最终由 assets/css/main.scss 统一引入编译为 main.css。
- CSS 方法论：以 BEM 风格命名为主（如 .page__title、.sidebar__right），配合工具类（.text-center、.align-left）与语义化组件类（.paper-box、.badge-*）。
- 网格与布局：基于 Susy（12 列流体网格）+ Breakpoint 媒体查询 mixin 实现响应式；同时大量使用 Flexbox 与 CSS Grid（如 .feature-grid）。
- 图标与字体：集成 Font Awesome 5（solid/brands/regular）、Academicons，字体资源位于 assets/fonts/。
- 动画与过渡：全局 $global-transition: all 0.2s ease-in-out，页面进入使用 @keyframes intro 淡入。

## 2. 关键文件与包
- 入口与构建：assets/css/main.scss（所有 SCSS 的统一入口）
- 设计令牌：_sass/_variables.scss（颜色、字号阶梯、断点、网格、圆角、阴影等）
- 通用混入：_sass/_mixins.scss（em() 函数、clearfix）
- 基础样式：_sass/_base.scss（html/body、排版、链接、代码块、figure、导航列表）
- 页面结构：_sass/_page.scss、_sass/_masthead.scss、_sass/_sidebar.scss、_sass/_footer.scss
- 组件与工具：_sass/_buttons.scss、_sass/_notices.scss、_sass/_utilities.scss、_sass/_animations.scss
- 第三方依赖：_sass/vendor/（Breakpoint、Susy、Font Awesome、Magnific Popup）
- 站点级覆盖：assets/css/main.scss 末尾追加的论文卡片、徽章、特性网格、教程步骤、对比表格、博客元信息等自定义样式

## 3. 架构与约定
- 变量即设计令牌：所有颜色（$primary-color、$info-color、$success-color…）、字号（$type-size-1 ~ $type-size-8）、断点（$small/$medium/$large/$x-large）、字体族、圆角、阴影均集中在 _variables.scss，修改一处即可全局生效。
- 模块按职责拆分：每个 _*.scss 对应一个 UI 区域或能力（reset/base → utilities/animations → buttons/notices/masthead/navigation/footer/syntax → forms/page/archive/sidebar → vendor），避免单文件膨胀。
- 响应式策略：通过 @include breakpoint($size) mixin 包裹样式，断点定义在 _variables.scss 中，默认从 600px/768px/925px/1280px 四档起步。
- 组件命名：主题内置组件使用双下划线 BEM（.page__title、.sidebar__right、.author__avatar），业务扩展组件使用短横线（.paper-box、.badge-kubernetes、.feature-grid、.tech-stack、.tutorial-steps、.comparison-table、.blog-meta）。
- 可访问性：提供 .visually-hidden、.screen-reader-text、.skip-link 等无障碍工具类。
- 打印适配：_print.scss 隐藏非必要元素，保证打印输出整洁。

## 4. 开发者应遵循的规则
1. 新增样式优先放入 _sass/ 下的对应模块文件，不要直接编辑 assets/css/main.css（该文件由 Jekyll 自动编译生成）。
2. 全局变量只改 _variables.scss：颜色、字号、断点、圆角、阴影等必须通过变量引用，禁止硬编码色值或尺寸。
3. 响应式写法统一使用 @include breakpoint(...)，不要手写 @media (min-width: ...)。
4. 组件类名保持语义化与一致性：主题内用 __ 分隔 BEM，业务扩展用 - 连接短横线命名。
5. 复用现有工具类：对齐、可见性、间距等优先使用 _utilities.scss 提供的 .text-center、.hidden、.cf 等，避免重复定义。
6. 图标统一走 Font Awesome / Academicons，通过 <i class="fa-solid fa-xxx"></i> 或 <span class="ai ai-xxx"></span> 引用，不在 SCSS 里写伪内容符。
7. 动画与过渡使用 $global-transition 和 intro keyframe，保持交互节奏一致。
8. 新增业务组件时，在 assets/css/main.scss 末尾追加 SCSS 片段并保留注释区块，便于后续维护定位。