---
type: concept
tags: [kubernetes, node, 核心概念]
title: Node概念
author: AI Infra / LLM Ops 基础设施
status: 已掌握
priority: 高
related: ["Pod", "Kubelet", "Scheduler"]
---

# Node概念

## 📋 定义

Node是Kubernetes集群中的工作节点，它可以是物理机或虚拟机，负责运行Pod并提供容器运行时环境。

## 🎯 核心特性

### 1. 资源管理
- **CPU**：节点可用的CPU资源
- **内存**：节点可用的内存资源
- **存储**：节点可用的存储资源
- **网络**：节点的网络配置

### 2. 状态管理
- **Ready**：节点健康，可以调度Pod
- **NotReady**：节点不健康，不可调度Pod
- **Unknown**：节点状态未知

### 3. 标签和污点
- **标签**：用于节点选择和分组
- **污点**：用于控制Pod调度
- **容忍度**：Pod容忍节点的污点

## 📝 Node对象

### Node对象示例
```yaml
apiVersion: v1
kind: Node
metadata:
  name: node-1
  labels:
    kubernetes.io/hostname: node-1
    node-role.kubernetes.io/worker: "true"
    zone: us-west-1
    disktype: ssd
  annotations:
    node.alpha.kubernetes.io/ttl: "0"
spec:
  podCIDR: 10.244.1.0/24
  providerID: aws:///us-west-1a/i-1234567890abcdef0
status:
  capacity:
    cpu: "4"
    memory: 16Gi
    pods: "110"
  allocatable:
    cpu: "4"
    memory: 15Gi
    pods: "110"
  conditions:
  - type: Ready
    status: "True"
    lastHeartbeatTime: "2026-01-13T00:00:00Z"
    lastTransitionTime: "2026-01-13T00:00:00Z"
    reason: KubeletReady
    message: kubelet is posting ready status
  addresses:
  - type: InternalIP
    address: 192.168.1.100
  - type: Hostname
    address: node-1
  nodeInfo:
    osImage: Ubuntu 20.04.3 LTS
    kernelVersion: 5.4.0-1042-aws
    kubeletVersion: v1.21.0
    containerRuntimeVersion: docker://20.10.7
    operatingSystem: linux
    architecture: amd64
```

## 🔧 Node组件

### Kubelet
- **职责**：与API Server通信，管理Pod生命周期
- **功能**：
  - 接收并执行Pod创建、更新、删除指令
  - 定期上报节点状态
  - 管理容器运行时
  - 挂载Volume

### Kube-proxy
- **职责**：维护网络规则，实现Service负载均衡
- **功能**：
  - 监听Service和Endpoint变化
  - 更新iptables或IPVS规则
  - 实现Pod间网络通信

### 容器运行时
- **Docker**：最常用的容器运行时
- **containerd**：更轻量的容器运行时
- **CRI-O**：Kubernetes原生容器运行时

## 📊 资源管理

### 资源容量
```yaml
status:
  capacity:
    cpu: "4"
    memory: 16Gi
    ephemeral-storage: 100Gi
    pods: "110"
```

### 可分配资源
```yaml
status:
  allocatable:
    cpu: "3.8"
    memory: 15Gi
    ephemeral-storage: 90Gi
    pods: "110"
```

### 资源预留
```yaml
# kubelet配置
--system-reserved=cpu=200m,memory=500Mi
--kube-reserved=cpu=200m,memory=500Mi
--eviction-hard=memory.available<500Mi,nodefs.available<10%
```

## 🔧 节点选择

### 节点标签
```bash
kubectl label nodes node-1 disktype=ssd
kubectl label nodes node-1 zone=us-west-1
```

### nodeSelector
```yaml
spec:
  template:
    spec:
      nodeSelector:
        disktype: ssd
        zone: us-west-1
```

### nodeAffinity
```yaml
spec:
  template:
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: disktype
                operator: In
                values:
                - ssd
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            preference:
              matchExpressions:
              - key: zone
                operator: In
                values:
                - us-west-1
```

## 🔧 污点和容忍度

### 污点类型
```bash
# NoSchedule：除非Pod有容忍度，否则不调度
kubectl taint nodes node-1 key=value:NoSchedule

# PreferNoSchedule：尽量避免调度
kubectl taint nodes node-1 key=value:PreferNoSchedule

# NoExecute：不调度且驱逐现有Pod
kubectl taint nodes node-1 key=value:NoExecute
```

### 容忍度
```yaml
spec:
  template:
    spec:
      tolerations:
      - key: "key"
        operator: "Equal"
        value: "value"
        effect: "NoSchedule"
      - key: "key1"
        operator: "Exists"
        effect: "NoExecute"
        tolerationSeconds: 3600
```

### 常见污点
```bash
# Master节点
kubectl taint nodes master node-role.kubernetes.io/master:NoSchedule

# 专用节点
kubectl taint nodes node-1 dedicated=database:NoSchedule

# 硬件特殊节点
kubectl taint nodes node-1 gpu=true:NoSchedule
```

## 🔗 相关概念

### 配合使用
- [[Pod概念]]：Pod调度到Node上运行
- [[DaemonSet概念]]：在每个Node上运行Pod
- [[Namespace概念]]：Node是集群级别的资源

### 控制平面
- **Scheduler**：负责Pod调度到Node
- **Controller Manager**：管理Node生命周期
- **API Server**：Node与控制平面通信

## 🚀 最佳实践

### 1. 节点标签
```bash
# 按角色标签
kubectl label nodes node-1 node-role.kubernetes.io/worker="true"

# 按区域标签
kubectl label nodes node-1 topology.kubernetes.io/zone=us-west-1

# 按硬件标签
kubectl label nodes node-1 node.kubernetes.io/instance-type=m5.large
```

### 2. 资源预留
```yaml
# kubelet配置
--system-reserved=cpu=200m,memory=500Mi,ephemeral-storage=1Gi
--kube-reserved=cpu=200m,memory=500Mi,ephemeral-storage=1Gi
```

### 3. 污点管理
```bash
# Master节点污点
kubectl taint nodes master node-role.kubernetes.io/master:NoSchedule

# 专用节点污点
kubectl taint nodes node-1 dedicated=database:NoSchedule
```

### 4. 节点监控
- 监控节点资源使用率
- 监控节点健康状态
- 设置节点告警规则

### 5. 节点维护
```bash
# 安全驱逐Pod
kubectl drain node-1 --ignore-daemonsets --delete-emptydir-data

# 恢复节点
kubectl uncordon node-1
```

### 6. 节点自动扩缩容
- 使用Cluster Autoscaler自动扩缩容
- 配置合理的扩缩容策略
- 监控扩缩容事件

## 💡 架构思考

Node是Kubernetes集群的基础计算单元：
- **资源抽象**：将物理或虚拟资源抽象为统一的管理单元
- **调度目标**：Pod调度到合适的Node上运行
- **状态管理**：维护Node的健康状态和资源信息

从架构师视角：
- Node设计体现了Kubernetes的资源抽象能力
- 理解Node的工作原理有助于优化集群资源利用率
- 合理的节点规划是构建高可用集群的基础

## 🔍 深入学习

### 相关文档
- [Kubernetes官方Node文档](https://kubernetes.io/docs/concepts/architecture/nodes/)
- [[Kubernetes/数据平面/Kubelet]]：Kubelet工作原理
- [[Kubernetes/调度/Scheduler]]：Pod调度原理

### 实践案例
- [[生产环境K8s部署/节点规划]]
- [[生产环境K8s部署/节点维护]]
- [[故障排查与恢复/节点问题]]
