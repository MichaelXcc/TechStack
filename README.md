# 云原生与架构技术知识体系 (Cloud Native Architecture & Tech Stack)

> 专为从开发转向架构与管理的云原生技术人员设计的结构化知识体系。

> 独学而无友，则孤陋而寡闻。-- 《礼记・学记》
>
> He who receives an idea from me, receives instruction himself without lessening mine; as he who lights his taper at mine, receives light without darkening me.（别人从你这里获得思想，并不会削弱你的思想，就像别人借你的蜡烛点燃自己的，你的蜡烛并不会因此变暗）
> -- 托马斯・杰斐逊（美国第三任总统）
>
> 费曼学习法：以教促学，通过“教别人”倒逼自己梳理知识、发现盲区、强化记忆，记忆留存率可达 90%（远高于被动阅读的 5%）。

## 🌟 职业转型与成长目标

- **技术架构**：从代码实现到系统设计，培养体系化思维。
- **技术决策**：从个人贡献者到团队领导者，提升决策与管理能力。
- **AI与云原生**：探索前沿智能技术与云原生的融合创新，把握未来技术演进趋势。

## 📚 知识分层结构

### 1. Foundation（基础层） - 编程语言与底层原理
> 夯实技术基础，深入理解底层运行机制与高级特性

- [Foundation MOC](Foundation/MOC%20-%20基础层.md)
  - [Go 语言](Foundation/Go/MOC%20-%20Go语言.md) - Go 语言核心特性及底层机制
  - [Python 语言](Foundation/Python/MOC%20-%20Python语言.md) - 元类、装饰器、生成器、并发编程等
  - [CUDA 编程](Foundation/CUDA/MOC%20-%20CUDA编程.md) - GPU 编程、核心机制与性能优化

### 2. Core（核心层） - 云原生技术栈
> 深入 Kubernetes 生态，掌握云原生核心抽象结构

- [Core MOC](Core/MOC%20-%20核心层.md)
  - [Kubernetes](Core/Kubernetes/MOC%20-%20Kubernetes.md) - 容器编排架构与核心组件内幕
  - [Operator 开发](Core/Operator/MOC%20-%20Operator开发.md) - 自定义 CRD 与控制器自动化闭环
  - [云原生生态](Core/CloudNativeEcosystem/MOC%20-%20云原生生态.md) - 服务网格、监控链路、CI/CD 等周边组件

### 3. Architecture（架构层） - 系统设计与架构思维
> 培养系统级架构全局观念，理解大规模分布式环境下的折衷与设计

- [Architecture MOC](Architecture/MOC%20-%20架构层.md)
  - [AI 基础设施](Architecture/AI-Infrastructure/MOC%20-%20AI基础设施.md) - 高性能计算、调度与 AI Infra 架构
    - [Ray 分布式计算](Architecture/AI-Infrastructure/Ray/MOC%20-%20Ray.md) - Task/Actor 模型、集群并发架构及生态库
  - [架构思维](Architecture/ArchitecturalThinking/MOC%20-%20架构思维.md) - 架构师方法论（例如：[从开发者到架构师](Architecture/ArchitecturalThinking/从开发者到架构师.md)）
  - [分布式系统](Architecture/DistributedSystems/MOC%20-%20分布式系统.md) - 分布式一致性协议、调度理论实践
  - [系统设计](Architecture/SystemDesign/MOC%20-%20系统设计.md) - 大规模分布式系统微服务架构设计
  - [高可用设计](Architecture/HighAvailability/MOC%20-%20高可用设计.md) - 容灾、限流、降级与高可靠弹性方案

### 4. Frontier（前瞻层） - AI 与云原生探索
> 保持对前沿技术的敏锐度，拥抱生产力的下一次革命

- [Frontier MOC](Frontier/MOC%20-%20前瞻层.md)
  - [AI 智能体 / 编码工具](Frontier/AI-CodingTools/MOC%20-%20AI编码工具.md) - 前沿 AI 辅助研发架构与编码流
  - [RAG 技术](Frontier/RAG/MOC%20-%20RAG技术.md) - 检索增强生成架构
  - [AI 与 K8s 融合应用](Frontier/AI-K8s-Integration/MOC%20-%20AI与K8s结合.md) - 复杂异构算力环境的容器化与大模型推理负载编排

