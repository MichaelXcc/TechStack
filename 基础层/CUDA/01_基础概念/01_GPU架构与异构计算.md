---
type: note
tags: [CUDA, 基础概念, 架构]
title: GPU架构与异构计算
date: 2026-01-13
---

# GPU架构与异构计算

## 1. 异构计算 (Heterogeneous Computing)

异构计算是指使用不同类型指令集和体系架构的计算单元组成的系统进行计算。在CUDA语境下，主要是指 **CPU + GPU** 的协同工作。

- **Host (主机)**: CPU及其内存（系统内存）。负责逻辑控制、串行计算、IO操作。
- **Device (设备)**: GPU及其内存（显存）。负责大规模并行计算、浮点运算。

### 为什么需要异构计算？
CPU和GPU的设计目标不同，互为补充：
- **CPU**: 优化 **延迟 (Latency)**。拥有巨大的缓存（Cache）和复杂的控制逻辑（Control），适合处理复杂的串行逻辑。
- **GPU**: 优化 **吞吐量 (Throughput)**。拥有大量的计算核心（ALU），适合处理数据并行的任务。

> [!INFO] 比喻
> - **CPU** 像一个法拉利赛车手，反应极快，适合处理单一复杂的指令。
> - **GPU** 像一辆运载量巨大的大巴车（或者成百上千个小学生），虽然单体反应不如赛车手，但在搬运大量砖块（数据）时效率极高。

## 2. GPU 硬件架构

### 核心组件
1.  **SM (Streaming Multiprocessor, 流多处理器)**
    - GPU的核心构建块。
    - 一个GPU包含多个SM。
    - 每个SM包含多个CUDA Core (SP)、共享内存 (Shared Memory)、寄存器文件 (Register File) 等。
    
2.  **SP (Streaming Processor, 流处理器) / CUDA Core**
    - 基本的运算单元，执行整数或浮点运算。
    
3.  **Memory Hierarchy (简述)**
    - **Global Memory**: 显存，容量大但速度慢。
    - **L2 Cache**: 所有SM共享。
    - **L1 Cache / Shared Memory**: SM内部，速度极快。
    - **Registers**: 速度最快，每个线程私有。

### SIMT 模型
**SIMT (Single Instruction, Multiple Threads)** 是NVIDIA GPU的执行模型。
- **单指令**: 许多线程执行相同的指令。
- **多线程**: 每个线程处理不同的数据。

这与CPU的SIMD (Single Instruction, Multiple Data) 类似，但SIMT允许更灵活的线程级并行，尽管在发生分支发散 (Branch Divergence) 时会降低效率。

## 3. 性能指标
- **FLOPS**: 每秒浮点运算次数。
- **Memory Bandwidth**: 显存带宽 (GB/s)。
- 相比CPU，GPU通常拥有高出一个数量级的FLOPS和带宽。

---
**下一篇**: [[01_基础概念/02_CUDA编程模型概述|CUDA编程模型概述]]
