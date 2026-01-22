---
type: moc
tags: [moc, 架构层, AIInfra]
title: AI基础设施MOC
created: 2026-01-13
updated: 2026-01-22
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

- [[AI-Infra核心架构/MOC - AI-Infra核心架构|AI-Infra 核心架构 MOC]]
  - [[AI-Infra核心架构/01_AI Infra架构设计概览|AI Infra 架构设计概览]]
  - [[AI-Infra核心架构/02_高性能计算集群设计|高性能计算集群设计]]
  - [[AI-Infra核心架构/03_调度与资源管理|调度与资源管理]]
  - [[AI-Infra核心架构/04_存储与数据架构|存储与数据架构]]

---

### GPU 性能优化
> GPU 性能测试、监控与调优

- [[GPU性能优化/MOC - GPU性能优化|GPU 性能优化 MOC]]
  - [[GPU性能优化/GPU性能-GEMM测试分析|GPU性能-GEMM测试分析]]
  - [[GPU性能优化/GPU性能指标与监控|GPU性能指标与监控]]
  - [[GPU性能优化/GPU性能调优最佳实践|GPU性能调优最佳实践]]

---

### 推理服务
> 大模型推理框架与部署

- [[推理服务/MOC - 推理服务|推理服务 MOC]]
  - [[推理服务/Megatron-LM vs. vLLM vs. SGLang|推理框架对比分析]]
  - [[推理服务/Megatron推理服务|Megatron 推理服务]]
  - [[推理服务/vLLM推理服务|vLLM 推理服务]]

---

### 分布式计算框架
> 分布式AI计算与调度

- [[Ray/MOC - Ray|Ray 分布式计算框架]]
  - Ray Core 核心概念
  - 集群架构与内存管理
  - 生态系统 (Train, Serve, Data, Tune)

---

### GPU 虚拟化与共享
> 细粒度 GPU 资源管理

- [[HAMi/MOC - HAMi|HAMi GPU 虚拟化中间件]]
  - GPU 共享与资源隔离
  - 调度策略与拓扑感知
  - 多租户 GPU 管理

---

### 批量调度与拓扑感知
> 高性能 AI 任务调度

- [[Volcano/MOC - Volcano|Volcano 批量调度系统]]
  - Gang Scheduling 与 Queue 管理
  - 网络拓扑感知调度 (HyperNode)
  - 分布式训练调度优化

---

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
