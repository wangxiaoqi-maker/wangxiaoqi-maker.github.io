---
title: AutoGen GraphFlow工作流实战：构建智能协作管道
date: 2025-12-08 21:00:00
tags:
  - Python
  - AutoGen
  - AI智能体
  - 工作流
  - GraphFlow
categories: AutoGen
comments: true
cover: https://images.unsplash.com/photo-1558494949-ef010cbdcc31?w=800&q=80
abbrlink: 15
---

# 前言

在之前的 AutoGen 系列文章中，我们学习了如何创建单个智能体和多智能体协作。但在实际业务场景中，我们往往需要更复杂的**工作流编排**能力——让多个智能体按照特定的顺序和逻辑协同工作。

本文将介绍 AutoGen 框架中强大的 **GraphFlow** 工作流机制，通过构建有向图（DAG）来编排复杂的多智能体协作流程。我们将实现一个**内容创作管道**：写作 → 并行编辑（语法+风格） → 最终审核。

---

## 什么是 GraphFlow？

**GraphFlow** 是 AutoGen 提供的基于图的工作流编排方案。它允许你：

- 🔀 **有向图编排**：使用 DiGraphBuilder 构建复杂的工作流拓扑
- 📊 **并行处理**：支持 Fan-out 模式，让多个智能体并行工作
- 🔗 **聚合结果**：支持 Fan-in 模式，汇总多路结果
- ⚡ **流式输出**：实时展示工作流执行过程
- 🎯 **灵活扩展**：轻松添加新的节点和边

### 适用场景

GraphFlow 特别适合以下场景：

| 场景 | 说明 |
|------|------|
| **内容创作流水线** | 写作 → 多维度审核 → 终审 |
| **代码审查系统** | 生成代码 → 安全检查 + 性能检查 → 合并 |
| **测试用例生成** | 需求分析 → 多类型用例生成 → 整合 |
| **文档翻译** | 翻译 → 多人校对 → 统稿 |

---

## 项目架构

```
project/
├── backend/
│   └── llms.py          # 模型客户端配置
├── test_graph_flow.py   # GraphFlow 工作流实现
└── .Env                 # 环境变量配置
```

### 工作流设计

我们将构建以下工作流：

```
                    ┌──────────┐
                    │  writer  │
                    │  (写作)   │
                    └────┬─────┘
                         │
            ┌────────────┼────────────┐
            │                         │
            ▼                         ▼
     ┌──────────┐              ┌──────────┐
     │ editor1  │              │ editor2  │
     │ (语法)   │              │ (风格)   │
     └────┬─────┘              └────┬─────┘
            │                         │
            └────────────┬────────────┘
                         │
                         ▼
                  ┌──────────────┐
                  │final_reviewer│
                  │   (终审)      │
                  └──────────────┘
```

这是一个典型的 **Fan-out/Fan-in** 模式：
- **Fan-out**：writer 的输出同时发送给 editor1 和 editor2
- **Fan-in**：editor1 和 editor2 的输出汇聚到 final_reviewer

---

## 环境准备

### 1. 安装依赖

```bash
pip install autogen-agentchat autogen-ext python-dotenv
```

### 2. 配置环境变量

创建 `.Env` 文件：

```env
MODEL=deepseek-chat
API_KEY=your-api-key
BASE_URL=https://api.deepseek.com
```

> 💡 同样支持千问、ChatGPT、Claude 等模型，只需修改配置即可。

---

## 核心代码实现

### 完整代码

