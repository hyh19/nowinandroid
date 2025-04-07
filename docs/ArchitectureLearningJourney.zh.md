# 架构学习之旅

在这个学习之旅中，你将了解 Now in Android 应用架构：其层次、关键类及它们之间的交互。

## 目标和要求

应用架构的目标是：

* 尽可能严格地遵循[官方架构指南](https://developer.android.com/jetpack/guide)。
* 让开发者易于理解，没有过于实验性的内容。
* 支持多个开发者在同一代码库上工作。
* 便于在开发者机器上和使用持续集成 (CI) 进行本地和仪器测试。
* 最小化构建时间。

## 架构概述

应用架构有三层：[数据层](https://developer.android.com/jetpack/guide/data-layer)、[领域层](https://developer.android.com/jetpack/guide/domain-layer)和[UI层](https://developer.android.com/jetpack/guide/ui-layer)。

<center>
<img src="images/architecture-1-overall.png" width="600px" alt="展示整体应用架构的图表" />
</center>

> [!Note]  
> 官方Android架构与其他架构（如"Clean Architecture"）不同。其他架构中的概念可能不适用于此处，或以不同方式应用。[这里有更多讨论](https://github.com/android/nowinandroid/discussions/1273)。

该架构采用具有[单向数据流](https://developer.android.com/jetpack/guide/ui-layer#udf)的响应式编程模型。以数据层为底层，关键概念是：

* 高层对低层的变化做出反应。
* 事件向下流动。
* 数据向上流动。

数据流通过使用[Kotlin Flows](https://developer.android.com/kotlin/flow)实现的流来实现。

### 示例：在"为你推荐"屏幕上显示新闻

首次运行应用时，它将尝试从远程服务器加载新闻资源列表（当选择`prod`构建风味时，`demo`构建将使用本地数据）。加载后，这些内容会根据用户选择的兴趣展示给用户。

下图显示了发生的事件以及相关对象之间数据流动的方式。

![展示新闻资源如何在"为你推荐"屏幕上显示的图表](images/architecture-2-example.png "展示新闻资源如何在"为你推荐"屏幕上显示的图表")

以下是每个步骤发生的情况。找到相关代码的最简单方法是将项目加载到Android Studio并搜索"代码"列中的文本（方便的快捷键：按两次<kbd>⇧ SHIFT</kbd>）。

<table>
  <tr>
   <td><strong>步骤</strong>
   </td>
   <td><strong>描述</strong>
   </td>
   <td><strong>代码</strong>
   </td>
  </tr>
  <tr>
   <td>1
   </td>
   <td>在应用启动时，将同步所有仓库的<a href="https://developer.android.com/topic/libraries/architecture/workmanager">WorkManager</a>任务排入队列。
   </td>
   <td><code>Sync.initialize</code>
   </td>
  </tr>
  <tr>
   <td>2
   </td>
   <td><code>ForYouViewModel</code>调用<code>GetUserNewsResourcesUseCase</code>获取带有书签/已保存状态的新闻资源流。在用户和新闻仓库都发出条目之前，不会有任何条目被发送到这个流中。在等待期间，Feed状态设置为<code>Loading</code>。
   </td>
   <td>搜索<code>NewsFeedUiState.Loading</code>的使用
   </td>
  </tr>
  <tr>
   <td>3
   </td>
   <td>用户数据仓库从Proto DataStore支持的本地数据源获取<code>UserData</code>对象流。
   </td>
   <td><code>NiaPreferencesDataSource.userData</code>
   </td>
  </tr>
  <tr>
   <td>4
   </td>
   <td>WorkManager执行同步任务，调用<code>OfflineFirstNewsRepository</code>开始与远程数据源同步数据。
   </td>
   <td><code>SyncWorker.doWork</code>
   </td>
  </tr>
  <tr>
   <td>5
   </td>
   <td><code>OfflineFirstNewsRepository</code>调用<code>RetrofitNiaNetwork</code>使用<a href="https://square.github.io/retrofit/">Retrofit</a>执行实际的API请求。
   </td>
   <td><code>OfflineFirstNewsRepository.syncWith</code>
   </td>
  </tr>
  <tr>
   <td>6
   </td>
   <td><code>RetrofitNiaNetwork</code>在远程服务器上调用REST API。
   </td>
   <td><code>RetrofitNiaNetwork.getNewsResources</code>
   </td>
  </tr>
  <tr>
   <td>7
   </td>
   <td><code>RetrofitNiaNetwork</code>从远程服务器接收网络响应。
   </td>
   <td><code>RetrofitNiaNetwork.getNewsResources</code>
   </td>
  </tr>
  <tr>
   <td>8
   </td>
   <td><code>OfflineFirstNewsRepository</code>通过在本地<a href="https://developer.android.com/training/data-storage/room">Room数据库</a>中插入、更新或删除数据，将远程数据与<code>NewsResourceDao</code>同步。
   </td>
   <td><code>OfflineFirstNewsRepository.syncWith</code>
   </td>
  </tr>
  <tr>
   <td>9
   </td>
   <td>当<code>NewsResourceDao</code>中的数据发生变化时，它会被发送到新闻资源数据流中（这是一个<a href="https://developer.android.com/kotlin/flow">Flow</a>）。
   </td>
   <td><code>NewsResourceDao.getNewsResources</code>
   </td>
  </tr>
  <tr>
   <td>10
   </td>
   <td><code>OfflineFirstNewsRepository</code>在这个流上充当<a href="https://developer.android.com/kotlin/flow#modify">中间操作符</a>，将传入的<code>PopulatedNewsResource</code>（数据层内部的数据库模型）转换为被其他层消费的公共<code>NewsResource</code>模型。
   </td>
   <td><code>OfflineFirstNewsRepository.getNewsResources</code>
   </td>
  </tr>
  <tr>
   <td>11
   </td>
   <td><code>GetUserNewsResourcesUseCase</code>将新闻资源列表与用户数据结合，发出<code>UserNewsResource</code>列表。  
   </td>
   <td><code>GetUserNewsResourcesUseCase.invoke</code>
   </td>
  </tr>
  <tr>
   <td>12
   </td>
   <td>当<code>ForYouViewModel</code>接收到可保存的新闻资源时，它将Feed状态更新为<code>Success</code>。

  <code>ForYouScreen</code>然后使用状态中的可保存新闻资源来渲染屏幕。
   </td>
   <td>搜索<code>NewsFeedUiState.Success</code>的实例
   </td>
  </tr>
</table>

## 数据层

数据层被实现为应用数据和业务逻辑的离线优先源。它是应用中所有数据的真实来源。

![展示数据层架构的图表](images/architecture-3-data-layer.png "展示数据层架构的图表")

每个仓库都有自己的模型。例如，`TopicsRepository`有一个`Topic`模型，而`NewsRepository`有一个`NewsResource`模型。

仓库是其他层的公共API，它们提供_唯一_访问应用数据的方式。仓库通常提供一个或多个用于读取和写入数据的方法。

### 读取数据

数据作为数据流公开。这意味着仓库的每个客户端都必须准备对数据变化做出反应。数据不会作为快照（例如`getModel`）公开，因为在使用时无法保证它仍然有效。

读取操作从本地存储执行，作为真实来源，因此从`Repository`实例读取时不会出现错误。但是，在尝试将本地存储中的数据与远程源进行协调时可能会出现错误。有关错误协调的更多信息，请查看下面的数据同步部分。

_示例：读取主题列表_

可以通过订阅发出`List<Topic>`的`TopicsRepository::getTopics`流获取Topic列表。

每当主题列表发生变化（例如，添加新主题时），更新后的`List<Topic>`会被发送到流中。

### 写入数据

要写入数据，仓库提供挂起函数。由调用者确保其执行被适当地范围化。

_示例：关注主题_

只需使用用户希望关注的主题的ID和`followed=true`调用`UserDataRepository.toggleFollowedTopicId`来表示应该关注该主题（使用`false`取消关注主题）。

### 数据源

一个仓库可能依赖一个或多个数据源。例如，`OfflineFirstTopicsRepository`依赖以下数据源：

<table>
  <tr>
   <td><strong>名称</strong>
   </td>
   <td><strong>后端支持</strong>
   </td>
   <td><strong>目的</strong>
   </td>
  </tr>
  <tr>
   <td>TopicsDao
   </td>
   <td><a href="https://developer.android.com/training/data-storage/room">Room/SQLite</a>
   </td>
   <td>与主题相关的持久关系数据
   </td>
  </tr>
  <tr>
   <td>NiaPreferencesDataSource
   </td>
   <td><a href="https://developer.android.com/topic/libraries/architecture/datastore">Proto DataStore</a>
   </td>
   <td>与用户偏好相关的持久非结构化数据，特别是用户感兴趣的主题。这在.proto文件中使用protobuf语法定义和建模。
   </td>
  </tr>
  <tr>
   <td>NiaNetworkDataSource
   </td>
   <td>使用Retrofit访问的远程API
   </td>
   <td>主题数据，通过REST API端点以JSON形式提供。
   </td>
  </tr>
</table>

### 数据同步

仓库负责协调本地存储与远程源之间的数据。一旦从远程数据源获取数据，它会立即写入本地存储。更新后的数据从本地存储（Room）发送到相关的数据流，并被任何监听的客户端接收。

这种方法确保应用的读取和写入关注点是分开的，并且不会相互干扰。

在数据同步过程中出现错误的情况下，会采用指数退避策略。这通过`Synchronizer`接口的实现`SyncWorker`委托给`WorkManager`。

有关数据同步的示例，请参见`OfflineFirstNewsRepository.syncWith`。

## 领域层

[领域层](https://developer.android.com/topic/architecture/domain-layer)包含用例。这些是具有单个可调用方法（`operator fun invoke`）的类，包含业务逻辑。

这些用例用于简化并消除ViewModel中的重复逻辑。它们通常组合和转换来自仓库的数据。

例如，`GetUserNewsResourcesUseCase`将`NewsRepository`的`NewsResource`流（使用`Flow`实现）与`UserDataRepository`的`UserData`对象流组合在一起，创建`UserNewsResource`流。各种ViewModel使用此流在屏幕上显示带有书签状态的新闻资源。

值得注意的是，Now in Android中的领域层(_目前_)不包含任何用于事件处理的用例。事件由UI层直接调用仓库的方法处理。

## UI层

[UI层](https://developer.android.com/topic/architecture/ui-layer)包括：

* 使用[Jetpack Compose](https://developer.android.com/jetpack/compose)构建的UI元素
* [Android ViewModels](https://developer.android.com/topic/libraries/architecture/viewmodel)

ViewModel从用例和仓库接收数据流，并将其转换为UI状态。UI元素反映这种状态，并提供用户与应用交互的方式。这些交互作为事件传递给ViewModel进行处理。

![展示UI层架构的图表](images/architecture-4-ui-layer.png "展示UI层架构的图表")

### 建模UI状态

UI状态使用接口和不可变数据类作为密封层次结构建模。状态对象只通过数据流的转换发出。这种方法确保：

* UI状态始终代表底层应用数据 - 应用数据是真实来源。
* UI元素处理所有可能的状态。

**示例："为你推荐"屏幕上的新闻Feed**

"为你推荐"屏幕上的新闻资源Feed（列表）使用`NewsFeedUiState`建模。这是一个密封接口，创建了两种可能状态的层次结构：

* `Loading`表示数据正在加载
* `Success`表示数据已成功加载。Success状态包含新闻资源列表。

`feedState`传递给`ForYouScreen`可组合函数，后者处理这两种状态。

### 将流转换为UI状态

ViewModel从一个或多个用例或仓库接收数据流作为冷[flows](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-flow/index.html)。这些流被[组合](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/combine.html)在一起，或简单地[映射](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/map.html)，以产生单个UI状态流。然后使用[stateIn](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/state-in.html)将这个单一流转换为热流。转换为状态流使UI元素能够从流中读取最后已知的状态。

**示例：显示已关注的主题**

`InterestsViewModel`将`uiState`公开为`StateFlow<InterestsUiState>`。这个热流是通过获取由`GetFollowableTopicsUseCase`提供的`List<FollowableTopic>`的冷流创建的。每次发出新列表时，它都会转换为暴露给UI的`InterestsUiState.Interests`状态。

### 处理用户交互

用户操作通过常规方法调用从UI元素传递给ViewModel。这些方法作为lambda表达式传递给UI元素。

**示例：关注主题**

`InterestsScreen`接受一个名为`followTopic`的lambda表达式，该表达式由`InterestsViewModel.followTopic`提供。每当用户点击要关注的主题时，都会调用此方法。然后ViewModel通过通知用户数据仓库来处理此操作。

## 进一步阅读

[应用架构指南](https://developer.android.com/topic/architecture)

[Jetpack Compose](https://developer.android.com/jetpack/compose)
