# Android架构组件——概述

Android体系结构组件是一组库（libraries），可帮助您设计健壮，可测试和可维护的应用程序。从用于管理UI组件生命周期和处理数据持久性的类开始。

- 轻松管理应用程序的生命周期。[lifecycle-aware components](https://developer.android.com/topic/libraries/architecture/lifecycle)可帮助您管理Activity和Fragment生命周期。生存配置更改（Survive configuration changes,），避免内存泄漏并轻松将数据加载到UI中。
- 使用[LiveData](https://developer.android.com/topic/libraries/architecture/livedata)构建数据对象，以便在基础数据库更改时通知视图（Vires）。
- [ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel)存储在应用程序切换（rotations）中未销毁的UI相关的数据。
- [Room](https://developer.android.com/topic/libraries/architecture/room)是一个SQLite对象映射库。使用它来避免样板（boilerplate）代码并轻松地将SQLite表数据转换为Java对象。 Room提供SQLite语句的编译时检查，可以返回RxJava，Flowable和LiveData observable。

## 最新的视频和新闻

### 视频

[Improve your App's Architecture](https://www.youtube.com/watch?v=7p22cSzniBM)

### 博客

[7 Steps To Room](https://medium.com/google-developers/7-steps-to-room-27a5fe5f99b2)

[ViewModels and LiveData: Patterns + AntiPatterns](https://medium.com/google-developers/viewmodels-and-livedata-patterns-antipatterns-21efaef74a54)

[ViewModels : A Simple Example](https://medium.com/google-developers/viewmodels-a-simple-example-ed5ac416317e)

[ViewModels: Persistence, onSaveInstanceState(), Restoring UI State and Loaders](https://medium.com/google-developers/viewmodels-persistence-onsaveinstancestate-restoring-ui-state-and-loaders-fc7cc4a6c090)

[Understanding migrations with Room](https://medium.com/google-developers/understanding-migrations-with-room-f01e04b07929)

## 其他资源

想学习更多Android架构组件的内容，请考虑查看以下资源：

### Samples

- [Sunflower](https://github.com/googlesamples/android-sunflower)：一个园艺应用程序，演示Android Jetpack的Android开发最佳实践。
- [Android Architecture Components basic sample](https://github.com/googlesamples/android-architecture-components/tree/master/BasicSample)
- [(more...)](https://developer.android.com/topic/libraries/architecture/additional-resources.html#samples)

### Codelabs

- Android Room with a View [(Java)](https://codelabs.developers.google.com/codelabs/android-room-with-a-view) [(Kotlin)](https://codelabs.developers.google.com/codelabs/android-room-with-a-view-kotlin)
- [Android Data Binding codelab](https://codelabs.developers.google.com/codelabs/android-databinding)
- [(more...)](https://developer.android.com/topic/libraries/architecture/additional-resources.html#codelabs)

### Training

- [Udacity: Developing Android Apps](https://www.udacity.com/course/new-android-fundamentals--ud851)

### Blog posts

- [Announcing Architecture Components 1.0 Stable](https://android-developers.googleblog.com/2017/11/announcing-architecture-components-10.html)
- [Android and Architecture](https://android-developers.googleblog.com/2017/05/android-and-architecture.html)
- [(more...)](https://developer.android.com/topic/libraries/architecture/additional-resources.html#blogs)

### Videos

- [Android Jetpack: what's new in Architecture Components (Google I/O '18)](https://www.youtube.com/watch?v=pErTyQpA390)
- [Android Jetpack: easy background processing with WorkManager (Google I/O '18)](https://www.youtube.com/watch?v=IrKoBFLwTN0)
- [(more...)](https://developer.android.com/topic/libraries/architecture/additional-resources.html#videos)