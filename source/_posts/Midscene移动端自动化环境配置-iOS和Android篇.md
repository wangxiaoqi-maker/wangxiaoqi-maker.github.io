---
title: Midscene移动端自动化环境配置：iOS和Android篇
date: 2025-12-21 11:00:00
tags:
  - AI
  - 自动化测试
  - Midscene
  - iOS
  - Android
  - WebDriverAgent
  - adb
categories: 测试
comments: true
cover: https://images.unsplash.com/photo-1512941937669-90a1b58e7e9c?w=800&q=80
abbrlink: 16
---

# 前言

在上一篇 [《Midscene：AI驱动的移动端UI自动化框架介绍》](/posts/15/) 中，我们了解了 Midscene 的核心概念和优势。但要真正使用 Midscene 进行移动端自动化测试，首先需要配置好开发环境。

本文是 **Midscene 移动端自动化系列** 的第二篇，将详细介绍如何配置 iOS 和 Android 的自动化测试环境，包括：

- 🍎 **iOS 环境配置**：WebDriverAgent 安装与配置
- 🤖 **Android 环境配置**：adb 工具链安装与设备连接
- 🔧 **AI 模型配置**：各种视觉语言模型的接入方式
- ❓ **常见问题排查**：环境配置中的坑和解决方案

---

## iOS 环境配置

### 前置条件

iOS 自动化测试必须在 **macOS** 系统上进行，需要准备以下环境：

| 要求 | 说明 |
|------|------|
| 操作系统 | macOS（Monterey 12.0 或更高版本） |
| Xcode | 14.0 或更高版本（含命令行工具） |
| Node.js | 18.0 或更高版本 |
| 设备 | iOS 模拟器或真机（iOS 15+） |

### 第一步：安装 Xcode

1. 打开 **App Store**，搜索 **Xcode** 并安装
2. 安装完成后，打开终端执行以下命令安装命令行工具：

```bash
xcode-select --install
```

3. 验证安装：

```bash
xcode-select -p
# 输出：/Applications/Xcode.app/Contents/Developer
```

### 第二步：安装 Node.js

推荐使用 nvm 管理 Node.js 版本：

```bash
# 安装 nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# 重新加载终端配置
source ~/.zshrc  # 或 source ~/.bash_profile

# 安装 Node.js 18
nvm install 18
nvm use 18

# 验证安装
node -v  # 应该输出 v18.x.x
npm -v   # 应该输出 9.x.x 或更高
```

### 第三步：配置 WebDriverAgent

WebDriverAgent（WDA）是 Facebook 开发的用于 iOS 自动化测试的框架，Midscene 通过它与 iOS 设备通信。

> ⚠️ **版本要求**：WebDriverAgent 版本需要 **>= 7.0.0**

#### 方法一：使用 Homebrew 安装（推荐）

```bash
# 安装 Homebrew（如果没有）
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 安装 ios-deploy（用于真机部署）
brew install ios-deploy

# 安装 ideviceinstaller（用于真机管理）
brew install ideviceinstaller

# 安装 libimobiledevice（iOS 设备通信库）
brew install libimobiledevice
```

#### 方法二：手动编译 WebDriverAgent

1. **克隆仓库**：

```bash
git clone https://github.com/appium/WebDriverAgent.git
cd WebDriverAgent
```

2. **打开 Xcode 项目**：

```bash
open WebDriverAgent.xcodeproj
```

3. **配置签名**：

   - 在 Xcode 中选择 `WebDriverAgentRunner` target
   - 点击 **Signing & Capabilities**
   - 选择你的 Apple 开发者账号（可以是免费账号）
   - 修改 Bundle Identifier 为唯一值，如 `com.yourname.WebDriverAgentRunner`

4. **构建 WDA**：

   - 选择目标设备（模拟器或真机）
   - 点击 **Product → Build** 或按 `Cmd + B`

#### 方法三：使用预编译的 WDA（模拟器推荐）

对于模拟器，可以使用预编译的 WDA：

```bash
# 安装 Appium
npm install -g appium

# 安装 XCUITest 驱动（包含预编译 WDA）
appium driver install xcuitest

# 启动 Appium 服务（会自动管理 WDA）
appium
```

### 第四步：启动 WebDriverAgent

#### 模拟器启动

1. 打开 Xcode，选择模拟器
2. 运行 WebDriverAgent 项目（选择 `WebDriverAgentRunner` scheme）
3. 或使用命令行：

