---
type: note
tags: [CUDA, 环境搭建, HelloCUDA]
title: 环境搭建与HelloCUDA
date: 2026-01-13
---

# 环境搭建与HelloCUDA

## 1. 环境准备

要进行 CUDA 开发，你需要：
1.  **NVIDIA GPU**: 支持 CUDA 的显卡。
2.  **GPU 驱动**: 安装最新的 NVIDIA 驱动。
3.  **CUDA Toolkit**: 包含编译器 (nvcc)、库 (cuBLAS等) 和工具。

验证安装：
```bash
nvcc --version
nvidia-smi
```

## 2. Hello World: 向量加法

这是一个经典的 "Hello World" 级别的并行计算示例：将两个数组相加。

`hello_cuda.cu`:

```cpp
#include <stdio.h>
#include <cuda_runtime.h>

// 1. 定义 Kernel 函数
// __global__ 表示在 GPU 执行，从 CPU 调用
__global__ void vectorAdd(const float *A, const float *B, float *C, int N) {
    // 计算全局索引
    int i = blockDim.x * blockIdx.x + threadIdx.x;
    
    // 边界检查
    if (i < N) {
        C[i] = A[i] + B[i];
    }
}

int main() {
    int N = 1 << 20; // 1M 元素
    size_t size = N * sizeof(float);
    
    // 2. 分配 Host 内存
    float *h_A = (float *)malloc(size);
    float *h_B = (float *)malloc(size);
    float *h_C = (float *)malloc(size);
    
    // 初始化数据
    for (int i = 0; i < N; ++i) {
        h_A[i] = rand() / (float)RAND_MAX;
        h_B[i] = rand() / (float)RAND_MAX;
    }
    
    // 3. 分配 Device 内存
    float *d_A, *d_B, *d_C;
    cudaMalloc((void **)&d_A, size);
    cudaMalloc((void **)&d_B, size);
    cudaMalloc((void **)&d_C, size);
    
    // 4. 将数据从 Host 拷贝到 Device
    cudaMemcpy(d_A, h_A, size, cudaMemcpyHostToDevice);
    cudaMemcpy(d_B, h_B, size, cudaMemcpyHostToDevice);
    
    // 5. 启动 Kernel
    // 256 是 blockDim (每个 block 256 线程)
    // (N + 255) / 256 是 gridDim (向上取整)
    int threadsPerBlock = 256;
    int blocksPerGrid = (N + threadsPerBlock - 1) / threadsPerBlock;
    
    vectorAdd<<<blocksPerGrid, threadsPerBlock>>>(d_A, d_B, d_C, N);
    
    // 检查 Kernel 启动错误
    cudaError_t err = cudaGetLastError();
    if (err != cudaSuccess) {
        fprintf(stderr, "Failed to launch vectorAdd kernel: %s\n", cudaGetErrorString(err));
        exit(EXIT_FAILURE);
    }
    
    // 6. 将结果从 Device 拷贝回 Host
    cudaMemcpy(h_C, d_C, size, cudaMemcpyDeviceToHost);
    
    // 简单验证
    printf("First 5 results:\n");
    for (int i = 0; i < 5; ++i) {
        printf("%f + %f = %f\n", h_A[i], h_B[i], h_C[i]);
    }
    
    // 7. 释放内存
    cudaFree(d_A);
    cudaFree(d_B);
    cudaFree(d_C);
    free(h_A);
    free(h_B);
    free(h_C);
    
    printf("Done\n");
    return 0;
}
```

## 3. 编译与运行

使用 `nvcc` 编译器：

```bash
nvcc hello_cuda.cu -o hello_cuda
./hello_cuda
```

> [!TIP] 重要API
> - `cudaMalloc`: 分配显存
> - `cudaMemcpy`: 数据传输 (Host <-> Device)
> - `cudaFree`: 释放显存
> - `<<<Grid, Block>>>`: Kernel 配置语法

---
**上一篇**: [[01_基础概念/02_CUDA编程模型概述|CUDA编程模型概述]]
