---
title: Midscene移动端自动化实战：从入门到精通
date: 2025-12-30 12:00:00
tags:
  - AI
  - 自动化测试
  - Midscene
  - iOS
  - Android
  - UI自动化
  - 实战教程
categories: 测试
comments: true
cover: https://images.unsplash.com/photo-1551650975-87deedd944c3?w=800&q=80
abbrlink: 17
---

# 前言

在前两篇文章中，我们了解了 [Midscene 框架的核心概念](/posts/15/) 和 [iOS/Android 环境配置方法](/posts/16/)。本文是 **Midscene 移动端自动化系列** 的第三篇，将通过完整的实战案例，带你掌握 Midscene 在真实项目中的应用。

本文将涵盖：

- 📱 **完整的 iOS/Android 自动化案例**
- 🎯 **常用操作方法详解**
- 🛡️ **稳定性优化技巧**
- 💰 **Token 成本控制策略**
- 📊 **测试报告生成与分析**

---

## 实战案例一：iOS 应用自动化

### 场景描述

我们将实现一个完整的 iOS Safari 浏览器自动化测试，包括：
1. 打开指定网站
2. 执行搜索操作
3. 提取搜索结果
4. 验证页面状态

### 完整代码

```javascript
import 'dotenv/config';
import { IOSAgent, IOSDevice } from '@midscene/ios';

const sleep = (ms) => new Promise((r) => setTimeout(r, ms));

/**
 * iOS Safari 搜索自动化测试
 */
async function iosSafariTest() {
  console.log('🍎 ========== iOS Safari 自动化测试 ==========');
  
  // 创建设备连接
  const device = new IOSDevice({
    wdaPort: 8100,
    wdaHost: 'localhost',
  });

  // 创建 Agent，配置全局上下文
  const agent = new IOSAgent(device, {
    aiActionContext: `
      如果弹出位置权限请求，点击"允许"或"使用App时允许"。
      如果弹出通知权限请求，点击"不允许"。
      如果出现登录弹窗，点击关闭按钮或遮罩层关闭。
      如果弹出键盘需要输入通行秘钥，则点击左下角的小键盘图标切换输入方式。
    `,
  });

  try {
    // 步骤1：连接设备
    console.log('📱 正在连接设备...');
    await device.connect();
    console.log('✅ 设备连接成功');

    // 步骤2：打开网页
    console.log('🌐 正在打开淘宝...');
    await device.launch('https://m.taobao.com');
    await sleep(5000); // 等待页面加载
    console.log('✅ 页面加载完成');

    // 步骤3：执行搜索
    console.log('🔍 正在搜索商品...');
    await agent.aiTap('搜索框');
    await sleep(1000);
    await agent.aiAct('输入"iPhone 手机壳"并点击搜索按钮');
    await sleep(3000);
    console.log('✅ 搜索完成');

    // 步骤4：等待结果加载
    console.log('⏳ 等待搜索结果...');
    await agent.aiWaitFor('页面显示了商品列表，至少有3个商品');
    console.log('✅ 搜索结果已加载');

    // 步骤5：提取商品信息
    console.log('📊 正在提取商品信息...');
    const products = await agent.aiQuery(`
      {
        name: string,
        price: string,
        shop: string
      }[],
      获取页面中前5个商品的名称、价格和店铺名称
    `);
    console.log('📦 商品列表:');
    products.forEach((p, i) => {
      console.log(`  ${i + 1}. ${p.name} - ${p.price} - ${p.shop}`);
    });

    // 步骤6：断言验证
    console.log('✅ 正在验证页面状态...');
    await agent.aiAssert('页面显示了多个商品，且每个商品都有价格');
    console.log('✅ 断言通过');

    // 步骤7：点击商品进入详情
    console.log('👆 正在点击第一个商品...');
    await agent.aiTap('第一个商品');
    await sleep(3000);

    // 步骤8：提取详情页信息
    const detail = await agent.aiQuery(`
      {
        title: string,
        price: string,
        sales: string,
        shopName: string
      },
      获取商品详情页的标题、价格、销量和店铺名称
    `);
    console.log('📋 商品详情:', detail);

    console.log('🎉 iOS 测试全部完成！');

  } catch (error) {
    console.error('❌ 测试失败:', error.message);
    // 失败时截图保存
    try {
      await agent.screenshot(`error_${Date.now()}.png`);
      console.log('📸 已保存错误截图');
    } catch (e) {
      console.error('截图失败:', e.message);
    }
  } finally {
    await device.destroy();
    console.log('🏁 设备连接已断开');
  }
}

// 运行测试
iosSafariTest();
```