### 5. Decision（决策层） - 技术视野与管控
> 建立系统化的技术决策方法体系，通过体系化记录留存方案沉淀与团队资产

- [Decision MOC](Decision/MOC%20-%20决策层.md)
  - [技术选型](Decision/TechSelection/MOC%20-%20技术选型.md) - 结合业务发展阶段的组件选型沙盘
  - [架构决策记录 (ADR)](Decision/ADR/MOC%20-%20ADR.md) - 将架构演进而非表层代码沉淀为长期资产
  - [风险与成本分析](Decision/RiskAndCost/MOC%20-%20风险与成本.md) - 云时代 ROI 分析与系统防范评估

## 🔗 核心技能图谱

```text
┌──────────────────────────────────────────────────────────────────┐
│                        云原生与架构技术体系图谱                      │
├──────────────────────────────────────────────────────────────────┤
│  1. 基础编程语言  │  2. 云原生核心      │  3. AI 基础设施 (Infra) │
│  ├─ Go            │  ├─ Kubernetes      │  ├─ 大模型 Infra 架构   │
│  ├─ Python        │  ├─ K8s Operator    │  ├─ Ray 分布式内核及组件 │
│  └─ CUDA / C++    │  └─ Service Mesh    │  └─ RAG / K8s AI 结合   │
├──────────────────────────────────────────────────────────────────┤
│  4. 架构全局视野  │  5. 技术决策梯队                             │
│  ├─ 系统设计范式   │  ├─ 选型基准评估 (Tech Selection)        │
│  ├─ 分布式系统    │  ├─ 架构设计记录驱动 (ADR)               │
│  └─ 极致高可用    │  └─ 成本效能(FinOps)与风险治理            │
└──────────────────────────────────────────────────────────────────┘
```

## 📊 模块内容统计

| 层级 | 主要领域 / 核心技术 | 文档数规模 |
| :--- | :--- | :--- |
| **Foundation** | Go 语言深入、Python 进阶、CUDA 底层 | 23+ |
| **Core** | Kubernetes 内幕、Operator 开发范式、云周边生态 | 34+ |
| **Architecture** | AI 基础设施、Ray、分布式核心理论与高可用保障 | 15+ |
| **Frontier** | RAG 技术内幕、AI 辅助研发工具流、AI on K8s | 10+ |
| **Decision** | 技术选型理论、ADR 标准化、风险防范体系 | 10+ |

## 🚀 最近更新

- **2026-01-19**: 新增 [Ray分布式计算框架](Architecture/AI-Infrastructure/Ray/MOC%20-%20Ray.md) 核心设计与学习资料
- **2026-01-15**: 梳理 [Python高级特性](Foundation/Python/MOC%20-%20Python语言.md) 体系
- **2026-01-13**: 跟进记录 [CUDA编程](Foundation/CUDA/MOC%20-%20CUDA编程.md) 体系学习笔记
- **2026-01-13**: 建立 [AI基础设施架构设计](Architecture/AI-Infrastructure/MOC%20-%20AI基础设施.md) 技术大纲

## 📝 知识库使用指南

1. **从 MOC 开始**：各个知识领域均配有相应的 MOC（Map of Content，目录/索引结构）文件，以此作为中心节点向相关主题辐射发散。
2. **构建链接闭环**：利用双向链接建立起不同知识点间的映射，形成一张灵活、多维的技术网络。
3. **长期更新追踪**：持续迭代对知识点的认知，并保持最新的生产环境实践的整理与梳理。
4. **跨界横向思考**：从单纯的代码实现向架构体系融合迁移，如尝试用 K8s 的控制器模式解决运维的终态声明难题。
5. **项目应用检验**：技术文档的最终目的是服务产品，应积极在社区及实际工作中落地并持续复盘。

---

**👋 探索起点**：建议从夯实基石底层开始 [👉 Foundation](Foundation/MOC%20-%20基础层.md)，或直接深入当前的热点前沿 [👉 Architecture](Architecture/MOC%20-%20架构层.md) 进行阅读。
