# SGLang 前端 DSL 语法

## 核心原语

| 原语     | 功能     | 示例                                |
| -------- | -------- | ----------------------------------- |
| `gen`    | 生成文本 | `sgl.gen("output", max_tokens=100)` |
| `select` | 约束选择 | `sgl.select("choice", ["A", "B"])`  |
| `fork`   | 并行分支 | `s.fork(3)`                         |
| `join`   | 合并分支 | `s.join(forks)`                     |
| `+=`     | 追加内容 | `s += "Hello"`                      |

## 基础示例

```python
import sglang as sgl

@sgl.function
def simple_chat(s, question):
    s += sgl.system("You are a helpful assistant.")
    s += sgl.user(question)
    s += sgl.assistant(sgl.gen("answer", max_tokens=256))

# 运行
result = simple_chat.run(question="What is Python?")
print(result["answer"])
```

## 高级用法

### 结构化输出 (JSON)

```python
@sgl.function
def extract_info(s, text):
    s += f"Extract name and age from: {text}\n"
    s += sgl.gen("output", regex=r'\{"name": ".*", "age": \d+\}')
```

### 并行推理

```python
@sgl.function
def parallel_thoughts(s, question):
    s += sgl.user(question)
    forks = s.fork(3)  # 并行 3 个思路

    for i, fork in enumerate(forks):
        fork += sgl.gen(f"thought_{i}")

    s.join(forks)
    s += sgl.gen("final_answer")
```

### 选择模式

```python
@sgl.function
def classify(s, text):
    s += f"Classify: {text}\nCategory:"
    s += sgl.select("category", ["positive", "negative", "neutral"])
```

## 角色标记

```python
sgl.system("...")   # 系统提示
sgl.user("...")     # 用户输入
sgl.assistant("...") # 助手回复
```

---

> 下一篇: [[05-性能优化技术]]
