---
layout: book-article
title: "Helm Chart 模板化最佳实践"
excerpt: "深入 Helm 模板语法与 Chart 结构设计，实现可复用、可组合的 Kubernetes 应用包管理。"
category_link: /books/devops/
category_title: DevOps 读书笔记
icon: ""
date: 2025-03-15
status: completed
tags: [Helm, Kubernetes, 模板化]
chapters:
  - title: "Helm 核心概念回顾"
    subtitle: "Chart、Release、Repository 的关系"
    content: |
      ## Helm 是什么？

      Helm 是 Kubernetes 的包管理器，类似于 Linux 的 apt/yum，用于定义、安装和升级复杂的 Kubernetes 应用。

      ### 核心概念

      - **Chart**：一组描述 Kubernetes 资源的文件集合，是 Helm 的"安装包"
      - **Release**：Chart 在集群中的一次运行实例，同一 Chart 可以安装多个 Release
      - **Repository**：存放和共享 Chart 的服务器

      ### Chart 目录结构

      ```
      my-chart/
      ├── Chart.yaml          # Chart 元数据（名称、版本、描述）
      ├── values.yaml          # 默认配置值
      ├── values-schema.json   # 值校验 Schema（Helm 3.10+）
      ├── templates/           # Kubernetes 资源模板
      │   ├── deployment.yaml
      │   ├── service.yaml
      │   ├── configmap.yaml
      │   ── _helpers.tpl     # 模板辅助函数
      ├── charts/              # 子 Chart 依赖
      └── tests/               # 集成测试
      ```

      > **设计原则**：Chart 应该是自包含的，包含运行应用所需的全部 Kubernetes 资源定义。

  - title: "模板语法深度解析"
    subtitle: "Go 模板引擎在 Helm 中的应用"
    content: |
      ## 基础语法

      Helm 使用 Go 的 `text/template` 引擎，核心语法：

      ```yaml
      # 变量引用
      replicas: {{ .Values.replicaCount }}

      # 默认值
      image: {{ .Values.image.repository }}:{{ .Values.image.tag | default "latest" }}

      # 条件判断
      {{- if .Values.ingress.enabled }}
      apiVersion: networking.k8s.io/v1
      kind: Ingress
      {{- end }}

      # 循环
      {{- range .Values.ports }}
      - port: {{ .port }}
        targetPort: {{ .targetPort }}
      {{- end }}
      ```

      ### 命名模板（Named Templates）

      ```yaml
      # _helpers.tpl
      {{- define "myapp.fullname" -}}
      {{- printf "%s-%s" .Release.Name .Chart.Name | trunc 63 | trimSuffix "-" }}
      {{- end }}

      # 使用
      name: {{ include "myapp.fullname" . }}
      ```

      ### 常用管道函数

      | 函数 | 用途 | 示例 |
      |------|------|------|
      | `default` | 设置默认值 | `.Values.tag \| default "latest"` |
      | `quote` | 添加引号 | `.Values.name \| quote` |
      | `nindent` | 换行+缩进 | `\{\{ toYaml . \| nindent 4 \}\}` |
      | `required` | 必填校验 | `required "name is required" .Values.name` |
      | `toYaml` | 转 YAML 字符串 | `.Values.config \| toYaml` |

      > **最佳实践**：复杂逻辑放在 `_helpers.tpl` 中，保持资源模板文件简洁可读。

  - title: "Chart 设计模式"
    subtitle: "构建可复用、可组合的 Chart 架构"
    content: |
      ## Library Chart

      Library Chart 不包含可部署资源，只提供模板和辅助函数供其他 Chart 复用：

      ```yaml
      # Chart.yaml
      apiVersion: v2
      name: common-lib
      type: library
      version: 1.0.0
      ```

      ### 子 Chart 依赖管理

      ```yaml
      # Chart.yaml
      dependencies:
        - name: postgresql
          version: "12.x.x"
          repository: https://charts.bitnami.com/bitnami
          condition: postgresql.enabled
        - name: redis
          version: "17.x.x"
          repository: https://charts.bitnami.com/bitnami
          condition: redis.enabled
      ```

      ## 多环境管理策略

      | 策略 | 适用场景 | 实现方式 |
      |------|---------|---------|
      | 多 Values 文件 | 环境差异小 | `values-dev.yaml` / `values-prod.yaml` |
      | Kustomize Overlay | 环境差异大 | Helm 渲染后 Kustomize 叠加 |
      | 多 Release | 完全隔离 | 每个环境独立 Release |
      | App-of-Apps | 大规模管理 | ArgoCD + Helm 组合 |

      > **经验总结**：避免在模板中硬编码环境差异，通过 values 文件 + 条件判断实现灵活配置。

  - title: "测试与 CI/CD 集成"
    subtitle: "保障 Chart 质量的自动化流程"
    content: |
      ## Chart 测试

      ### helm lint

      ```bash
      helm lint ./my-chart
      # 检查 Chart 结构、模板语法、values 一致性
      ```

      ### helm template

      ```bash
      helm template my-release ./my-chart -f values-test.yaml
      # 渲染模板，检查输出是否符合预期
      ```

      ### helm test

      ```yaml
      # templates/tests/test-connection.yaml
      apiVersion: v1
      kind: Pod
      metadata:
        name: "{{ include "myapp.fullname" . }}-test"
        annotations:
          "helm.sh/hook": test
      spec:
        containers:
          - name: test
            image: busybox
            command: ['wget', '{{ include "myapp.fullname" . }}:{{ .Values.service.port }}']
        restartPolicy: Never
      ```

      ## CI/CD 流水线集成

      ```yaml
      # GitHub Actions
      - name: Lint Chart
        run: helm lint ./chart

      - name: Run Tests
        run: helm test my-release

      - name: Package Chart
        run: helm package ./chart

      - name: Publish to Registry
        run: helm push my-chart-1.0.0.tgz oci://registry.example.com/charts
      ```

      > **质量门禁**：lint 通过 + template 渲染无错误 + test 全部通过 = 允许发布。

---
