---
type: guide
tags: [架构层, AIInfra, GPU, 监控, 性能]
title: GPU性能指标与监控
created: 2026-01-22
updated: 2026-01-22
author: AI Agent
status: 学习中
priority: 高
related: ["GPU性能优化", "Prometheus", "Grafana"]
---

# GPU 性能指标与监控

## 📋 概述

在 AI 基础设施中，GPU 是最昂贵的资源。有效的监控系统能够帮助我们：
- 识别性能瓶颈和资源浪费
- 预测故障和维护需求
- 优化集群利用率和成本

## 🎯 核心性能指标

### 1. 算力利用率指标

| 指标 | 全称 | 定义 | 理想值 |
|------|------|------|--------|
| **MFU** | Model FLOPS Utilization | 模型实际算力 / 理论峰值算力 | >50% |
| **HFU** | Hardware FLOPS Utilization | 硬件实际算力 / 理论峰值算力 | >60% |
| **SM Activity** | Streaming Multiprocessor 活跃率 | SM 核心繁忙时间占比 | >80% |

#### MFU 计算公式
```
MFU = (实际 Tokens/s × 模型 FLOPs/Token) / GPU 理论 TFLOPS
```

#### 示例：H100 训练 LLaMA-70B
```
理论峰值: 1979 TFLOPS (BF16)
实际吞吐: 3500 tokens/s
模型计算量: 约 280 TFLOPs/token

MFU = (3500 × 280) / 1979000 ≈ 49.5%
```

### 2. 显存指标

| 指标 | 含义 | 监控要点 |
|------|------|----------|
| **Memory Used** | 已使用显存 | 接近上限时需警惕 OOM |
| **Memory Reserved** | PyTorch 预留显存 | 可能存在碎片化 |
| **Memory Fragmentation** | 碎片率 | 影响大 Tensor 分配 |

### 3. 温度与功耗

| 指标 | 阈值 | 影响 |
|------|------|------|
| **GPU Temp** | >80°C 警告 | 触发降频 |
| **Memory Temp** | >95°C 警告 | HBM 过热 |
| **Power Draw** | >TDP 90% | 可能被限流 |

### 4. 错误计数

| 类型 | 含义 | 处理方式 |
|------|------|----------|
| **ECC Correctable** | 可纠正内存错误 | 累积过多需更换 |
| **ECC Uncorrectable** | 不可纠正错误 | 立即下线检修 |
| **Xid Error** | NVIDIA 驱动错误码 | 根据 Xid 类型处理 |

---

## 🔧 监控工具

### 1. nvidia-smi

最基础的 GPU 监控命令：

```bash
# 实时监控
nvidia-smi -l 1

# 查询特定指标
nvidia-smi --query-gpu=gpu_name,temperature.gpu,utilization.gpu,memory.used,power.draw --format=csv

# 监控进程
nvidia-smi pmon -s um
```

### 2. DCGM (Data Center GPU Manager)

NVIDIA 官方的数据中心 GPU 管理工具：

```bash
# 启动 DCGM 服务
systemctl start nvidia-dcgm

# 查询健康状态
dcgmi health -c

# 运行诊断
dcgmi diag -r 3

# 查看 GPU 拓扑
dcgmi topo --gpuid 0
```

#### DCGM Exporter 部署

```yaml
# dcgm-exporter-daemonset.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: dcgm-exporter
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: dcgm-exporter
  template:
    metadata:
      labels:
        app: dcgm-exporter
    spec:
      containers:
      - name: dcgm-exporter
        image: nvcr.io/nvidia/k8s/dcgm-exporter:3.3.0-3.2.0-ubuntu22.04
        ports:
        - containerPort: 9400
          name: metrics
        securityContext:
          runAsNonRoot: false
          runAsUser: 0
        volumeMounts:
        - name: pod-gpu-resources
          mountPath: /var/lib/kubelet/pod-resources
          readOnly: true
      volumes:
      - name: pod-gpu-resources
        hostPath:
          path: /var/lib/kubelet/pod-resources
```

### 3. Prometheus + Grafana

#### Prometheus 配置

```yaml
# prometheus-config.yaml
scrape_configs:
  - job_name: 'dcgm-exporter'
    kubernetes_sd_configs:
    - role: pod
    relabel_configs:
    - source_labels: [__meta_kubernetes_pod_label_app]
      action: keep
      regex: dcgm-exporter
    - source_labels: [__meta_kubernetes_pod_ip]
      target_label: __address__
      replacement: $1:9400
```

#### 关键 PromQL 查询

```promql
# GPU 利用率
avg(DCGM_FI_DEV_GPU_UTIL) by (gpu, Hostname)

# 显存使用率
DCGM_FI_DEV_FB_USED / DCGM_FI_DEV_FB_TOTAL * 100

# 功耗占比
DCGM_FI_DEV_POWER_USAGE / DCGM_FI_DEV_POWER_LIMIT * 100

# ECC 错误率 (每小时)
increase(DCGM_FI_DEV_ECC_SBE_VOL_TOTAL[1h])

# GPU 温度
DCGM_FI_DEV_GPU_TEMP
```

---

## 📊 Grafana 仪表盘

### 推荐面板

1. **集群概览**
   - 总 GPU 数量与在线率
   - 平均利用率热力图
   - 故障节点列表

2. **节点详情**
   - 8 卡 GPU 利用率/温度/功耗
   - NVLink 带宽利用率
   - 进程资源占用

3. **训练任务**
   - MFU/HFU 趋势
   - Throughput (tokens/s)
   - Loss 曲线

### 告警规则示例

```yaml
# alertmanager-rules.yaml
groups:
- name: gpu-alerts
  rules:
  - alert: GPUHighTemperature
    expr: DCGM_FI_DEV_GPU_TEMP > 83
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "GPU 温度过高"
      description: "{{ $labels.Hostname }} GPU {{ $labels.gpu }} 温度 {{ $value }}°C"

  - alert: GPUMemoryExhausted
    expr: DCGM_FI_DEV_FB_USED / DCGM_FI_DEV_FB_TOTAL > 0.95
    for: 2m
    labels:
      severity: critical
    annotations:
      summary: "GPU 显存即将耗尽"

  - alert: GPUXidError
    expr: increase(DCGM_FI_DEV_XID_ERRORS[5m]) > 0
    labels:
      severity: critical
    annotations:
      summary: "检测到 GPU Xid 错误"
```

---

## 🔗 相关链接

### 内部文档
- [[GPU性能-GEMM测试分析]]：GEMM 测试中的性能分析
- [[GPU性能调优最佳实践]]：性能调优方法
- [[../AI-Infra核心架构/02_高性能计算集群设计|高性能计算集群设计]]

### 外部资源
- [DCGM 官方文档](https://docs.nvidia.com/datacenter/dcgm/)
- [NVIDIA SMI 参考](https://developer.nvidia.com/nvidia-system-management-interface)
- [Grafana DCGM Dashboard](https://grafana.com/grafana/dashboards/12239)

---

## 💬 思考问题

1. 如何设计 GPU 监控的告警策略，避免告警疲劳？
2. MFU 低于预期时，如何定位是计算瓶颈还是通信瓶颈？
3. 如何利用监控数据优化 GPU 调度策略？
