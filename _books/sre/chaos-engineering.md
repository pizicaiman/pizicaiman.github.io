---
layout: book-article
title: "混沌工程：系统韧性验证"
excerpt: "基于 Chaos Mesh 的故障注入实验设计，验证分布式系统在异常场景下的自愈能力。"
category_link: /books/sre/
category_title: SRE 读书笔记
icon: "🛡️"
date: 2025-03-25
status: planned
tags: [混沌工程, Chaos Mesh, 韧性]
chapters:
  - title: "混沌工程基础理论"
    subtitle: "从 Netflix Chaos Monkey 到现代混沌工程"
    content: |
      ## 什么是混沌工程？

      混沌工程是在分布式系统上进行实验的学科，通过主动注入故障来验证系统的韧性（Resilience），建立对系统在生产环境中应对动荡条件的信心。

      ### 混沌工程原则

      1. **建立稳态假设**：定义系统在正常情况下的可量化行为
      2. **多样化真实世界事件**：模拟真实故障（网络延迟、节点宕机、依赖故障）
      3. **在生产环境运行实验**：测试环境无法完全模拟生产复杂度
      4. **持续自动化运行**：将混沌实验纳入 CI/CD 流水线
      5. **最小化爆炸半径**：控制实验影响范围，保护真实用户

      ### 混沌工程 vs 故障测试

      | 维度 | 传统故障测试 | 混沌工程 |
      |------|-------------|---------|
      | 目标 | 验证单个组件的故障处理 | 验证系统整体的韧性 |
      | 范围 | 预定义的故障场景 | 探索未知的系统行为 |
      | 环境 | 测试环境 | 生产环境（受控） |
      | 频率 | 发布前执行 | 持续执行 |
      | 心态 | "证明系统能处理已知故障" | "发现系统在面对未知故障时的行为" |

      > **核心理念**：混沌工程不是破坏系统，而是通过受控实验建立对系统行为的信心。

  - title: "Chaos Mesh 实践指南"
    subtitle: "云原生混沌工程平台的部署与使用"
    content: |
      ## 安装部署

      ```bash
      # 安装 Chaos Mesh
      curl -sSL https://mirrors.chaos-mesh.org/v2.6.0/install.sh | bash

      # 验证安装
      kubectl get pods -n chaos-testing
      ```

      ## 故障类型

      ### Pod Chaos

      ```yaml
      apiVersion: chaos-mesh.org/v1alpha1
      kind: PodChaos
      metadata:
        name: pod-kill-example
      spec:
        action: pod-kill
        mode: one
        selector:
          namespaces:
            - default
          labelSelectors:
            app: my-service
        scheduler:
          cron: "@every 5m"
      ```

      ### Network Chaos

      ```yaml
      apiVersion: chaos-mesh.org/v1alpha1
      kind: NetworkChaos
      metadata:
        name: network-delay
      spec:
        action: delay
        mode: all
        selector:
          namespaces: ["default"]
          labelSelectors:
            app: my-service
        delay:
          latency: "200ms"
          correlation: "25"
          jitter: "50ms"
        duration: "5m"
      ```

      ### Stress Chaos

      ```yaml
      apiVersion: chaos-mesh.org/v1alpha1
      kind: StressChaos
      metadata:
        name: cpu-stress
      spec:
        mode: one
        selector:
          namespaces: ["default"]
          labelSelectors:
            app: my-service
        stressors:
          cpu:
            workers: 4
            load: 80
      ```

      > **安全提示**：在生产环境运行混沌实验前，确保有完善的监控告警和快速恢复机制。

  - title: "实验设计与评估"
    subtitle: "从假设到结论的混沌实验方法论"
    content: |
      ## 实验设计框架

      ### 第一步：定义稳态假设

      ```
      稳态假设：在正常条件下，API 服务的 P99 延迟 < 200ms，
      错误率 < 0.1%，且自动扩缩容在 2 分钟内响应。
      ```

      ### 第二步：设计故障场景

      | 场景 | 注入方式 | 预期行为 |
      |------|---------|---------|
      | 单 Pod 崩溃 | PodChaos (pod-kill) | 服务不中断，新 Pod 自动拉起 |
      | 网络延迟 200ms | NetworkChaos (delay) | P99 延迟上升但不超过 500ms |
      | 依赖服务不可用 | NetworkChaos (loss) | 熔断器触发，降级返回缓存数据 |
      | CPU 满载 | StressChaos (cpu) | HPA 触发扩容，延迟短暂上升后恢复 |
      | 节点宕机 | NodeChaos (node-stop) | Pod 迁移到其他节点，服务短暂中断后恢复 |

      ### 第三步：执行与观察

      ```
      实验时间线：
      T+0:00  开始注入故障
      T+0:30  监控指标异常告警
      T+1:00  系统自动响应（扩容/熔断/迁移）
      T+2:00  指标开始恢复
      T+5:00  停止注入，系统完全恢复
      ```

      ### 第四步：评估与改进

      - 稳态假设是否成立？
      - 系统自动恢复时间是否符合预期？
      - 是否有未预见的级联故障？
      - 需要改进哪些组件或配置？

      > **关键产出**：每次混沌实验都应产生改进 Action Items，形成"实验→发现→改进→再实验"的正向循环。

---
