---
type: concept
tags: [kubernetes, etcd, 控制平面]
title: etcd概念
author: 云原生技术架构师
status: 已掌握
priority: 高
related: ["API Server", "Controller Manager", "Scheduler"]
---

# etcd概念

## 📋 定义

etcd是Kubernetes集群的键值存储数据库，用于存储集群的所有状态数据，是Kubernetes控制平面的数据存储后端。

## 🎯 核心特性

### 1. 分布式存储
- **一致性**：保证数据强一致性
- **可用性**：高可用集群部署
- **分区容错**：容忍节点故障

### 2. 键值存储
- **简单API**：提供简单的键值对API
- **事务支持**：支持多键事务操作
- **版本控制**：支持数据版本管理

### 3. Watch机制
- **事件通知**：实时推送数据变化
- **历史版本**：支持历史数据查询
- **租约机制**：支持键的租约管理

## 📝 etcd架构

### etcd集群架构
```
etcd Cluster
├── etcd-1 (Leader)
│   ├── Raft Log
│   ├── Snapshot
│   └── WAL
├── etcd-2 (Follower)
│   ├── Raft Log
│   ├── Snapshot
│   └── WAL
└── etcd-3 (Follower)
    ├── Raft Log
    ├── Snapshot
    └── WAL
```

### Raft协议
```
1. Leader接收写请求
   ↓
2. Leader追加到本地日志
   ↓
3. Leader复制到Follower
   ↓
4. Follower确认接收
   ↓
5. Leader提交写入
   ↓
6. Leader通知Follower提交
```

## 🔧 etcd操作

### 基本操作
```bash
# 设置键值
etcdctl put /key value

# 获取键值
etcdctl get /key

# 删除键
etcdctl del /key

# 列出所有键
etcdctl get / --prefix
```

### 事务操作
```bash
# 事务写入
etcdctl txn -i
compares:
value("key1") = "value1"
success requests (get, put, del):
get key1
put key1 newvalue1
failure requests (get, put, del):
get key1
```

### Watch操作
```bash
# 监听键变化
etcdctl watch /key

# 监听前缀
etcdctl watch / --prefix

# 监听历史版本
etcdctl watch /key --rev=1
```

### 租约操作
```bash
# 创建租约
etcdctl lease grant 60

# 绑定租约
etcdctl put /key value --lease=1234567890

# 续租
etcdctl lease keep-alive 1234567890

# 撤销租约
etcdctl lease revoke 1234567890
```

## 🔧 etcd配置

### 集群配置
```yaml
# etcd.yaml
name: etcd-1
data-dir: /var/lib/etcd
listen-client-urls: https://192.168.1.100:2379
listen-peer-urls: https://192.168.1.100:2380
advertise-client-urls: https://192.168.1.100:2379
initial-advertise-peer-urls: https://192.168.1.100:2380
initial-cluster: etcd-1=https://192.168.1.100:2380,etcd-2=https://192.168.1.101:2380,etcd-3=https://192.168.1.102:2380
initial-cluster-state: new
initial-cluster-token: etcd-cluster
client-transport-security:
  cert-file: /etc/etcd/ssl/server.crt
  key-file: /etc/etcd/ssl/server.key
  client-cert-auth: true
  trusted-ca-file: /etc/etcd/ssl/ca.crt
peer-transport-security:
  cert-file: /etc/etcd/ssl/peer.crt
  key-file: /etc/etcd/ssl/peer.key
  peer-client-cert-auth: true
  trusted-ca-file: /etc/etcd/ssl/ca.crt
```

### 性能调优
```yaml
# etcd.yaml
snapshot-count: 10000
heartbeat-interval: 100
election-timeout: 1000
max-snapshots: 5
max-wals: 5
quota-backend-bytes: 8589934592
auto-compaction-mode: periodic
auto-compaction-retention: 5h
```

### 安全配置
```yaml
# etcd.yaml
auth:
  enabled: true
  auth-token-ttl: 300
auth-token-ttl: 300
```

## 🔧 Kubernetes集成

### API Server配置
```yaml
# kube-apiserver.yaml
--etcd-servers=https://192.168.1.100:2379,https://192.168.1.101:2379,https://192.168.1.102:2379
--etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
--etcd-certfile=/etc/kubernetes/pki/etcd/server.crt
--etcd-keyfile=/etc/kubernetes/pki/etcd/server.key
```

