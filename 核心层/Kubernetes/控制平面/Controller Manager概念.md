---
type: concept
tags: [kubernetes, controller-manager, 控制平面]
title: Controller Manager概念
author: 云原生技术架构师
status: 已掌握
priority: 高
related: ["API Server", "Scheduler", "Controller"]
---

# Controller Manager概念

## 📋 定义

Controller Manager是Kubernetes控制平面的核心组件，运行多个控制器，负责维护集群状态。

## 🎯 核心特性

### 1. 控制器集合
- **节点控制器**：管理节点生命周期
- **副本控制器**：管理Pod副本数
- **端点控制器**：管理Service端点
- **服务账户控制器**：管理ServiceAccount

### 2. 状态协调
- **声明式API**：监控期望状态和实际状态
- **控制循环**：持续协调状态差异
- **自动修复**：自动恢复故障资源

### 3. 高可用性
- **Leader选举**：多个实例通过Leader选举协调
- **故障恢复**：实例故障不影响控制器运行
- **状态一致性**：确保控制器状态的一致性

## 📝 内置控制器

### 节点控制器
- **职责**：管理节点生命周期
- **功能**：
  - 监控节点健康状态
  - 标记不可用节点
  - 驱逐节点上的Pod
  - 处理节点删除

### 副本控制器
- **职责**：管理Pod副本数
- **功能**：
  - 确保Pod副本数符合期望
  - 创建或删除Pod
  - 监控Pod状态

### 端点控制器
- **职责**：管理Service端点
- **功能**：
  - 监听Pod变化
  - 更新Service的Endpoint
  - 实现服务发现

### 服务账户控制器
- **职责**：管理ServiceAccount
- **功能**：
  - 为Namespace创建默认ServiceAccount
  - 为ServiceAccount创建Token
  - 管理ServiceAccount权限

### 命名空间控制器
- **职责**：管理Namespace生命周期
- **功能**：
  - 监听Namespace删除
  - 删除Namespace下的所有资源
  - 清理相关资源

### 持久卷绑定控制器
- **职责**：管理PV和PVC绑定
- **功能**：
  - 监听PVC创建
  - 绑定合适的PV
  - 管理PV生命周期

### 令牌控制器
- **职责**：管理ServiceAccount Token
- **功能**：
  - 为ServiceAccount生成Token
  - 定期轮换Token
  - 删除过期Token

## 🔧 控制器工作原理

### 控制循环
```go
// 伪代码
func controllerLoop() {
  for {
    // 1. 获取期望状态
    desiredState := getDesiredState()
    
    // 2. 获取实际状态
    currentState := getCurrentState()
    
    // 3. 比较状态
    if desiredState != currentState {
      // 4. 协调状态
      reconcile(desiredState, currentState)
    }
    
    // 5. 等待下次循环
    time.Sleep(reconciliationInterval)
  }
}
```

### 事件驱动
```go
// 伪代码
func watchResources() {
  // 1. 创建Informer
  informer := informers.NewSharedInformerFactory(clientset, time.Second*30)
  
  // 2. 添加事件处理器
  podInformer := informer.Core().V1().Pods().Informer()
  podInformer.AddEventHandler(cache.ResourceEventHandlerFuncs{
    AddFunc: func(obj interface{}) {
      pod := obj.(*v1.Pod)
      handlePodAdd(pod)
    },
    UpdateFunc: func(oldObj, newObj interface{}) {
      oldPod := oldObj.(*v1.Pod)
      newPod := newObj.(*v1.Pod)
      handlePodUpdate(oldPod, newPod)
    },
    DeleteFunc: func(obj interface{}) {
      pod := obj.(*v1.Pod)
      handlePodDelete(pod)
    },
  })
  
  // 3. 启动Informer
  informerFactory.Start(stopCh)
}
```

## 🔧 控制器配置

### 启用控制器
```bash
# kube-controller-manager配置
--controllers=*,bootstrapsigner,tokencleaner
```

### 禁用控制器
```bash
# kube-controller-manager配置
--controllers=*,bootstrapsigner,tokencleaner
--controllers=-node-lifecycle
```

### 控制器并发度
```bash
# kube-controller-manager配置
--concurrent-deployment-syncs=5
--concurrent-replicaset-syncs=5
--concurrent-endpoint-syncs=5
```

## 🔧 Leader选举

### Leader选举配置
```bash
# kube-controller-manager配置
--leader-elect=true
--leader-elect-lease-duration=15s
--leader-elect-renew-deadline=10s
--leader-elect-retry-period=2s
```

### Leader选举原理
```go
// 伪代码
func electLeader() {
  for {
    // 1. 尝试获取租约
    lease, err := tryAcquireLease()
    if err == nil {
      // 2. 成为Leader
      becomeLeader()
      
      // 3. 定期续约
      for {
        err := renewLease(lease)
        if err != nil {
          break
        }
        time.Sleep(renewInterval)
      }
    } else {
      // 4. 等待下次选举
      time.Sleep(electInterval)
    }
  }
}
```

