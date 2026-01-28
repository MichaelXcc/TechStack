---
type: moc
tags: [moc, 架构层, AIInfra, GPU, 性能优化]
title: GPU性能优化 MOC
---

# GPU 性能优化

> 本目录包含 GPU 性能测试、监控、调优相关的文档，帮助最大化 AI 算力利用率。

## 📁 目录内容

### 性能分析
- [[GPU性能-GEMM测试分析|GPU性能-GEMM测试分析]]
  - TFLOPS 测试中的功耗与频率影响
  - 显存压缩与带宽优化
  - 结构化稀疏技术

### 性能监控
- [[GPU性能指标与监控|GPU性能指标与监控]]
  - 核心性能指标（MFU/HFU/TFLOPS）
  - 监控工具（DCGM/nvidia-smi）
  - Prometheus + Grafana 集成

### 调优实践
- [[GPU性能调优最佳实践|GPU性能调优最佳实践]]
  - 算子融合与 Kernel 优化
  - 通信重叠技术
  - 显存优化策略

## 🎯 核心指标

| 指标 | 含义 | 目标值 |
|------|------|--------|
| **MFU** | Model FLOPS Utilization | >50% |
| **HFU** | Hardware FLOPS Utilization | >60% |
| **GPU Util** | GPU 核心利用率 | >90% |
| **Memory BW** | 显存带宽利用率 | >70% |

## 🔗 相关链接

- [[../MOC - AI基础设施|返回 AI 基础设施 MOC]]
- [[../../基础层/CUDA/MOC - CUDA编程|CUDA 编程]]
- [[../AI-Infra核心架构/02_高性能计算集群设计|高性能计算集群设计]]
