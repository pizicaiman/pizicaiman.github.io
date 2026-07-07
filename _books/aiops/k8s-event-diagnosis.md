---
layout: book-article
title: "Kubernetes 事件智能分析与故障诊断"
excerpt: "利用大语言模型对 Kubernetes 集群事件进行语义分析，实现故障自动诊断与修复建议生成。"
category_link: /books/aiops/
category_title: AIOps 读书笔记
icon: "🤖"
date: 2025-05-20
status: planned
tags: [Kubernetes, LLM, 故障诊断]
chapters:
  - title: "Kubernetes 事件体系概述"
    subtitle: "理解 K8s 事件的产生机制与数据结构"
    content: |
      ## 什么是 Kubernetes 事件？

      Kubernetes 事件是集群中资源状态变化的记录，由控制器、调度器、kubelet 等组件在资源生命周期中产生。

      ### 事件数据结构

      ```yaml
      apiVersion: v1
      kind: Event
      metadata:
        name: my-pod.1234abcd
        namespace: default
      involvedObject:
        kind: Pod
        name: my-pod-xyz
      reason: FailedScheduling
      message: "0/3 nodes are available: insufficient cpu"
      type: Warning
      count: 5
      firstTimestamp: "2025-05-20T10:00:00Z"
      lastTimestamp: "2025-05-20T10:05:00Z"
      ```

      ### 常见事件类型

      | Reason | 类型 | 含义 |
      |--------|------|------|
      | FailedScheduling | Warning | 调度失败，资源不足或节点不满足条件 |
      | OOMKilling | Warning | 容器因内存超限被杀死 |
      | Unhealthy | Warning | 健康检查失败 |
      | BackOff | Warning | 容器反复崩溃，退避重启 |
      | Pulled | Normal | 镜像拉取成功 |
      | Scheduled | Normal | Pod 成功调度到节点 |

      > **关键洞察**：事件本身只记录"发生了什么"，不包含"为什么"和"怎么办"——这正是 AI 诊断的价值所在。

  - title: "基于 LLM 的事件分析方案"
    subtitle: "用大语言模型实现事件语义理解与根因推理"
    content: |
      ## 分析流程设计

      ```
      kubectl get events / Event API
              ↓
        事件聚合与过滤
              ↓
      ┌────────────────────┐
      │  上下文增强         │
      │  - Pod 描述         │
      │  - 节点状态         │
      │  - 相关日志片段     │
      └────────────────────┘
              ↓
        LLM 诊断推理
              ↓
      诊断报告 + 修复建议
      ```

      ### 上下文收集策略

      单纯的事件信息不足以做出准确诊断，需要收集关联上下文：

      1. **Pod 描述**：`kubectl describe pod` 获取资源请求、限制、环境变量
      2. **节点状态**：节点资源使用率、条件状态（Ready、MemoryPressure）
      3. **最近日志**：容器最近 50 行日志，捕捉崩溃前的异常输出
      4. **关联事件**：同一 Pod 或同一节点的其他事件

      ### Prompt 模板设计

      ```
      你是一个 Kubernetes 专家。请分析以下集群事件并给出诊断：

      【事件信息】
      {event_json}

      【Pod 描述】
      {pod_describe}

      【节点状态】
      {node_status}

      【最近日志】
      {recent_logs}

      请输出：
      1. 根因分析（一句话概括）
      2. 详细解释
      3. 修复建议（具体命令）
      4. 预防措施
      ```

      > **注意事项**：LLM 的诊断结果需要人工确认后再执行修复操作，避免自动化误操作。

  - title: "工程化落地实践"
    subtitle: "将 AI 诊断集成到日常运维工作流"
    content: |
      ## 集成方式

      ### 方案一：ChatOps 机器人

      在 Slack/飞书中部署 K8s 诊断机器人：

      ```
      @k8s-bot diagnose pod my-app-xyz -n production
      ```

      机器人自动收集上下文、调用 LLM、返回诊断报告。

      ### 方案二：告警联动

      当 Prometheus 告警触发时，自动调用诊断服务并将结果附加到告警通知中。

      ### 方案三：CLI 插件

      ```bash
      kubectl ai-diagnose pod my-pod-xyz
      # 输出诊断报告和修复建议
      ```

      ## 效果评估

      | 场景 | 传统方式耗时 | AI 辅助耗时 | 准确率 |
      |------|-------------|------------|--------|
      | OOMKilled | 10 分钟 | 1 分钟 | 95% |
      | FailedScheduling | 5 分钟 | 30 秒 | 98% |
      | CrashLoopBackOff | 15 分钟 | 2 分钟 | 85% |
      | 网络不通 | 20 分钟 | 3 分钟 | 78% |

      > **待探索**：结合 RAG 引入团队历史故障处理经验，提升复杂场景的诊断准确率。

---
