# Now in Android 项目网络请求架构分析

## 项目使用的网络请求库和协议

Now in Android 项目主要使用以下网络请求库和协议：

1. **Retrofit**：作为主要的 HTTP 客户端库，用于处理网络请求
2. **OkHttp**：作为底层 HTTP 客户端，由 Retrofit 使用
3. **Kotlinx Serialization**：用于 JSON 序列化和反序列化
4. **HTTP/HTTPS 协议**：作为基础网络通信协议

## 网络配置类位置

核心网络配置类位于以下文件中：

1. **网络模块配置**：
   - `core/network/src/main/kotlin/com/google/samples/apps/nowinandroid/core/network/di/NetworkModule.kt`
   - 主要负责提供 JSON 序列化器、OkHttp 客户端和图片加载器

2. **Retrofit 网络实现**：
   - `core/network/src/main/kotlin/com/google/samples/apps/nowinandroid/core/network/retrofit/RetrofitNiaNetwork.kt`
   - 实现了 `NiaNetworkDataSource` 接口，使用 Retrofit 处理实际的网络请求

3. **网络数据源接口**：
   - `core/network/src/main/kotlin/com/google/samples/apps/nowinandroid/core/network/NiaNetworkDataSource.kt`
   - 定义了应用所需的网络操作接口

4. **环境特定实现**：
   - 生产环境：`core/network/src/prod/kotlin/com/google/samples/apps/nowinandroid/core/network/di/FlavoredNetworkModule.kt`
   - 演示环境：`core/network/src/demo/kotlin/com/google/samples/apps/nowinandroid/core/network/di/FlavoredNetworkModule.kt`

## 初始化代码

Retrofit 客户端初始化代码位于 `RetrofitNiaNetwork` 类中：

```kotlin
private val networkApi = trace("RetrofitNiaNetwork") {
    Retrofit.Builder()
        .baseUrl(NIA_BASE_URL)
        // 使用 dagger.Lazy<Call.Factory> 防止在主线程上初始化 OkHttp
        .callFactory { okhttpCallFactory.get().newCall(it) }
        .addConverterFactory(
            networkJson.asConverterFactory("application/json".toMediaType()),
        )
        .build()
        .create(RetrofitNiaNetworkApi::class.java)
}
```

OkHttp 客户端初始化代码位于 `NetworkModule` 类中：

```kotlin
@Provides
@Singleton
fun okHttpCallFactory(): Call.Factory = trace("NiaOkHttpClient") {
    OkHttpClient.Builder()
        .addInterceptor(
            HttpLoggingInterceptor()
                .apply {
                    if (BuildConfig.DEBUG) {
                        setLevel(HttpLoggingInterceptor.Level.BODY)
                    }
                },
        )
        .build()
}
```

## 拦截器设置

项目在 `NetworkModule` 类中配置了 OkHttp 的日志拦截器（HttpLoggingInterceptor）：

```kotlin
.addInterceptor(
    HttpLoggingInterceptor()
        .apply {
            if (BuildConfig.DEBUG) {
                setLevel(HttpLoggingInterceptor.Level.BODY)
            }
        },
)
```

该拦截器仅在调试模式下记录完整的网络请求和响应内容，包括请求体和响应体，有助于调试，同时避免在生产环境中记录敏感信息。 