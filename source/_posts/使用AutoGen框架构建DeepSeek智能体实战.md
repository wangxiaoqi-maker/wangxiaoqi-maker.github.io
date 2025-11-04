---
title: AutoGen智能体开发实战（基础篇）：从零构建你的第一个AI智能体
date: 2025-11-01 18:00:00
tags:
  - Python
  - AutoGen
  - AI智能体
  - 异步编程
  - LLM
categories: AutoGen
comments: true
cover: https://images.unsplash.com/photo-1677442136019-21780ecad995?w=800&q=80
abbrlink: 9
---

# 前言

在AI应用开发中，**智能体（Agent）** 是一个非常重要的概念。微软开源的 **AutoGen** 框架为我们提供了强大的智能体开发能力，支持多种大模型（如 DeepSeek、千问、ChatGPT、Claude 等）。

本文是 **AutoGen智能体开发系列** 的第一篇，将从零开始介绍如何使用 AutoGen 框架接入大模型，构建一个支持流式输出的基础智能体应用。

后续文章将会介绍如何基于 **AutoGen + FastAPI** 构建完整的测试平台，包括测试用例生成、自动化用例生成等实战功能。让我们先从基础开始吧！

---

## 什么是 AutoGen？

**AutoGen** 是由微软开发的一个开源框架，用于构建和编排多智能体系统。它的核心优势包括：

- 🤖 **多智能体协作**：支持多个AI智能体之间的对话和协作
- 🔄 **灵活的对话模式**：支持人机对话、智能体间对话等多种模式
- 🎯 **任务自动化**：可以让智能体自主完成复杂任务
- 🔌 **模型无关**：支持OpenAI、DeepSeek、千问、Claude等多种大模型
- ⚡ **异步支持**：原生支持Python异步编程

---

## 项目架构

我们的项目分为两个核心模块：

```
backend/
├── llms.py          # 模型客户端配置
├── chat.py          # 智能体实现
└── .Env             # 环境变量配置
```

### 架构设计思路

1. **llms.py**：负责创建和配置大模型客户端（支持多种LLM）
2. **chat.py**：使用AssistantAgent创建智能体并实现对话功能
3. **.Env**：存储敏感信息（API Key等）

> 💡 **提示**：本文以 DeepSeek 为例进行演示，但代码同样适用于其他兼容 OpenAI API 格式的大模型（如千问、ChatGPT、Claude 等），只需修改环境变量配置即可。

---

## 环境准备

### 1. 安装依赖

```bash
pip install autogen-agentchat autogen-ext python-dotenv
```

### 2. 创建 .Env 文件

在项目根目录创建 `.Env` 文件，配置你想使用的大模型：

**示例1：使用 DeepSeek**
```env
MODEL=deepseek-chat
API_KEY=your-deepseek-api-key
BASE_URL=https://api.deepseek.com
```

**示例2：使用千问（通义千问）**
```env
MODEL=qwen-plus
API_KEY=your-qwen-api-key
BASE_URL=https://dashscope.aliyuncs.com/compatible-mode/v1
```

**示例3：使用 OpenAI ChatGPT**
```env
MODEL=gpt-4
API_KEY=your-openai-api-key
BASE_URL=https://api.openai.com/v1
```

**配置说明**：
- `MODEL`: 模型名称（根据服务商文档填写）
- `API_KEY`: 你的 API 密钥
- `BASE_URL`: API 地址（OpenAI 兼容格式）

---

## 核心代码实现

### 模块一：模型客户端配置 (llms.py)

这个模块负责创建 OpenAI 兼容的大模型客户端：

```python
import os
from dotenv import load_dotenv
from autogen_ext.models.openai import OpenAIChatCompletionClient
from autogen_core.models import UserMessage, SystemMessage

# 加载环境变量
load_dotenv('.Env')


def get_client():
    """
    创建OpenAI兼容的大模型客户端
    支持 DeepSeek、千问、ChatGPT、Claude 等多种模型
    
    Returns:
        OpenAIChatCompletionClient: 配置好的模型客户端
    """
    openai_model_client = OpenAIChatCompletionClient(
        model=os.getenv("MODEL"),           # 模型名称（从环境变量读取）
        api_key=os.getenv("API_KEY"),       # API密钥
        base_url=os.getenv("BASE_URL"),     # API地址
        model_info={
            "vision": True,                  # 视觉能力
            "function_calling": True,        # 函数调用
            "json_output": True,             # JSON输出
            "family": "Unknown",             # 模型家族
            "structured_output": True,       # 结构化输出
            "multiple_system_messages": True,# 多系统消息
        },
    )
    return openai_model_client


# 创建全局客户端实例
client = get_client()
```

**关键点解析**：

1. **环境变量管理**：使用 `python-dotenv` 加载配置，提高安全性
2. **模型能力配置**：通过 `model_info` 定义模型支持的功能
3. **OpenAI兼容**：大部分主流大模型（DeepSeek、千问、Claude等）都兼容 OpenAI API 格式
4. **灵活切换**：只需修改 `.Env` 文件即可切换不同的大模型，无需改动代码

---

### 模块二：智能体实现 (chat.py)

