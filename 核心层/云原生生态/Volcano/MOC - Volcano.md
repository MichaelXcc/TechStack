---
type: moc
tags: [volcano, batch-scheduling, kubernetes, cncf]
title: Volcano专栏总览
created: 2026-01-06
updated: 2026-01-06
author: 云原生技术架构师
status: 学习中
priority: 高
related: ["Kubernetes", "调度器", "AI训练", "大数据"]
---

# Volcano专栏总览

## 📋 定义

Volcano是基于Kubernetes的云原生批处理调度系统，是CNCF孵化项目，专为AI、大数据、基因计算、渲染等高性能计算工作负载设计，提供强大的成组调度（Gang Scheduling）、层级队列管理和资源预留等能力。

## 🎯 核心价值

- **高性能调度**：专为大规模批处理和分布式训练优化
- **云原生原生**：无缝集成Kubernetes生态
- **多框架支持**：支持TensorFlow、PyTorch、Spark、Flink等多种计算框架
- **高级调度策略**：Gang Scheduling、Fair Sharing、Priority-based调度等
- **异构资源管理**：支持GPU、TPU、NPU等多种硬件加速设备

## 📁 目录结构

### 1. 核心概念
> 理解Volcano的基础概念和术语

- [[核心概念/Volcano核心概念|Volcano核心概念]]
  - [[核心概念/Gang Scheduling|Gang Scheduling]]
  - [[核心概念/Queue|Queue队列]]
  - [[核心概念/Job|Volcano Job]]
  - [[核心概念/Task|Task任务]]
  - [[核心概念/Resource Quota|资源配额]]

### 2. 架构设计
> 深入了解Volcano的系统架构和组件

- [[架构设计/Volcano架构设计|Volcano架构设计]]
  - [[架构设计/Scheduler|调度器]]
  - [[架构设计/Controller|控制器]]
  - [[架构设计/Admission Webhook|准入控制器]]
  - [[架构设计/CLI|命令行工具]]

### 3. 部署指南
> 学习如何在Kubernetes集群中部署和配置Volcano

- [[部署指南/Volcano部署指南|Volcano部署指南]]
  - [[部署指南/基于Helm部署|基于Helm部署]]
  - [[部署指南/基于YAML部署|基于YAML部署]]
  - [[部署指南/配置优化|配置优化]]
  - [[部署指南/版本升级|版本升级]]

### 4. 实践案例
> 通过实际案例学习Volcano的使用

- [[实践案例/Volcano实践案例|Volcano实践案例]]
  - [[实践案例/AI训练调度|AI训练调度]]
  - [[实践案例/大数据任务调度|大数据任务调度]]
  - [[实践案例/混合工作负载调度|混合工作负载调度]]
  - [[实践案例/多租户调度|多租户调度]]

### 5. 高级特性
> 深入学习Volcano的高级功能

- [[高级特性/Volcano高级特性|Volcano高级特性]]
  - [[高级特性/LeaderWorkerSet|LeaderWorkerSet]]
  - [[高级特性/节点亲和性|节点亲和性]]
  - [[高级特性/资源预留|资源预留]]
  - [[高级特性/事件驱动调度|事件驱动调度]]

## 🔗 相关链接

### 生态系统
- [[核心层/Kubernetes/MOC - Kubernetes|Kubernetes]]：Volcano的底层平台
- [[核心层/云原生生态/MOC - 云原生生态|云原生生态]]：Volcano所属的生态系统
- [[核心层/Operator/MOC - Operator开发|Operator开发]]：Volcano基于Operator模式

### 应用场景
- [[前瞻层/AI智能体/Megatron推理服务|Megatron推理服务]]：AI推理工作负载
- [[智能运维案例/AI故障诊断|AI故障诊断]]：AI应用案例


## 💬 思考问题

1. Volcano与Kubernetes原生调度器的区别是什么？
2. 为什么AI训练需要Gang Scheduling？
3. Volcano如何处理异构资源调度？
4. 在大规模集群中，如何优化Volcano的调度性能？
5. Volcano与其他批处理系统（如YARN）相比有哪些优势？
