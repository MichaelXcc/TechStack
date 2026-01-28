---
type: concept
tags: [kubernetes, networkpolicy, 网络]
title: NetworkPolicy概念
status: 已掌握
priority: 高
related: ["Pod", "Service", "Network"]
---

# NetworkPolicy概念

## 📋 定义

NetworkPolicy是Kubernetes中用于控制Pod间网络通信的策略，实现精细的网络隔离和安全控制。

## 🎯 核心特性

### 1. 网络隔离
- **默认拒绝**：默认拒绝所有网络流量
- **白名单**：只允许指定的网络流量
- **命名空间隔离**：不同Namespace的网络隔离

### 2. 精细控制
- **Pod选择器**：基于标签选择Pod
- **端口控制**：控制特定端口的访问
- **协议控制**：控制TCP、UDP等协议

### 3. 流量方向
- **Ingress**：控制入站流量
- **Egress**：控制出站流量
- **双向控制**：同时控制入站和出站流量

## 📝 NetworkPolicy结构

### 基本结构
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: my-network-policy
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

### 字段说明
- **podSelector**：选择应用策略的Pod
- **policyTypes**：策略类型（Ingress/Egress）
- **ingress**：入站规则
- **egress**：出站规则

## 🔧 Ingress规则

### 允许特定Pod访问
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 80
```

### 允许特定Namespace访问
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-namespace
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: frontend-namespace
    ports:
    - protocol: TCP
      port: 80
```

### 允许IP段访问
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-ipblock
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - ipBlock:
        cidr: 192.168.1.0/24
        except:
        - 192.168.1.1/32
    ports:
    - protocol: TCP
      port: 80
```

## 🔧 Egress规则

### 允许访问特定Service
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-service
spec:
  podSelector:
    matchLabels:
      app: frontend
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: backend
    ports:
    - protocol: TCP
      port: 80
```

### 允许访问外部网络
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-external
spec:
  podSelector:
    matchLabels:
      app: frontend
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 0.0.0.0/0
    ports:
    - protocol: TCP
      port: 443
```

### 允许DNS查询
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns
spec:
  podSelector:
    matchLabels:
      app: frontend
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector: {}
    ports:
    - protocol: UDP
      port: 53
```

## 🔧 默认拒绝策略

### 拒绝所有入站流量
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

### 拒绝所有出站流量
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-egress
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Egress
```

### 拒绝所有流量
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

## 🔧 复杂策略

### 多层策略
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: multi-layer-policy
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    - namespaceSelector:
        matchLabels:
          name: monitoring
    ports:
    - protocol: TCP
      port: 80
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    - ipBlock:
        cidr: 10.0.0.0/8
    ports:
    - protocol: TCP
      port: 3306
```

### 端口范围
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: port-range-policy
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector: {}
    ports:
    - protocol: TCP
      port: 3000
    - protocol: TCP
      port: 3001
    - protocol: TCP
      port: 3002
```

## 🔧 策略应用

### 查看NetworkPolicy
```bash
# 查看所有NetworkPolicy
kubectl get networkpolicy --all-namespaces

# 查看特定Namespace的NetworkPolicy
kubectl get networkpolicy -n <namespace>

# 查看NetworkPolicy详情
kubectl describe networkpolicy <policy-name> -n <namespace>
```

### 测试NetworkPolicy
```bash
# 从Pod1访问Pod2
kubectl exec -it pod1 -- ping pod2

# 从Pod访问Service
kubectl exec -it pod1 -- curl http://<service-name>.<namespace>.svc.cluster.local

# 查看NetworkPolicy日志
kubectl logs -n kube-system <cni-plugin-pod>
```

## 🔧 故障排查

### 检查NetworkPolicy
```bash
# 查看NetworkPolicy
kubectl get networkpolicy -n <namespace>

# 查看NetworkPolicy详情
kubectl describe networkpolicy <policy-name> -n <namespace>

# 检查Pod标签
kubectl get pod -n <namespace> --show-labels
```

### 检查网络连接
```bash
# 从Pod1访问Pod2
kubectl exec -it pod1 -- ping pod2

# 从Pod访问Service
kubectl exec -it pod1 -- curl http://<service-name>.<namespace>.svc.cluster.local

# 检查网络规则
kubectl exec -it pod1 -- iptables -L -n -v
```

### 检查CNI插件
```bash
# 查看CNI配置
cat /etc/cni/net.d/*.conf

# 查看CNI插件日志
kubectl logs -n kube-system <cni-plugin-pod>

# 检查网络接口
ip addr show
```

## 🔗 相关概念

### 网络相关
- [[Kubernetes网络模型]]：Kubernetes网络模型
- [[Service网络]]：Service网络
- [[Pod概念]]：NetworkPolicy控制的Pod

### 安全相关
- **RBAC**：基于角色的访问控制
- **PodSecurityPolicy**：Pod安全策略
- **Secret**：敏感配置管理

## 🚀 最佳实践

### 1. 默认拒绝策略
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

### 2. 最小权限原则
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-specific
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 80
```

### 3. 分层策略
```yaml
# 基础策略：默认拒绝
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress

---
# 应用策略：允许特定流量
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-specific
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
```

### 4. 监控NetworkPolicy
- 监控NetworkPolicy应用情况
- 监控网络连接
- 监控网络错误

### 5. 定期审查策略
- 定期审查NetworkPolicy
- 移除不必要的策略
- 更新过时的策略

### 6. 测试策略
- 在测试环境测试策略
- 验证策略效果
- 记录测试结果

## 💡 架构思考

NetworkPolicy是Kubernetes网络安全的核心：
- **网络隔离**：实现Pod间的网络隔离
- **精细控制**：控制Pod间的网络通信
- **安全合规**：满足安全合规要求

从架构师视角：
- NetworkPolicy设计体现了网络安全的最佳实践
- 理解NetworkPolicy的工作原理有助于设计安全的网络架构
- 合理的NetworkPolicy配置是构建安全集群的关键

## 🔍 深入学习

### 相关文档
- [Kubernetes官方NetworkPolicy文档](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [[Kubernetes/网络/Kubernetes网络模型]]：网络模型
- [[Kubernetes/网络/Service网络]]：Service网络

### 实践案例
- [[生产环境K8s部署/NetworkPolicy配置]]
- [[安全合规/网络策略]]
- [[故障排查与恢复/NetworkPolicy问题]]
