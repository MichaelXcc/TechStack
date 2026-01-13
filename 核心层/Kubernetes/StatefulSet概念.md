---
type: concept
tags: [kubernetes, statefulset, 核心概念]
title: StatefulSet概念
author: 云原生技术架构师
status: 已掌握
priority: 高
related: ["Pod", "Deployment", "Service"]
---

# StatefulSet概念

## 📋 定义

StatefulSet是Kubernetes中用于管理有状态应用的工作负载资源，它为Pod提供稳定的网络标识、持久化存储和有序的部署、扩缩容。

## 🎯 核心特性

### 1. 稳定的网络标识
- **有序Pod名称**：Pod名称包含序号（web-0、web-1、web-2）
- **稳定的DNS名称**：Pod拥有稳定的DNS记录
- **有序部署和删除**：Pod按顺序创建和删除

### 2. 持久化存储
- **稳定的存储**：每个Pod绑定独立的PVC
- **存储保留**：Pod删除后PVC保留
- **数据一致性**：确保数据与Pod绑定关系不变

### 3. 有序管理
- **有序部署**：从0到N-1依次创建Pod
- **有序删除**：从N-1到0依次删除Pod
- **有序扩缩容**：扩容从N开始，缩容到N-1

## 📝 YAML示例

### 基础StatefulSet
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "web"
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

### Headless Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
```

## 🔧 Pod标识

### Pod命名规则
- **格式**：`<statefulset-name>-<ordinal-index>`
- **示例**：web-0、web-1、web-2
- **特点**：序号从0开始，依次递增

### DNS记录
- **格式**：`<pod-name>.<service-name>.<namespace>.svc.cluster.local`
- **示例**：web-0.web.default.svc.cluster.local
- **特点**：Pod重建后DNS名称不变

### 网络标识
```bash
# Pod间通过稳定的DNS名称通信
web-0.web.default.svc.cluster.local
web-1.web.default.svc.cluster.local
web-2.web.default.svc.cluster.local
```

## 📊 部署策略

### 有序部署
```
web-0 (Pending → Running)
web-1 (Pending → Running)
web-2 (Pending → Running)
```

### 有序删除
```
web-2 (Running → Terminating)
web-1 (Running → Terminating)
web-0 (Running → Terminating)
```

### 有序扩容
```
web-0、web-1 (Running)
web-2 (Pending → Running)
```

### 有序缩容
```
web-0、web-1、web-2 (Running)
web-2 (Running → Terminating)
```

## 🔧 更新策略

### RollingUpdate（默认）
```yaml
spec:
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      partition: 0
```

- **partition**：指定更新边界，序号>=partition的Pod才会更新
- **示例**：partition=2时，只更新web-2、web-3等

### OnDelete
```yaml
spec:
  updateStrategy:
    type: OnDelete
```

- **手动触发**：删除Pod后才会创建新版本Pod
- **场景**：需要手动控制更新时机

## 🔗 相关概念

### 配合使用
- [[Pod概念]]：StatefulSet管理的Pod
- [[Service概念]]：Headless Service提供稳定的DNS
- [[Volume概念]]：PVC提供持久化存储

### 对比Deployment
- **Deployment**：无状态应用，Pod可互换
- **StatefulSet**：有状态应用，Pod有稳定标识

## 🚀 最佳实践

### 1. 使用Headless Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  clusterIP: None
  selector:
    app: nginx
```

### 2. 设置合理的partition
```yaml
spec:
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      partition: 2
```

### 3. 使用volumeClaimTemplates
```yaml
spec:
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
```

### 4. 配置Pod管理策略
```yaml
spec:
  podManagementPolicy: OrderedReady
  # 或 Parallel
```

### 5. 设置资源限制
```yaml
spec:
  template:
    spec:
      containers:
      - name: nginx
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
```

### 6. 健康检查
```yaml
spec:
  template:
    spec:
      containers:
      - name: nginx
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
```

## 💡 架构思考

StatefulSet体现了Kubernetes对有状态应用的支持：
- **稳定标识**：Pod拥有稳定的网络标识和存储
- **有序管理**：保证Pod的创建、删除顺序
- **数据持久化**：确保数据与Pod的绑定关系

从架构师视角：
- StatefulSet适合数据库、消息队列等有状态应用
- 理解StatefulSet的工作原理有助于设计高可用的有状态应用
- 结合Operator模式实现复杂有状态应用的自动化管理

## 🔍 深入学习

### 相关文档
- [Kubernetes官方StatefulSet文档](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
- [[Kubernetes/工作负载/StatefulSet控制器]]：StatefulSet控制器原理
- [[Kubernetes/存储/PV和PVC]]：持久化存储管理

### 实践案例
- [[生产环境K8s部署/数据库部署]]
- [[有状态应用/MySQL集群]]
- [[有状态应用/Kafka集群]]
- [[故障排查与恢复/StatefulSet问题]]
