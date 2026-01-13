---
type: concept
tags: [kubernetes, controller, 核心概念]
title: Controller概念
author: AI Infra / LLM Ops 基础设施
status: 已掌握
priority: 高
related: ["Pod", "Deployment", "StatefulSet"]
---

# Controller概念

## 📋 定义

Controller是Kubernetes中用于管理资源状态的控制器，它通过持续监控集群状态，确保实际状态与期望状态一致。

## 🎯 核心特性

### 1. 控制循环
- **声明式API**：用户声明期望状态
- **状态监控**：持续监控实际状态
- **状态协调**：调整实际状态使其与期望状态一致

### 2. 自愈能力
- **自动恢复**：Pod故障时自动创建新Pod
- **弹性伸缩**：根据负载自动调整副本数
- **故障隔离**：隔离故障节点，保证服务可用性

### 3. 扩展性
- **自定义控制器**：支持自定义CRD和Controller
- **Operator模式**：封装领域知识，实现复杂应用管理
- **控制器组合**：多个控制器协同工作

## 📝 控制器类型

### Deployment Controller
- **管理对象**：Deployment、ReplicaSet、Pod
- **核心功能**：
  - 管理ReplicaSet副本数
  - 执行滚动更新
  - 支持回滚
- **工作流程**：
  1. 监听Deployment变化
  2. 创建或更新ReplicaSet
  3. 确保Pod副本数符合期望

### StatefulSet Controller
- **管理对象**：StatefulSet、Pod、PVC
- **核心功能**：
  - 管理有序的Pod创建和删除
  - 管理稳定的网络标识
  - 管理持久化存储
- **工作流程**：
  1. 监听StatefulSet变化
  2. 按顺序创建Pod
  3. 为每个Pod创建PVC

### DaemonSet Controller
- **管理对象**：DaemonSet、Pod
- **核心功能**：
  - 在每个节点运行Pod副本
  - 自动处理节点加入和离开
  - 支持节点选择器
- **工作流程**：
  1. 监听DaemonSet变化
  2. 监听节点变化
  3. 确保每个节点运行Pod

### Job Controller
- **管理对象**：Job、Pod
- **核心功能**：
  - 管理一次性任务
  - 确保任务成功完成
  - 支持并行任务
- **工作流程**：
  1. 监听Job变化
  2. 创建Pod执行任务
  3. 监控任务完成状态

### CronJob Controller
- **管理对象**：CronJob、Job、Pod
- **核心功能**：
  - 管理定时任务
  - 按Cron表达式调度任务
  - 管理任务历史
- **工作流程**：
  1. 监听CronJob变化
  2. 按Cron表达式创建Job
  3. 管理Job历史记录

### Endpoint Controller
- **管理对象**：Service、Endpoint
- **核心功能**：
  - 监听Pod变化
  - 更新Service的Endpoint
  - 实现服务发现
- **工作流程**：
  1. 监听Service和Pod变化
  2. 匹配Pod标签
  3. 更新Endpoint列表

### Namespace Controller
- **管理对象**：Namespace
- **核心功能**：
  - 监听Namespace删除
  - 删除Namespace下的所有资源
  - 清理相关资源
- **工作流程**：
  1. 监听Namespace删除事件
  2. 删除Namespace下的所有对象
  3. 清理相关资源

## 🔧 控制器工作原理

### 控制循环
```go
for {
  desiredState := getDesiredState()  // 获取期望状态
  currentState := getCurrentState()   // 获取当前状态
  if desiredState != currentState {
    reconcile(currentState, desiredState)  // 协调状态
  }
  time.Sleep(reconciliationInterval)       // 等待下次循环
}
```

### 事件驱动
```go
// 监听资源变化
watcher := clientset.CoreV1().Pods("").Watch(metav1.ListOptions{})

for event := range watcher.ResultChan() {
  pod := event.Object.(*v1.Pod)
  switch event.Type {
  case watch.Added:
    handlePodAdd(pod)
  case watch.Modified:
    handlePodUpdate(pod)
  case watch.Deleted:
    handlePodDelete(pod)
  }
}
```

### 状态协调
```go
func reconcile(desired, current int32) {
  if current < desired {
    // 创建新Pod
    createPod(desired - current)
  } else if current > desired {
    // 删除多余Pod
    deletePod(current - desired)
  }
}
```

## 📊 控制器模式

### 水平触发
- **特点**：控制器持续监控状态
- **优势**：确保最终一致性
- **场景**：Deployment、ReplicaSet

### 边缘触发
- **特点**：只在状态变化时触发
- **优势**：减少不必要的计算
- **场景**：部分自定义控制器

### 工作队列
- **特点**：使用队列处理事件
- **优势**：避免事件丢失，支持限流
- **场景**：生产环境控制器

## 🔗 相关概念

### 控制平面组件
- **API Server**：提供REST API接口
- **Controller Manager**：运行多个控制器
- **Scheduler**：调度Pod到节点

### 工作负载资源
- [[Deployment概念]]：Deployment控制器
- [[StatefulSet概念]]：StatefulSet控制器
- [[DaemonSet概念]]：DaemonSet控制器
- [[Pod概念]]：控制器管理的Pod

## 🚀 最佳实践

### 1. 自定义控制器
```go
// 使用client-go构建自定义控制器
import (
  "k8s.io/client-go/informers"
  "k8s.io/client-go/kubernetes"
  "k8s.io/client-go/tools/cache"
)

func main() {
  config, _ := rest.InClusterConfig()
  clientset, _ := kubernetes.NewForConfig(config)
  
  informerFactory := informers.NewSharedInformerFactory(clientset, time.Second*30)
  podInformer := informerFactory.Core().V1().Pods().Informer()
  
  podInformer.AddEventHandler(cache.ResourceEventHandlerFuncs{
    AddFunc: func(obj interface{}) {
      pod := obj.(*v1.Pod)
      handlePodAdd(pod)
    },
  })
  
  informerFactory.Start(stopCh)
}
```

### 2. Operator模式
- 封装领域知识
- 实现复杂应用管理
- 提供声明式API

### 3. 控制器优化
- 使用SharedInformer减少API调用
- 使用WorkQueue处理事件
- 实现合理的重试机制

### 4. 监控和告警
- 监控控制器性能指标
- 设置合理的告警规则
- 记录控制器事件日志

### 5. 测试和验证
- 编写单元测试
- 编写集成测试
- 进行混沌测试

## 💡 架构思考

Controller是Kubernetes控制平面的核心：
- **声明式管理**：用户声明期望状态，控制器实现状态协调
- **自愈能力**：控制器持续监控，自动恢复故障
- **扩展性**：支持自定义控制器和Operator

从架构师视角：
- Controller体现了Kubernetes的自动化管理能力
- 理解Controller的工作原理有助于设计高可用系统
- 自定义控制器和Operator是实现复杂应用管理的关键

## 🔍 深入学习

### 相关文档
- [Kubernetes官方Controller文档](https://kubernetes.io/docs/concepts/architecture/controller/)
- [[Controller Manager概念]]：控制器管理器
- [[Kubernetes/扩展/Operator开发]]：Operator开发指南

### 实践案例
- [[生产环境K8s部署/自定义控制器]]
- [[Operator开发/最佳实践]]
- [[故障排查与恢复/Controller问题]]
