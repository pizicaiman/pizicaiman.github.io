---
layout: book-article
title: "GitOps 实战：从理论到落地"
excerpt: "系统梳理 GitOps 核心理念，结合 ArgoCD 与 Flux 实现声明式基础设施管理。"
category_link: /books/devops/
category_title: DevOps 读书笔记
icon: ""
date: 2025-04-10
status: completed
tags: [GitOps, ArgoCD, Flux, CI/CD]
chapters:
  - title: "GitOps 的核心理念"
    subtitle: "以 Git 为唯一真相源的声明式运维范式"
    content: |
      ## 什么是 GitOps？

      GitOps 是一种以 Git 仓库作为基础设施和应用程序**唯一真相源**（Single Source of Truth）的运维范式。所有变更通过 Git 提交触发，由自动化控制器将集群状态收敛到 Git 中声明的期望状态。

      ### GitOps 四大原则

      1. **声明式描述**：系统期望状态用声明式方式（YAML/Helm/Kustomize）描述
      2. **版本化与不可变**：所有声明存储在 Git 中，版本化、可审计、可回滚
      3. **自动拉取**：软件代理自动从 Git 拉取期望状态并与实际状态对比
      4. **持续调和**：控制器持续监控并修正偏差，确保实际状态 = 期望状态

      ### GitOps vs 传统 CI/CD

      | 维度 | 传统 CI/CD (Push) | GitOps (Pull) |
      |------|-------------------|---------------|
      | 部署触发 | CI 推送变更到集群 | CD 控制器从 Git 拉取 |
      | 权限模型 | CI 需要集群写权限 | 集群内控制器有权限，CI 只需写 Git |
      | 状态管理 | 依赖 CI 记录部署状态 | Git 即为状态，天然可审计 |
      | 回滚方式 | 重新触发 CI 部署旧版本 | Git revert 即可回滚 |
      | 安全边界 | CI 系统暴露集群凭据 | 集群不暴露外部访问入口 |

      > **核心优势**：GitOps 将部署权限从 CI 系统转移到集群内部，大幅缩小攻击面。

  - title: "ArgoCD 深度实践"
    subtitle: "企业级 GitOps 控制器的部署与配置"
    content: |
      ## ArgoCD 架构

      ```
      ┌──────────┐     ┌──────────────┐     ┌─────────────┐
      │   Git    │────▶│   ArgoCD     │────▶│  Kubernetes  │
      │  Repo    │     │  Controller  │     │   Cluster    │
      └──────────┘     └──────────────┘     └─────────────
                             │
                      ┌──────┴──────┐
                      │  ArgoCD UI  │
                      │  (Web/Dashboard)│
                      └─────────────┘
      ```

      ### 安装与配置

      ```yaml
      # Application 资源定义
      apiVersion: argoproj.io/v1alpha1
      kind: Application
      metadata:
        name: my-app
        namespace: argocd
      spec:
        project: default
        source:
          repoURL: https://github.com/org/k8s-manifests.git
          targetRevision: HEAD
          path: overlays/production
        destination:
          server: https://kubernetes.default.svc
          namespace: production
        syncPolicy:
          automated:
            prune: true
            selfHeal: true
          syncOptions:
            - CreateNamespace=true
      ```

      ### 关键特性

      - **多集群管理**：一个 ArgoCD 实例管理多个目标集群
      - **Helm + Kustomize**：支持多种清单渲染方式
      - **Sync Waves**：控制资源同步顺序（先 Namespace，再 ConfigMap，最后 Deployment）
      - **健康检查**：自定义资源健康评估逻辑

      > **生产建议**：开启 `selfHeal: true` 确保集群状态自动收敛；使用 App-of-Apps 模式管理多应用。

  - title: "Flux CD 对比与选型"
    subtitle: "两大 GitOps 方案的差异化分析"
    content: |
      ## Flux CD 架构

      Flux 采用模块化设计，每个组件负责特定职责：

      | 组件 | 职责 |
      |------|------|
      | source-controller | 管理 Git/Helm/OCI 仓库源 |
      | kustomize-controller | 渲染 Kustomize 清单 |
      | helm-controller | 管理 Helm Release 生命周期 |
      | notification-controller | 发送告警和通知 |
      | image-reflector / updater | 自动追踪和更新镜像版本 |

      ## ArgoCD vs Flux 选型指南

      | 维度 | ArgoCD | Flux |
      |------|--------|------|
      | UI 界面 | 丰富的 Web UI | 无原生 UI（依赖 Weave GitOps） |
      | 多集群 | 原生支持 | 需要额外配置 |
      | 多租户 | 项目隔离 + RBAC | Namespace 隔离 |
      | 镜像更新 | 需配合外部工具 | 原生 image updater |
      | 社区生态 | CNCF 毕业项目 | CNCF 毕业项目 |
      | 适用场景 | 需要可视化管理的团队 | 偏好 CLI 和 Git 原生的团队 |

      > **选型建议**：需要可视化管理和多集群统一视图选 ArgoCD；偏好轻量、Git 原生、镜像自动更新选 Flux。

  - title: "GitOps 安全与合规"
    subtitle: "保障 GitOps 工作流的安全最佳实践"
    content: |
      ## 安全模型

      ### 最小权限原则

      ```
      开发人员 → 提交 PR → Code Review → Merge → Git
                                                  ↓
      集群 ← ArgoCD 同步 ← Git Webhook 通知
      ```

      - 开发人员**不需要**集群访问权限
      - ArgoCD ServiceAccount 仅拥有目标 Namespace 的权限
      - Git 仓库配置 Branch Protection 防止未授权变更

      ### 密钥管理

      | 方案 | 适用场景 |
      |------|---------|
      | Sealed Secrets | 简单场景，加密后存入 Git |
      | External Secrets Operator | 对接 Vault/AWS Secrets Manager |
      | SOPS + Age | 文件级加密，支持多密钥 |
      | ArgoCD Vault Plugin | 直接从 Vault 读取密钥 |

      ### 合规审计

      - **Git 历史即审计日志**：每次变更都有作者、时间、审批记录
      - **ArgoCD Audit Log**：记录所有同步操作和手动干预
      - **Policy as Code**：使用 OPA/Gatekeeper 在同步前验证清单合规性

      > **核心原则**：Git 是审计的起点，但不是终点。结合运行时安全监控形成完整的安全闭环。

---
