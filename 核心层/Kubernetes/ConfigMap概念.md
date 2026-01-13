---
type: concept
tags: [kubernetes, configmap, 核心概念]
title: ConfigMap概念
author: AI Infra / LLM Ops 基础设施
status: 已掌握
priority: 高
related: ["Pod", "Secret", "Deployment"]
---

# ConfigMap概念

## 📋 定义

ConfigMap是Kubernetes中用于存储非敏感配置数据的API对象，它将配置数据与容器镜像解耦，实现配置的集中管理和动态更新。

## 🎯 核心特性

### 1. 配置解耦
- **镜像无关**：配置不包含在镜像中，实现一次构建，多环境部署
- **环境隔离**：不同环境使用不同ConfigMap
- **版本管理**：配置变更可追溯、可回滚

### 2. 多种注入方式
- **环境变量**：将配置注入为容器环境变量
- **命令行参数**：在启动命令中引用配置
- **配置文件**：挂载为Volume中的文件

### 3. 动态更新
- **热更新**：挂载为Volume的ConfigMap更新后自动生效
- **无需重建**：Pod无需重启即可获取最新配置
- **原子性**：更新操作是原子的

## 📝 YAML示例

### 字面值方式
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database.url: "mysql://localhost:3306/mydb"
  cache.enabled: "true"
  log.level: "info"
```

### 文件方式
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  nginx.conf: |
    server {
      listen 80;
      server_name example.com;
      
      location / {
        root /usr/share/nginx/html;
        index index.html;
      }
    }
```

### 从文件创建
```bash
kubectl create configmap my-config \
  --from-file=config.properties \
  --from-file=nginx.conf
```

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
    - name: DATABASE_URL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database.url
    - name: LOG_LEVEL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: log.level
```

### 2. 命令行参数
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: my-image
    command: ["/app/start.sh"]
    args: ["--log-level", "$(LOG_LEVEL)"]
    env:
    - name: LOG_LEVEL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: log.level
```

### 3. Volume挂载
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
    - name: config-volume
      mountPath: /etc/nginx/conf.d
  volumes:
  - name: config-volume
    configMap:
      name: nginx-config
      items:
      - key: nginx.conf
        path: default.conf
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
    - configMapRef:
        name: app-config
```

## 📊 不可变ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: immutable-config
immutable: true
data:
  config.yaml: |
    key: value
```

**优势**：
- 提升性能：kubelet不监听不可变ConfigMap
- 安全性：防止意外修改
- 一致性：确保Pod使用相同配置

## 🔗 相关概念

### 配合使用
- [[Pod概念]]：ConfigMap注入到Pod中
- [[Deployment概念]]：管理使用ConfigMap的Pod
- [[Secret概念]]：存储敏感配置信息

### 对比
- **ConfigMap**：非敏感配置，明文存储
- **Secret**：敏感配置，Base64编码存储

## 🚀 最佳实践

### 1. 配置分层
```yaml
# 基础配置
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-base-config
data:
  app.name: "myapp"
  app.version: "1.0.0"

---
# 环境配置
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-prod-config
data:
  database.url: "mysql://prod-db:3306/mydb"
  cache.ttl: "3600"
```

### 2. 命名规范
```yaml
metadata:
  name: <app-name>-<component>-<environment>-config
  # 例如: myapp-frontend-prod-config
  labels:
    app: myapp
    component: frontend
    environment: production
```

### 3. 配置验证
- 在CI/CD流程中验证ConfigMap格式
- 使用Schema验证工具（如Kubeval）
- 在应用启动时验证配置完整性

### 4. 版本管理
- ConfigMap名称包含版本号
- 使用Git管理ConfigMap的YAML文件
- 通过GitOps工具（如ArgoCD）自动同步

### 5. 敏感数据处理
- 密码、密钥等敏感信息使用[[Secret]]
- ConfigMap中不包含任何敏感数据
- 定期审计ConfigMap内容

## 💡 架构思考

ConfigMap体现了配置即代码的理念：
- **配置外部化**：配置与代码分离，提高灵活性
- **环境一致性**：通过Git管理配置，确保环境一致
- **可观测性**：配置变更可追踪、可审计

从架构师视角：
- ConfigMap是实现12-Factor App中"配置"原则的关键
- 理解ConfigMap的注入方式有助于设计灵活的应用架构
- 结合GitOps实现配置的自动化管理

## 🔍 深入学习

### 相关文档
- [Kubernetes官方ConfigMap文档](https://kubernetes.io/docs/concepts/configuration/configmap/)
- [[Kubernetes/最佳实践/配置管理]]：配置管理最佳实践
- [[CI/CD集成/GitOps]]：GitOps配置管理

### 实践案例
- [[生产环境K8s部署/多环境配置]]
- [[微服务架构/配置中心集成]]
- [[故障排查与恢复/ConfigMap更新问题]]
