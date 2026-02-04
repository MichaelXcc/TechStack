# DeepSeek H20 8卡推理测试用例

## 概述

本文档定义了 DeepSeek 系列模型（32B、70B、671B）在 NVIDIA H20 8卡机器上使用 vLLM 和 SGLang 推理框架的测试用例。

## 测试环境

| 配置项   | 规格                                           |
| -------- | ---------------------------------------------- |
| GPU      | NVIDIA H20 × 8                                 |
| GPU显存  | 141GB × 8 = 1128GB                             |
| 互联     | NVLink/NVSwitch                                |
| 框架版本 | vLLM latest (≥0.13.0) / SGLang latest (≥0.5.2) |
| CUDA     | 12.4+                                          |
| PyTorch  | 2.4+                                           |

## 模型配置

| 模型             | 参数量 | 推荐TP | 显存需求(FP16) | 显存需求(FP8/INT8) |
| ---------------- | ------ | ------ | -------------- | ------------------ |
| DeepSeek-V3-32B  | 32B    | 2-4    | ~64GB          | ~32GB              |
| DeepSeek-V3-70B  | 70B    | 4-8    | ~140GB         | ~70GB              |
| DeepSeek-V3-671B | 671B   | 8      | ~1.3TB         | ~670GB (FP8可用)   |

---

## 测试用例

### TC-001: vLLM DeepSeek-32B 基准测试

**目的**: 验证 DeepSeek-32B 在 vLLM 框架下的推理性能

**前置条件**:

- H20 8卡机器环境就绪
- vLLM 安装完成
- 模型权重已下载

**测试步骤**:

1. **启动 vLLM 服务**

```bash
python -m vllm.entrypoints.openai.api_server \
    --model deepseek-ai/DeepSeek-V3-32B \
    --tensor-parallel-size 4 \
    --gpu-memory-utilization 0.90 \
    --max-model-len 8192 \
    --trust-remote-code \
    --port 8000
```

2. **运行吞吐量测试**

```bash
python benchmark_throughput.py \
    --backend vllm \
    --model deepseek-ai/DeepSeek-V3-32B \
    --input-len 512 \
    --output-len 512 \
    --num-prompts 100 \
    --tensor-parallel-size 4
```

3. **运行延迟测试**

```bash
python benchmark_latency.py \
    --model deepseek-ai/DeepSeek-V3-32B \
    --batch-size 1 \
    --input-len 512 \
    --output-len 128
```

**预期结果**:

- 吞吐量 ≥ 2000 tokens/s
- 首Token延迟 ≤ 200ms
- 生成延迟 ≤ 30ms/token
- 无OOM错误

---

### TC-002: vLLM DeepSeek-70B 基准测试

**目的**: 验证 DeepSeek-70B 在 vLLM 框架下的推理性能

**前置条件**:

- H20 8卡机器环境就绪
- vLLM 安装完成
- 模型权重已下载

**测试步骤**:

1. **启动 vLLM 服务**

```bash
python -m vllm.entrypoints.openai.api_server \
    --model deepseek-ai/DeepSeek-V3-70B \
    --tensor-parallel-size 8 \
    --gpu-memory-utilization 0.90 \
    --max-model-len 8192 \
    --trust-remote-code \
    --port 8000
```

2. **运行吞吐量测试**

```bash
python benchmark_throughput.py \
    --backend vllm \
    --model deepseek-ai/DeepSeek-V3-70B \
    --input-len 512 \
    --output-len 512 \
    --num-prompts 100 \
    --tensor-parallel-size 8
```

**预期结果**:

- 吞吐量 ≥ 1200 tokens/s
- 首Token延迟 ≤ 350ms
- 生成延迟 ≤ 50ms/token

---

### TC-003: vLLM DeepSeek-671B 基准测试 (FP8量化)

**目的**: 验证 DeepSeek-671B FP8量化模型在 vLLM 框架下的推理性能

**前置条件**:

- H20 8卡机器环境就绪 (141GB × 8 = 1128GB)
- 模型权重已下载

**测试步骤**:

1. **启动 vLLM 服务 (FP8量化)**

```bash
python -m vllm.entrypoints.openai.api_server \
    --model deepseek-ai/DeepSeek-V3-671B \
    --tensor-parallel-size 8 \
    --dtype float16 \
    --kv-cache-dtype fp8 \
    --gpu-memory-utilization 0.90 \
    --max-model-len 8192 \
    --trust-remote-code \
    --port 8000
```

2. **运行吞吐量测试**

```bash
python benchmark_throughput.py \
    --backend vllm \
    --model deepseek-ai/DeepSeek-V3-671B \
    --input-len 512 \
    --output-len 512 \
    --num-prompts 50 \
    --tensor-parallel-size 8
```

**预期结果**:

- 吞吐量 ≥ 480 tokens/s
- 首Token延迟 ≤ 720ms
- 模型成功加载无OOM

---

### TC-004: SGLang DeepSeek-32B 基准测试

**目的**: 验证 DeepSeek-32B 在 SGLang 框架下的推理性能

**测试步骤**:

1. **启动 SGLang 服务**

```bash
python -m sglang.launch_server \
    --model-path deepseek-ai/DeepSeek-V3-32B \
    --tp 4 \
    --host 0.0.0.0 \
    --port 30000 \
    --mem-fraction-static 0.85 \
    --chunked-prefill-size 8192
```

2. **运行吞吐量测试**

