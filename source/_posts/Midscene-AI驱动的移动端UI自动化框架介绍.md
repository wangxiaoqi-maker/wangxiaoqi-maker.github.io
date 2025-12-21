---
title: Midscene：AI驱动的移动端UI自动化框架介绍
date: 2025-12-21 10:00:00
tags:
  - AI
  - 自动化测试
  - Midscene
  - iOS
  - Android
  - UI自动化
categories: 测试
comments: true
cover: https://images.unsplash.com/photo-1526498460520-4c246339dccb?w=800&q=80
abbrlink: 15
---

# 前言

在移动应用测试领域，传统的 UI 自动化测试一直面临着诸多挑战：元素定位困难、维护成本高、脚本编写复杂等。而随着 AI 大模型技术的发展，一种全新的自动化测试方式正在兴起——**基于视觉语言模型的智能 UI 自动化**。

本文将介绍 **Midscene.js**，一款基于多模态大语言模型的 UI 自动化测试框架。它允许你使用**自然语言**描述测试步骤，让 AI 来理解界面、执行操作，彻底颠覆传统自动化测试的开发方式！

本文是 **Midscene 移动端自动化系列** 的第一篇，将带你全面了解这个框架的核心概念和优势。

---

## Midscene 是什么？

### 核心定义

[Midscene.js](https://midscenejs.com/zh/) 是一个基于 **多模态大语言模型（Vision Language Model, VLM）** 的 UI 自动化框架。它的核心理念是：

> **用自然语言描述你想做的事，让 AI 来完成界面操作。**

传统自动化测试需要精确定位元素（XPath、CSS 选择器、Accessibility ID 等），而 Midscene 通过截取屏幕截图，利用 AI 模型理解界面内容，自动识别并操作目标元素。

### 支持平台

| 平台 | 包名 | 驱动方式 |
|------|------|----------|
| Web 浏览器 | `@midscene/web` | Playwright / Puppeteer |
| Android | `@midscene/android` | adb（Android Debug Bridge） |
| iOS | `@midscene/ios` | WebDriverAgent |
| 桌面应用 | `@midscene/web` | Chrome DevTools |

### 技术架构

```
┌─────────────────────────────────────────────────────────────┐
│                      用户层（User Layer）                    │
│  ┌─────────────────────────────────────────────────────┐   │
│  │           自然语言指令 / YAML 脚本 / JS 代码          │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                   Midscene 核心（Core）                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  截图引擎    │  │  AI 规划器   │  │  动作执行器  │      │
│  │ Screenshot  │  │  AI Planner  │  │  Executor    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                     AI 模型层（AI Layer）                    │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐            │
│  │  GPT-4o   │  │  Qwen-VL   │  │  UI-TARS   │  ...       │
│  └────────────┘  └────────────┘  └────────────┘            │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                   设备层（Device Layer）                     │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐            │
│  │  Android  │  │    iOS     │  │  Browser   │            │
│  │   (adb)   │  │   (WDA)    │  │(Playwright)│            │
│  └────────────┘  └────────────┘  └────────────┘            │
└─────────────────────────────────────────────────────────────┘
```

---

## 为什么选择 Midscene？

### 传统自动化测试的痛点

❌ **元素定位困难**：XPath、CSS 选择器频繁变化，维护成本高  
❌ **跨平台不兼容**：iOS、Android、Web 需要编写不同的脚本  
❌ **技术门槛高**：需要掌握多种工具和编程技能  
❌ **脚本易失效**：UI 微调导致测试脚本大面积失败  
❌ **无法处理动态内容**：弹窗、加载状态等难以预测  

### Midscene 的解决方案

✅ **自然语言操作**：用人话描述测试步骤，无需关心元素定位  
✅ **智能识别**：AI 理解界面语义，自动适应 UI 变化  
✅ **多平台统一**：同一套 API 支持 Web、iOS、Android  
✅ **低技术门槛**：非开发人员也能编写自动化脚本  
✅ **智能处理异常**：自动识别并处理弹窗、权限请求等  

### 对比示例

**传统方式（Appium）：**

```python
# 需要精确的元素定位器
email_field = driver.find_element(
    AppiumBy.XPATH, 
    "//android.widget.EditText[@resource-id='com.app:id/email']"
)
email_field.send_keys("test@example.com")

password_field = driver.find_element(
    AppiumBy.XPATH,
    "//android.widget.EditText[@resource-id='com.app:id/password']"
)
password_field.send_keys("password123")

login_button = driver.find_element(
    AppiumBy.XPATH,
    "//android.widget.Button[@text='登录']"
)
login_button.click()
```

**Midscene 方式：**

```javascript
// 用自然语言描述，AI 自动完成
await agent.aiAct('在邮箱输入框输入 test@example.com');
await agent.aiAct('在密码输入框输入 password123');
await agent.aiAct('点击登录按钮');
```

**优势一目了然！** 🚀

---

## Midscene 核心 API 介绍

Midscene 提供了简洁而强大的 API，主要分为以下几类：

### 1. 动作类 API（Action）

#### `aiAct()` - 智能动作执行

**自动规划模式**：AI 会自动规划并执行一系列操作。

```javascript
// 复杂操作一句话搞定
await agent.aiAct('在搜索框中输入 Midscene，执行搜索，跳转到第一条结果');

// 表单填写
await agent.aiAct('填写完整的注册表单，让所有字段通过校验');

// 页面导航
await agent.aiAct('点击设置按钮，进入账户页面，开启深色模式');
```

**参数说明**：

| 参数 | 类型 | 说明 |
|------|------|------|
| instruction | string | 自然语言描述的操作指令 |
| options | object | 可选配置，如超时时间等 |

#### `aiTap()` - 即时点击

**即时操作模式**：直接点击指定元素，不进行复杂规划。

```javascript
// 直接点击
await agent.aiTap('登录按钮');
await agent.aiTap('右上角的设置图标');
await agent.aiTap('第二条搜索结果');
```

**适用场景**：
- 简单的单次点击操作
- 需要快速响应的场景
- 已知明确目标元素的情况

#### `aiInput()` - 智能输入

```javascript
// 在指定位置输入文本
await agent.aiInput('用户名输入框', 'testuser');
await agent.aiInput('搜索框', '今天天气怎么样');
```

### 2. 查询类 API（Query）

#### `aiQuery()` - 数据提取

从界面中提取结构化数据，支持自定义返回格式。

```javascript
// 提取商品列表
const products = await agent.aiQuery(
  '{name: string, price: number}[], 获取页面中所有商品的名称和价格'
);
console.log(products);
// 输出: [{ name: "iPhone 15", price: 6999 }, { name: "iPad Air", price: 4799 }]

// 提取单个值
const title = await agent.aiQuery('string, 页面标题是什么');

// 提取复杂数据
const orderInfo = await agent.aiQuery(`{
  orderId: string,
  status: string,
  totalAmount: number,
  items: { name: string, quantity: number }[]
}, 获取订单详情`);
```

#### `aiString()` / `aiNumber()` / `aiBoolean()` - 类型化查询

```javascript
// 获取字符串
const userName = await agent.aiString('当前登录的用户名');

// 获取数字
const itemCount = await agent.aiNumber('购物车中的商品数量');

// 获取布尔值
const isLoggedIn = await agent.aiBoolean('用户是否已登录');
```

### 3. 断言类 API（Assert）

#### `aiAssert()` - 智能断言

验证界面状态，如果条件不满足则抛出错误。

```javascript
// 验证页面状态
await agent.aiAssert('页面上显示了欢迎用户的信息');
await agent.aiAssert('购物车图标上显示了数字 3');
await agent.aiAssert('登录按钮是可点击的状态');

// 复杂断言
await agent.aiAssert('商品列表至少显示了 5 个商品，且每个商品都有价格标签');
```

#### `aiWaitFor()` - 智能等待

等待某个条件满足后再继续执行。

```javascript
// 等待加载完成
await agent.aiWaitFor('页面加载完成，不再显示加载动画');

// 等待元素出现
await agent.aiWaitFor('搜索结果已经显示');

// 等待状态变化
await agent.aiWaitFor('订单状态变为"已完成"');
```

### 4. 设备控制 API

#### 通用控制

```javascript
// 启动应用
await agent.launch('com.example.app');  // Android 包名
await agent.launch('https://example.com');  // iOS Safari

// 截图
await agent.screenshot('screenshot.png');

// 返回
await agent.pressKey('BACK');  // Android
await agent.pressKey('HOME');
```

#### 坐标操作

```javascript
// 点击坐标
await agent.tap(100, 200);

// 滑动
await agent.swipe(100, 500, 100, 100);  // 从 (100,500) 滑动到 (100,100)

// 输入文本
await agent.input('Hello World');
```

---

## 支持的 AI 模型

Midscene 支持多种视觉语言模型，你可以根据需求和成本选择：

### 模型对比

| 模型 | 提供商 | 特点 | 推荐场景 |
|------|--------|------|----------|
| GPT-4o | OpenAI | 综合能力强，识别精准 | 复杂场景、高精度要求 |
| GPT-4o-mini | OpenAI | 成本低，速度快 | 简单场景、预算有限 |
| Qwen2.5-VL | 阿里云 | 中文优化，价格实惠 | 中文应用、国内部署 |
| UI-TARS | 字节跳动 | 专为 UI 优化 | UI 自动化专用 |
| Claude 3.5 | Anthropic | 推理能力强 | 复杂逻辑判断 |

### 配置示例

```bash
# 环境变量配置
export MIDSCENE_MODEL_BASE_URL="https://api.openai.com/v1"
export MIDSCENE_MODEL_API_KEY="sk-your-api-key"
export MIDSCENE_MODEL_NAME="gpt-4o"
export MIDSCENE_MODEL_FAMILY="openai"
```

**使用豆包（UI-TARS）：**

```bash
export MIDSCENE_MODEL_BASE_URL="https://ark.cn-beijing.volces.com/api/v3"
export MIDSCENE_MODEL_API_KEY="your-doubao-api-key"
export MIDSCENE_MODEL_NAME="ep-your-endpoint-id"
export MIDSCENE_MODEL_FAMILY="doubao"
```

**使用千问（Qwen-VL）：**

```bash
export MIDSCENE_MODEL_BASE_URL="https://dashscope.aliyuncs.com/compatible-mode/v1"
export MIDSCENE_MODEL_API_KEY="your-qwen-api-key"
export MIDSCENE_MODEL_NAME="qwen-vl-max"
export MIDSCENE_MODEL_FAMILY="qwen"
```

---

## Midscene 工作流程

### 执行一条 AI 指令的完整流程

```
1. 用户发出指令
   └─ agent.aiAct('点击登录按钮')
          ↓
2. 截取当前屏幕截图
   └─ 获取设备屏幕的 base64 图像
          ↓
3. 构建 AI 请求
   └─ 将截图 + 指令发送给 VLM 模型
          ↓
4. AI 分析并返回结果
   └─ 模型识别界面，返回目标元素坐标/动作计划
          ↓
5. 执行操作
   └─ Midscene 将 AI 返回的坐标转换为实际操作
          ↓
6. 验证结果
   └─ 可选：再次截图验证操作是否成功
```

### 自动规划 vs 即时操作

| 模式 | API | 特点 | Token 消耗 |
|------|-----|------|-----------|
| 自动规划 | `aiAct()` | AI 自动分解复杂任务，逐步执行 | 较高 |
| 即时操作 | `aiTap()` | 直接执行单一操作，不做规划 | 较低 |

**使用建议**：
- 简单操作使用 `aiTap()`、`aiInput()`，减少 Token 消耗
- 复杂流程使用 `aiAct()`，让 AI 自动规划

---

## 最佳实践：降低 Token 消耗

### 1. 使用 AI 缓存

Midscene 支持缓存 AI 规划结果，重复执行相同步骤时无需再次调用 AI。

```javascript
// 启用缓存
const agent = new IOSAgent(device, {
  cacheEnabled: true,
  cacheDir: './midscene_cache'
});
```

### 2. 优先使用即时操作

```javascript
// ❌ 不推荐：简单操作使用自动规划
await agent.aiAct('点击登录按钮');

// ✅ 推荐：简单操作使用即时操作
await agent.aiTap('登录按钮');
```

### 3. 合并相关操作

```javascript
// ❌ 不推荐：分开执行
await agent.aiAct('输入用户名');
await agent.aiAct('输入密码');
await agent.aiAct('点击登录');

// ✅ 推荐：合并为一条指令
await agent.aiAct('输入用户名 testuser，输入密码 123456，然后点击登录');
```

### 4. 使用 aiActionContext 处理弹窗

```javascript
const agent = new IOSAgent(device, {
  aiActionContext: '如果出现权限弹窗请点击同意，如果出现登录弹窗请关闭'
});

// 这样就不需要单独处理各种弹窗了
```

### 5. 选择合适的模型

| 场景 | 推荐模型 | 原因 |
|------|----------|------|
| 开发调试 | GPT-4o-mini | 成本低，速度快 |
| 生产环境 | GPT-4o / Qwen-VL | 稳定性高 |
| 中文应用 | Qwen-VL | 中文理解更好 |
| UI 密集型 | UI-TARS | 专为 UI 优化 |

---

## 总结

Midscene 作为一款 AI 驱动的 UI 自动化框架，带来了以下革命性的改变：

✅ **自然语言操作**：用人话描述测试步骤  
✅ **智能识别**：AI 理解界面语义，自动定位元素  
✅ **多平台统一**：一套 API 支持 Web、iOS、Android  
✅ **低维护成本**：UI 变化不影响测试脚本  
✅ **结构化数据提取**：从界面提取 JSON 数据  

## 系列预告

接下来的两篇文章将带你深入实践：

📌 **第二篇：iOS 和 Android 自动化环境配置**
- WebDriverAgent 配置（iOS）
- adb 环境搭建（Android）
- 真机和模拟器的区别
- 常见问题排查

📌 **第三篇：Midscene 移动端自动化实战**
- 完整的 iOS/Android 自动化案例
- 测试脚本编写技巧
- 稳定性优化方案
- Token 成本控制策略

敬请期待！💪

---

## 参考资源

**官方文档**：
- [Midscene.js 官方网站](https://midscenejs.com/zh/)
- [iOS 快速开始](https://midscenejs.com/zh/ios-getting-started.html)
- [Android 快速开始](https://midscenejs.com/zh/android-getting-started.html)
- [API 参考文档](https://midscenejs.com/zh/api.html)

**示例项目**：
- [iOS SDK 示例](https://github.com/web-infra-dev/midscene-example/blob/main/ios/javascript-sdk-demo)
- [Android SDK 示例](https://github.com/web-infra-dev/midscene-example/tree/main/android)

---

> 💡 **温馨提示**：AI 驱动的 UI 自动化是测试领域的新趋势，掌握 Midscene 将让你的自动化测试更加高效和智能！

> 🔥 **推荐阅读**：本系列的后续文章将带你从环境配置到实战应用，全面掌握 Midscene！

> 📚 **系列文章**：本系列会持续更新，记得关注后续的配置篇和实战篇！

