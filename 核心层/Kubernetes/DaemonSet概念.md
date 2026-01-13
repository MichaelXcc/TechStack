---
type: concept
tags: [kubernetes, daemonset, 核心概念]
title: DaemonSet概念
author: 云原生技术架构师
status: 已掌握
priority: 高
related: ["Pod", "Node", "Deployment"]
---

# DaemonSet概念

## 📋 定义

DaemonSet是Kubernetes中用于在每个节点上运行Pod副本的工作负载资源，它确保集群中所有（或部分）节点都运行一个Pod副本。

## 🎯 核心特性

### 1. 节点级部署
- **全节点覆盖**：在集群所有节点上运行Pod
- **选择性部署**：通过节点选择器在特定节点上运行Pod
- **自动扩展**：新节点加入时自动部署Pod

### 2. 系统级服务
- **监控代理**：在每个节点上运行监控代理
- **日志收集**：在每个节点上运行日志收集器
- **网络插件**：在每个节点上运行网络插件

### 3. 资源保证
- **优先级**：DaemonSet Pod通常具有高优先级
- **容忍度**：自动容忍污点，可在所有节点运行
- **资源预留**：系统预留资源确保DaemonSet Pod运行

## 📝 YAML示例

### 基础DaemonSet
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: kube-system
spec:
  selector:
    matchLabels:
      name: fluentd
  template:
    metadata:
      labels:
        name: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluent/fluentd:v1.14
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

### 节点选择器
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      nodeSelector:
        node-role.kubernetes.io/worker: "true"
      containers:
      - name: node-exporter
        image: prom/node-exporter:latest
        ports:
        - containerPort: 9100
          hostPort: 9100
```

### 容忍度配置
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: monitoring-agent
spec:
  selector:
    matchLabels:
      app: monitoring-agent
  template:
    metadata:
      labels:
        app: monitoring-agent
    spec:
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      - key: node.kubernetes.io/not-ready
        effect: NoExecute
        operator: Exists
      containers:
      - name: agent
        image: monitoring-agent:latest
```

## 🔧 节点选择

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
              - key: kubernetes.io/arch
                operator: In
                values:
                - amd64
                - arm64
```

### 污点和容忍度
```yaml
spec:
  template:
    spec:
      tolerations:
      - key: "key1"
        operator: "Equal"
        value: "value1"
        effect: "NoSchedule"
      - key: "key1"
        operator: "Equal"
        value: "value1"
        effect: "NoExecute"
```

## 📊 更新策略

### RollingUpdate（默认）
```yaml
spec:
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
```

- **maxUnavailable**：同时不可用的最大Pod数
- **特点**：逐个节点更新，确保服务连续性

### OnDelete
```yaml
spec:
  updateStrategy:
    type: OnDelete
```

- **手动触发**：删除Pod后才会创建新版本Pod
- **场景**：需要手动控制更新时机

## 🔧 资源管理

### 资源请求和限制
```yaml
spec:
  template:
    spec:
      containers:
      - name: fluentd
        resources:
          requests:
            cpu: 100m
            memory: 200Mi
          limits:
            cpu: 200m
            memory: 400Mi
```

### 优先级
```yaml
spec:
  template:
    spec:
      priorityClassName: system-node-critical
```

### 资源预留
```yaml
# kubelet配置
--system-reserved=cpu=200m,memory=500Mi
--kube-reserved=cpu=200m,memory=500Mi
```

## 🔗 相关概念

### 配合使用
- [[Pod概念]]：DaemonSet管理的Pod
- [[Node概念]]：DaemonSet Pod运行的节点
- [[Volume概念]]：访问节点文件系统

### 对比Deployment
- **Deployment**：运行指定数量的Pod副本
- **DaemonSet**：在每个节点运行一个Pod副本

## 🚀 最佳实践

### 1. 设置合理的资源限制
```yaml
spec:
  template:
    spec:
      containers:
      - name: fluentd
        resources:
          requests:
            cpu: 100m
            memory: 200Mi
          limits:
            cpu: 200m
            memory: 400Mi
```

### 2. 使用优先级
```yaml
spec:
  template:
    spec:
      priorityClassName: system-node-critical
```

### 3. 配置容忍度
```yaml
spec:
  template:
    spec:
      tolerations:
      - operator: Exists
```

### 4. 使用hostPath访问节点资源
```yaml
spec:
  template:
    spec:
      volumes:
      - name: host-root
        hostPath:
          path: /
```

### 5. 设置优雅终止
```yaml
spec:
  template:
    spec:
      terminationGracePeriodSeconds: 30
```

### 6. 监控DaemonSet状态
- 监控DaemonSet Pod的运行状态
- 检查节点覆盖率
- 监控资源使用情况

## 💡 架构思考

DaemonSet体现了Kubernetes的节点级管理能力：
- **节点代理**：在每个节点运行代理程序
- **系统服务**：提供系统级的基础设施服务
- **自动化部署**：新节点自动部署Pod

从架构师视角：
- DaemonSet适合监控、日志、网络等系统级服务
- 理解DaemonSet的工作原理有助于设计集群级基础设施
- 结合节点选择器实现灵活的节点管理策略

## 🔍 深入学习

### 相关文档
- [Kubernetes官方DaemonSet文档](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)
- [[Kubernetes/工作负载/DaemonSet控制器]]：DaemonSet控制器原理
- [[Kubernetes/调度/节点选择]]：节点选择和调度

### 实践案例
- [[生产环境K8s部署/监控部署]]
- [[生产环境K8s部署/日志收集]]
- [[生产环境K8s部署/网络插件]]
- [[故障排查与恢复/DaemonSet问题]]
