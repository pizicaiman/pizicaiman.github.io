# Kourier 源码分析

Kourier 是 Knative 官方主推的轻量级 Ingress 网关实现，专为 Knative Serving 优化。其架构简洁，仅依赖 Kubernetes CRD 机制和 Envoy Proxy，聚焦于无 Serverless 场景下高性能、高扩展性的北向流量接入。

---

## 1. 架构简介

- **核心组件：**
  - `kourier-controller`：K8s 控制器，监听 Knative Ingress 资源，自动生成 Envoy 配置（via CRD）
  - `kourier-gateway`：嵌入 Envoy，作为实际的数据面入口，对外暴露流量
- **数据流：**
  1. 用户提交 Knative Service/Route
  2. Knative Serving 控制器生成 Ingress CR（networking.internal.knative.dev Ingress）
  3. Kourier Controller Watch/Sync Ingress CR 生成 Envoy 配置
  4. Kourier Gateway 热加载新路由，承载实际流量

![](https://github.com/knative/net-kourier/raw/main/docs/kourier-overview.png)

---

## 2. 源码主线与关键模块

Kourier 主要源码仓库：[https://github.com/knative/net-kourier](https://github.com/knative/net-kourier)

### 2.1 Controller 主循环

- **入口：** `cmd/kourier/main.go`、`./pkg/reconciler/ingress/`
- **工作方式：**
  - 使用标准 Controller Runtime 监听 `Ingress`（Knative CRD）
  - 调用 `Reconcile()` 协调 Envoy 配置
  - 生成 Envoy config via `pkg/envoy/config_translator.go`，通过 ConfigMap/CRD 触发 gateway reload

**核心代码示例：**
```go
func (r *Reconciler) ReconcileKind(ctx context.Context, ing *v1alpha1.Ingress) reconciler.Event {
    // 1. 解析 Ingress 规则
    // 2. 生成 Envoy 路由/Cluster
    // 3. 更新 ConfigMap 触发 Kourier Gateway reload
}
```

### 2.2 Envoy 配置生成

- **模块：** `pkg/envoy/`
  - `config_translator.go`：Ingress -> Envoy Listener/Route/Cluster
  - `resources.go`：Envoy xDS 配置片段构造
- **特点：** 所有配置原子渲染，热更新，无需重启 Envoy

### 2.3 Gateway 数据面

- 启动参数中加载配置，暴露 80/443 端口
- 从 Controller 自动 watch/config 更新配置

---

## 3. 设计亮点与二次开发典型场景

- 极简接入，零外部依赖，仅依赖 Envoy
- 完全解耦 Knative Serving 与网络层，支持多种 Serving 场景接入
- 支持标准 K8s Service/Ingress，适配多云网络架构
- 开发者可通过扩展 config_translator 插件，实现自定义 Envoy 插件、自定义路由头等
- 可集成 JWT、mTLS、WebAssembly 等高级 Envoy 功能，通过配置扩展

---

## 4. 调试与实践建议

- 启动时 `--log-level debug` 可跟踪配置同步过程
- 常见问题可通过 Controller 日志 + Envoy stats/metrics 排查
- 变更配置建议观察 `/config_dump` 与 xDS event，追踪 Envoy 路由生效
- 支持本地 kind/minikube 环境快速上手
- 适用于低资源 Serverless Gateway、高性能南北向流量接入等场景

---

## 5. 参考资料与源码解析

- [Kourier 官方文档](https://knative.dev/docs/serving/using-a-custom-ingress-gateway/)
- [Kourier/Knative 网络架构解读](https://github.com/knative/net-kourier)
- [核心控制器与 Envoy 配置生成逻辑](https://github.com/knative/net-kourier/tree/main/pkg)
- [深入 Serving 数据面链路分析](https://jimmysong.io/kubernetes-handbook/serving/knative-serving.html)

---

> 总结：Kourier 源码清晰、定位单一、易于集成与二次开发。理解 K8s CRD、Knative Ingress 与 Envoy xDS 配置生成主线，可高效定制 Serverless 网络接入与网关能力。如需深入插件开发、自定义安全/认证，请关注 config_translator 相关模块。如有定制问题，欢迎留言交流！

