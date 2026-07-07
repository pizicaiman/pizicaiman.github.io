---
layout: book-article
title: "SLI/SLO 指标体系设计实战"
excerpt: "从用户视角定义服务质量指标，构建可量化、可告警、可度量的 SLO 体系。"
category_link: /books/sre/
category_title: SRE 读书笔记
icon: "🛡️"
date: 2025-04-18
status: reading
tags: [SLI, SLO, 可观测性]
chapters:
  - title: "SLI/SLO/SLA 概念辨析"
    subtitle: "理解服务质量度量的三层体系"
    content: |
      ## 核心定义

      - **SLI（Service Level Indicator）**：服务等级指标，衡量服务某个方面的量化指标
      - **SLO（Service Level Objective）**：服务等级目标，SLI 应该达到的目标值
      - **SLA（Service Level Agreement）**：服务等级协议，未达 SLO 时的商业后果

      ### 三者关系

      ```
      SLI:  实际测量值（如 99.5% 请求成功）
            ↓ 对比
      SLO:  目标值（如 99.9% 请求成功）
            ↓ 未达标时触发
      SLA:  商业协议（如赔偿 credits）
      ```

      ### 常见误区

      | 误区 | 正确理解 |
      |------|---------|
      | SLO = 100% | SLO 应留有错误预算，通常 99%~99.99% |
      | 指标越多越好 | 聚焦用户感知的关键指标，避免指标疲劳 |
      | SLA 由技术团队制定 | SLA 是商业协议，需产品/法务参与 |
      | SLO 一旦设定不变 | SLO 应随业务发展和用户期望动态调整 |

      > **核心原则**：好的 SLI 应该直接反映用户体验，而非内部系统指标。

  - title: "四大黄金信号实践"
    subtitle: "Google SRE 推荐的通用 SLI 框架"
    content: |
      ## 黄金信号

      Google 在 SRE 书中提出四个通用 SLI 类别：

      ### 1. 延迟（Latency）

      请求处理所需的时间。区分成功请求和失败请求的延迟。

      ```promql
      # P99 延迟 SLI
      histogram_quantile(0.99,
        rate(http_request_duration_seconds_bucket{job="api"}[5m])
      )
      ```

      ### 2. 流量（Traffic）

      系统承载的需求量，通常用 QPS/RPS 衡量。

      ```promql
      sum(rate(http_requests_total{job="api"}[5m]))
      ```

      ### 3. 错误率（Errors）

      请求失败的比例。

      ```promql
      # 错误率 SLI
      sum(rate(http_requests_total{job="api", status=~"5.."}[5m]))
      /
      sum(rate(http_requests_total{job="api"}[5m]))
      ```

      ### 4. 饱和度（Saturation）

      系统资源的使用程度，预示即将出现的问题。

      ```promql
      # CPU 饱和度
      avg(rate(node_cpu_seconds_total{mode="idle"}[5m]))
      ```

      ## 面向用户的 SLI 设计

      | 服务类型 | 推荐 SLI | SLO 示例 |
      |---------|---------|---------|
      | Web API | 请求成功率 + P95 延迟 | 99.9% 成功，P95 < 200ms |
      | 存储系统 | 读/写成功率 + 延迟 | 99.99% 读成功，P99 读 < 50ms |
      | 批处理 | 作业完成率 + 完成时间 | 99% 作业在 SLA 时间内完成 |
      | 消息队列 | 消息投递成功率 + 端到端延迟 | 99.95% 消息在 1s 内投递 |

      > **设计建议**：从用户旅程出发定义 SLI，而非从系统组件出发。

  - title: "错误预算驱动决策"
    subtitle: "将 SLO 转化为可操作的工程决策框架"
    content: |
      ## 错误预算计算

      ```
      错误预算 = 1 - SLO

      例如 SLO = 99.9% → 错误预算 = 0.1%
      月度请求量 1 亿 → 允许 10 万次失败
      ```

      ## 预算消耗与决策矩阵

      | 预算剩余 | 决策策略 |
      |---------|---------|
      | > 70% | 加速发布，尝试大胆变更 |
      | 30-70% | 正常发布节奏，关注稳定性 |
      | 10-30% | 减缓发布，优先修复稳定性问题 |
      | < 10% | 冻结非紧急变更，全力修复 |

      ## 实施步骤

      1. **定义 SLO**：与产品团队协商确定用户可接受的服务水平
      2. **建立监控**：实时计算 SLI 并展示预算消耗 Dashboard
      3. **制定策略**：明确预算耗尽时的自动化响应（冻结部署、告警升级）
      4. **定期回顾**：每月评估 SLO 合理性，根据业务发展调整

      ### 多窗口多燃烧率告警

      ```yaml
      # Prometheus 告警规则
      groups:
        - name: slo-burn-rate
          rules:
            - alert: HighBurnRate
              expr: |
                (
                  (1 - (sum(rate(success_requests[1h])) / sum(rate(total_requests[1h]))))
                  / (1 - 0.999)
                ) > 14.4
              for: 5m
              labels:
                severity: critical
              annotations:
                summary: "SLO 消耗速度过快（1小时窗口，14.4x 燃烧率）"
      ```

      > **关键洞察**：错误预算不是限制创新的枷锁，而是让团队在"速度"和"稳定"之间做出明智决策的指南针。

  - title: "SLO 落地工具链"
    subtitle: "从指标采集到 Dashboard 的完整工具链"
    content: |
      ## 工具选型

      | 环节 | 开源方案 | 商业方案 |
      |------|---------|---------|
      | 指标采集 | Prometheus / OpenTelemetry | Datadog / New Relic |
      | SLO 管理 | Sloth / Nobl9 | Grafana SLO / Lightstep |
      | 告警 | Alertmanager | PagerDuty / OpsGenie |
      | Dashboard | Grafana | Datadog Dashboard |
      | 错误预算报告 | 自定义脚本 | Nobl9 / Polar Signals |

      ## Grafana SLO 配置示例

      ```json
      {
        "slo": {
          "name": "API Availability",
          "objective": 99.9,
          "query": {
            "good": "sum(rate(http_requests_total{status!~\"5..\"}[5m]))",
            "total": "sum(rate(http_requests_total[5m]))"
          },
          "alerting": {
            "fastBurn": { "threshold": 14.4, "window": "1h" },
            "slowBurn": { "threshold": 6, "window": "6h" }
          }
        }
      }
      ```

      ## 最佳实践总结

      1. **从少量关键 SLO 开始**：先覆盖核心用户路径，再逐步扩展
      2. **SLO 属于团队而非个人**：开发和 SRE 共同承担
      3. **自动化预算监控**：不要依赖人工检查
      4. **定期校准**：每季度回顾 SLO 是否仍然反映用户期望

      > **最终目标**：SLO 体系让"系统是否健康"这个问题有明确的、数据驱动的答案。

---
