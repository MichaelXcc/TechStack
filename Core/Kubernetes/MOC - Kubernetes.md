---
type: MOC
tags: [kubernetes, 核心层, MOC]
title: Kubernetes核心概念MOC
status: 已掌握
priority: 高
---

# Kubernetes核心概念MOC

## 📚 概述

本文档是Kubernetes核心概念的知识图谱，涵盖了Kubernetes的核心概念、工作负载、存储、网络、配置等方面。

## 🎯 核心概念

### 基础概念
- [[Pod概念]]：Kubernetes最小的部署单元
- [[Node概念]]：Kubernetes集群的工作节点
- [[Namespace概念]]：资源隔离的逻辑分组
- [[Controller概念]]：管理资源状态的控制器

### 工作负载
- [[Deployment概念]]：管理无状态应用
- [[StatefulSet概念]]：管理有状态应用
- [[DaemonSet概念]]：在每个节点运行Pod
- [[Service概念]]：服务发现和负载均衡

### 配置和存储
- [[ConfigMap概念]]：非敏感配置管理
- [[Secret概念]]：敏感配置管理
- [[Volume概念]]：存储数据抽象

### 网络
- [[Ingress概念]]：HTTP/HTTPS路由规则

## 📊 知识图谱

```
Kubernetes核心概念
├── 基础概念
│   ├── Pod (最小部署单元)
│   ├── Node (工作节点)
│   ├── Namespace (资源隔离)
│   └── Controller (状态管理)
├── 工作负载
│   ├── Deployment (无状态应用)
│   ├── StatefulSet (有状态应用)
│   └── DaemonSet (节点级应用)
├── 配置和存储
│   ├── ConfigMap (配置管理)
│   ├── Secret (敏感配置)
│   └── Volume (存储抽象)
└── 网络
    ├── Service (服务发现)
    └── Ingress (HTTP路由)
```

## 🔗 概念关系

### Pod相关
- Pod是Kubernetes最小的部署单元
- Deployment、StatefulSet、DaemonSet都管理Pod
- Service为Pod提供稳定的服务发现

### 存储相关
- Volume为Pod提供存储能力
- ConfigMap和Secret可以作为Volume使用
- StatefulSet使用PVC实现持久化存储

### 网络相关
- Service为Pod提供集群内服务发现
- Ingress为集群提供HTTP/HTTPS入口
- Namespace实现网络隔离

### 控制相关
- Controller管理所有资源的状态
- Deployment、StatefulSet、DaemonSet都是Controller的实现
- Controller通过控制循环实现状态协调

## 🚀 学习路径

### 初级阶段
1. 理解Pod和Node的基本概念
2. 学习Deployment的使用
3. 掌握Service的服务发现
4. 了解ConfigMap和Secret的配置管理

### 中级阶段
1. 深入理解StatefulSet的有状态应用管理
2. 学习DaemonSet的节点级应用部署
3. 掌握Volume的存储管理
4. 理解Ingress的HTTP路由

### 高级阶段
1. 深入理解Controller的工作原理
2. 学习自定义Controller和Operator开发
3. 掌握Kubernetes的网络模型
4. 理解Kubernetes的调度机制

## 💡 架构视角

### 微服务架构
- 使用Deployment管理无状态微服务
- 使用Service实现服务发现和负载均衡
- 使用ConfigMap和Secret管理配置
- 使用Ingress暴露HTTP服务

### 有状态应用
- 使用StatefulSet管理数据库等有状态应用
- 使用PVC实现数据持久化
- 使用Headless Service提供稳定的网络标识

### 系统级服务
- 使用DaemonSet部署监控、日志等系统服务
- 使用Node访问节点资源
- 使用污点和容忍度控制Pod调度

## 🔍 深入学习

### 相关文档
- [Kubernetes官方文档](https://kubernetes.io/docs/)
- [Kubernetes API参考](https://kubernetes.io/docs/reference/kubernetes-api/)

### 控制平面
- [[控制平面/API Server概念]]：API Server详解
- [[控制平面/Scheduler概念]]：调度器详解
- [[控制平面/Controller Manager概念]]：控制器管理器详解
- [[控制平面/etcd概念]]：etcd存储详解

### 数据平面
- [[数据平面/Kubelet概念]]：Kubelet详解
- [[数据平面/Kube-proxy概念]]：Kube-proxy详解
- [[数据平面/容器运行时概念]]：容器运行时详解

### 网络
- [[网络/Kubernetes网络模型]]：Kubernetes网络模型详解
- [[网络/Service网络]]：Service网络详解
- [[网络/NetworkPolicy]]：网络策略详解

### 存储
- [[存储/PV和PVC概念]]：PV和PVC详解
- [[存储/StorageClass概念]]：StorageClass详解

### 实践案例
- [[生产环境K8s部署]]：生产环境部署实践
- [[微服务架构]]：微服务架构设计
- [[故障排查与恢复]]：故障排查和恢复

## 📝 总结

Kubernetes核心概念构成了云原生应用的基础设施：
- **Pod**：最小部署单元
- **Controller**：自动化管理
- **Service**：服务发现
- **ConfigMap/Secret**：配置管理
- **Volume**：存储抽象
- **Ingress**：HTTP路由

理解这些核心概念是掌握Kubernetes的关键，也是构建云原生应用的基础。