### 运行结果示例

```
🍎 ========== iOS Safari 自动化测试 ==========
📱 正在连接设备...
✅ 设备连接成功
🌐 正在打开淘宝...
✅ 页面加载完成
🔍 正在搜索商品...
✅ 搜索完成
⏳ 等待搜索结果...
✅ 搜索结果已加载
📊 正在提取商品信息...
📦 商品列表:
  1. iPhone 15 Pro Max 透明防摔壳 - ¥29.9 - 壳王旗舰店
  2. 苹果14手机壳磨砂保护套 - ¥19.9 - 数码配件专营
  3. iPhone 13 液态硅胶壳 - ¥39.0 - 优品数码
  ...
✅ 正在验证页面状态...
✅ 断言通过
👆 正在点击第一个商品...
📋 商品详情: { title: 'iPhone 15 Pro Max...', price: '¥29.9', ... }
🎉 iOS 测试全部完成！
🏁 设备连接已断开
```

---

## 实战案例二：Android 应用自动化

### 场景描述

实现一个 Android 应用的完整自动化测试流程，包括：
1. 启动 App
2. 处理权限弹窗
3. 执行核心功能操作
4. 验证操作结果

### 完整代码

```javascript
import 'dotenv/config';
import { AndroidAgent, AndroidDevice, getConnectedDevices } from '@midscene/android';

const sleep = (ms) => new Promise((r) => setTimeout(r, ms));

/**
 * Android 应用自动化测试
 */
async function androidAppTest() {
  console.log('🤖 ========== Android 应用自动化测试 ==========');

  let device;

  try {
    // 步骤1：获取设备
    console.log('📱 正在检测设备...');
    const devices = await getConnectedDevices();
    
    if (devices.length === 0) {
      throw new Error('未检测到 Android 设备，请确保设备已连接并开启 USB 调试');
    }
    
    console.log(`✅ 检测到 ${devices.length} 台设备:`);
    devices.forEach((d, i) => {
      console.log(`  ${i + 1}. ${d.model || d.udid}`);
    });

    // 使用第一台设备
    device = new AndroidDevice(devices[0].udid);

    // 创建 Agent
    const agent = new AndroidAgent(device, {
      aiActionContext: `
        如果弹出权限请求弹窗，点击"允许"或"始终允许"。
        如果弹出更新提示，点击"稍后"或关闭按钮。
        如果出现广告弹窗，点击关闭或跳过按钮。
        如果需要输入 PIN 码，输入 000000。
      `,
    });

    // 连接设备
    await device.connect();
    console.log('✅ 设备连接成功');

    // 步骤2：启动 Chrome 浏览器
    console.log('🌐 正在启动 Chrome...');
    await agent.launch('com.android.chrome');
    await sleep(3000);
    console.log('✅ Chrome 已启动');

    // 步骤3：访问目标网站
    console.log('🔗 正在访问京东...');
    await agent.aiAct('点击地址栏，输入 m.jd.com，然后回车确认');
    await sleep(5000);
    console.log('✅ 网站加载完成');

    // 步骤4：执行搜索
    console.log('🔍 正在搜索商品...');
    await agent.aiTap('搜索框');
    await sleep(1000);
    await agent.aiAct('输入"无线耳机"并点击搜索');
    await sleep(3000);
    console.log('✅ 搜索完成');

    // 步骤5：等待并验证结果
    await agent.aiWaitFor('页面显示了商品搜索结果列表');
    
    // 步骤6：提取商品信息
    console.log('📊 正在提取商品信息...');
    const products = await agent.aiQuery(`
      {
        name: string,
        price: number,
        comments: string
      }[],
      获取前3个商品的名称、价格（数字）和评论数
    `);
    
    console.log('📦 搜索结果:');
    console.table(products);

    // 步骤7：滚动页面查看更多
    console.log('📜 正在滚动页面...');
    await agent.aiAct('向下滚动两屏，查看更多商品');
    await sleep(2000);

    // 步骤8：点击进入商品详情
    console.log('👆 正在查看商品详情...');
    await agent.aiTap('第一个商品');
    await sleep(3000);

    // 步骤9：验证详情页
    await agent.aiAssert('当前在商品详情页，显示了商品图片和价格');
    console.log('✅ 商品详情页验证通过');

    // 步骤10：模拟加入购物车
    console.log('🛒 正在加入购物车...');
    const hasCartButton = await agent.aiBoolean('页面上有加入购物车按钮');
    
    if (hasCartButton) {
      await agent.aiTap('加入购物车按钮');
      await sleep(2000);
      
      // 检查是否需要选择规格
      const needSelectSpec = await agent.aiBoolean('是否弹出了规格选择弹窗');
      if (needSelectSpec) {
        await agent.aiAct('选择第一个可用的规格选项，然后点击确认');
        await sleep(1000);
      }
      
      await agent.aiAssert('显示了加入购物车成功的提示或购物车数量增加');
      console.log('✅ 商品已加入购物车');
    } else {
      console.log('⚠️ 未找到加入购物车按钮');
    }

    console.log('🎉 Android 测试全部完成！');

  } catch (error) {
    console.error('❌ 测试失败:', error.message);
  } finally {
    if (device) {
      await device.disconnect();
    }
    console.log('🏁 设备连接已断开');
  }
}

// 运行测试
androidAppTest();
```

