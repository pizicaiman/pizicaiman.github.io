# Kubernetes kube-proxy v1.34 源码深度分析

---

## 1. 总览与架构定位

kube-proxy 是 K8s 集群节点实现 Service 网络代理（负载均衡/转发/网络编程）的核心组件。它运行在每个节点上，根据 apiserver 下发的 Service/Endpoints 变更事件，自动配置主机网络规则，实现集群内 Service 通信。

**三大工作模式：**

- userspace（极少用）：流量先到 kube-proxy，再转发到 Pod
- iptables（主流）：通过批量编程 iptables 规则，数据面全内核转发
- ipvs：内核级 LVS 负载均衡效率更高，推荐大集群

---

## 2. 主流程梳理

入口为 `cmd/kube-proxy/proxy.go → NewProxyCommand → Run`，主要流程如下：

1. 解析配置参数、判定代理模式（iptables/ipvs/others）
2. 启动 informer 监听 Service、Endpoints(或 EndpointsSlice) 变化
3. 事件驱动同步规则到内核（iptables/ipvs/...）

简化源码主流程：

```go
func main() {
    command := app.NewProxyCommand()
    command.Execute()
}

// app/server.go
func NewProxyCommand() *cobra.Command {
    // 解析配置，启动 Run
}

func (o *Options) Run() error {
    proxyServer, _ := NewProxyServer(o)
    proxyServer.Run()
}

// app/server_others.go
func (s *ProxyServer) Run() {
    go s.ServiceConfig.Run()
    go s.EndpointsConfig.Run()
    // 事件监听，定期 sync rules
    s.Proxier.SyncLoop()
}
```

---

## 3. 关键数据流与对象说明

- **Service/Event Informer**
  - 监听和缓存所有 Service/Endpoints/EndpointSlice
- **Proxier**
  - IptablesProxier、IPVSProxier 等策略分发
  - 逐步感知资源变化，构建转发表

### 数据关联链

Service/Endpoints 更新 → informer 通知 Proxier → 生成目标规则（iptables/ipvs）→ 写入内核

---

## 4. iptables 模式源码要点

- 实现位于 `pkg/proxy/iptables/proxier.go`
- **syncProxyRules 主循环**：感知 Service/Endpoints 变化后，重新写全量规则
- 大量使用 `github.com/coreos/go-iptables` 进行规则管理，效率高
- 起核心表 chain 如：`KUBE-SERVICES`, `KUBE-SEP-xxxx`, `KUBE-SVC-xxxx`

简要主流程：

```go
func (proxier *Proxier) syncProxyRules() {
    // 1. 读取缓存 Service/Endpoints
    // 2. 构建需要的 iptables chain, rules
    // 3. 应用到系统内核 iptables
}
```

**优化点与陷阱：**
- 批量写入减少抖动
- rules sync 队列优化（v1.34 引入 fast sync, 适应大集群全局更新）

---

## 5. IPVS 模式源码要点

- 入口 `pkg/proxy/ipvs/proxier.go`
- 基于内核 `/proc/net/ip_vs*` 文件系统直接管理 LVS 虚拟服务
- 规模大（1w+ Service/Endpoint）时性能明显优于 iptables
- 支持 sessionAffinity/DSR/多种调度算法

主流程：

```go
func (proxier *Proxier) syncProxyRules() {
    // 1. Service/Endpoints 变化后，算出 LVS 期望转发表
    // 2. 调用 ipvs 操作接口创建/删除虚拟服务和真实后端
}
```

---

## 6. v1.34 关键变更与新特性

- EndpointsSlice informer/数据结构全面默认化（提升大集群性能）
- iptables、ipvs Proxier 支持更加异步/增量 sync，减少全量刷写
- healthz/readiness probe 机制增强，支持批量 Service 状态实时检测
- 更完善的 `conntrack` 清理优化，避免大规模端口漂移累积内核垃圾
- 动态 Proxier Metrics/事件追踪接入 K8s 原生监控体系

---

## 7. 调试分析与实践建议

- 参数调优关注：`--proxy-mode`，`--iptables-sync-period`, `--ipvs-sync-period`
- 查看现有 rules:
  - iptables: `iptables-save | grep KUBE`
  - ipvs: `ipvsadm -Ln`
- 日志追查：`-v=4` 以上可看到规则变更详细日志
- 排查连接追溯：`conntrack -L | grep <svc/ip>`
- 源码追踪建议入口：
    - `pkg/proxy/service.go`
    - `pkg/proxy/iptables/`
    - `pkg/proxy/ipvs/`

---

## 8. 参考资料

- [Kube-proxy 官方架构文档](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/)
- [kubernetes/kubernetes/pkg/proxy/ 源码](https://github.com/kubernetes/kubernetes/tree/master/pkg/proxy)
- [K8s v1.34 Release Notes](https://github.com/kubernetes/kubernetes/releases/tag/v1.34.0)
- [Service & kube-proxy 深度剖析](https://jimmysong.io/kubernetes-handbook/network/service.html)

---

> 总结：kube-proxy 是 Service 网络背后的内核规则引擎（iptables/ipvs/ebpf），源码以全事件驱动为核心，关注 Proxier 多实现与规则增量刷写，尤其 v1.34 重点在大规模集群性能、事件感知、易用性等多方面同步优化。结合调试工具与源码多主线分析，可快速定位代理异常与网络策略。

如需具体模式/数据结构/调试用例等更详细源码剖析，欢迎留言讨论！
