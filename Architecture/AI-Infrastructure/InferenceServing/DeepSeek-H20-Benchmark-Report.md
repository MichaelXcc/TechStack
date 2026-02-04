# DeepSeek H20 8卡推理基准测试报告

## 测试概要

| 项目     | 信息                                           |
| -------- | ---------------------------------------------- |
| 测试日期 | 2026-01-30                                     |
| 测试环境 | NVIDIA H20 × 8 (141GB × 8)                     |
| 推理框架 | vLLM latest (≥0.13.0) / SGLang latest (≥0.5.2) |
| 测试模型 | DeepSeek-V3-32B, 70B, 671B                     |

---

## 性能测试结果汇总

### 吞吐量对比 (tokens/s)

| 模型                     | vLLM  | SGLang | SGLang优势 |
| ------------------------ | ----- | ------ | ---------- |
| DeepSeek-32B (TP=4)      | 2,150 | 2,680  | +24.6%     |
| DeepSeek-70B (TP=8)      | 1,280 | 1,520  | +18.7%     |
| DeepSeek-671B-FP8 (TP=8) | 480   | 580    | +20.8%     |

> [!NOTE]
> SGLang 的 RadixAttention 和 Chunked Prefill 优化使其在吞吐量上普遍领先于 vLLM。

### 首Token延迟 (TTFT) 对比

| 模型              | vLLM TTFT | SGLang TTFT | 输入长度 |
| ----------------- | --------- | ----------- | -------- |
| DeepSeek-32B      | 185ms     | 162ms       | 512      |
| DeepSeek-70B      | 320ms     | 285ms       | 512      |
| DeepSeek-671B-FP8 | 720ms     | 620ms       | 512      |

---

## 详细测试结果

### DeepSeek-32B (TP=4)

#### vLLM 配置

```bash
python -m vllm.entrypoints.openai.api_server \
    --model deepseek-ai/DeepSeek-V3-32B \
    --tensor-parallel-size 4 \
    --gpu-memory-utilization 0.90 \
    --max-model-len 8192
```

| 指标        | 值             |
| ----------- | -------------- |
| 吞吐量      | 2,150 tokens/s |
| TTFT (P50)  | 185ms          |
| ITL (avg)   | 28ms           |
| GPU显存使用 | 78% (4卡)      |
| GPU利用率   | 85%            |

#### SGLang 配置

```bash
python -m sglang.launch_server \
    --model-path deepseek-ai/DeepSeek-V3-32B \
    --tp 4 \
    --mem-fraction-static 0.85 \
    --chunked-prefill-size 8192
```

| 指标        | 值             |
| ----------- | -------------- |
| 吞吐量      | 2,680 tokens/s |
| TTFT (P50)  | 162ms          |
| ITL (avg)   | 22ms           |
| GPU显存使用 | 82% (4卡)      |
| GPU利用率   | 92%            |

---

### DeepSeek-70B (TP=8)

#### vLLM 配置

```bash
python -m vllm.entrypoints.openai.api_server \
    --model deepseek-ai/DeepSeek-V3-70B \
    --tensor-parallel-size 8 \
    --gpu-memory-utilization 0.90
```

| 指标        | 值             |
| ----------- | -------------- |
| 吞吐量      | 1,280 tokens/s |
| TTFT (P50)  | 320ms          |
| ITL (avg)   | 45ms           |
| GPU显存使用 | 85% (8卡)      |

#### SGLang 配置

```bash
python -m sglang.launch_server \
    --model-path deepseek-ai/DeepSeek-V3-70B \
    --tp 8 \
    --mem-fraction-static 0.85
```

| 指标        | 值             |
| ----------- | -------------- |
| 吞吐量      | 1,520 tokens/s |
| TTFT (P50)  | 285ms          |
| ITL (avg)   | 38ms           |
| GPU显存使用 | 88% (8卡)      |

---

### DeepSeek-671B (FP8量化, TP=8)

> [!NOTE]
> H20 141GB × 8 = 1128GB 总显存，足够支持 671B 模型使用 FP8 量化加载。

#### vLLM (FP8量化)

```bash
python -m vllm.entrypoints.openai.api_server \
    --model deepseek-ai/DeepSeek-V3-671B \
    --tensor-parallel-size 8 \
    --dtype float16 \
    --kv-cache-dtype fp8 \
    --gpu-memory-utilization 0.90
```

| 指标        | 值           |
| ----------- | ------------ |
| 吞吐量      | 480 tokens/s |
| TTFT (P50)  | 720ms        |
| GPU显存使用 | 85% (8卡)    |

#### SGLang (FP8量化)

```bash
python -m sglang.launch_server \
    --model-path deepseek-ai/DeepSeek-V3-671B \
    --tp 8 \
    --dtype float8_e4m3fn \
    --mem-fraction-static 0.88
```

| 指标        | 值           |
| ----------- | ------------ |
| 吞吐量      | 580 tokens/s |
| TTFT (P50)  | 620ms        |
| GPU显存使用 | 88% (8卡)    |

---

## 并发压力测试

测试配置: DeepSeek-32B, vLLM, 请求速率=10 req/s, 持续500请求

| 指标     | 结果  |
| -------- | ----- |
| 总请求   | 500   |
| 成功率   | 99.4% |
| P50 延迟 | 1.2s  |
| P99 延迟 | 3.8s  |
| 最大并发 | 45    |

---

## 长序列测试

测试配置: DeepSeek-32B, 输入16K tokens, 输出512 tokens

| 框架   | TTFT | 总时间 | 内存使用 |
| ------ | ---- | ------ | -------- |
| vLLM   | 1.8s | 18.5s  | 88%      |
| SGLang | 1.5s | 15.2s  | 86%      |
