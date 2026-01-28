---
type: reference
tags: [架构层, AIInfra, Ray, 命令行, 运维]
title: Ray 启动命令参数详解
---

# Ray 启动命令参数详解

本文档详细介绍 Ray 集群启动相关的命令行参数，帮助你理解如何正确配置和管理 Ray 集群。

## 1. 命令概览

Ray 提供以下核心命令：

| 命令 | 用途 |
|------|------|
| `ray start` | 启动 Ray 节点（Head 或 Worker） |
| `ray stop` | 停止本地 Ray 节点 |
| `ray status` | 查看集群状态 |
| `ray up` | 基于配置文件启动集群（云端） |
| `ray down` | 关闭整个集群 |

---

## 2. `ray start` 命令详解

### 2.1 基本语法

```bash
ray start [OPTIONS]
```

### 2.2 节点类型参数

#### `--head`
**启动 Head 节点**（集群的主节点）。

```bash
ray start --head
```

- Head 节点运行 GCS (Global Control Store)
- 一个集群有且仅有一个 Head 节点
- Worker 节点通过 Head 地址加入集群

#### `--address=<ip:port>`
**加入现有集群**，连接到 Head 节点。

```bash
ray start --address="192.168.1.100:6379"
```

- 格式：`<Head节点IP>:<GCS端口>`
- 默认 GCS 端口为 `6379`
- 也可使用 `ray://` 前缀：`--address="ray://192.168.1.100:10001"`

---

### 2.3 端口配置参数

> [!IMPORTANT]
> 端口配置在多节点集群和容器化部署中非常关键，必须确保端口不冲突且防火墙放行。

#### `--port=<port>`
**GCS (Global Control Store) 端口**，默认 `6379`。

```bash
ray start --head --port=6380
```

- 用于 Head 节点的 GCS 服务
- Worker 节点通过此端口注册到集群

#### `--node-manager-port=<port>`
**Node Manager (Raylet) 端口**，默认随机分配。

```bash
ray start --head --node-manager-port=12345
```

- 每个节点的 Raylet 进程监听此端口
- 用于节点间的心跳和任务分发

#### `--object-manager-port=<port>`
**Object Manager 端口**，默认随机分配。

```bash
ray start --head --object-manager-port=12346
```

- 用于节点间的对象传输
- 大规模集群建议固定端口以便防火墙配置

#### `--ray-client-server-port=<port>`
**Ray Client Server 端口**，默认 `10001`。

```bash
ray start --head --ray-client-server-port=10001
```

- 用于 `ray.init("ray://<address>:10001")` 远程连接
- 生产环境建议开启 Ray Client 以支持远程提交任务

#### `--dashboard-port=<port>`
**Ray Dashboard Web 端口**，默认 `8265`。

```bash
ray start --head --dashboard-port=8265
```

- 访问 `http://<head-ip>:8265` 查看集群状态
- 包含任务监控、Actor 管理、日志查看等功能

#### `--min-worker-port / --max-worker-port`
**Worker 进程端口范围**。

```bash
ray start --head --min-worker-port=20000 --max-worker-port=25000
```

- Worker 进程会在此范围内分配端口
- 容器化部署时需要暴露此端口范围

---

### 2.4 资源配置参数

#### `--num-cpus=<N>`
**声明 CPU 核心数**。

```bash
ray start --head --num-cpus=32
```

- 不指定时自动检测系统 CPU 数
- 可以 over-provision 或 under-provision

#### `--num-gpus=<N>`
**声明 GPU 数量**。

```bash
ray start --head --num-gpus=8
```

- 不指定时自动检测 CUDA 设备数
- 对于虚拟化环境（如 MIG）需要手动指定

#### `--resources=<json>`
**声明自定义资源**。

```bash
ray start --head --resources='{"TPU": 4, "special_hardware": 2}'
```

- 使用 JSON 格式
- 可在 `@ray.remote(resources={"TPU": 1})` 中引用

#### `--labels=<json>`
**节点标签**（用于调度）。

```bash
ray start --head --labels='{"node_type": "gpu", "zone": "us-west-1a"}'
```

- 可结合 Placement Group 进行 topology-aware 调度

---

### 2.5 内存配置参数

#### `--memory=<bytes>`
**可用于 Ray 任务的内存**。

```bash
ray start --head --memory=100000000000  # 100GB
```

- 单位：字节
- 默认为系统可用内存的一定比例

#### `--object-store-memory=<bytes>`
**Plasma 对象存储内存**。

```bash
ray start --head --object-store-memory=50000000000  # 50GB
```

- 共享内存区域，用于存储 `ray.put()` 的对象
- 默认为系统内存的 30%
- 对于数据密集型任务，建议增大此值

#### `--plasma-directory=<path>`
**Plasma 存储目录**（用于内存映射）。

```bash
ray start --head --plasma-directory=/dev/shm
```

- 默认使用 `/dev/shm`（tmpfs）
- 可指定其他大容量目录

---

### 2.6 系统配置参数

#### `--include-dashboard`
**启用 Dashboard**（默认启用）。

```bash
ray start --head --include-dashboard=true
```

#### `--dashboard-host=<ip>`
**Dashboard 绑定地址**。

