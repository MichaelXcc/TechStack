---
type: concept
tags: [kubernetes, secret, 核心概念]
title: Secret概念
status: 已掌握
priority: 高
related: ["Pod", "ConfigMap", "Deployment"]
---

# Secret概念

## 📋 定义

Secret是Kubernetes中用于存储敏感数据的对象，如密码、OAuth令牌、SSH密钥等。Secret数据以Base64编码存储，并通过etcd加密保护。

## 🎯 核心特性

### 1. 敏感数据保护
- **Base64编码**：数据以Base64编码存储（非加密）
- **etcd加密**：启用EncryptionConfig后etcd数据加密
- **访问控制**：通过RBAC控制Secret访问权限

### 2. 多种注入方式
- **环境变量**：将Secret注入为容器环境变量
- **Volume挂载**：挂载为Volume中的文件
- **镜像拉取**：用于私有镜像仓库认证

### 3. 自动管理
- **ServiceAccount**：自动创建API访问Secret
- **TLS证书**：自动管理TLS证书Secret
- **Docker Registry**：自动管理镜像仓库认证

## 📝 YAML示例

### Opaque Secret
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  username: YWRtaW4=  # admin
  password: MWYyZDFlMmU2N2Rm  # 1f2d1e2e67df
```

### 从命令行创建
```bash
kubectl create secret generic my-secret \
  --from-literal=username=admin \
  --from-literal=password=1f2d1e2e67df

kubectl create secret generic my-secret \
  --from-file=./username.txt \
  --from-file=./password.txt
```

### TLS Secret
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret
type: kubernetes.io/tls
data:
  tls.crt: LS0tLS1CRUdJTi...
  tls.key: LS0tLS1CRUdJTi...
```

### Docker Registry Secret
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: docker-registry-secret
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: eyJhdXRocyI6eyJyZWdpc3RyeS5leGFtcGxlLmNvbSI6eyJ1c2VybmFtZSI6ImFkbWluIiwicGFzc3dvcmQiOiIxZjJkMWUyZTY3ZGYiLCJhdXRoIjoiYWRtaW46MWYyZDFlMmU2N2RmIn19fQ==
```

## 🔧 Secret类型

### Opaque
- **用途**：通用密钥数据
- **内容**：任意键值对
- **场景**：密码、API密钥等

### kubernetes.io/service-account-token
- **用途**：ServiceAccount访问API的令牌
- **内容**：自动生成，包含token、ca.crt等
- **场景**：Pod访问Kubernetes API

### kubernetes.io/dockercfg
- **用途**：Docker Registry认证
- **内容**：~/.dockercfg文件内容
- **场景**：私有镜像仓库访问

### kubernetes.io/dockerconfigjson
- **用途**：Docker Registry认证（新版）
- **内容**：~/.docker/config.json文件内容
- **场景**：私有镜像仓库访问

### kubernetes.io/tls
- **用途**：TLS证书和私钥
- **内容**：tls.crt和tls.key
- **场景**：HTTPS服务、Ingress配置

### bootstrap.kubernetes.io/token
- **用途**：节点引导令牌
- **内容**：用于节点加入集群的令牌
- **场景**：集群初始化

## 🔧 使用方式

### 1. 环境变量注入
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: my-image
    env:
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: my-secret
          key: username
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: my-secret
          key: password
```

### 2. Volume挂载
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
    - name: secret-volume
      mountPath: /etc/secrets
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: my-secret
      optional: false
```

### 3. 镜像拉取凭证
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: registry.example.com/my-image:latest
  imagePullSecrets:
  - name: docker-registry-secret
```

### 4. 所有键作为环境变量
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: my-image
    envFrom:
    - secretRef:
        name: my-secret
```

## 📊 安全最佳实践

### 1. 启用etcd加密
```yaml
# /etc/kubernetes/manifests/kube-apiserver.yaml
apiVersion: v1
kind: Pod
metadata:
  name: kube-apiserver
spec:
  containers:
  - name: kube-apiserver
    command:
    - kube-apiserver
    - --encryption-provider-config=/etc/kubernetes/encryption-config.yaml
```

### 2. 最小权限原则
- 使用RBAC限制Secret访问
- 只授予必要的权限
- 定期审计Secret访问日志

### 3. Secret轮换
- 定期轮换敏感数据
- 使用自动化工具管理Secret生命周期
- 监控Secret使用情况

### 4. 避免环境变量
- 环境变量可能被日志记录
- 优先使用Volume挂载方式
- 确保日志不包含敏感信息

### 5. 使用外部Secret管理
- 集成Vault、AWS Secrets Manager等
- 使用External Secrets Operator
- 实现Secret的集中管理

## 🔗 相关概念

### 配合使用
- [[Pod概念]]：Secret注入到Pod中
- [[Deployment]]：管理使用Secret的Pod
- [[ConfigMap]]：存储非敏感配置
- [[ServiceAccount]]：自动创建API访问Secret

### 安全相关
- **RBAC**：控制Secret访问权限
- **NetworkPolicy**：限制Pod间网络访问
- **PodSecurityPolicy**：Pod安全策略

## 🚀 最佳实践

### 1. 命名规范
```yaml
metadata:
  name: <app-name>-<component>-<type>-secret
  # 例如: myapp-database-credentials-secret
  labels:
    app: myapp
    component: database
    type: credentials
```

### 2. 使用不可变Secret
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: immutable-secret
immutable: true
type: Opaque
data:
  key: value
```

### 3. 分离敏感和非敏感配置
- 密码、密钥使用Secret
- 配置参数使用ConfigMap
- 避免在ConfigMap中包含敏感信息

### 4. 审计和监控
- 启用审计日志记录Secret访问
- 监控异常的Secret访问行为
- 定期审查Secret使用情况

### 5. 备份和恢复
- 定期备份etcd数据
- 测试Secret恢复流程
- 确保备份加密存储

## 💡 架构思考

Secret是Kubernetes安全模型的核心：
- **数据分离**：敏感数据与代码、配置分离
- **访问控制**：通过RBAC实现细粒度权限管理
- **可追溯性**：审计日志记录所有Secret访问

从架构师视角：
- Secret是实现安全合规的关键组件
- 理解Secret的安全机制有助于设计安全的应用架构
- 结合外部Secret管理系统实现企业级安全

## 🔍 深入学习

### 相关文档
- [Kubernetes官方Secret文档](https://kubernetes.io/docs/concepts/configuration/secret/)
- [[Kubernetes/安全/Secret管理]]：Secret安全管理
- [[Kubernetes/安全/RBAC]]：基于角色的访问控制

### 实践案例
- [[生产环境K8s部署/Secret加密]]
- [[安全合规/密钥管理最佳实践]]
- [[故障排查与恢复/Secret访问问题]]
