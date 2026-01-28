---
type: moc
tags: [moc, 架构层, AIInfra, HAMi, GPU虚拟化]
title: HAMi MOC
---

# HAMi - Kubernetes GPU 虚拟化中间件

## 🎯 目标
- 理解 GPU 虚拟化与共享技术原理
- 掌握 HAMi 架构设计与核心组件
- 学会部署和配置 HAMi
- 实现 GPU 资源的细粒度管理

## 📁 目录结构

### 基础概念
> HAMi 核心理念与关键特性

- [[01_HAMi概述与核心概念|HAMi 概述与核心概念]]
  - 什么是 HAMi
  - GPU 共享技术对比
  - 核心特性与优势

### 架构设计
> 深入理解 HAMi 工作原理

- [[02_HAMi架构与组件|HAMi 架构与组件]]
  - 整体架构设计
  - 核心组件详解
  - 工作流程分析

### 部署运维
> 安装配置与日常运维

- [[03_HAMi安装与配置|HAMi 安装与配置]]
  - 环境准备
  - Helm Chart 部署
  - 配置参数详解

### 资源管理
> GPU 资源隔离与共享机制

- [[04_GPU资源隔离与共享|GPU 资源隔离与共享]]
  - 内存隔离原理
  - 算力限制机制
  - vCUDA 技术实现

### 调度优化
> 智能调度与资源优化

- [[05_HAMi调度策略|HAMi 调度策略]]
  - Binpack vs Spread
  - 拓扑感知调度
  - 调度策略配置

### 实战应用
> 真实场景案例

- [[06_HAMi实战案例|HAMi 实战案例]]
  - 推理服务共享 GPU
  - 开发环境资源隔离
  - 多租户 GPU 管理

## 🔗 相关链接

- [[MOC - AI基础设施|AI 基础设施 MOC]]
- [[Ray/MOC - Ray|Ray 分布式计算]]
- [HAMi 官方文档](https://project-hami.io)
- [HAMi GitHub](https://github.com/Project-HAMi/HAMi)

## 📊 技术对比

| 方案 | 隔离级别 | 算力限制 | 内存隔离 | 侵入性 |
|------|---------|---------|---------|-------|
| **HAMi** | 软件级 | ✅ 支持 | ✅ 硬隔离 | 低 |
| NVIDIA MPS | 进程级 | ❌ 不支持 | ❌ 软隔离 | 中 |
| NVIDIA MIG | 硬件级 | ✅ 支持 | ✅ 硬隔离 | 需特定 GPU |
| Time-slicing | 时间片 | ❌ 不支持 | ❌ 无隔离 | 低 |
