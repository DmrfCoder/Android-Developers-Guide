# Service——概述

[原文(英文)地址](https://developer.android.com/guide/components/services)

`Service` 是一个可以在后台执行长时间运行操作而不提供用户界面的应用组件。Service可由其他应用组件启动，而且即使用户切换到其他应用，Service仍将在后台继续运行。 此外，组件可以绑定（bind）到Service，以与之进行交互，甚至是执行进程间通信 (IPC)。 例如，Service可以处理网络事务、播放音乐，执行文件 I/O 或与Content Provider交互，而所有这一切均可在后台进行。

Service分为三种类型：

- Foreground(前台)

  前台服务主要做一些用户可察觉的操作。比如一个音频App在前台Service播放音乐。前台Service必须展示一个[Notification](https://developer.android.com/guide/topics/ui/notifiers/notifications.html)。即使用户的注意力不在App上，App中的前台Service仍然会继续运行。

- BackGround(后台)

  后台服务主要做一些用户不能直接察觉到的操作。比如，如果一个app使用一个Service去做压缩存储的操作，那么该Service通常是一个后台Service。

  > 注意：如果你的app target API在26及以上，当应用程序本身不在前台时，系统会对运行的后台服务施加限制（[restrictions on running background services](https://developer.android.com/about/versions/oreo/background.html)）。在大多数情况下，您的应用应该使用[scheduled job](https://developer.android.com/topic/performance/scheduling.html)而不是后台Service。

- Bound(绑定)

  当应用组件通过调用 `bindService()` 绑定到Service时，Service即处于“绑定”状态。绑定Service提供了一个Client-Service接口，允许组件与Service进行交互、发送请求、获取结果，甚至是利用进程间通信 (IPC) 跨进程执行这些操作。 Bound Service仅当与另一个应用组件绑定时才会运行。 多个组件可以同时绑定到该Service，但全部取消绑定后，该Service即会被销毁。

虽然本文档将started Service和bound Service分开讨论，但是您的Service可以同时以这两种方式运行，也就是说，它既可以是started Service（以无限期运行），也允许binding。问题只是在于您是否实现了一组回调方法：`onStartCommand()`（允许组件started Service）和 `onBind()`（允许binding Service）。

无论应用是处于启动状态还是绑定状态，抑或处于启动且绑定状态，任何应用组件均可像使用 Activity 那样通过调用 `Intent` 来使用Service（即使此Service来自另一应用）。 不过，您可以通过清单文件将Service声明为私有Service，并阻止其他应用访问。 [使用清单文件声明Service](#declaring-a-service-in-the-manifest)部分将对此做更详尽的阐述。

> **注意：**Service在其托管进程的主线程中运行，它既**不**创建自己的线程，也**不**在单独的进程中运行（除非另行指定）。 这意味着，如果Service需要执行任何 CPU 密集型工作( CPU-intensive work)或阻塞性操作（例如 MP3 播放或联网），则应在Service内创建新线程来完成这项工作。通过使用单独的线程，可以降低发生“应用无响应”(ANR) 错误的风险，而应用的主线程仍可继续专注于运行用户与 Activity 之间的交互。

## 在Service和Thread之间做出选择

Service是一种即使用户未与应用交互也可在后台运行的简单组件。 因此，您应仅在必要时才创建Service。

如果仅仅需要当用户正在关注你的app时启动主线程之外的线程执行工作，则应创建新线程而非Service。 例如，如果您只是想在 Activity 运行的同时播放一些音乐，则可在`onCreate()` 中创建子线程，在 `onStart()` 中启动子线程，然后在 `onStop()` 中停止子线程。您还可以考虑使用 `AsyncTask` 或 `HandlerThread`，而非传统的 `Thread` 类。如需了解有关线程的详细信息，请参阅[进程和线程](../BestPractices/Performance/进程和线程——概述.md)文档。

请记住，如果您确实要使用Service，默认情况下，它仍会在应用的主线程中运行，因此，如果Service执行的是密集型或阻塞性操作，则您仍应在Service内创建新线程。

### 基础知识

要创建Service，您必须创建 `Service` 的子类（或使用它的一个现有子类）。在实现中，您需要重写一些回调方法，以处理Service生命周期的某些关键点，如果合适，也提供一种机制以允许其他组件绑定到你的Service。 应重写的最重要的回调方法包括：

- `onStartCommand()`

  当另一个组件（如 Activity）通过调用 `startService()` 请求启动Service时，系统将调用此方法。一旦执行此方法，Service即会启动并可在后台无限期运行。 如果您实现此方法，则在Service工作完成后，需要由您通过调用 `stopSelf()` 或 `stopService()` 来停止Service，如果您只想提供binding方式，则无需实现此方法。

- `onBind()`

  当另一个组件想通过调用 `bindService()` 与Service绑定（例如执行 RPC）时，系统将调用此方法。在此方法的实现中，您必须通过返回 `IBinder` 来提供一个接口，该接口供客户端与Service进行通信。请务必实现此方法，但如果您并不希望允许binding，则应返回 null。

- `onCreate()`

  首次创建Service时，系统将调用此方法来执行一次性设置程序（在调用`onStartCommand()` 或 `onBind()` 之前）。如果Service已在运行，则不会调用此方法。

- `onDestroy()`

  当Service不再被使用且将被销毁时，系统将调用此方法。Service应该实现此方法来清理所有资源，如线程、注册的侦听器（listener）、接收器（receiver）等。 这是Service接收的最后一个调用。

如果组件通过调用 `startService()` 启动Service（这会导致对 `onStartCommand()` 的调用），则Service将一直运行，直到Service使用 `stopSelf()` 自行停止运行，或由其他组件通过调用 `stopService()` 停止它为止。

如果组件是通过调用 `bindService()` 来创建Service且未调用 `onStartCommand()`，则Service只会在该组件与其绑定时运行。一旦该Service与所有客户端之间的绑定全部取消，系统便会销毁它。

仅当内存过低且必须回收系统资源以供具有用户焦点的 Activity 使用时，Android 系统才会强制停止Service。如果将Service绑定到具有用户焦点的 Activity，则它不太可能会被kill；如果将Service声明为[在前台运行](#running-a-service-in-the-foeground)（稍后讨论），则它几乎很难会被kill。或者，如果Service已启动并要长时间运行，则系统会随着时间的推移降低Service在后台任务列表中的优先级排名，Service也将随之变得非常容易被终止；如果Service是started Service，则您必须将其设计为能够妥善处理系统对它的重启。 如果系统终止Service，那么一旦资源变得再次可用，系统便会重启Service（不过这还取决于从`onStartCommand()` 返回的值，本文稍后会对此加以讨论）。如需了解有关系统会在何时销毁Service的详细信息，请参阅[进程和线程](../BestPractices/Performance/进程和线程——概述.md)文档。

在下文中，您将了解如何创建各类Service以及如何在其他应用组件中使用Service。

### <span id="declaring-a-service-in-the-manifest">使用清单文件声明Service</span>

如同 Activity（以及其他组件）一样，您必须在应用的清单文件中声明你应用中的所有Service。

要声明Service，请添加<service\>元素作为<application\>元素的子元素。例如：

```xml
<manifest ... >
  ...
  <application ... >
      <service android:name=".ExampleService" />
      ...
  </application>
</manifest>
```

如需了解有关使用清单文件声明Service的详细信息，请参阅 [`<service>`](https://developer.android.com/guide/topics/manifest/service-element.html)元素参考文档。

您还可将其他属性包括在<service\>元素中，以定义一些特性，如启动Service及其运行所在进程所需的权限。[`android:name`](https://developer.android.com/guide/topics/manifest/service-element.html?hl=zh-cn#nm) 属性是唯一必需的属性，用于指定Service的类名。应用一旦发布，即不应更改此类名，如若不然，可能会存在因依赖显式 Intent 来启动或绑定Service而破坏代码的风险（详情请阅读博客文章[Things That Cannot Change](http://android-developers.blogspot.com/2011/06/things-that-cannot-change.html)）。

> 注意：为了保证你app的安全性，启动Service时应该尽量使用显式Intent的启动方式，并且不要给你的Service生命<intent-filter\>元素。使用隐式Intent启动Service具有一个安全隐患，因为您无法确定响应Intent的Service，并且用户无法查看启动的Service。从Android 5.0（API级别21）开始，如果使用隐式intent调用bindService（），系统将抛出异常。

你可以通过添加 [`android:exported`](https://developer.android.com/guide/topics/manifest/service-element.html?hl=zh-cn#exported) 属性并将其设置为 `"false"`，确保Service仅适用于您的应用。这可以有效阻止其他应用启动您的Service（即便其他应用使用显式 Intent 也无法启动你的Service）。

## 创建启动（started）Service

启动Service由另一个组件通过调用 `startService()` 启动，这会导致Service的`onStartCommand()` 方法被调用。

Service启动之后，其生命周期即独立于启动它的组件，并且可以在后台无限期地运行，即使启动Service的组件已被销毁也不受影响。 因此，Service应在工作执行完成后通过调用 `stopSelf()` 来自行停止运行，或者由另一个组件通过调用 `stopService()` 来停止它。

应用组件（如 Activity）可以通过调用 `startService()` 方法并传递 `Intent` 对象（指定Service并包含待使用Service的所有数据）来启动Service，Service通过 `onStartCommand()` 方法接收此 `Intent`。

例如，假设某 Activity 需要将一些数据保存到在线数据库中。该 Activity 可以启动一个协同Service，并通过向 `startService()` 传递一个 Intent，为该Service提供要保存的数据。Service通过 `onStartCommand()` 接收 Intent，连接到互联网并执行数据库事务。事务完成之后，Service自行停止运行并随即被销毁。

> **注意：**默认情况下，Service与声明Service的应用运行于同一进程，而且运行于该应用的主线程中。 因此，如果Service在执行密集型或阻塞性操作时用户正在与同一应用的 Activity 进行交互，则会降低 Activity 性能。 为了避免影响应用性能，您应在Service内启动新线程。

从传统上讲，您可以扩展两个类来创建启动Service：

- `Service`

  这是适用于所有Service的基类。扩展此类时，必须创建一个用于执行所有Service工作的新线程，因为默认情况下，Service将使用应用的主线程，这会降低应用正在运行的所有 Activity 的性能。

- `IntentService`

  这是 `Service` 的子类，它使用工作线程逐一（串行）处理所有start request。如果您不要求Service同时处理多个请求，这是最好的选择。 您只需实现 `onHandleIntent()` 方法即可，该方法会接收每个start request对应的 Intent，使您能够执行后台工作。

下文介绍如何使用其中任一个类来实现Service。

### <span id="extending-the-intentService-class">扩展 IntentService 类</span>

由于大多数启动Service都不必同时处理多个请求（实际上，这种多线程情况可能很危险），因此使用 `IntentService` 类实现Service也许是最好的选择。

`IntentService` 执行以下操作：

- 创建默认的工作线程，用于在应用的主线程外执行传递给 `onStartCommand()` 的所有 Intent，该工作线程独立于你应用程序的主线程。
- 创建一个工作队列，用于将 Intent 逐一传递给 `onHandleIntent()` 实现，这样您就永远不必担心多线程问题。
- 在处理完所有启动请求后停止Service，因此您永远不必调用 `stopSelf()`。
- 提供 `onBind()` 的默认实现（返回 null）。
- 提供 `onStartCommand()` 的默认实现，将 Intent 依次发送到工作队列和 `onHandleIntent()` 实现。

综上所述，您只需实现 `onHandleIntent()` 来完成客户端提供的工作即可。（不过，您还需要为Service提供小型构造函数（small constructor ）。）

以下是 `IntentService` 的实现示例：

- kotlin

- ```kotlin
  /**
   * A constructor is required, and must call the super [android.app.IntentService.IntentService]
   * constructor with a name for the worker thread.
   */
  class HelloIntentService : IntentService("HelloIntentService") {
  
      /**
       * The IntentService calls this method from the default worker thread with
       * the intent that started the service. When this method returns, IntentService
       * stops the service, as appropriate.
       */
      override fun onHandleIntent(intent: Intent?) {
          // Normally we would do some work here, like download a file.
          // For our sample, we just sleep for 5 seconds.
          try {
              Thread.sleep(5000)
          } catch (e: InterruptedException) {
              // Restore interrupt status.
              Thread.currentThread().interrupt()
          }
  
      }
  }
  ```

- java

- ```java
  public class HelloIntentService extends IntentService {
  
    /**
     * A constructor is required, and must call the super <code><a href="/reference/android/app/IntentService.html#IntentService(java.lang.String)">IntentService(String)</a></code>
     * constructor with a name for the worker thread.
     */
    public HelloIntentService() {
        super("HelloIntentService");
    }
  
    /**
     * The IntentService calls this method from the default worker thread with
     * the intent that started the service. When this method returns, IntentService
     * stops the service, as appropriate.
     */
    @Override
    protected void onHandleIntent(Intent intent) {
        // Normally we would do some work here, like download a file.
        // For our sample, we just sleep for 5 seconds.
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            // Restore interrupt status.
            Thread.currentThread().interrupt();
        }
    }
  }
  ```

您只需要一个构造函数和一个 `onHandleIntent()` 实现即可。

如果您决定还重写其他回调方法（如 `onCreate()`、`onStartCommand()` 或 `onDestroy()`），请确保调用超类实现，以便 `IntentService` 能够妥善处理工作线程的生命周期。

例如，`onStartCommand()` 必须返回默认实现（即，如何将 Intent 传递给 `onHandleIntent()`）：

- kotlin

- ```kotlin
  override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
      Toast.makeText(this, "service starting", Toast.LENGTH_SHORT).show()
      return super.onStartCommand(intent, flags, startId)
  }
  ```

- java

  ```java
  @Override
  public int onStartCommand(Intent intent, int flags, int startId) {
      Toast.makeText(this, "service starting", Toast.LENGTH_SHORT).show();
      return super.onStartCommand(intent,flags,startId);
  }
  ```

除 `onHandleIntent()` 之外，您无需从中调用超类对应方法的唯一方法就是 `onBind()`（仅当Service允许绑定时，才需要实现该方法）。

在下一节中，您将了解如何在extend `Service` 基类时实现同类Service，该方法需要实现更多代码，但如需同时处理多个启动请求，则更适合使用该基类。

### 扩展Service类

正如上一部分中所述，使用 `IntentService` 显著简化了启动Service的实现。但是，若要求Service执行多线程（而不是通过工作队列处理启动请求），则可extend `Service` 类来处理每个 Intent。

为了便于比较，以下提供了 `Service` 类实现的代码示例，该类执行的工作与上述使用 `IntentService` 的示例完全相同。也就是说，对于每个启动请求，它均使用工作线程执行作业，且每次仅处理一个请求。

- kotlin

- ```kotlin
  class HelloService : Service() {
  
      private var serviceLooper: Looper? = null
      private var serviceHandler: ServiceHandler? = null
  
      // Handler that receives messages from the thread
      private inner class ServiceHandler(looper: Looper) : Handler(looper) {
  
          override fun handleMessage(msg: Message) {
              // Normally we would do some work here, like download a file.
              // For our sample, we just sleep for 5 seconds.
              try {
                  Thread.sleep(5000)
              } catch (e: InterruptedException) {
                  // Restore interrupt status.
                  Thread.currentThread().interrupt()
              }
  
              // Stop the service using the startId, so that we don't stop
              // the service in the middle of handling another job
              stopSelf(msg.arg1)
          }
      }
  
      override fun onCreate() {
          // Start up the thread running the service.  Note that we create a
          // separate thread because the service normally runs in the process's
          // main thread, which we don't want to block.  We also make it
          // background priority so CPU-intensive work will not disrupt our UI.
          HandlerThread("ServiceStartArguments", Process.THREAD_PRIORITY_BACKGROUND).apply {
              start()
  
              // Get the HandlerThread's Looper and use it for our Handler
              serviceLooper = looper
              serviceHandler = ServiceHandler(looper)
          }
      }
  
      override fun onStartCommand(intent: Intent, flags: Int, startId: Int): Int {
          Toast.makeText(this, "service starting", Toast.LENGTH_SHORT).show()
  
          // For each start request, send a message to start a job and deliver the
          // start ID so we know which request we're stopping when we finish the job
          serviceHandler?.obtainMessage()?.also { msg ->
              msg.arg1 = startId
              serviceHandler?.sendMessage(msg)
          }
  
          // If we get killed, after returning from here, restart
          return START_STICKY
      }
  
      override fun onBind(intent: Intent): IBinder? {
          // We don't provide binding, so return null
          return null
      }
  
      override fun onDestroy() {
          Toast.makeText(this, "service done", Toast.LENGTH_SHORT).show()
      }
  }
  ```

- java

- ```java
  public class HelloService extends Service {
    private Looper serviceLooper;
    private ServiceHandler serviceHandler;
  
    // Handler that receives messages from the thread
    private final class ServiceHandler extends Handler {
        public ServiceHandler(Looper looper) {
            super(looper);
        }
        @Override
        public void handleMessage(Message msg) {
            // Normally we would do some work here, like download a file.
            // For our sample, we just sleep for 5 seconds.
            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                // Restore interrupt status.
                Thread.currentThread().interrupt();
            }
            // Stop the service using the startId, so that we don't stop
            // the service in the middle of handling another job
            stopSelf(msg.arg1);
        }
    }
  
    @Override
    public void onCreate() {
      // Start up the thread running the service. Note that we create a
      // separate thread because the service normally runs in the process's
      // main thread, which we don't want to block. We also make it
      // background priority so CPU-intensive work doesn't disrupt our UI.
      HandlerThread thread = new HandlerThread("ServiceStartArguments",
              Process.THREAD_PRIORITY_BACKGROUND);
      thread.start();
  
      // Get the HandlerThread's Looper and use it for our Handler
      serviceLooper = thread.getLooper();
      serviceHandler = new ServiceHandler(serviceLooper);
    }
  
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Toast.makeText(this, "service starting", Toast.LENGTH_SHORT).show();
  
        // For each start request, send a message to start a job and deliver the
        // start ID so we know which request we're stopping when we finish the job
        Message msg = serviceHandler.obtainMessage();
        msg.arg1 = startId;
        serviceHandler.sendMessage(msg);
  
        // If we get killed, after returning from here, restart
        return START_STICKY;
    }
  
    @Override
    public IBinder onBind(Intent intent) {
        // We don't provide binding, so return null
        return null;
    }
  
    @Override
    public void onDestroy() {
      Toast.makeText(this, "service done", Toast.LENGTH_SHORT).show();
    }
  }
  ```

正如您所见，与使用 `IntentService` 相比，这种方式完成更多工作。

但是，因为是由您自己处理对 `onStartCommand()` 的每个调用，因此可以同时执行多个请求。此示例并未这样做，但如果您希望如此，则可为每个请求创建一个新线程，然后立即运行这些线程（而不是等待上一个请求完成）。

请注意，`onStartCommand()` 方法必须返回整型数。整型数是一个值，用于描述系统应该如何在Service终止的情况下继续运行Service（如上所述，`IntentService` 的默认实现将为您处理这种情况，不过您可以对其进行修改），从 `onStartCommand()` 返回的值必须是以下常量之一：

- `START_NOT_STICKY`

  如果系统在 `onStartCommand()` 返回后终止Service，则除非有pending Intent 要传递，否则系统*不会*重建Service。这是最安全的选项，可以避免在不必要时以及应用能够轻松重启所有未完成的作业时运行Service。

- `START_STICKY`

  如果系统在 `onStartCommand()` 返回后终止Service，则会重建Service并调用`onStartCommand()`，但*不会*重新传递最后一个 Intent。相反，除非有pending Intent 要启动Service（在这种情况下，将传递这些 Intent ），否则系统会通过空 Intent 调用`onStartCommand()`。这适用于不执行命令、但无限期运行并等待作业的媒体播放器（或类似Service）。

- `START_REDELIVER_INTENT`

  如果系统在 `onStartCommand()` 返回后终止Service，则会重建Service，并通过传递给Service的最后一个 Intent 调用 `onStartCommand()`。任何pending Intent 均依次传递。这适用于主动执行应该立即恢复的作业（例如下载文件）的Service。

有关这些返回值的更多详细信息，请查阅[MessagingService sample on GitHub](https://github.com/googlesamples/android-MessagingService)。

### 启动Service

您可以通过将 `Intent`（指定要启动的Service）传递给 `startService()`，从 Activity 或其他应用组件启动Service。Android 系统调用Service的 `onStartCommand()` 方法，并向其传递 `Intent`。（切勿直接调用 `onStartCommand()`。）

> 注意：如果您的应用面向API级别26或更高级别，除非应用程序本身位于前台，否则系统会对应用程序使用或创建后台Service施加限制。如果应用程序需要创建前台Service，则应用程序应调用startForegroundService（），该方法创建后台Service，但该方法向系统发出信号，表示该Service将自己提升到前台，创建Service后，服务必须在五秒钟内调用其startForeground（）方法。

例如，Activity 可以结合使用显式 Intent 与 `startService()`，启动上文中的示例Service (`HelloService`)：

- kotlin

- ```kotlin
  Intent(this, HelloService::class.java).also { intent ->
      startService(intent)
  }
  ```

- java

  ```java
  Intent intent = new Intent(this, HelloService.class);
  startService(intent);
  ```

`startService()` 方法将立即返回，且 Android 系统调用Service的 `onStartCommand()` 方法。如果Service尚未运行，则系统会先调用 `onCreate()`，然后再调用 `onStartCommand()`。

如果Service没有提供binding，则使用 `startService()` 传递的 Intent 是应用组件与Service之间唯一的通信模式。但是，如果您希望Service返回结果，则启动Service的客户端可以为broadcast创建一个`PendingIntent` （使用 `getBroadcast()`），并通过将其传递给启动Service的 `Intent` 达到将其传递给Service的目的。然后，Service就可以使用广播传递结果。

多个Service启动请求会导致多次对Service的 `onStartCommand()` 进行相应的调用。但是，要停止Service，只需一个Service发起停止请求（使用 `stopSelf()` 或 `stopService()`）即可。

### 停止Service

started Service必须管理自己的生命周期。也就是说，除非系统必须回收内存资源，否则系统不会停止或销毁Service，而且Service在 `onStartCommand()` 返回后会继续运行。因此，Service必须通过调用`stopSelf()` 自行停止运行，或者由另一个组件通过调用 `stopService()` 来停止它。

一旦请求使用 `stopSelf()` 或 `stopService()` 停止Service，系统就会尽快销毁Service。

但是，如果Service同时处理多个 `onStartCommand()` 请求，则您不应在处理完一个启动请求之后停止Service，因为您可能又收到了新的启动请求（在第一个请求结束时停止Service会终止第二个请求）。为了避免这一问题，您可以使用 `stopSelf(int)` 确保Service的stop request基于最近的start request。也就说，在调用 `stopSelf(int)` 时，传递与stop请求的 ID 对应的start请求的 ID（传递给 `onStartCommand()` 的 `startId`）。然后，如果在您调用 `stopSelf(int)`之前Service收到了新的启动请求，ID 就不匹配，Service也就不会停止。

> **注意：**为了避免浪费系统资源和消耗电池电量，应用必须在工作完成之后停止其Service。 如有必要，其他组件可以通过调用 `stopService()` 来停止Service。即使为Service启用了绑定，一旦Service收到对 `onStartCommand()` 的调用，您始终须亲自停止Service。

如需了解有关Service生命周期的详细信息，请参阅下面有关[管理Service生命周期](#managing-the-lifecycle-of-a-service)的部分。

## 创建绑定(bound)Service

绑定Service允许应用组件通过调用 `bindService()` 与其绑定，以便创建长期连接,通常不允许组件通过调用 `startService()` 来*启动*这种类型的Service。

如需与 Activity 和其他应用组件中的Service进行交互，或者需要通过进程间通信 (IPC) 向其他应用公开某些应用功能，则应创建绑定Service。

要创建绑定Service，必须实现 `onBind()` 回调方法并返回 `IBinder`，返回的IBinder用于定义与Service通信的接口。然后，其他应用组件可以调用 `bindService()` 来接收该接口，并开始调用Service的方法。Service只用于与其绑定的应用组件，因此如果没有组件绑定到Service，则系统会销毁Service（您*不必*按通过 `onStartCommand()` 启动的Service那样来停止绑定Service）。

要创建绑定Service，首先必须定义指定客户端如何与Service通信的接口。 Service与客户端之间的这个接口必须是 `IBinder` 的实现，并且Service必须从 `onBind()` 回调方法返回它。一旦客户端收到`IBinder`，即可开始通过该接口与Service进行交互。

多个客户端可以同时绑定到Service。客户端完成与Service的交互后，会调用 `unbindService()` 取消绑定。一旦没有客户端绑定到该Service，系统就会销毁它。

有多种方法实现绑定Service，其实现比启动Service更为复杂，因此绑定Service将在有关[绑定Service](./Service——绑定（bound）Service.md)的单独文档中专门讨论。

## 向用户发送通知(notifications)

Service一旦运行起来，Service即可使用[Toast Notifications](https://developer.android.com/guide/topics/ui/notifiers/toasts.html)或[Status Bar Notifications](https://developer.android.com/guide/topics/ui/notifiers/notifications.html)来通知用户所发生的事件。

Toast 通知是指出现在当前窗口的表面、片刻随即消失不见的消息，而状态栏通知则在状态栏中随消息一起提供图标，用户可以选择该图标来采取操作（例如启动 Activity）。

通常，当某些后台工作已经完成（例如文件下载完成）且用户现在可以对其进行操作时，状态栏通知是最佳方法。 当用户从展开视图中选定通知时，通知即可启动 Activity（例如查看已下载的文件）。

如需了解详细信息，请参阅[Toast Notifications](https://developer.android.com/guide/topics/ui/notifiers/toasts.html)或[Status Bar Notifications](https://developer.android.com/guide/topics/ui/notifiers/notifications.html)开发者指南。

## <span id="running-a-service-in-the-foeground">在前台运行Service</span>

前台Service被认为是用户主观可以感知到的一种Service，因此即使内存不足，系统也不会考虑将其终止。 前台Service必须在状态栏提供通知，放在“正在进行（OnGoing）”标题下方，这意味着除非Service停止或从前台移除，否则不能清除通知。

> 警告：请限制您的应用程序使用前台服务。
>
> 当应用程序需要执行用户可以注意到的任务时，即使用户没有直接与应用程序交互，您也应该只使用前台服务。因此，前台服务必须显示优先级为PRIORITY_LOW或更高的状态栏通知，这有助于确保用户了解您的应用正在执行的操作。如果操作的重要性足够低，您希望使用最低优先级通知，那么您可能不应该使用服务而是考虑使用[scheduled job](https://developer.android.com/topic/performance/scheduling.html)。
>
> 每个运行服务的应用程序都会给系统带来额外的负担，从而消耗系统资源。如果应用尝试使用低优先级通知隐藏其服务，则可能会影响用户正在与之交互的应用的性能。因此，如果某个应用尝试运行具有最低优先级通知的服务，系统会在通知抽屉的底部调出应用的行为。

例如，应该将通过Service播放音乐的音乐播放器设置为在前台运行，这是因为用户明确意识到其操作。 状态栏中的通知可能表示正在播放的歌曲，并允许用户启动 Activity 来与音乐播放器进行交互。同样，让用户跟踪其行走路程的应用程序需要前台服务来跟踪用户的位置。

> 注意：针对Android 9（API级别28）或更高级别并使用前台服务的应用程序必须请求FOREGROUND_SERVICE权限。这是常规权限，因此系统会自动将其授予请求的应用程序。
>
> 如果针对API级别28或更高级别的应用尝试在不请求FOREGROUND_SERVICE的情况下创建前台服务，则系统会抛出SecurityException。

要请求让Service运行于前台，请调用 `startForeground()`。此方法采用两个参数：唯一标识通知的整型数和状态栏的 `Notification`，Notification必须拥有PRIORITY_LOW或者更高的优先级，例如：

- kotlin

- ```kotlin
  val pendingIntent: PendingIntent =
          Intent(this, ExampleActivity::class.java).let { notificationIntent ->
              PendingIntent.getActivity(this, 0, notificationIntent, 0)
          }
  
  val notification: Notification = Notification.Builder(this, CHANNEL_DEFAULT_IMPORTANCE)
          .setContentTitle(getText(R.string.notification_title))
          .setContentText(getText(R.string.notification_message))
          .setSmallIcon(R.drawable.icon)
          .setContentIntent(pendingIntent)
          .setTicker(getText(R.string.ticker_text))
          .build()
  
  startForeground(ONGOING_NOTIFICATION_ID, notification)
  ```

- java

- ```java
  Intent notificationIntent = new Intent(this, ExampleActivity.class);
  PendingIntent pendingIntent =
          PendingIntent.getActivity(this, 0, notificationIntent, 0);
  
  Notification notification =
            new Notification.Builder(this, CHANNEL_DEFAULT_IMPORTANCE)
      .setContentTitle(getText(R.string.notification_title))
      .setContentText(getText(R.string.notification_message))
      .setSmallIcon(R.drawable.icon)
      .setContentIntent(pendingIntent)
      .setTicker(getText(R.string.ticker_text))
      .build();
  
  startForeground(ONGOING_NOTIFICATION_ID, notification);
  ```



> **注意：**提供给 `startForeground()` 的整型 ID 不得为 0。

要从前台移除Service，请调用 `stopForeground()`，此方法需要一个布尔值作为参数，指示是否也移除状态栏通知， 此方法*不会*停止Service， 但是，如果您在Service正在前台运行时将其停止，则通知也会被移除。

如需了解有关通知的详细信息，请参阅[创建状态栏通知](https://developer.android.com/guide/topics/ui/notifiers/notifications.html?hl=zh-cn)。

## <span id="managing-the-lifecycle-of-a-service">管理Service生命周期</span>

Service的生命周期比 Activity 的生命周期要简单得多。但是，密切关注如何创建和销毁Service反而更加重要，因为Service可以在用户没有意识到的情况下运行于后台。

Service生命周期（从创建到销毁）可以遵循两条不同的路径：

- started Service

  该Service在其他组件调用 `startService()` 时创建，然后无限期运行，且必须通过调用`stopSelf()` 来自行停止运行。此外，其他组件也可以通过调用 `stopService()` 来停止Service。Service停止后，系统会将其销毁。

- bound Service

  该Service在另一个组件（客户端）调用 `bindService()` 时创建。然后，客户端通过`IBinder` 接口与Service进行通信。客户端可以通过调用 `unbindService()` 关闭连接。多个客户端可以绑定到相同Service，而且当所有绑定全部取消后，系统即会销毁该Service。 （Service*不必*自行停止运行。）

这两条路径并非完全独立。也就是说，您可以绑定到已经使用 `startService()` 启动的Service。例如，可以通过使用 `Intent`（标识要播放的音乐）调用 `startService()` 来启动后台音乐Service。随后，可能在用户需要稍加控制播放器或获取有关当前播放歌曲的信息时，Activity 可以通过调用 `bindService()` 绑定到Service。在这种情况下，除非所有客户端均取消绑定，否则 `stopService()` 或 `stopSelf()` 不会实际停止Service。

### 实现生命周期回调

与 Activity 类似，Service也拥有生命周期回调方法，您可以实现这些方法来监控Service状态的变化并适时执行工作。 以下框架Service展示了每种生命周期方法：

- kotlin

  ```kotlin
  class ExampleService : Service() {
      private var startMode: Int = 0             // indicates how to behave if the service is killed
      private var binder: IBinder? = null        // interface for clients that bind
      private var allowRebind: Boolean = false   // indicates whether onRebind should be used
  
      override fun onCreate() {
          // The service is being created
      }
  
      override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
          // The service is starting, due to a call to startService()
          return mStartMode
      }
  
      override fun onBind(intent: Intent): IBinder? {
          // A client is binding to the service with bindService()
          return mBinder
      }
  
      override fun onUnbind(intent: Intent): Boolean {
          // All clients have unbound with unbindService()
          return mAllowRebind
      }
  
      override fun onRebind(intent: Intent) {
          // A client is binding to the service with bindService(),
          // after onUnbind() has already been called
      }
  
      override fun onDestroy() {
          // The service is no longer used and is being destroyed
      }
  }
  ```

- java

  ```java
  public class ExampleService extends Service {
      int startMode;       // indicates how to behave if the service is killed
      IBinder binder;      // interface for clients that bind
      boolean allowRebind; // indicates whether onRebind should be used
  
      @Override
      public void onCreate() {
          // The service is being created
      }
      @Override
      public int onStartCommand(Intent intent, int flags, int startId) {
          // The service is starting, due to a call to startService()
          return mStartMode;
      }
      @Override
      public IBinder onBind(Intent intent) {
          // A client is binding to the service with bindService()
          return mBinder;
      }
      @Override
      public boolean onUnbind(Intent intent) {
          // All clients have unbound with unbindService()
          return mAllowRebind;
      }
      @Override
      public void onRebind(Intent intent) {
          // A client is binding to the service with bindService(),
          // after onUnbind() has already been called
      }
      @Override
      public void onDestroy() {
          // The service is no longer used and is being destroyed
      }
  }
  ```



> **注：**与 Activity 生命周期回调方法不同，您*不*需要调用这些回调方法的超类实现。

![service_lifecycle](https://ws4.sinaimg.cn/large/006tNc79gy1g1zy8tc58zj30at0e3t9k.jpg)

<center>图 2. Service生命周期。左图显示了使用 `startService()` 所创建的Service的生命周期，右图显示了使用`bindService()` 所创建的Service的生命周期.</center>
图2说明了Service的典型回调方法。虽然该图将startService（）创建的服务与bindService（）创建的服务分开，但请记住，任何服务（无论它是如何启动的）都可能允许客户端绑定到它。最初使用onStartCommand（）（通过调用startService（）的客户端）启动的服务仍然可以接收对onBind（）的调用（当客户端调用bindService（）时）。

通过实现这些方法，您可以监控Service生命周期的两个嵌套循环：



- Service的**整个生命周期**从调用onCreate()开始起，到onDestroy()返回时结束。与 Activity 类似，Service也在onCreate()中完成初始设置，并在onDestroy()中释放所有剩余资源。例如，音乐播放Service可以在onCreate()中创建用于播放音乐的线程，然后在onDestroy()中停止该线程。

  无论Service是通过 `startService()` 还是 `bindService()` 创建，都会为所有Service调用 `onCreate()` 和 `onDestroy()` 方法。

- Service的**有效生命周期**从调用onStartCommand()或onBind()方法开始。每种方法均有Intent对象，该对象分别传递到startService()或bindService()。

  对于started Service，有效生命周期与整个生命周期同时结束（即便是在 `onStartCommand()`返回之后，Service仍然处于活动状态）。对于bound Service，有效生命周期在 `onUnbind()` 返回时结束。

> **注：**尽管启动Service是通过调用 `stopSelf()` 或 `stopService()` 来停止，但是该Service并无相应的回调（没有 `onStop()` 回调）。因此，除非Service绑定到客户端，否则在Service停止时，系统会将其销毁 — `onDestroy()` 是接收到的唯一回调。

如需了解有关创建提供绑定的Service的详细信息，请参阅[绑定Service](./Service——绑定（bound）Service.md)文档，该文档的**管理绑定Service的生命周期**部分提供了有关 `onRebind()` 回调方法的更多信息。