### 数据加密
```yaml
# encryption-config.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
    - secrets
    providers:
    - aescbc:
        keys:
        - name: key1
          secret: <base64-encoded-secret>
    - identity: {}
```

## 🔧 备份和恢复

### 备份etcd
```bash
# 快照备份
ETCDCTL_API=3 etcdctl snapshot save snapshot.db \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# 验证快照
ETCDCTL_API=3 etcdctl snapshot status snapshot.db \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

### 恢复etcd
```bash
# 停止etcd服务
systemctl stop etcd

# 恢复快照
ETCDCTL_API=3 etcdctl snapshot restore snapshot.db \
  --name etcd-1 \
  --data-dir /var/lib/etcd \
  --initial-cluster etcd-1=https://192.168.1.100:2380,etcd-2=https://192.168.1.101:2380,etcd-3=https://192.168.1.102:2380 \
  --initial-cluster-token etcd-cluster \
  --initial-advertise-peer-urls https://192.168.1.100:2380 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# 启动etcd服务
systemctl start etcd
```

## 🔧 监控和告警

### Prometheus监控
```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'etcd'
    scheme: https
    tls_config:
      ca_file: /etc/etcd/ssl/ca.crt
      cert_file: /etc/etcd/ssl/server.crt
      key_file: /etc/etcd/ssl/server.key
    static_configs:
    - targets: ['192.168.1.100:2379', '192.168.1.101:2379', '192.168.1.102:2379']
```

### 关键指标
```
etcd_server_has_leader
etcd_disk_wal_fsync_duration_seconds
etcd_mvcc_db_total_size_in_bytes
etcd_network_client_grpc_received_bytes_total
etcd_request_duration_seconds
```

### 告警规则
```yaml
# alerting.yml
groups:
- name: etcd
  rules:
  - alert: EtcdNoLeader
    expr: etcd_server_has_leader == 0
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "etcd cluster has no leader"
      
  - alert: EtcdHighDiskLatency
    expr: etcd_disk_wal_fsync_duration_seconds > 0.1
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "etcd disk write latency is high"
```

## 🔗 相关概念

### 控制平面组件
- [[API Server概念]]：与etcd通信
- [[Controller Manager概念]]：通过API Server访问etcd
- [[Scheduler概念]]：通过API Server访问etcd

### 数据存储
- **键值存储**：etcd的存储模型
- **Raft协议**：etcd的一致性协议
- **WAL**：预写日志

## 🚀 最佳实践

### 1. 使用奇数节点
- 推荐使用3、5、7个节点
- 保证多数派选举

### 2. 分离存储
```yaml
# etcd.yaml
data-dir: /var/lib/etcd
wal-dir: /var/lib/etcd/wal
```

### 3. 定期备份
```bash
# 定时备份脚本
#!/bin/bash
DATE=$(date +%Y%m%d-%H%M%S)
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-snapshot-$DATE.db \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

### 4. 启用数据加密
```yaml
# encryption-config.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
    - secrets
    providers:
    - aescbc:
        keys:
        - name: key1
          secret: <base64-encoded-secret>
```

### 5. 监控etcd性能
- 监控磁盘延迟
- 监控网络延迟
- 监控Raft日志

### 6. 定期压缩
```bash
# 手动压缩
ETCDCTL_API=3 etcdctl compact $(ETCDCTL_API=3 etcdctl endpoint status --write-out="json" | grep -o '"revision":[0-9]*' | grep -o '[0-9]*')

# 自动压缩
ETCDCTL_API=3 etcdctl defrag
```

## 💡 架构思考

etcd是Kubernetes的数据基石：
- **数据一致性**：通过Raft协议保证数据一致性
- **高可用性**：分布式部署保证高可用
- **Watch机制**：实时通知数据变化

从架构师视角：
- etcd设计体现了分布式系统的最佳实践
- 理解etcd的工作原理有助于优化集群性能
- 合理的etcd配置是构建高可用集群的基础

## 🔍 深入学习

### 相关文档
- [etcd官方文档](https://etcd.io/docs/)
- [Kubernetes官方etcd文档](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/)
- [[Kubernetes/控制平面/API Server]]：API Server

### 实践案例
- [[生产环境K8s部署/etcd配置]]
- [[生产环境K8s部署/etcd备份恢复]]
- [[故障排查与恢复/etcd问题]]
