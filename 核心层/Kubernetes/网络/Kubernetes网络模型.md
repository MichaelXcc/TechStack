---
type: concept
tags: [kubernetes, network, 网络]
title: Kubernetes网络模型
author: AI Infra / LLM Ops 基础设施
status: 已掌握
priority: 高
related: ["Service", "Pod", "CNI"]
---

# Kubernetes网络模型

## 📋 定义

Kubernetes网络模型定义了Pod之间、Pod与Service之间、Pod与外部之间的网络通信规则。

## 🎯 核心特性

### 1. 扁平网络
- **所有Pod在同一网络**：Pod之间可以直接通信
- **无需NAT**：Pod之间通信无需NAT
- **IP分配**：每个Pod拥有独立的IP地址

### 2. 网络隔离
- **Namespace隔离**：不同Namespace的网络隔离
- **NetworkPolicy**：精细的网络策略控制
- **节点隔离**：节点级别的网络隔离

### 3. 服务发现
- **DNS服务**：通过CoreDNS实现服务发现
- **环境变量**：通过环境变量注入服务信息
- **Service**：通过Service实现负载均衡

## 📝 网络模型要求

### 基本要求
1. **所有Pod可以在不使用NAT的情况下与所有其他Pod通信**
2. **所有Node可以在不使用NAT的情况下与所有Pod通信**
3. **Pod看到的自己IP与其他看到的Pod IP相同**

### 网络架构
```
┌─────────────────────────────────────────────────┐
│             Kubernetes Cluster               │
│                                          │
│  ┌──────────┐  ┌──────────┐            │
│  │  Node 1  │  │  Node 2  │            │
│  │          │  │          │            │
│  │  ┌────┐ │  │  ┌────┐ │            │
│  │  │Pod1│ │  │  │Pod3│ │            │
│  │  │10.0│ │  │  │10.0│ │            │
│  │  │.1.1│ │  │  │.1.3│ │            │
│  │  └────┘ │  │  └────┘ │            │
│  │  ┌────┐ │  │  ┌────┐ │            │
│  │  │Pod2│ │  │  │Pod4│ │            │
│  │  │10.0│ │  │  │10.0│ │            │
│  │  │.1.2│ │  │  │.1.4│ │            │
│  │  └────┘ │  │  └────┘ │            │
│  └──────────┘  └──────────┘            │
│                                          │
│  ┌──────────────────────────────┐          │
│  │      Service (ClusterIP)     │          │
│  │      10.96.0.1            │          │
│  └──────────────────────────────┘          │
└─────────────────────────────────────────────────┘
```

## 🔧 Pod网络

### Pod IP分配
```yaml
# kube-controller-manager配置
--cluster-cidr=10.244.0.0/16
--allocate-node-cidrs=true
```

### Pod间通信
```bash
# Pod1访问Pod2
kubectl exec -it pod1 -- ping 10.244.1.2

# 查看Pod IP
kubectl get pod pod1 -o wide
```

### Pod网络命名空间
```bash
# 查看Pod网络命名空间
kubectl exec -it pod1 -- ip netns identify

# 进入Pod网络命名空间
kubectl exec -it pod1 -- nsenter -t 1 -n
```

## 🔧 CNI插件

### CNI插件类型
- **Flannel**：简单的VXLAN网络
- **Calico**：支持NetworkPolicy的网络
- **Weave**：简单易用的网络插件
- **Cilium**：基于eBPF的网络插件

### Flannel配置
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: kube-flannel-cfg
  namespace: kube-system
data:
  net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "vxlan"
      }
    }
```

### Calico配置
```yaml
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  calicoNetwork:
    ipPools:
    - blockSize: 26
      cidr: 192.168.0.0/16
      encapsulation: VXLAN
      natOutgoing: Enabled
      nodeSelector: all()
```

### CNI插件安装
```bash
# 安装Flannel
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

# 安装Calico
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

