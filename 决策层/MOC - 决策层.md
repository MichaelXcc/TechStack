---
type: moc
tags: [moc, 决策层, 技术决策, ADR]
title: 决策层MOC
created: 2026-01-28
updated: 2026-01-28
author: 云原生技术架构师
---

# 决策层 - 技术决策与架构选型

## 🎯 目标

- 掌握技术决策的方法论和框架
- 建立系统化的技术选型流程
- 学习使用 ADR（架构决策记录）记录重要决策
- 平衡技术理想与业务现实

## 📁 目录结构

### 技术选型框架

> 系统化的技术选型方法论

- [[技术选型/MOC - 技术选型|技术选型 MOC]]
  - 选型评估维度
  - POC 验证方法
  - 技术雷达应用
  - 开源 vs 商业评估

### 架构决策记录 (ADR)

> 使用 ADR 记录和追踪重要的架构决策

- [[ADR/MOC - ADR|ADR MOC]]
  - ADR 模板与规范
  - 决策状态管理
  - 历史决策追溯
  - 决策影响分析

### 风险与成本分析

> 技术决策的风险评估与成本效益分析

- [[风险与成本/MOC - 风险与成本|风险与成本 MOC]]
  - 技术风险评估
  - TCO 总体拥有成本
  - ROI 投资回报分析
  - 技术债务评估

## 🔗 决策案例

### 基础设施决策

- [[ADR/ADR-001-容器运行时选择]]
- [[ADR/ADR-002-服务网格选型]]
- [[ADR/ADR-003-存储方案选择]]

### AI/ML 基础设施决策

- [[ADR/ADR-004-GPU调度方案选型]]
- [[ADR/ADR-005-分布式训练框架选择]]
- [[ADR/ADR-006-模型服务推理引擎选型]]

## 📊 决策模型

### 常用决策框架

- **DACI**: Driver, Approver, Contributor, Informed
- **RACI**: Responsible, Accountable, Consulted, Informed
- **权衡滑块**: 在相互冲突的目标间做权衡
- **决策矩阵**: 多维度评分比较

## 🛠️ 实用工具

### 决策辅助

- 技术雷达 (Technology Radar)
- 架构健身函数 (Architecture Fitness Functions)
- 影响图 (Impact Map)
- 权重评分卡 (Weighted Scorecard)

### 文档模板

- [[模板/ADR模板]]
- [[模板/技术选型报告模板]]
- [[模板/POC评估报告模板]]
- [[模板/风险评估矩阵模板]]

## 📖 学习资源

### 推荐阅读

- _Design It!_ - Michael Keeling
- _Software Architecture: The Hard Parts_ - Neal Ford
- _Technology Strategy Patterns_ - Eben Hewitt

### 相关链接

- [ADR GitHub](https://adr.github.io/)
- [ThoughtWorks Technology Radar](https://www.thoughtworks.com/radar)
- [CNCF Landscape](https://landscape.cncf.io/)
