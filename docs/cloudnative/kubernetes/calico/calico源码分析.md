---
layout: doc
title: Calico源码分析
date: 2024-06-25
category: kubernetes
tags: [calico, kubernetes, cni, source-code]
excerpt: 深入解析Calico核心组件架构与关键源码实现，助力理解Kubernetes网络方案底层原理。
permalink: /docs/cloudnative/kubernetes/calico/source-code-analysis/
---

# Calico源码分析

Calico 是云原生环境下主流的网络与网络安全方案，支持多种数据平面（纯三层、BPF、VXLAN 等），具备良好的性能与可扩展性。本文围绕 Calico 源码架构，结合核心组件、模块职责及设计实现进行梳理，帮助读者高效入门源码级调试和定制开发。

---

## 目录

- [Calico源码分析](#calico源码分析)
  - [目录](#目录)
  - [1. Calico 架构总览](#1-calico-架构总览)
  - [2. 核心组件与职责](#2-核心组件与职责)
  - [3. 关键源码目录结构](#3-关键源码目录结构)
  - [4. Felix 主流程与关键数据结构](#4-felix-主流程与关键数据结构)
  - [5. 数据平面实现分析](#5-数据平面实现分析)
  - [6. Calico 与 Kubernetes 集成流程](#6-calico-与-kubernetes-集成流程)
  - [7. 常见调试与开发技巧](#7-常见调试与开发技巧)
  - [8. 参考与延伸](#8-参考与延伸)

---

## 1. Calico 架构总览

Calico 主要组件包括：

- **calico/node**：核心 Daemon，运行在每个节点，完成路由配置，管理 BGP、IP IP 隧道等
- **Felix**：策略引擎，负责监听数据存储(如 etcd、Kubernetes API)，下发安全策略，实现网络隔离
- **Typha**：集群优化组件，降低大规模场景下 API Server 压力
- **CNI 插件**：为 Pod 动态分配 IP，并与主机网络集成

整体架构如下：

```
 K8s API/etcd
      │
  [Typha] (可选)
      │
[calico/node↔Felix]——[BGP/vxlan/tunnel]——
      │                          │
   [CNI插件]—————> [Pod/Workload Endpoint]
```

---

## 2. 核心组件与职责

| 组件         | 说明                                      |
|--------------|-------------------------------------------|
| calico/node  | 包含 Felix 与 BGP Daemon，负责节点间路由  |
| Felix        | 解析策略，编译为底层数据平面规则           |
| Typha        | 可选，用于提升大规模集群下控制面性能       |
| CNI Plugin   | 创建Pod网络接口，添加到OVS或linux网络栈    |

---

## 3. 关键源码目录结构

Calico 用 Go 语言实现，典型 repo 结构（以 [projectcalico/felix](https://github.com/projectcalico/felix) 为例）：

- `pkg/`        核心功能包（如 datamodel, ipsets, policy, routing等）
- `daemon/`     Felix 主进程入口
- `config/`     配置加载与解析
- `dataplane/`  数据平面抽象与实现（iptables、bpf等）
- `ext/dataplane/linux/`   Linux 平台具体实现

CNI 插件项目为 [calico/cni-plugin](https://github.com/projectcalico/cni-plugin)；Typha 实现参见 [calico/typha](https://github.com/projectcalico/typha)。

---

## 4. Felix 主流程与关键数据结构

**Felix** 作为 Calico 的大脑，负责：

1. Watch K8s API/etcd，监听 NetworkPolicy/WorkloadEndpoint 等变化
2. 维护本地缓存（Policy、Endpoint、IPSet等）
3. 编译策略→数据平面对象（如 iptables Chain，bpf map）
4. 调用系统命令或内核接口，写入规则

Felix 主循环伪代码简化如下：

```go
func main() {
    // 加载配置与初始化日志
    client := NewDatastoreClient()
    // 启动k8s/etcd Watcher
    go client.Watch(func(update Update) {
        // 下发策略
        ApplyPolicy(update)
    })
    // 数据面管理主循环
    for {
        // 检查本地策略/endpoint/state
        // 若有变更，生成iptables或bpf规则
        // 调用系统接口批量写入
    }
}
```

**主要结构体举例**（位于`pkg/dataplane`等）：

```go
type Policy struct {
    Selector    string
    Ingress     []Rule
    Egress      []Rule
}
type Rule struct {
    Protocol    string
    Src, Dst    Selector
    Action      string // Allow/Deny
}
type Endpoint struct {
    Name        string
    IP          net.IP
    Labels      map[string]string
}
```

---

## 5. 数据平面实现分析

Calico 支持多种数据平面，核心包括：

- **iptables** (最广泛)：策略以 chain/规则下发
- **BPF dataplane**（高性能）：基于 eBPF map/object 动态分发
- **VXLAN/Ipip**：隧道封装作为跨节点通信方案

以 iptables 数据平面为例：

- `felix/pkg/dataplane/linux/iptables.go`：封装 chain/rule 管理及刷写逻辑
- 数据变更→生成新 chain/rule → 批量应用到系统 iptables

eBPF 数据平面主要管理策略与 Endpoint Map，通过 `/sys/fs/bpf` 目录与 Kernel 交互。

---

## 6. Calico 与 Kubernetes 集成流程

1. **Pod 创建触发 CNI**  
   - `calico/cni-plugin` 执行，为 Pod 分配 IP、写入 etcd/K8s API
2. **Felix Watch CRD/etcd**  
   - 网络策略、Endpoint、IPPool 等变更实时同步
3. **路由&策略同步**  
   - Felix 下发数据面规则，保证安全与转发
4. **BGP/vxlan 自动组网**  
   - 节点间路由自动建立

K8s-CRD 模式下，Calico 主要依赖 API Server 存储配置、状态。

---

## 7. 常见调试与开发技巧

- **日志分析**  
  Felix 可通过 `FELIX_LOGSEVERITYSCREEN=debug` 获取详细追踪信息
- **策略流程追踪**  
  关注 Felix watch/policy sync 日志，抓包对比 chain 变动
- **CNI 问题排查**  
  /var/log/calico/cni/ 目录与 CNI 插件返回码详解
- **数据平面回滚/批量调试**  
  通过 `calicoctl node status / iptables-save`/`bpftool` 查看现有链与对象

---

## 8. 参考与延伸

- [Calico GitHub 源码](https://github.com/projectcalico/calico)
- [Felix 代码详解](https://github.com/projectcalico/felix)
- [官方设计文档](https://docs.projectcalico.org)
- [BPF dataplane 设计](https://docs.projectcalico.org/reference/architecture/bpf)

---

如需针对某模块/源码文件深度解读，请留言交流！

