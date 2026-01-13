---
type: note
tags: [CUDA, 核心编程, 内存模型]
title: CUDA内存模型详解
date: 2026-01-13
---

# CUDA内存模型详解

理解内存层次结构是编写高性能 CUDA 程序的关键。

## 1. 内存层次图谱

| 内存类型 | 作用域 (Scope) | 生命周期 (Lifetime) | 存取速度 | 存储位置 | 缓存 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Registers** | Thread | Thread | 极快 (1周期) | 片上 (On-chip) | 无 |
| **Local Memory** | Thread | Thread | 慢 (同Global) | 显存 (Off-chip) | L1/L2 |
| **Shared Memory** | Block | Block | 很快 | 片上 (On-chip) | N/A |
| **Global Memory** | Grid | Application | 慢 (几百周期) | 显存 (Off-chip) | L1/L2 |
| **Constant Memory** | Grid | Application | 快 (有缓存) | 显存 (Off-chip) | Constant Cache |
| **Texture Memory** | Grid | Application | 快 (空间局部性) | 显存 (Off-chip) | Texture Cache |

## 2. 详解

### Registers (寄存器)
- 编译器自动分配。
- 速度最快。
- 资源有限：每个 SM 有固定数量的寄存器文件（例如 64K 个 32-bit 寄存器）。如果 Kernel 使用过多寄存器，会限制 SM 上能同时运行的 Warp 数量（降低 Occupancy），导致寄存器溢出 (Register Spilling) 到 Local Memory。

### Shared Memory (共享内存)
- **Programmable Cache (可编程缓存)**。
- Block 内线程共享，用于通过协作交换数据。
- 速度接近寄存器。
- 常见用途：
    - 缓存 Global Memory 数据，减少重复读取。
    - 线程间通信/归约 (Reduction)。
- **Bank Conflict (存储体冲突)**: Shared Memory 被划分为 Bank（通常32个）。如果 Warp 内多个线程同时访问同一个 Bank 的不同地址，会由并行变串行。

### Global Memory (全局内存)
- 容量最大，速度最慢。
- 所有线程可见。
- **Coalesced Access (合并访问)**: 
    - 硬件总是以 32 字节（或 128 字节）的事务块读取显存。
    - 如果 Warp 内 32 个线程读取的地址是连续且对齐的，只需 1 次内存事务。
    - 如果地址是跨步的 (Strided) 或乱序的，可能需要多次事务，导致带宽利用率极低。

### Local Memory (局部内存)
- 名字误导：实际上位于 **显存 (Off-chip)** 中，速度慢。
- 用于存放寄存器放不下的变量（寄存器溢出）或索引无法在编译期确定的数组。

### Constant Memory (常量内存)
- 只读。
- 对所有线程广播相同数据时非常高效（广播机制）。
- 有专用缓存。
- 只有 64KB 大小。

## 3. 优化原则 (Rule of Thumb)
1.  **最大化数据重用**: 将频繁使用的数据从 Global Memory 搬运到 Shared Memory。
2.  **保证合并访问**: 让 Global Memory 的读取模式满足 Coalescing 要求。
3.  **避免 Bank Conflict**: 优化 Shared Memory 的访问模式。

---
**上一篇**: [[02_核心编程/01_线程组织与执行|线程组织与执行]]
**下一篇**: [[02_核心编程/03_Kernel编写与启动|Kernel编写与启动]]
