# RadixAttention 深度解析

## 什么是 RadixAttention？

**RadixAttention** 是 SGLang 的核心创新，基于 Radix Tree 的 KV Cache 自动复用机制。

```
传统方式 vs RadixAttention:

传统: 每个请求独立计算 KV Cache
Request 1: "You are an AI. What is Python?" → 100% 计算
Request 2: "You are an AI. What is Java?"   → 100% 计算

RadixAttention: 自动复用共享前缀
Request 1: [████████████████████] 100% 计算
Request 2: [░░░░░░░░░░░░][████] 35% 计算 (复用 65%)
```

## Radix Tree 结构

```python
class RadixNode:
    children: Dict[int, RadixNode]  # 子节点
    token_ids: List[int]            # 边上的 tokens
    kv_cache: Optional[KVCache]     # KV 缓存
    ref_count: int                  # 引用计数
```

## 工作流程

```
输入: [A, B, C, D, E, F]

1. 匹配: 在 Radix Tree 中找到 [A,B,C,D,E]
2. 复用: 使用已缓存的 KV
3. 计算: 只计算新的 [F]
4. 更新: 将完整序列加入树
```

## 应用场景与加速

| 场景     | 复用率 | Prefill 加速 |
| -------- | ------ | ------------ |
| 多轮对话 | 85%    | 6.7x         |
| RAG      | 90%    | 10x          |
| Few-shot | 80%    | 5x           |

## vs PagedAttention

| 特性     | RadixAttention | PagedAttention |
| -------- | -------------- | -------------- |
| 前缀复用 | 自动           | 需手动启用     |
| 粒度     | Token 级       | Page 级        |

---

> 下一篇: [[04-前端DSL语法]]