```bash
python -m sglang.bench_throughput \
    --backend sglang \
    --tokenizer deepseek-ai/DeepSeek-V3-32B \
    --num-prompts 100 \
    --input-len 512 \
    --output-len 512 \
    --port 30000
```

3. **运行延迟测试**

```bash
python -m sglang.bench_latency \
    --model deepseek-ai/DeepSeek-V3-32B \
    --batch-size 1 \
    --input-len 512 \
    --output-len 128
```

**预期结果**:

- 吞吐量 ≥ 2500 tokens/s (SGLang RadixAttention优化)
- 首Token延迟 ≤ 180ms
- 生成延迟 ≤ 25ms/token

---

### TC-005: SGLang DeepSeek-70B 基准测试

**目的**: 验证 DeepSeek-70B 在 SGLang 框架下的推理性能

**测试步骤**:

1. **启动 SGLang 服务**

```bash
python -m sglang.launch_server \
    --model-path deepseek-ai/DeepSeek-V3-70B \
    --tp 8 \
    --host 0.0.0.0 \
    --port 30000 \
    --mem-fraction-static 0.85 \
    --chunked-prefill-size 4096
```

2. **运行吞吐量测试**

```bash
python -m sglang.bench_throughput \
    --backend sglang \
    --tokenizer deepseek-ai/DeepSeek-V3-70B \
    --num-prompts 100 \
    --input-len 512 \
    --output-len 512 \
    --port 30000
```

**预期结果**:

- 吞吐量 ≥ 1500 tokens/s
- 首Token延迟 ≤ 300ms

---

### TC-006: SGLang DeepSeek-671B 基准测试 (FP8量化)

**目的**: 验证 DeepSeek-671B FP8量化模型在 SGLang 框架下的推理性能

**前置条件**:

- H20 8卡机器环境就绪 (141GB × 8 = 1128GB)
- 模型权重已下载

**测试步骤**:

1. **启动 SGLang 服务 (FP8)**

```bash
python -m sglang.launch_server \
    --model-path deepseek-ai/DeepSeek-V3-671B \
    --tp 8 \
    --host 0.0.0.0 \
    --port 30000 \
    --mem-fraction-static 0.88 \
    --dtype float8_e4m3fn \
    --chunked-prefill-size 4096
```

2. **运行吞吐量测试**

```bash
python -m sglang.bench_throughput \
    --backend sglang \
    --tokenizer deepseek-ai/DeepSeek-V3-671B \
    --num-prompts 50 \
    --input-len 512 \
    --output-len 512 \
    --port 30000
```

**预期结果**:

- 吞吐量 ≥ 580 tokens/s
- 首Token延迟 ≤ 620ms

---

### TC-007: 并发压力测试

**目的**: 验证高并发场景下的服务稳定性

**测试步骤**:

1. **启动服务** (以vLLM DeepSeek-32B为例)

```bash
python -m vllm.entrypoints.openai.api_server \
    --model deepseek-ai/DeepSeek-V3-32B \
    --tensor-parallel-size 4 \
    --max-num-seqs 256 \
    --port 8000
```

2. **运行并发测试**

```bash
python benchmark_serving.py \
    --backend openai \
    --base-url http://localhost:8000/v1 \
    --model deepseek-ai/DeepSeek-V3-32B \
    --dataset-name sharegpt \
    --num-prompts 500 \
    --request-rate 10
```

**预期结果**:

- P99延迟 ≤ 5s
- 错误率 ≤ 1%
- 无服务崩溃

---

### TC-008: 长序列测试

**目的**: 验证长上下文场景下的推理能力

**测试步骤**:

1. **配置长序列支持**

```bash
python -m vllm.entrypoints.openai.api_server \
    --model deepseek-ai/DeepSeek-V3-32B \
    --tensor-parallel-size 4 \
    --max-model-len 32768 \
    --enable-chunked-prefill \
    --port 8000
```

2. **运行长序列测试**

```bash
python benchmark_latency.py \
    --model deepseek-ai/DeepSeek-V3-32B \
    --input-len 16384 \
    --output-len 512 \
    --batch-size 1
```

**预期结果**:

- 16K输入成功处理
- 首Token延迟 ≤ 2s
- 无OOM错误

---

## 监控指标

测试过程中需要监控以下指标：

| 指标          | 采集方式        | 告警阈值             |
| ------------- | --------------- | -------------------- |
| GPU显存使用率 | nvidia-smi      | > 95%                |
| GPU利用率     | nvidia-smi      | < 70% (可能存在瓶颈) |
| 吞吐量        | 日志/Prometheus | 低于预期20%          |
| P99延迟       | 日志/Prometheus | > 5s                 |
| 错误率        | 日志统计        | > 1%                 |

---

## 测试执行清单

| 测试ID | 模型          | 框架        | 状态 | 执行人 | 日期 |
| ------ | ------------- | ----------- | ---- | ------ | ---- |
| TC-001 | DeepSeek-32B  | vLLM        | done |        |      |
| TC-002 | DeepSeek-70B  | vLLM        | done |        |      |
| TC-003 | DeepSeek-671B | vLLM        | done |        |      |
| TC-004 | DeepSeek-32B  | SGLang      | done |        |      |
| TC-005 | DeepSeek-70B  | SGLang      | done |        |      |
| TC-006 | DeepSeek-671B | SGLang      | done |        |      |
| TC-007 | DeepSeek-32B  | vLLM/SGLang | done |        |      |
| TC-008 | DeepSeek-32B  | vLLM        | done |        |      |
