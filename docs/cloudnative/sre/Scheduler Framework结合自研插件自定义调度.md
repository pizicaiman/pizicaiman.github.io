# Scheduler Framework 结合自研插件自定义调度

Kubernetes Scheduler Framework 自 v1.19 起提供了强大的调度插件链机制，支持多阶段定制扩展，极大提升了原生调度能力。通过自研插件，可以灵活实现业务自定义调度策略、资源约束、优先级、拓扑、能耗等诉求，而无需 patch 或完全重写 scheduler。本文结合实战，详细介绍如何基于 Scheduler Framework 定制插件，打造适配业务场景的调度器。

---

## 1. Scheduler Framework 插件模型概览

- **插件加载链路**：`PreFilter` → `Filter` → `PreScore` → `Score` → `Reserve` → `Permit` → `PreBind` → `Bind` 每阶段可注册一个或多个插件。
- **插件类型**：
    - **Filter**：节点过滤，比如资源、拓扑、地域等
    - **Score**：节点打分、优选排序
    - **Preemption**：优先级抢占
    - **Reserve、Permit**：抢占、异步校验、预留
    - **Bind**：自定义 Pod 绑定逻辑
- 插件间通过 `framework.Handle` 共享上下文及辅助 API

---

## 2. 自定义插件开发流程

1. **实现对应接口**
   - 以自定义节点优选（Score）为例，需实现 `ScorePlugin` 接口：
     ```go
     type MyScorePlugin struct {}
     func (p *MyScorePlugin) Name() string { return "MyScore" }
     func (p *MyScorePlugin) Score(ctx context.Context, state *framework.CycleState, pod *v1.Pod, nodeName string) (int64, *framework.Status) {
         // 业务自定义打分逻辑
         return score, framework.NewStatus(framework.Success, "")
     }
     func (p *MyScorePlugin) ScoreExtensions() framework.ScoreExtensions { return nil }
     ```
   
2. **插件注册**
   - 实现 `Init()` 并注册到插件工厂 map
   - 通过 config.yaml（或代码）指定加载顺序、参数
   - 例：`pkg/scheduler/framework/plugins/yourplugin/myplugin.go`

3. **加载与调试**
   - 配置调度器使用自定义配置文件，指定插件链顺序、Weight
   - 日志埋点辅助调试，`--v=5` 打开详细日志

---

## 3. 插件链编排与多插件协同

- 可组合多个插件（官方预置+自研），有序执行
    - 例：`Filter` 阶段先官方资源限制，再自研拓扑亲和
    - `Score` 阶段多插件打分，通过 `Weight` 权重配比
- 支持动态热插拔、参数热更新（v1.23+）
- 插件间可共享 state/cache 提升效率

---

## 4. 典型自研插件应用场景举例

- **GPU/NUMA 特殊调度**：按 Pod/Node 拓扑感知筛选/打分
- **业务亲和/反亲和调度**：自研 Label、AppID 聚合分布限制
- **作业调度/批量扩展**：实现 Gang/Coscheduling、批量抢占
- **能耗/性能优化**：按节点能耗/温度动态选择最优
- **异构集群混部**：多租户资源隔离及策略调度

---

## 5. 实战经验与源码阅读指引

- 推荐插件开发入口：
    - `pkg/scheduler/framework/plugins/sample/`
    - `cmd/kube-scheduler/app/config.go`（插件链配置点）
    - `pkg/scheduler/framework/runtime/`（插件注册、调用主链）
- 日志与 trace 捕获调度全过程
- 建议贴合实际业务关键：如批量作业、GPU 调度、数据感知伸缩等点

---

## 6. 参考资料

