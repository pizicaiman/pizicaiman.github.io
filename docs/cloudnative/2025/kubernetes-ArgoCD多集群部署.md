---
layout: doc
title: Kubernetes下通过ArgoCD实现多集群部署
date: 2025-01-25
category: cloudnative
tags: [kubernetes, cloud-native, alibaba-cloud, ack]
excerpt: 深入解析阿里云ACK集群架构设计、最佳实践及企业级应用案例
---

# Kubernetes下通过ArgoCD实现多集群部署

本指南介绍如何通过ArgoCD实现Kubernetes多集群的GitOps持续部署。

## 一、环境准备

1. **安装ArgoCD（在管理集群）**
   ```bash
   kubectl create namespace argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```
2. **安装ArgoCD CLI**
   [下载地址](https://argo-cd.readthedocs.io/en/stable/getting_started/#2-download-argo-cd-cli)

3. **准备多个Kubernetes集群**
   - 假设有`cluster-a`（中央管理）、`cluster-b`、`cluster-c`（业务集群）

---

## 二、注册目标集群

以`cluster-a`为主控集群，`cluster-b`、`cluster-c`为目标集群。

1. 获取每个目标集群kubeconfig。
2. 将目标集群注册到ArgoCD：
   ```bash
   # 切换到ArgoCD管理集群
   kubectl config use-context cluster-a

   # 添加cluster-b
   argocd cluster add CONTEXT_CLUSTER_B

   # 添加cluster-c
   argocd cluster add CONTEXT_CLUSTER_C
   ```

---

## 三、为每个集群配置Application资源

每个目标集群部署一个ArgoCD Application CR，例如：

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: demo-app-cluster-b
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'https://github.com/your-org/your-repo.git'
    targetRevision: main
    path: manifests/
  destination:
    server: https://<cluster-b-api-server>
    namespace: demo
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

> `server:` 可在 `argocd cluster list` 查得。

---

## 四、同步与管理

- 通过ArgoCD UI或CLI统一管理并同步多集群资源。
- 可用`ApplicationSet`自动为多集群生成部署配置。

---

## 五、常见问题与建议

- 保证各集群间网络互通，`argocd-server` 能连通目标APIServer。
- 管理集群权限建议用RBAC最小权限原则。

---

## 参考文档

- [ArgoCD官方文档](https://argo-cd.readthedocs.io/)
- [ArgoCD多集群场景介绍](https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/#repositories-clusters-and-ssh-known-hosts)

---

