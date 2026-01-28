---
type: moc
tags: [moc, 核心层, 云原生]
title: 核心层MOC
author: 云原生技术架构师
---

# 核心层 - 云原生技术栈

## 🎯 目标
- 深入理解Kubernetes内部原理
- 掌握Operator开发技术
- 熟悉云原生生态系统
- 能够设计和优化云原生架构

## 📁 目录结构

### Kubernetes
> 容器编排与管理平台，云原生的核心

- [[Kubernetes/MOC - Kubernetes|Kubernetes MOC]]
  - 核心概念与架构
  - 控制平面与数据平面
  - Pod生命周期管理
  - 调度器原理
  - 服务发现与网络
  - 存储系统
  - 安全机制
  - 性能优化

### Operator开发
> 扩展Kubernetes能力的最佳实践

- [[Operator/MOC - Operator开发|Operator开发 MOC]]
  - Operator模式原理
  - 控制器模式
  - 使用Go开发Operator
  - Operator SDK与Framework
  - 最佳实践与案例

### 云原生生态
> 围绕Kubernetes的完整生态系统

- [[云原生生态/MOC - 云原生生态|云原生生态 MOC]]
  - 服务网格（Istio, Linkerd）
  - 容器运行时（Containerd, CRI-O）
  - CI/CD（Tekton, Argo CD）
  - 监控与可观测性（Prometheus, Grafana, Jaeger）
  - 日志管理（ELK Stack, Loki）
  - 安全工具（Trivy, Falco）

## 🔗 相关链接

### 架构设计
- [[云原生架构设计原则]]
- [[微服务架构与K8s]]
- [[多集群管理]]

### 实践案例
- [[生产环境K8s部署]]
- [[大规模集群优化]]
- [[故障排查与恢复]]

## 📊 技能图谱
```dataview
LIST
FROM "核心层"
WHERE type = "skill"
SORT tags ASC
```
