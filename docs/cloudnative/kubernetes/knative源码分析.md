# Karmada 源码分析

Karmada（Kubernetes Multi-cluster管理器）通过原生 Kubernetes 机制实现多集群编排、统一调度和资源治理。源码层面高度复用了 K8s 控制器及调度框架，并围绕 CRD、调度、资源同步等核心流程模块化分层，易于扩展和二次开发。

---

## 1. 整体架构与核心组件

Karmada 主要源码仓库：[github.com/karmada-io/karmada](https://github.com/karmada-io/karmada)

### 主要组件

- **karmada-controller-manager**：核心控制循环及资源同步，通过各类 Controller 管理 Policy、资源分发等。
- **karmada-scheduler**：统一调度器，决定 workload 派发目标集群，插件化调度主链。
- **karmada-apiserver**：多集群资源聚合与 API 入口，负责 CRD 注册和管理。
- **karmada-agent**：部署于成员集群，负责目标资源下发、状态回报。
- **scheduler-estimator**：辅助调度，动态感知成员集群资源能力。

### 典型源码结构

```
cmd/                 # 各组件启动入口 main.go
pkg/
  controllers/       # controller-manager 控制器子模块
  scheduler/         # 调度主链与插件框架
  apiserver/         # API 服务层
  agent/             # 集群代理
  util/              # 辅助工具代码
apis/                # CRD 类型和 API 声明
test/                # 测试代码
```

---

## 2. 核心流程源码主线

### 2.1 资源同步与 Policy 控制

Karmada 通过 CRD `PropagationPolicy`、`ClusterPropagationPolicy` 定义资源同步规则。

**Controller 主干源码**：`pkg/controllers/binding/binding_controller.go`
- 监听 Policy 或资源对象，生成 ResourceBinding（同步状态 CRD）
- 根据调度计划派发目标集群

主要逻辑框架示例：
```go
func (c *Controller) syncWork(...) error {
  // 1. 查询匹配 Policy
  // 2. 计算目标分发集群
  // 3. 生成/更新 ResourceBinding
  // ...
}
```

---

### 2.2 调度器插件链路

**入口代码**：`pkg/scheduler/scheduler.go`

- 监听 ResourceBinding，执行调度决策
- 插件化调度链路（Filter/Score/Bind，自定义扩展极简）
- 支持调度策略自定义开发，机制与 K8s Scheduling Framework 类似

接口示例：
```go
type Scheduler interface {
  Schedule(binding *workv1alpha1.ResourceBinding) (scheduleResult, error)
}
```
插件注册与实现见 `pkg/scheduler/framework/plugins/`

---

### 2.3 Agent 同步与状态反馈

- karmada-agent 持续拉取 control plane 下发的资源 Work CRD
- 自动 apply 到本地集群，并上报同步状态/健康报告
- 核心控制代码：`pkg/agent/controller/work_status_controller.go`

---

## 3. 二次开发/扩展典型场景

- **自定义调度插件**：在 `pkg/scheduler/framework/plugins/` 扩展 Filter/Score 运算，支持多云容灾、成本感知等。
- **资源类型扩展**：通过额外 CRD 适配更多自定义资源或融合 Operator。
- **健康与灾备增强**：基于 agent 上报的集群状态，实现业务自愈/多活。

---

## 4. 源码调试与建议

- 推荐用 kind/多集群模拟，单独运行 controller/scheduler/agent，便于断点和日志追查
- 聚焦 Binding/Policy/Work CRD 链路，梳理事件流动与状态变化
- 调度/同步链每步均有详尽日志和 Event，可用于排查和观测

---

## 5. 资料参考

- [Karmada 架构与官方文档](https://karmada.io/zh/docs/)
- [核心 CRD/控制器源码](https://github.com/karmada-io/karmada/tree/master/apis)
- [社区分析专栏](https://jimmysong.io/kubernetes-handbook/integrate/karmada.html)

---

> 总结：Karmada 源码解耦清晰，强依赖 K8s CRD 与控制/调度框架，便于插件化、多云场景下的自定义开发。理解控制器-调度-同步主链，有助于二次开发与集成。如需插件开发、资源跨集群同步等问题，欢迎留言交流。
