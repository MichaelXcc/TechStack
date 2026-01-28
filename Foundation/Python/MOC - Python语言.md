---
type: moc
tags: [moc, python, 基础层]
title: Python语言MOC
---

# Python语言 - 进阶与最佳实践

## 🎯 目标
- 深入理解Python的高级特性与底层机制
- 掌握Python并发编程与性能优化
- 学习Python在大型项目中的工程化实践

## 📁 核心知识体系

### 1. 底层机制与高级特性
> 深入理解Python的动态特性与元编程能力

- [[基础层/Python/元类|元类 (Metaclasses)]]：类创建的本质、`type`的使用、自定义元类
- [[基础层/Python/装饰器|装饰器 (Decorators)]]：闭包、函数装饰器、类装饰器、`functools`
- [[基础层/Python/上下文管理器|上下文管理器 (Context Managers)]]：`with`语句、`__enter__`/`__exit__`、`contextlib`
- [[基础层/Python/生成器与迭代器|生成器与迭代器 (Generators & Iterators)]]：`yield`、惰性求值、迭代协议

### 2. 并发与异步编程
> 突破GIL限制，提升IO密集型与CPU密集型任务性能

- [[基础层/Python/并发编程|并发编程 (Concurrency)]]：多线程 vs 多进程、GIL机制、Asyncio异步编程模型

### 3. 工程化与代码质量
> 编写健壮、可维护的Python代码

- [[基础层/Python/类型提示|类型提示 (Type Hinting)]]：Type Hints、Generics、Mypy静态检查
- [[基础层/Python/内存管理|内存管理 (Memory Management)]]：引用计数、垃圾回收(GC)、内存池机制

## 🔗 扩展阅读
- [[Go/MOC - Go语言|Go语言体系]] (对比学习)
- [[架构层/AI架构/AI训练调度|AI训练调度]] (Python应用场景)

## 📊 学习进度
```dataview
TASK
FROM "基础层/Python"
WHERE file.name != this.file.name
```
