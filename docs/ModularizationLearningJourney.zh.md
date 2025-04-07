# 模块化学习之旅

在这个学习之旅中，你将了解模块化，以及用于创建 Now in Android 应用模块的模块化策略。


## 概述

模块化是将一个单体的、单模块代码库的概念分解为松散耦合、自包含模块的实践。


### 模块化的好处

这提供了许多好处，包括：

**可扩展性** - 在紧密耦合的代码库中，单个更改可能会触发一系列变更。一个正确模块化的项目将采用[关注点分离](https://en.wikipedia.org/wiki/Separation_of_concerns)原则。这反过来又使贡献者拥有更多自主权，同时也强制执行架构模式。

**支持并行工作** - 模块化有助于减少版本控制冲突，并使大型团队中的开发者能够更高效地并行工作。

**所有权** - 一个模块可以有一个专门的负责人，负责维护代码和测试、修复错误以及审查更改。

**封装** - 隔离的代码更容易阅读、理解、测试和维护。

**减少构建时间** - 利用 Gradle 的并行和增量构建可以减少构建时间。

**动态交付** - 模块化是[Play 功能交付](https://developer.android.com/guide/playcore/feature-delivery)的要求，它允许有条件地交付应用的某些功能或按需下载。

**可重用性** - 适当的模块化为代码共享和从相同基础构建多个应用（跨不同平台）创造了机会。


### 模块化的陷阱

然而，模块化是一种可能被误用的模式，在模块化应用时需要注意一些问题：

**模块过多** - 每个模块都有一个以构建配置增加复杂性形式出现的开销。这可能导致 Gradle 同步时间增加，并产生持续的维护成本。此外，添加更多模块会增加项目的 Gradle 设置的复杂性，与单一单体模块相比。这可以通过使用约定插件来缓解，将可重用和可组合的构建配置提取到类型安全的 Kotlin 代码中。在 Now in Android 应用中，这些约定插件可以在 [`build-logic` 文件夹](https://github.com/android/nowinandroid/tree/main/build-logic)中找到。

**模块不足** - 相反，如果你的模块很少、很大且紧密耦合，你最终会得到另一个单体。这意味着你会失去模块化的一些好处。如果你的模块臃肿且没有明确定义的单一目的，你应该考虑拆分它。

**过于复杂** - 这里没有万能的解决方案。事实上，模块化你的项目并不总是有意义的。一个主导因素是代码库的大小和相对复杂性。如果你的项目预计不会超过某个阈值，那么可扩展性和构建时间的收益将不适用。


## 模块化策略

重要的是要注意，没有一种模块化策略适合所有项目。然而，可以遵循一些通用指南，以确保你最大化其好处并最小化其缺点。

一个基本的模块只是一个包含 Gradle 构建脚本的目录。通常，一个模块会包含一个或多个源集和可能的资源或资产集合。模块可以独立构建和测试。由于 Gradle 的灵活性，你可以以多种方式组织项目。一般来说，你应该追求低耦合和高内聚。

* **低耦合** - 模块应该尽可能独立于彼此，以便对一个模块的更改对其他模块有零或最小影响。它们不应该了解其他模块的内部工作原理。

* **高内聚** - 一个模块应该包含作为一个系统运行的代码集合。它应该有明确定义的责任，并保持在特定领域知识的边界内。例如，Now in Android 中的 [`core:network` 模块](https://github.com/android/nowinandroid/tree/main/core/network)负责发出网络请求、处理来自远程数据源的响应，并向其他模块提供数据。


## Now in Android 中的模块类型

![展示 Now in Android 中模块类型及其依赖关系的图表](images/modularization-graph.drawio.png "展示 Now in Android 中模块类型及其依赖关系的图表")

**重要提示**：在模块化规划期间，模块图（如上所示）对于可视化模块之间的依赖关系非常有用。

Now in Android 应用包含以下类型的模块：

* `app` 模块 - 包含应用级别和搭建类，这些类将代码库的其余部分绑定在一起，例如 `MainActivity`、`NiaApp` 和应用级控制的导航。一个很好的例子是通过 `NiaNavHost` 设置导航和通过 `TopLevelDestination` 设置底部导航栏。`app` 模块依赖于所有 `feature` 模块和所需的 `core` 模块。

* `feature:` 模块 - 特定功能的模块，其范围限定为处理应用中的单一责任。这些模块可以被任何应用重用，包括测试或其他风格的应用，在需要时仍然保持分离和隔离。如果一个类只被一个 `feature` 模块需要，它应该保留在该模块中。如果不是，它应该被提取到适当的 `core` 模块中。一个 `feature` 模块不应该依赖于其他功能模块。它们只依赖于它们需要的 `core` 模块。

* `core:` 模块 - 包含需要在应用中其他模块之间共享的辅助代码和特定依赖的通用库模块。这些模块可以依赖于其他核心模块，但它们不应该依赖于功能或应用模块。

* 其他模块 - 如 `sync`、`benchmark` 和 `test` 模块，以及 `app-nia-catalog` - 一个用于快速显示我们设计系统的目录应用。


## 模块

使用上述模块化策略，Now in Android 应用具有以下模块：

<table>
  <tr>
   <td><strong>名称</strong>
   </td>
   <td><strong>责任</strong>
   </td>
   <td><strong>关键类和好例子</strong>
   </td>
  </tr>
  <tr>
   <td><code>app</code>
   </td>
   <td>汇集应用正常运行所需的一切。这包括 UI 脚手架和导航。
   </td>
   <td><code>NiaApp, MainActivity</code><br>
   通过 <code>NiaNavHost, NiaAppState, TopLevelDestination</code> 控制的应用级导航
   </td>
  </tr>
  <tr>
   <td><code>feature:1,</code><br>
   <code>feature:2</code><br>
   ...
   </td>
   <td>与特定功能或用户旅程相关的功能。通常包含 UI 组件和从其他模块读取数据的 ViewModels。<br>
   示例包括：<br>
   <ul>
      <li><a href="https://github.com/android/nowinandroid/tree/main/feature/topic"><code>feature:topic</code></a> 在 TopicScreen 上显示关于主题的信息。</li>
      <li><a href="https://github.com/android/nowinandroid/tree/main/feature/foryou"><code>feature:foryou</code></a> 在"为你推荐"屏幕上显示用户的新闻订阅，以及首次运行时的入门引导。</li>
      </ul>
   </td>
   <td><code>TopicScreen</code><br>
   <code>TopicViewModel</code>
   </td>
  </tr>
  <tr>
   <td><code>core:data</code>
   </td>
   <td>从多个源获取应用数据，由不同功能共享。
   </td>
   <td><code>TopicsRepository</code><br>
   </td>
  </tr>
  <tr>
   <td><code>core:designsystem</code>
   </td>
   <td>设计系统，包括核心 UI 组件（许多是自定义的 Material 3 组件）、应用主题和图标。可以通过运行 <code>app-nia-catalog</code> 运行配置查看设计系统。
   </td>
   <td>
   <code>NiaIcons</code>    <code>NiaButton</code>    <code>NiaTheme</code> 
   </td>
  </tr>
  <tr>
   <td><code>core:ui</code>
   </td>
   <td>功能模块使用的复合 UI 组件和资源，如新闻订阅。与 <code>designsystem</code> 模块不同，它依赖于数据层，因为它渲染模型，如新闻资源。
   </td>
   <td> <code>NewsFeed</code> <code>NewsResourceCardExpanded</code>
   </td>
  </tr>
  <tr>
   <td><code>core:common</code>
   </td>
   <td>模块之间共享的通用类。
   </td>
   <td><code>NiaDispatchers</code><br>
   <code>Result</code>
   </td>
  </tr>
  <tr>
   <td><code>core:network</code>
   </td>
   <td>发出网络请求并处理来自远程数据源的响应。
   </td>
   <td><code>RetrofitNiaNetworkApi</code>
   </td>
  </tr>
  <tr>
   <td><code>core:testing</code>
   </td>
   <td>测试依赖、仓库和工具类。
   </td>
   <td><code>NiaTestRunner</code><br>
   <code>TestDispatcherRule</code>
   </td>
  </tr>
  <tr>
   <td><code>core:datastore</code>
   </td>
   <td>使用 DataStore 存储持久数据。
   </td>
   <td><code>NiaPreferences</code><br>
   <code>UserPreferencesSerializer</code>
   </td>
  </tr>
  <tr>
   <td><code>core:database</code>
   </td>
   <td>使用 Room 进行本地数据库存储。
   </td>
   <td><code>NiaDatabase</code><br>
   <code>DatabaseMigrations</code><br>
   <code>Dao</code> 类
   </td>
  </tr>
  <tr>
   <td><code>core:model</code>
   </td>
   <td>整个应用中使用的模型类。
   </td>
   <td><code>Topic</code><br>
   <code>Episode</code><br>
   <code>NewsResource</code>
   </td>
  </tr>
</table>


## Now in Android 中的模块化

我们的模块化方法是考虑到"Now in Android"项目路线图、即将开展的工作和新功能而定义的。此外，我们这次的目标是在过度模块化一个相对较小的应用和利用这个机会展示一个适合生产环境中更大代码库的模块化模式之间找到适当的平衡。

这种方法与 Android 社区讨论过，并考虑了他们的反馈进行了发展。然而，在模块化方面，没有一种正确的答案使所有其他答案都变得错误。最终，有许多方式和方法来模块化应用，很少有一种方法适合所有目的、代码库和团队偏好。这就是为什么事先规划和考虑所有目标、你试图解决的问题、未来工作以及预测潜在障碍都是为了在你自己的独特情况下定义最佳结构的关键步骤。开发者可以从头脑风暴会议中受益，绘制模块和依赖关系图以更好地可视化和规划。

我们的方法就是这样一个例子 - 我们不期望它是适用于所有情况的不可变结构，实际上，它可能在未来演变和改变。这是我们发现最适合我们项目的一般指南，我们将其作为一个例子，你可以进一步修改、扩展和在此基础上构建。一种做法是增加代码库的粒度。粒度是你的代码库由模块组成的程度。如果你的数据层很小，将其保留在单个模块中是可以的。但是一旦仓库和数据源的数量开始增长，可能值得考虑将它们拆分为单独的模块。

我们也始终对你的建设性反馈持开放态度 - 向社区学习和交流想法是改进我们指导的关键要素之一。 