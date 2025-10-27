# Koordinator 源码分析

---

## 1. 项目简介

[Koordinator](https://github.com/koordinator-sh/koordinator) 是阿里云开源的 Kubernetes 原生混部调度与智能资源管理系统。它基于 kube-scheduler 扩展，实现了弹性 oversell、NUMA 亲和、QoS 隔离、批量任务优化等高级功能，适用于 AI、大数据与在线服务混部场景。

---

## 2. 架构组件与主链路

Koordinator 主要组件：

- **koord-scheduler**：基于插件链定制化调度扩展，兼容和增强原生 scheduler。
- **koordlet**：Node Agent，精细监控、按 QoS 自动治理。如动态限速、NUMA 跨核隔离等。
- **koord-controller-manager**：批量任务 CRD 管理、Job/PodGroup 生命周期调控。
- **CRD 扩展**：如 ElasticQuota/PodGroup/Reservation，支撑多维超配、批量排队。

调度主链：

1. APIServer 收到 Pod/Job
2. koord-controller-manager 监听 CR，生成调度对象
3. koord-scheduler 按插件（如 ElasticQuota、NUMA、Batch等）依序评分/预选
4. 下发调度决策，koordlet 在节点侧动态治理

---

## 3. 关键源码路径

- **cmd/koord-scheduler/**  
  启动入口，注册自定义调度器和插件
- **pkg/scheduler/plugins/**  
  各种独立能力插件（如 ElasticQuota、NUMA、Batch/Colocation 等），实现 kube-scheduler 扩展接口
- **pkg/koordlet/**  
  节点级治理/agent 主逻辑
- **pkg/controllers/**  
  Job/PodGroup/ElasticQuota 等核心 CRD 控制器

典型插件说明：

- **elasticquota**：弹性资源超配，支持线上/离线/弹性抢占
- **numatopology**：NUMA-aware 资源亲和调度
- **batching**：批量任务优先公平等排队策略
- **colocation**：服务与 AI 作业混部隔离保障

---

## 4. 调度主流程源码

以下是 Koordinator 调度主流程（以 elasticquota/numa 为例）：

1. **入口**：`cmd/koord-scheduler/main.go`
   - 加载插件/框架，启动 informer 与 leader 选举
2. **注册插件**：`pkg/scheduler/plugins/*/register.go`
   - 插件需实现 Filter/Score/PreFilter 等调度阶段接口，注册到 framework
3. **调度调用链**：
   - `pkg/scheduler/framework.go` → 逐插件 Filter/PreFilter/Score
   - `elasticquota` 插件调整每个节点可用资源
   - `numatopology` 强制 Pod/Node NUMA 匹配
   - `batching` 根据 CRD/FIFO/优先级等组合队列
4. 最终通过 Patch 或 Bind API，分配节点

---

## 5. 节点治理与 QoS 管控

- **koordlet** 通过 cgroup/fs/proc/CRI 查询和治理 pods：
    - 根据 QoS/资源类型限速/阻隔
    - 自动 NUMA 亲和、冷热分级迁移
    - 弹性/批量 pod 动态升降级，controller 管理生命周期

重要源码：

- `pkg/koordlet/metriccollect`（数据收集）
- `pkg/koordlet/resource/manager`（资源限速/隔离）
- `pkg/koordlet/qosmanager`（QoS 动态控制）

---

## 6. CRD 工作流与扩展点

Koordinator 强依赖 CRD 扩展。典型如 ElasticQuota、PodGroup、Reservation：

- CRD 定义 → Controller 实时 watch → 插件感知全局资源/批量排队
- 支持算力池、冷热调度场景的第三方调度注入

---

## 7. 调试与源码追踪建议

- 推荐入口：
    - `cmd/koord-scheduler/main.go`、`pkg/scheduler/`
    - 插件链注册 & 具体插件代码（如 `plugin/elasticquota/`）
    - 节点治理主流程：`koordlet/`
- 开启详细日志：`--v=4`
- CRD/调度对象状态跟踪：`kubectl get elasticquota/podgroup/reservation`
- `kubectl cordon`/`koordinator-cli` 实现节点、group 级别模拟

---

## 8. 参考资料与进阶阅读

- [Koordinator 官方文档](https://koordinator.sh/docs/)
- [Koordinator GitHub 源码](https://github.com/koordinator-sh/koordinator)
- [核心设计与热点 PR](https://github.com/koordinator-sh/koordinator/pulls)
- [K8s Scheduler Framework - 官方文档](https://kubernetes.io/zh-cn/docs/concepts/scheduling-eviction/scheduler-extensions/)
- [NUMA 调度设计详解](https://koordinator.sh/docs/arch/numa/)

---

> 总结：Koordinator 以 CRD 插件链和节点治理为核心，形成了 K8s 混部场景下最完整的“不打补丁”深度扩展方案。深度源码分析应聚焦 scheduler 框架定制点、elasticquota/numa 等核心插件机制。欢迎评论区提出针对功能/源码片段的深入分析需求！