---

## 常用 UI 自动化方法详解

### 1. 元素操作方法

#### aiTap() - 点击操作

最常用的点击方法，适用于按钮、链接、图标等。

```javascript
// 基础点击
await agent.aiTap('登录按钮');
await agent.aiTap('右上角的设置图标');
await agent.aiTap('第三个商品');

// 精确定位点击
await agent.aiTap('用户协议旁边的复选框');
await agent.aiTap('价格最低的那个商品');
await agent.aiTap('红色的确认按钮');
```

#### aiInput() - 输入操作

用于文本输入，自动处理焦点。

```javascript
// 基础输入
await agent.aiInput('用户名输入框', 'testuser');
await agent.aiInput('密码框', 'password123');

// 搜索输入
await agent.aiInput('搜索框', '今天天气怎么样');
```

#### aiAct() - 复合操作

执行复杂的多步骤操作，AI 会自动规划执行。

```javascript
// 登录流程
await agent.aiAct('输入用户名 test@example.com，输入密码 123456，然后点击登录');

// 搜索流程
await agent.aiAct('点击搜索框，输入"笔记本电脑"，然后点击搜索按钮');

// 表单填写
await agent.aiAct(`
  填写注册表单：
  - 用户名：testuser
  - 邮箱：test@example.com
  - 密码：Test123456
  然后点击注册按钮
`);

// 导航操作
await agent.aiAct('点击底部导航的"我的"，然后点击设置按钮，进入账户安全');
```

### 2. 页面滚动方法

```javascript
// 使用 aiAct 进行滚动
await agent.aiAct('向下滚动页面');
await agent.aiAct('向上滚动到页面顶部');
await agent.aiAct('向下滚动直到看到"加载更多"按钮');

// 使用坐标滚动（更精确）
await agent.swipe(200, 600, 200, 200);  // 从下往上滑
await agent.swipe(200, 200, 200, 600);  // 从上往下滑

// 滚动到特定元素
await agent.aiAct('滚动页面直到看到"立即购买"按钮');
```

