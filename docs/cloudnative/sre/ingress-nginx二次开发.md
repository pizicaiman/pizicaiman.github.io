# ingress-nginx 二次开发实践

---

## 1. 需求场景分析

ingress-nginx 支持大量 annotation 及全方位自定义，但在复杂业务下，经常有如下二次开发需求：

- 注入企业自定义认证/鉴权/审计逻辑至 nginx 配置
- 扩展 annotation 支持（如自定义 header、QPS 限流等场景）
- 增强 WAF、安全策略、流量调度等接入
- 热加载/无损 reload 新特性适配业务场景

---

## 2. 核心二次开发能力分类

| 方向              | 推荐方式                        | 实践说明                                         |
|-------------------|--------------------------------|--------------------------------------------------|
| 自定义 Annotation | 修改 `internal/ingress/config/annotations.go` 并注册      | 新加或扩展 Annotation，需补 schema 校验、文档注释 |
| 配置模板扩展      | 修改 `internal/ingress/backend/template/nginx.tmpl`        | NGINX location/header 变动等典型场景             |
| 事件处理增强      | 扩展 `internal/ingress/controller/` 相关事件/探针逻辑      | 事件驱动自动 reload/自定义状态上报                |
| 健康探针/审计     | 注入日志、metrics、APIGW 鉴权等流程                        | 配合 Prometheus/Lua/外部系统扩展                  |
| 热加载/多实例优化 | 适配新版 nginx reload、边车扩展等                          | 动态变更、弹性扩容、故障转移                      |

---

## 3. 典型扩展流程

### 3.1 添加自定义 Annotation 示例

1. **定义 Annotation 结构体**  
   在 `internal/ingress/config/annotations.go` 新增解析和默认值逻辑。

   ```go
   // 以 X-My-Header 注解为例
   type MyHeader struct {
       Value string `json:"value"`
   }
   ```

2. **解析注解**
   ```go
   func (a *MyHeader) Parse(ing *networking.Ingress) error {
       v := util.GetStringAnnotation("nginx.ingress.kubernetes.io/x-my-header", ing)
       if v != "" {
           a.Value = v
       }
       return nil
   }
   ```

3. **注册到 AnnotationParser 映射表**

4. **模板应用**  
   在 `nginx.tmpl` 中 {{ .MyHeader.Value }} 注入 target header 配置。

5. **贡献/维护文档 Schema**  
   注明可选项和安全约束。

---

### 3.2 进阶开发建议

- 推荐优先通过注解机制满足大多数场景，尽量避免 hardcode NGINX 配置
- 编写完善单元/integration 测试（见 `test/e2e/` 目录）
- 生产环境务必关注 reload 行为与 config 热备份
- 推荐二次开发同步 PR 至官方社区，便于升级/维护

---

## 4. 热加载/无损扩容相关优化

- 配合 v1.9+ 动态 backend reload，支持大规模规则变动无中断服务
- 结合 readiness/liveness 探针可动态感知各实例健康
- 多 AZ/多实例时首选 WorkloadController 自动滚动扩容

---

## 5. 参考资料

- [ingress-nginx 源码仓库](https://github.com/kubernetes/ingress-nginx)
- [官方 Annotation 文档](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/)
- [主循环源码与template机制社区解读](https://jimmysong.io/kubernetes-handbook/network/ingress.html)

---

> 总结：ingress-nginx 具备极强的“软继承”与注解自定义能力，绝大多数企业扩展需求都可通过二次开发 Annotation、模板与控制器模块实现。善用项目源码结构与事件驱动主线能力，可高效定制和排障大规模生产流量入口。如果有特定插件/template扩展需求，建议结合测试与社区贡献同步推进，以保障后续升级的可维护性。

如需针对 annotation/plugin/controller 任一链路源码定制更详细解读，欢迎留言交流！

