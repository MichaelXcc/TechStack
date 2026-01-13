---
type: concept
tags: [kubernetes, service-network, 网络]
title: Service网络
author: 云原生技术架构师
status: 已掌握
priority: 高
related: ["Service", "Kube-proxy", "Network"]
---

# Service网络

## 📋 定义

Service网络是Kubernetes中实现服务发现和负载均衡的网络机制，为Pod提供稳定的网络端点。

## 🎯 核心特性

### 1. 服务发现
- **ClusterIP**：集群内部虚拟IP
- **DNS解析**：通过CoreDNS实现DNS解析
- **环境变量**：通过环境变量注入服务信息

### 2. 负载均衡
- **随机分发**：默认使用随机算法
- **会话保持**：支持ClientIP会话保持
- **健康检查**：检查后端Pod健康状态

### 3. 网络代理
- **iptables**：使用iptables实现代理
- **ipvs**：使用IPVS实现代理
- **userspace**：使用用户空间代理

## 📝 Service类型

### ClusterIP
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: ClusterIP
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

**特点**：
- 仅集群内部可访问
- 默认Service类型
- 通过ClusterIP访问

### NodePort
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
    nodePort: 30080
```

**特点**：
- 通过节点IP和端口访问
- 端口范围：30000-32767
- 适合开发测试环境

### LoadBalancer
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

**特点**：
- 通过外部负载均衡器访问
- 需要云厂商支持
- 适合生产环境

### ExternalName
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: ExternalName
  externalName: my.database.example.com
```

**特点**：
- 映射到外部DNS名称
- 不创建代理
- 适合访问外部服务

## 🔧 服务发现

### DNS解析
```bash
# Service DNS记录
<service-name>.<namespace>.svc.cluster.local

# 示例
my-service.default.svc.cluster.local
```

### 环境变量
```bash
# Pod启动时注入的环境变量
MY_SERVICE_SERVICE_HOST=10.96.0.1
MY_SERVICE_SERVICE_PORT=80
```

### Headless Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-headless-service
spec:
  clusterIP: None
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

**特点**：
- 不分配ClusterIP
- 直接返回Pod IP列表
- 适合有状态应用

## 🔧 负载均衡

### 随机分发
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

### 会话保持
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10800
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

### 外部流量策略
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  externalTrafficPolicy: Local
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

## 🔧 Endpoints

### 自动创建
```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: my-service
subsets:
- addresses:
  - ip: 10.244.1.2
  - ip: 10.244.1.3
  - ip: 10.244.1.4
  ports:
  - port: 8080
    protocol: TCP
```

### 手动创建
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080

---
apiVersion: v1
kind: Endpoints
metadata:
  name: my-service
subsets:
- addresses:
  - ip: 192.168.1.100
  ports:
  - port: 8080
    protocol: TCP
```

## 🔧 网络代理

### iptables规则
```bash
# 查看Service规则
iptables -t nat -L KUBE-SERVICES -n -v

# 查看特定Service规则
iptables -t nat -L KUBE-SVC-XXX -n -v
```

### ipvs规则
```bash
# 查看ipvs规则
ipvsadm -Ln

# 查看详细信息
ipvsadm -Ln --stats
```

### 连接跟踪
```bash
# 查看连接跟踪
conntrack -L

# 查看Service连接
conntrack -L | grep <service-ip>
```

## 🔧 监控和日志

### Prometheus监控
```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'kube-proxy'
    scheme: http
    static_configs:
    - targets: ['<node-ip>:10249']
```

### 关键指标
```
kubeproxy_sync_proxy_rules_last_timestamp_seconds
kubeproxy_sync_proxy_rules_latency_seconds
kubeproxy_network_programming_duration_seconds
kubeproxy_sync_proxy_rules_endpoint_changes_pending
```

### 日志配置
```bash
# 查看Kube-proxy日志
kubectl logs -n kube-system <kube-proxy-pod>

# 查看网络规则
iptables -t nat -L -n -v
ipvsadm -Ln
```

## 🔧 故障排查

### 检查Service
```bash
# 查看Service
kubectl get svc

# 查看Service详情
kubectl describe svc <service-name>

# 查看Endpoints
kubectl get endpoints <service-name>
```

### 检查网络连接
```bash
# 从Pod访问Service
kubectl exec -it <pod-name> -- curl http://<service-name>.<namespace>.svc.cluster.local

# 检查DNS解析
kubectl exec -it <pod-name> -- nslookup <service-name>.<namespace>.svc.cluster.local
```

### 检查网络规则
```bash
# 查看iptables规则
iptables -t nat -L KUBE-SERVICES -n -v

# 查看ipvs规则
ipvsadm -Ln

# 查看连接跟踪
conntrack -L
```

## 🔗 相关概念

### 网络相关
- [[Kubernetes网络模型]]：Kubernetes网络模型
- [[NetworkPolicy]]：网络策略
- [[Kube-proxy概念]]：网络代理

### Service相关
- [[Service概念]]：Service概念
- [[Ingress概念]]：HTTP/HTTPS路由

## 🚀 最佳实践

### 1. 使用Headless Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-headless-service
spec:
  clusterIP: None
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

### 2. 配置会话保持
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10800
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

### 3. 配置外部流量策略
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  externalTrafficPolicy: Local
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

### 4. 监控Service性能
- 监控Service响应时间
- 监控Service错误率
- 监控后端Pod健康状态

### 5. 优化网络规则
- 使用ipvs模式提高性能
- 配置合理的同步周期
- 优化连接跟踪表

### 6. 定期检查Service
- 检查Service状态
- 检查Endpoints状态
- 检查网络规则

## 💡 架构思考

Service网络是Kubernetes服务发现的核心：
- **服务发现**：通过DNS实现服务发现
- **负载均衡**：实现Service的负载均衡
- **网络代理**：维护网络规则

从架构师视角：
- Service网络设计体现了服务发现的最佳实践
- 理解Service网络的工作原理有助于优化网络性能
- 合理的Service配置是构建高可用集群的关键

## 🔍 深入学习

### 相关文档
- [Kubernetes官方Service文档](https://kubernetes.io/docs/concepts/services-networking/service/)
- [[Kubernetes/网络/Kubernetes网络模型]]：网络模型
- [[Kubernetes/数据平面/Kube-proxy]]：Kube-proxy

### 实践案例
- [[生产环境K8s部署/Service配置]]
- [[性能优化/Service优化]]
- [[故障排查与恢复/Service问题]]
