---
type: moc
tags: [moc, 架构层, AIInfra, 核心架构]
title: AI-Infra核心架构 MOC
---

# AI-Infra 核心架构

> 本目录包含 AI 基础设施的整体架构设计文档，涵盖从物理基础设施到平台层的完整架构蓝图。

## 📁 目录内容

### 架构概览
- [[01_AI Infra架构设计概览|AI Infra 架构设计概览]]
  - 四层架构模型（物理层、调度层、框架层、平台层）
  - 设计原则与关键挑战
  - 全栈协同优化理念

### 计算与网络
- [[02_高性能计算集群设计|高性能计算集群设计]]
  - GPU 模组架构（HGX/NVLink/NVSwitch）
  - 网络拓扑设计（Fat-Tree/Rail-Optimized）
  - 物理部署与散热方案

### 调度系统
- [[03_调度与资源管理|调度与资源管理]]
  - Gang Scheduling 与拓扑感知
  - 资源隔离（MIG/MPS/Time-Slicing）
  - 故障容错与弹性训练

### 存储架构
- [[04_存储与数据架构|存储与数据架构]]
  - 分层存储（热/温/冷）
  - Checkpoint 优化技术
  - 数据格式与加载优化

## 🔗 相关链接

- [[../MOC - AI基础设施|返回 AI 基础设施 MOC]]
- [[../Volcano/MOC - Volcano|Volcano 批量调度]]
- [[../HAMi/MOC - HAMi|HAMi GPU 虚拟化]]
- [[../Ray/MOC - Ray|Ray 分布式计算]]