# 安装Weave
kubectl apply -f https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')
```

## 🔧 DNS服务

### CoreDNS配置
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        errors
        health {
            lameduck 5s
        }
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
            pods insecure
            fallthrough in-addr.arpa ip6.arpa
            ttl 30
        }
        prometheus :9153
        forward . /etc/resolv.conf {
            max_concurrent 1000
        }
        cache 30
        loop
        reload
        loadbalance
    }
```

### DNS解析
```bash
# Pod间通过DNS通信
kubectl exec -it pod1 -- ping pod2.default.svc.cluster.local

# Service通过DNS通信
kubectl exec -it pod1 -- curl http://my-service.default.svc.cluster.local
```

### DNS记录
```
Service DNS记录：
<service-name>.<namespace>.svc.cluster.local

Pod DNS记录：
<pod-ip-address>.<namespace>.pod.cluster.local

Headless Service DNS记录：
<pod-name>.<service-name>.<namespace>.svc.cluster.local
```

## 🔧 网络策略

### NetworkPolicy示例
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

### 允许特定流量
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-nginx
spec:
  podSelector:
    matchLabels:
      app: nginx
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

## 🔧 网络故障排查

### 检查网络连接
```bash
# 检查Pod IP
kubectl get pod -o wide

# 检查Pod网络
kubectl exec -it pod1 -- ip addr

# 检查路由
kubectl exec -it pod1 -- ip route

# 检查DNS
kubectl exec -it pod1 -- nslookup kubernetes.default.svc.cluster.local
```

### 检查CNI插件
```bash
# 查看CNI配置
cat /etc/cni/net.d/*.conf

# 查看CNI插件
ls -la /opt/cni/bin/

# 查看网络接口
ip addr show
```

### 检查网络策略
```bash
# 查看NetworkPolicy
kubectl get networkpolicy

# 查看NetworkPolicy详情
kubectl describe networkpolicy <policy-name>
```

## 🔗 相关概念

### 网络相关
- [[Service概念]]：服务发现和负载均衡
- [[NetworkPolicy]]：网络策略
- [[Ingress概念]]：HTTP/HTTPS路由

### 网络插件
- **CNI**：容器网络接口
- **CoreDNS**：DNS服务
- **Kube-proxy**：网络代理

## 🚀 最佳实践

### 1. 选择合适的CNI插件
- 根据需求选择CNI插件
- 考虑性能、功能、易用性
- 评估社区支持

### 2. 配置网络策略
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

### 3. 配置DNS
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        errors
        health
        kubernetes cluster.local in-addr.arpa ip6.arpa {
            pods insecure
            fallthrough in-addr.arpa ip6.arpa
            ttl 30
        }
        forward . /etc/resolv.conf
        cache 30
        loop
        reload
        loadbalance
    }
```

### 4. 监控网络性能
- 监控网络延迟
- 监控网络吞吐量
- 监控网络错误率

### 5. 优化网络配置
- 合理配置Pod CIDR
- 合理配置Service CIDR
- 优化网络插件配置

### 6. 定期检查网络
- 检查Pod间网络连接
- 检查Service访问
- 检查DNS解析

## 💡 架构思考

Kubernetes网络模型体现了云原生网络的最佳实践：
- **扁平网络**：简化网络架构
- **服务发现**：通过DNS实现服务发现
- **网络策略**：精细的网络控制

从架构师视角：
- 理解Kubernetes网络模型有助于设计高可用网络架构
- 合理的网络配置是构建稳定集群的基础
- 选择合适的CNI插件是优化网络性能的关键

## 🔍 深入学习

### 相关文档
- [Kubernetes官方网络文档](https://kubernetes.io/docs/concepts/cluster-administration/networking/)
- [[Kubernetes/网络/Service网络]]：Service网络
- [[Kubernetes/网络/NetworkPolicy]]：网络策略

### 实践案例
- [[生产环境K8s部署/网络配置]]
- [[性能优化/网络优化]]
- [[故障排查与恢复/网络问题]]
