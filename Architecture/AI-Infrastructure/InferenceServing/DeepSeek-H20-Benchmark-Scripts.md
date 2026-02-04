# DeepSeek H20 基准测试脚本

## 概述

本文档提供 DeepSeek 模型在 H20 8卡机器上的基准测试脚本。

> [!IMPORTANT]
> 请使用最新版本的推理框架：
>
> - **vLLM**: `pip install vllm --upgrade` (≥0.13.0)
> - **SGLang**: `pip install sglang[all] --upgrade` (≥0.5.2)

---

## 环境检查脚本

```bash
#!/bin/bash
# check_environment.sh

echo "========== GPU 环境检查 =========="

GPU_COUNT=$(nvidia-smi -L | wc -l)
echo "GPU 数量: $GPU_COUNT"

nvidia-smi --query-gpu=index,name,memory.total,memory.free --format=csv
echo ""
nvidia-smi topo -m

python --version
pip show vllm 2>/dev/null && echo "✅ vLLM 已安装" || echo "❌ vLLM 未安装"
pip show sglang 2>/dev/null && echo "✅ SGLang 已安装" || echo "❌ SGLang 未安装"
```

---

## 吞吐量测试脚本

```python
#!/usr/bin/env python3
# benchmark_throughput.py
import argparse, asyncio, time, json, aiohttp, statistics
from tqdm.asyncio import tqdm_asyncio

class ThroughputBenchmark:
    def __init__(self, backend, base_url, model, input_len, output_len, num_prompts, concurrency=32):
        self.backend = backend
        self.base_url = base_url.rstrip("/")
        self.model = model
        self.input_len = input_len
        self.output_len = output_len
        self.num_prompts = num_prompts
        self.concurrency = concurrency
        self.prompts = self._generate_prompts()

    def _generate_prompts(self):
        base_text = "这是一个用于测试大语言模型推理性能的文本。"
        prompts = []
        for _ in range(self.num_prompts):
            target_chars = int(self.input_len * 1.5)
            prompt = (base_text * (target_chars // len(base_text) + 1))[:target_chars]
            prompts.append(f"请对以下文本进行总结：\n\n{prompt}")
        return prompts

    async def _send_request(self, session, prompt, semaphore):
        async with semaphore:
            start_time = time.perf_counter()
            url = f"{self.base_url}/v1/completions" if self.backend == "vllm" else f"{self.base_url}/generate"
            payload = {"model": self.model, "prompt": prompt, "max_tokens": self.output_len} if self.backend == "vllm" \
                else {"text": prompt, "sampling_params": {"max_new_tokens": self.output_len}}
            try:
                async with session.post(url, json=payload) as resp:
                    result = await resp.json()
                    return {"success": True, "latency": time.perf_counter() - start_time,
                            "output_tokens": result.get("usage", {}).get("completion_tokens", self.output_len)}
            except Exception as e:
                return {"success": False, "latency": time.perf_counter() - start_time, "error": str(e)}

    async def run(self):
        print(f"\n{'='*50}\n吞吐量测试: {self.backend} | {self.model}\n{'='*50}")
        semaphore = asyncio.Semaphore(self.concurrency)
        async with aiohttp.ClientSession() as session:
            start = time.perf_counter()
            results = await tqdm_asyncio.gather(*[self._send_request(session, p, semaphore) for p in self.prompts])
            total_time = time.perf_counter() - start

        success = [r for r in results if r["success"]]
        total_tokens = sum(r["output_tokens"] for r in success)
        latencies = [r["latency"] for r in success]

        print(f"吞吐量: {total_tokens/total_time:.2f} tokens/s")
        print(f"请求成功: {len(success)}/{len(results)}")
        print(f"P50延迟: {statistics.median(latencies):.3f}s")
        return {"throughput": total_tokens/total_time, "success": len(success)}

async def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("--backend", default="vllm")
    parser.add_argument("--base-url", default="http://localhost:8000")
    parser.add_argument("--model", required=True)
    parser.add_argument("--input-len", type=int, default=512)
    parser.add_argument("--output-len", type=int, default=512)
    parser.add_argument("--num-prompts", type=int, default=100)
    args = parser.parse_args()

    benchmark = ThroughputBenchmark(args.backend, args.base_url, args.model,
                                     args.input_len, args.output_len, args.num_prompts)
    await benchmark.run()

if __name__ == "__main__":
    asyncio.run(main())
```

