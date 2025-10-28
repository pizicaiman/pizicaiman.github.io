# Kubernetes ingress-nginx 控制器源码深度分析

---

## 1. 项目定位与架构简介

**ingress-nginx** 是 K8s 社区官方维护的 NGINX-based Ingress Controller，实现了 Ingress 资源的多租户 HTTP/HTTPS 七层代理。其通过监听 Ingress/Service/Endpoint/ConfigMap/Secret 等资源，自动生成并热刷新 nginx 配置，实现灰度、TLS、rewrite、canary 等多样流量管理能力。

**核心架构：**

- **Controller Pod**（Go实现）：watch K8s 资源，组织并重载 nginx 配置
- **NGINX Worker Pod**（同 Pod）：实际网络流量代理与转发
- **热加载通道**：Controller 实时与 nginx worker 交互，完成零停机 reload

---

## 2. 核心源码主流程

Controller 的主入口及循环在 `cmd/controller/main.go`：

```go
func main() {
    // 初始化 Controller，解析配置
    k8sController := ingress.NewController(...)
    // 启动主循环
    k8sController.Start()
}
```

### 2.1 资源监听与事件驱动

利用 `k8s.io/client-go` informer 机制监听 Ingress、Service 等：

```go
func (ic *Controller) Start() {
    ...
    // informer 启动，自动 watch 资源，推进 queue
    go ic.syncQueue.Run(stopCh)
    ...
}
```

### 2.2 配置生成与同步

事件变化时统一推进 `syncQueue`，核心调和逻辑：

```go
func (ic *Controller) syncIngress(key string) error {
    // 1. 汇总全部 Ingress/Backend 当前状态
    ings := ic.store.ListIngresses()
    ...
    // 2. 合成 NGINX config template
    config := ic.generateNginxCfg(ings, ...)
    // 3. 热更新 NGINX
    ic.reloadNginx(config)
}
```

- `generateNginxCfg` 负责合并路由、证书、rewrite、annotation 等多维规则
- `reloadNginx` 支持 config test 及失败回滚，重载 NGINX worker

---

## 3. 关键模块剖析

| 模块                   | 主要职责                                               |
|------------------------|--------------------------------------------------------|
| store/                 | K8s 资源缓存与索引，支持 Lister/Indexer                |
| backend/               | NGINX 配置模板构建/变量替换                             |
| controller/            | 事件循环、reload、健康探测、metrics                     |
| nginx/nginx.go         | 启动、热加载、优雅重启 nginx 进程                       |
| config/annotations.go  | 解析 Ingress annotation 丰富相关功能                    |

---

## 4. v1.9+ 关键特性与源码变动

- 支持 dynamic backend reload，极快配置刷新无中断
- annotation 可扩展性强化，支持 WAF/canary/limit/cors/自定义 header 等
- 支持自定义 Lua/3rd 模块提升可编程能力
- 兼容 IngressClass/v1 资源（新版 API 支持）

---

## 5. 调试与源码分析建议

- **主调试参数：** `--v=5+` 可打印关联资源事件
- **入口文件：**
    - `cmd/controller/`（启动流程）
    - `internal/ingress/controller.go`（主业务线）
    - `internal/ingress/backend/`（nginx模板系统）
    - `internal/ingress/store/`（缓存与索引）
- **实用命令：**
    - 检查 nginx 配置 `/etc/nginx/nginx.conf`
    - 热加载 nginx: `nginx -s reload`
    - metrics/探针：参看 `/metrics` `/healthz` 路径

- **典型调试流程：**
    - kubectl describe ingress 查看资源已同步到 controller
    - controller pod 日志确认事件/变更生效
    - 验证 nginx 配置已按预期变动，request 流量正常分发

---

## 6. 参考资料

- [ingress-nginx 源码](https://github.com/kubernetes/ingress-nginx)
- [官方使用文档](https://kubernetes.github.io/ingress-nginx/)
- [In-Depth: How ingress-nginx Works](https://sysdig.com/blog/ingress-controller/)
- [Kubernetes Ingress 官方文档](https://kubernetes.io/zh-cn/docs/concepts/services-networking/ingress/)
- [主循环与事件源码分析](https://jimmysong.io/kubernetes-handbook/network/ingress.html)

---

> ingress-nginx 是 K8s 七层流量入口的事实标准与最佳实践，其源码清晰、插件与模板机制灵活。掌握资源事件驱动、模板配置自动化与高可用热加载原理，可助你快速定制与排障生产环境 Ingress 控制器。

如需具体模块、配置热加载或自定义扩展源码深度解读，欢迎留言交流！
