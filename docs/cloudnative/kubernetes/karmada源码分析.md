# Karmada 源码分析

Karmada（Kubernetes Multi-cluster管理器）通过原生 Kubernetes 机制提供多集群编排、调度与治理能力。其核心设计理念是**控制面全容器化、资源声明式同步、统一调度入口、分布式执行与健康探测**。以下围绕其主要源码结构、核心控制器和扩展模型做主线分析。

---

## 1. 整体架构与源码布局

Karmada 主要仓库：[github.com/karmada-io/karmada](https://github.com/karmada-io/karmada)

### 主要组件
- `karmada-controller-manager`：核心控制器集群，负责资源同步、分发、预调度等控制循环。
- `karmada-scheduler`：统一调度器，接收工作负载并决定目标成员集群，通过 Plugin 化调度框架实现。
- `karmada-apiserver`：API 聚合服务，实现多集群 API 网关与资源归集。
- `karmada-agent`：部署于成员集群的代理，负责资源拉取、状态回报与健康心跳。
- `scheduler-estimator`：度量 target 集群负载，用于智能调度建议。

### 源码结构
```
cmd/                  # 主可执行文件入口，各组件 main.go
pkg/                  # 主要控制器、调度、API、工具实现
  - controllers/      # 同步/分发等主控制器包
  - scheduler/        # 调度框架与扩展插件
  - apiserver/        # API 服务实现
  - agent/            # 成员集群代理
  - util/             # 通用工具/辅助库
apis/                 # CRD 类型声明
test/                 # E2E/集成测试
```

---

## 2. 核心流程源码主线

### 2.1 统一资源同步与 PropagationPolicy

Karmada 通过 CRD 对资源同步策略进行声明，如 `PropagationPolicy`（单资源同步）、`ClusterPropagationPolicy`（跨集群同步）。

**主控制器源码入口**：`pkg/controllers/binding/binding_controller.go`
- 监听目标资源/Policy 变化，生成 `ResourceBinding`（同步状态）对象
- `ResourceBinding` 再触发成员集群资源分发

**核心逻辑：**
```go
func (c *Controller) syncWork(...) error {
  // 查询匹配的Policy
  // 生成/更新 ResourceBinding 资源
  // 更新调度状态、期望分发集群
}
```

---

### 2.2 调度器调度链路

**主线入口**：`pkg/scheduler/scheduler.go`
- 监听 `ResourceBinding`，根据策略与集群能力执行调度
- 插件化调度链（开发者可自定义 Score, Filter, Spread 策略）
- 调配后写回 ResourceBinding，由控制器分发执行

**示例核心代码：**
```go
// 典型调度接口
type Scheduler interface {
  Schedule(binding *workv1alpha1.ResourceBinding) (scheduleResult, error)
}
```
- 插件注册见 `pkg/scheduler/framework/plugins/`
- 类似 K8s Scheduling Framework，可参考其扩展开发

---

### 2.3 成员集群同步执行链

- `karmada-agent` 角色：
  - 定期从 Karmada control plane 拉取目标资源（Work CRD），并 apply 到本地集群
  - 汇报状态，心跳，上报执行结果

参考代码：`pkg/agent/controller/work_status_controller.go`

---

## 3. 调度扩展与常见二次开发点

- **调度策略扩展**：可在 `pkg/scheduler/framework/plugins/` 实现自定义插件，支持跨集群负载感知、标签亲和、成本感知等场景
- **资源类型拓展**：通过增加 Work CRD 适配更多资源类型，实现 Operator 等扩展同步
- **健康/韧性增强**：基于 Agent 上报状态自定义健康探针与灾备逻辑

---

## 4. 源码调试与实用建议

- 支持 kind/多集群本地模拟，建议开发时单独拉起 controller/scheduler/agent 便于跟踪
- 推荐关注 Binding/Policy/Work 三大 CRD 的流转链路和状态变更
- 调度日志、Event 以及资源 Annotation 是排查首要切入点

---

## 5. 资料参考

- [Karmada 官方架构文档](https://karmada.io/zh/docs/)
- [核心 CRD 设计与 API 源码](https://github.com/karmada-io/karmada/tree/master/apis)
- [多集群 Kubernetes 源码解析专栏](https://jimmysong.io/kubernetes-handbook/integrate/karmada.html)

---

> 总结：Karmada 源码高度模块化，并与 K8s 调度框架保持兼容，便于通用多云/多集群场景的功能二次扩展。掌握控制器链路、调度框架与 Agent 执行体系，是深度定制的基础。如需插件开发、资源同步等进阶问题，可留言交流！