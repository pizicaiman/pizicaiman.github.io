# Kubernetes v1.34 apiserver 源码简要分析

Kubernetes apiserver 是 K8s 控制面的核心组件，负责所有 API 请求的接收、校验、授权、存储与分发。
以 v1.34 为例，核心源码分布于 `staging/src/k8s.io/apiserver/` 与 `cmd/kube-apiserver/`。

---

## 1. 启动流程与主入口

apiserver 主入口在 `cmd/kube-apiserver/apiserver.go`：

```go
func main() {
    command := app.NewAPIServerCommand()
    code := cli.Run(command)
    os.Exit(code)
}
```

- `NewAPIServerCommand()`完成参数解析，加载配置，并初始化完整 apiserver。
- 各类 Runtime 配置、认证、授权、存储后端、准入插件在初始化时装配。

---

## 2. HTTP Server 与路由注册

主 HTTP Server 实现在 `k8s.io/apiserver/pkg/server/genericapiserver.go`：

- `GenericAPIServer` 主体负责 HTTP 服务启动、路由注册。
- 支持聚合(apiserver aggregator)、动态热插拔 Webhook、准入控制等机制。

核心 server 启动逻辑：

```go
func (s *GenericAPIServer) PrepareRun() preparedGenericAPIServer {
    // 添加所有 API 路由、健康检查、OpenAPI、metrics、webhook 接口等
    // ...
    return preparedGenericAPIServer{...}
}
func (s preparedGenericAPIServer) Run(stopCh <-chan struct{}) error {
    // 实际启动 HTTP(s) 服务监听
}
```

---

## 3. 请求处理核心链路（Handler Chain）

请求从 kube-apiserver 入口到具体资源处理需经过完整的 handler chain，逻辑堆叠如下：

1. **认证(Authentication)**：
    - `pkg/authentication`，支持多种认证（Token, X509, Webhook, OIDC等）。

2. **授权(Authorization)**：
    - `pkg/authorization`，支持 RBAC、ABAC、Webhook 等。
    
3. **准入控制(Admission)**：
    - `pkg/admission`，以插件链形式依次处理（如 Namespace 存在性校验、资源配额等）。

4. **审计(Audit)**：
    - 审计中间件记录 API 请求。

5. **实际资源处理：**
    - 路由分发到各 GroupVersion 的 RESTStorage（通常注册在 `pkg/registry/`，如 pod 对应 `pkg/registry/core/pod`）。
    - controller-runtime (Informer/Reflector) 支持高效数据分发。

整体请求链搭建见 `pkg/server/handler.go` 的 `BuildHandlerChain`。

---

## 4. 对象存储与 CRUD 实现

apiserver 并不直接管理资源的存储，通常通过实现 Storage 接口，默认由 etcd 提供：

- 核心接口见 `pkg/registry/generic/registry/store.go` 中 `Store` 结构体和其 `Create`/`Update`/`Delete`/`List`等方法。
- 存储后端实现接口：`k8s.io/apiserver/pkg/storage/`，可支持 etcd, memory 等多种方式。

典型存储请求流程：

- HTTP 请求 → handler chain → RESTStorage → Store → etcd3（或其他后端）。
- 存储结构通常采用标准化的 `ObjectMeta` 统一管理。

---

## 5. 内部核心机制

- **Informer 和 Watch:**  
  `client-go` 内置高效本地缓存机制，实现高性能 API 订阅与事件分发。
- **聚合层支持:**  
  `apiservice` 资源可注册扩展 apiserver，自动聚合到主 apiserver 统一路由分发。
- **OpenAPI / 自发现:**  
  自动暴露 OpenAPI schema 及 API 资源发现机制，便于客户端动态适配。

---

## 6. v1.34 关键变更点举例（相较前版本）

- 更完整的优雅关停（Graceful Shutdown）与 HTTP 超时处理。
- Server-side Apply/FieldManager 优化，增强合并语义。
- Audit、准入插件的多样化扩展配套（如 CEL-based admission）。
- 支持更多认证/授权后端和信任代理场景。

---

## 7. 常见调试技巧

- `kubectl --v=8` 开启 API 详细追踪。
- 查看 apiserver 日志，grep `audit`、`admission` 等关键字。
- 使用 `/metrics`、`/healthz` 等接口实时监控健康及性能。
- 推荐结合代码阅读入口：
    - `cmd/kube-apiserver`
    - `pkg/registry/`、`pkg/apiserver/`
    - `staging/src/k8s.io/apiserver/pkg/server/`

---

> 总结：kube-apiserver 作为 K8s 核心 API 管道、凭借严密的 handler 链、动态的 CRD/聚合机制和高效本地缓存 watch 构建起强大的云原生 API 基石。建议结合实际需求从主流程与存储实现两条线索同时溯源，有针对性阅读与调试源码。

参考资料：

- [Kubernetes 官方 apiserver 设计文档](https://github.com/kubernetes/kubernetes/tree/master/staging/src/k8s.io/apiserver)
- [最新源码- GitHub](https://github.com/kubernetes/kubernetes/tree/master)
- [API Server 源码核心解读（社区精选）](https://jimmysong.io/kubernetes-handbook/architect/apiserver.html)

