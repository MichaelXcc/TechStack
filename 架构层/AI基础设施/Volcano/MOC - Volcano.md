---
type: moc
tags: [moc, 架构层, AIInfra, Volcano, 批量调度]
title: Volcano MOC
created: 2026-01-21
updated: 2026-01-21
author: AI Agent
---

# Volcano - Kubernetes 批量调度系统

## 🎯 目标
- 理解 Volcano 批量调度的核心机制
- 掌握网络拓扑感知调度技术
- 学会配置 HyperNode 和调度策略
- 优化 AI 大模型训练的调度效率

## 📁 目录结构

### 基础概念
> Volcano 核心理念与特性

- [[01_Volcano概述与核心概念|Volcano 概述与核心概念]]
  - 什么是 Volcano
  - Gang Scheduling
  - Queue 管理

### 网络拓扑感知调度
> 核心特性深入解析

- [[02_网络拓扑感知调度原理|网络拓扑感知调度原理]]
  - HyperNode 抽象
  - 拓扑层级设计
  - 调度决策流程

- [[03_HyperNode配置与自动发现|HyperNode 配置与自动发现]]
  - HyperNode CRD
  - 拓扑自动发现
  - 手动配置

### 实战应用
> 生产环境配置

- [[04_拓扑感知调度实战|拓扑感知调度实战]]
  - 分布式训练调度
  - 多层级拓扑配置
  - 最佳实践

## 🔗 相关链接

- [[MOC - AI基础设施|AI 基础设施 MOC]]
- [[HAMi/MOC - HAMi|HAMi GPU 虚拟化]]
- [Volcano 官方文档](https://volcano.sh)
- [Volcano GitHub](https://github.com/volcano-sh/volcano)

## 📊 核心特性

| 特性 | 说明 |
|------|------|
| **Gang Scheduling** | 确保分布式任务原子调度 |
| **Queue 管理** | 多队列资源配额控制 |
| **网络拓扑感知** | 基于网络拓扑优化调度 |
| **NUMA 感知** | CPU/内存 NUMA 亲和调度 |
| **公平调度** | DRF 等公平调度算法 |
