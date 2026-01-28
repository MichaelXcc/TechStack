---
type: framework
tags: [架构层, AIInfra, 推理服务, vLLM, LLM]
title: vLLM推理服务
author: AI Agent
status: 学习中
priority: 高
related: ["推理服务", "大模型部署", "Kubernetes", "GPU调度"]
---

# vLLM 推理服务

## 📋 概述

vLLM 是由 UC Berkeley 开发的高性能 LLM 推理引擎，通过创新的 **PagedAttention** 技术，实现了业界领先的吞吐量和显存效率。它是目前部署 7B-70B 规模模型的首选方案。

## 🎯 核心特性

### 1. PagedAttention

借鉴操作系统虚拟内存的分页思想，将 KV Cache 分割为固定大小的 Block：

```
传统方式：连续显存分配
┌─────────────────────────────────────┐
│ Request 1 KV Cache (预分配最大长度)  │  ← 大量浪费
└─────────────────────────────────────┘

PagedAttention：分页显存分配
┌───┬───┬───┐   ┌───┬───┐   ┌───┐
│ 1 │ 1 │ 1 │   │ 2 │ 2 │   │ 3 │  ← 按需分配
└───┴───┴───┘   └───┴───┘   └───┘
```

**优势：**
- 显存利用率提升 2-4x
- 支持更大的并发请求数
- 消除显存碎片

### 2. Continuous Batching

动态批处理，无需等待整个 Batch 完成：

```
传统 Static Batching:
R1: ████████████░░░░░░░░  (等待)
R2: ████████████████████
R3: ████████████████░░░░  (等待)
                    ↑ Batch 完成

Continuous Batching:
R1: ████████████ → 返回
R4: ----████████████████  (插入)
R2: ████████████████████ → 返回
R3: ████████████████ → 返回
```

### 3. 模型支持

- **HuggingFace 模型**: 开箱即用
- **量化模型**: GPTQ, AWQ, SqueezeLLM
- **LoRA 适配器**: 动态加载多个 LoRA
- **多模态**: LLaVA, Qwen-VL

---

## 🚀 快速开始

### 安装

```bash
# 推荐使用 pip 安装
pip install vllm

# 或从源码安装（获取最新特性）
git clone https://github.com/vllm-project/vllm.git
cd vllm
pip install -e .
```

### 离线推理

```python
from vllm import LLM, SamplingParams

# 加载模型
llm = LLM(
    model="meta-llama/Llama-2-7b-chat-hf",
    tensor_parallel_size=1,  # GPU 数量
    gpu_memory_utilization=0.9
)

# 设置采样参数
sampling_params = SamplingParams(
    temperature=0.7,
    top_p=0.95,
    max_tokens=512
)

# 批量推理
prompts = [
    "Hello, my name is",
    "The capital of France is",
    "The meaning of life is"
]

outputs = llm.generate(prompts, sampling_params)

for output in outputs:
    print(f"Prompt: {output.prompt}")
    print(f"Output: {output.outputs[0].text}")
```

### 在线服务

```bash
# 启动 OpenAI 兼容 API 服务
python -m vllm.entrypoints.openai.api_server \
    --model meta-llama/Llama-2-7b-chat-hf \
    --tensor-parallel-size 1 \
    --port 8000
```

```python
# 客户端调用
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:8000/v1",
    api_key="EMPTY"
)

response = client.chat.completions.create(
    model="meta-llama/Llama-2-7b-chat-hf",
    messages=[
        {"role": "user", "content": "Hello!"}
    ]
)
print(response.choices[0].message.content)
```

---

## 📊 Kubernetes 部署

### Deployment 配置

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vllm-inference
  namespace: ai-inference
spec:
  replicas: 1
  selector:
    matchLabels:
      app: vllm-inference
  template:
    metadata:
      labels:
        app: vllm-inference
    spec:
      containers:
      - name: vllm
        image: vllm/vllm-openai:latest
        args:
        - --model
        - meta-llama/Llama-2-7b-chat-hf
        - --tensor-parallel-size
        - "1"
        - --gpu-memory-utilization
        - "0.9"
        ports:
        - containerPort: 8000
        resources:
          limits:
            nvidia.com/gpu: 1
            memory: 32Gi
          requests:
            nvidia.com/gpu: 1
            memory: 16Gi
        env:
        - name: HF_TOKEN
          valueFrom:
            secretKeyRef:
              name: hf-secret
              key: token
        volumeMounts:
        - name: model-cache
          mountPath: /root/.cache/huggingface
      volumes:
      - name: model-cache
        persistentVolumeClaim:
          claimName: model-cache-pvc
```

### Service 配置

```yaml
apiVersion: v1
kind: Service
metadata:
  name: vllm-inference
  namespace: ai-inference
spec:
  selector:
    app: vllm-inference
  ports:
  - port: 80
    targetPort: 8000
  type: ClusterIP
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: vllm-ingress
  namespace: ai-inference
spec:
  rules:
  - host: llm-api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: vllm-inference
            port:
              number: 80
```

---

## ⚡ 性能调优

### 关键参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `--gpu-memory-utilization` | 0.9 | 显存预分配比例 |
| `--max-num-seqs` | 256 | 最大并发序列数 |
| `--max-num-batched-tokens` | 根据模型 | 单批次最大 Token 数 |
| `--block-size` | 16 | PagedAttention Block 大小 |
| `--swap-space` | 4 | Swap 空间 (GB) |

### 多 GPU 部署

```bash
# 张量并行 (单节点多卡)
python -m vllm.entrypoints.openai.api_server \
    --model meta-llama/Llama-2-70b-chat-hf \
    --tensor-parallel-size 4

# 流水线并行 (多节点)
# 需要配合 Ray 使用
ray start --head
python -m vllm.entrypoints.openai.api_server \
    --model meta-llama/Llama-2-70b-chat-hf \
    --tensor-parallel-size 2 \
    --pipeline-parallel-size 2
```

### 量化部署

```python
from vllm import LLM

# AWQ 量化 (推荐)
llm = LLM(
    model="TheBloke/Llama-2-7B-Chat-AWQ",
    quantization="awq"
)

# GPTQ 量化
llm = LLM(
    model="TheBloke/Llama-2-7B-Chat-GPTQ",
    quantization="gptq"
)
```

---

## 📈 性能对比

| 模型 | 框架 | 吞吐量 (tokens/s) | 显存占用 |
|------|------|-------------------|----------|
| LLaMA-7B | HuggingFace | ~50 | 14GB |
| LLaMA-7B | vLLM | ~800 | 14GB |
| LLaMA-7B | vLLM + AWQ | ~1200 | 8GB |
| LLaMA-70B | vLLM (4xA100) | ~600 | 4x40GB |

---

## 🔗 相关链接

### 内部文档
- [[Megatron-LM vs. vLLM vs. SGLang|推理框架对比]]
- [[Megatron推理服务]]
- [[../GPU性能优化/MOC - GPU性能优化|GPU 性能优化]]

### 外部资源
- [vLLM GitHub](https://github.com/vllm-project/vllm)
- [vLLM 官方文档](https://docs.vllm.ai/)
- [PagedAttention 论文](https://arxiv.org/abs/2309.06180)

---

## 💬 思考问题

1. PagedAttention 如何解决显存碎片化问题？
2. 在什么场景下应该选择 vLLM 而非 TensorRT-LLM？
3. 如何设计 vLLM 的自动扩缩容策略？
