# Volcano 源码分析

Volcano 是 CNCF 生态重要的批量/高性能任务调度框架，专为 AI、深度学习及大规模企业计算场景设计。源于华为开源，兼容 Kubernetes 调度插件接口、支持 gang 调度、资源公平、作业生命周期管理等复杂场景。

---

## 1. 架构总览

Volcano 以 Scheduler 为核心，辅以 Job 控制器、Queue 管理等 CRD 扩展：

- **Scheduler**：自定义调度框架，核心二次开发入口
- **Controller**：负责 Job/Task CRD 主体编排、事件驱动
- **Admission**：Webhook 实现提交任务预处理
- **RestServer**：暴露作业管理、队列控制等 API
- **CRD**：Job/Queue/Command/PodGroup/PodDisruptionBudget 等核心对象

**主源码目录结构：**

- `cmd/volcano-scheduler/`       —— Scheduler 主进程
- `pkg/scheduler/`               —— 核心调度器代码、插件与策略
- `pkg/controllers/`             —— Job、Queue、PodGroup Controller
- `pkg/apis/`                    —— CRD 类型定义
- `pkg/webhooks/`                —— Admission 扩展
- `pkg/cli/`                     —— volcano job 等命令行
- `test/`                        —— 测试代码

---

## 2. 核心调度框架源码分析

### 2.1 Scheduler 流程主线

Scheduler 持续 watch pod/pg 事件，增量维护内存缓存，自定义多阶段调度：

- **入口代码**：`pkg/scheduler/scheduler.go`、`run()` 主循环
- **插件机制**：Filter/Score/Preempt/Binder 多阶段可插拔
- **队列公平**：核运行队列（Queue）优先分配策略
- **Gang 调度**：PodGroup 全体满足才调度，体现 batch 特色

调度 loop 主要步骤：

```
for {
  SyncClusterState()
  for each queue {
    Session.Open()
    plugins.RunPreFilter()
    plugins.RunFilter()
    plugins.RunScore()
    plugins.RunPreempt()
    Allocate()
    plugins.RunReserve()
    plugins.RunPermit()
    plugins.RunPreBind()
    plugins.RunBind()
    Session.Close()
  }
}
```

---

### 2.2 插件开发与典型插件

自带丰富插件（`pkg/scheduler/plugins/`）：

- **conformance**：资源约束校验
- **gang**：保证 PodGroup/gang 作业整体调度，要么全要么不要
- **priority/queue**：多租户/作业优先级控制
- **drf**：公平资源分配策略 Dominant Resource Fairness
- **numa/topology/coscheduling**：AI/大数据 hw 约束下特殊调度

**自定义插件**  
以 `plugin.Sample` 为例，只需实现 `OnSessionOpen/Close`、参与所需阶段接口，注册后自动加载。

---

### 2.3 Job/PodGroup CRD 生命周期

- 提交 Job（volcano job create） → 生成 PodGroup（和普通 Pod 判别）
- PodGroup 满足 minMember/资源门槛，进入调度（gang/elastic 场景编排）
- Controller 监控子 Pod 状态自动 patch Job Phase（Running/Succeed/Failed）

---

## 3. 二次开发扩展点

- **自定义调度策略/插件**：实现 DRF、公平、抢占等复杂业务约束
- **扩展 CRD**：支持新型批量任务语义/调度形态
- **Webhook Admission**：实现任务校验、默认值补齐等
- **队列租户/限流**：Queue 绑定租户、配额、多队列动态伸缩

---

## 4. 调试与开发建议

- 推荐单独部署 volcano-scheduler，开启 `--v=4` 详细日志
- 关注 `pkg/scheduler/framework/session.go` 每阶段触发点
- CRD 对象可通过 `kubectl get podgroup/job/queue` 追踪调度状态
- 插件开发用本地 `go run` 断点测试

---

## 5. 参考资料

- [Volcano 官方文档](https://volcano.sh/zh/docs/)
- [源码仓库](https://github.com/volcano-sh/volcano)
- [Gang 调度论文](https://www.cs.cornell.edu/home/kleinber/scheduling.pdf)
- [K8s Scheduler 扩展对比](https://kubernetes.io/zh-cn/docs/concepts/scheduling-eviction/scheduler-extensions/)

---

> 总结：Volcano 源码层面率先实现了批量/Gang/公平/多维约束等作业调度“超集”能力，插件化接口与 CRD 扩展链路便于对接 AI/大数据/企业 HPC 场景。聚焦 scheduler 主链和插件编排，是高效掌握其源码与二次开发的关键。如需插件开发、作业编排等详细解读，欢迎留言讨论！
