---
title: "构建全栈可观测性体系"
date: 2024-01-05 09:00:00 +0800
categories: [监控, 可观测性]
tags: [Prometheus, Grafana, Loki, Tempo, 监控, 可观测性]
excerpt: "使用 Prometheus + Grafana + Loki + Tempo 构建企业级可观测性平台"
author_profile: true
---

# 构建全栈可观测性体系

本文介绍如何使用现代开源工具构建完整的可观测性平台，实现指标、日志和链路追踪的统一管理。

## 🎯 可观测性三大支柱

1. **指标**：数值型数据，用于监控和告警
2. **日志**：离散事件记录，用于故障诊断
3. **链路追踪**：请求路径追踪，用于性能分析

## 🏗️ 架构设计

### 整体架构

```
应用 → Prometheus Exporter → Prometheus → Alertmanager → Grafana
应用 → Log Agent → Loki → Grafana
应用 → Tracing Agent → Tempo → Grafana
```

## 📊 指标监控

### Prometheus 配置

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']

rule_files:
  - "alerts/*.yml"

scrape_configs:
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
```

### 告警规则

```yaml
groups:
- name: application
  rules:
  - alert: HighErrorRate
    expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "高错误率告警"
      description: "{{ $labels.instance }} 错误率超过 5%"
```

## 📝 日志管理

### Loki 配置

```yaml
server:
  http_listen_port: 3100

ingester:
  lifecycler:
    address: 127.0.0.1
    ring:
      store: consul
    max_transfer_retries: 0

schema_config:
  configs:
    - from: 2020-10-24
      store: boltdb
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 24h
```

### 日志采集

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: promtail-config
data:
  promtail.yaml: |
    server:
      http_listen_port: 9080

    clients:
      - url: http://loki:3100/loki/api/v1/push

    scrape_configs:
      - job_name: kubernetes-pods
        kubernetes_sd_configs:
          - role: pod
        relabel_configs:
          - source_labels: [__meta_kubernetes_pod_name]
            target_label: pod
          - source_labels: [__meta_kubernetes_namespace]
            target_label: namespace
```

## 🔗 链路追踪

### Tempo 配置

```yaml
server:
  http_listen_port: 3200

distributor:
  receivers:
    otlp:
      protocols:
        grpc:
        http:

storage:
  trace:
    backend: local
    local:
      path: /tmp/tempo/traces
```

### 应用集成

```go
import (
    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/exporters/jaeger"
    "go.opentelemetry.io/otel/sdk/trace"
)

func initTracer() {
    exporter, err := jaeger.New(jaeger.WithCollectorEndpoint(
        jaeger.WithEndpoint("http://jaeger:14268/api/traces"),
    ))
    if err != nil {
        log.Fatal(err)
    }
    
    tp := trace.NewTracerProvider(
        trace.WithBatcher(exporter),
    )
    otel.SetTracerProvider(tp)
}
```

## 📈 Grafana 仪表板

### 数据源配置

```json
{
  "name": "Prometheus",
  "type": "prometheus",
  "url": "http://prometheus:9090",
  "access": "proxy",
  "isDefault": true
}
```

### 仪表板示例

```json
{
  "dashboard": {
    "title": "应用监控",
    "panels": [
      {
        "title": "QPS",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])"
          }
        ]
      },
      {
        "title": "错误率",
        "targets": [
          {
            "expr": "rate(http_requests_total{status=~'5..'}[5m]) / rate(http_requests_total[5m])"
          }
        ]
      }
    ]
  }
}
```

## 🎯 SLO/SLI 指标体系

### SLI 定义

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: slo-config
data:
  availability-sli.yaml: |
    name: service_availability
    description: 服务可用性
    query: |
      sum(rate(http_requests_total{status!~"5.."}[5m])) /
      sum(rate(http_requests_total[5m]))
```

### SLO 告警

```yaml
- alert: SLOViolation
  expr: service_availability < 0.99
  for: 5m
  labels:
    severity: critical
  annotations:
    summary: "SLO 违规告警"
    description: "服务可用性低于 99%"
```

## 🔍 故障排查

### 统一查询

```bash
# 查询指标
curl 'http://prometheus:9090/api/v1/query?query=up'

# 查询日志
curl 'http://loki:3100/loki/api/v1/query_range?query={app="myapp"}'

# 查询链路
curl 'http://tempo:3200/api/search?traceID=xxx'
```

### 关联分析

通过 Grafana 实现三大支柱的关联：

1. 从告警跳转到相关日志
2. 从日志跳转到相关链路
3. 从链路跳转到相关指标

## 📚 最佳实践

1. **统一标签**：在所有系统中使用一致的标签体系
2. **采样策略**：对高流量链路进行采样
3. **数据保留**：根据业务需求设置合理的保留策略
4. **性能优化**：避免过度采集影响应用性能
5. **告警降噪**：设置合理的告警阈值和聚合规则

---

**作者：** Pizicaiman  
**发布时间：** 2024-01-05  
**分类：** 监控, 可观测性  
**标签：** Prometheus, Grafana, Loki, Tempo, 监控, 可观测性