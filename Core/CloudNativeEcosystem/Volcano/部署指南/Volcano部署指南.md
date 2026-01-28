---
type: guide
tags: [volcano, deployment, kubernetes]
title: Volcano部署指南
status: 学习中
priority: 高
related: ["Kubernetes部署", "Helm", "YAML"]
---

# Volcano部署指南

## 📋 概述

本指南介绍如何在Kubernetes集群中部署和配置Volcano，支持基于Helm和基于YAML两种部署方式。

## 📋 准备工作

### 1. 环境要求

| 组件 | 版本要求 |
|------|----------|
| Kubernetes | v1.20+ |
| Helm | v3.5+（如果使用Helm部署） |
| 节点数量 | 至少2个节点 |
| 资源要求 | 每个节点至少2CPU、4GB内存 |

### 2. 权限要求

- 集群管理员权限（cluster-admin）
- 能够创建CRD和命名空间

## 🚀 部署方式

### 1. 基于Helm部署
> 推荐使用Helm部署，便于管理和升级

#### 1.1 添加Helm仓库

```bash
helm repo add volcano-sh https://volcano-sh.github.io/helm-charts
helm repo update
```

#### 1.2 部署Volcano

```bash
# 创建命名空间
kubectl create namespace volcano-system

# 部署Volcano
ghelm install volcano volcano-sh/volcano \
  --namespace volcano-system \
  --set scheduler.enabled=true \
  --set controllerManager.enabled=true \
  --set admission.enabled=true \
  --set webhook.enabled=true
```

#### 1.3 自定义配置

```bash
# 使用自定义values.yaml部署
helm install volcano volcano-sh/volcano \
  --namespace volcano-system \
  -f custom-values.yaml
```

**custom-values.yaml示例：**

```yaml
scheduler:
  replicas: 2  # 启用高可用
  resources:
    requests:
      cpu: 2
      memory: 4Gi
    limits:
      cpu: 4
      memory: 8Gi

controllerManager:
  replicas: 2  # 启用高可用

admission:
  enabled: true

webhook:
  enabled: true
```

### 2. 基于YAML部署
> 适合没有Helm的环境

#### 2.1 下载部署文件

```bash
# 下载最新版本的部署文件
wget https://raw.githubusercontent.com/volcano-sh/volcano/master/installer/volcano-development.yaml
```

#### 2.2 部署Volcano

```bash
# 部署Volcano
kubectl apply -f volcano-development.yaml
```

#### 2.3 验证部署

```bash
# 检查Volcano组件状态
kubectl get pods -n volcano-system

# 检查CRD是否创建
kubectl get crds | grep volcano
```

## ⚙️ 配置优化

### 1. 调度器配置

#### 1.1 调整调度器参数

```bash
# 编辑调度器ConfigMap
kubectl edit configmap volcano-scheduler-config -n volcano-system
```

**核心配置项：**

| 配置项 | 说明 | 默认值 |
|--------|------|--------|
| `schedulerPolicyName` | 调度策略名称 | `default` |
| `actions` | 调度动作列表 | `["enqueue", "allocate", "backfill"]` |
| `tiers` | 调度器插件层级 | 包含默认插件 |

#### 1.2 启用自定义调度插件

```yaml
# 在ConfigMap中添加自定义插件
apiVersion: v1
kind: ConfigMap
metadata:
  name: volcano-scheduler-config
  namespace: volcano-system
data:
  volcano-scheduler.conf: |
    apiVersion: scheduler.volcano.sh/v1beta1
    kind: SchedulerConfiguration
    profiles:
    - name: default
      plugins:
        queue:
        - name: priority
        - name: fifo
        allocate:
        - name: drf
        - name: predicates
        - name: proportion
        - name: nodeorder
        - name: binpack
        backfill:
        - name: reclaim
        - name: predicates
        - name: proportion
        - name: nodeorder
        - name: binpack
        - name: custom-plugin  # 自定义插件
```

### 2. 资源配置优化

#### 2.1 调整组件资源限制

```bash
# 编辑Deployment资源限制
kubectl edit deployment volcano-scheduler -n volcano-system
kubectl edit deployment volcano-controllers -n volcano-system
```

#### 2.2 配置节点资源预留

```yaml
# 在节点上添加标签，用于资源预留
kubectl label nodes node-1 volcano.sh/reserved-cpu=2 volcano.sh/reserved-memory=4Gi
```

## 🔄 版本升级

### 1. Helm升级

```bash
# 更新Helm仓库
helm repo update

# 升级Volcano
helm upgrade volcano volcano-sh/volcano \
  --namespace volcano-system \
  -f custom-values.yaml
```

### 2. YAML升级

```bash
# 备份当前部署
kubectl get all -n volcano-system -o yaml > volcano-backup.yaml

# 下载新版本部署文件
wget https://raw.githubusercontent.com/volcano-sh/volcano/master/installer/volcano-development.yaml

# 升级部署
kubectl apply -f volcano-development.yaml
```

## ✅ 验证部署

### 1. 检查组件状态

```bash
# 检查Pod状态
kubectl get pods -n volcano-system

# 检查服务状态
kubectl get svc -n volcano-system

# 检查CRD
kubectl get crds | grep volcano
```

### 2. 运行测试Job

**测试Job YAML：**

```yaml
apiVersion: batch.volcano.sh/v1alpha1
kind: Job
metadata:
  name: test-job
spec:
  minAvailable: 1
  schedulerName: volcano
  queue: default
  tasks:
  - name: test-task
    replicas: 1
    template:
      spec:
        containers:
        - name: test-container
          image: busybox:latest
          command: ["sh", "-c", "echo Hello Volcano && sleep 10"]
        restartPolicy: OnFailure
```

**提交测试Job：**

```bash
kubectl apply -f test-job.yaml

# 检查Job状态
kubectl get jobs.batch.volcano.sh test-job
kubectl get pods -l volcano.sh/job-name=test-job
```

## 🧹 卸载Volcano

### 1. Helm卸载

```bash
helm uninstall volcano -n volcano-system
kubectl delete namespace volcano-system
```

### 2. YAML卸载

```bash
kubectl delete -f volcano-development.yaml
```

## 🔗 相关链接

### 部署方式
- [[基于Helm部署]]
- [[基于YAML部署]]

### 配置优化
- [[配置优化]]

### 版本升级
- [[版本升级]]

### 架构设计
- [[../架构设计/Volcano架构设计|Volcano架构设计]]

### 实践案例
- [[../实践案例/Volcano实践案例|Volcano实践案例]]

## 💡 最佳实践

1. **生产环境建议**：
   - 启用调度器和控制器的高可用（replicas=2+）
   - 为组件配置合适的资源限制
   - 定期备份配置和状态

2. **监控建议**：
   - 使用Prometheus监控Volcano组件
   - 配置Grafana仪表板查看调度状态
   - 配置告警规则，监控组件异常

3. **安全建议**：
   - 限制Volcano组件的RBAC权限
   - 启用Pod安全策略
   - 定期更新Volcano版本

## 💬 思考问题

1. 在生产环境中，你会选择哪种部署方式？为什么？
2. 如何配置Volcano的高可用？
3. 如何监控Volcano的调度性能？
4. 如何优化Volcano的资源使用？
