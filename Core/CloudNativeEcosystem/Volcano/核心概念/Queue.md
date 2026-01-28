---
type: concept
tags: [volcano, queue, scheduling]
title: Queue队列
status: 学习中
priority: 高
related: ["Volcano Job", "资源配额", "调度策略"]
---


# Queue队列

## 📋 定义

Queue是Volcano中的核心资源之一，用于管理和调度多个Job。Job提交到Queue中，按照配置的调度策略进行调度和资源分配。Queue支持层级结构、资源配额和优先级等特性，是Volcano实现公平调度和资源管理的基础。

## 🎯 核心价值

### 1. 资源管理
- **资源配额**：为不同队列设置资源上限，确保资源公平分配
- **资源预留**：支持为关键队列预留资源
- **资源隔离**：实现不同团队或业务之间的资源隔离

### 2. 调度策略
- **FIFO**：先进先出调度
- **Fair Sharing**：公平共享调度
- **Priority-based**：基于优先级的调度
- **DRF (Dominant Resource Fairness)**：主导资源公平算法

### 3. 层级管理
- 支持创建层级队列
- 父队列资源可以分配给子队列
- 支持复杂的组织架构

## 🔍 核心特性

### 1. 队列状态

| 状态 | 说明 |
|------|------|
| **Open** | 队列处于活跃状态，接受新Job |
| **Closed** | 队列处于关闭状态，不接受新Job，但已提交的Job可以继续执行 |
| **Paused** | 队列处于暂停状态，不接受新Job，已提交的Job暂停执行 |

### 2. 队列属性

| 属性 | 说明 |
|------|------|
| **name** | 队列名称，唯一标识符 |
| **state** | 队列状态（Open/Closed/Paused） |
| **weight** | 队列权重，用于Fair Sharing调度 |
| **capacity** | 队列资源配额，包括CPU、内存、GPU等 |
| **namespace** | 队列所属的命名空间 |
| **hierarchy** | 队列层级关系 |

## 🚀 使用方法

### 1. 创建Queue

**YAML配置示例：**

```yaml
apiVersion: scheduling.volcano.sh/v1beta1
kind: Queue
metadata:
  name: ai-training
spec:
  weight: 100  # 队列权重，用于公平调度
  capacity:
    cpu: "100"
    memory: "400Gi"
    nvidia.com/gpu: "20"  # GPU资源配额
  state: Open  # 队列状态
```

**创建命令：**

```bash
kubectl apply -f ai-training-queue.yaml
```

### 2. 创建层级Queue

**YAML配置示例：**

```yaml
# 父队列
apiVersion: scheduling.volcano.sh/v1beta1
kind: Queue
metadata:
  name: team-a
spec:
  weight: 200
  capacity:
    cpu: "200"
    memory: "800Gi"
  state: Open

# 子队列1
apiVersion: scheduling.volcano.sh/v1beta1
kind: Queue
metadata:
  name: team-a-ai
spec:
  weight: 150
  capacity:
    cpu: "150"
    memory: "600Gi"
  state: Open
  hierarchy: team-a  # 父队列名称

# 子队列2
apiVersion: scheduling.volcano.sh/v1beta1
kind: Queue
metadata:
  name: team-a-bigdata
spec:
  weight: 50
  capacity:
    cpu: "50"
    memory: "200Gi"
  state: Open
  hierarchy: team-a  # 父队列名称
```

### 3. 将Job提交到Queue

**YAML配置示例：**

```yaml
apiVersion: batch.volcano.sh/v1alpha1
kind: Job
metadata:
  name: pytorch-training
spec:
  minAvailable: 4
  schedulerName: volcano
  queue: ai-training  # 指定队列名称
  tasks:
  - name: worker
    replicas: 4
    template:
      spec:
        containers:
        - name: pytorch
          image: pytorch/pytorch:latest
          resources:
            requests:
              cpu: "8"
              memory: "32Gi"
              nvidia.com/gpu: 1
        restartPolicy: OnFailure
```

## ⚙️ 配置优化

### 1. 资源配额配置

**最佳实践：**
- 为队列设置合理的资源配额，避免资源浪费
- 考虑峰值需求和平均需求
- 定期调整资源配额，根据实际使用情况优化

**示例配置：**

```yaml
apiVersion: scheduling.volcano.sh/v1beta1
kind: Queue
metadata:
  name: production
spec:
  weight: 300
  capacity:
    cpu: "300"
    memory: "1200Gi"
    nvidia.com/gpu: "60"
  state: Open
```

### 2. 优先级配置

**示例：创建PriorityClass**

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
description: "High priority queue for critical jobs"
```

**在Queue中使用：**

```yaml
apiVersion: batch.volcano.sh/v1alpha1
kind: Job
metadata:
  name: critical-job
spec:
  minAvailable: 2
  schedulerName: volcano
  queue: production
  priorityClassName: high-priority  # 使用高优先级
  tasks:
  - name: worker
    replicas: 2
    template:
      spec:
        containers:
        - name: app
          image: critical-app:latest
          resources:
            requests:
              cpu: "4"
              memory: "16Gi"
        restartPolicy: OnFailure
```

## 📊 监控和管理

### 1. 查看Queue状态

```bash
# 查看所有Queue
kubectl get queues.scheduling.volcano.sh

# 查看特定Queue的详细信息
kubectl describe queues.scheduling.volcano.sh ai-training
```

### 2. 更新Queue

```bash
# 编辑Queue配置
kubectl edit queues.scheduling.volcano.sh ai-training

# 更新Queue状态为Closed
kubectl patch queue ai-training -p '{"spec":{"state":"Closed"}}' --type=merge
```

### 3. 删除Queue

```bash
kubectl delete queues.scheduling.volcano.sh ai-training
```

## 💡 最佳实践

### 1. 队列设计原则

- **清晰的命名规则**：使用有意义的队列名称，如`ai-training`、`bigdata-processing`
- **合理的权重分配**：根据团队规模和业务重要性分配权重
- **适当的资源配额**：避免过度分配或分配不足
- **层级化管理**：对于大型团队，使用层级队列管理

### 2. 资源管理最佳实践

- **预留缓冲资源**：为队列设置80%的实际需求，预留20%的缓冲
- **定期审计**：每月检查队列资源使用情况，调整配额
- **设置默认队列**：为每个命名空间设置默认队列

### 3. 调度策略最佳实践

- **结合优先级**：为关键任务设置高优先级
- **合理使用Fair Sharing**：确保多个团队公平共享资源
- **避免资源饥饿**：为小团队设置最小资源保障

## 🔗 相关链接

### 核心概念
- [[Volcano核心概念]]：Volcano核心概念总览
- [[Gang Scheduling]]：成组调度
- [[Job]]：Volcano Job

### 架构设计
- [[../架构设计/Volcano架构设计|Volcano架构设计]]：Volcano架构总览

### 实践案例
- [[../实践案例/AI训练调度|AI训练调度]]：AI训练调度案例
- [[../实践案例/多租户调度|多租户调度]]：多租户调度案例

## 💬 思考问题

1. 如何设计适合你的组织的Queue层级结构？
2. 如何平衡不同团队之间的资源需求？
3. 如何监控和优化Queue的资源使用？
4. 如何处理Queue资源不足的情况？
