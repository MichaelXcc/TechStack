---
type: concept
tags: [kubernetes, deployment, 核心概念]
title: Deployment概念
author: AI Infra / LLM Ops 基础设施
status: 已掌握
priority: 高
related: ["Pod", "ReplicaSet", "Service"]
---

# Deployment概念

## 📋 定义

Deployment是Kubernetes中用于管理无状态应用的工作负载资源，它提供了声明式的更新方式来管理Pod和ReplicaSet。

## 🎯 核心特性

### 1. 声明式管理
- **期望状态**：定义期望的Pod副本数、镜像版本等
- **自动协调**：Controller持续监控实际状态，使其与期望状态一致
- **自我修复**：Pod故障时自动创建新Pod

### 2. 滚动更新
- **零停机部署**：逐步替换旧Pod，保证服务连续性
- **回滚能力**：支持回滚到任意历史版本
- **健康检查**：基于Readiness Probe确保新Pod就绪后再继续

### 3. 扩缩容
- **手动扩缩容**：通过kubectl scale命令修改副本数
- **自动扩缩容**：结合HPA（Horizontal Pod Autoscaler）根据CPU/内存使用率自动调整

## 📝 YAML示例

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
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
        image: nginx:1.21.0
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

## 🔧 更新策略

### RollingUpdate（默认）
- **maxSurge**：升级过程中最多可以比期望状态多出的Pod数量（百分比或绝对值）
- **maxUnavailable**：升级过程中最多可以比期望状态少的Pod数量（百分比或绝对值）

### Recreate
- 先删除所有旧Pod，再创建新Pod
- 适用于无法同时运行新旧版本的应用

## 📊 部署状态

### Progressing
- Deployment正在创建ReplicaSet
- 正在扩容或缩容
- 正在进行滚动更新

### Available
- 所有期望的副本都已就绪
- 最小可用时长要求已满足

### Failed
- 部署失败（镜像拉取失败、资源不足等）
- Readiness Probe检查失败
- 配置错误

## 🔗 相关概念

### 管理的资源
- [[Pod概念]]：Deployment管理的最小部署单元
- [[ReplicaSet]]：Deployment创建和管理ReplicaSet来保证Pod副本数

### 配合使用
- [[Service概念]]：为Deployment暴露服务
- [[ConfigMap概念]]：配置管理
- [[Secret概念]]：敏感信息管理
- [[HPA]]：自动扩缩容

## 🚀 最佳实践

### 1. 健康检查配置
```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 3

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
  timeoutSeconds: 3
  failureThreshold: 3
```

### 2. 资源限制
- 始终设置`resources.requests`和`resources.limits`
- 避免资源争抢影响集群稳定性
- 合理设置CPU和内存比例

### 3. 镜像标签管理
- 避免使用`latest`标签
- 使用语义化版本（如`v1.2.3`）
- 结合CI/CD自动化构建和推送

### 4. 滚动更新优化
- 设置合理的`maxSurge`和`maxUnavailable`
- 使用Readiness Probe确保新Pod就绪
- 监控更新过程，准备回滚方案

## 💡 架构思考

Deployment体现了Kubernetes的"控制器模式"：
- **声明式API**：只需声明期望状态，无需关心如何实现
- **闭环控制**：持续监控和调整，确保系统稳定
- **自愈能力**：自动处理故障，减少人工干预

从架构师视角：
- Deployment适合无状态应用，如Web服务、API网关
- 对于有状态应用，应使用[[StatefulSet]]
- 理解Deployment的工作原理有助于设计高可用架构

## 🔍 深入学习

### 相关文档
- [Kubernetes官方Deployment文档](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [[Kubernetes/控制平面/DeploymentController]]：Deployment控制器原理
- [[Kubernetes/数据平面/Kubelet]]：Pod生命周期管理

### 实践案例
- [[生产环境K8s部署/金丝雀发布]]
- [[故障排查与恢复/Deployment回滚]]
- [[CI/CD集成/K8s部署流水线]]