## 🔧 自定义控制器

### 使用client-go
```go
package main

import (
  "k8s.io/client-go/informers"
  "k8s.io/client-go/kubernetes"
  "k8s.io/client-go/tools/cache"
)

func main() {
  // 1. 创建客户端
  config, _ := rest.InClusterConfig()
  clientset, _ := kubernetes.NewForConfig(config)
  
  // 2. 创建Informer工厂
  informerFactory := informers.NewSharedInformerFactory(clientset, time.Second*30)
  
  // 3. 获取Informer
  podInformer := informerFactory.Core().V1().Pods().Informer()
  
  // 4. 添加事件处理器
  podInformer.AddEventHandler(cache.ResourceEventHandlerFuncs{
    AddFunc: func(obj interface{}) {
      pod := obj.(*v1.Pod)
      handlePodAdd(pod)
    },
    UpdateFunc: func(oldObj, newObj interface{}) {
      oldPod := oldObj.(*v1.Pod)
      newPod := newObj.(*v1.Pod)
      handlePodUpdate(oldPod, newPod)
    },
    DeleteFunc: func(obj interface{}) {
      pod := obj.(*v1.Pod)
      handlePodDelete(pod)
    },
  })
  
  // 5. 启动Informer
  informerFactory.Start(stopCh)
  
  // 6. 等待信号
  <-stopCh
}
```

### 使用Operator SDK
```bash
# 创建Operator项目
operator-sdk init --domain my.domain --repo my.domain/my-operator

# 创建API
operator-sdk create api --group myapp --version v1 --kind MyApp

# 创建控制器
operator-sdk create controller --kind MyApp
```

## 🔗 相关概念

### 控制平面组件
- [[API Server概念]]：提供API接口
- [[Scheduler概念]]：调度Pod到节点
- [[etcd概念]]：存储集群状态

### 控制器相关
- [[Controller概念]]：控制器概念
- [[Deployment概念]]：Deployment控制器
- [[StatefulSet概念]]：StatefulSet控制器

## 🚀 最佳实践

### 1. 监控控制器性能
```yaml
# Prometheus监控指标
kube_controller_manager_pod_deletions_total
kube_controller_manager_pod_evictions_total
kube_controller_manager_reconcile_loop_duration_seconds
```

### 2. 配置合理的并发度
```bash
# kube-controller-manager配置
--concurrent-deployment-syncs=5
--concurrent-replicaset-syncs=5
--concurrent-endpoint-syncs=5
```

### 3. 启用Leader选举
```bash
# kube-controller-manager配置
--leader-elect=true
--leader-elect-lease-duration=15s
--leader-elect-renew-deadline=10s
```

### 4. 配置资源限制
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kube-controller-manager
  namespace: kube-system
spec:
  containers:
  - name: kube-controller-manager
    resources:
      requests:
        cpu: 200m
        memory: 512Mi
      limits:
        cpu: 1000m
        memory: 2Gi
```

### 5. 使用工作队列
```go
// 使用工作队列处理事件
queue := workqueue.NewRateLimitingQueue(workqueue.DefaultControllerRateLimiter())

// 处理事件
for {
  obj, shutdown := queue.Get()
  if shutdown {
    break
  }
  
  err := processObj(obj)
  if err == nil {
    queue.Forget(obj)
  } else if queue.NumRequeues(obj) < maxRetries {
    queue.AddRateLimited(obj)
  } else {
    queue.Forget(obj)
  }
}
```

### 6. 实现优雅关闭
```go
// 处理关闭信号
stopCh := make(chan struct{})
signalCh := make(chan os.Signal, 1)
signal.Notify(signalCh, syscall.SIGINT, syscall.SIGTERM)

go func() {
  <-signalCh
  close(stopCh)
}()

// 等待关闭
<-stopCh
```

## 💡 架构思考

Controller Manager是Kubernetes的控制中心：
- **自动化管理**：自动维护集群状态
- **声明式API**：通过声明式API管理资源
- **可扩展性**：支持自定义控制器

从架构师视角：
- Controller Manager设计体现了控制循环的最佳实践
- 理解Controller Manager的工作原理有助于设计高可用集群
- 自定义控制器和Operator是实现复杂应用管理的关键

## 🔍 深入学习

### 相关文档
- [Kubernetes官方Controller Manager文档](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/)
- [[Kubernetes/控制平面/API Server]]：API Server
- [[Kubernetes/控制平面/Scheduler]]：调度器

### 实践案例
- [[生产环境K8s部署/Controller Manager配置]]
- [[Operator开发/最佳实践]]
- [[故障排查与恢复/Controller Manager问题]]