### 3. 数据提取方法

#### aiQuery() - 结构化数据提取

```javascript
// 提取列表数据
const products = await agent.aiQuery(`
  {
    name: string,
    price: number,
    rating: number,
    sold: string
  }[],
  提取页面中所有商品的信息
`);

// 提取单个对象
const userInfo = await agent.aiQuery(`
  {
    nickname: string,
    level: string,
    balance: number
  },
  获取当前用户的个人信息
`);

// 提取嵌套数据
const orderDetail = await agent.aiQuery(`
  {
    orderId: string,
    status: string,
    createTime: string,
    items: {
      name: string,
      quantity: number,
      price: number
    }[],
    totalAmount: number
  },
  提取订单详情
`);
```

#### aiString() / aiNumber() / aiBoolean() - 类型化提取

```javascript
// 提取字符串
const title = await agent.aiString('页面标题');
const userName = await agent.aiString('当前登录的用户名');

// 提取数字
const cartCount = await agent.aiNumber('购物车中的商品数量');
const totalPrice = await agent.aiNumber('订单总金额');

// 提取布尔值
const isLoggedIn = await agent.aiBoolean('用户是否已登录');
const hasStock = await agent.aiBoolean('商品是否有库存');
const isButtonEnabled = await agent.aiBoolean('购买按钮是否可点击');
```

### 4. 断言验证方法

#### aiAssert() - 状态断言

```javascript
// 基础断言
await agent.aiAssert('页面显示了欢迎信息');
await agent.aiAssert('购物车图标显示了数字 3');
await agent.aiAssert('登录按钮是可见的');

// 复杂断言
await agent.aiAssert('商品列表至少显示了5个商品，每个商品都有价格和图片');
await agent.aiAssert('当前在个人中心页面，且用户昵称已显示');
await agent.aiAssert('支付成功页面显示了订单号');
```

#### aiWaitFor() - 条件等待

```javascript
// 等待加载完成
await agent.aiWaitFor('页面加载完成，不再显示加载动画');
await agent.aiWaitFor('搜索结果已经显示');

// 等待元素出现
await agent.aiWaitFor('弹窗已经关闭');
await agent.aiWaitFor('商品列表已加载');

// 等待状态变化
await agent.aiWaitFor('订单状态变为"已发货"');
await agent.aiWaitFor('验证码已发送');

// 带超时的等待
await agent.aiWaitFor('支付成功', { timeout: 30000 });
```

### 5. 设备控制方法

```javascript
// 启动应用
await agent.launch('com.example.app');           // Android 包名
await agent.launch('https://www.example.com');    // iOS Safari

// 按键操作
await agent.pressKey('BACK');     // 返回（Android）
await agent.pressKey('HOME');     // Home 键
await agent.pressKey('ENTER');    // 确认键

// 截图
await agent.screenshot('screenshot.png');
await agent.screenshot(`step_${Date.now()}.png`);

// 坐标操作
await agent.tap(100, 200);                      // 点击坐标
await agent.swipe(100, 500, 100, 100, 500);     // 滑动
```

---

## 稳定性优化技巧

### 1. 使用 aiActionContext 统一处理弹窗

将常见弹窗处理逻辑配置在 `aiActionContext` 中，避免每次都要单独处理。

```javascript
const agent = new IOSAgent(device, {
  aiActionContext: `
    【权限处理】
    - 如果弹出位置权限请求，点击"使用App时允许"
    - 如果弹出通知权限请求，点击"不允许"
    - 如果弹出相机/相册权限请求，点击"允许"
    
    【弹窗处理】
    - 如果出现登录弹窗，点击关闭按钮
    - 如果出现广告弹窗，点击关闭或跳过
    - 如果出现更新提示，点击"稍后再说"
    
    【输入处理】
    - 如果键盘弹出通行密钥选项，点击左下角小键盘图标切换
    
    【页面状态】
    - 如果页面正在加载，等待加载完成
    - 如果出现网络错误，点击重试按钮
  `,
});
```

