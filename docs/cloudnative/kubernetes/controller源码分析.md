---
layout: doc
title: Kubernetes Controller v1.34 源码分析
date: 2024-06-25
category: kubernetes
tags: [calico, kubernetes, cni, source-code]
excerpt: Kubernetes Controller v1.34 源码分析
permalink: /docs/cloudnative/kubernetes/controller源码分析/
---

## 1. Controller 作用与设计概览

Kubernetes Controller 是控制面关键组成部分，用于持续驱动集群状态向期望状态收敛（self-healing）。常见控制器包括 Deployment、ReplicaSet、Job、CronJob、StatefulSet、ServiceAccount、Node Controller 等。

整体架构特点：
- **声明式**：用户准备资源清单（YAML），控制器负责实际状态（Actual） 与期望状态（Desired）自动对齐
- **事件驱动+异步循环**：主要通过监听资源变更事件，触发控制循环
- **幂等设计**：Reconcile 方法可多次调用都能保证状态一致

---

## 2. 核心流程主干

以典型控制器（如 Deployment Controller）为例，源码分布于 `k8s.io/kubernetes/pkg/controller/`

**主循环核心代码结构**（以 informer+workqueue 标准模型为例）：

```go
// pkg/controller/deployment/deployment_controller.go
func (dc *DeploymentController) Run(threadiness int, stopCh <-chan struct{}) {
    for i := 0; i < threadiness; i++ {
        go wait.Until(dc.worker, time.Second, stopCh)
    }
    <-stopCh
}

func (dc *DeploymentController) worker() {
    for dc.processNextWorkItem() { }
}

// 核心业务调谐
func (dc *DeploymentController) processNextWorkItem() bool {
    key, quit := dc.queue.Get()
    if quit { return false }
    defer dc.queue.Done(key)
    err := dc.syncHandler(key.(string))
    // 错误处理/backoff
    ...
}
```

**标准分层结构**：
- informer（cache/shared informer）：监听 & 本地缓存资源对象
- workqueue：按 key(如 `namespace/name`) 排队，保证并发安全与速率控制
- syncHandler/reconcile：核心调谐业务，如 ReplicaSet 副本数对齐、Pod 创建或删除

---

## 3. Informer + WorkQueue 机制详解

- **Informer**（`client-go/tools/cache`）接管资源的 list/watch，内存本地缓存（Indexer）+回调事件
- **Controller** 注册 event handler，将变更对象 key 放入队列
- **WorkQueue** 支持速率限制（RateLimitingQueue）、去重、失败重试（backoff）
- **Reconcile Loop** 取 key 幂等执行一次调谐操作

这一模式已成为 controller-runtime、operator-sdk 等控制器/Operator 通用范式。

---

## 4. 典型内置控制器举例

- **Deployment/ReplicaSet/StatefulSet Controller**
  - 监听相关资源（如 Deployment、ReplicaSet、Pod）变更
  - 对比副本数量，欠缺则创建 Pod，多余则删除 Pod
  - 支持滚动升级、自动恢复等高级特性
- **Job/CronJob Controller**
  - 管理批量任务和定时调度
  - watch 任务状态，自动补充/重启
- **ServiceAccount/Token Controller**
  - 管理 ServiceAccount 及其 Token 自动发放
- **Node Controller**
  - 探测节点健康、处理 NotReady/回收孤儿 Pod

---

## 5. v1.34 关键代码变更（相较前版本）

- Informer 深度兼容 CRD 变化，第三方资源自定义控制器开发更易用
- Job/CronJob 控制器效率与稳定性优化，支持更大规模定时任务
- 内置控制器默认并发度和速率控制参数更合理（各 controller flags 调优）
- 对调和结果 metrics 采集增强，便于 SLO 可观测

---

## 6. 源码阅读与调试建议

- 推荐入口：`pkg/controller/`、`cmd/kube-controller-manager/`
- 熟悉 `client-go` informer、workqueue 使用（自定义 controller 也通用）
- 结合 `kubectl describe` 及 controller-manager 日志，对比资源变更与调和动作
- 关注 controller flags，如 `--concurrent-deployment-syncs`、`--node-monitor-grace-period` 调试并发与容错策略

---

## 7. 延伸阅读 & 重要参考

- [Kubernetes 官方 controller-core 设计](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-api-machinery/controllers.md)
- [client-go Informer/WorkQueue 机制](https://github.com/kubernetes/client-go/tree/master/tools/cache)
- [controller-runtime 项目](https://github.com/kubernetes-sigs/controller-runtime)
- [深入理解控制器模式-中文社区精选](https://jimmysong.io/kubernetes-handbook/concepts/controller.html)

---

> 总结：控制器是 K8s 自动化、弹性与自愈的基石。务必掌握同步主循环（Informer+Queue+Reconcile）三件套，结合实际调试与源码溯源，可高效理解和扩展所有 K8s 控制逻辑。

如需针对特定控制器/自定义 controller/operator 细节源码剖析，欢迎留言讨论！
