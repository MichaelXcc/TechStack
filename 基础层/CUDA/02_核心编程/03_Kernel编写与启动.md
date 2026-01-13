---
type: note
tags: [CUDA, 核心编程, Kernel]
title: Kernel编写与启动
date: 2026-01-13
---

# Kernel编写与启动

## 1. 函数限定符

CUDA 扩展了 C++ 语法，使用限定符来定义函数运行的位置和调用方式：

| 限定符 | 执行位置 | 调用位置 | 用途 |
| :--- | :--- | :--- | :--- |
| `__global__` | Device (GPU) | Host (CPU)* | 定义 Kernel 入口函数，必须返回 `void` |
| `__device__` | Device (GPU) | Device (GPU) | 辅助函数，帮助 Kernel 封装逻辑 |
| `__host__` | Host (CPU) | Host (CPU) | 普通 C++ 函数 (默认) |
| `__host__ __device__` | Both | Both | 在两端都可以编译运行的函数 |

> *注：从计算能力 3.5 开始，支持动态并行 (Dynamic Parallelism)，允许从 Device 调用 `__global__` 函数。

## 2. Kernel 配置与启动

使用 `<<<GridDim, BlockDim, SharedMemBytes, Stream>>>` 语法。

### dim3 结构体
CUDA 使用 `dim3` 结构体来定义 Grid 和 Block 的维度 (x, y, z)。未指定的维度默认为 1。

```cpp
// 定义 2D Grid 和 2D Block
dim3 block(16, 16); // 16*16 = 256 thread per block
dim3 grid((width + block.x - 1) / block.x, (height + block.y - 1) / block.y);

// 启动 Kernel
imageFilter<<<grid, block>>>(d_img, width, height);
```

### 参数传递
- Kernel 参数通过 **值传递** 到 Constant Memory 区域（如果参数很少，有些架构是这样，或者通过 Kernel Argument Buffer）。
- 只能传递指针（指向显存地址）或普通标量。不能直接传递 Host 端的复杂 C++ 对象（除非是 POD 类型且不包含 Host 指针）。

## 3. 错误处理

Kernel 启动是 **异步** 的。Host 发出指令后立即返回，不会等待 GPU 执行完毕。

### 捕获启动错误
```cpp
myKernel<<<grid, block>>>(...);
cudaError_t err = cudaGetLastError(); // 捕获启动参数等配置错误
if (err != cudaSuccess) {
    printf("Launch Error: %s\n", cudaGetErrorString(err));
}
```

### 捕获执行错误
要捕获 Kernel 执行过程中发生的错误（如非法内存访问），需要同步：
```cpp
cudaDeviceSynchronize(); // 强制 CPU 等待 GPU 结束
err = cudaGetLastError();
if (err != cudaSuccess) {
    printf("Execution Error: %s\n", cudaGetErrorString(err));
}
```
> [!TIP]
> 在发布版代码中，通常只在该同步点之后检查错误。开发调试时建议使用 `CUDA_LAUNCH_BLOCKING=1` 环境变量来强制同步。

---
**上一篇**: [[02_核心编程/02_CUDA内存模型详解|CUDA内存模型详解]]
**下一篇**: [[03_性能优化/01_访存优化策略|访存优化策略]]
