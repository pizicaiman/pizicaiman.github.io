# ingress-nginx路由优先级探索与实践

Kubernetes 的 Ingress 允许我们通过声明式的方式配置 HTTP 路由，将外部流量按规则转发到后端服务。ingress-nginx 作为主流的 Ingress Controller，其路由规则的优先级和匹配实践是众多开发者关注的重点。

## 路由匹配优先级规则

ingress-nginx 的路由优先级遵循 [RFC 2616](https://datatracker.ietf.org/doc/html/rfc2616) 标准，优先级从高到低如下：

1. **基于 Host 的精确匹配**
2. **基于 Path 的精确匹配（Exact Path；如 `/foo`）**
3. **基于 Path 的前缀匹配（Prefix Path；如 `/foo/*`）**
4. **正则匹配**

同一个 Host 下，路径越具体（越长/越精确）的规则优先级越高。

## 路由优先级实践示例

假如我们有如下 Ingress 规则：

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: demo-ingress
spec:
  rules:
  - host: "example.com"
    http:
      paths:
      - path: /api/v1/user
        pathType: Exact
        backend:
          service:
            name: user-service
            port:
              number: 80
      - path: /api/v1
        pathType: Prefix
        backend:
          service:
            name: v1-service
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
```

路由匹配优先级为：
1. 访问 `example.com/api/v1/user` 会匹配到 `user-service` （精确路径优先）
2. 访问 `example.com/api/v1/anything`（除 `/user` 以外）会匹配到 `v1-service`
3. 访问 `example.com/api/xxx` 会匹配到 `api-service`

## 路由优先级可能踩的坑

- 路径写法不一致如 `/api/` 和 `/api`，可能导致实际匹配出现偏差，建议统一规范路径写法。
- 多个 Ingress 资源定义了重叠路径时，以 nginx 合并后的 server 配置为准，实际优先级取决于合并逻辑。
- pathType 不同（Exact vs Prefix）时，优先级不同。

## 优先级查看与验证

可以通过 `kubectl describe ingress` 查看所有生效的规则；或通过 ingress-nginx 的 debug 界面 (`http://<ingress-nginx-ip>:10254/nginx/server_template` 或 ConfigMap 日志）查看实际下发到 nginx 的配置。

## 实践建议

- 优先使用精准的路径和 Host 配置。
- 尽量避免多个覆盖性 Ingress 分布定义相似或重叠的路径。
- 使用 `pathType: Exact` 明确指明精确路由，避免优先级陷阱。

## 参考资料

- [ingress-nginx 官方文档: Path types](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/#path-types)
- [Ingress 路由优先级说明](https://github.com/kubernetes/ingress-nginx/issues/7399)


如果前后端分开，常见场景是前端服务（如 web-frontend）和后端接口（如 api-backend）分别有各自的 Ingress 规则。这时路由优先级的判断、查看与验证涉及以下几个关键点：

### 1. 优先级判断原则

- **同一个 Ingress 内部规则**：依然遵循 Exact > Prefix（长度更长优先） > 更短规则的顺序。
- **多个 Ingress 之间**：nginx ingress controller 会将所有 Ingress 资源合并生成唯一的 nginx 配置。优先级还是取决于 path 的类型与长度，与 Ingress 归属无关。
- **host 匹配**：不同 ingress 配置不同 host 时，先分流到 host，再根据 path 决定优先级。

### 2. 优先级查看方法

- 使用 `kubectl describe ingress <NAME>`，分别查看前端和后端所有 Ingress 资源下的完整规则。
- 更直观的方式：获取合并后的 nginx 实际配置。
    - 方法一：访问 ingress-nginx 的 debug 接口，例如  
      ```
      curl http://<ingress-nginx-pod-ip>:10254/nginx/server_template
      ```
      或最新版 controller 用 `/nginx/ingresses` 接口。
    - 方法二：进入 ingress-nginx pod 内，查看 `/etc/nginx/nginx.conf` 或 `/etc/nginx/conf.d/` 下的自动生成文件。
- 在这些文件中，可以清楚看到所有 Host、location（路径）、转发顺序，能直观验证最终生效的优先级与匹配逻辑。

### 3. 验证建议

- 最直接方式是实际访问测试（如 curl 请求），确认各路径会转发到预期的服务。
- 也可在 ingress controller 日志中查看访问路径路由到的 backend 服务名，进一步验证优先级。

### 4. 实践避坑建议

- 如果路径规则有重叠（如 `/api` 和 `/api/v1`），即使分属不同 ingress，也应明确 pathType、Host，避免意外覆盖。
- 分离管理的 ingress 建议约定明确的前缀/host，避免引发不可预期的优先级混淆。

**总结**：无论一个还是多个 Ingress，路由优先级最终由 ingress-nginx 合并所有规则后按统一规范决定。建议关键场景配合 debug 跟踪 nginx 配置与实际请求验证，避免掉入“以为分开就不冲突”的误区。


### 多个 Ingress 路由如何确认优先级

在实际生产场景中，常见需求是不同团队或应用通过多个 Ingress 资源分别管理路由（比如 frontend-vs-backend、各独立业务线分别部署自己的 ingress.yaml）。这会引发一个核心问题：**多个 Ingress 下的规则是怎么“合并”以及如何确认最终优先级**？

**核心原理：**
- ingress-nginx controller 会自动收集（Watch）整个命名空间（或全集群）的所有 Ingress 资源，将其所有规则合并生成统一的 Nginx 配置。
- “优先级”**完全取决于所有 Ingress 合并后路径与 host 的匹配规则**，即无论规则散落在几个 Ingress，nginx 转发顺序与来源无关，和你是 frontend.yaml 还是 backend.yaml 没有关系。
- 路由转发时，仍然遵循“host 匹配 -> pathType 匹配（Exact 优先 > Prefix 长度优先）”的层次，和单一 Ingress 全部规则写一起时完全一致。

**如何确认优先级？**

1. **查看合并后的配置：**
   - 通过访问 ingress-nginx 控制器暴露的 debug 接口，如：
     ```
     curl http://<ingress-nginx-pod-ip>:10254/nginx/server_template
     # 或新版 controller:
     curl http://<ingress-nginx-pod-ip>:10254/nginx/ingresses
     ```
   - 也可进入 pod 内查看 `/etc/nginx/nginx.conf` 或 `/etc/nginx/conf.d/` 下的实际生成文件。所有 Ingress 资源的合并规则都可被查到。

2. **命令行工具辅助：**
   - 结合 `kubectl describe ingress` 查看所有相关 Ingress 资源。
   - 标明 `host`、`path` 和 `pathType`，罗列出来所有路径后，按优先级规则人工梳理一遍。

3. **实际请求测试法（推荐）：**
   - 用 curl 或 HTTP-client 针对不同路径和 host 发请求，观察各自转发到的后端 service 名字是否符合预期。
   - 若配有访问日志或 debug 日志，可利用日志确认路由目标。

4. **注意事项：**
   - 不要假设“分开写的 Ingress 互不影响”；只要 host/path 有交叠，路由解析就全局统一生效。
   - **优先级冲突时**，最终的转发永远以“host 精准匹配 > pathType: Exact > pathType: Prefix（长路径优先）”生效，无论规则被写在哪个 Ingress 资源中。

**小结：**
- 多个 Ingress 共管路由时，优先级确认要用实际 Nginx 合并后的配置分析，规则来源和文件归属不影响优先级。
- 推荐上线前**清点所有 Ingress 路由，明确所有 host/path 组合唯一性和覆盖关系**，尤其对大集群或多团队碎片化维护的环境。


ingress-nginx anotations可用注解字段有哪些

**ingress-nginx 可用注解字段有哪些？**

在 `ingress-nginx` 控制器中，Ingress 资源支持大量以 `nginx.ingress.kubernetes.io/` 为前缀的注解（Annotation），用以灵活定制路由、转发、认证、安全、重写、安全加固等功能。以下是一些常用且实用的注解分类和字段举例：

#### 1. 后端转发类
- `nginx.ingress.kubernetes.io/service-upstream`: 启用原生 Service 负载均衡（不经过 Pod 列表）。
- `nginx.ingress.kubernetes.io/upstream-hash-by`: 依据特定 header/cookie/变量做 upstream 路由。

#### 2. 认证/授权类
- `nginx.ingress.kubernetes.io/auth-type`: 启用 basic/auth-request 第三方认证等方式。
- `nginx.ingress.kubernetes.io/auth-secret`: Basic Auth secret 名称。
- `nginx.ingress.kubernetes.io/auth-url`: 外部认证后端 URL。

#### 3. 重定向与重写类
- `nginx.ingress.kubernetes.io/rewrite-target`: 路径重写，常用于 API 转发或前端单页应用（SPA）。
- `nginx.ingress.kubernetes.io/ssl-redirect`: 是否强制 HTTPS 跳转。
- `nginx.ingress.kubernetes.io/force-ssl-redirect`: 开启后无论 global 配置，都强制跳转到 HTTPS。
- `nginx.ingress.kubernetes.io/from-to-www-redirect`: 在 www 和非 www 之间自动跳转。

#### 4. 代理/超时/缓存类
- `nginx.ingress.kubernetes.io/proxy-body-size`: 单次请求最大体积（如上传文件）。
- `nginx.ingress.kubernetes.io/proxy-read-timeout`: 读取后端响应的最大超时时间。
- `nginx.ingress.kubernetes.io/proxy-send-timeout`: 发送到后端的最大超时时间。
- `nginx.ingress.kubernetes.io/proxy-buffer-size`: 缓冲响应头的最大字节数。

#### 5. 白名单与安全类
- `nginx.ingress.kubernetes.io/whitelist-source-range`: 允许访问的 IP/CIDR 白名单。
- `nginx.ingress.kubernetes.io/configuration-snippet`: 允许为 server/location 注入自定义配置片段。
- `nginx.ingress.kubernetes.io/server-snippet`: 注入自定义 Nginx 配置到 server 块。

#### 6. 限流与安全加固
- `nginx.ingress.kubernetes.io/limit-connections`: 限定并发连接数。
- `nginx.ingress.kubernetes.io/limit-rpm`: 每分钟请求数限制（Rate Limit）。
- `nginx.ingress.kubernetes.io/limit-rps`: 每秒请求数限制。
- `nginx.ingress.kubernetes.io/enable-modsecurity`: 开启 ModSecurity 防护。
- `nginx.ingress.kubernetes.io/enable-owasp-core-rules`: 开启 OWASP core rules。

#### 7. 其他常用注解
- `nginx.ingress.kubernetes.io/use-regex`: 路径匹配支持正则。
- `nginx.ingress.kubernetes.io/custom-http-errors`: 后端哪些 http code 时由 Nginx 自定义 error_page 兜底。
- `nginx.ingress.kubernetes.io/canary`: 标记金丝雀发布路由。
- `nginx.ingress.kubernetes.io/session-cookie-name`: sticky session 场景自定义 cookie 名。
- `nginx.ingress.kubernetes.io/affinity`: upstream 亲和性（如基于 IP、Cookie 一致性转发）。
- `nginx.ingress.kubernetes.io/upstream-vhost`: 后端真实 host 替换（虚拟主机）。

---

**完整注解列表与文档：**

- 官方文档：[Annotation Index](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/)
- 注解字段通常为低驼峰、小写、分隔符均用 `-`

> 实战建议：部署上线前，建议使用官方 Annotation Index 文档，结合实际 controller 版本验证支持的注解字段，部分高级特性或扩展注解可能需要 controller 配置开启或特定版本才可用。

常见注解使用示例（Ingress yaml 片段）:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: demo
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/proxy-body-size: "10m"
    nginx.ingress.kubernetes.io/whitelist-source-range: "10.0.0.0/8,192.168.0.0/16"
    nginx.ingress.kubernetes.io/configuration-snippet: |
      more_set_headers "X-Env: production";
```

如需特殊功能、Header 注入、路由控制、安全加固，记得翻查 `ingress-nginx` 官方注解详细表格。



### ingress-nginx 通过注解提升默认路由的优先级

在实际生产环境中，经常遇到 Ingress 路由规则存在“默认路由”或 catch-all 路径（如 `/`），但希望某些更具体的路径优先匹配，避免请求被默认路由捕获。ingress-nginx 在底层实现上，会根据规则的“路径长度、更精确路径优先、Host 更优先”原则进行路由排序，但如果遇到规则有重叠、或希望人为干预优先级，可以结合注解来“提升”某些路由规则的处理优先级。

#### 核心注解：`nginx.ingress.kubernetes.io/server-snippet`

通过 `server-snippet`，可以向 Nginx 的 server 区块注入自定义配置。例如：插入配置让默认路由的 `location /` 提前 return，或插入优先级较高的 rewrite/conditional。

#### 实践示例

假设存在如下需求：  
1. `/api` 路径有单独的 Ingress 控制。  
2. `/`（catch-all）为兜底默认路由，但希望仅在明确没有其它规则命中的情况下生效。  
3. 防止因定义顺序错误导致默认路由"抢跑"。  

示例 YAML：

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: default-route
  annotations:
    nginx.ingress.kubernetes.io/server-snippet: |
      # 避免默认 location / 抢占优先级，非/api路径才兜底
      location = /api {
        return 404;
      }
spec:
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: default-service
                port:
                  number: 80
```

在上述 server-snippet 注入后，`/api` 的请求会被前面的具体规则命中，其余流量才会由默认路由接管，实现了更精细的优先级控制。

#### 进阶用法

如果路径规则复杂或有多种特殊情况，可以配合 `configuration-snippet` 对 location 块做更细粒度的控制，也可以为“高优先级”路由单独设优先注解分组，保证路由匹配顺序更加可控。

**参考文档：**
- [官方优先级说明](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/#server-snippet)
- [如何提升 Ingress 路由优先级](https://github.com/kubernetes/ingress-nginx/issues/6218)

> 小结：  
> 默认情况下，路径越具体优先级越高。若需手工干预，优先考虑调整 paths 定义，其次通过注解（如 server-snippet 或 configuration-snippet）增强控制。

```yaml
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: devops-uat.orsd.tech
  namespace: uat
  resourceVersion: '901431080'
spec:
  ingressClassName: gigacloud
  rules:
    - host: devops-uat.orsd.tech
      http:
        paths:
          - backend:
              service:
                name: auto-ops-web
                port:
                  number: 80
            path: /
            pathType: Prefix

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /jeecgboot/$1
    nginx.ingress.kubernetes.io/use-regex: 'true'
  name: devops-uat.orsd.tech-rewrite
  namespace: uat
  resourceVersion: '916634859'
spec:
  ingressClassName: gigacloud
  rules:
    - host: devops-uat.orsd.tech
      http:
        paths:
          - backend:
              service:
                name: auto-ops
                port:
                  number: 8080
            path: /jeecg-boot/(.*)
            pathType: Prefix
```

### 2.2.2. 路由优先级

通过上面的配置，我们实现了对`/jeecg-boot`路径的自动重写，但是当请求路径为`/`时，会优先匹配到`auto-ops-web`服务，而不是`auto-ops`服务，这并不是我们期望的结果。

为了解决这个问题，我们需要调整路由的优先级，让`/jeecg-boot`路径的请求优先匹配到`auto-ops`服务。

我们可以通过以下方式来调整 Ingress 配置，提升 `/jeecg-boot` 路径的匹配优先级：

1. **将更具体的路径（如 `/jeecg-boot/(.*)`）放在更通用路径（如 `/`）之前。**  
   NGINX Ingress Controller 会优先匹配更长、更具体的 path。

2. **pathType 建议使用 `ImplementationSpecific` 或 `Exact`，避免 Prefix 带来的模糊匹配问题。**

下面是推荐的 Ingress 资源 YAML 示例，展示了如何将 `/jeecg-boot/(.*)` 的路由规则写在 `/` 之前，从而优先路由到 `auto-ops` 服务：

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: devops-uat.orsd.tech
  namespace: uat
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /jeecgboot/$1
    nginx.ingress.kubernetes.io/use-regex: "true"
spec:
  ingressClassName: gigacloud
  rules:
    - host: devops-uat.orsd.tech
      http:
        paths:
          - path: /jeecg-boot/(.*)
            pathType: ImplementationSpecific
            backend:
              service:
                name: auto-ops
                port:
                  number: 8080
          - path: /
            pathType: Prefix
            backend:
              service:
                name: auto-ops-web
                port:
                  number: 80
```

**重点说明：**
- 将 `/jeecg-boot/(.*)` 这条规则放在前面，确保对该路径的请求优先路由到 `auto-ops` 服务。
- 保持 `/` 路径用于兜底，指向 `auto-ops-web` 服务。
- 合并两个 Ingress 资源为一个，可以防止优先级不一致造成的异常路由。

这样配置后，`/jeecg-boot` 相关路径请求会优先匹配到 `auto-ops` 服务，其余请求则会走到 `auto-ops-web`，达到我们预期的路由优先级效果。


**答：**  
如果 `/api` 和 `/` 路由都定义在同一个 Ingress 资源的 paths 下，并且有重定向（如 `nginx.ingress.kubernetes.io/rewrite-target` 注解），匹配优先级始终遵循“更长路径更优先”的原则：

- 当请求为 `/api/xxx` 时，会**优先匹配到 `/api` 这条 path**，不会直接匹配 `/`，即使两个路径都存在重写（rewrite）配置。
- 只有当请求的路径既不符合 `/api`（比如 `/other` 或 `/`）时，才会落入兜底的 `/` 路径规则。

**举例说明：**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
    - host: test.example.com
      http:
        paths:
          - path: /api/?(.*)
            pathType: ImplementationSpecific
            backend:
              service:
                name: api-svc
                port:
                  number: 80
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web-svc
                port:
                  number: 80
```
- 请求 `/api/foo` 会被 `/api/?(.*)` 匹配，不会走 `/`。
- 请求 `/` 或 `/about` 会被 `/` 匹配到 web-svc。

**注意：**  
- 若 paths 都使用较宽松的 pathType（如 Prefix），并路径有交集，则优先级依然是“更长的匹配优先”；`/api` 比 `/` 优先。
- 若重定向或重写规则配置不当，可能导致转发到同一 backend，但路径匹配规则始终优先更具体的 path。

**结论：**  
即使有 rewrite 操作，只要 paths 都定义在一个 ingress 下，Nginx 是先依据 path 长度和类型选择路由 backend，再应用 rewrite，**不会让 `/api` 的请求先匹配 `/` 再重写**。正确书写 paths 和 rewrite，就能规避“被兜底 `/` 路径抢先匹配”的风险。


举例，若你的 Ingress 这样写：

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: orsd-login-demo
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
    - host: devops-uat.orsd.tech
      http:
        paths:
          - path: /login
            pathType: Prefix
            backend:
              service:
                name: login-service
                port:
                  number: 80
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web-home
                port:
                  number: 80
```

- 访问 `https://devops-uat.orsd.tech/login` 会先被 `/login` 匹配（因为路径更长/更具体），请求会被转发到 `login-service`。
- 访问 `https://devops-uat.orsd.tech/` 或除 `/login` 外其他路径（如 `/about`），会被 `/` 匹配，转发到 `web-home`。

**结论：**  
无论你 `/login` 路径出现在 paths 里，还是和 `/` 并列，Nginx 路由优先级都是**更长的路径优先**。因此 `/login` 的路径不会被 `/` 路径抢先兜底拦截，只要两条都存在且 pathType 相同 `/login` 就总是更优先。

**实用建议：**
- 只要写出 `/login` 路径专属规则，不会有被 `/` 路径误兜底的风险。
- 若 `/login` 需要被特殊处理，写明独立后端或 rewrite 即可。
