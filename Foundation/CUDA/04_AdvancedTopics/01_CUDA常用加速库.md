---
type: note
tags: [CUDA, 进阶, 库]
title: CUDA常用加速库
date: 2026-01-13
---

# CUDA常用加速库

NVIDIA 提供了丰富的经过高度优化的数学库，通常比手写的 Kernel 性能更好且更稳定。**"不要重复造轮子"**。

## 1. 核心数学库

### cuBLAS (CUDA Basic Linear Algebra Subroutines)
- **用途**: 稠密线性代数运算（矩阵乘法 GEMM、向量运算等）。
- **地位**: 几乎是任何涉及矩阵运算的 CUDA 应用的基础。所有深度学习框架底层都依赖它。
- **特点**: 提供 Tensor Core 支持。

### cuFFT (CUDA Fast Fourier Transform)
- **用途**: 快速傅里叶变换。
- **特点**: 支持 1D, 2D, 3D 变换，不仅比 CPU 快，甚至比手写的 GPU FFT 快得多。

### cuSPARSE
- **用途**: 稀疏矩阵运算。
- **特点**: 针对稀疏数据结构优化的矩阵乘法、求解器。

### cuSOLVER
- **用途**: 高级线性代数求解器（特征值分解、SVD、LU/QR分解）。

## 2. 深度学习与图像

### cuDNN (CUDA Deep Neural Network library)
- **用途**: 深度神经网络的原语库（卷积、池化、激活函数）。
- **地位**: TensorFlow, PyTorch 的核心后端。

### NPP (NVIDIA Performance Primitives)
- **用途**: 图像和信号处理（滤波、几何变换、统计）。

## 3. C++ 模板库

### Thrust
- **用途**: 类似于 C++ STL 的并行算法库。
- **接口**: host_vector, device_vector, transform, reduce, sort。
- **优点**: 极大简化代码，只需几行即可实现并行排序或归约。

```cpp
#include <thrust/host_vector.h>
#include <thrust/device_vector.h>
#include <thrust/sort.h>

int main() {
    thrust::host_vector<int> h_vec(100);
    // ... fill h_vec ...
    thrust::device_vector<int> d_vec = h_vec;
    thrust::sort(d_vec.begin(), d_vec.end());
    // ...
}
```

---
**上一篇**: [[03_性能优化/03_流与并发执行|流与并发执行]]
**下一篇**: [[04_进阶主题/02_调试与性能分析工具|调试与性能分析工具]]
