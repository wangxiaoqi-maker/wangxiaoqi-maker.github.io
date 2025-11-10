---
title: AutoGen多智能体协作实战：构建智能测试用例生成器（进阶篇）
date: 2025-11-02 10:00:00
tags:
  - Python
  - AutoGen
  - AI智能体
  - 测试用例
  - 多智能体协作
categories: AutoGen
comments: true
cover: https://images.unsplash.com/photo-1551288049-bebda4e38f71?w=800&q=80
abbrlink: 10
---

# 前言

在上一篇 [《AutoGen智能体开发实战（基础篇）》](/posts/9/) 中，我们学习了如何创建单个智能体。但 AutoGen 的真正威力在于**多智能体协作**——让多个AI智能体相互配合，共同完成复杂任务。

本文是 **AutoGen智能体开发系列** 的第二篇，将介绍如何使用 AutoGen 的 **Team** 功能，构建一个由两个智能体协作的测试用例生成系统：
- 🤖 **测试工程师智能体**：负责编写测试用例
- 🔍 **质量评审专家智能体**：负责评审和优化测试用例

通过这个实战案例，你将学会如何设计多智能体协作流程，打造真正实用的AI工具！

---

## 为什么需要多智能体协作？

在实际工作中，复杂任务往往需要多个角色协作：

### 单智能体的局限性

❌ **缺乏自我检查**：无法发现自己的错误和遗漏  
❌ **思维单一**：缺少多角度审视  
❌ **质量不稳定**：没有质量把关机制  

### 多智能体协作的优势

✅ **分工明确**：每个智能体专注于自己擅长的领域  
✅ **互相检查**：通过评审机制提高输出质量  
✅ **持续优化**：多轮对话逐步完善结果  
✅ **模拟真实团队**：还原实际工作流程  

---

## 系统架构设计

### 整体架构

```
用户需求
   ↓
测试工程师智能体 (Primary Agent)
   ├─ 分析需求
   ├─ 设计测试用例
   └─ 输出初版测试用例
   ↓
质量评审专家智能体 (Critic Agent)
   ├─ 检查完整性
   ├─ 评估覆盖度
   ├─ 提出改进建议
   └─ 是否通过？
       ├─ Yes → 输出 APPROVE（结束）
       └─ No  → 返回改进建议
           ↓
       测试工程师智能体
           └─ 根据建议修改
               ↓
           （循环直到通过）
```

### 技术选型

| 组件 | 技术 | 说明 |
|------|------|------|
| 智能体框架 | AutoGen | 多智能体协作 |
| 团队模式 | RoundRobinGroupChat | 轮流发言 |
| 终止条件 | TextMentionTermination | 关键词触发 |
| 输出方式 | Stream | 流式输出 |

---

## 核心代码实现

### 完整代码