这个模块实现了基于AutoGen的智能体：

```python
import asyncio
import os
from autogen_agentchat.agents import AssistantAgent
from autogen_agentchat.messages import TextMessage, ModelClientStreamingChunkEvent
from autogen_ext.models.openai import OpenAIChatCompletionClient
from dotenv import load_dotenv

# 加载环境变量
load_dotenv('.Env')

# 创建模型客户端
model_client = OpenAIChatCompletionClient(
    model=os.getenv("MODEL"),
    api_key=os.getenv("API_KEY"),
    base_url=os.getenv("BASE_URL"),
    model_info={
        "vision": True,
        "function_calling": True,
        "json_output": True,
        "family": "Unknown",
        "structured_output": True,
        "multiple_system_messages": True,
    },
)

# 创建助手智能体
agent = AssistantAgent(
    name="assistant",
    model_client=model_client,
    system_message="你是一名测试工程师",
    model_client_stream=True  # 启用流式输出
)


async def chat():
    """
    非流式对话示例
    直接获取完整的回复内容
    """
    result = await agent.run(task="介绍一下你自己")
    print(result.messages)


async def chat_stream():
    """
    流式对话示例
    实现打字机效果，逐字输出
    """
    result = agent.run_stream(task="编写一首四言古诗")
    
    # 异步遍历流式输出
    async for message in result:
        if isinstance(message, ModelClientStreamingChunkEvent):
            print(message.content, end="", flush=True)


if __name__ == '__main__':
    # 运行流式对话
    asyncio.run(chat_stream())
```

**核心特性**：

1. **AssistantAgent**：AutoGen的核心智能体类
2. **system_message**：定义智能体的角色和行为
3. **流式输出**：通过 `run_stream()` 实现打字机效果
4. **异步编程**：使用 `async/await` 提高性能

---

## 功能详解

### 1. 非流式对话 (chat)

适用场景：需要等待完整回复的场景

```python
async def chat():
    result = await agent.run(task="介绍一下你自己")
    print(result.messages)
```

**特点**：
- ✅ 一次性返回完整结果
- ✅ 适合需要完整处理的场景
- ❌ 用户需要等待较长时间

### 2. 流式对话 (chat_stream)

适用场景：需要实时反馈的交互场景

```python
async def chat_stream():
    result = agent.run_stream(task="编写一首四言古诗")
    
    async for message in result:
        if isinstance(message, ModelClientStreamingChunkEvent):
            print(message.content, end="", flush=True)
```

**特点**：
- ✅ 逐字输出，用户体验好
- ✅ 减少等待时间感
- ✅ 适合聊天机器人场景

**流式输出原理**：

```
用户提问 → 智能体处理 → 逐块返回内容
                          ↓
        ModelClientStreamingChunkEvent
                          ↓
                    实时显示给用户
```

---

## 总结

本文作为 **AutoGen智能体开发系列** 的基础篇，详细介绍了：

✅ AutoGen框架基础概念与优势  
✅ 环境搭建和依赖安装  
✅ 支持多种大模型的客户端配置（DeepSeek、千问、ChatGPT等）  
✅ 创建第一个智能体  
✅ 实现流式输出功能  
✅ Python异步编程基础  

通过本文的学习，你应该能够：
1. 理解AutoGen的核心概念和优势
2. 掌握基础的智能体创建和配置
3. 灵活切换不同的大模型服务
4. 实现流式对话功能，提升用户体验
5. 为后续进阶开发打下基础

## 系列预告

这只是系列的开始！接下来我会陆续分享：

📌 **进阶篇**：
- 多智能体协作开发
- 自定义工具和函数调用
- 智能体工作流设计
- 对话历史管理

📌 **实战篇**：
- 基于 AutoGen + FastAPI 构建测试平台
- 智能测试用例生成器
- 自动化测试用例生成
- 测试报告智能分析
- 多模型对比和切换实践

敬请期待！💪

---

## 参考资源

**框架文档**：
- [AutoGen 官方文档](https://microsoft.github.io/autogen/)
- [Python异步编程指南](https://docs.python.org/zh-cn/3/library/asyncio.html)

**大模型API文档**：
- [DeepSeek API](https://platform.deepseek.com/docs)
- [阿里云千问 API](https://help.aliyun.com/zh/dashscope/)
- [OpenAI API](https://platform.openai.com/docs)
- [Claude API](https://docs.anthropic.com/)

---

## 项目源码

完整代码已在文中展示，你可以直接复制使用。记得：
1. 安装必要的依赖包
2. 根据你选择的大模型配置 `.Env` 文件
3. 替换为你自己的 API Key
4. 根据需要切换不同的大模型服务

---

> 💡 **温馨提示**: 这是AutoGen系列的基础篇，重点是让你快速上手。后续文章会深入更多高级特性！

> 🔥 **推荐阅读**: [《Python调用DeepSeek API实战教程》](/2025/11/01/Python调用DeepSeek-API实战教程/) - 了解如何直接调用大模型API

> 📚 **系列文章**: 如果你对本系列感兴趣，欢迎关注后续的进阶篇和实战篇！
