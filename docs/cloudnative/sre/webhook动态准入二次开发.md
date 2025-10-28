# webhook 动态准入二次开发指南

Kubernetes 支持通过动态准入 webhook 扩展集群资源创建、更新、删除等操作的策略校验与自动化修正能力，是企业实现安全、合规和自定义业务准入控制的关键机制。以下聚焦 `ValidatingAdmissionWebhook`（校验型）与 `MutatingAdmissionWebhook`（变更型）的二次开发典型流程与实用建议。

---

## 1. Webhook 基本类型

- **MutatingAdmissionWebhook（变更型）**  
  可以自动修改用户请求（如注入 sidecar、补全字段等）。

- **ValidatingAdmissionWebhook（校验型）**  
  只做校验，拒绝/允许请求，不能修改内容（如策略合规校验）。

---

## 2. 开发基本流程示例

1. **定义 Webhook Server（推荐用 Go 开发，兼容 K8s 用 JSON/AdmissionReview 协议）**

    ```go
    // 示例: Mutating/Validating 共用 Admission Handler 代码段
    func serve(w http.ResponseWriter, r *http.Request) {
        var admissionReview v1.AdmissionReview
        if err := json.NewDecoder(r.Body).Decode(&admissionReview); err != nil {
            // 错误处理
            return
        }
        req := admissionReview.Request
        // 解析资源（如 Pod）
        var pod corev1.Pod
        if err := json.Unmarshal(req.Object.Raw, &pod); err != nil {
            // 错误处理
            return
        }
        // 业务逻辑: 校验或变更 pod
        var patches []map[string]interface{}
        var allowed = true
        var reason = ""
        if req.Kind.Kind == "Pod" {
            // 示例1: 校验某 label 是否存在
            if _, ok := pod.Labels["my-label"]; !ok {
                allowed = false
                reason = "必须包含 my-label"
            }
            // 示例2: 如需变更，可构造 json patch
            // patches = append(patches, map[string]interface{}{"op": "add", "path": "/metadata/labels/injected", "value": "yes"})
        }
        response := v1.AdmissionResponse{
            UID:     req.UID,
            Allowed: allowed,
            Result: &metav1.Status{
                Message: reason,
            },
        }
        if len(patches) > 0 {
            patchBytes, _ := json.Marshal(patches)
            pt := v1.PatchTypeJSONPatch
            response.PatchType = &pt
            response.Patch = patchBytes
        }
        json.NewEncoder(w).Encode(v1.AdmissionReview{
            Response: &response,
        })
    }
    ```

2. **部署并暴露 Webhook Server**

   - 建议使用 Deployment+Service 管理（TLS 证书必需）
   - 结合 `cert-manager` 自动签发证书（推荐）

3. **注册 Webhook 配置对象**

    - `MutatingWebhookConfiguration` / `ValidatingWebhookConfiguration`
    - 参考官方结构：

      ```yaml
      apiVersion: admissionregistration.k8s.io/v1
      kind: ValidatingWebhookConfiguration
      metadata:
        name: my-webhook
      webhooks:
        - name: my-webhook.mycompany.com
          clientConfig:
            service:
              namespace: default
              name: webhook-service
              path: "/"
            caBundle: <base64 ca>
          rules:
            - apiGroups: [""] 
              apiVersions: ["v1"]
              resources: ["pods"]
              operations: ["CREATE", "UPDATE"]
          admissionReviewVersions: ["v1"]
          sideEffects: None
          failurePolicy: Fail
      ```

---

## 3. 实用二次开发建议

