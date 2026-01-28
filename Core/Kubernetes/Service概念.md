---
type: concept
tags: [kubernetes, service, 核心概念]
title: Service概念
status: 已掌握
priority: 高
related: ["Pod", "Deployment", "Ingress"]
---

# Service概念

## 📋 定义

Service是Kubernetes中定义一组Pod访问策略的抽象，它提供稳定的网络端点，实现服务发现和负载均衡。

## 🎯 核心特性

### 1. 稳定的网络标识
- **ClusterIP**：集群内部可访问的虚拟IP
- **DNS名称**：通过CoreDNS提供域名解析
- **解耦Pod IP**：Pod重建后Service IP不变

### 2. 服务发现
- **环境变量**：Pod启动时注入相关Service信息
- **DNS解析**：通过`<service-name>.<namespace>.svc.cluster.local`访问
- **自动注册**：新Pod加入Service后自动被发现

### 3. 负载均衡
- **随机分发**：默认使用随机算法分发请求
- **会话保持**：通过`sessionAffinity`实现ClientIP会话保持
- **外部负载均衡**：集成云厂商的LoadBalancer

## 📝 YAML示例

### ClusterIP（默认）
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

## 🔧 Service类型

### ClusterIP
- **用途**：集群内部访问
- **特点**：仅在集群内可访问
- **场景**：微服务间通信

### NodePort
- **用途**：通过节点IP和端口访问
- **特点**：每个节点开放相同端口（30000-32767）
- **场景**：开发测试环境

### LoadBalancer
- **用途**：通过外部负载均衡器访问
- **特点**：需要云厂商支持
- **场景**：生产环境对外服务

### ExternalName
- **用途**：映射到外部DNS名称
- **特点**：不创建代理，仅DNS CNAME
- **场景**：访问外部服务

## 📊 端口配置

### 常用端口字段
- **port**：Service端口，客户端访问的端口
- **targetPort**：Pod容器端口，流量转发的目标端口
- **nodePort**：节点端口，NodePort类型使用

### 多端口配置
```yaml
ports:
- name: http
  protocol: TCP
  port: 80
  targetPort: 8080
- name: https
  protocol: TCP
  port: 443
  targetPort: 8443
```

## 🔗 相关概念

### 选择器
- **Pod选择器**：通过标签选择Pod
- **无选择器Service**：手动定义Endpoint

### Endpoints
- **自动创建**：基于选择器自动管理Endpoint
- **手动管理**：无选择器Service需手动创建Endpoint

### 配合使用
- [[Pod概念]]：Service代理的Pod集合
- [[Deployment概念]]：管理Service后端的Pod
- [[Ingress概念]]：HTTP/HTTPS路由规则
- [[CoreDNS]]：Service域名解析

## 🚀 最佳实践

### 1. 命名规范
```yaml
metadata:
  name: my-service  # 使用kebab-case
  labels:
    app: my-app
    tier: frontend
```

### 2. 端口命名
```yaml
ports:
- name: http-web
  port: 80
  targetPort: http-web  # 与Pod容器端口名称一致
```

### 3. 会话保持
```yaml
spec:
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10800  # 3小时
```

### 4. 健康检查
- 结合Readiness Probe确保只有就绪的Pod接收流量
- 使用合适的`externalTrafficPolicy`（Local/Cluster）

### 5. 资源限制
```yaml
spec:
  internalTrafficPolicy: Local  # 仅本地节点Pod
  externalTrafficPolicy: Local  # 保留客户端源IP
```

## 💡 架构思考

Service是Kubernetes服务网格的基础：
- **服务抽象**：将动态Pod集合抽象为稳定服务
- **解耦依赖**：服务间通过Service解耦，不直接依赖Pod IP
- **可观测性**：便于监控、日志和链路追踪

从架构师视角：
- Service是微服务架构的核心组件
- 理解Service的工作原理有助于设计高可用、可扩展的服务架构
- 生产环境建议使用[[Ingress]] + [[Service]]组合

## 🔍 深入学习

### 相关文档
- [Kubernetes官方Service文档](https://kubernetes.io/docs/concepts/services-networking/service/)
- [[Kubernetes/网络/Service网络]]：Service网络原理
- [[Kubernetes/网络/CoreDNS]]：DNS服务发现

### 实践案例
- [[生产环境K8s部署/服务暴露]]
- [[微服务架构/服务网格集成]]
- [[故障排查与恢复/Service访问问题]]