```bash
# 列出可用的模拟器
xcrun simctl list devices

# 启动指定模拟器
xcrun simctl boot "iPhone 15 Pro"

# 使用 xcodebuild 启动 WDA
xcodebuild -project WebDriverAgent.xcodeproj \
  -scheme WebDriverAgentRunner \
  -destination 'platform=iOS Simulator,name=iPhone 15 Pro' \
  test
```

#### 真机启动

1. 确保真机已连接并信任此 Mac
2. 在手机上开启 **开发者模式**：
   - 设置 → 隐私与安全性 → 开发者模式 → 开启
3. 开启 **UI Automation**：
   - 设置 → 开发者 → UI Automation → 开启
4. 在 Xcode 中运行 WDA：

```bash
xcodebuild -project WebDriverAgent.xcodeproj \
  -scheme WebDriverAgentRunner \
  -destination 'id=<设备UDID>' \
  test
```

获取设备 UDID：

```bash
# 方法1：使用 instruments
instruments -s devices

# 方法2：使用 idevice_id
idevice_id -l

# 方法3：使用 Xcode
# Window → Devices and Simulators → 选择设备查看 Identifier
```

### 第五步：验证 WDA 服务

WDA 启动成功后，会在终端显示类似信息：

```
ServerURLHere->http://192.168.1.100:8100<-ServerURLHere
```

访问状态接口验证：

```bash
curl http://localhost:8100/status
```

**正确响应示例**：

```json
{
  "value": {
    "build": {
      "version": "10.1.1",
      "productBundleIdentifier": "com.facebook.WebDriverAgentRunner"
    },
    "os": {
      "name": "iOS",
      "version": "17.0"
    },
    "device": "iphone",
    "message": "WebDriverAgent is ready to accept commands",
    "state": "success",
    "ready": true
  }
}
```

### 真机端口转发

真机需要使用 `iproxy` 进行端口转发：

```bash
# 安装 iproxy（包含在 libimobiledevice 中）
brew install libimobiledevice

# 转发端口
iproxy 8100 8100 <设备UDID>

# 后台运行
nohup iproxy 8100 8100 <设备UDID> &
```

---

## Android 环境配置

### 前置条件

Android 自动化可以在 **Windows**、**macOS** 或 **Linux** 上进行：

| 要求 | 说明 |
|------|------|
| 操作系统 | Windows 10+、macOS 10.15+、Ubuntu 20.04+ |
| Node.js | 18.0 或更高版本 |
| Android SDK | 包含 adb（platform-tools） |
| 设备 | Android 模拟器或真机（Android 8+） |

### 第一步：安装 adb

adb（Android Debug Bridge）是与 Android 设备通信的核心工具。

#### macOS 安装

```bash
# 使用 Homebrew 安装（推荐）
brew install android-platform-tools

# 验证安装
adb version
# 输出：Android Debug Bridge version 1.0.41
```

#### Windows 安装

