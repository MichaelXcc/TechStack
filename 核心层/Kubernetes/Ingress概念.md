---
type: concept
tags: [kubernetes, ingress, 核心概念]
title: Ingress概念
author: 云原生技术架构师
status: 已掌握
priority: 高
related: ["Service", "Pod", "Deployment"]
---

# Ingress概念

## 📋 定义

Ingress是Kubernetes中用于管理集群外部访问服务的API对象，它提供HTTP/HTTPS路由规则，将外部流量路由到集群内的Service。

## 🎯 核心特性

### 1. HTTP/HTTPS路由
- **基于主机名路由**：根据Host头路由到不同Service
- **基于路径路由**：根据URL路径路由到不同Service
- **TLS终止**：在Ingress层处理HTTPS

### 2. 负载均衡
- **七层负载均衡**：基于HTTP/HTTPS协议的负载均衡
- **会话保持**：支持基于Cookie的会话保持
- **健康检查**：后端Service健康检查

### 3. 高级功能
- **重定向**：HTTP到HTTPS重定向
- **重写**：URL重写规则
- **限流**：请求限流和速率限制
- **认证**：基本认证、OAuth等

## 📝 YAML示例

### 基础Ingress
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /app1
        pathType: Prefix
        backend:
          service:
            name: app1-service
            port:
              number: 80
      - path: /app2
        pathType: Prefix
        backend:
          service:
            name: app2-service
            port:
              number: 80
```

### TLS配置
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-ingress
spec:
  tls:
  - hosts:
    - example.com
    secretName: tls-secret
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-service
            port:
              number: 80
```

### 默认后端
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: default-backend-ingress
spec:
  defaultBackend:
    service:
      name: default-service
      port:
        number: 80
  rules:
  - host: example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
```

## 🔧 Ingress Controller

### Nginx Ingress Controller
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /app(/|$)(.*)
        pathType: ImplementationSpecific
        backend:
          service:
            name: app-service
            port:
              number: 80
```

### Traefik Ingress Controller
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: traefik-ingress
  annotations:
    traefik.ingress.kubernetes.io/router.entrypoints: web,websecure
    traefik.ingress.kubernetes.io/router.tls: "true"
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-service
            port:
              number: 80
```

### Istio Gateway
```yaml
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: my-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"

---
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: my-virtualservice
spec:
  hosts:
  - "*"
  gateways:
  - my-gateway
  http:
  - match:
    - uri:
        prefix: /app1
    route:
    - destination:
        host: app1-service
  - match:
    - uri:
        prefix: /app2
    route:
    - destination:
        host: app2-service
```

## 📊 路径类型

### Exact
- **含义**：精确匹配URL路径
- **示例**：`/app`只匹配`/app`，不匹配`/app/`或`/app1`

### Prefix
- **含义**：前缀匹配URL路径
- **示例**：`/app`匹配`/app`、`/app/`、`/app1`

### ImplementationSpecific
- **含义**：由Ingress Controller决定匹配方式
- **示例**：Nginx使用正则匹配

## 🔧 常用注解

### Nginx Ingress注解
```yaml
metadata:
  annotations:
    # 重写路径
    nginx.ingress.kubernetes.io/rewrite-target: /$2
    
    # SSL重定向
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    
    # 限流
    nginx.ingress.kubernetes.io/limit-rps: "10"
    
    # CORS
    nginx.ingress.kubernetes.io/enable-cors: "true"
    nginx.ingress.kubernetes.io/cors-allow-origin: "*"
    
    # 认证
    nginx.ingress.kubernetes.io/auth-type: basic
    nginx.ingress.kubernetes.io/auth-secret: basic-auth
    
    # 超时
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "30"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "30"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "30"
```

### Traefik Ingress注解
```yaml
metadata:
  annotations:
    # 入口点
    traefik.ingress.kubernetes.io/router.entrypoints: web,websecure
    
    # 中间件
    traefik.ingress.kubernetes.io/router.middlewares: namespace-middleware@kubernetescrd
    
    # 优先级
    traefik.ingress.kubernetes.io/router.priority: "10"
```

## 🔗 相关概念

### 配合使用
- [[Service概念]]：Ingress路由到Service
- [[Pod概念]]：Service代理的Pod
- [[Deployment概念]]：管理Service后端的Pod
- [[Secret概念]]：存储TLS证书

### Ingress Controller
- **Nginx Ingress Controller**：最常用的Ingress Controller
- **Traefik**：云原生边缘路由器
- **Istio Gateway**：服务网格入口
- **HAProxy**：高性能负载均衡器

## 🚀 最佳实践

### 1. 使用TLS
```yaml
spec:
  tls:
  - hosts:
    - example.com
    secretName: tls-secret
```

### 2. 设置默认后端
```yaml
spec:
  defaultBackend:
    service:
      name: default-service
      port:
        number: 80
```

### 3. 使用命名空间隔离
```yaml
metadata:
  name: my-ingress
  namespace: my-namespace
```

### 4. 监控和日志
- 启用Ingress Controller的访问日志
- 监控Ingress的请求和响应指标
- 设置告警规则

### 5. 限流保护
```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/limit-rps: "100"
    nginx.ingress.kubernetes.io/limit-connections: "50"
```

### 6. 健康检查
- 配置后端Service的健康检查
- 使用Readiness Probe确保Pod就绪
- 监控Ingress Controller的健康状态

## 💡 架构思考

Ingress是Kubernetes网络流量的入口：
- **统一入口**：为集群提供统一的HTTP/HTTPS入口
- **流量管理**：实现基于域名的流量路由
- **安全防护**：在入口层实现认证、限流等安全措施

从架构师视角：
- Ingress是实现微服务架构的关键组件
- 理解Ingress的工作原理有助于设计高可用的网络架构
- 结合Service Mesh实现更复杂的流量管理

## 🔍 深入学习

### 相关文档
- [Kubernetes官方Ingress文档](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [[Kubernetes/网络/Ingress网络]]：Ingress网络原理
- [[Kubernetes/网络/Ingress Controller]]：Ingress Controller配置

### 实践案例
- [[生产环境K8s部署/Ingress配置]]
- [[微服务架构/流量管理]]
- [[故障排查与恢复/Ingress访问问题]]
