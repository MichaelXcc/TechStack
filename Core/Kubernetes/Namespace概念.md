---
type: concept
tags: [kubernetes, namespace, 核心概念]
title: Namespace概念
status: 已掌握
priority: 高
related: ["Pod", "ResourceQuota", "RBAC"]
---

# Namespace概念

## 📋 定义

Namespace是Kubernetes中用于实现资源隔离的逻辑分组，它将集群资源划分为多个虚拟集群，实现多租户、多环境的资源隔离。

## 🎯 核心特性

### 1. 资源隔离
- **命名隔离**：不同Namespace中的资源可以同名
- **资源隔离**：Pod、Service等资源在Namespace间隔离
- **网络隔离**：可通过NetworkPolicy实现网络隔离

### 2. 权限控制
- **RBAC绑定**：为不同Namespace设置不同的访问权限
- **资源配额**：限制Namespace的资源使用量
- **策略隔离**：不同Namespace应用不同的策略

### 3. 环境管理
- **多环境部署**：dev、staging、prod等环境隔离
- **多租户支持**：不同团队或客户使用不同Namespace
- **应用分组**：按应用或服务分组管理资源

## 📝 YAML示例

### 创建Namespace
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: my-namespace
  labels:
    name: my-namespace
    environment: production
    team: platform
```

### 资源指定Namespace
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  namespace: my-namespace
spec:
  containers:
  - name: my-container
    image: nginx
```

## 🔧 系统Namespace

### default
- **用途**：默认Namespace，未指定Namespace的资源创建在此
- **特点**：集群创建时自动创建
- **场景**：测试、临时资源

### kube-system
- **用途**：Kubernetes系统组件
- **特点**：包含kube-apiserver、kube-scheduler等
- **场景**：系统管理，一般不在此创建应用

### kube-public
- **用途**：公共资源，所有用户可读
- **特点**：包含集群信息
- **场景**：公共配置、集群信息

### kube-node-lease
- **用途**：节点租约，用于心跳检测
- **特点**：包含节点健康状态
- **场景**：节点管理

## 📊 资源配额

### ResourceQuota
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources
  namespace: my-namespace
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    persistentvolumeclaims: "4"
```

### LimitRange
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: my-namespace
spec:
  limits:
  - default:
      cpu: "500m"
      memory: "512Mi"
    defaultRequest:
      cpu: "100m"
      memory: "256Mi"
    type: Container
```

## 🔧 管理操作

### 查看Namespace
```bash
kubectl get namespaces
kubectl describe namespace my-namespace
```

### 创建Namespace
```bash
kubectl create namespace my-namespace
kubectl apply -f namespace.yaml
```

### 切换默认Namespace
```bash
kubectl config set-context --current --namespace=my-namespace
```

### 删除Namespace
```bash
kubectl delete namespace my-namespace
```

## 🔗 相关概念

### 配合使用
- [[Pod概念]]：Pod属于特定Namespace
- [[Service概念]]：Service在Namespace内提供服务发现
- [[ConfigMap]]：ConfigMap在Namespace内隔离
- [[Secret]]：Secret在Namespace内隔离

### 策略和配额
- **ResourceQuota**：限制Namespace资源使用
- **LimitRange**：设置默认资源限制
- **NetworkPolicy**：控制Namespace间网络访问
- **RBAC**：控制Namespace访问权限

## 🚀 最佳实践

### 1. 命名规范
```yaml
metadata:
  name: <team>-<environment>
  # 例如: platform-prod, frontend-dev
  labels:
    team: platform
    environment: production
    owner: platform-team
```

### 2. 环境隔离
```yaml
# 开发环境
apiVersion: v1
kind: Namespace
metadata:
  name: app-dev
  labels:
    environment: development

---
# 测试环境
apiVersion: v1
kind: Namespace
metadata:
  name: app-staging
  labels:
    environment: staging

---
# 生产环境
apiVersion: v1
kind: Namespace
metadata:
  name: app-prod
  labels:
    environment: production
```

### 3. 资源配额设置
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: namespace-quota
spec:
  hard:
    pods: "10"
    services: "5"
    persistentvolumeclaims: "4"
    requests.cpu: "2"
    requests.memory: 4Gi
    limits.cpu: "4"
    limits.memory: 8Gi
```

### 4. 网络隔离
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-from-other-namespaces
  namespace: my-namespace
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector: {}
```

### 5. RBAC配置
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: my-namespace
  name: namespace-admin
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: namespace-admin-binding
  namespace: my-namespace
subjects:
- kind: User
  name: admin@example.com
roleRef:
  kind: Role
  name: namespace-admin
  apiGroup: rbac.authorization.k8s.io
```

## 💡 架构思考

Namespace体现了Kubernetes的多租户设计理念：
- **逻辑隔离**：在同一物理集群上实现逻辑隔离
- **资源共享**：多个Namespace共享集群资源，提高利用率
- **灵活管理**：不同Namespace可应用不同的策略和配额

从架构师视角：
- Namespace是实现多环境、多租户的关键机制
- 合理的Namespace设计有助于资源管理和权限控制
- 结合RBAC和ResourceQuota实现企业级资源管理

## 🔍 深入学习

### 相关文档
- [Kubernetes官方Namespace文档](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)
- [[Kubernetes/安全/RBAC]]：基于角色的访问控制
- [[Kubernetes/资源管理/ResourceQuota]]：资源配额管理

### 实践案例
- [[生产环境K8s部署/多环境管理]]
- [[多租户架构/Namespace隔离]]
- [[故障排查与恢复/Namespace资源问题]]
