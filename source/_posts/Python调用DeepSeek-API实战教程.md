---
title: Python调用DeepSeek API实战教程
date: 2025-11-01 17:00:00
tags:
  - Python
  - DeepSeek
  - API
  - AI
categories: Python
comments: true
cover: https://images.unsplash.com/photo-1555949963-ff9fe0c870eb?w=800
abbrlink: 7
---

# 前言

DeepSeek 是一款由深度求索（DeepSeek）公司开发的大语言模型，以其强大的性能和亲民的价格受到开发者的欢迎。本文将详细介绍如何使用 Python 调用 DeepSeek API，帮助你快速上手 AI 应用开发。

---

## 什么是 DeepSeek？

DeepSeek 是一个高性能的大语言模型，支持：
- 💬 对话生成
- 📝 文本创作
- 🔍 信息提取
- 💻 代码生成
- 🌐 多语言支持

相比其他 AI 模型，DeepSeek 的优势在于：
1. **性价比高**：价格相对亲民
2. **响应速度快**：推理速度优秀
3. **中文友好**：对中文的理解和生成能力强
4. **API 兼容**：兼容 OpenAI 的 API 格式

---

## 环境准备

### 1. 安装必要的库

```bash
pip install openai requests
```

### 2. 获取 API Key

1. 访问 [DeepSeek 官网](https://platform.deepseek.com/)
2. 注册并登录账号
3. 在控制台创建 API Key
4. 保存好你的 API Key（注意保密）

---

## 基础用法

### 方法一：使用 OpenAI SDK（推荐）

DeepSeek API 兼容 OpenAI 的接口格式，可以直接使用 OpenAI 的 Python SDK：

```python
from openai import OpenAI

# 初始化客户端
client = OpenAI(
    api_key="your-deepseek-api-key",  # 替换为你的 API Key
    base_url="https://api.deepseek.com"  # DeepSeek API 地址
)

# 调用 API
response = client.chat.completions.create(
    model="deepseek-chat",  # 使用的模型
    messages=[
        {"role": "system", "content": "你是一个有帮助的AI助手"},
        {"role": "user", "content": "用Python写一个快速排序算法"}
    ],
    temperature=0.7,  # 控制输出的随机性
    max_tokens=2000   # 最大token数
)

# 打印结果
print(response.choices[0].message.content)
```

### 方法二：使用 requests 库

如果你想更灵活地控制请求，可以直接使用 requests：

```python
import requests
import json

def call_deepseek(prompt, api_key):
    url = "https://api.deepseek.com/v1/chat/completions"
    
    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {api_key}"
    }
    
    data = {
        "model": "deepseek-chat",
        "messages": [
            {"role": "user", "content": prompt}
        ],
        "temperature": 0.7,
        "max_tokens": 2000
    }
    
    response = requests.post(url, headers=headers, json=data)
    
    if response.status_code == 200:
        result = response.json()
        return result['choices'][0]['message']['content']
    else:
        return f"Error: {response.status_code} - {response.text}"

# 使用示例
api_key = "your-deepseek-api-key"
prompt = "请介绍一下Python的装饰器"
answer = call_deepseek(prompt, api_key)
print(answer)
```

---

## 进阶功能

### 1. 流式输出

实现打字机效果，逐字输出内容：

```python
from openai import OpenAI

client = OpenAI(
    api_key="your-deepseek-api-key",
    base_url="https://api.deepseek.com"
)

# 启用流式输出
stream = client.chat.completions.create(
    model="deepseek-chat",
    messages=[
        {"role": "user", "content": "写一首关于编程的诗"}
    ],
    stream=True  # 开启流式输出
)

# 逐步打印输出
for chunk in stream:
    if chunk.choices[0].delta.content is not None:
        print(chunk.choices[0].delta.content, end='', flush=True)
print()
```

### 2. 多轮对话

实现连续对话功能：

```python
from openai import OpenAI

class DeepSeekChat:
    def __init__(self, api_key):
        self.client = OpenAI(
            api_key=api_key,
            base_url="https://api.deepseek.com"
        )
        self.messages = [
            {"role": "system", "content": "你是一个友好的AI助手"}
        ]
    
    def chat(self, user_input):
        # 添加用户消息
        self.messages.append({"role": "user", "content": user_input})
        
        # 调用API
        response = self.client.chat.completions.create(
            model="deepseek-chat",
            messages=self.messages,
            temperature=0.7
        )
        
        # 获取助手回复
        assistant_message = response.choices[0].message.content
        self.messages.append({"role": "assistant", "content": assistant_message})
        
        return assistant_message
    
    def clear_history(self):
        # 清除对话历史，保留系统提示
        self.messages = [self.messages[0]]

# 使用示例
chat = DeepSeekChat("your-deepseek-api-key")

print(chat.chat("你好，请介绍一下自己"))
print(chat.chat("你能帮我做什么？"))
print(chat.chat("我想学习Python，有什么建议吗？"))
```

### 3. 函数调用（Function Calling）

让 AI 能够调用特定函数：

```python
from openai import OpenAI
import json

client = OpenAI(
    api_key="your-deepseek-api-key",
    base_url="https://api.deepseek.com"
)

# 定义可用的函数
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "获取指定城市的天气信息",
            "parameters": {
                "type": "object",
                "properties": {
                    "city": {
                        "type": "string",
                        "description": "城市名称，例如：北京、上海"
                    }
                },
                "required": ["city"]
            }
        }
    }
]

def get_weather(city):
    # 这里模拟获取天气（实际应该调用天气API）
    return f"{city}今天天气晴朗，气温25度"

# 调用API
messages = [{"role": "user", "content": "北京今天天气怎么样？"}]

response = client.chat.completions.create(
    model="deepseek-chat",
    messages=messages,
    tools=tools,
    tool_choice="auto"
)

# 检查是否需要调用函数
message = response.choices[0].message
if message.tool_calls:
    # 执行函数调用
    for tool_call in message.tool_calls:
        if tool_call.function.name == "get_weather":
            args = json.loads(tool_call.function.arguments)
            weather = get_weather(args["city"])
            print(f"天气信息: {weather}")
```

---

## 实用封装类

将常用功能封装成一个完整的工具类：

```python
from openai import OpenAI
from typing import List, Dict, Optional
import json

class DeepSeekAPI:
    """DeepSeek API 封装类"""
    
    def __init__(self, api_key: str, model: str = "deepseek-chat"):
        """
        初始化DeepSeek客户端
        
        Args:
            api_key: DeepSeek API密钥
            model: 使用的模型名称
        """
        self.client = OpenAI(
            api_key=api_key,
            base_url="https://api.deepseek.com"
        )
        self.model = model
        self.conversation_history = []
    
    def chat(
        self, 
        message: str, 
        system_prompt: Optional[str] = None,
        temperature: float = 0.7,
        max_tokens: int = 2000,
        stream: bool = False
    ) -> str:
        """
        发送单次对话请求
        
        Args:
            message: 用户消息
            system_prompt: 系统提示（可选）
            temperature: 温度参数，控制随机性
            max_tokens: 最大token数
            stream: 是否使用流式输出
            
        Returns:
            AI的回复内容
        """
        messages = []
        
        if system_prompt:
            messages.append({"role": "system", "content": system_prompt})
        
        messages.append({"role": "user", "content": message})
        
        response = self.client.chat.completions.create(
            model=self.model,
            messages=messages,
            temperature=temperature,
            max_tokens=max_tokens,
            stream=stream
        )
        
        if stream:
            return self._handle_stream(response)
        else:
            return response.choices[0].message.content
    
    def chat_with_history(
        self,
        message: str,
        system_prompt: Optional[str] = "你是一个有帮助的AI助手",
        temperature: float = 0.7,
        max_tokens: int = 2000
    ) -> str:
        """
        带历史记录的多轮对话
        
        Args:
            message: 用户消息
            system_prompt: 系统提示
            temperature: 温度参数
            max_tokens: 最大token数
            
        Returns:
            AI的回复内容
        """
        # 如果是第一条消息，添加系统提示
        if not self.conversation_history:
            self.conversation_history.append({
                "role": "system",
                "content": system_prompt
            })
        
        # 添加用户消息
        self.conversation_history.append({
            "role": "user",
            "content": message
        })
        
        # 调用API
        response = self.client.chat.completions.create(
            model=self.model,
            messages=self.conversation_history,
            temperature=temperature,
            max_tokens=max_tokens
        )
        
        # 获取并保存助手回复
        assistant_message = response.choices[0].message.content
        self.conversation_history.append({
            "role": "assistant",
            "content": assistant_message
        })
        
        return assistant_message
    
    def _handle_stream(self, stream):
        """处理流式输出"""
        full_content = ""
        for chunk in stream:
            if chunk.choices[0].delta.content is not None:
                content = chunk.choices[0].delta.content
                print(content, end='', flush=True)
                full_content += content
        print()
        return full_content
    
    def clear_history(self):
        """清除对话历史"""
        self.conversation_history = []
    
    def get_history(self) -> List[Dict]:
        """获取对话历史"""
        return self.conversation_history
    
    def save_history(self, filepath: str):
        """保存对话历史到文件"""
        with open(filepath, 'w', encoding='utf-8') as f:
            json.dump(self.conversation_history, f, ensure_ascii=False, indent=2)
    
    def load_history(self, filepath: str):
        """从文件加载对话历史"""
        with open(filepath, 'r', encoding='utf-8') as f:
            self.conversation_history = json.load(f)

# 使用示例
if __name__ == "__main__":
    # 初始化
    api = DeepSeekAPI(api_key="your-deepseek-api-key")
    
    # 单次对话
    response = api.chat("用Python实现一个二分查找算法")
    print(response)
    
    # 多轮对话
    print("\n=== 多轮对话示例 ===")
    api.clear_history()
    print("用户: 你好")
    print(f"AI: {api.chat_with_history('你好')}\n")
    
    print("用户: 我想学习Python")
    print(f"AI: {api.chat_with_history('我想学习Python')}\n")
    
    print("用户: 有什么好的学习资源推荐吗？")
    print(f"AI: {api.chat_with_history('有什么好的学习资源推荐吗？')}\n")
    
    # 保存对话历史
    api.save_history("conversation.json")
```

---

## 最佳实践

### 1. API Key 安全管理

**不要**直接在代码中硬编码 API Key，使用环境变量：

```python
import os
from dotenv import load_dotenv

# 加载.env文件
load_dotenv()

# 从环境变量获取API Key
api_key = os.getenv("DEEPSEEK_API_KEY")

client = OpenAI(
    api_key=api_key,
    base_url="https://api.deepseek.com"
)
```

创建 `.env` 文件：
```
DEEPSEEK_API_KEY=your-api-key-here
```

### 2. 错误处理

添加完善的错误处理机制：

```python
from openai import OpenAI
import time

def call_with_retry(client, messages, max_retries=3):
    """带重试机制的API调用"""
    for attempt in range(max_retries):
        try:
            response = client.chat.completions.create(
                model="deepseek-chat",
                messages=messages,
                timeout=30  # 设置超时
            )
            return response.choices[0].message.content
        
        except Exception as e:
            if attempt < max_retries - 1:
                wait_time = 2 ** attempt  # 指数退避
                print(f"请求失败，{wait_time}秒后重试... 错误: {str(e)}")
                time.sleep(wait_time)
            else:
                print(f"达到最大重试次数，请求失败: {str(e)}")
                raise
```

### 3. Token 计数与成本控制

```python
def estimate_tokens(text):
    """粗略估算token数量（中文约1.5字符=1token，英文约4字符=1token）"""
    chinese_chars = sum(1 for char in text if '\u4e00' <= char <= '\u9fff')
    other_chars = len(text) - chinese_chars
    return int(chinese_chars / 1.5 + other_chars / 4)

def chat_with_budget(client, message, max_budget_tokens=1000):
    """带预算控制的对话"""
    input_tokens = estimate_tokens(message)
    
    if input_tokens > max_budget_tokens:
        return "输入内容过长，超出预算限制"
    
    response = client.chat.completions.create(
        model="deepseek-chat",
        messages=[{"role": "user", "content": message}],
        max_tokens=max_budget_tokens - input_tokens
    )
    
    return response.choices[0].message.content
```

---

## 实战案例

### 案例1：智能代码审查助手

```python
def code_review(code: str, language: str = "Python") -> str:
    """使用DeepSeek进行代码审查"""
    api = DeepSeekAPI(api_key=os.getenv("DEEPSEEK_API_KEY"))
    
    prompt = f"""
请对以下{language}代码进行审查，重点关注：
1. 代码质量和规范性
2. 潜在的bug和安全问题
3. 性能优化建议
4. 可读性和可维护性

代码：
```{language.lower()}
{code}
```

请给出详细的审查意见和改进建议。
"""
    
    return api.chat(prompt, temperature=0.3)

# 使用示例
code = """
def divide(a, b):
    return a / b

result = divide(10, 0)
print(result)
"""

review = code_review(code)
print(review)
```

### 案例2：智能文档生成器

```python
def generate_documentation(code: str) -> str:
    """自动生成代码文档"""
    api = DeepSeekAPI(api_key=os.getenv("DEEPSEEK_API_KEY"))
    
    prompt = f"""
请为以下代码生成详细的文档注释，包括：
1. 函数/类的功能描述
2. 参数说明
3. 返回值说明
4. 使用示例
5. 注意事项

代码：
```python
{code}
```
"""
    
    return api.chat(prompt, temperature=0.5)
```

### 案例3：智能问答机器人

```python
class QABot:
    def __init__(self, knowledge_base: str):
        self.api = DeepSeekAPI(api_key=os.getenv("DEEPSEEK_API_KEY"))
        self.knowledge_base = knowledge_base
    
    def answer(self, question: str) -> str:
        """基于知识库回答问题"""
        system_prompt = f"""
你是一个专业的问答助手，基于以下知识库回答用户问题：

{self.knowledge_base}

如果问题超出知识库范围，请明确告知用户。
"""
        
        return self.api.chat(
            message=question,
            system_prompt=system_prompt,
            temperature=0.3
        )

# 使用示例
knowledge = """
公司名称：ABC科技有限公司
营业时间：周一至周五 9:00-18:00
联系电话：123-456-7890
主营业务：软件开发、AI解决方案
"""

bot = QABot(knowledge)
print(bot.answer("你们公司的营业时间是什么？"))
print(bot.answer("如何联系你们？"))
```

---

## 常见问题

### Q1: API 调用速度慢怎么办？

- 使用流式输出提升用户体验
- 适当降低 `max_tokens` 参数
- 检查网络连接
- 考虑使用异步调用

### Q2: 如何节省 API 费用？

- 合理设置 `max_tokens` 限制
- 使用更精确的提示词，减少重试
- 定期清理不必要的对话历史
- 对于简单任务使用更小的模型

### Q3: 如何提高回答质量？

- 使用清晰、具体的提示词
- 提供足够的上下文信息
- 适当调整 `temperature` 参数
- 使用系统提示定义角色和行为

---

## 总结

本文介绍了使用 Python 调用 DeepSeek API 的完整流程，包括：

✅ 基础调用方法  
✅ 流式输出实现  
✅ 多轮对话功能  
✅ 函数调用特性  
✅ 完整的工具类封装  
✅ 最佳实践和实战案例  

DeepSeek 凭借其出色的性能和亲民的价格，是开发 AI 应用的优秀选择。希望本文能帮助你快速上手 DeepSeek API，开发出更多有趣的 AI 应用！

---

## 参考资源

- [DeepSeek 官网](https://www.deepseek.com/)
- [DeepSeek API 文档](https://platform.deepseek.com/docs)
- [OpenAI Python SDK](https://github.com/openai/openai-python)

---

> 💡 **提示**: 本文的示例代码都已经过测试，可以直接使用。记得将 `your-deepseek-api-key` 替换为你自己的 API Key！

> 🔥 **推荐阅读**: 如果你对AI开发感兴趣，欢迎关注我的博客，后续会分享更多AI应用开发教程！