### 2. 添加适当的等待时间

```javascript
const sleep = (ms) => new Promise(r => setTimeout(r, ms));

// 页面切换后等待
await agent.aiTap('下一步');
await sleep(2000);  // 等待页面切换动画

// 网络请求后等待
await agent.aiAct('点击提交');
await sleep(3000);  // 等待接口响应

// 使用 aiWaitFor 智能等待（推荐）
await agent.aiWaitFor('页面加载完成');
```

### 3. 错误处理与重试机制

```javascript
/**
 * 带重试的操作执行
 */
async function retryAction(agent, action, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      await action();
      return true;
    } catch (error) {
      console.log(`⚠️ 第 ${i + 1} 次尝试失败: ${error.message}`);
      if (i < maxRetries - 1) {
        await sleep(2000);
        // 尝试恢复操作
        try {
          await agent.pressKey('BACK');
          await sleep(1000);
        } catch (e) {}
      }
    }
  }
  return false;
}

// 使用示例
const success = await retryAction(agent, async () => {
  await agent.aiTap('提交按钮');
  await agent.aiAssert('提交成功');
});

if (!success) {
  console.error('操作最终失败');
}
```

### 4. 分步骤执行复杂操作

```javascript
// ❌ 不推荐：一条指令执行太多操作
await agent.aiAct('登录账号，搜索商品，加入购物车，去结算，选择地址，选择支付方式，完成支付');

// ✅ 推荐：分步骤执行，便于定位问题
console.log('步骤1：登录');
await agent.aiAct('输入账号密码并登录');
await agent.aiWaitFor('登录成功，进入首页');

console.log('步骤2：搜索商品');
await agent.aiAct('搜索"手机"');
await agent.aiWaitFor('显示搜索结果');

console.log('步骤3：加入购物车');
await agent.aiTap('第一个商品');
await agent.aiTap('加入购物车');
await agent.aiAssert('加入成功');

// ... 后续步骤
```

### 5. 使用截图辅助调试

```javascript
async function stepWithScreenshot(agent, stepName, action) {
  console.log(`📌 ${stepName}`);
  
  // 执行前截图
  await agent.screenshot(`before_${stepName}_${Date.now()}.png`);
  
  try {
    await action();
    // 执行后截图
    await agent.screenshot(`after_${stepName}_${Date.now()}.png`);
    console.log(`✅ ${stepName} 完成`);
  } catch (error) {
    // 错误时截图
    await agent.screenshot(`error_${stepName}_${Date.now()}.png`);
    throw error;
  }
}

// 使用示例
await stepWithScreenshot(agent, '点击登录', async () => {
  await agent.aiTap('登录按钮');
});
```

---

## Token 成本控制策略

使用 AI 驱动的自动化测试会产生 API 调用费用，以下是降低成本的策略。

### 1. 优先使用即时操作

即时操作（`aiTap`、`aiInput`）比自动规划（`aiAct`）消耗更少的 Token。

```javascript
// ❌ 高消耗
await agent.aiAct('点击登录按钮');
await agent.aiAct('输入用户名');
await agent.aiAct('输入密码');
await agent.aiAct('点击确认');

// ✅ 低消耗
await agent.aiTap('登录按钮');
await agent.aiInput('用户名输入框', 'testuser');
await agent.aiInput('密码输入框', 'password');
await agent.aiTap('确认按钮');

// 或者合并为一条指令
await agent.aiAct('输入用户名 testuser，密码 password，然后点击确认登录');
```

### 2. 合并相关操作

