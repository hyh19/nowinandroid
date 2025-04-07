![Now in Android](docs/images/nia-splash.jpg "Now in Android")

<a href="https://play.google.com/store/apps/details?id=com.google.samples.apps.nowinandroid"><img src="https://play.google.com/intl/en_us/badges/static/images/badges/en_badge_web_generic.png" height="70"></a>

Now in Android 应用
==================

**在[设计案例研究](https://goo.gle/nia-figma)、[架构学习之旅](docs/ArchitectureLearningJourney.zh.md)和[模块化学习之旅](docs/ModularizationLearningJourney.zh.md)中了解该应用的设计和构建方式。**

这是 [Now in Android](https://developer.android.com/series/now-in-android) 应用的代码仓库。目前仍在**持续开发中** 🚧。

**Now in Android** 是一个完全使用 Kotlin 和 Jetpack Compose 构建的功能完善的 Android 应用。它遵循 Android 设计和开发最佳实践，旨在成为开发者的有用参考。作为一个实际运行的应用，它旨在通过提供定期的新闻更新，帮助开发者跟上 Android 开发领域的最新动态。

该应用目前正在开发中。`prodRelease` 版本[可在 Play Store 上获取](https://play.google.com/store/apps/details?id=com.google.samples.apps.nowinandroid)。

# 功能特性

**Now in Android** 展示来自 [Now in Android](https://developer.android.com/series/now-in-android) 系列的内容。用户可以浏览最近的视频、文章和其他内容的链接。用户还可以关注他们感兴趣的主题，并在发布与其关注的兴趣相匹配的新内容时收到通知。

## 截图

![展示"为你推荐"屏幕、"兴趣"屏幕和主题详情屏幕的截图](docs/images/screenshots.png '展示"为你推荐"屏幕、"兴趣"屏幕和主题详情屏幕的截图')

# 开发环境

**Now in Android** 使用 Gradle 构建系统，可以直接导入到 Android Studio 中（确保使用[此处](https://developer.android.com/studio)提供的最新稳定版本）。

将运行配置更改为 `app`。

![image](https://user-images.githubusercontent.com/873212/210559920-ef4a40c5-c8e0-478b-bb00-4879a8cf184a.png)

可以构建和运行 `demoDebug` 和 `demoRelease` 构建变体（`prod` 变体使用目前尚未公开可用的后端服务器）。

![image](https://user-images.githubusercontent.com/873212/210560507-44045dc5-b6d5-41ca-9746-f0f7acf22f8e.png)

一旦你启动并运行，可以参考下面的学习之旅，更好地了解所使用的库和工具，了解 UI、测试、架构等方面的方法背后的原因，以及项目中所有这些不同部分如何结合在一起创建一个完整的应用。

# 架构

**Now in Android** 应用遵循[官方架构指南](https://developer.android.com/topic/architecture)，并在[架构学习之旅](docs/ArchitectureLearningJourney.zh.md)中详细描述。

# 模块化

**Now in Android** 应用已完全模块化，你可以在[模块化学习之旅](docs/ModularizationLearningJourney.zh.md)中找到详细的指导和所使用的模块化策略描述。

# 构建

该应用包含常规的 `debug` 和 `release` 构建变体。

此外，`app` 的 `benchmark` 变体用于测试启动性能并生成基准配置文件（有关更多信息，请参见下文）。

`app-nia-catalog` 是一个独立的应用，展示为 **Now in Android** 定制样式的组件列表。

该应用还使用[产品风味](https://developer.android.com/studio/build/build-variants#product-flavors)来控制应用内容应该从何处加载。

`demo` 风味使用静态本地数据，允许立即构建和探索 UI。

`prod` 风味向后端服务器发出真实的网络请求，提供最新内容。目前，还没有公开可用的后端。

对于正常开发，请使用 `demoDebug` 变体。对于 UI 性能测试，请使用 `demoRelease` 变体。

# 测试

为了便于组件测试，**Now in Android** 使用 [Hilt](https://developer.android.com/training/dependency-injection/hilt-android) 进行依赖注入。

大多数数据层组件都被定义为接口。然后，具体实现（具有各种依赖关系）被绑定以向应用中的其他组件提供这些接口。
在测试中，**Now in Android** 值得注意的是**不**使用任何模拟库。相反，可以使用 Hilt 的测试 API（或通过手动构造函数注入 `ViewModel` 测试）替换生产实现。

这些测试替身实现与生产实现相同的接口，通常提供简化（但仍然现实）的实现，并附加测试钩子。
这导致测试不那么脆弱，可能会执行更多的生产代码，而不仅仅是验证对模拟的特定调用。

示例：

- 在仪器测试中，使用临时文件夹存储用户的偏好设置，该文件夹在每次测试后都会被清除。
  这允许使用真实的 `DataStore` 并执行所有相关代码，而不是模拟数据更新流。

- 每个仓库都有 `Test` 实现，它们实现正常、完整的仓库接口，并提供仅用于测试的钩子。
  `ViewModel` 测试使用这些 `Test` 仓库，因此可以使用仅用于测试的钩子来操作 `Test` 仓库的状态并验证结果行为，而不是检查是否调用了特定的仓库方法。

要运行测试，请执行以下 gradle 任务：

- `testDemoDebug` 针对 `demoDebug` 变体运行所有本地测试。截图测试将失败（请参阅下面的解释）。为避免这种情况，请在运行单元测试之前运行 `recordRoborazziDemoDebug`。
- `connectedDemoDebugAndroidTest` 针对 `demoDebug` 变体运行所有仪器测试。

> [!Note]
> 你不应该运行 `./gradlew test` 或 `./gradlew connectedAndroidTest`，因为这将针对**所有**构建变体执行测试，这既不必要，也会导致失败，因为只有 `demoDebug` 变体受支持。其他变体没有任何测试（尽管这在将来可能会改变）。

## 截图测试

截图测试会对应用中的屏幕或 UI 组件进行截图，并将其与已知正确渲染的先前记录的截图进行比较。

例如，Now in Android 有[截图测试](https://github.com/android/nowinandroid/blob/main/app/src/testDemo/kotlin/com/google/samples/apps/nowinandroid/ui/NiaAppScreenSizesScreenshotTests.kt)来验证导航在不同屏幕尺寸上是否正确显示（[已知正确的截图](https://github.com/android/nowinandroid/tree/main/app/src/testDemo/screenshots)）。

Now In Android 使用 [Roborazzi](https://github.com/takahirom/roborazzi) 对某些屏幕和 UI 组件运行截图测试。在处理截图测试时，以下 gradle 任务很有用：

- `verifyRoborazziDemoDebug` 运行所有截图测试，根据已知正确的截图验证截图。
- `recordRoborazziDemoDebug` 记录新的"已知正确"截图。当你对 UI 进行更改并手动验证它们正确渲染时，使用此命令。截图将存储在 `modulename/src/test/screenshots` 中。
- `compareRoborazziDemoDebug` 在失败的测试和已知正确的图像之间创建比较图像。这些也可以在 `modulename/src/test/screenshots` 中找到。

> [!Note]
> **关于失败的截图测试的说明**
> 此代码库中存储的已知正确截图是在 CI 上使用 Linux 录制的。其他平台可能（很可能会）生成略有不同的图像，导致截图测试失败。在非 Linux 平台上工作时，解决这个问题的一种方法是在开始工作之前在 `main` 分支上运行 `recordRoborazziDemoDebug`。在做出更改后，`verifyRoborazziDemoDebug` 将仅识别合法更改。

有关截图测试的更多信息，[请查看这个演讲](https://www.droidcon.com/2023/11/15/easy-screenshot-testing-with-compose/)。

# UI

该应用使用 [Material 3 指南](https://m3.material.io/)设计。在 [Now in Android Material 3 案例研究](https://goo.gle/nia-figma)中了解更多关于设计过程的信息并获取设计文件（设计资产[也可作为 PDF 获取](docs/Now-In-Android-Design-File.pdf)）。

屏幕和 UI 元素完全使用 [Jetpack Compose](https://developer.android.com/jetpack/compose) 构建。

该应用有两个主题：

- 动态颜色 - 使用基于[用户当前颜色主题](https://material.io/blog/announcing-material-you)的颜色（如果支持）
- 默认主题 - 在不支持动态颜色时使用预定义颜色

每个主题还支持暗黑模式。

该应用使用自适应布局来[支持不同的屏幕尺寸](https://developer.android.com/guide/topics/large-screens/support-different-screen-sizes)。

在[这里](docs/ArchitectureLearningJourney.zh.md#ui-layer)了解更多关于 UI 架构的信息。

# 性能

## 基准测试

在 `benchmarks` 模块中查找使用 [`Macrobenchmark`](https://developer.android.com/topic/performance/benchmarking/macrobenchmark-overview) 编写的所有测试。该模块还包含生成基准配置文件的测试。

## 基准配置文件

该应用的基准配置文件位于 [`app/src/main/baseline-prof.txt`](app/src/main/baseline-prof.txt)。
它包含允许在应用启动期间对关键用户路径进行 AOT 编译的规则。
有关基准配置文件的更多信息，请阅读[此文档](https://developer.android.com/studio/profile/baselineprofiles)。

> [!Note]
> 对于涉及更改应用启动代码的发布版本，需要重新生成基准配置文件。

要生成基准配置文件，请选择 `benchmark` 构建变体，并在 AOSP Android 模拟器上运行 `BaselineProfileGenerator` 基准测试。
然后将模拟器上生成的基准配置文件复制到 [`app/src/main/baseline-prof.txt`](app/src/main/baseline-prof.txt)。

## Compose 编译器指标

运行以下命令获取和分析 Compose 编译器指标：

```bash
./gradlew assembleRelease -PenableComposeCompilerMetrics=true -PenableComposeCompilerReports=true
```

报告文件将添加到 [build/compose-reports](build/compose-reports)。指标文件也将添加到 [build/compose-metrics](build/compose-metrics)。

有关 Compose 编译器指标的更多信息，请参阅[这篇博客文章](https://medium.com/androiddevelopers/jetpack-compose-stability-explained-79c10db270c8)。

# 许可证

**Now in Android** 根据 Apache License（版本 2.0）的条款分发。有关更多信息，请参阅[许可证](LICENSE)。