```python
import asyncio

from autogen_agentchat.agents import AssistantAgent
from autogen_agentchat.conditions import TextMentionTermination
from autogen_agentchat.messages import ModelClientStreamingChunkEvent
from autogen_agentchat.teams import RoundRobinGroupChat

from backend.llms import get_client

# ============================================
# 智能体1：测试工程师
# ============================================
primary_agent = AssistantAgent(
    "primary",
    model_client=get_client(),
    model_client_stream=True,
    system_message="""
    角色：
你是一名资深的软件测试工程师，专注于设计全面、高效和可重复的测试用例。你的任务是基于给定的需求或功能描述，生成结构化测试用例，以确保软件质量。

任务：
请为以下功能或系统设计一组测试用例。测试用例应覆盖正常场景、异常场景和边界条件，并遵循标准的测试设计技术。

输出要求：
测试用例应以表格形式呈现，包括以下列：

测试用例ID：唯一标识符（例如：TC_LOGIN_001）
测试用例描述：简要说明测试目的
前置条件：执行测试前需要满足的条件
测试步骤：详细、可操作的步骤列表
预期结果：每个步骤后或整体的预期输出
实际结果（可选）：用于执行后记录
优先级：高/中/低（基于业务影响）
测试类型：功能、性能、安全等
备注：任何额外说明，如依赖项或风险

指导原则：

使用测试设计技术：
- 等价类划分：将输入数据划分为有效和无效类。
- 边界值分析：测试输入范围的边界（如最小、最大值）。
- 决策表：针对复杂逻辑，覆盖所有条件组合。
- 状态转换：适用于有状态变化的功能。

覆盖场景：
- 正常流程（有效输入）
- 异常流程（无效输入、错误处理）
- 边缘情况（如超时、网络中断）

确保测试用例可重复、独立且易于理解。
优先覆盖高风险和高频使用场景。
    """,
)

# ============================================
# 智能体2：质量评审专家
# ============================================
critic_agent = AssistantAgent(
    "critic",
    model_client=get_client(),
    model_client_stream=True,
    system_message="""
    角色：
你是一名资深的质量保障专家，负责对已编写的测试用例进行系统性评审。你的目标是发现测试用例在设计、覆盖面和可执行性方面的缺陷与遗漏，确保其能有效保障软件质量。

任务：
请对提供的测试用例集进行全面的同行评审，并从多个维度评估其质量，提出具体、可操作的改进建议。

评审维度：

1. 正确性与完整性
   • 测试用例是否100%覆盖了需求/验收标准？
   • 所有功能路径和业务规则是否都被测试到？
   • 预期结果是否符合产品设计且无歧义？

2. 场景覆盖度
   • 正常流程：是否覆盖了主要的"Happy Path"？
   • 异常流程：是否覆盖了无效输入、错误操作、异常数据、网络故障等场景？
   • 边界值：是否对输入字段进行了边界测试（如最小值、最大值、空值）？

3. 设计与结构
   • 测试用例独立性：每个用例是否可以独立运行？
   • 清晰度与可执行性：测试步骤是否清晰、明确、无歧义？
   • 数据依赖：测试数据是否被明确定义？

4. 可维护性
   • 用例标识：测试用例ID是否唯一且易于识别？
   • 语言与表述：描述和步骤是否简洁、专业？
   • 冗余度：是否存在重复的测试用例？

如果用例比较满意，请回复APPROVE
    """,
)

# ============================================
# 创建团队和终止条件
# ============================================
# 当检测到"APPROVE"关键词时终止对话
text_termination = TextMentionTermination("APPROVE")

# 创建轮流发言的团队
team = RoundRobinGroupChat(
    [primary_agent, critic_agent], 
    termination_condition=text_termination
)

# 运行团队任务（流式输出）
result = team.run_stream(task="编写5条登录的测试用例")


# ============================================
# 主函数：处理流式输出
# ============================================
async def main():
    async for message in result:
        if isinstance(message, ModelClientStreamingChunkEvent):
            print(message.content, end="", flush=True)


if __name__ == '__main__':
    asyncio.run(main())
```

---

## 核心概念详解

### 1. AssistantAgent（智能体）

**测试工程师智能体**：

```python
primary_agent = AssistantAgent(
    "primary",                      # 智能体名称
    model_client=get_client(),      # 模型客户端
    model_client_stream=True,       # 启用流式输出
    system_message="""..."""        # 角色定义和任务说明
)
```

**关键要素**：
- **name**：智能体的唯一标识
- **model_client**：使用的大模型（支持 DeepSeek、千问、ChatGPT 等）
- **system_message**：定义智能体的角色、能力和输出要求

### 2. RoundRobinGroupChat（轮流对话团队）

```python
team = RoundRobinGroupChat(
    [primary_agent, critic_agent],      # 智能体列表
    termination_condition=text_termination  # 终止条件
)
```

**工作机制**：
1. 智能体按列表顺序依次发言
2. `primary_agent` → `critic_agent` → `primary_agent` → ...
3. 直到满足终止条件

### 3. TextMentionTermination（关键词终止条件）

```python
text_termination = TextMentionTermination("APPROVE")
```

**作用**：
- 当任意智能体的回复中包含 `APPROVE` 时，对话自动结束
- 相当于质量评审专家的"通过"信号

### 4. 流式输出处理

```python
async def main():
    async for message in result:
        if isinstance(message, ModelClientStreamingChunkEvent):
            print(message.content, end="", flush=True)
```

**优势**：
- 实时看到智能体的思考过程
- 更好的用户体验
- 适合长文本输出场景

---

## 系统提示词设计技巧

### 测试工程师的提示词结构

```
角色定义
   ↓
任务说明
   ↓
输出要求（结构化）
   ↓
指导原则（方法论）
```

**关键点**：
1. **明确角色**：资深测试工程师，专注于测试用例设计
2. **清晰任务**：生成结构化测试用例
3. **标准化输出**：要求表格形式，包含固定字段
4. **方法论指导**：等价类、边界值、决策表等测试技术

### 质量评审专家的提示词结构

```
角色定义
   ↓
任务说明
   ↓
评审维度（检查清单）
   ↓
终止信号（APPROVE）
```

**关键点**：
1. **批判性角色**：质量保障专家，负责发现问题
2. **多维度评审**：正确性、覆盖度、可维护性等
3. **具体建议**：不仅指出问题，还要给出改进方案
4. **明确终止**：满意时输出 `APPROVE`

---

## 运行效果展示

### 执行流程

```bash
$ python teams.py
```

### 典型输出流程