```javascript
// ❌ 分开执行（多次 AI 调用）
await agent.aiAct('输入用户名');
await agent.aiAct('输入密码');
await agent.aiAct('勾选记住密码');
await agent.aiAct('点击登录');

// ✅ 合并执行（一次 AI 调用）
await agent.aiAct(`
  完成登录表单：
  1. 用户名输入 testuser
  2. 密码输入 Test123456
  3. 勾选"记住密码"
  4. 点击登录按钮
`);
```

### 3. 使用缓存机制

Midscene 支持缓存 AI 规划结果：

```javascript
const agent = new IOSAgent(device, {
  // 启用缓存
  cacheEnabled: true,
  cacheDir: './midscene_cache',
});

// 相同的操作指令会使用缓存，不会重复调用 AI
```

### 4. 选择合适的模型

| 场景 | 推荐模型 | Token 成本 |
|------|----------|-----------|
| 开发调试 | GPT-4o-mini | 低 |
| 简单操作 | Qwen-VL | 低 |
| 复杂场景 | GPT-4o | 中高 |
| UI 密集型 | UI-TARS | 中 |

```javascript
// 开发环境使用便宜模型
// .env.development
MIDSCENE_MODEL_NAME=gpt-4o-mini

// 生产环境使用高精度模型
// .env.production
MIDSCENE_MODEL_NAME=gpt-4o
```

### 5. 减少不必要的 Query 调用

```javascript
// ❌ 不必要的查询
const title = await agent.aiQuery('string, 页面标题');
const subTitle = await agent.aiQuery('string, 页面副标题');
const userName = await agent.aiQuery('string, 用户名');

// ✅ 合并查询
const pageInfo = await agent.aiQuery(`
  {
    title: string,
    subTitle: string,
    userName: string
  },
  获取页面标题、副标题和用户名
`);
```

### 6. 使用坐标操作替代 AI 操作

对于固定位置的元素，使用坐标操作更快更省：

```javascript
// ❌ 每次都调用 AI
await agent.aiTap('返回按钮');  // 左上角固定位置

// ✅ 使用坐标（已知位置）
await agent.tap(30, 50);  // 左上角返回按钮
```

### Token 消耗对比

| 操作类型 | 平均 Token 消耗 | 说明 |
|---------|----------------|------|
| `aiTap()` | ~200 | 即时点击 |
| `aiInput()` | ~200 | 即时输入 |
| `aiAct()` 简单 | ~500 | 单步操作 |
| `aiAct()` 复杂 | ~1000+ | 多步规划 |
| `aiQuery()` | ~300 | 数据提取 |
| `aiAssert()` | ~200 | 状态断言 |
| `aiWaitFor()` | ~200/次 | 可能多次调用 |

---

## 测试报告生成与分析

### 自动生成的报告

Midscene 会自动生成 HTML 格式的测试报告，保存在 `midscene_run/report/` 目录下。

```bash
# 查看报告目录
ls midscene_run/report/

# 打开最新报告
open midscene_run/report/$(ls -t midscene_run/report/ | head -1)
```

### 报告内容

报告包含以下信息：

1. **执行时间线**：每个操作的执行时间
2. **屏幕截图**：每个步骤的截图
3. **AI 分析结果**：AI 识别的元素和决策过程
4. **操作结果**：成功/失败状态
5. **错误信息**：失败时的详细错误

### 自定义报告

```javascript
import fs from 'fs';

async function runTestWithReport() {
  const results = [];
  const startTime = Date.now();

  try {
    // 记录每个步骤
    const step1Start = Date.now();
    await agent.aiTap('登录按钮');
    results.push({
      step: '点击登录',
      status: 'success',
      duration: Date.now() - step1Start,
    });

    // ... 更多步骤

  } catch (error) {
    results.push({
      step: '当前步骤',
      status: 'failed',
      error: error.message,
    });
  }

  // 生成报告
  const report = {
    testName: 'iOS 自动化测试',
    totalDuration: Date.now() - startTime,
    results,
    timestamp: new Date().toISOString(),
  };

  fs.writeFileSync(
    `test_report_${Date.now()}.json`,
    JSON.stringify(report, null, 2)
  );

  console.log('📊 测试报告已生成');
}
```

