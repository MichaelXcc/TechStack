---
type: moc
tags: [moc, 架构层, AIInfra, Ray, 分布式计算]
title: Ray 分布式计算框架 MOC
---

# Ray 分布式计算框架

## 🎯 目标
- 掌握 Ray 分布式计算框架的核心原理
- 理解 Ray 集群架构与资源调度机制
- 熟练使用 Ray 进行分布式 AI 训练与推理
- 掌握 Ray 生态系统（Train, Serve, Data, Tune）的应用

## 📁 目录结构

### 基础入门
> Ray 的核心概念与快速上手

- [[01_Ray概述与核心概念|Ray 概述与核心概念]]
  - 什么是 Ray
  - 核心抽象：Task 与 Actor
  - 分布式对象存储
- [[02_Ray启动命令参数详解|Ray 启动命令参数详解]] ⭐
  - `ray start` 命令详解
  - Head 节点与 Worker 节点配置
  - 资源配置与端口设置

### 架构与原理
> 深入理解 Ray 的内部机制

- [[03_Ray集群架构|Ray 集群架构]]
  - GCS (Global Control Store)
  - Raylet 与 Worker 进程
  - 分布式调度机制
- [[04_Ray对象存储与内存管理|Ray 对象存储与内存管理]]
  - Plasma 对象存储
  - 内存管理策略
  - 溢出至磁盘机制

### 生态系统
> Ray 的高级组件库

- [[05_Ray生态系统组件|Ray 生态系统组件]]
  - Ray Data：分布式数据处理
  - Ray Train：分布式训练
  - Ray Serve：模型服务
  - Ray Tune：超参调优

## 🔗 相关链接

- [[架构层/AI基础设施/MOC - AI基础设施|AI 基础设施 MOC]]
- [[架构层/AI基础设施/03_调度与资源管理|调度与资源管理]]
- [[核心层/Kubernetes/MOC - Kubernetes|Kubernetes MOC]]

## 📚 外部资源

- [Ray 官方文档](https://docs.ray.io/)
- [Ray GitHub](https://github.com/ray-project/ray)
- [Ray on Kubernetes (KubeRay)](https://ray-project.github.io/kuberay/)
