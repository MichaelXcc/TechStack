---
type: moc
tags: [moc, 总览]
title: 知识体系总览
---

# 知识体系总览

## 🌟 职业转型目标

- **技术架构**：从代码实现到系统设计，培养体系化思维
- **技术管理**：从个人贡献者到团队领导者，提升决策能力
- **AI+云原生**：探索智能技术与云原生的融合创新

## 📚 知识分层结构

### 1. Foundation - 编程语言与底层原理

> 夯实技术基础，理解底层运行机制

- [[Foundation/MOC - 基础层|Foundation MOC]]
  - [[Foundation/Go/MOC - Go语言|Go语言]] - Go 语言核心特性
  - [[Foundation/Python/MOC - Python语言|Python语言]] - 元类、装饰器、生成器、并发编程等
  - [[Foundation/CUDA/MOC - CUDA编程|CUDA编程]] - GPU 编程与性能优化

### 2. Core - 云原生技术栈

> 深入Kubernetes生态，掌握核心技术

- [[Core/MOC - 核心层|Core MOC]]
  - [[Core/Kubernetes/MOC - Kubernetes|Kubernetes]] - 容器编排核心
  - [[Core/Operator/MOC - Operator开发|Operator开发]] - 自定义控制器
  - [[Core/CloudNativeEcosystem/MOC - 云原生生态|云原生生态]] - 服务网格、监控等

### 3. Architecture - 系统设计与架构思维

> 培养架构设计能力，理解分布式系统本质

- [[Architecture/MOC - 架构层|Architecture MOC]]
  - [[Architecture/AI-Infrastructure/MOC - AI基础设施|AI基础设施]] - AI Infra 架构设计
    - [[Architecture/AI-Infrastructure/Ray/MOC - Ray|Ray分布式计算]] - Task、Actor、生态系统
  - [[Architecture/ArchitecturalThinking/MOC - 架构思维|架构思维]] - 架构设计方法论
  - [[Architecture/DistributedSystems/MOC - 分布式系统|分布式系统]] - 分布式理论与实践
  - [[Architecture/SystemDesign/MOC - 系统设计|系统设计]] - 大规模系统设计
  - [[Architecture/HighAvailability/MOC - 高可用设计|高可用设计]] - 容错与可靠性

### 4. Frontier - AI与云原生融合

> 探索前沿技术，把握未来趋势

- [[Frontier/MOC - 前瞻层|Frontier MOC]]
  - [[Frontier/AI-CodingTools/MOC - AI编码工具|AI编码工具]] - AI 辅助编程
  - [[Frontier/RAG/MOC - RAG技术|RAG技术]] - 检索增强生成
  - [[Frontier/AI-K8s-Integration/MOC - AI与K8s结合|AI与K8s结合]] - AI on Kubernetes

### 5. Decision - 技术决策与架构选型

> 系统化的技术决策方法论，记录和追踪重要架构决策

- [[Decision/MOC - 决策层|Decision MOC]]
  - [[Decision/TechSelection/MOC - 技术选型|技术选型]] - 技术评估与选型框架
  - [[Decision/ADR/MOC - ADR|ADR]] - 架构决策记录
  - [[Decision/RiskAndCost/MOC - 风险与成本|风险与成本]] - 风险评估与成本分析

## 🔗 关键主题链接

### 职业转型相关

- [[架构师思维模型]]
- [[技术管理者成长路径]]
- [[从开发者到架构师]]

### 核心技能图谱

```dataview
LIST
FROM ""
WHERE type = "skill"
SORT tags ASC
```

### 最近更新

```dataview
LIST
FROM ""
SORT updated DESC
LIMIT 10
```

## 📊 内容统计

| 层级         | 主要内容                         | 文档数 |
| ------------ | -------------------------------- | ------ |
| Foundation   | Go、Python、CUDA                 | 23+    |
| Core         | Kubernetes、Operator、云原生生态 | 34+    |
| Architecture | AI基础设施、Ray、分布式系统      | 15+    |
| Frontier     | AI编码工具、RAG、AI&K8s          | 10+    |
| Decision     | 技术选型、ADR、风险成本          | 10+    |

## 📝 使用指南

1. **从MOC开始**：每个层级都有对应的MOC文件，作为该领域的导航中心
2. **双向链接**：通过`[[]]`建立知识间的关联，形成知识网络
3. **属性标记**：为笔记添加属性（如`type`、`tags`、`status`），便于Dataview查询
4. **定期更新**：保持笔记的时效性，反映最新的技术进展和个人理解
5. **跨领域思考**：鼓励在不同层级间建立联系，培养综合思维能力

## 🎯 长期目标

- 建立完整的云原生技术知识体系
- 培养架构设计和技术决策能力
- 探索AI与云原生的创新应用
- 成为优秀的技术架构师和管理者
