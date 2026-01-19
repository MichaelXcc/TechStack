---
type: moc
tags: [moc, 架构层, AIInfra]
title: AI基础设施MOC
created: 2026-01-13
updated: 2026-01-13
author: AI Agent
---

# AI基础设施 (AI Infra) 架构设计

## 🎯 目标
- 构建高性能、高可用的大模型训练与推理基础设施
- 掌握AI算力集群的规划与设计
- 理解软硬协同优化技术
- 打造高效的MLOps/LLMOps平台

## 📁 目录结构

### 核心架构
> AI Infra 的整体设计理念与蓝图

- [[01_AI Infra架构设计概览|AI Infra 架构设计概览]]
  - 基础设施层（计算、存储、网络）
  - 调度层（Kubernetes, Slurm）
  - 框架层（PyTorch, TensorFlow）
  - 平台层（训练、推理、数据管理）

### 关键组件
> 深入各个子系统的设计

- [[02_高性能计算集群设计|高性能计算集群设计]]
  - GPU选型与拓扑 (NVLink/NVSwitch)
  - RDMA网络架构 (InfiniBand/RoCE)
  - 存储系统选型 (并行文件系统/对象存储)
- [[03_调度与资源管理|调度与资源管理]]
  - 任务调度 (Gang Scheduling, Bin-packing)
  - 弹性扩缩容
  - 故障容错与Checkpointing
- [[04_存储与数据架构|存储与数据架构]]
  - 高性能Checkpoint存储
  - 数据湖与缓存系统

### 分布式计算框架
> 分布式AI计算与调度

- [[Ray/MOC - Ray|Ray 分布式计算框架]]
  - Ray Core 核心概念
  - 集群架构与内存管理
  - 生态系统 (Train, Serve, Data, Tune)

### 推理服务
- [[05_推理服务架构|推理服务架构]]
  - vLLM/TRT-LLM 优化
  - 连续批处理 (Continuous Batching)
  - KV Cache 优化

## 🔗 相关链接

- [[架构层/MOC - 架构层|架构层 MOC]]
- [[核心层/Kubernetes/MOC - Kubernetes|Kubernetes MOC]]
- [[基础层/CUDA/MOC - CUDA编程|CUDA编程]]

## 📊 技能图谱
```dataview
LIST
FROM "架构层/AI基础设施"
WHERE type = "skill"
SORT tags ASC
```