**第一轮 - 测试工程师**：
```
好的，我将为登录功能设计5条测试用例...

| 测试用例ID | 测试用例描述 | 前置条件 | 测试步骤 | 预期结果 | 优先级 | 测试类型 |
|-----------|------------|---------|---------|---------|-------|---------|
| TC_LOGIN_001 | 使用有效用户名和密码登录 | ... | ... | ... | 高 | 功能 |
| TC_LOGIN_002 | 使用无效密码登录 | ... | ... | ... | 高 | 功能 |
...
```

**第二轮 - 质量评审专家**：
```
我对这些测试用例进行了评审，发现以下问题：

1. 缺少边界值测试：
   - 建议：增加用户名长度边界测试（最小、最大长度）
   - 建议：增加特殊字符测试

2. 缺少安全性测试：
   - 建议：增加SQL注入测试用例
   - 建议：增加密码强度验证测试

请补充这些测试用例。
```

**第三轮 - 测试工程师**：
```
好的，根据您的建议，我补充了以下测试用例：

| 测试用例ID | 测试用例描述 | ... |
|-----------|------------|-----|
| TC_LOGIN_006 | 用户名长度边界测试 | ... |
| TC_LOGIN_007 | SQL注入安全测试 | ... |
...
```

**第四轮 - 质量评审专家**：
```
测试用例已经覆盖了主要场景，包括正常流程、异常处理、边界值和安全性。
测试用例设计合理，步骤清晰，可以投入使用。

APPROVE
```

**对话结束** ✅

---

## 进阶优化

### 1. 增加更多智能体角色

```python
# 安全专家智能体
security_agent = AssistantAgent(
    "security_expert",
    model_client=get_client(),
    system_message="你是安全测试专家，专注于发现安全漏洞..."
)

# 性能测试专家
performance_agent = AssistantAgent(
    "performance_expert",
    model_client=get_client(),
    system_message="你是性能测试专家，关注系统性能和负载..."
)

# 多角色团队
team = RoundRobinGroupChat(
    [primary_agent, critic_agent, security_agent, performance_agent],
    termination_condition=text_termination
)
```

### 2. 自定义终止条件

```python
from autogen_agentchat.conditions import MaxMessageTermination, TextMentionTermination

# 组合终止条件：达到最大消息数 OR 检测到APPROVE
termination = MaxMessageTermination(10) | TextMentionTermination("APPROVE")

team = RoundRobinGroupChat(
    [primary_agent, critic_agent],
    termination_condition=termination
)
```

### 3. 添加人工审核

```python
from autogen_agentchat.conditions import ExternalTermination

# 外部终止条件（需要人工确认）
external_termination = ExternalTermination()

team = RoundRobinGroupChat(
    [primary_agent, critic_agent],
    termination_condition=external_termination
)

# 在代码中手动触发终止
# await external_termination.set()
```

### 4. 保存对话历史

```python
import json
from datetime import datetime

async def main():
    conversation = []
    
    async for message in result:
        if isinstance(message, ModelClientStreamingChunkEvent):
            content = message.content
            print(content, end="", flush=True)
            
            # 记录对话
            conversation.append({
                "timestamp": datetime.now().isoformat(),
                "content": content
            })
    
    # 保存到文件
    with open(f"test_case_generation_{datetime.now().strftime('%Y%m%d_%H%M%S')}.json", 'w', encoding='utf-8') as f:
        json.dump(conversation, f, ensure_ascii=False, indent=2)
    
    print("\n\n对话已保存！")
```

---

## 实战应用场景

### 1. 测试用例生成

```python
# 不同功能的测试用例生成
tasks = [
    "编写用户注册功能的测试用例",
    "编写购物车功能的测试用例",
    "编写支付功能的测试用例"
]

for task in tasks:
    result = team.run_stream(task=task)
    # 处理结果...
```

### 2. 代码审查

```python
code_reviewer = AssistantAgent(
    "code_reviewer",
    model_client=get_client(),
    system_message="你是资深代码审查专家，专注于发现代码质量问题..."
)

code_improver = AssistantAgent(
    "code_improver",
    model_client=get_client(),
    system_message="你是代码优化专家，根据审查意见改进代码..."
)

review_team = RoundRobinGroupChat(
    [code_improver, code_reviewer],
    termination_condition=TextMentionTermination("LGTM")
)
```

### 3. 文档编写与审校

```python
doc_writer = AssistantAgent(
    "doc_writer",
    model_client=get_client(),
    system_message="你是技术文档编写专家..."
)

doc_reviewer = AssistantAgent(
    "doc_reviewer",
    model_client=get_client(),
    system_message="你是文档审校专家，检查准确性、完整性和可读性..."
)

doc_team = RoundRobinGroupChat(
    [doc_writer, doc_reviewer],
    termination_condition=TextMentionTermination("APPROVE")
)
```

