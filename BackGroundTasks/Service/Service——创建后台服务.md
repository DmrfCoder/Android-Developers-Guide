# Service——创建后台服务

IntentService类提供了简单结构用于在单个后台线程运行操作。这使得它能够处理长时间运行的操作，而不会影响用户界面的响应能力。此外，IntentService不受大多数用户界面生命周期事件的影响，因此它会在关闭AsyncTask的情况下继续运行。

IntentService有以下几点限制：

- 他不能直接和你的用户界面通信，要将其执行结果显示在UI上，你必须先将结果发送到Activity。
- 工作任务需要串行执行，如果IntentService中已经有一个操作在执行了，这时你又发送了一个新的任务，它会在前一个任务执行完成之后才执行第二个任务。
- IntentService中运行的任务不能被中断。

虽然有上述限制，IntentService依然是简单后台任务的最佳实现方式。

本篇文章向你介绍如何创建你自己的IntentService子类，以及如何创建onHandleIntent()回调方法，最后像你介绍如何在manifest文件中定义IntentService。

## 处理传入的Intent

要为你的app创建IntentService组件，你应该创建一个继承自IntentService的子类，在该子类中你应该重写onHandleIntent()方法，就像下面这样：

- kotlin

- ```kotlin
  class RSSPullService : IntentService(RSSPullService::class.simpleName)
  
      override fun onHandleIntent(workIntent: Intent) {
          // Gets data from the incoming Intent
          val dataString = workIntent.dataString
          ...
          // Do work here, based on the contents of dataString
          ...
      }
  }
  ```

- java

- ```java
  public class RSSPullService extends IntentService {
      @Override
      protected void onHandleIntent(Intent workIntent) {
          // Gets data from the incoming Intent
          String dataString = workIntent.getDataString();
          ...
          // Do work here, based on the contents of dataString
          ...
      }
  }
  ```

注意，其他Service的常见回调方法，比如onStartCommand()会被IntentService自动调用，在IntentService类中，你应该避免重写此类回调方法。

要了解创建IntentService的详细内容，请参考[拓展IntentService类](./Service——概述.md#extending-the-intentService-class)

## 在manifest文件中定义IntentService

IntentService同样也需要在你应用的manifest文件中定义，你应该像Service那样为在<application\>下定义IntentService条目(entry)：

```xml
    <application
        android:icon="@drawable/icon"
        android:label="@string/app_name">
        ...
        <!--
            Because android:exported is set to "false",
            the service is only available to this app.
        -->
        <service
            android:name=".RSSPullService"
            android:exported="false"/>
        ...
    </application>
```

android：name属性唯一标识了该IntentService的类名。

注意<service\>元素不含有intent filter，发送工作给IntentService执行的Activity需要使用显示Intent，所以没必要定义Intent filter。这也意味着只有同一个app中的组件或者其他含有相同user ID的应用中的组件才有权限访问该Service。

现在你已经有了基本的IntentService类，你可以使用Intent对象向其发送工作请求。[下篇文章](./Service——向后台Service发送工作请求.md)将介绍如何构建这些Intent对象并将它们发送到IntentService。

