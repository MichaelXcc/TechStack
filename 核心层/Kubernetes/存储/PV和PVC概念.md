---
type: concept
tags: [kubernetes, pv, pvc, 存储]
title: PV和PVC概念
author: AI Infra / LLM Ops 基础设施
status: 已掌握
priority: 高
related: ["Volume", "StorageClass", "Pod"]
---

# PV和PVC概念

## 📋 定义

PersistentVolume（PV）是集群级别的存储资源，PersistentVolumeClaim（PVC）是命名空间级别的存储声明，通过PVC请求PV实现存储的动态分配。

## 🎯 核心特性

### 1. 存储抽象
- **PV**：集群级别的存储资源
- **PVC**：命名空间级别的存储声明
- **动态分配**：通过StorageClass动态创建PV

### 2. 生命周期管理
- **回收策略**：PV删除后的处理方式
- **状态管理**：PV和PVC的状态管理
- **绑定关系**：PVC与PV的绑定关系

### 3. 访问模式
- **ReadWriteOnce**：单节点读写
- **ReadOnlyMany**：多节点只读
- **ReadWriteMany**：多节点读写

## 📝 PV类型

### 静态PV
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /mnt/data
```

### 动态PV
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: fast
```

## 🔧 PV配置

### NFS PV
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
spec:
  capacity:
    storage: 100Gi
  accessModes:
  - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: nfs
  nfs:
    server: 192.168.1.100
    path: /exports/data
```

### AWS EBS PV
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: aws-ebs-pv
spec:
  capacity:
    storage: 100Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: gp2
  awsElasticBlockStore:
    volumeID: vol-1234567890abcdef0
    fsType: ext4
```

### GCE PD PV
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: gce-pd-pv
spec:
  capacity:
    storage: 100Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: standard
  gcePersistentDisk:
    pdName: my-disk
    fsType: ext4
```

## 🔧 PVC配置

### 基本PVC
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: fast
```

### 带StorageClass的PVC
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: fast
  selector:
    matchLabels:
      environment: production
```

### 只读PVC
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
  - ReadOnlyMany
  resources:
    requests:
      storage: 10Gi
  storageClassName: nfs
```

## 🔧 访问模式

### ReadWriteOnce (RWO)
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: rwo-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteOnce
```

**特点**：
- 单节点读写
- 适合块存储
- 支持大多数存储类型

### ReadOnlyMany (ROX)
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: rox-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadOnlyMany
```

**特点**：
- 多节点只读
- 适合共享数据
- 支持NFS等存储类型

### ReadWriteMany (RWX)
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: rwx-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteMany
```

**特点**：
- 多节点读写
- 适合共享存储
- 支持NFS、Ceph等存储类型

## 🔧 回收策略

### Retain
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  persistentVolumeReclaimPolicy: Retain
```

**特点**：
- PVC删除后PV保留
- 需要手动清理
- 适合重要数据

### Delete
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  persistentVolumeReclaimPolicy: Delete
```

**特点**：
- PVC删除后PV自动删除
- 自动清理底层存储
- 适合临时数据

### Recycle
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  persistentVolumeReclaimPolicy: Recycle
```

**特点**：
- PVC删除后PV保留但清空数据
- 已废弃，不推荐使用

## 🔧 状态管理

### PV状态
- **Available**：PV可用，未绑定
- **Bound**：PV已绑定到PVC
- **Released**：PVC已删除，PV未回收
- **Failed**：PV自动回收失败

### PVC状态
- **Pending**：PVC等待绑定
- **Bound**：PVC已绑定到PV
- **Lost**：PVC绑定的PV丢失

## 🔧 使用PVC

### Pod使用PVC
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: nginx
    volumeMounts:
    - name: data-volume
      mountPath: /data
  volumes:
  - name: data-volume
    persistentVolumeClaim:
      claimName: my-pvc
```

### StatefulSet使用PVC
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: my-statefulset
spec:
  serviceName: my-service
  replicas: 3
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: nginx
        volumeMounts:
        - name: data-volume
          mountPath: /data
  volumeClaimTemplates:
  - metadata:
      name: data-volume
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
      storageClassName: fast
```

## 🔧 监控和日志

### 查看PV和PVC
```bash
# 查看所有PV
kubectl get pv

# 查看所有PVC
kubectl get pvc --all-namespaces

# 查看PV详情
kubectl describe pv <pv-name>

# 查看PVC详情
kubectl describe pvc <pvc-name>
```

### 监控存储使用
```bash
# 查看存储使用
kubectl top pod <pod-name> --containers

# 查看节点存储
kubectl describe node <node-name> | grep -A 5 "Allocated resources"
```

## 🔧 故障排查

### 检查PV和PVC状态
```bash
# 查看PV状态
kubectl get pv

# 查看PVC状态
kubectl get pvc

# 查看PV详情
kubectl describe pv <pv-name>

# 查看PVC详情
kubectl describe pvc <pvc-name>
```

### 常见问题
```bash
# PVC无法绑定
kubectl describe pvc <pvc-name>
kubectl get pv

# 存储空间不足
kubectl describe pv <pv-name>
kubectl top node <node-name>

# 权限问题
kubectl auth can-i get pv
kubectl auth can-i get pvc
```

## 🔗 相关概念

### 存储相关
- [[Volume概念]]：Volume概念
- [[StorageClass]]：存储类
- [[Pod概念]]：使用PVC的Pod

### 存储类型
- **NFS**：网络文件系统
- **Ceph**：分布式存储
- **云存储**：AWS EBS、GCE PD等

## 🚀 最佳实践

### 1. 使用StorageClass动态分配
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: fast
```

### 2. 设置合理的回收策略
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  persistentVolumeReclaimPolicy: Retain
```

### 3. 使用合适的访问模式
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

### 4. 监控存储使用
- 监控PV使用情况
- 监控PVC使用情况
- 监控存储性能

### 5. 定期清理未使用的PV
```bash
# 查看未使用的PV
kubectl get pv | grep Available

# 删除未使用的PV
kubectl delete pv <pv-name>
```

### 6. 备份重要数据
- 定期备份PV数据
- 测试备份恢复流程
- 确保备份加密存储

## 💡 架构思考

PV和PVC是Kubernetes存储管理的核心：
- **存储抽象**：将底层存储抽象为PV
- **动态分配**：通过PVC动态分配存储
- **生命周期管理**：管理存储的生命周期

从架构师视角：
- PV和PVC设计体现了存储管理的最佳实践
- 理解PV和PVC的工作原理有助于设计存储架构
- 合理的存储配置是构建高可用集群的基础

## 🔍 深入学习

### 相关文档
- [Kubernetes官方PV和PVC文档](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- [[Kubernetes/存储/StorageClass]]：存储类
- [[Kubernetes/存储/Volume]]：Volume概念

### 实践案例
- [[生产环境K8s部署/存储配置]]
- [[有状态应用/数据库部署]]
- [[故障排查与恢复/存储问题]]
