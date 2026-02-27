# SGLang 知识图谱

> SGLang (Structured Generation Language) 是由 Berkeley 大学开发的高性能 LLM 推理框架，专注于低延迟、高吞吐的模型服务。

## 📚 知识结构

```
SGLang/
├── 01-概述与定位.md          # SGLang 是什么，解决什么问题
├── 02-核心架构.md            # 三元架构：前端、编译器、运行时
├── 03-RadixAttention.md     # 核心创新：KV Cache 复用机制
├── 04-前端DSL语法.md         # 前端语言原语和使用方法
├── 05-性能优化技术.md        # 各种优化技术详解
├── 06-分布式部署.md          # 张量并行、流水线并行等
├── 07-实战案例.md            # 实际使用案例和最佳实践
└── MOC - SGLang.md          # 本文件
```

## 🎯 学习路径

### 第一阶段：基础理解

1. [[01-概述与定位]] - 了解 SGLang 的背景和定位
2. [[02-核心架构]] - 掌握整体架构设计

### 第二阶段：核心技术

3. [[03-RadixAttention]] - 深入理解 KV Cache 复用
4. [[04-前端DSL语法]] - 学习前端编程语言

### 第三阶段：高级应用

5. [[05-性能优化技术]] - 掌握各种优化手段
6. [[06-分布式部署]] - 学习大规模部署方案
7. [[07-实战案例]] - 实践应用

## 🔗 相关链接

- **GitHub**: https://github.com/sgl-project/sglang
- **论文**: https://arxiv.org/abs/2312.07104
- **文档**: https://sgl-project.github.io/

## 📊 SGLang vs 其他框架

| 特性          | SGLang               | vLLM           | TensorRT-LLM |
| ------------- | -------------------- | -------------- | ------------ |
| KV Cache 复用 | RadixAttention ✅    | PagedAttention | 有限支持     |
| 前端 DSL      | 原生支持 ✅          | 无             | 无           |
| 结构化生成    | 原生支持 ✅          | 需额外工具     | 需额外工具   |
| 多模态支持    | ✅                   | ✅             | ✅           |
| 硬件支持      | NVIDIA/AMD/Intel/TPU | 主要 NVIDIA    | NVIDIA       |

## 🏷️ 标签

#LLM #推理框架 #SGLang #高性能计算 #KVCache
