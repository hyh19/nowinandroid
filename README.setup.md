# Now in Android 项目设置指南

## 环境参数

### 系统环境

- 操作系统：Darwin MacBook-Pro.local 24.3.0 (macOS)
- 处理器架构：arm64

### 开发工具

- JDK 版本：OpenJDK 23.0.2 (Homebrew)
- Gradle 版本：8.13
- Kotlin 版本：2.0.21
- Android SDK 路径：/Users/hyh/Library/Android/sdk

### 测试设备

- 设备名称：Pixel_3a_API_34_extension_level_7_arm64-v8a (AVD)
- Android 版本：14 (API 34)
- CPU 架构：arm64-v8a

## 构建与安装过程

### 1. 项目概述

Now in Android 应用有多种构建变体：

- 构建类型：debug 和 release
- 产品风味：demo（使用本地静态数据）和 prod（连接真实后端服务器）

根据官方建议，开发时应使用 `demoDebug` 变体，性能测试时使用 `demoRelease` 变体。

### 2. 构建应用

使用以下命令构建应用：

```bash
./gradlew assembleDemoDebug
```

这个命令编译源代码并创建 APK 文件，但不会安装到设备上。构建输出位于 `app/build/outputs/apk/demo/debug/` 目录下。

### 3. 构建并安装应用

要构建并直接安装到连接的设备或模拟器上，使用以下命令：

```bash
./gradlew installDemoDebug
```

该命令会先构建应用，然后通过 ADB 安装到所有连接的设备上。

### 4. 运行应用

安装完成后，可以在设备上手动启动应用，或使用以下命令启动：

```bash
adb shell am start -n com.google.samples.apps.nowinandroid.demo.debug/com.google.samples.apps.nowinandroid.MainActivity
```

也可以通过以下命令查找正确的包名和活动名：

```bash
# 查找已安装的相关包
adb shell pm list packages | grep nowin

# 查看主活动信息
adb shell dumpsys package com.google.samples.apps.nowinandroid.demo.debug | grep -A3 "android.intent.action.MAIN"
```

### 5. 常见问题排查

- **构建失败**：确保已安装所有必要的 SDK 组件和 Build Tools
- **安装失败**：检查设备连接状态，可使用 `adb devices` 命令验证
- **应用崩溃**：查看日志 `adb logcat | grep nowinandroid`
- **找不到主活动**：debug变体的包名会在原有包名后附加`.debug`后缀

## 其他有用命令

### 清理项目

```bash
./gradlew clean
```

### 运行单元测试

```bash
./gradlew testDemoDebug
```

### 运行 UI 测试

```bash
./gradlew connectedDemoDebugAndroidTest
```

### 生成代码分析报告

```bash
./gradlew assembleDemoDebug -PenableComposeCompilerMetrics=true -PenableComposeCompilerReports=true
```
