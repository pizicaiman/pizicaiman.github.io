---
layout: doc
title: Cilium 源码分析
date: 2024-06-25
category: kubernetes
tags: [calico, kubernetes, cni, source-code]
excerpt: 深入解析Cilium核心组件架构与关键源码实现，助力理解Kubernetes网络方案底层原理。
permalink: /docs/cloudnative/kubernetes/cilium/source-code-analysis/
---

# Cilium 源码分析

Cilium 是基于 eBPF 技术实现的云原生容器网络与安全方案，具备高性能数据面、强安全性和丰富网络可观测能力。下面简要梳理 Cilium 的核心架构与主要源码实现路径，并提供源码分析及追踪建议。

---

## 1. 核心架构与组件

- **Cilium Agent：** Cilium 的主守护进程，负责 CRD Watch、策略渲染、eBPF 程序下发、数据面监测、状态同步等。
- **Operator：** 集群控制平面辅助组件，负责 Node 信息同步、IPAM、LB 等操作。
- **CNI 插件：** Pod 网络加入/删除的触发入口，调用 Cilium API 实际分配、回收网络资源。
- **eBPF 程序：** 位于内核的数据面实现，核心包括流量过滤、负载均衡、网络透明代理。

---

## 2. 核心源码结构

Cilium 项目地址：https://github.com/cilium/cilium

主要目录结构如下：

- `daemon/`：Cilium Agent 主入口和控制代码
- `pkg/`：核心功能库，实现策略、LB、IPAM、状态同步、eBPF 管理等
- `bpf/`：主要的 eBPF C 源码
- `cilium-cni/`：CNI 插件实现
- `operator/`：各类 cluster operator
- `api/`：CRD 定义与协议

---

## 3. 启动流程与关键调用链

1. **Cilium Agent 启动**
   - 入口：`daemon/main.go`
   - 加载配置参数、初始化日志、启动 CRD Watch（如 CiliumNetworkPolicy、Endpoint、Node）
2. **BPF 程序加载与下发**
   - 调用 `pkg/datapath/loader/` 完成 eBPF 程序编译、Attach/Detach 到网卡（TC、XDP）、挂接到对应位置
3. **Pod 创建时的 CNI 交互**
   - CNI 调用 `cilium-cni`，通过 Unix Domain Socket/API 与 Agent 通信
   - Agent 分配 Pod IP，生成 Endpoint，更新状态
   - 通过 `pkg/endpoint` 增/删网络命名空间映射
4. **策略、LB 实时更新**
   - 通过 Watcher 实时监听 CRD
   - 触发渲染新策略、LB 配置，Agent 自动刷新 BPF Map 等内核状态对象

---

## 4. 数据面流量转发与安全策略核心逻辑

- **eBPF 流量转发：**
  - L3 路由/L4 透明代理/L7 协议可观测与增强处理等全部逻辑在 `bpf/` 内各 .c 源码
  - 用户空间 Agent 通过 Map 管理策略、映射、端点
- **安全策略实现：**
  - Cilium 支持网络层、协议层丰富的策略模型，policy 渲染代码主要在 `pkg/policy/`
  - 策略变更后，Agent 将规则同步到 BPF Map，并通知 bpf 程序即时生效

---

### 源码分析建议

- **常用入口点：**
    - `daemon/main.go` （Agent 启动总入口）
    - `pkg/endpoint/`（Pod Endpoint 生命周期管理、BPF map 数据操作）
    - `pkg/datapath/loader/`（BPF 程序加载与挂载核心）
    - `pkg/policy/`、`bpf/`（安全策略渲染、内核旁路）
- **调试技巧：**
    - 开启 debug 日志：`cilium-agent --debug`
    - `cilium monitor` 命令实时抓包
    - 查看 BPF map：`bpftool map show`、`bpftool prog show`
    - CRD、状态检查：`kubectl get ciliumnetworkpolicy` 等

---

## 5. 示例调用链追踪

以 Pod 创建后访问 Service 为例，核心调用链如下：

1. K8s 调用 CNI（cilium-cni），发起 ADD
2. Agent 分配 IP、初始化 Endpoint
3. Agent 下发策略和 BPF 编程
4. Pod 流量经过网卡，触发 BPF 程序（如 tc ingress）
5. 实时根据 policy/LB Map 处理，决定转发、重定向、拦截等

---

## 6. 进阶阅读与参考

- [Cilium 官方文档](https://docs.cilium.io/)
- [Cilium GitHub 源码](https://github.com/cilium/cilium)
- [eBPF 学习资料](https://ebpf.io/)

如需具体模块、BPF 源码、策略下发等更详细解析，欢迎留言交流！

