# Service——向后台Service发送工作请求

[上一篇文章](./Service——创建后台服务.md)介绍了如何创建一个[`JobIntentService`](https://developer.android.com/reference/android/support/v4/app/JobIntentService)类.这篇文章将介绍如何通过一个`Intent`来触发`JobIntentService`去执行一个操作，这个`Intent`中也可以用于`JobintentService`执行任务（操作）的一些数据。

## 创建工作请求并将其发送到`JobIntentService`

为了创建一个工作请求并将其发送给`JobIntentService`，我们需要创建一个`Intent`并`JobIntentService`通过调用`enqueueWork()`将其加入工作队列。你还可以向`Intent`中添加一些数据以提供给`JobIntentService`去执行操作。您还可以选择将数据添加到`Intent`（以`Intent extras`的形式）以供`JobIntentService`进行任务处理。有关创建Intent的更多信息，请阅读[Intent和IntentFilters——概述](../Intent-And-Intent-Filter/Intent和IntentFilters——概述.md)中的[构建Intent](../Intent-And-Intent-Filter/Intent和IntentFilters——概述.md#Building-an-intent)部分。

下面的代码段演示了这个过程：

1：为名为`RSSPullService`的`JobIntentService`类创建一个`Intent`：

- kotlin

  ```java
  /*
   * Creates a new Intent to start the RSSPullService
   * JobIntentService. Passes a URI in the
   * Intent's "data" field.
   */
  serviceIntent = Intent().apply {
      putExtra("download_url", dataUrl)
  }
  ```

- java

  ```java
  /*
   * Creates a new Intent to start the RSSPullService
   * JobIntentService. Passes a URI in the
   * Intent's "data" field.
   */
  serviceIntent = new Intent();
  serviceIntent.putExtra("download_url", dataUrl));
  ```

2：调用`enqueueWork()`

- kotlin

  ```java
  private const val RSS_JOB_ID = 1000
  RSSPullService.enqueueWork(context, RSSPullService::class.java, RSS_JOB_ID, serviceIntent)
  ```

- java

  ```java
  // Starts the JobIntentService
  private static final int RSS_JOB_ID = 1000;
  RSSPullService.enqueueWork(getContext(), RSSPullService.class, RSS_JOB_ID, serviceIntent);
  ```

注意你可以在`Activity`或者`Fragment`的任何地方发送这个请求(`Request`)。比如，如果你需要先获取用户的输入，你可以在`button`点击的回调事件中发起这个请求(`Request`)。

当你调用了`enqueueWork()`之后，`JobIntentService`将会执行定义在它`onHandleWork()`方法中的任务，执行完成之后`JobIntentService`会自己停止自己（`stop itself`）。

下一步就是将任务执行的结果返回给源的`Activity`或者`Fragment`，[下个文档](./Service——报告工作状态.md)将会介绍如何使用`BroadCastReceiver`去实现这个目的。