1. 下载 [Android SDK Platform-Tools](https://developer.android.com/studio/releases/platform-tools)
2. 解压到指定目录，如 `C:\android\platform-tools`
3. 添加到系统 PATH：
   - 右键"此电脑" → 属性 → 高级系统设置 → 环境变量
   - 在 Path 中添加 `C:\android\platform-tools`
4. 重启终端验证：

```bash
adb version
```

#### Linux 安装

```bash
# Ubuntu/Debian
sudo apt-get update
sudo apt-get install android-tools-adb

# 或使用官方工具
wget https://dl.google.com/android/repository/platform-tools-latest-linux.zip
unzip platform-tools-latest-linux.zip
sudo mv platform-tools /opt/android-sdk/
echo 'export PATH=$PATH:/opt/android-sdk/platform-tools' >> ~/.bashrc
source ~/.bashrc

# 验证安装
adb version
```

### 第二步：准备 Android 设备

#### 真机配置

1. **开启开发者选项**：
   - 设置 → 关于手机 → 连续点击"版本号" 7 次
   - 返回设置，会看到"开发者选项"

2. **开启 USB 调试**：
   - 设置 → 开发者选项 → USB 调试 → 开启

3. **连接设备**：
   - 使用数据线连接手机和电脑
   - 手机上会弹出授权提示，点击"允许"

4. **验证连接**：

```bash
adb devices
```

**正确输出**：

```
List of devices attached
ABC123456789    device
```

如果状态是 `unauthorized`，需要在手机上授权。

#### 模拟器配置

**使用 Android Studio 模拟器**：

1. 安装 [Android Studio](https://developer.android.com/studio)
2. 打开 AVD Manager（Tools → Device Manager）
3. 创建新的虚拟设备
4. 启动模拟器
5. 验证连接：

```bash
adb devices
# 输出：emulator-5554    device
```

**使用命令行创建模拟器**：

```bash
# 列出可用的系统镜像
sdkmanager --list | grep system-images

# 安装系统镜像
sdkmanager "system-images;android-34;google_apis;x86_64"

# 创建 AVD
avdmanager create avd -n Pixel_7_API_34 -k "system-images;android-34;google_apis;x86_64"

# 启动模拟器
emulator -avd Pixel_7_API_34
```

### 第三步：配置 adb 无线调试（可选）

支持 Android 11+ 的设备可以使用无线调试：

```bash
# 方法一：通过 USB 配置
# 1. 先通过 USB 连接设备
adb tcpip 5555

# 2. 断开 USB，使用 WiFi 连接
adb connect <手机IP地址>:5555

# 方法二：使用 Android 11+ 无线调试
# 1. 开启开发者选项 → 无线调试
# 2. 点击"使用配对码配对设备"
# 3. 在电脑上执行
adb pair <手机IP>:<配对端口>
# 4. 输入手机显示的配对码
# 5. 连接
adb connect <手机IP>:<连接端口>
```

### 第四步：验证 adb 连接

```bash
# 查看已连接设备
adb devices

# 获取设备信息
adb shell getprop ro.product.model
adb shell getprop ro.build.version.release

# 测试截图功能
adb shell screencap -p /sdcard/screenshot.png
adb pull /sdcard/screenshot.png ./test_screenshot.png
```

---

## Midscene 项目配置

### 第一步：创建项目

```bash
# 创建项目目录
mkdir midscene-mobile-demo
cd midscene-mobile-demo

# 初始化 npm 项目
npm init -y

# 安装 Midscene 依赖
npm install @midscene/ios @midscene/android @midscene/core dotenv

# 创建 package.json 配置
```

**package.json 配置**：

```json
{
  "name": "midscene-mobile-demo",
  "version": "1.0.0",
  "description": "Midscene.js 移动端自动化 Demo",
  "type": "module",
  "scripts": {
    "ios": "node demo-ios.js",
    "android": "node demo-android.js",
    "ios:debug": "DEBUG=midscene* node demo-ios.js",
    "android:debug": "DEBUG=midscene* node demo-android.js"
  },
  "dependencies": {
    "@midscene/android": "^1.0.2",
    "@midscene/core": "^1.0.0",
    "@midscene/ios": "^1.0.2",
    "dotenv": "^16.3.1"
  }
}
```

### 第二步：配置 AI 模型

创建 `.env` 文件：

```bash
touch .env
```

**OpenAI GPT-4o 配置**：

```env
MIDSCENE_MODEL_BASE_URL=https://api.openai.com/v1
MIDSCENE_MODEL_API_KEY=sk-your-openai-api-key
MIDSCENE_MODEL_NAME=gpt-4o
MIDSCENE_MODEL_FAMILY=openai
```

**阿里云千问 Qwen-VL 配置**：

```env
MIDSCENE_MODEL_BASE_URL=https://dashscope.aliyuncs.com/compatible-mode/v1
MIDSCENE_MODEL_API_KEY=sk-your-qwen-api-key
MIDSCENE_MODEL_NAME=qwen-vl-max
MIDSCENE_MODEL_FAMILY=qwen
```

**字节豆包 UI-TARS 配置**：

```env
MIDSCENE_MODEL_BASE_URL=https://ark.cn-beijing.volces.com/api/v3
MIDSCENE_MODEL_API_KEY=your-doubao-api-key
MIDSCENE_MODEL_NAME=ep-your-endpoint-id
MIDSCENE_MODEL_FAMILY=doubao
```

**智谱 GLM-4V 配置**：

```env
MIDSCENE_MODEL_BASE_URL=https://open.bigmodel.cn/api/paas/v4
MIDSCENE_MODEL_API_KEY=your-zhipu-api-key
MIDSCENE_MODEL_NAME=glm-4v
MIDSCENE_MODEL_FAMILY=zhipu
```

### 第三步：创建测试脚本

**iOS 测试脚本 (demo-ios.js)**：

```javascript
import 'dotenv/config';
import { IOSAgent, IOSDevice } from '@midscene/ios';

const sleep = (ms) => new Promise((r) => setTimeout(r, ms));

(async () => {
  console.log('🍎 开始 iOS 自动化测试...');
  
  // 创建设备连接
  const device = new IOSDevice({
    wdaPort: 8100,
    wdaHost: 'localhost',
  });

  // 创建 Agent
  const agent = new IOSAgent(device, {
    aiActionContext: '如果弹出权限请求，请点击同意。如果出现登录弹窗，请关闭。',
  });

  try {
    // 连接设备
    await device.connect();
    console.log('✅ 设备连接成功');

    // 打开 Safari 并访问网页
    await device.launch('https://www.baidu.com');
    await sleep(3000);
    console.log('✅ 已打开百度');

    // 执行搜索
    await agent.aiAct('在搜索框中输入"Midscene 自动化"并点击搜索');
    await sleep(2000);

    // 验证结果
    await agent.aiAssert('页面显示了搜索结果');
    console.log('✅ 搜索成功');

    // 提取数据
    const results = await agent.aiQuery(
      '{title: string}[], 获取前3条搜索结果的标题'
    );
    console.log('📊 搜索结果:', results);

  } catch (error) {
    console.error('❌ 测试失败:', error.message);
  } finally {
    await device.destroy();
    console.log('🏁 测试完成');
  }
})();
```

**Android 测试脚本 (demo-android.js)**：

```javascript
import 'dotenv/config';
import { AndroidAgent, AndroidDevice, getConnectedDevices } from '@midscene/android';

const sleep = (ms) => new Promise((r) => setTimeout(r, ms));

(async () => {
  console.log('🤖 开始 Android 自动化测试...');

  try {
    // 获取已连接设备
    const devices = await getConnectedDevices();
    if (devices.length === 0) {
      throw new Error('没有找到已连接的 Android 设备');
    }
    console.log('📱 发现设备:', devices[0].udid);

    // 创建设备连接
    const device = new AndroidDevice(devices[0].udid);

    // 创建 Agent
    const agent = new AndroidAgent(device, {
      aiActionContext: '如果弹出权限请求，请点击同意。如果出现登录弹窗，请关闭。',
    });

    // 连接设备
    await device.connect();
    console.log('✅ 设备连接成功');

    // 启动 Chrome 浏览器
    await agent.launch('com.android.chrome');
    await sleep(3000);
    console.log('✅ 已打开 Chrome');

    // 访问网页
    await agent.aiAct('点击地址栏，输入 baidu.com，然后回车');
    await sleep(3000);

    // 执行搜索
    await agent.aiAct('在搜索框中输入"Midscene 自动化"并点击搜索');
    await sleep(2000);

    // 验证结果
    await agent.aiAssert('页面显示了搜索结果');
    console.log('✅ 搜索成功');

    // 提取数据
    const results = await agent.aiQuery(
      '{title: string}[], 获取前3条搜索结果的标题'
    );
    console.log('📊 搜索结果:', results);

  } catch (error) {
    console.error('❌ 测试失败:', error.message);
  } finally {
    console.log('🏁 测试完成');
  }
})();
```

### 第四步：运行测试

```bash
# 运行 iOS 测试
npm run ios

# 运行 Android 测试
npm run android

# 带调试日志运行
npm run ios:debug
npm run android:debug
```

---

## Playground 快速体验

在编写脚本之前，可以使用 Midscene Playground 快速验证环境配置。

### iOS Playground

```bash
npx --yes @midscene/ios-playground
```

### Android Playground

```bash
npx --yes @midscene/android-playground
```

**Playground 功能**：

| Tab | 功能 | 对应 API |
|-----|------|----------|
| Act | 执行操作 | `aiAct()` |
| Tap | 点击元素 | `aiTap()` |
| Query | 提取数据 | `aiQuery()` |
| Assert | 验证状态 | `aiAssert()` |

**使用步骤**：

1. 启动 Playground
2. 点击齿轮图标配置 API Key
3. 在对应 Tab 输入自然语言指令
4. 点击执行，观察结果

---

## 常见问题排查

### iOS 常见问题

#### Q1: WebDriverAgent 编译失败？

**A**: 检查以下几点：

1. **签名问题**：确保选择了有效的开发者账号
2. **Bundle ID 冲突**：修改为唯一的 Bundle Identifier
3. **Xcode 版本**：确保 Xcode 版本与 iOS 版本兼容

```bash
# 清理构建缓存
xcodebuild clean -project WebDriverAgent.xcodeproj -scheme WebDriverAgentRunner
```

#### Q2: 真机无法连接？

**A**: 检查以下设置：

1. **开发者模式**：设置 → 隐私与安全性 → 开发者模式 → 开启
2. **UI Automation**：设置 → 开发者 → UI Automation → 开启
3. **信任此电脑**：重新插拔 USB，在手机上点击"信任"
4. **端口转发**：确保 iproxy 正在运行

```bash
# 检查端口转发状态
lsof -i :8100
```

#### Q3: 模拟器与真机有什么区别？

| 特性 | 真机 | 模拟器 |
|------|------|--------|
| 端口转发 | 需要 iproxy | 不需要 |
| 开发者模式 | 需手动开启 | 默认开启 |
| UI Automation | 需手动开启 | 默认开启 |
| 性能 | 真实设备性能 | 取决于 Mac |
| 传感器 | 真实硬件 | 模拟数据 |

### Android 常见问题

#### Q1: adb 找不到设备？

**A**: 按以下步骤排查：

```bash
# 1. 重启 adb 服务
adb kill-server
adb start-server

# 2. 检查设备
adb devices

# 3. 如果状态是 unauthorized
# 在手机上检查是否有授权弹窗，点击"允许"

# 4. 如果状态是 offline
# 尝试更换 USB 线或端口
```

#### Q2: USB 连接模式问题？

**A**: 连接手机后，在通知栏选择：
- "文件传输"（MTP）模式
- 或 "仅充电" 模式（需开启调试）

#### Q3: 如何指定特定设备？

**A**: 多设备时需要指定 UDID：

```javascript
// 获取设备列表
const devices = await getConnectedDevices();
console.log(devices);
// [{ udid: 'ABC123', model: 'Pixel 7' }, { udid: 'DEF456', model: 'Pixel 8' }]

// 指定设备
const device = new AndroidDevice('ABC123');
```

### AI 模型常见问题

#### Q1: API 调用失败？

**A**: 检查以下配置：

1. **API Key 正确**：确保没有多余空格
2. **URL 正确**：不同模型 URL 不同
3. **网络连通**：确保能访问 API 端点
4. **余额充足**：检查账户余额

```bash
# 测试 API 连通性
curl -X POST https://api.openai.com/v1/chat/completions \
  -H "Authorization: Bearer $MIDSCENE_MODEL_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model": "gpt-4o", "messages": [{"role": "user", "content": "Hello"}]}'
```

#### Q2: 如何减少 Token 消耗？

**A**: 参考以下策略：

1. 使用 `aiTap()` 替代 `aiAct()` 进行简单点击
2. 合并多个操作为一条指令
3. 使用缓存功能
4. 选择合适的模型（如 gpt-4o-mini）

---

## 环境配置清单

### iOS 配置清单

- [ ] macOS 系统
- [ ] Xcode 14+ 已安装
- [ ] Xcode Command Line Tools 已安装
- [ ] Node.js 18+ 已安装
- [ ] WebDriverAgent 已编译
- [ ] 真机/模拟器已准备
- [ ] 开发者模式已开启（真机）
- [ ] UI Automation 已开启（真机）
- [ ] iproxy 端口转发已配置（真机）
- [ ] WDA 服务已启动
- [ ] AI 模型 API Key 已配置

### Android 配置清单

- [ ] Node.js 18+ 已安装
- [ ] adb 已安装并配置 PATH
- [ ] Android 设备已准备
- [ ] USB 调试已开启
- [ ] 设备已授权调试
- [ ] adb devices 显示 device 状态
- [ ] AI 模型 API Key 已配置

---

## 总结

本文详细介绍了 Midscene 移动端自动化的环境配置：

✅ **iOS 环境**：Xcode、WebDriverAgent、端口转发  
✅ **Android 环境**：adb 安装、设备连接、调试配置  
✅ **AI 模型**：多种模型的配置方式  
✅ **项目配置**：package.json、.env、测试脚本  
✅ **问题排查**：常见问题的解决方案  

## 系列预告

下一篇我们将进入**实战篇**：

📌 **Midscene 移动端自动化实战**
- 完整的电商 App 自动化测试案例
- 复杂场景处理（弹窗、加载、滚动）
- 测试报告生成与分析
- 稳定性优化技巧
- Token 成本控制策略

敬请期待！💪

---

## 参考资源

**官方文档**：
- [Midscene iOS 快速开始](https://midscenejs.com/zh/ios-getting-started.html)
- [Midscene Android 快速开始](https://midscenejs.com/zh/android-getting-started.html)
- [WebDriverAgent 官方文档](https://github.com/appium/WebDriverAgent)
- [Android Platform Tools](https://developer.android.com/studio/releases/platform-tools)

**相关文章**：
- [Midscene：AI驱动的移动端UI自动化框架介绍](/posts/15/)

---

> 💡 **温馨提示**：环境配置是自动化测试的基础，花时间配置好环境，后续开发会事半功倍！

> 🔥 **推荐阅读**：如果你还没看过框架介绍，建议先阅读 [《Midscene：AI驱动的移动端UI自动化框架介绍》](/posts/15/)

> 📚 **系列文章**：本系列会持续更新，记得关注后续的实战篇！