---

## 实战案例三：复杂业务流程

### 场景：完整的电商购物流程

```javascript
import 'dotenv/config';
import { AndroidAgent, AndroidDevice, getConnectedDevices } from '@midscene/android';

const sleep = (ms) => new Promise((r) => setTimeout(r, ms));

/**
 * 电商购物完整流程测试
 */
async function shoppingFlowTest() {
  console.log('🛒 ========== 电商购物流程测试 ==========');

  const devices = await getConnectedDevices();
  const device = new AndroidDevice(devices[0].udid);
  
  const agent = new AndroidAgent(device, {
    aiActionContext: `
      处理各种弹窗：权限请求点击允许，广告点击关闭，登录提示点击跳过。
      如果页面加载中，等待加载完成。
    `,
  });

  await device.connect();

  try {
    // ========== 第一阶段：启动和登录 ==========
    console.log('\n📱 阶段1: 启动应用');
    await agent.launch('com.jd.lib.jdmobile');  // 京东 App
    await sleep(5000);
    
    // 检查登录状态
    const isLoggedIn = await agent.aiBoolean('是否已经登录（能看到用户头像或昵称）');
    
    if (!isLoggedIn) {
      console.log('🔑 需要登录...');
      await agent.aiAct('点击"我的"，然后点击登录按钮');
      await agent.aiAct('使用手机号登录，输入 13800138000');
      // 实际场景需要处理验证码
      await sleep(5000);
    }
    
    console.log('✅ 登录状态确认');

    // ========== 第二阶段：搜索商品 ==========
    console.log('\n🔍 阶段2: 搜索商品');
    await agent.aiTap('搜索框');
    await sleep(1000);
    await agent.aiAct('输入"蓝牙耳机"并搜索');
    await agent.aiWaitFor('显示了商品搜索结果');
    
    // 筛选商品
    await agent.aiAct('点击筛选，选择价格区间 100-500 元');
    await sleep(2000);
    console.log('✅ 搜索完成');

    // ========== 第三阶段：选择商品 ==========
    console.log('\n👆 阶段3: 选择商品');
    
    // 获取商品列表
    const products = await agent.aiQuery(`
      { name: string, price: number, sales: string }[],
      获取前5个商品信息
    `);
    console.log('📦 商品列表:');
    console.table(products);
    
    // 选择评价最多的商品
    await agent.aiTap('销量最高或评价最多的商品');
    await sleep(3000);
    console.log('✅ 已进入商品详情');

    // ========== 第四阶段：加入购物车 ==========
    console.log('\n🛒 阶段4: 加入购物车');
    
    // 获取商品详情
    const detail = await agent.aiQuery(`
      { 
        title: string, 
        price: number, 
        originalPrice: number,
        stock: string
      },
      获取商品详情
    `);
    console.log('📋 商品详情:', detail);
    
    // 选择规格并加入购物车
    await agent.aiTap('加入购物车');
    await sleep(1000);
    
    // 如果有规格选择
    const hasSpecSelect = await agent.aiBoolean('是否显示了规格选择弹窗');
    if (hasSpecSelect) {
      await agent.aiAct('选择第一个可用的颜色和规格，数量选择 1');
      await agent.aiTap('确定');
    }
    
    await agent.aiWaitFor('加入购物车成功的提示');
    console.log('✅ 已加入购物车');

    // ========== 第五阶段：去结算 ==========
    console.log('\n💳 阶段5: 去结算');
    await agent.aiTap('去购物车');
    await sleep(2000);
    
    // 验证购物车
    const cartItems = await agent.aiNumber('购物车中的商品数量');
    console.log(`🛒 购物车商品数量: ${cartItems}`);
    
    await agent.aiAssert('购物车中有刚才添加的蓝牙耳机');
    
    // 选择商品结算
    await agent.aiAct('勾选刚才添加的商品');
    await agent.aiTap('去结算');
    await sleep(3000);
    console.log('✅ 进入结算页面');

    // ========== 第六阶段：确认订单 ==========
    console.log('\n📝 阶段6: 确认订单');
    
    // 检查收货地址
    const hasAddress = await agent.aiBoolean('是否已有收货地址');
    if (!hasAddress) {
      console.log('⚠️ 需要添加收货地址');
      await agent.aiAct('添加新地址，输入测试地址信息');
    }
    
    // 获取订单信息
    const orderInfo = await agent.aiQuery(`
      {
        address: string,
        totalAmount: number,
        itemCount: number
      },
      获取订单的收货地址、总金额和商品数量
    `);
    console.log('📋 订单信息:', orderInfo);
    
    // 此处不实际提交订单
    console.log('⏸️ 测试完成，不提交真实订单');

    console.log('\n🎉 购物流程测试全部完成！');

  } catch (error) {
    console.error('❌ 测试失败:', error.message);
    await agent.screenshot(`error_shopping_${Date.now()}.png`);
  } finally {
    await device.disconnect();
  }
}

shoppingFlowTest();
```

