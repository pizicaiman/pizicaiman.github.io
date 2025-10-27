# Karma 源码分析

Karma 是 Prometheus Alertmanager 的一个 UI 管理面，支持多集群/多环境告警聚合、强大的过滤与抑制、标签管理和自定义展示。其核心设计为**前后端解耦，单页应用，动态配置加载**，易于二次开发和多云告警场景适配。

---

## 1. 整体架构与核心模块

Karma 源码仓库：[github.com/prymitive/karma](https://github.com/prymitive/karma)

### 主要架构
- 前端：纯前端 SPA，基于 React（typescript 重构中），UI 交互全部经 REST API 完成。
- 后端：Go 实现的 API 服务，负责 Alertmanager Discovery、聚合、转发、配置动态加载。
- 无状态设计，配置支持热加载，直接跟 alertmanager 对接，无需持久化。

### 目录结构
```
/
  cmd/             # karma API Server 主入口
  internal/        # 配置管理、API 代理、缓存核心逻辑
  ui/              # React SPA 前端代码
  static/          # 前端静态资源编译产物
  pkg/             # 公共 Go library
  examples/        # 示例配置
```

---

## 2. 后端主流程

### 2.1 启动与配置加载

入口: `cmd/karma/main.go`

- 启动参数或配置 (`karma.yml`) 中定义接入的多个 Alertmanager 实例及其聚合规则。
- 支持热加载，监控配置更改自动生效。

**核心加载逻辑:**
```go
func loadConfig() error {
  // 解析 yaml，填充 config 结构体
  // 实例化 alertmanager discovery
  // 构建聚合参数和路由
}
```

### 2.2 Alertmanager 发现与聚合

- 定期探测/拉取所有配置的 alertmanager，读取其 `/api/v2/alerts`、`/api/v2/silences` 等接口
- 聚合告警、静默状态，统一标准数据结构

**实现核心:**
- `internal/alertmanager/`
- `internal/mapper/mapper.go` (多源归一/规范化)

### 2.3 API 转发与缓存

- UI 前端 *所有查询* 均走 karma 后端 REST API（提供过滤、聚合与频控，如 `/alerts.json`, `/silences.json` 等）
- Go server 实现前置缓存，均匀后端压力、防止 alertmanager 被频繁刷爆

```go
func (api *AlertAPI) Alerts(w http.ResponseWriter, r *http.Request) {
  // 解析前端 filter 请求
  // 命中缓存则返回
  // 否则并发拉取多 alertmanager 数据聚合后返回
}
```

---

## 3. 前端主干与定制能力

- 单页 SPA，目录：`ui/src/`，数据全由 `/alerts.json`、`/silences.json` 拉取渲染
- 支持复杂标签过滤、自定义列、静默管理、告警分组与快速搜索
- 状态、过滤、视图参数均 URL 化，便于收藏和分享

**二次开发建议：**
- UI 组件类型化，便于定制过滤器、分组渲染、扩展外部跳转链接
- 前端与后端配置解耦，开发时支持 mock、接口自定义

---

## 4. 常见扩展场景

- **多集群/多告警源聚合**：基础配置中通过 `alertmanager` section 增加多环境、多 endpoint，实现云原生多区域整合。
- **企业 CMDB 联动**：二次开发可在后端聚合层加挂载信息、或前端弹窗内透出业务或资产链路。
- **深度定制告警呈现**：前端自定义分组模板、自定义列表达式满足不同业务告警维度。
- **自定义认证**：karma 原生支持 header 透传、可扩展反向代理或后端自实现 auth deny 逻辑。

---

## 5. 调试与开发实用建议

- 推荐本地启用 `KARMA_LOG_LEVEL=debug`，方便追踪聚合链路与每一次 filter 命中情况
- 配置热加载适合大规模环境下参数快速调优
- 前端开发建议用 `yarn start` 代理本地 API 以获得热 reload 和 mock 能力
- 二次开发新功能建议优先用前端插件或后端 API 协议扩展

---

## 6. 核心源码资料与最佳实践

- [Karma 配置与部署官方文档](https://karma-docs.prymitive.com/)
- [alertmanager 多实例聚合核心代码](https://github.com/prymitive/karma/tree/main/internal/alertmanager)
- [企业静默批量管理、自动标签扩展示例](https://github.com/prymitive/karma/issues?q=silence)
- [社区二次开发 hook 与集成样例](https://github.com/prymitive/karma/pulls)

---

> 总结：Karma 以灵活聚合、多环境适配和极致 UI 体验成为主流告警管理面板。其源码结构高度模块化、适合快速集成多源、深度定制及企业场景二次开发。掌握多 alertmanager 聚合主线与前端过滤核心，将极大提升 Prometheus 生态运维效率。如需多云、CMDB 对接或企业认证自定义，欢迎留言交流！
