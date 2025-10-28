# Kubernetes Scheduler v1.34 源码深度分析

---

## 1. 总览与架构

Kubernetes Scheduler 负责从待调度的 Pod 列表出发，依据集群资源、调度策略和调度插件等，决定每个 Pod 最合适的目标节点（node），并将调度结果写入 etcd（即 Pod 的 nodeName 字段）。

核心架构如下：

- **入口（main.go/Run函数）**
  - 初始化配置，注册 client、informer
  - 加载全部调度算法插件
- **Informer**: 监听未绑定 node 的 Pod、新 Node、新资源等事件
- **调度队列(SchedulingQueue)**: 管理 schedulable pod 的队列，支持优先级/回退
- **主流程(Scheduler.ScheduleOne→SchedulePod)**
  - 过滤(Filter) → 打分(Score) → 选优(Permit/Bind)
- **插件(Plugins)机制**: 核心通过插件化支持几乎全部调度策略扩展

---

## 2. 主要流程源码分解

以下以 kube-scheduler v1.34 源码为例，主调用路径为：

`cmd/kube-scheduler/app/server.go → Run → scheduler.Run(context)`

### 2.1 Pod 调度主循环

```go
// pkg/scheduler/scheduler.go
func (sched *Scheduler) Run(ctx context.Context) {
    ...
    for {
        // 从队列获取下一个待调度 Pod
        podInfo := sched.NextPod()
        ...
        sched.scheduleOne(ctx, podInfo)
    }
}
```

### 2.2 单个 Pod 的调度流程

```go
func (sched *Scheduler) scheduleOne(ctx context.Context, podInfo *framework.QueuedPodInfo) {
    // 1. 过滤节点（Filter Plugins）
    feasibleNodes, filteredStatuses, err := sched.Filtering(...)
    // 2. 节点打分（Score Plugins）
    scores, err := sched.Scoring(...)
    // 3. 预绑定/准入（Permit Plugins）
    ...
    // 4. 绑定 Pod 到 Node
    err = sched.bind(...)
}
```
各阶段通过注册的插件链调用，实现高度可拓展。

---

## 3. 核心插件机制

Scheduler 将调度各环节全部抽象为插件（framework plugin），主要接口在 `pkg/scheduler/framework`：

- Filter: 过滤不满足资源/规则的节点（如 NodeUnschedulable, NodeAffinity, PodAffinity, Taints）
- Score: 对候选节点打分排序（如 LeastAllocated, NodeResourcesBalancedAllocation）
- PreBind/Bind/Reserve/Prioritize: 细颗粒度阶段性控制
- Permit: 支持异步抢占、多 Pod 互斥调度
- 等等

插件编排与参数全部由调度器配置文件（kubescheduler.config.k8s.io/v1beta3）动态注入。

---

## 4. 调度队列与优先级

调度队列（`pkg/scheduler/internal/queue`）支持多队列、backoff、unschedulable pod 处理。典型逻辑：

- 新 Pod/重试 Pod 进入 activeQ
- 若调度失败，backoff 转 unschedulableQ
- 事件触发唤醒 Pod 重新尝试

---

## 5. 典型默认插件举例

- **NodeResourcesFit**：节点资源 filter
- **PodTopologySpread**：同/异构 Pod 拓扑分布
- **TaintToleration**：污点容忍/忽略节点
- **DefaultPreemption**：原生抢占不可调度 Pod

---

## 6. v1.34 新变化与重点特性

- **框架插件链更灵活，支持 plugin weight、动态配置热更新**
- **Gang scheduling、Coscheduling 插件原生增强**
- **优先级队列调度逻辑优化，unschedulable Pod反应更快**
- **支持更丰富的硬件、拓扑、能耗策略**
- **多实例水平扩展：支持并行多调度器，提高大集群吞吐率**

---

## 7. 调试与源码阅读技巧

- 推荐入口：`cmd/kube-scheduler`、`pkg/scheduler/`
- `--v=5`/`--v=10` 打开详细调度日志
- 查看 K8s Scheduler Config 配置插件顺序及参数
- 结合 Event、Pod/Node 对象更新跟踪调度决策

---

## 8. 参考资料

- [Kube-scheduler 官方文档](https://kubernetes.io/zh-cn/docs/concepts/scheduling-eviction/kube-scheduler/)
- [kubernetes/kubernetes/pkg/scheduler/ 源码](https://github.com/kubernetes/kubernetes/tree/master/pkg/scheduler)
- [K8s v1.34 Release Notes](https://github.com/kubernetes/kubernetes/releases/tag/v1.34.0)
- [Scheduler Framework 设计详解](https://jasonwzx.github.io/2023/01/09/Kubernetes-Scheduler-Framework/)

如需要对具体插件源码/调用链详细剖析，欢迎留言讨论！