```python
import asyncio

from autogen_agentchat.agents import AssistantAgent
from autogen_agentchat.teams import DiGraphBuilder, GraphFlow
from autogen_agentchat.ui import Console
from dotenv import load_dotenv

load_dotenv('.Env')
from backend.llms import get_client

# 创建模型客户端
client = get_client()

# ========== 1. 创建智能体 ==========

# 写作智能体：负责起草初稿
writer = AssistantAgent(
    "writer", 
    model_client=client, 
    system_message="起草一段关于气候变化的短文。"
)

# 语法编辑智能体：专注于语法改进
editor1 = AssistantAgent(
    "editor1", 
    model_client=client, 
    system_message="编辑段落以改进语法。"
)

# 风格编辑智能体：专注于风格优化
editor2 = AssistantAgent(
    "editor2", 
    model_client=client, 
    system_message="编辑段落以改进风格。"
)

# 终审智能体：整合所有编辑意见
final_reviewer = AssistantAgent(
    "final_reviewer",
    model_client=client,
    system_message="整合语法和风格编辑，形成最终版本。",
)

# ========== 2. 构建工作流图 ==========

builder = DiGraphBuilder()

# 添加所有节点
builder.add_node(writer) \
       .add_node(editor1) \
       .add_node(editor2) \
       .add_node(final_reviewer)

# 构建边：Fan-out（从 writer 分发到两个 editor）
builder.add_edge(writer, editor1)
builder.add_edge(writer, editor2)

# 构建边：Fan-in（从两个 editor 汇聚到 final_reviewer）
builder.add_edge(editor1, final_reviewer)
builder.add_edge(editor2, final_reviewer)

# 构建并验证图
graph = builder.build()

# ========== 3. 创建工作流 ==========

flow = GraphFlow(
    participants=builder.get_participants(),
    graph=graph,
)

# ========== 4. 运行工作流 ==========

async def main():
    await Console(flow.run_stream(task="写一段关于气候变化的短文。"))

asyncio.run(main())
```

---

## 代码详解

### 1. 创建智能体

每个智能体都有明确的职责：

```python
writer = AssistantAgent(
    "writer",                                    # 智能体名称
    model_client=client,                         # 模型客户端
    system_message="起草一段关于气候变化的短文。"  # 系统提示词
)
```

**职责分工**：

| 智能体 | 职责 | 系统提示词 |
|--------|------|-----------|
| writer | 创作初稿 | 起草一段关于气候变化的短文 |
| editor1 | 语法编辑 | 编辑段落以改进语法 |
| editor2 | 风格编辑 | 编辑段落以改进风格 |
| final_reviewer | 终审整合 | 整合语法和风格编辑，形成最终版本 |

### 2. 构建工作流图

使用 **DiGraphBuilder** 构建有向无环图（DAG）：

```python
builder = DiGraphBuilder()

# 添加节点（链式调用）
builder.add_node(writer) \
       .add_node(editor1) \
       .add_node(editor2) \
       .add_node(final_reviewer)

# 添加边（定义执行顺序）
builder.add_edge(writer, editor1)      # writer → editor1
builder.add_edge(writer, editor2)      # writer → editor2
builder.add_edge(editor1, final_reviewer)  # editor1 → final_reviewer
builder.add_edge(editor2, final_reviewer)  # editor2 → final_reviewer
```

**核心概念**：

- `add_node()`：添加工作流节点（智能体）
- `add_edge(source, target)`：添加有向边，定义执行依赖
- `build()`：构建并验证图结构

### 3. 创建 GraphFlow

```python
flow = GraphFlow(
    participants=builder.get_participants(),  # 所有参与者
    graph=graph,                              # 工作流图
)
```

**GraphFlow** 会根据图结构自动：
- 确定执行顺序（拓扑排序）
- 并行执行无依赖的节点
- 等待依赖节点完成后再继续

### 4. 流式运行

```python
async def main():
    await Console(flow.run_stream(task="写一段关于气候变化的短文。"))
```

- `run_stream()`：流式执行工作流
- `Console()`：在控制台实时显示执行过程

---

## 执行流程解析

当运行这个工作流时，执行流程如下：

