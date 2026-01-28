---
type: concept
tags: [kubernetes, scheduler, 控制平面]
title: Scheduler概念
status: 已掌握
priority: 高
related: ["API Server", "Node", "Pod"]
---

# Scheduler概念

## 📋 定义

Scheduler是Kubernetes控制平面的核心组件，负责将未调度的Pod分配到合适的节点上运行。

## 🎯 核心特性

### 1. 智能调度
- **资源感知**：考虑节点的CPU、内存等资源
- **约束满足**：满足Pod的调度约束
- **策略优化**：根据调度策略选择最优节点

### 2. 可扩展性
- **自定义调度器**：支持自定义调度逻辑
- **调度框架**：提供插件化的调度框架
- **多调度器**：支持运行多个调度器

### 3. 高可用性
- **Leader选举**：多个Scheduler实例通过Leader选举协调
- **故障恢复**：Scheduler故障不影响已调度的Pod
- **状态一致性**：确保调度状态的一致性

## 📝 调度流程

### 调度流程图
```
1. 监听未调度Pod
   ↓
2. 过滤节点
   ↓
3. 打分节点
   ↓
4. 选择最优节点
   ↓
5. 绑定Pod到节点
   ↓
6. 更新API Server
```

### 详细流程
```go
// 伪代码
func schedule(pod *v1.Pod) {
  // 1. 获取所有节点
  nodes := listNodes()
  
  // 2. 过滤节点
  feasibleNodes := filterNodes(pod, nodes)
  
  // 3. 打分节点
  scoredNodes := scoreNodes(pod, feasibleNodes)
  
  // 4. 选择最优节点
  bestNode := selectBestNode(scoredNodes)
  
  // 5. 绑定Pod
  bindPod(pod, bestNode)
}
```

## 🔧 调度策略

### 默认调度策略
```yaml
# Kube-scheduler配置
apiVersion: kubescheduler.config.k8s.io/v1beta2
kind: KubeSchedulerConfiguration
profiles:
- schedulerName: default-scheduler
  plugins:
    queueSort:
      enabled:
      - name: PrioritySort
    preFilter:
      enabled:
      - name: NodeResourcesFit
      - name: NodePorts
      - name: PodToleratesNodeTaints
    filter:
      enabled:
      - name: NodeUnschedulable
      - name: NodeName
      - name: TaintToleration
    postFilter:
      enabled:
      - name: DefaultPreemption
    preScore:
      enabled:
      - name: InterPodAffinity
      - name: NodeResourcesFit
    score:
      enabled:
      - name: NodeResourcesBalancedAllocation
      - name: InterPodAffinity
      - name: NodePreferAvoidPods
    reserve:
      enabled:
      - name: VolumeBinding
    permit:
      enabled:
      - name: VolumeBinding
```

### 调度插件
#### 过滤插件
- **NodeUnschedulable**：过滤不可调度节点
- **NodeName**：匹配指定节点名称
- **NodePorts**：检查端口冲突
- **TaintToleration**：检查污点和容忍度
- **NodeAffinity**：检查节点亲和性
- **PodAffinity**：检查Pod亲和性

#### 打分插件
- **NodeResourcesBalancedAllocation**：资源均衡分配
- **InterPodAffinity**：Pod亲和性打分
- **NodePreferAvoidPods**：避免特定节点
- **ImageLocality**：镜像本地性打分

## 🔧 调度约束

### 节点选择器
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  nodeSelector:
    disktype: ssd
    zone: us-west-1
```

### 节点亲和性
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/arch
            operator: In
            values:
            - amd64
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        preference:
          matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
```

### Pod亲和性
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - cache
        topologyKey: kubernetes.io/hostname
```

### Pod反亲和性
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - web
        topologyKey: kubernetes.io/hostname
```

### 污点和容忍度
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  tolerations:
  - key: "key1"
    operator: "Equal"
    value: "value1"
    effect: "NoSchedule"
  - key: "key1"
    operator: "Exists"
    effect: "NoExecute"
    tolerationSeconds: 3600
```

## 🔧 自定义调度器

### 自定义调度器配置
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  schedulerName: my-custom-scheduler
```

### 自定义调度器实现
```go
package main

import (
  "k8s.io/kubernetes/pkg/scheduler/framework"
)

type MyScheduler struct {
  frameworkHandle framework.Handle
}

func (s *MyScheduler) Name() string {
  return "my-scheduler"
}

func (s *MyScheduler) Schedule(
  ctx context.Context,
  state *framework.CycleState,
  pod *v1.Pod,
) (result framework.ScheduleResult, err error) {
  // 自定义调度逻辑
  nodes, err := s.frameworkHandle.SnapshotSharedLister().NodeInfos().List()
  if err != nil {
    return framework.ScheduleResult{}, err
  }
  
  for _, node := range nodes {
    if s.isNodeSuitable(node, pod) {
      return framework.ScheduleResult{
        SuggestedHost: node.Node().Name,
      }, nil
    }
  }
  
  return framework.ScheduleResult{}, fmt.Errorf("no suitable node found")
}

func (s *MyScheduler) isNodeSuitable(node *framework.NodeInfo, pod *v1.Pod) bool {
  // 自定义节点选择逻辑
  return true
}
```

## 🔧 调度优化

### 优先级和抢占
```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000
globalDefault: false
description: "High priority class"

---
apiVersion: v1
kind: Pod
metadata:
  name: high-priority-pod
spec:
  priorityClassName: high-priority
  containers:
  - name: my-container
    image: nginx
```

### Pod干扰预算
```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: my-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: my-app
```

## 🔗 相关概念

### 控制平面组件
- [[API Server概念]]：提供API接口
- [[Controller Manager概念]]：管理控制器
- [[etcd概念]]：存储集群状态

### 调度相关
- [[Node概念]]：调度目标节点
- [[Pod概念]]：调度对象
- [[Taint和Toleration]]：污点和容忍度

## 🚀 最佳实践

### 1. 合理设置资源请求
```yaml
spec:
  containers:
  - name: my-container
    resources:
      requests:
        cpu: "100m"
        memory: "128Mi"
      limits:
        cpu: "200m"
        memory: "256Mi"
```

### 2. 使用节点亲和性
```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: zone
            operator: In
            values:
            - us-west-1
```

### 3. 配置Pod亲和性
```yaml
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            app: cache
        topologyKey: kubernetes.io/hostname
```

### 4. 使用优先级类
```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: production
value: 1000
globalDefault: false
```

### 5. 监控调度性能
- 监控调度延迟
- 监控调度失败率
- 监控节点资源利用率

### 6. 优化调度策略
- 根据业务需求调整调度策略
- 使用自定义调度器满足特殊需求
- 定期审查调度配置

## 💡 架构思考

Scheduler是Kubernetes的智能调度引擎：
- **资源优化**：优化集群资源利用率
- **约束满足**：满足各种调度约束
- **策略灵活**：支持自定义调度策略

从架构师视角：
- Scheduler设计体现了智能调度的最佳实践
- 理解Scheduler的工作原理有助于优化集群性能
- 合理的调度策略是构建高效集群的关键

## 🔍 深入学习

### 相关文档
- [Kubernetes官方Scheduler文档](https://kubernetes.io/docs/concepts/scheduling-eviction/kube-scheduler/)
- [[Kubernetes/控制平面/API Server]]：API Server
- [[Kubernetes/控制平面/Controller Manager]]：控制器管理器

### 实践案例
- [[生产环境K8s部署/Scheduler配置]]
- [[性能优化/调度优化]]
- [[故障排查与恢复/Scheduler问题]]
