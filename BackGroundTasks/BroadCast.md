# BroadCast(广播)

[原文(英文)地址](https://developer.android.com/guide/components/broadcasts)

[TOC]

应用程序可以注册去接收特定的`broadcast`，当一个`broadcast`被发送，系统自动将该`broadcast`路由到已订阅接收该特定类型`broadcast`的应用程序。

一般来说，`broadcast`可以用作跨应用程序和正常用户流之外的消息传递系统。但是，您必须小心，不要滥用在后台响应`broadcast`和执行任务的机会，这可能导致系统运行变慢，如以下视频所述。

[视频地址](https://youtu.be/vBjTXKpaFj8)

## 关于系统`broadcast`

系统会在一些系统事件发生的时候发生`broadcast`，比如当系统在飞行模式和正常模式之间切换的时候。系统广播会发送给所有订阅了要接收对应事件的应用程序。

`broadcast Message`本身包装在一个Intent对象中，该对象的`action`属性标识发生的事件（例如`android.intent.action.AIRPLAN_MODE`）。`Intent`还可能包括捆绑到其`extra`字段中的附加信息。例如，飞行模式`Intent`包括一个布尔类型的`extra`值，用于指示飞行模式是否打开。

关于如何读取`Intent`以及如何从`Intent`中获取`action`字段值，请参阅[Intent和Intent Filter](./Intent-And-Intent-Filter/Intent和IntentFilters——概述.md).

关于完整的系统`broadcast actions`信息，请参阅`Android SDK`中的`BROADCAST_ACTION.TXT`文件。每一个`broadcast action`都有一个常量值与之对应，比如[ACTION_AIRPLANE_MODE_CHANGED](https://developer.android.com/reference/android/content/Intent.html#ACTION_AIRPLANE_MODE_CHANGED)对应`android.intent.action.AIRPLANE_MODE`， `BROADCAST_ACTION.TXT`文档中有每一个`broadcast action`对应的常量值的信息。

### 系统`broadcast`的更改

随着`Android`平台的发展，它会定期改变系统`broadcast`的行为方式。如果您的应用程序针对`Android 7.0`（`API`级别24）或更高版本，或者安装在运行`Android 7.0`或更高版本的设备上，请记住以下更改。

#### Android 9

从`Android 9`（`API`级别28）开始，[`NETWORK_STATE_CHANGED_ACTION`](https://developer.android.com/reference/android/net/wifi/WifiManager.html#NETWORK_STATE_CHANGED_ACTION)不再接收用户的位置或用户个人数据的信息。
此外，如果您的应用程序是安装`Android 9`或者更高版本的设备的上，从`WI-FI`发送的`broadcast`不再含有`SSIDs`、`BSSIDs`、连接信息、扫描结果等信息。想得到这些信息，应该调用[`getConnectionInfo()`](https://developer.android.com/reference/android/net/wifi/WifiManager.html#getConnectionInfo())方法。

#### Android 8.0

从`Android8.0（API 26）`开始，系统对在`manifest`中声明的接受者添加了额外限制。
如果你的应用程序的目标设备版本为`Android8.0`或更高，您无法使用`manifest`为大多数隐式`broadcast`（`broadcast`不专门针对您的应用）声明`receiver`。当用户主动使用您的应用时，您仍然可以使用`context`注册的接收器（[context-registered receiver](https://developer.android.com/guide/components/broadcasts#context-registered-recievers)）。

#### Android 7.0

`Android 7.0`（`API`级别24）及更高版本不发送以下系统广播：

- `ACTION_NEW_PICTURE`
- `ACTION_NEW_VIDEO`

此外，针对`Android 7.0`及更高版本的应用必须使用[registerReceiver(BroadcastReceiver, IntentFilter)](https://developer.android.com/reference/android/content/Context.html#registerReceiver(android.content.BroadcastReceiver, android.content.IntentFilter))注册[CONNECTIVITY_ACTION](https://developer.android.com/reference/android/net/ConnectivityManager.html#CONNECTIVITY_ACTION)广播。在`manifest`中声明`receiver`不起作用。

## 接收broadcast

应用程序可以通过两种方式接收广播：在`manifest`中声明`broadcast receiver（manifest-declared receivers）`和使用`context`注册`receiver（ context-registered receivers）`。

### 在manifest中声明receiver

如果你在你的`manifest`中声明`broadcast receiver`，系统会在`broadcast`被发送后启动你的应用程序(如果发送`broadcast`的时候你的应用处于未启动状态)。

> 注意：如果您的应用程序的`target`是`API`级别26或更高，则您无法使用`manifest`来声明隐式`broadcast`（不特定针对你`app`的`broadcast`）的`receiver`，除了一些免于该限制（[exempted from that restriction](https://developer.android.com/guide/components/broadcast-exceptions.html)）的隐式`broadcast`，在大多数情况下，您可以使用[scheduled jobs](https://developer.android.com/topic/performance/scheduling.html)。

为了在`manifest`中声明`broadcast receiver`，请按照以下步骤实现：

1：在`manifest`中声明<`receiver`\>属性：

```xml
<receiver android:name=".MyBroadcastReceiver"  android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED"/>
        <action android:name="android.intent.action.INPUT_METHOD_CHANGED" />
    </intent-filter>
</receiver>
```

`intent-filter`指定`receiver`订阅的`broadcast action`。

2：继承`BroadcastReceiver`并实现`onReceive（Context，Intent）`方法。以下示例中的`BroadCastReceiver`记录并显示`broadcast`的内容：

- kotlin

- ```kotlin
  private const val TAG = "MyBroadcastReceiver"
  
  class MyBroadcastReceiver : BroadcastReceiver() {
  
      override fun onReceive(context: Context, intent: Intent) {
          StringBuilder().apply {
              append("Action: ${intent.action}\n")
              append("URI: ${intent.toUri(Intent.URI_INTENT_SCHEME)}\n")
              toString().also { log ->
                  Log.d(TAG, log)
                  Toast.makeText(context, log, Toast.LENGTH_LONG).show()
              }
          }
      }
  }
  ```

- java

- ```java
  public class MyBroadcastReceiver extends BroadcastReceiver {
          private static final String TAG = "MyBroadcastReceiver";
          @Override
          public void onReceive(Context context, Intent intent) {
              StringBuilder sb = new StringBuilder();
              sb.append("Action: " + intent.getAction() + "\n");
              sb.append("URI: " + intent.toUri(Intent.URI_INTENT_SCHEME).toString() + "\n");
              String log = sb.toString();
              Log.d(TAG, log);
              Toast.makeText(context, log, Toast.LENGTH_LONG).show();
          }
      }
  ```

系统的`package manager`在安装应用程序时会注册`register`。然后，`register`成为应用程序的单独入口点（`separate entry point` ），这意味着如果应用程序当前未运行，系统可以启动应用程序并发送`broadcast`。

系统创建一个新的`BroadcastReceiver`组件对象来处理它接收的每个广播。此对象仅在调用`onReceive（Context，Intent）`期间有效。一旦您的代码从此方法返回，系统会认为该组件不再处于活动状态。

### Context注册register

通过`context`注册`receiver`，大致应该按照以下步骤：

1：创建一个[BroadcastReceiver](https://developer.android.com/reference/android/content/BroadcastReceiver.html)实例

- kotlin

- ```kotlin
  val br: BroadcastReceiver = MyBroadcastReceiver()
  ```

- java

- ```java
  BroadcastReceiver br = new MyBroadcastReceiver();
  ```

2：创建一个[IntentFilter](https://developer.android.com/reference/android/content/IntentFilter.html)并通过调用[registerReceiver(BroadcastReceiver, IntentFilter)](https://developer.android.com/reference/android/content/Context.html#registerReceiver(android.content.BroadcastReceiver, android.content.IntentFilter))注册`receiver`：

- kotlin

- ```kotlin
  val filter = IntentFilter(ConnectivityManager.CONNECTIVITY_ACTION).apply {
      addAction(Intent.ACTION_AIRPLANE_MODE_CHANGED)
  }
  registerReceiver(br, filter)
  ```

- java

- ```java
  IntentFilter filter = new IntentFilter(ConnectivityManager.CONNECTIVITY_ACTION);
      filter.addAction(Intent.ACTION_AIRPLANE_MODE_CHANGED);
      this.registerReceiver(br, filter);
  ```

> 注意：如果要注册一个local broadcast，请调用[LocalBroadcastManager.registerReceiver(BroadcastReceiver, IntentFilter)](https://developer.android.com/reference/androidx/localbroadcastmanager/content/LocalBroadcastManager.html#registerReceiver(android.content.BroadcastReceiver, android.content.IntentFilter))。

只要注册的`context`有效，`context`注册的`receiver`就会接收`broadcast`。例如，如果您在`Activity context`中注册，则只要`Activity`未被销毁，您就会收到`broadcast`。如果您在`application context`中注册，则只要应用程序正在运行，您就会收到`broadcast`。

3：要停止接收广播，请调用[unregisterReceiver(android.content.BroadcastReceiver)](https://developer.android.com/reference/android/content/Context.html#unregisterReceiver(android.content.BroadcastReceiver))。当您不再需要`Regis`或`context`不再有效时，请务必取消注册的`register`。

请注意注册和取消注册`register`的位置，例如，如果使用`Activity context`在`onCreate（Bundle）`中注册`register`，则应在`onDestroy（）`中取消注册，以防止`register`泄漏到`Activity context`之外。如果在`onResume（）`中注册`register`，则应在`onPause（）`中注销它以防止多次注册（如果您不希望在`Paused`时接收广播，这可以减少不必要的系统开销）。不要在`onSaveInstanceState（Bundle）`中取消注册，因为如果用户在历史堆栈（`history stack`）中向后移动（`moves back`），则不会调用此方法。

### <span id="effect-on-process-state">对进程生命状态的影响</span>

`BroadcastReceiver`的状态（无论是否正在运行）会影响其宿主进程的状态，从而影响该对应进程被系统杀死的可能性。例如，当进程执行`receiver`（即运行`onReceive（）`方法中的代码）时，它被认为是前台进程，除极端内存压力外，系统会维持其运行。

但是，一旦您的代码从`onReceive（）`返回，`BroadcastReceiver`就不再处于活动状态。`Register`的宿主进程会与系统中其他正在运行的应用进程一样重要。如果该进程仅承载`manifest`中声明的`register`（常见的比如用户从未或最近未与之交互过的应用程序），则在从`onReceive（）`返回时，系统将其进程视为低优先级进程并且可能杀死它以使资源可用于其他更重要的进程。

因此，您不应该在`broadcast receiver`中启动长时间运行在后台的线程。在`onReceive（）`返回结果之后，系统可以随时终止进程以回收内存，这样做会终止在该进程中运行的线程。要避免这种情况，您应该调用`goAsync（）`（如果您希望`broadcast`的后台线程拥有更多处理任务的时间）或使用`JobScheduler`在`receiver`中调度`JobService`，这样系统知道该进程正在继续执行有效工作。有关更多信息，请参阅[进程和应用程序的生命周期](../Activity/进程和应用程序的生命周期.md)。

以下代码段展示了一个`BroadcastReceiver`，它使用`goAsync（）`标记在`onReceive（）`完成后需要更多时间才能完成（如果`onReceive（）`中的任务耗时足够多，导致`UI`线程错过一个帧（`> 16ms`），使其更适合放在后台线程执行，这将特别有用）。

- kotlin

- ```kotlin
  private const val TAG = "MyBroadcastReceiver"
  
  class MyBroadcastReceiver : BroadcastReceiver() {
  
      override fun onReceive(context: Context, intent: Intent) {
          val pendingResult: PendingResult = goAsync()
          val asyncTask = Task(pendingResult, intent)
          asyncTask.execute()
      }
  
      private class Task(
              private val pendingResult: PendingResult,
              private val intent: Intent
      ) : AsyncTask<String, Int, String>() {
  
          override fun doInBackground(vararg params: String?): String {
              val sb = StringBuilder()
              sb.append("Action: ${intent.action}\n")
              sb.append("URI: ${intent.toUri(Intent.URI_INTENT_SCHEME)}\n")
              return toString().also { log ->
                  Log.d(TAG, log)
              }
          }
  
          override fun onPostExecute(result: String?) {
              super.onPostExecute(result)
              // Must call finish() so the BroadcastReceiver can be recycled.
              pendingResult.finish()
          }
      }
  }
  ```

- java

  ```java
  public class MyBroadcastReceiver extends BroadcastReceiver {
      private static final String TAG = "MyBroadcastReceiver";
  
      @Override
      public void onReceive(Context context, Intent intent) {
          final PendingResult pendingResult = goAsync();
          Task asyncTask = new Task(pendingResult, intent);
          asyncTask.execute();
      }
  
      private static class Task extends AsyncTask<String, Integer, String> {
  
          private final PendingResult pendingResult;
          private final Intent intent;
  
          private Task(PendingResult pendingResult, Intent intent) {
              this.pendingResult = pendingResult;
              this.intent = intent;
          }
  
          @Override
          protected String doInBackground(String... strings) {
              StringBuilder sb = new StringBuilder();
              sb.append("Action: " + intent.getAction() + "\n");
              sb.append("URI: " + intent.toUri(Intent.URI_INTENT_SCHEME).toString() + "\n");
              String log = sb.toString();
              Log.d(TAG, log);
              return log;
          }
  
          @Override
          protected void onPostExecute(String s) {
              super.onPostExecute(s);
              // Must call finish() so the BroadcastReceiver can be recycled.
              pendingResult.finish();
          }
      }
  }
  ```

## 发送Broadcast

`Android`为我们提供了3种发送`broadcast`的方法：

- [sendOrderedBroadcast(Intent, String)](https://developer.android.com/reference/android/content/Context.html#sendOrderedBroadcast(android.content.Intent, java.lang.String))方法一次向一个`receiver`发送广播。当每个`receiver`依次执行时，它可以将结果传播到下一个`receiver`，或者它可以中止广播，这样该广播就不会传递给其他`receiver`。可以使用`intent-filter`的`android：priority`属性来控制`receiver`的接收顺序，具有相同优先级的`receiver`将以任意顺序执行。
- [sendBroadcast(Intent)](https://developer.android.com/reference/android/content/Context.html#sendBroadcast(android.content.Intent))方法以未定义的顺序向所有`receiver`发送广播。这称为正常广播。通常情况下这种情况更有效，但意味着`receiver`无法从其他`receiver`中读取结果，`receiver`传播从广播接收的数据或者终止它。
- [LocalBroadcastManager.sendBroadcast](https://developer.android.com/reference/androidx/localbroadcastmanager/content/LocalBroadcastManager.html#sendBroadcast(android.content.Intent))方法将广播发送到与发送方位于同一应用程序中的receiver。如果您不需要跨应用程序发送广播，请使用本地广播。这种方式的实现效率更高（无需进程间通信），您无需担心由于其他应用程序接收或发送您的广播引发的任何安全问题。

以下代码演示了如何通过创建一个`Intent`以及使用[sendBroadcast(Intent)](https://developer.android.com/reference/android/content/Context.html#sendBroadcast(android.content.Intent))发送一个广播：

- kotlin

- ```kotlin
  Intent().also { intent ->
      intent.setAction("com.example.broadcast.MY_NOTIFICATION")
      intent.putExtra("data", "Notice me senpai!")
      sendBroadcast(intent)
  }
  ```

- java

- ```java
  Intent intent = new Intent();
  intent.setAction("com.example.broadcast.MY_NOTIFICATION");
  intent.putExtra("data","Notice me senpai!");
  sendBroadcast(intent);
  ```

`broadcast`发送的消息包含在`Intent`对象中，`intent`的`action`属性必须提供该`app`的`java`包名并唯一标识`broadcast event`。您可以使用`putExtra（String，Bundle）`将其他信息附加到`intent`。您还可以通过调用`intent`的[setPackage(String)](https://developer.android.com/reference/android/content/Intent.html#setPackage(java.lang.String))将广播限制为同一组织中的一组应用程序。

> 注意：尽管`Intent`可以被同时用于发送广播和启动`Activity`，但是这二者的`action`属性是完全无关的。广播接收器无法查看或捕获用于启动`Activity`的`Intent`。同样，您无法使用`broadcast` 的`Intent`找到或启动`Activity`。

## 使用权限（permissions）限制broadcast

`permissions`允许您将广播限制在具有特定权限的应用程序集中。您可以对广播的发送者或接收者施加限制。

### 发送时施加权限

当你调用[sendBroadcast(Intent, String)](https://developer.android.com/reference/android/content/Context.html#sendBroadcast(android.content.Intent, java.lang.String))或者[sendOrderedBroadcast(Intent, String, BroadcastReceiver, Handler, int, String, Bundle)](https://developer.android.com/reference/android/content/Context.html#sendOrderedBroadcast(android.content.Intent, java.lang.String, android.content.BroadcastReceiver, android.os.Handler, int, java.lang.String, android.os.Bundle))时，你可以指定权限参数。只有那些已经在其`manifest`中指定权限`tag`的`receiver`（并且如果当前权限是危险权限，则需要手动授予权限）可以接收广播。例如，以下代码发送广播：

- kotlin

- ```kotlin
  sendBroadcast(Intent("com.example.NOTIFY"), Manifest.permission.SEND_SMS)
  ```

- java

  ```java
  sendBroadcast(new Intent("com.example.NOTIFY"),
                Manifest.permission.SEND_SMS);
  ```

如果要接收该广播，则需要下面的权限：

```xml
<uses-permission android:name="android.permission.SEND_SMS"/>
```

您可以指定现有系统权限（如`SEND_SMS`）或使用`<permission>`元素自定义权限。有关一般权限和安全性的信息，请参阅[System Permissions](https://developer.android.com/guide/topics/security/permissions.html)。

> 注意：安装应用程序时会注册自定义权限，所以必须在使用自定义的权限之前安装定义自定义权限的应用程序。

### 接收时施加权限

如果在注册广播接收器时指定了权限参数（使用[registerReceiver(BroadcastReceiver, IntentFilter, String, Handler)](https://developer.android.com/reference/android/content/Context.html#registerReceiver(android.content.BroadcastReceiver, android.content.IntentFilter, java.lang.String, android.os.Handler))或`manifest`中的`<receiver>`标记），则只有在`manifest`中使用`<uses-permission>`请求权限（如果是危险权限，随后需要被手动授权）的`broadcast`才能向该`receiver`发送广播。

例如，假设您的接收程序的`manifest`中的`receiver`具有以下声明：

```xml
<receiver android:name=".MyBroadcastReceiver"
          android:permission="android.permission.SEND_SMS">
    <intent-filter>
        <action android:name="android.intent.action.AIRPLANE_MODE"/>
    </intent-filter>
</receiver>
```

或者接收程序中含有使用以下代码进行`context`注册的`receiver`：

- kotlin

- ```kotlin
  var filter = IntentFilter(Intent.ACTION_AIRPLANE_MODE_CHANGED)
  registerReceiver(receiver, filter, Manifest.permission.SEND_SMS, null )
  ```

- java

- ```java
  IntentFilter filter = new IntentFilter(Intent.ACTION_AIRPLANE_MODE_CHANGED);
  registerReceiver(receiver, filter, Manifest.permission.SEND_SMS, null );
  ```

这样，如果想向上面的`receiver`发送`broadcast`，则必须向下面这样请求权限：

```xml
<uses-permission android:name="android.permission.SEND_SMS"/>
```

## 安全注意事项及最佳做法

以下是发送和接收`broadcast`的一些安全注意事项和最佳做法：

- 如果您不需要向应用程序外部的组件发送广播，则使用支持库中提供的`LocalBroadcastManager`发送和接收本地广播。 `LocalBroadcastManager`效率更高（无需进程间通信），而且您可以不用考虑与其他应用程序接收或发送您的广播相关的任何安全问题。本地广播可以在您的应用中用作通用发布/订阅事件总线，而无需系统广播的任何开销。

- 如果很多应用在其`manifest`文件中注册接收相同的广播，则可能导致系统启动大量应用，从而对设备性能和用户体验产生重大影响。为避免这种情况，请优先使用`context`注册而不是`manifest`声明的方式，`Android`系统的有些广播会强制使用`context`注册的接收器。例如，`CONNECTIVITY_ACTION`广播仅传递给上下文注册的接收器。

- 不要使用隐式`Intent`广播敏感信息。因为任何注册接收广播的应用都可以读取该信息。有三种方法可以控制谁可以接收您的广播：
  - 您可以在发送广播时指定权限。
  - 在`Android 4.0`及更高版本中，您可以在发送广播时指定包含`setPackage（String）`的包。系统将广播限制为与包匹配的应用程序集。
  - 您可以使用`LocalBroadcastManager`发送本地广播。
- 当您注册接收器时，任何应用都可以向您的应用接收器发送潜在的恶意广播。有三种方法可以限制应用收到的广播：
  - 您可以在注册广播接收器时指定权限。
  - 对于`manifest`中声明的接收器，您可以在`manifest`中将`android：exported`属性设置为“`false`”，即`receiver`不接收来自应用程序之外的来源的广播。
  - 您可以将自己限制为仅接收使用`LocalBroadcastManager`发送的本地广播。
- 广播的`action`的命名空间是全局的。确保`action`名称和其他字符串都写在您拥有的命名空间中，否则您可能会无意中与其他应用程序发生冲突。

- 因为接收者的`onReceive（Context，Intent）`方法在主线程上运行，所以它应该执行并快速返回。如果您需要执行长时间运行的工作，请谨慎启动线程或启动后台`Service`，因为系统可以在`onReceive（）`返回后终止整个进程。有关更多信息，请参阅对[进程生命状态的影响](#effect-on-process-state)。如果要执行长时间运行的工作，我们建议：
  - 在接收者的`onReceive（）`方法中调用`goAsync（）`并将`BroadcastReceiver.PendingResult`传递给后台线程。这使得从`onReceive（）`返回后`broadcast`依然可以保持活动状态。但是，即使采用这种方法，系统也希望您能够非常快速地完成广播中的任务（10秒以内）。它允许您将工作移动到另一个线程，以避免主线程发生阻塞。
  - 使用`JobScheduler`调度任务。有关更多信息，请参阅[Intelligent Job Scheduling](https://developer.android.com/topic/performance/scheduling.html)。
- 不要在广播接收器中启动`Activity`，因为用户体验很不稳定，特别是如果有多个`receiver`的时候，请考虑使用[notification](https://developer.android.com/guide/topics/ui/notifiers/notifications.html)。