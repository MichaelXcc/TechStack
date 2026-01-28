---
type: guide
tags: [前瞻层, AI, MCP, Skill, 编码工具, 工具扩展, 协议]
title: MCP协议与工具扩展
author: 云原生技术架构师
---

# MCP 协议与工具扩展

## 📋 概述

> [!NOTE] 定义
> **MCP (Model Context Protocol)** 是由 Anthropic 开发的开放标准协议，定义了 AI 模型与外部工具、数据源之间的标准化交互方式。

MCP 解决的核心问题：**如何让 AI 安全、标准化、可扩展地访问外部能力？**

```mermaid
flowchart TB
    subgraph Before["❌ 无 MCP 时代"]
        B1[每个工具<br/>自定义接口]
        B2[集成成本高]
        B3[不可复用]
    end

    subgraph After["✅ MCP 时代"]
        A1[统一协议标准]
        A2[即插即用]
        A3[生态共享]
    end

    Before -->|协议标准化| After

    style Before fill:#ffe6e6
    style After fill:#e6ffe6
```

---

## 🏗️ MCP 核心架构

### 架构概览

```mermaid
flowchart LR
    subgraph Host["🖥️ Host（主机）"]
        direction TB
        AI[AI 模型]
        Client[MCP Client]
        AI <--> Client
    end

    subgraph Transport["📡 传输层"]
        STDIO[stdio]
        HTTP[HTTP/SSE]
    end

    subgraph Servers["🔧 MCP Servers"]
        direction TB
        S1[文件系统]
        S2[Git/GitHub]
        S3[数据库]
        S4[Kubernetes]
        S5[自定义服务]
    end

    Client <--> Transport
    Transport <--> Servers

    style Host fill:#e6f3ff
    style Servers fill:#e6ffe6
```

### 核心组件

| 组件          | 角色        | 职责                          |
| ------------- | ----------- | ----------------------------- |
| **Host**      | AI 应用主机 | 运行 AI 模型，管理 MCP 客户端 |
| **Client**    | MCP 客户端  | 与服务器建立连接，转发请求    |
| **Server**    | MCP 服务器  | 提供工具、资源和提示模板      |
| **Transport** | 传输层      | 处理通信（stdio、HTTP/SSE）   |

### 三大能力类别

```mermaid
mindmap
  root((MCP 能力))
    Tools 工具
      函数调用
      命令执行
      API 请求
    Resources 资源
      文件读取
      数据库查询
      网络内容
    Prompts 提示
      模板定义
      参数化提示
      工作流模板
```

| 能力          | 描述              | 示例                                      |
| ------------- | ----------------- | ----------------------------------------- |
| **Tools**     | AI 可调用的函数   | `run_command`、`write_file`、`search_web` |
| **Resources** | AI 可读取的数据源 | 文件内容、数据库记录、API 响应            |
| **Prompts**   | 预定义的交互模板  | 代码审查模板、重构建议模板                |

---

## 🔌 MCP 工作原理

### 生命周期

```mermaid
sequenceDiagram
    participant Host as AI Host
    participant Client as MCP Client
    participant Server as MCP Server

    Note over Host,Server: 初始化阶段
    Host->>Client: 启动 Client
    Client->>Server: 建立连接
    Server-->>Client: 连接确认
    Client->>Server: 请求能力列表
    Server-->>Client: 返回 Tools/Resources/Prompts

    Note over Host,Server: 运行阶段
    Host->>Client: 调用 Tool
    Client->>Server: 发送请求
    Server->>Server: 执行操作
    Server-->>Client: 返回结果
    Client-->>Host: 传递结果

    Note over Host,Server: 关闭阶段
    Host->>Client: 关闭请求
    Client->>Server: 断开连接
```

### 工具调用流程

```mermaid
flowchart TD
    A[用户请求<br/>'读取配置文件'] --> B[AI 分析意图]
    B --> C{确定工具}
    C --> D[调用 read_file<br/>工具]
    D --> E[MCP Server<br/>执行操作]
    E --> F[返回文件内容]
    F --> G[AI 处理并响应]

    style A fill:#e6f3ff
    style G fill:#e6ffe6
```

---

## 🛠️ MCP 服务器配置

### 常用官方服务器

| 服务器包                                    | 功能         | 典型用途       |
| ------------------------------------------- | ------------ | -------------- |
| `@modelcontextprotocol/server-filesystem`   | 文件系统操作 | 读写本地文件   |
| `@modelcontextprotocol/server-github`       | GitHub 集成  | 管理 Issue、PR |
| `@modelcontextprotocol/server-postgres`     | PostgreSQL   | 数据库查询     |
| `@modelcontextprotocol/server-sqlite`       | SQLite       | 轻量数据库     |
| `@modelcontextprotocol/server-brave-search` | Brave 搜索   | 网络搜索       |
| `@modelcontextprotocol/server-puppeteer`    | 浏览器控制   | 网页抓取、测试 |

### 配置示例

**Claude Desktop 配置文件路径：**

- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%APPDATA%\Claude\claude_desktop_config.json`

**配置示例：**

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/Users/username/projects"
      ]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_xxxx"
      }
    },
    "postgres": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-postgres",
        "postgresql://user:pass@localhost:5432/mydb"
      ]
    }
  }
}
```

### 自定义服务器开发

**TypeScript 服务器示例：**

```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new Server(
  { name: "my-server", version: "1.0.0" },
  { capabilities: { tools: {} } },
);

// 定义工具
server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [
    {
      name: "my_tool",
      description: "执行自定义操作",
      inputSchema: {
        type: "object",
        properties: {
          param: { type: "string", description: "参数" },
        },
        required: ["param"],
      },
    },
  ],
}));

// 处理工具调用
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  if (request.params.name === "my_tool") {
    const param = request.params.arguments.param;
    // 执行操作
    return { content: [{ type: "text", text: `结果: ${param}` }] };
  }
  throw new Error(`Unknown tool: ${request.params.name}`);
});

// 启动服务器
const transport = new StdioServerTransport();
await server.connect(transport);
```

**Python 服务器示例：**

```python
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp.types import Tool, TextContent

server = Server("my-server")

@server.list_tools()
async def list_tools():
    return [
        Tool(
            name="my_tool",
            description="执行自定义操作",
            inputSchema={
                "type": "object",
                "properties": {
                    "param": {"type": "string", "description": "参数"}
                },
                "required": ["param"]
            }
        )
    ]

@server.call_tool()
async def call_tool(name: str, arguments: dict):
    if name == "my_tool":
        param = arguments["param"]
        return [TextContent(type="text", text=f"结果: {param}")]
    raise ValueError(f"Unknown tool: {name}")

async def main():
    async with stdio_server() as (read, write):
        await server.run(read, write)

if __name__ == "__main__":
    import asyncio
    asyncio.run(main())
```

---

## 📦 Skill 系统详解

### 什么是 Skill

> [!NOTE] 定义
> **Skill** 是一种封装特定领域知识和工作流程的模块化机制，通过结构化的指令文件赋能 AI 执行专业任务。

Skill 与 MCP 的关系：

- **MCP** 提供工具调用能力（**怎么做**）
- **Skill** 提供领域知识和流程（**做什么、何时做**）

### Skill 文件结构

```
.agent/skills/
└── kubernetes-deploy/          # Skill 目录
    ├── SKILL.md                # 主指令文件（必需）
    ├── scripts/                # 辅助脚本
    │   ├── deploy.sh
    │   └── validate.py
    ├── examples/               # 参考示例
    │   ├── deployment.yaml
    │   └── service.yaml
    └── resources/              # 模板和资源
        ├── base-config.yaml
        └── checklist.md
```

### SKILL.md 规范

````yaml
---
name: Kubernetes Deployment
description: 指导 AI 进行 Kubernetes 应用部署
---

# Kubernetes 部署技能

## 触发条件
当用户请求部署应用到 Kubernetes 时使用此技能。

## 前置检查
1. 确认 kubectl 已安装并配置
2. 验证集群连接：`kubectl cluster-info`
3. 确认目标命名空间存在

## 部署流程

### 步骤1：创建 Namespace
```bash
kubectl create namespace <app-name> --dry-run=client -o yaml | kubectl apply -f -
````

### 步骤2：部署 ConfigMap 和 Secret

- 使用 `examples/` 目录下的模板
- 替换环境变量占位符

### 步骤3：部署应用

```bash
kubectl apply -f deployment.yaml -n <namespace>
```

### 步骤4：验证部署

```bash
kubectl rollout status deployment/<app-name> -n <namespace>
kubectl get pods -n <namespace>
```

## 错误处理

- ImagePullBackOff：检查镜像名称和拉取权限
- CrashLoopBackOff：查看 Pod 日志排查启动问题

## 注意事项

- 生产环境必须设置资源限制
- 确保 liveness 和 readiness 探针配置正确

````

### Skill 开发最佳实践

**1. 明确触发条件**
```markdown
## 触发条件
- 用户请求 "部署 xxx 到 Kubernetes"
- 用户请求 "创建 K8s 部署配置"
- 关键词匹配：deploy, kubernetes, k8s, 部署
````

**2. 结构化步骤**

```markdown
## 执行步骤

// turbo-all <- 标记可自动执行的步骤

1. [前置检查] 验证环境配置
2. [资源创建] 创建 Namespace
3. [部署] 应用 Deployment
4. [验证] 检查部署状态
5. [输出] 生成部署报告
```

**3. 提供示例模板**

```yaml
# examples/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ${APP_NAME}
spec:
  replicas: ${REPLICAS}
  template:
    spec:
      containers:
        - name: ${APP_NAME}
          image: ${IMAGE}
          resources:
            requests:
              memory: "128Mi"
              cpu: "100m"
            limits:
              memory: "256Mi"
              cpu: "200m"
```

**4. 错误处理指南**

```markdown
## 常见错误处理

| 错误             | 原因         | 解决方案                |
| ---------------- | ------------ | ----------------------- |
| ImagePullBackOff | 镜像拉取失败 | 检查镜像名、权限        |
| CrashLoopBackOff | 容器启动失败 | 查看日志 `kubectl logs` |
| Pending          | 资源不足     | 检查节点资源            |
```