---

## 最佳实践总结

### 代码组织

```
project/
├── src/
│   ├── agents/
│   │   ├── ios-agent.js      # iOS Agent 封装
│   │   └── android-agent.js  # Android Agent 封装
│   ├── tests/
│   │   ├── login.test.js     # 登录测试
│   │   ├── search.test.js    # 搜索测试
│   │   └── shopping.test.js  # 购物测试
│   ├── utils/
│   │   ├── retry.js          # 重试工具
│   │   ├── report.js         # 报告生成
│   │   └── screenshot.js     # 截图工具
│   └── config/
│       ├── contexts.js       # AI 上下文配置
│       └── devices.js        # 设备配置
├── midscene_run/             # 自动生成的报告
├── screenshots/              # 截图保存
├── .env                      # 环境变量
└── package.json
```

### 关键原则

1. **分层设计**：将 Agent 创建、测试逻辑、工具函数分离
2. **统一配置**：通过 `aiActionContext` 统一处理弹窗
3. **适当等待**：在页面切换、网络请求后添加等待
4. **错误处理**：捕获异常并截图保存
5. **成本控制**：优先使用即时操作，合并相关指令
6. **日志清晰**：每个步骤都有明确的日志输出

---

## 总结

通过本文的学习，你应该掌握了：

✅ **iOS/Android 完整自动化流程**  
✅ **常用操作方法的使用技巧**  
✅ **稳定性优化的最佳实践**  
✅ **Token 成本控制策略**  
✅ **测试报告的生成与分析**  

## 系列回顾

本系列三篇文章完整介绍了 Midscene 移动端自动化：

📌 [第一篇：Midscene 框架介绍](/posts/15/) - 核心概念和 API
📌 [第二篇：环境配置](/posts/16/) - iOS 和 Android 配置
📌 **第三篇：实战教程**（本文） - 完整案例和最佳实践

---

## 参考资源

**官方文档**：
- [Midscene.js 官方网站](https://midscenejs.com/zh/)
- [API 参考文档](https://midscenejs.com/zh/api.html)
- [示例项目](https://github.com/web-infra-dev/midscene-example)

**相关文章**：
- [Midscene：AI驱动的移动端UI自动化框架介绍](/posts/15/)
- [Midscene移动端自动化环境配置：iOS和Android篇](/posts/16/)

---

> 💡 **温馨提示**：AI 驱动的自动化测试虽然强大，但仍需要合理的测试设计和成本控制。结合传统自动化方法，才能发挥最大效果！

> 🔥 **实践建议**：从简单场景开始，逐步积累经验，建立适合自己项目的最佳实践！

> 📚 **持续学习**：关注 Midscene 官方更新，及时了解新功能和优化！

