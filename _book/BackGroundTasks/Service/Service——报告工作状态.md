# Service——报告工作状态

[原文(英文)地址](https://developer.android.com/training/run-background-service/report-status)

本文介绍如何中在后台`Service`中向发送任务处理请求的组件(指`Activity`、`Fragment`等)发送任务的执行状态，这允许你在一个`Activity`的`UI`上展示当前任务处理的进度。我们推荐使用[`LocalBroadcastManager`](https://developer.android.com/reference/androidx/localbroadcastmanager/content/LocalBroadcastManager.html)作为发送任务执行状态的方式，它将`broadcast Intent`对象限制为只能在你的`app`中可见。

## 从JobIntentService中报告状态

从`JobIntentService`中发送任务的执行状态到其他组件，首先需要创建一个`Intent`对象，将任务状态信息添加到这个`Intent`对象的`data`中。作为一个可选的操作，你可以向这个`Intent`中添加`action`和`data URI`。

下一步，调用[LocalBroadcastManager.sendBroadcast()](https://developer.android.com/reference/androidx/localbroadcastmanager/content/LocalBroadcastManager.html#sendBroadcast(android.content.Intent))将这个Intent对象发送出去。浙江把这个Intent对象发送到你app中所有注册需要接收它的组件中。要获得一个[LocalBroadcastManager](https://developer.android.com/reference/androidx/localbroadcastmanager/content/LocalBroadcastManager.html)对象，你可以调用[getInstance()](https://developer.android.com/reference/androidx/localbroadcastmanager/content/LocalBroadcastManager.html#getInstance(android.content.Context))。

比如：

- kotlin

  ```kotlin
  ...
  // Defines a custom Intent action
  const val BROADCAST_ACTION = "com.example.android.threadsample.BROADCAST"
  ...
  // Defines the key for the status "extra" in an Intent
  const val EXTENDED_DATA_STATUS = "com.example.android.threadsample.STATUS"
  ...
  class RSSPullService : JobIntentService() {
      ...
      /*
       * Creates a new Intent containing a Uri object
       * BROADCAST_ACTION is a custom Intent action
       */
      val localIntent = Intent(BROADCAST_ACTION).apply {
          // Puts the status into the Intent
          putExtra(EXTENDED_DATA_STATUS, status)
      }
      // Broadcasts the Intent to receivers in this app.
      LocalBroadcastManager.getInstance(this).sendBroadcast(localIntent)
      ...
  }
  ```

- java

  ```java
  public final class Constants {
      ...
      // Defines a custom Intent action
      public static final String BROADCAST_ACTION =
          "com.example.android.threadsample.BROADCAST";
      ...
      // Defines the key for the status "extra" in an Intent
      public static final String EXTENDED_DATA_STATUS =
          "com.example.android.threadsample.STATUS";
      ...
  }
  public class RSSPullService extends JobIntentService {
  ...
      /*
       * Creates a new Intent containing a Uri object
       * BROADCAST_ACTION is a custom Intent action
       */
      Intent localIntent =
              new Intent(Constants.BROADCAST_ACTION)
              // Puts the status into the Intent
              .putExtra(Constants.EXTENDED_DATA_STATUS, status);
      // Broadcasts the Intent to receivers in this app.
      LocalBroadcastManager.getInstance(this).sendBroadcast(localIntent);
  ...
  }
  ```

下一步就是在向`RSSPullService`发送工作请求的组件中接收这个`Intent`对象。

## 接收从JobIntentService中发送出来的broadcast

为了接收broadcast Intent，请使用[BroadcastReceiver](https://developer.android.com/reference/android/content/BroadcastReceiver.html)的基类，在基类中实现[BroadcastReceiver.onReceive()](https://developer.android.com/reference/android/content/BroadcastReceiver.html#onReceive(android.content.Context, android.content.Intent))回调方法，[LocalBroadcastManager](https://developer.android.com/reference/androidx/localbroadcastmanager/content/LocalBroadcastManager.html)在收到`Intent.LocalBroadcastManager`时会调用该回调`BroadcastReceiver.onReceive（）`方法并将`Intent`传递给`BroadcastReceiver.onReceive（）`。

比如：

- kotlin

  ```java
  // Broadcast receiver for receiving status updates from the IntentService.
  private class DownloadStateReceiver : BroadcastReceiver() {
  
      override fun onReceive(context: Context, intent: Intent) {
          ...
          /*
           * Handle Intents here.
           */
          ...
      }
  }
  ```

- java

  ```java
  // Broadcast receiver for receiving status updates from the IntentService.
  private class DownloadStateReceiver extends BroadcastReceiver
  {
      // Called when the BroadcastReceiver gets an Intent it's registered to receive
      @Override
      public void onReceive(Context context, Intent intent) {
  ...
          /*
           * Handle Intents here.
           */
  ...
      }
  }
  ```

当你定义[BroadcastReceiver](https://developer.android.com/reference/android/content/BroadcastReceiver.html)的时候，你可以为其定义用来匹配特殊的`actions`、`categories`、`data`的过滤器，为了做到这一点，你需要创建`IntentFilter`，下面这个例子像你演示了如何定义过滤器：

- kotlin

  ```kotlin
  // Class that displays photos
  class DisplayActivity : FragmentActivity() {
      ...
      override fun onCreate(savedInstanceState: Bundle?) {
          ...
          super.onCreate(savedInstanceState)
          ...
          // The filter's action is BROADCAST_ACTION
          var statusIntentFilter = IntentFilter(BROADCAST_ACTION).apply {
              // Adds a data filter for the HTTP scheme
              addDataScheme("http")
          }
          ...
  ```

- java

  ```java
  // Class that displays photos
  public class DisplayActivity extends FragmentActivity {
      ...
      public void onCreate(Bundle stateBundle) {
          ...
          super.onCreate(stateBundle);
          ...
          // The filter's action is BROADCAST_ACTION
          IntentFilter statusIntentFilter = new IntentFilter(
                  Constants.BROADCAST_ACTION);
  
          // Adds a data filter for the HTTP scheme
          statusIntentFilter.addDataScheme("http");
          ...
  ```

为了向系统注册`BroadCastReceiver`和`IntentFilter`，你需要先得到一个`LocalBroadcatManager`实例，然后调用它的[registerReceiver()](https://developer.android.com/reference/androidx/localbroadcastmanager/content/LocalBroadcastManager.html#registerReceiver(android.content.BroadcastReceiver, android.content.IntentFilter))方法，下面这个例子向你演示了如何注册`BroadCastReceiver`和`IntentFilter`：

- kotlin

  ```kotlin
          // Instantiates a new DownloadStateReceiver
          val downloadStateReceiver = DownloadStateReceiver()
          // Registers the DownloadStateReceiver and its intent filters
          LocalBroadcastManager.getInstance(this)
                  .registerReceiver(downloadStateReceiver, statusIntentFilter)
          ...
  ```

- java

  ```java
          // Instantiates a new DownloadStateReceiver
          DownloadStateReceiver downloadStateReceiver =
                  new DownloadStateReceiver();
          // Registers the DownloadStateReceiver and its intent filters
          LocalBroadcastManager.getInstance(this).registerReceiver(
                  downloadStateReceiver,
                  statusIntentFilter);
          ...
  ```

单个`BroadcastReceiver`可以处理多种类型的`broadcast Intent`对象，每个对象都有自己的`action`。此功能允许您为每个`action`执行不同的代码，而无需为每个`action`定义单独的`BroadcastReceiver`。要为同一个`BroadcastReceiver`定义另一个`IntentFilter`，请创建`IntentFilter`并重复调用`registerReceiver（）`。比如：

- kotlin

  ```kotlin
          /*
           * Instantiates a new action filter.
           * No data filter is needed.
           */
          statusIntentFilter = IntentFilter(ACTION_ZOOM_IMAGE)
          // Registers the receiver with the new filter
          LocalBroadcastManager.getInstance(this)
                  .registerReceiver(downloadStateReceiver, statusIntentFilter)
  ```

- java

  ```java
          /*
           * Instantiates a new action filter.
           * No data filter is needed.
           */
          statusIntentFilter = new IntentFilter(Constants.ACTION_ZOOM_IMAGE);
          // Registers the receiver with the new filter
          LocalBroadcastManager.getInstance(this).registerReceiver(
                  downloadStateReceiver,
                  statusIntentFilter);
  ```

发送`broadcast Intent`不会`start`或者`resume`一个`Activity`。即使您的应用程序在后台，`Activity`的`BroadcastReceiver`也会接收并处理`Intent`对象，但不会强制使您的应用程序到达前台。如果您想要在应用程序不可见时通知用户有关在后台发生的事件，请使用[`Notification`](https://developer.android.com/reference/android/app/Notification.html)。永远不要启动`Activity`以响应传入的 `broadcast Intent`。