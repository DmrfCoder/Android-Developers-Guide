# Android架构组件——给你的项目中添加架构组件

在开始之前，我们建议您阅读[应用程序架构指南](../应用程序架构指南.md)，该指南包含一些适用于所有Android应用程序的有用原则，并展示了如何将架构组件一起使用。

架构组件可从Google的Maven存储库获得。要使用它们，必须将存储库添加到项目中。

打开项目的build.gradle文件（不是您的应用或模块的文件）并添加google（）存储库，如下所示：

```groovy
allprojects {
    repositories {
        google()
        jcenter()
    }
}
```

## 声明依赖

打开应用程序或模块的build.gradle文件，并添加所需的工件作为依赖项。您可以为所有体系结构组件添加依赖项，也可以选择子集。

请参阅发行说明中有关为每个Archiecture Component声明依赖关系的说明：

- [Futures (found in androidx.concurrent)](https://developer.android.com/jetpack/androidx/releases/concurrent)
- [Lifecycle components (including ViewModel)](https://developer.android.com/jetpack/androidx/releases/lifecycle)
- [Navigation (including SafeArgs)](https://developer.android.com/jetpack/androidx/releases/navigation)
- [Paging](https://developer.android.com/jetpack/androidx/releases/paging)
- [Room](https://developer.android.com/jetpack/androidx/releases/room)
- [WorkManager](https://developer.android.com/jetpack/androidx/releases/work)

有关AndroidX重构的更多信息，以及它如何影响这些类包和模块ID，请参阅AndroidX重构文档（[refactor documentation](https://developer.android.com/topic/libraries/support-library/refactor)）。

## Kotlin

几个AndroidX依赖项支持Kotlin扩展模块。这些模块的名称后面附加了后缀“-ktx”。例如：

```
implementation "androidx.lifecycle:lifecycle-viewmodel:$lifecycle_version"
```

变为：

```
implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:$lifecycle_version"
```

更多信息包括在Kotlin扩展的文档，可以在[ktx documentation](https://developer.android.com/kotlin/ktx)查看。

> 注意：对于基于Kotlin的应用程序，请确保使用kapt而不是annotationProcessor。你还应该添加kotlin-kapt插件。