```
1️⃣ 任务开始
   ↓
2️⃣ writer 接收任务，生成初稿
   ↓
3️⃣ Fan-out: 初稿同时发送给 editor1 和 editor2
   ↓
4️⃣ editor1 和 editor2 并行工作
   │
   ├─ editor1: 专注语法修改
   │
   └─ editor2: 专注风格优化
   ↓
5️⃣ Fan-in: 两个编辑的结果汇聚
   ↓
6️⃣ final_reviewer 整合所有意见，输出最终版本
   ↓
7️⃣ 任务完成
```

---

## 进阶应用

### 1. 添加条件分支

你可以根据条件动态选择执行路径：

```python
# 添加条件边
builder.add_conditional_edge(
    source=writer,
    condition=lambda msg: "紧急" in msg.content,
    true_target=urgent_reviewer,
    false_target=normal_reviewer
)
```

### 2. 循环处理

实现迭代优化：

```python
# 添加反馈循环
builder.add_edge(reviewer, writer)  # reviewer 可以让 writer 重写
```

### 3. 更复杂的拓扑

```python
# 多层 Fan-out/Fan-in
builder.add_edge(writer, reviewer1)
builder.add_edge(writer, reviewer2)
builder.add_edge(writer, reviewer3)

builder.add_edge(reviewer1, aggregator)
builder.add_edge(reviewer2, aggregator)
builder.add_edge(reviewer3, aggregator)
```

---

## 实际应用场景

### 场景1：测试用例生成管道

```python
# 需求分析 → 多类型用例生成 → 整合
analyzer = AssistantAgent("analyzer", ..., system_message="分析测试需求")
func_tester = AssistantAgent("func_tester", ..., system_message="生成功能测试用例")
perf_tester = AssistantAgent("perf_tester", ..., system_message="生成性能测试用例")
sec_tester = AssistantAgent("sec_tester", ..., system_message="生成安全测试用例")
integrator = AssistantAgent("integrator", ..., system_message="整合所有测试用例")

builder.add_edge(analyzer, func_tester)
builder.add_edge(analyzer, perf_tester)
builder.add_edge(analyzer, sec_tester)
builder.add_edge(func_tester, integrator)
builder.add_edge(perf_tester, integrator)
builder.add_edge(sec_tester, integrator)
```

## 总结

本文介绍了 AutoGen 的 **GraphFlow** 工作流机制：

✅ **DiGraphBuilder** 构建有向图  
✅ **Fan-out** 并行分发任务  
✅ **Fan-in** 汇聚多路结果  
✅ **流式输出** 实时展示进度  
✅ **灵活扩展** 适应复杂场景  

通过 GraphFlow，你可以轻松构建复杂的多智能体协作管道，实现：
- 内容创作流水线
- 测试用例生成系统

---

## 系列回顾

📌 **基础篇**：[AutoGen智能体开发实战（基础篇）](/2025/11/01/使用AutoGen框架构建DeepSeek智能体实战/)

📌 **本文**：GraphFlow 工作流实战

📌 **延伸阅读**：
- [深入理解AutoGen消息机制](/2025/11/01/深入理解AutoGen消息机制-发布订阅与点对点通信/)
- [AutoGen多智能体协作实战](/2025/11/01/AutoGen多智能体协作实战-构建智能测试用例生成器/)

---

## 参考资源

**官方文档**：
- [AutoGen GraphFlow 文档](https://microsoft.github.io/autogen/docs/tutorial/graph)
- [DiGraphBuilder API](https://microsoft.github.io/autogen/docs/reference/agentchat/teams/DiGraphBuilder)

**相关概念**：
- [有向无环图（DAG）](https://zh.wikipedia.org/wiki/有向无环图)
- [拓扑排序](https://zh.wikipedia.org/wiki/拓扑排序)

---

> 💡 **提示**：GraphFlow 让复杂的多智能体协作变得简单直观。通过可视化的图结构，你可以清晰地设计和理解工作流逻辑。

> 🚀 **下一步**：尝试将 GraphFlow 应用到你的实际项目中，构建属于你自己的智能协作管道！