### 团队 Skill 库建设

```mermaid
flowchart TD
    subgraph Create["创建阶段"]
        C1[识别高频场景]
        C2[提炼最佳实践]
        C3[编写 SKILL.md]
    end

    subgraph Use["使用阶段"]
        U1[加载 Skill]
        U2[AI 遵循指南]
        U3[完成任务]
    end

    subgraph Maintain["维护阶段"]
        M1[收集反馈]
        M2[持续优化]
        M3[版本管理]
    end

    Create --> Use --> Maintain
    Maintain -.->|迭代| Create

    style Create fill:#e6f3ff
    style Use fill:#fff2e6
    style Maintain fill:#e6ffe6
```

**推荐的团队 Skill 分类：**

| 类别         | 示例 Skill                      | 用途         |
| ------------ | ------------------------------- | ------------ |
| **代码规范** | code-review, naming-convention  | 保证代码质量 |
| **开发流程** | feature-development, bug-fix    | 标准化流程   |
| **部署运维** | kubernetes-deploy, docker-build | 运维自动化   |
| **文档生成** | api-docs, readme-template       | 文档标准化   |
| **测试策略** | unit-test, integration-test     | 测试覆盖     |

---

## 🎯 实战案例

### 案例一：数据库访问 MCP

**场景：** 让 AI 直接查询和分析数据库

**配置：**

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-postgres",
        "postgresql://readonly:password@localhost:5432/analytics"
      ]
    }
  }
}
```

**使用示例：**

```
用户：分析过去 7 天的订单趋势

AI 执行：
1. 调用 query 工具查询订单数据
2. 分析数据趋势
3. 生成可视化图表描述
4. 提供业务洞察
```

### 案例二：Kubernetes 操作 Skill

**Skill 结构：**

```
.agent/skills/kubernetes-ops/
├── SKILL.md
├── scripts/
│   ├── get-pods.sh
│   ├── get-logs.sh
│   └── port-forward.sh
└── examples/
    ├── debug-checklist.md
    └── common-issues.md
```

**SKILL.md 核心内容：**

```markdown
---
name: Kubernetes Operations
description: K8s 集群操作和故障排查
---

# K8s 运维技能

## 资源状态检查

1. `kubectl get pods -A` - 查看所有 Pod
2. `kubectl get events --sort-by='.lastTimestamp'` - 查看事件

## 故障排查流程

1. 定位问题 Pod
2. 查看 Pod 描述：`kubectl describe pod <name>`
3. 查看容器日志：`kubectl logs <pod> -c <container>`
4. 进入容器调试：`kubectl exec -it <pod> -- /bin/sh`

## 常用操作

- 重启 Deployment：`kubectl rollout restart deployment/<name>`
- 扩缩容：`kubectl scale deployment/<name> --replicas=<n>`
```

### 案例三：API 文档生成 Skill

**使用场景：** 根据代码自动生成 API 文档

**SKILL.md：**

```markdown
---
name: API Documentation
description: 生成 OpenAPI 格式的 API 文档
---

# API 文档生成

## 分析步骤

1. 扫描所有路由定义文件
2. 提取 HTTP 方法、路径、参数
3. 分析请求/响应模型
4. 识别认证要求

## 文档格式

使用 OpenAPI 3.0 规范生成：

- 基本信息（title, version）
- 路径定义
- 请求体 Schema
- 响应 Schema
- 安全定义

## 输出模板

见 `examples/openapi-template.yaml`
```

---

## 📊 MCP vs Skill 对比

| 维度         | MCP                  | Skill            |
| ------------ | -------------------- | ---------------- |
| **本质**     | 工具调用协议         | 知识封装模块     |
| **解决问题** | 如何调用外部工具     | 如何执行专业任务 |
| **复用级别** | 工具级别             | 流程级别         |
| **维护方**   | 社区/官方            | 团队/个人        |
| **学习成本** | 较高（需开发服务器） | 较低（编写文档） |

**协同使用：**

```mermaid
flowchart LR
    Skill[📦 Skill<br/>定义流程] --> Agent[🤖 Agent<br/>规划执行]
    Agent --> MCP[🔌 MCP<br/>调用工具]
    MCP --> Result[✅ 完成任务]

    style Skill fill:#e6ffe6
    style Agent fill:#e6f3ff
    style MCP fill:#fff2e6
```

---

## 🔗 相关资源

### 官方资源

- [MCP 官方文档](https://modelcontextprotocol.io/)
- [MCP 服务器列表](https://github.com/modelcontextprotocol/servers)
- [MCP TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk)
- [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk)

### 相关文档

- [[Agent-MCP-Skill核心概念]] - 核心概念详解
- [[Agentic Coding实践指南]] - Agent 模式实战
- [[Vibe Coding提效指南]] - AI 编码基础方法论
- [[AI编码工具对比分析]] - 工具选型参考
- [[MOC - AI编码工具]] - 返回索引

---

_最后更新：2026-01-26_
