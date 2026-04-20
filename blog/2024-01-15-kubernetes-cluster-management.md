---
title: "Kubernetes 集群管理最佳实践"
date: 2024-01-15 10:00:00 +0800
categories: [云原生, Kubernetes]
tags: [Kubernetes, 集群管理, 最佳实践, 运维]
excerpt: "深入探讨 Kubernetes 集群的规划、部署和运维最佳实践，涵盖生产环境实战经验"
author_profile: true
---

# Kubernetes 集群管理最佳实践

本文基于大规模生产环境经验，总结 Kubernetes 集群管理的最佳实践，涵盖集群规划、部署、运维和故障排查等关键环节。

## 🏗️ 集群架构设计

### 控制平面高可用

生产环境控制平面必须配置高可用：

```yaml
# 控制平面节点配置
apiVersion: v1
kind: Node
metadata:
  name: control-plane-1
  labels:
    node-role.kubernetes.io/control-plane: ""
spec:
  taints:
  - key: node-role.kubernetes.io/control-plane
    effect: NoSchedule
```

**关键要点：**
- 至少 3 个控制平面节点
- 使用外部负载均衡器（如 HAProxy、Nginx）
- etcd 集群独立部署，使用专用存储

### 工作节点规划

根据业务负载特征规划工作节点：

```yaml
# 工作节点资源配置
resources:
  requests:
    cpu: "4"
    memory: "16Gi"
  limits:
    cpu: "8"
    memory: "32Gi"
```

**规划原则：**
- 按业务类型分组节点（计算密集、IO 密集、GPU 节点）
- 预留 20% 资源用于系统组件和应急扩容
- 使用节点亲和性实现业务隔离

## 🚀 部署与配置

### 集群初始化

使用 kubeadm 初始化集群：

```bash
# 初始化控制平面
kubeadm init \
  --control-plane-endpoint "LOAD_BALANCER_DNS:6443" \
  --upload-certs \
  --pod-network-cidr "10.244.0.0/16" \
  --service-cidr "10.96.0.0/12"
```

### 网络插件选择

推荐使用 Cilium 或 Calico：

```bash
# 安装 Cilium
cilium install \
  --cluster-name production \
  --kube-proxy-replacement=strict \
  --encryption=wireguard
```

**选择标准：**
- Cilium：eBPF 技术，性能优异，支持网络策略
- Calico：成熟稳定，社区活跃，支持 BGP

## ⚙️ 资源调度优化

### HPA 配置

基于 CPU/内存和自定义指标实现自动扩缩容：

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Pods
    pods:
      metric:
        name: requests_per_second
      target:
        type: AverageValue
        averageValue: "1000"
```

### 节点亲和性调度

使用亲和性规则优化 Pod 调度：

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: node-type
          operator: In
          values:
          - compute
    preferredDuringSchedulingIgnoredDuringExecution:
    - weight: 100
      preference:
        matchExpressions:
        - key: disk-type
          operator: In
          values:
          - ssd
```

## 🔒 安全配置

### Pod 安全策略

实施 Pod 安全标准：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
  containers:
  - name: app
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
```

### 网络策略

限制 Pod 间网络通信：

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

## 🔍 故障排查

### 常见问题诊断

```bash
# 检查节点状态
kubectl get nodes -o wide

# 查看 Pod 事件
kubectl describe pod <pod-name>

# 检查资源使用情况
kubectl top nodes
kubectl top pods
```

### 日志收集

```bash
# 收集控制平面日志
journalctl -u kubelet -f

# 收集 Pod 日志
kubectl logs <pod-name> --previous
kubectl logs -f <pod-name> -c <container-name>
```

## 📊 监控指标

### 关键指标

| 指标类别 | 具体指标 | 告警阈值 |
|---------|---------|---------|
| 集群健康 | NodeReady | < 100% |
| 资源使用 | CPU/Memory 利用率 | > 80% |
| Pod 状态 | CrashLoopingOff | > 0 |
| 网络连接 | Connection refused | > 5/min |

### Prometheus 告警规则

```yaml
groups:
- name: kubernetes-cluster
  rules:
  - alert: NodeMemoryUsageHigh
    expr: (1 - node_memory_MemoryAvailable_bytes / node_memory_MemoryTotal_bytes) * 100 > 80
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "节点内存使用率过高"
      description: "节点 {{ $labels.instance }} 内存使用率超过 80%"
```

## 🎯 最佳实践总结

1. **架构设计**：控制平面高可用，工作节点按业务类型分组
2. **资源规划**：预留系统资源，使用资源配额限制
3. **安全加固**：实施最小权限原则，启用网络策略
4. **监控告警**：建立完善的监控体系，设置合理的告警阈值
5. **故障预案**：制定应急响应流程，定期演练

## 📚 参考资源

- [Kubernetes 官方文档](https://kubernetes.io/docs/)
- [Kubernetes 最佳实践](https://kubernetes.io/docs/concepts/configuration/overview/)
- [Cilium 文档](https://docs.cilium.io/)

---

**作者：** Pizicaiman  
**发布时间：** 2024-01-15  
**分类：** 云原生, Kubernetes  
**标签：** Kubernetes, 集群管理, 最佳实践, 运维