- [Kubernetes Scheduler Framework 插件文档](https://kubernetes.io/zh-cn/docs/concepts/scheduling-eviction/scheduler-extensions/)
- [Scheduler Framework 插件开发实战](https://jasonwzx.github.io/2023/01/09/Kubernetes-Scheduler-Framework/)
- [koordinator/volcano 等调度器插件源码](https://github.com/koordinator-sh/koordinator) / [https://github.com/volcano-sh/volcano](https://github.com/volcano-sh/volcano)

---

> 结语：充分利用 Scheduler Framework 插件化架构，既可无缝接入官方预置能力，又能通过自研插件快速适配复杂/新兴业务调度需求，是云原生调度解决方案演进的核心。欢迎交流实际插件设计实现、疑难杂症与性能优化经验！

---
## 7. 自定义调度器实现示例：基于 CPU/内存利用率的 Filter + Score 插件

### 7.1 简单自研插件核心代码片段

以 Go 实现调度器插件为例，实现一个自定义 Scheduler、指定名称，并用 CPU 利用率“越小越优”做打分和过滤：

```go
// 1. 自定义插件结构体，注册到调度器插件链
type LowCpuScheduler struct{}

func (pl *LowCpuScheduler) Name() string { return "LowCpuScheduler" }

// Filter 阶段，只允许CPU利用率低于阈值的节点
func (pl *LowCpuScheduler) Filter(ctx context.Context, state *framework.CycleState, pod *v1.Pod, nodeInfo *framework.NodeInfo) *framework.Status {
    cpuUsage := getNodeCPUUsage(nodeInfo) // 伪函数，获取节点当前CPU利用率
    if cpuUsage > 0.8 {  // 例如: 利用率大于80%则过滤
        return framework.NewStatus(framework.Unschedulable, "cpu usage too high")
    }
    return nil // 通过
}

// Score 阶段，CPU利用率越低分数越高
func (pl *LowCpuScheduler) Score(ctx context.Context, state *framework.CycleState, pod *v1.Pod, nodeName string) (int32, *framework.Status) {
    cpuUsage := getNodeCPUUsageByName(nodeName) // 伪函数
    // 假定分数范围0~100，利用率0分数100，利用率100%分数0
    score := int32((1.0 - cpuUsage) * 100)
    return score, nil
}

func (pl *LowCpuScheduler) ScoreExtensions() framework.ScoreExtensions {
    return nil
}

// 注册插件
framework.RegisterPlugin(&LowCpuScheduler{})
```

> 注：`getNodeCPUUsage` 需根据节点监控信息/metrics 实现获取。

### 7.2 注册自定义调度器并绑定

调度组件支持多调度器运行。Pod 通过 `spec.schedulerName` 指定：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-workload
spec:
  schedulerName: lowcpu-scheduler    # 对应启动的自定义调度器名称
  containers:
    - name: ...
      image: ...
```

调度器启动时自定义：

```shell
kube-scheduler --scheduler-name=lowcpu-scheduler --config=your-config.yaml
```

### 7.3 配置插件链（config 示例）

```yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
- schedulerName: lowcpu-scheduler
  plugins:
    filter:
      enabled:
      - name: LowCpuScheduler
    score:
      enabled:
      - name: LowCpuScheduler
```

---

> 通过指定 schedulerName、插件注册名和定制 Filter + Score 逻辑，便可服务化实现“低 CPU/内存利用率优先调度”的生产场景。

---

**答：**  
不会影响原集群默认调度组件，风险极低。

原因如下：
- 多调度器机制是 Kubernetes 原生支持的。自定义调度器（如 `lowcpu-scheduler`）与默认调度器（如 `default-scheduler`）可在同一集群共存。
- 只有 `spec.schedulerName` 明确为 `lowcpu-scheduler` 的 Pod，才会被你自定义的调度器调度。其他未指定或指定为 `default-scheduler` 的资源，仍然由默认调度器处理，互不干扰。
- 插件链配置、插件注册等，均作用于你自定义调度器的进程和 config，不会篡改或影响默认调度器的行为。
- **注意**：仅当你误将全局 KubeScheduler 的参数覆盖、或所有 Pod 的 `schedulerName` 改为自定义调度器，才会存在风险。正常按上面步骤单独注册、运行、指定，不会影响集群原有调度。

> 建议前期可只测试小量、特定 schedulerName 的 pod，验证定制插件调度行为，确保与集群业务解耦。

---

### 7.4 多级调度器原理及节点崩溃风险分析

#### 1. 多级调度器实现原理

Kubernetes 多级（多实例）调度器机制，是通过支持运行多个调度器实例，并结合 `Pod.spec.schedulerName` 字段完成的：

- **核心机制**
  - 多个 kube-scheduler 进程可在同一集群分别运行，各自用不同的 `--scheduler-name`（或配置文件中的 `profiles.schedulerName`）参数启动。
  - 调度器会自动 Watch 所有未 `spec.nodeName` 绑定的 Pod，仅调度 `schedulerName` 字段与自己名字匹配的 Pod。
  - 普通业务 Pod 默认 `schedulerName=default-scheduler`，如需让 Pod 只被自定义调度器管控，需手动指定 `spec.schedulerName` 为自定义调度器名称。
- **调度器之间互不抢占任务**，不会重复调度同一个 Pod。
- 多调度器适合于不同租户、队列、或者不同业务需求下的隔离化调度需求。

#### 2. 多调度器对节点崩溃风险影响

- **不会增加节点崩溃风险**：
    - 调度器是 Kubernetes 控制面组件，仅负责“决定”Pod 应该被分配到哪个节点，并不会直接干预节点的内核进程或硬件资源。
    - 多个调度器的出现，并不会直接在节点侧产生额外负载、或让节点更易崩溃。

- **自动/默认调度器安全性**：
    - 默认调度器及其插件链一般经过大量测试和生产验证，能够确保较高的稳定性。
    - 多实例调度器相互独立，常规情况下互不影响。只要每个实例正确过滤、处理各自 `schedulerName` 的 Pod，即不会出现调度“串用”或资源干扰风险。

- **出现节点资源压力的情形**：
    - 如果调度器插件链编写不严谨，例如忽略节点已分配资源、资源配额、Taint/Toleration 等，**有可能导致过量调度**，间接出现节点资源超载、OOM、性能下降等问题（无论是单实例还是多实例）。但这属于调度策略/规则配置不当，而非多调度器机制本身带来的风险。
    - 正确实现的调度器（无论是默认还是自定义），会保障节点分配安全，不会额外提升节点崩溃概率。

#### 3. 最佳实践

- 每个调度器保持独立 config 与插件链设计，**严防多个调度器“争抢”同一 Pod**。
- 插件链开发需充分校验节点实际可用资源，并加固排错、backoff、事件反馈机制，避免不合理分配。
- 推荐单元测试/灰度测试，观测调度决策和节点压力变化，确保多实例协同安全性。

---

> 总结：Kubernetes 多级调度器通过 schedulerName 实现资源隔离与分工，不影响节点稳定性。调度安全性核心在于合理插件链设计与资源检查，默认调度器和自定义调度器本身不会提升节点崩溃风险。