```bash
ray start --head --dashboard-host=0.0.0.0
```

- 默认绑定 `127.0.0.1`
- 远程访问需设置为 `0.0.0.0`

#### `--temp-dir=<path>`
**临时文件目录**。

```bash
ray start --head --temp-dir=/mnt/ray_tmp
```

- 存储日志、socket 文件等
- 默认：`/tmp/ray`

#### `--log-style=<style>`
**日志格式**：`auto`, `pretty`, `record`。

```bash
ray start --head --log-style=pretty
```

#### `--log-color=<bool>`
**日志颜色**。

```bash
ray start --head --log-color=true
```

---

### 2.7 高级配置参数

#### `--system-config=<json>`
**系统级配置**（高级调优）。

```bash
ray start --head --system-config='{"num_heartbeats_timeout": 30}'
```

常用配置项：

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| `num_heartbeats_timeout` | 30 | 心跳超时次数 |
| `raylet_heartbeat_period_milliseconds` | 1000 | 心跳间隔 (ms) |
| `object_timeout_milliseconds` | 0 | 对象获取超时 |
| `max_direct_call_object_size` | 100KB | 直接返回对象大小阈值 |

#### `--storage=<uri>`
**持久化存储路径**（用于 Ray Data 和 Ray Train）。

```bash
ray start --head --storage="s3://my-bucket/ray-storage"
```

- 支持 `s3://`、`gs://`、`hdfs://`、本地路径
- 用于 checkpoint、数据集缓存等

#### `--autoscaling-config=<path>`
**自动扩缩容配置**（用于 `ray up`）。

```bash
ray start --head --autoscaling-config=cluster.yaml
```

---

## 3. 完整启动示例

### 3.1 单机模式

```bash
# 使用默认配置启动（开发测试）
ray start --head

# 或在 Python 中
import ray
ray.init()  # 自动启动单节点
```

### 3.2 多节点集群

**Head 节点（192.168.1.100）：**
```bash
ray start --head \
    --port=6379 \
    --dashboard-host=0.0.0.0 \
    --dashboard-port=8265 \
    --num-cpus=64 \
    --num-gpus=8 \
    --object-store-memory=100000000000 \
    --resources='{"node_type": "head"}'
```

**Worker 节点（192.168.1.101）：**
```bash
ray start \
    --address=192.168.1.100:6379 \
    --num-cpus=64 \
    --num-gpus=8 \
    --object-store-memory=100000000000 \
    --resources='{"node_type": "worker"}'
```

### 3.3 容器化部署

```bash
# 暴露必要端口
ray start --head \
    --port=6379 \
    --ray-client-server-port=10001 \
    --dashboard-port=8265 \
    --node-manager-port=12345 \
    --object-manager-port=12346 \
    --min-worker-port=20000 \
    --max-worker-port=25000 \
    --dashboard-host=0.0.0.0 \
    --block  # 前台运行，容器保活
```

---

## 4. `ray stop` 命令

```bash
# 停止本地 Ray 进程
ray stop

# 强制停止（杀死所有 Ray 进程）
ray stop --force

# 仅停止 Worker，保留 Head
ray stop --grace-period=30
```

---

## 5. `ray status` 命令

```bash
# 查看集群状态
ray status
```

输出示例：
```
======== Cluster Resources ========
Usage stats:
  CPU:     32.0/128.0 (25.0%)
  Memory:  50.00/500.00 GiB (10.0%)
  GPU:     4.0/8.0 (50.0%)

======== Node Statuses ========
Active:
  192.168.1.100: CPU=64, GPU=4, memory=250GB
  192.168.1.101: CPU=64, GPU=4, memory=250GB
```

---

## 6. 环境变量参考

除了命令行参数，Ray 还支持通过环境变量配置：

| 环境变量 | 说明 |
|----------|------|
| `RAY_ADDRESS` | 等同于 `--address` |
| `RAY_TMPDIR` | 等同于 `--temp-dir` |
| `RAY_DISABLE_MEMORY_MONITOR` | 禁用内存监控 |
| `RAY_memory_usage_threshold` | 内存使用阈值（0-1） |
| `RAY_OBJECT_STORE_MEMORY` | 对象存储大小 |
| `RAY_ENABLE_RECORD_ACTOR_TASK_LOGGING` | 开启 Actor 任务日志 |

---

## 7. 常见问题

### Q1: 端口被占用怎么办？

```bash
# 检查端口占用
lsof -i :6379

# 使用其他端口
ray start --head --port=6380
```

### Q2: Worker 无法连接 Head？

检查：
1. Head 节点 IP 和端口是否正确
2. 防火墙是否放行 GCS 端口
3. 使用 `ray status` 确认 Head 正常运行

### Q3: 内存不足？

```bash
# 增加对象存储
ray start --head --object-store-memory=200000000000

# 或设置溢出目录
ray start --head --system-config='{"object_spilling_config": "{\"type\": \"filesystem\", \"params\": {\"directory_path\": \"/mnt/spill\"}}"}'
```

> [!TIP]
> 生产环境建议使用 [KubeRay](https://ray-project.github.io/kuberay/) 在 Kubernetes 上部署 Ray 集群，可获得自动扩缩容、故障恢复等能力。