- 强烈建议用 [k8s.io/api/admission/v1](https://pkg.go.dev/k8s.io/api/admission/v1) 官方包结构实现 JSON 解析/响应
- 保证 webhook 服务高可用、快速响应，否则会影响业务资源创建
- 生产环境用 Cert-Manager 动态证书管理，切勿自签过期
- webhook 逻辑建议无状态，必要信息建议外部配置下发
- 编写详尽 e2e 测试，尤其是错误链路和 Patch 变更效果
- Metrics/日志建议全面，方便溯源及故障追踪

---

## 4. 典型扩展场景举例

- **资源配额与命名合规校验**（Validating）  
  拒绝不合规资源名/标签/pvc 配额等。

- **自动注入管理 sidecar/环境变量/agent**（Mutating）  
  动态为指令性 Pod 自动注入 Istio/Datadog/自有 sidecar。

- **跨多集群统一管控**  
  联合 webhook/CRD 实现多集群策略下发。

---

## 5. 参考资料

- [官方 Webhook 扩展文档](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/extensible-admission-controllers/)
- [k8s webhook server go example](https://github.com/kubernetes/sample-controller/tree/master/admission-webhook-example)
- [云原生安全：准入 webhook 最佳实践](https://jimmysong.io/kubernetes-handbook/extend/admission.html)

---

> 总结：Webhook 动态准入是 K8s 安全与企业策略落地的强力工具，二次开发时关注协议规范、性能与可追踪性，能快速实现“所见即所得”的资源合规与自动变更场景。

如需示例工程、Patch 细节或复杂场景定制，欢迎留言交流！

### 示例工程

可参考如下简单的 Go 语言 Mutating Webhook 示例（支持 Patch 注入 Label）：

```go
// main.go
package main

import (
    "crypto/tls"
    "encoding/json"
    "io/ioutil"
    "net/http"
    admissionv1 "k8s.io/api/admission/v1"
    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
)

func serveMutate(w http.ResponseWriter, r *http.Request) {
    body, _ := ioutil.ReadAll(r.Body)
    review := admissionv1.AdmissionReview{}
    _ = json.Unmarshal(body, &review)

    // 构造 patch 操作，给 Pod 增加标签
    patch := `[{"op": "add", "path": "/metadata/labels/webhook-injected", "value": "true"}]`
    patchType := admissionv1.PatchTypeJSONPatch

    resp := admissionv1.AdmissionReview{
        TypeMeta: metav1.TypeMeta{
            Kind:       "AdmissionReview",
            APIVersion: "admission.k8s.io/v1",
        },
        Response: &admissionv1.AdmissionResponse{
            UID:       review.Request.UID,
            Allowed:   true,
            Patch:     []byte(patch),
            PatchType: &patchType,
        },
    }
    respBytes, _ := json.Marshal(resp)
    w.Header().Set("Content-Type", "application/json")
    w.Write(respBytes)
}

func main() {
    mux := http.NewServeMux()
    mux.HandleFunc("/mutate", serveMutate)

    server := &http.Server{
        Addr:      ":9443",
        Handler:   mux,
        TLSConfig: &tls.Config{MinVersion: tls.VersionTLS12},
    }
    // cert.pem/key.pem 自行配置
    server.ListenAndServeTLS("cert.pem", "key.pem")
}
```

- 配套 Kubernetes 配置可见：[官方 admission webhook 示例](https://github.com/kubernetes/sample-controller/tree/master/admission-webhook-example)
- 建议实际开发采用 [controller-runtime Webhook](https://book.kubebuilder.io/zh/cronjob-tutorial/webhook-implementation.html) 脚手架简化证书与接口逻辑

// Patch 细节说明：  
// 上例中的 patch 是一个 JSON Patch 格式数组，Kubernetes 审批控制器会按此修改对象：
// 例如：`[{"op": "add", "path": "/metadata/labels/webhook-injected", "value": "true"}]`
// - op: 操作类型，这里为 "add"（可选值有 add/replace/remove 等）
// - path: 描述需要修改的字段路径，本例增加 Labels
// - value: 新字段的值
//
// 更多 JSON Patch 语法详见：https://datatracker.ietf.org/doc/html/rfc6902
// 可根据业务调整 op/path/value 用法，完成对资源对象的任意结构patch。
//
// 若需同时 patch 多个字段，可给 patch 数组添加多组操作元素。例如：
// [
//   {"op": "add", "path": "/metadata/labels/env", "value": "prod"},
//   {"op": "replace", "path": "/spec/containers/0/image", "value": "yourrepo/yourimage:latest"}
// ]


// 复杂场景定制范例：根据资源内容定制 patch 动作
//
// 你可以根据 AdmissionReview 里的对象内容动态构造 patch，实现按需变更。比如：
// - 判断命名空间只对特定命名空间 patch
// - 检查 label/annotation 是否存在再 patch 或跳过
// - 根据 spec 字段内容有条件地变更
//
// 伪代码参考
/*
func serveMutate(w http.ResponseWriter, r *http.Request) {
    // ...解码 review 省略
    var patches []map[string]interface{}

    // 条件 patch 示例
    if review.Request.Namespace == "prod" {
        patches = append(patches, map[string]interface{}{
            "op":    "add",
            "path":  "/metadata/labels/production-injected",
            "value": "true",
        })
    }

    // 检查自定义 annotation，不存在则添加
    if _, ok := review.Request.Object.Object["metadata"].(map[string]interface{})["annotations"].(map[string]interface{})["your-annotation"]; !ok {
        patches = append(patches, map[string]interface{}{
            "op":    "add",
            "path":  "/metadata/annotations/your-annotation",
            "value": "yes",
        })
    }

    // 根据 spec 容器镜像字段 patch
    containers := review.Request.Object.Object["spec"].(map[string]interface{})["containers"].([]interface{})
    for i, c := range containers {
        container := c.(map[string]interface{})
        if container["image"] == "forbidden/image:tag" {
            patches = append(patches, map[string]interface{}{
                "op":    "replace",
                "path":  fmt.Sprintf("/spec/containers/%d/image", i),
                "value": "safe/image:latest",
            })
        }
    }

    patchBytes, _ := json.Marshal(patches)
    // ...构造 AdmissionReview Response Patch 字段
}
*/
// 实际代码实现时可根据业务复杂度按需扩展逻辑流程。