---

## 延迟测试脚本

```python
#!/usr/bin/env python3
# benchmark_latency.py
import argparse, asyncio, time, aiohttp

class LatencyBenchmark:
    def __init__(self, backend, base_url, model, input_len, output_len, num_runs=10):
        self.backend, self.base_url, self.model = backend, base_url.rstrip("/"), model
        self.input_len, self.output_len, self.num_runs = input_len, output_len, num_runs

    async def run(self):
        print(f"\n{'='*50}\n延迟测试: {self.backend} | {self.model}\n{'='*50}")
        prompt = "测试文本 " * (self.input_len // 3)
        results = []

        async with aiohttp.ClientSession() as session:
            for i in range(self.num_runs):
                url = f"{self.base_url}/v1/completions" if self.backend == "vllm" else f"{self.base_url}/generate"
                payload = {"model": self.model, "prompt": prompt, "max_tokens": self.output_len, "stream": True}
                start = time.perf_counter()
                first_token = None
                async with session.post(url, json=payload) as resp:
                    async for _ in resp.content:
                        if first_token is None:
                            first_token = time.perf_counter()
                end = time.perf_counter()
                ttft = (first_token - start) * 1000 if first_token else 0
                results.append({"ttft": ttft, "total": end - start})
                print(f"  Run {i+1}: TTFT={ttft:.2f}ms")

        avg_ttft = sum(r["ttft"] for r in results) / len(results)
        print(f"\n平均 TTFT: {avg_ttft:.2f}ms")
        return {"avg_ttft_ms": avg_ttft}

async def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("--backend", default="vllm")
    parser.add_argument("--base-url", default="http://localhost:8000")
    parser.add_argument("--model", required=True)
    parser.add_argument("--input-len", type=int, default=512)
    parser.add_argument("--output-len", type=int, default=128)
    args = parser.parse_args()
    await LatencyBenchmark(args.backend, args.base_url, args.model, args.input_len, args.output_len).run()

if __name__ == "__main__":
    asyncio.run(main())
```

---

## GPU 监控脚本

```python
#!/usr/bin/env python3
# monitor_gpu.py
import subprocess, time, csv, argparse
from datetime import datetime

def get_gpu_stats():
    result = subprocess.run(
        ["nvidia-smi", "--query-gpu=index,utilization.gpu,memory.used,memory.total,temperature.gpu",
         "--format=csv,noheader,nounits"], capture_output=True, text=True)
    return [dict(zip(["id", "util", "mem_used", "mem_total", "temp"],
                     [float(x.strip()) for x in line.split(",")]))
            for line in result.stdout.strip().split("\n") if line]

def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("--interval", type=float, default=1.0)
    parser.add_argument("--output", default="gpu_stats.csv")
    args = parser.parse_args()

    print(f"监控中... 输出: {args.output}")
    with open(args.output, "w", newline="") as f:
        writer = csv.writer(f)
        writer.writerow(["timestamp", "gpu_id", "util%", "mem_used_mb", "mem%", "temp_c"])
        try:
            while True:
                for gpu in get_gpu_stats():
                    writer.writerow([datetime.now().isoformat(), gpu["id"], gpu["util"],
                                    gpu["mem_used"], gpu["mem_used"]/gpu["mem_total"]*100, gpu["temp"]])
                f.flush()
                time.sleep(args.interval)
        except KeyboardInterrupt:
            print("\n停止")

if __name__ == "__main__":
    main()
```
