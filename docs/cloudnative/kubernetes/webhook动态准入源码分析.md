# webhook 动态准入源码分析

Kubernetes 的 Admission Webhook 机制是动态扩展准入控制（Admission Control）的核心实现方式，它允许用户自定义集群对象在创建/更新时的校验和默认化逻辑。Webook 由两类组成：

- **ValidatingAdmissionWebhook**（验证型）：拦截并判断资源是否允许请求。
- **MutatingAdmissionWebhook**（变更型）：可对资源做默认值补齐及动态变更。

本分析以 `k8s.io/apiserver/pkg/admission/plugin/webhook` 和 `k8s.io/apiserver/pkg/admission/plugin/webhook/validating`/`mutating` 源码为主，拆解调用主线、异步调用及安全策略。

---

## 1. 准入主链 & 注册流程

### 触发入口

以验证型 Webhook (`ValidatingAdmissionWebhook`) 为例：

- 注册于 apiserver 启动阶段（flags/配置文件驱动）。
- 构建 Launch Sequence 时从配置拉取 webhook 配置，装入 `admission.ChainAdmissionHandler`。

**主入口：**
```go
func (a *ValidatingAdmissionWebhook) Admit(ctx context.Context, attr admission.Attributes, out runtime.Object) (err error)
```
入参为当前请求的对象、操作类型、用户等。

### 配置加载

- 配置来源一致（CRD：`ValidatingWebhookConfiguration`/`MutatingWebhookConfiguration`）。
- apiserver 内通过 informer/SharedIndex cache 动态监听这些配置。

---

## 2. 调用链详解

### 2.1 触发逻辑

当集群发生资源创建/修改等操作时：

1. 触发对应 admission handler
2. 遍历配置的 webhooks，按 rules 匹配
3. 异步并发构造 admissionReview 请求，发往 webhook 服务端点

主要代码位于 `dispatchWebhook`、`shouldCallHook`、`callHook` 等函数：

**dispatchWebhook 主流程示例：**
```go
for webhook := range relevantWebhooks {
  go func() {
    client := webhookClientManager.HookClient()
    admissionReview := buildAdmissionReview(attr)
    resp, err := client.Do(admissionReview)
    // 超时、失败处理
    // 解析 response，做 allow/deny
  }
}
```

### 2.2 匹配 & 策略控制

- 根据 `namespaceSelector`、`objectSelector`、`operations` 等字段精准控制生效资源和事件。
- 支持 `failurePolicy`：如 Ignore/Fail，决定 webhook 无法访问时请求行为。

---

## 3. 并发与超时机制

- 多个 webhook 可并发请求，通过 context 设置超时时间。
- 若有一些 webhook 响应慢/失败，`failurePolicy`=Ignore 不会影响主请求，否则拒绝。
- 支持缓存加速、不影响主 apiserver QPS。

---

## 4. Webhook 服务要求

- 必须提供 HTTPS，证书受 apiserver 驱信（可用 CA Bundle 指定）。
- 路径约定 `/mutate` `/validate`
- request/response 严格 AdmissionReview 格式

**示例结构：**
```go
type AdmissionReview struct {
  Request  *AdmissionRequest
  Response *AdmissionResponse
}
```

---

## 5. 开发要点与二次扩展建议

- 推荐用 controller-runtime/webhook SDK 快速实现簇内外部 webhook。
- 多 webhook 顺序执行，mutating 可层层叠加修改，validating 全部通过才放行。
- 可配合自定义 selector、条件化触发，适应复杂场景，如多租户、资源准入治理。

---

## 6. 参考资料与最佳实践

- [Kubernetes Admission Webhook 官方文档/流程图](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/extensible-admission-controllers/)
- [apiserver webhook 源码入口](https://github.com/kubernetes/kubernetes/tree/master/staging/src/k8s.io/apiserver/pkg/admission/plugin/webhook)
- [controller-runtime webhook 实战](https://github.com/kubernetes-sigs/controller-runtime/tree/master/pkg/webhook)

---

> 总结：动态准入 webhook 实现灵活、安全、强扩展的资源准入链。掌握其注册-分发-异步-容错主线流程，可快速开发企业定制校验与变更能力。如需 webhook 自定义开发/调优及源码追踪，欢迎留言交流。