---

## 最佳实践

### 1. 提示词设计原则

✅ **角色清晰**：明确每个智能体的职责  
✅ **任务具体**：详细说明输出要求  
✅ **标准化输出**：使用表格、JSON等结构化格式  
✅ **包含示例**：在提示词中给出优秀示例  
✅ **设置终止信号**：明确告知如何结束对话  

### 2. 团队设计原则

✅ **分工明确**：每个智能体专注一个领域  
✅ **角色互补**：设计者 + 评审者，编码者 + 测试者  
✅ **控制轮次**：设置合理的终止条件，避免无限循环  
✅ **流程可视**：通过日志清晰展示协作过程  

### 3. 性能优化

✅ **模型选择**：评审角色可以使用更强的模型（如 GPT-4）  
✅ **并发处理**：对于独立任务可以并行执行  
✅ **缓存结果**：对于相同需求，复用之前的结果  

### 4. 错误处理

```python
async def main():
    try:
        async for message in result:
            if isinstance(message, ModelClientStreamingChunkEvent):
                print(message.content, end="", flush=True)
    except Exception as e:
        print(f"\n错误：{str(e)}")
        # 记录错误日志
        # 发送告警通知
```

---

## 常见问题

### Q1: 智能体对话陷入循环怎么办？

**A**: 设置多重终止条件：

```python
from autogen_agentchat.conditions import MaxMessageTermination

# 最多10轮对话
max_turns = MaxMessageTermination(10)
keyword_term = TextMentionTermination("APPROVE")

# 满足任一条件即终止
team = RoundRobinGroupChat(
    [primary_agent, critic_agent],
    termination_condition=max_turns | keyword_term
)
```

### Q2: 如何控制智能体的发言顺序？

**A**: 使用 `RoundRobinGroupChat` 时，智能体按列表顺序轮流发言。如果需要更复杂的控制，可以使用 `SelectorGroupChat`：

```python
from autogen_agentchat.teams import SelectorGroupChat

# 自动选择下一个发言者
team = SelectorGroupChat(
    [primary_agent, critic_agent, security_agent],
    model_client=get_client()
)
```

### Q3: 如何让智能体看到完整的对话历史？

**A**: AutoGen 会自动维护对话历史，每个智能体都能看到之前的所有消息。

### Q4: 可以混合使用不同的大模型吗？

**A**: 可以！每个智能体都可以使用不同的模型：

```python
# 测试工程师使用 DeepSeek
primary_agent = AssistantAgent(
    "primary",
    model_client=get_deepseek_client(),
    ...
)

# 评审专家使用 GPT-4
critic_agent = AssistantAgent(
    "critic",
    model_client=get_openai_client(),
    ...
)
```

---

## 总结

本文作为 **AutoGen智能体开发系列** 的进阶篇，介绍了：

✅ 多智能体协作的核心概念  
✅ RoundRobinGroupChat 的使用方法  
✅ 系统提示词的设计技巧  
✅ 终止条件的灵活配置  
✅ 实战案例：智能测试用例生成器  
✅ 进阶优化和最佳实践  

通过本文的学习，你应该能够：
1. 设计多智能体协作系统
2. 构建生产级的AI工具
3. 优化提示词以提高输出质量
4. 处理复杂的业务场景

## 系列预告

下一篇我们将进入**实战篇**：

📌 **AutoGen + FastAPI 构建测试平台**
- Web API 接口设计
- 前后端分离架构
- 测试用例管理系统
- 自动化测试用例生成
- 多模型对比和切换

敬请期待！💪

---

## 参考资源

**框架文档**：
- [AutoGen 官方文档](https://microsoft.github.io/autogen/)
- [AutoGen Teams 文档](https://microsoft.github.io/autogen/docs/tutorial/teams)
- [Python异步编程指南](https://docs.python.org/zh-cn/3/library/asyncio.html)

**相关文章**：
- [AutoGen智能体开发实战（基础篇）](/posts/9/)

---

## 项目源码

完整代码已在文中展示，你可以直接复制使用。关键步骤：

1. 安装依赖：`pip install autogen-agentchat autogen-ext python-dotenv`
2. 配置 `.Env` 文件（参考基础篇）
3. 创建 `backend/llms.py`（参考基础篇）
4. 创建 `teams.py`（本文代码）
5. 运行：`python teams.py`

---

> 💡 **温馨提示**: 多智能体协作是AI应用的未来方向，掌握这项技能将让你的AI工具更加强大和实用！

> 🔥 **推荐阅读**: [《AutoGen智能体开发实战（基础篇）》](/posts/9/) - 如果你还没看过基础篇，建议先从那里开始

> 📚 **系列文章**: 本系列会持续更新，记得关注后续的实战篇！

