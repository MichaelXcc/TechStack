---
type: note
tags: [CUDA, 基础概念, 编程模型]
title: CUDA编程模型概述
date: 2026-01-13
---

# CUDA编程模型概述

## 1. 核心概念

CUDA (Compute Unified Device Architecture) 是NVIDIA推出的并行计算平台和编程模型。它允许开发者使用 C/C++ 扩展语言编写 GPU 程序。

### Kernel (核函数)
- 运行在 GPU 上的函数。
- 在 Host 代码中调用，在 Device 上执行。
- 启动时会产生大量的 **线程 (Threads)**，每个线程执行相同的代码，但处理不同的数据。
- 使用 `__global__` 限定符声明。

## 2. 线程层次结构 (Thread Hierarchy)

为了管理成千上万的线程，CUDA 将其组织成一个两层的层级结构：

### Grid (网格)
- Kernel启动时产生的所有线程的集合称为一个 Grid。
- Grid 由多个 **Block (线程块)** 组成。
- 维度: 1D, 2D, 或 3D。

### Block (线程块)
- 一个 Block 包含多个 **Thread (线程)**。
- Block 内的线程可以通过 **Shared Memory (共享内存)** 通信，并进行同步 (`__syncthreads()`)。
- Block 也是 1D, 2D, 或 3D 的。
- 硬件限制: 一个 Block 最多通常包含 1024 个线程。

### Thread (线程)
- 最小执行单元。
- 拥有唯一的索引，用于计算要处理的数据地址。

## 3. 索引计算

在 Kernel 内部，我们可以通过内置变量访问线程的坐标：

- `threadIdx`: 线程在 Block 内的索引 (x, y, z)。
- `blockIdx`: Block 在 Grid 内的索引 (x, y, z)。
- `blockDim`: Block 的维度大小 (即一个Block有多少线程)。
- `gridDim`: Grid 的维度大小 (即一个Grid有多少Block)。

### 全局唯一索引计算公式
对于 1D Grid 和 1D Block：
```cpp
int tid = blockIdx.x * blockDim.x + threadIdx.x;
```

对于 2D 图片处理 (x为列, y为行):
```cpp
int x = blockIdx.x * blockDim.x + threadIdx.x;
int y = blockIdx.y * blockDim.y + threadIdx.y;
```

## 4. 硬件映射
- **Thread** -> **SP (CUDA Core)**
- **Block** -> **SM (Streaming Multiprocessor)**
    - 一个 Block 必须分配给一个 SM，且整个生命周期驻留在该 SM 上。
    - 一个 SM 可以同时驻留多个 Block (取决于资源限制)。
- **Warp (线程束)**
    - 硬件调度的基本单位。
    - 通常包含 32 个连续的线程。
    - 一个 Warp 内的线程执行相同的指令 (SIMT)。

---
**上一篇**: [[01_基础概念/01_GPU架构与异构计算|GPU架构与异构计算]]
**下一篇**: [[01_基础概念/03_环境搭建与HelloCUDA|环境搭建与HelloCUDA]]
