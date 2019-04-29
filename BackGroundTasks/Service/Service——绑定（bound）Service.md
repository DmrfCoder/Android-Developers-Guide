# Service——绑定（bound）Service

[原文(英文)地址](https://developer.android.com/guide/components/bound-services)

[TOC]

本文向您介绍如何创建`bound service`，包括如何绑定到来自其他应用组件的`service`。

有关一般`Service`的其他信息，例如如何从`Service`传递通知以及将`Service`设置为在前台运行，请参阅[Service](./Service——概述.md)文档。

## 基础知识

`bound service`是 `Service` 的实现类，允许其他应用与其绑定和交互。要提供`service`绑定，您必须实现 `onBind()` 回调方法。该方法返回的 `IBinder` 对象定义了客户端用来与`service`进行交互的接口。

## 绑定到已经启动的service

正如[Service](./Service——概述.md)文档中描述的那样，您可以创建同时具有`started`和`bound`两种状态的`service`。 也就是说，可通过调用 `startService()` 启动`service`，让`service`无限期运行，还可通过调用 `bindService()` 使客户端绑定到`service`。

如果您允许您的`service`同时具有`started`和`bound`状态，则`service`启动后，系统不会在所有客户端都取消绑定时自动销毁`service`。 所以，您必须通过调用 `stopSelf()` 或 `stopService()` 显式停止`service`。

尽管您通常只实现 `onBind()` *或* `onStartCommand()`中的一个，但有时需要同时实现这两个方法。例如，对于音乐播放器来说，让其`service`无限期运行的同时提供绑定很有用处。 这样一来，`Activity` 便可启动`service`进行音乐播放，即使用户离开应用，音乐播放也不会停止。 然后，当用户返回应用时，`Activity` 可绑定到`service`，重新获得回放的控制权。

要了解关于添加binding方式启动service时service的生命周期，请参阅[管理bound service的生命周期](#Managing-the-lifecycle-of-a-bound-Service)部分.

客户端可通过调用 `bindService()` 绑定到`service`。调用时，它必须提供 `ServiceConnection`的实现，它会监控与service的连接。`bindService()` 方法的返回值表示Service是否存在以及是否允许客户端访问他。当 Android 系统创建客户端与`service`之间的连接时，会对 `ServiceConnection` 调用 `onServiceConnected()`，`onServiceConnected（）`中包含了一个`IBinder`类型的参数，客户端使用该参数与`bound service`进行通信。

多个客户端可同时连接到一个service。不过，只有在第一个客户端绑定时，系统才会调用service的`onBind()` 方法来检索 `IBinder`，之后系统无需再次调用 `onBind()`，便可将同一 `IBinder`传递至任何其他绑定的客户端。

当最后一个客户端取消与service的绑定时，系统会将`service`销毁（除非该`service`同时又被通过 `startService()` 方法启动）。

当您实现`bound service`时，最重要的环节是定义您的 `onBind()` 回调方法返回的接口。您可以通过几种不同的方法定义`service`的 `IBinder` 接口，下文对这些方法逐一做了阐述。

## 创建bound service

创建提供绑定的`service`时，您必须给客户端一个用来与`service`进行交互的编程接口`IBinder`。 您可以通过三种方法定义`IBinder`接口：

- [继承 Binder 类](#extend-binder-class)

  如果`service`您的应用专用的（私有的），并且在与客户端相同的进程中运行（这是最常见的情况），则应通过继承 `Binder` 类并从 `onBind()` 中返回一个该实现类的实例以创建IBinder接口。客户端收到 `Binder` 对象后，可利用它直接访问 `Binder` 实现中乃至 `Service` 中可用的`public`方法。

  如果`service`只是您自己应用的后台工作线程，则优先采用这种方法。 不以这种方式创建接口的唯一原因是，您的`service`被其他应用或不同的进程占用。

- [使用 Messenger](#using-Messenger)

  如需让你的接口跨进程工作，则可使用 `Messenger` 为`service`创建接口。`service`可以通过这种方式定义对应于不同类型 `Message` 对象的 `Handler`。此 `Handler` 是 `Messenger` 的基础，`Messenger`随后可与客户端分享一个 `IBinder`，从而让客户端能利用 `Message` 对象向`service`发送命令。此外，客户端还可定义自有的 `Messenger`，以便`service`回传消息。这是执行进程间通信 (`IPC`) 的最简单方法，因为 `Messenger` 会在单一线程中创建包含所有请求的队列，这样您就不必特异为你的`service`进行线程安全设计。

- [使用 AIDL](./Service——AIDL.md)

  AIDL（Android 接口定义语言）执行所有将对象（objects）分解成原语（primiitives）的工作，操作系统可以识别这些原语并将它们编组（marshals）到各进程中，以执行 IPC。 之前采用 `Messenger` 的方法的底层实际上也是基于 AIDL的。 如上所述，`Messenger` 会在单一线程中创建包含所有客户端请求的队列，以便service一次接收一个请求。 不过，如果您想让service同时处理多个请求，则可直接使用 AIDL。 在此情况下，您的service必须具备多线程处理能力，并采用线程安全（thread-safe ）式设计。

  如需直接使用 AIDL，您必须创建一个定义编程接口的 `.aidl` 文件。Android SDK 工具利用该文件生成一个实现了定义的接口并处理 IPC 的抽象类，您随后可在service内对其进行继承（extend）。

> **注**：大多数应用都不应该使用 AIDL 来创建bound service，因为它可能要求具备多线程处理能力，并可能导致增加实现的复杂性。因此，AIDL 并不适合大多数应用，本文也不会阐述如何将其用于您的service。如果您确定自己需要直接使用 AIDL，请参阅 [AIDL](./Service——AIDL.md) 文档。

## <span id="extend-binder-class">继承Binder 类</span>

如果您的service仅供本地应用使用，不需要跨进程工作，则可以实现自己的 `Binder` 类，让您的客户端通过该类直接访问service中的public方法。

>  **注**：此方法只有在客户端和service位于同一应用和进程内这一情况下才有效。 例如，对于需要将 Activity 绑定到在后台播放音乐的service的音乐应用，此方法非常有效。

以下是具体的设置方法：

1. 在您的service中，创建一个可满足下列任一要求的Binder实例：
   - 包含客户端可调用的public方法
   - 返回当前 `Service` 实例，其含有客户端可调用的public方法
   - 返回由service承载（hosted）的其他类的实例，其中包含客户端可调用的public方法
2. 从 `onBind()` 回调方法返回此 `Binder` 实例。
3. 在客户端中，在 `onServiceConnected()` 回调方法中接收 `Binder`，并使用提供的方法调用bound service。

> **注**：之所以要求service和客户端必须在同一应用内，是为了便于客户端转换返回的对象和正确调用其 API。service和客户端还必须在同一进程内，因为此方法不执行任何跨进程编组（marshaling）。

例如，以下这个service可让客户端通过 `Binder` 实现访问service中的方法：

- kotlin

  ```kotlin
  class LocalService : Service() {
      // Binder given to clients
      private val binder = LocalBinder()
  
      // Random number generator
      private val mGenerator = Random()
  
      /** method for clients  */
      val randomNumber: Int
          get() = mGenerator.nextInt(100)
  
      /**
       * Class used for the client Binder.  Because we know this service always
       * runs in the same process as its clients, we don't need to deal with IPC.
       */
      inner class LocalBinder : Binder() {
          // Return this instance of LocalService so clients can call public methods
          fun getService(): LocalService = this@LocalService
      }
  
      override fun onBind(intent: Intent): IBinder {
          return binder
      }
  }
  ```

- java

  ```java
  public class LocalService extends Service {
      // Binder given to clients
      private final IBinder binder = new LocalBinder();
      // Random number generator
      private final Random mGenerator = new Random();
  
      /**
       * Class used for the client Binder.  Because we know this service always
       * runs in the same process as its clients, we don't need to deal with IPC.
       */
      public class LocalBinder extends Binder {
          LocalService getService() {
              // Return this instance of LocalService so clients can call public methods
              return LocalService.this;
          }
      }
  
      @Override
      public IBinder onBind(Intent intent) {
          return binder;
      }
  
      /** method for clients */
      public int getRandomNumber() {
        return mGenerator.nextInt(100);
      }
  }
  ```

`LocalBinder` 为客户端提供 `getService()` 方法，以检索 `LocalService` 的当前实例。这样，客户端便可调用service中的公共方法。 例如，客户端可调用service中的 `getRandomNumber()`。

点击按钮时，以下这个 Activity 会绑定到 `LocalService` 并调用 `getRandomNumber()` 方法：

- kotlin

  ```kotlin
  class BindingActivity : Activity() {
      private lateinit var mService: LocalService
      private var mBound: Boolean = false
  
      /** Defines callbacks for service binding, passed to bindService()  */
      private val connection = object : ServiceConnection {
  
          override fun onServiceConnected(className: ComponentName, service: IBinder) {
              // We've bound to LocalService, cast the IBinder and get LocalService instance
              val binder = service as LocalService.LocalBinder
              mService = binder.getService()
              mBound = true
          }
  
          override fun onServiceDisconnected(arg0: ComponentName) {
              mBound = false
          }
      }
  
      override fun onCreate(savedInstanceState: Bundle?) {
          super.onCreate(savedInstanceState)
          setContentView(R.layout.main)
      }
  
      override fun onStart() {
          super.onStart()
          // Bind to LocalService
          Intent(this, LocalService::class.java).also { intent ->
              bindService(intent, connection, Context.BIND_AUTO_CREATE)
          }
      }
  
      override fun onStop() {
          super.onStop()
          unbindService(connection)
          mBound = false
      }
  
      /** Called when a button is clicked (the button in the layout file attaches to
       * this method with the android:onClick attribute)  */
      fun onButtonClick(v: View) {
          if (mBound) {
              // Call a method from the LocalService.
              // However, if this call were something that might hang, then this request should
              // occur in a separate thread to avoid slowing down the activity performance.
              val num: Int = mService.randomNumber
              Toast.makeText(this, "number: $num", Toast.LENGTH_SHORT).show()
          }
      }
  }
  ```

- java

  ```java
  public class BindingActivity extends Activity {
      LocalService mService;
      boolean mBound = false;
  
      @Override
      protected void onCreate(Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          setContentView(R.layout.main);
      }
  
      @Override
      protected void onStart() {
          super.onStart();
          // Bind to LocalService
          Intent intent = new Intent(this, LocalService.class);
          bindService(intent, connection, Context.BIND_AUTO_CREATE);
      }
  
      @Override
      protected void onStop() {
          super.onStop();
          unbindService(connection);
          mBound = false;
      }
  
      /** Called when a button is clicked (the button in the layout file attaches to
        * this method with the android:onClick attribute) */
      public void onButtonClick(View v) {
          if (mBound) {
              // Call a method from the LocalService.
              // However, if this call were something that might hang, then this request should
              // occur in a separate thread to avoid slowing down the activity performance.
              int num = mService.getRandomNumber();
              Toast.makeText(this, "number: " + num, Toast.LENGTH_SHORT).show();
          }
      }
  
      /** Defines callbacks for service binding, passed to bindService() */
      private ServiceConnection connection = new ServiceConnection() {
  
          @Override
          public void onServiceConnected(ComponentName className,
                  IBinder service) {
              // We've bound to LocalService, cast the IBinder and get LocalService instance
              LocalBinder binder = (LocalBinder) service;
              mService = binder.getService();
              mBound = true;
          }
  
          @Override
          public void onServiceDisconnected(ComponentName arg0) {
              mBound = false;
          }
      };
  }
  ```

上例展示了客户端如何使用 `ServiceConnection` 的实现和 `onServiceConnected()` 回调绑定到service。下文会介绍更多关于绑定到service的过程。

> **注**：在上例中，`onStop()` 方法将客户端与service取消绑定。 客户端应在适当时机与service取消绑定，如[附加说明](#additional-notes)中所述。

如需查看更多示例代码，请参见 [ApiDemos](https://developer.android.com/resources/samples/ApiDemos/index.html?hl=zh-cn) 中的 [`LocalService.java`](https://developer.android.com/resources/samples/ApiDemos/src/com/example/android/apis/app/LocalService.html?hl=zh-cn) 类和 [`LocalServiceActivities.java`](https://developer.android.com/resources/samples/ApiDemos/src/com/example/android/apis/app/LocalServiceActivities.html?hl=zh-cn) 类。

## <span id="using-Messenger">使用 Messenger</span>

如需让service与远程进程通信，则可使用 `Messenger` 为您的service提供接口。利用此方法，您无需使用 AIDL 便可执行进程间通信 (IPC)。

对你的接口使用Messenger比使用AIDL更简单，因为 `Messenger` 会将所有service的调用排入队列，而纯粹的 AIDL 接口会同时向Service发送多个请求，Service必须对多线程进行处理。

对于大多数应用，Service不需要进行多线程处理，因此使用 `Messenger` 可让Service一次处理一个调用。如果您的服务必须执行多线程处理，则应使用 [AIDL](./Service——AIDL.md) 来定义接口。

以下是 `Messenger` 的使用方法摘要：

- service实现一个 `Handler`，由其接收来自客户端的每个调用的回调
- Service使用该`Handler` 创建一个 `Messenger` 对象（该对象是 `Handler` 的一个引用）
- `Messenger` 创建service通过 `onBind()` 使其返回客户端的 `IBinder`
- 客户端使用 `IBinder` 将 `Messenger`（引用service的 `Handler`）实例化，然后使用后者将 `Message` 对象发送给service
- service在其 `Handler` 中（具体地讲，是在 `handleMessage()` 方法中）接收每个 `Message`。

这样，客户端并没有调用service的方法，客户端传递的消息（`Message` 对象）是service在其 `Handler` 中接收的。

以下是一个使用 `Messenger` 接口的service示例：

- kotlin

- ```kotlin
  /** Command to the service to display a message  */
  private const val MSG_SAY_HELLO = 1
  
  class MessengerService : Service() {
  
      /**
       * Target we publish for clients to send messages to IncomingHandler.
       */
      private lateinit var mMessenger: Messenger
  
      /**
       * Handler of incoming messages from clients.
       */
      internal class IncomingHandler(
              context: Context,
              private val applicationContext: Context = context.applicationContext
      ) : Handler() {
          override fun handleMessage(msg: Message) {
              when (msg.what) {
                  MSG_SAY_HELLO ->
                      Toast.makeText(applicationContext, "hello!", Toast.LENGTH_SHORT).show()
                  else -> super.handleMessage(msg)
              }
          }
      }
  
      /**
       * When binding to the service, we return an interface to our messenger
       * for sending messages to the service.
       */
      override fun onBind(intent: Intent): IBinder? {
          Toast.makeText(applicationContext, "binding", Toast.LENGTH_SHORT).show()
          mMessenger = Messenger(IncomingHandler(this))
          return mMessenger.binder
      }
  }
  ```

- java

  ```java
  public class MessengerService extends Service {
      /**
       * Command to the service to display a message
       */
      static final int MSG_SAY_HELLO = 1;
  
      /**
       * Handler of incoming messages from clients.
       */
      static class IncomingHandler extends Handler {
          private Context applicationContext;
  
          IncomingHandler(Context context) {
              applicationContext = context.getApplicationContext();
          }
  
          @Override
          public void handleMessage(Message msg) {
              switch (msg.what) {
                  case MSG_SAY_HELLO:
                      Toast.makeText(applicationContext, "hello!", Toast.LENGTH_SHORT).show();
                      break;
                  default:
                      super.handleMessage(msg);
              }
          }
      }
  
      /**
       * Target we publish for clients to send messages to IncomingHandler.
       */
      Messenger mMessenger;
  
      /**
       * When binding to the service, we return an interface to our messenger
       * for sending messages to the service.
       */
      @Override
      public IBinder onBind(Intent intent) {
          Toast.makeText(getApplicationContext(), "binding", Toast.LENGTH_SHORT).show();
          mMessenger = new Messenger(new IncomingHandler(this));
          return mMessenger.getBinder();
      }
  }
  ```

请注意，service就是在 `Handler` 的 `handleMessage()` 方法中接收传入的 `Message`，并根据 `what` 属性决定下一步操作。

客户端只需根据service返回的 `IBinder` 创建一个 `Messenger`，然后利用 `send()` 发送一条消息。例如，以下就是一个绑定到service并向service传递 `MSG_SAY_HELLO` 消息的 Activity：

- kotlin

- ```kotlin
  class ActivityMessenger : Activity() {
      /** Messenger for communicating with the service.  */
      private var mService: Messenger? = null
  
      /** Flag indicating whether we have called bind on the service.  */
      private var bound: Boolean = false
  
      /**
       * Class for interacting with the main interface of the service.
       */
      private val mConnection = object : ServiceConnection {
  
          override fun onServiceConnected(className: ComponentName, service: IBinder) {
              // This is called when the connection with the service has been
              // established, giving us the object we can use to
              // interact with the service.  We are communicating with the
              // service using a Messenger, so here we get a client-side
              // representation of that from the raw IBinder object.
              mService = Messenger(service)
              bound = true
          }
  
          override fun onServiceDisconnected(className: ComponentName) {
              // This is called when the connection with the service has been
              // unexpectedly disconnected -- that is, its process crashed.
              mService = null
              bound = false
          }
      }
  
      fun sayHello(v: View) {
          if (!bound) return
          // Create and send a message to the service, using a supported 'what' value
          val msg: Message = Message.obtain(null, MSG_SAY_HELLO, 0, 0)
          try {
              mService?.send(msg)
          } catch (e: RemoteException) {
              e.printStackTrace()
          }
  
      }
  
      override fun onCreate(savedInstanceState: Bundle?) {
          super.onCreate(savedInstanceState)
          setContentView(R.layout.main)
      }
  
      override fun onStart() {
          super.onStart()
          // Bind to the service
          Intent(this, MessengerService::class.java).also { intent ->
              bindService(intent, mConnection, Context.BIND_AUTO_CREATE)
          }
      }
  
      override fun onStop() {
          super.onStop()
          // Unbind from the service
          if (bound) {
              unbindService(mConnection)
              bound = false
          }
      }
  }
  ```

- java

- ```java
  public class ActivityMessenger extends Activity {
      /** Messenger for communicating with the service. */
      Messenger mService = null;
  
      /** Flag indicating whether we have called bind on the service. */
      boolean bound;
  
      /**
       * Class for interacting with the main interface of the service.
       */
      private ServiceConnection mConnection = new ServiceConnection() {
          public void onServiceConnected(ComponentName className, IBinder service) {
              // This is called when the connection with the service has been
              // established, giving us the object we can use to
              // interact with the service.  We are communicating with the
              // service using a Messenger, so here we get a client-side
              // representation of that from the raw IBinder object.
              mService = new Messenger(service);
              bound = true;
          }
  
          public void onServiceDisconnected(ComponentName className) {
              // This is called when the connection with the service has been
              // unexpectedly disconnected -- that is, its process crashed.
              mService = null;
              bound = false;
          }
      };
  
      public void sayHello(View v) {
          if (!bound) return;
          // Create and send a message to the service, using a supported 'what' value
          Message msg = Message.obtain(null, MessengerService.MSG_SAY_HELLO, 0, 0);
          try {
              mService.send(msg);
          } catch (RemoteException e) {
              e.printStackTrace();
          }
      }
  
      @Override
      protected void onCreate(Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          setContentView(R.layout.main);
      }
  
      @Override
      protected void onStart() {
          super.onStart();
          // Bind to the service
          bindService(new Intent(this, MessengerService.class), mConnection,
              Context.BIND_AUTO_CREATE);
      }
  
      @Override
      protected void onStop() {
          super.onStop();
          // Unbind from the service
          if (bound) {
              unbindService(mConnection);
              bound = false;
          }
      }
  }
  ```

请注意，此示例并未说明service如何对客户端作出响应。如果您想让service作出响应，则还需要在客户端中创建一个 `Messenger`。然后，当客户端收到 `onServiceConnected()` 回调时，会向service发送一条 `Message`，并在其 `send()` 方法的 `replyTo` 参数中包含客户端的 `Messenger`。

如需查看如何提供双向消息传递的示例，请参阅 [`MessengerService.java`](https://developer.android.com/resources/samples/ApiDemos/src/com/example/android/apis/app/MessengerService.html?hl=zh-cn)（service）和 [`MessengerServiceActivities.java`](https://developer.android.com/resources/samples/ApiDemos/src/com/example/android/apis/app/MessengerServiceActivities.html?hl=zh-cn)（客户端）示例。

## 绑定到service

应用组件（客户端）可通过调用 `bindService()` 绑定到service，Android 系统紧接着会调用service的 `onBind()` 方法，该方法返回用于与service交互的 `IBinder`。

绑定是异步的，`bindService()` 会立即返回（不会将 `IBinder` 直接返回客户端）。要接收 `IBinder`，客户端必须创建一个 `ServiceConnection` 实例，并将其传递给 `bindService()`，`ServiceConnection` 包括一个回调方法，系统通过调用它来传递`IBinder`。

> **注**：只有 Activity、service和content providers可以绑定到service — 您**无法**从broadcast receiver绑定到service。

因此，要想从您的客户端绑定到service，您必须：

1. 实现ServiceConnection

   您的实现必须重写两个回调方法：

   - `onServiceConnected()`

     系统会调用该方法以传递service的　`onBind()` 方法返回的 `IBinder`。

   - `onServiceDisconnected()`

     Android 系统会在与service的连接意外中断时（例如当service崩溃或被终止时）调用该方法。当客户端取消绑定时，系统不会调用该方法。

2. 调用 `bindService()`，传递 `ServiceConnection` 实现。

   > 注意：如果该方法返回false，说明你的客户端将没有和service建立一个有效的连接。但是，你的客户端仍需调用unbindService()，否则，你的客户端将防止Service在空间时关闭。

3. 当系统调用您的 `onServiceConnected()` 回调方法时，您可以使用接口定义的方法开始调用service。

4. 要断开与service的连接，请调用unbindService()

   如果应用在客户端仍绑定到service时销毁客户端，则销毁会导致客户端取消绑定。 更好的做法是在客户端与service交互完成后立即取消绑定。 这样可以关闭空闲service。如需了解有关绑定和取消绑定的适当时机的详细信息，请参阅[附加说明](#additional-notes)。

例如，以下代码段通过[继承 Binder 类](#extend-binder-class)将客户端与上面创建的service相连，因此它只需将返回的 `IBinder` 转换为 `LocalService` 类并get  `LocalService` 实例：

- kotlin

  ```kotlin
  var mService: LocalService
  
  val mConnection = object : ServiceConnection {
      // Called when the connection with the service is established
      override fun onServiceConnected(className: ComponentName, service: IBinder) {
          // Because we have bound to an explicit
          // service that is running in our own process, we can
          // cast its IBinder to a concrete class and directly access it.
          val binder = service as LocalService.LocalBinder
          mService = binder.getService()
          mBound = true
      }
  
      // Called when the connection with the service disconnects unexpectedly
      override fun onServiceDisconnected(className: ComponentName) {
          Log.e(TAG, "onServiceDisconnected")
          mBound = false
      }
  }
  ```

- java

  ```java
  LocalService mService;
  private ServiceConnection mConnection = new ServiceConnection() {
      // Called when the connection with the service is established
      public void onServiceConnected(ComponentName className, IBinder service) {
          // Because we have bound to an explicit
          // service that is running in our own process, we can
          // cast its IBinder to a concrete class and directly access it.
          LocalBinder binder = (LocalBinder) service;
          mService = binder.getService();
          mBound = true;
      }
  
      // Called when the connection with the service disconnects unexpectedly
      public void onServiceDisconnected(ComponentName className) {
          Log.e(TAG, "onServiceDisconnected");
          mBound = false;
      }
  };
  ```

客户端可通过将此 `ServiceConnection` 传递至 `bindService()` 进而绑定到service。例如：

- kotlin

- ```kotlin
  Intent(this, LocalService::class.java).also { intent ->
      bindService(intent, connection, Context.BIND_AUTO_CREATE)
  }
  ```

- java

  ```java
  Intent intent = new Intent(this, LocalService.class);
  bindService(intent, connection, Context.BIND_AUTO_CREATE);
  ```
  - `bindService()` 的第一个参数是一个 `Intent`，用于显式命名要绑定的service（但 Intent 可能是隐式的）

    > 注意：

  - 第二个参数是 `ServiceConnection` 对象

  - 第三个参数是一个指示绑定选项的标志。它通常应该是 `BIND_AUTO_CREATE`，以便创建尚未激活的service。其他可能的值为 `BIND_DEBUG_UNBIND` 和 `BIND_NOT_FOREGROUND`，或 `0`（表示无）。

## <span id="additional-notes">附加说明</span>

以下是一些有关绑定到service的重要说明：

- 您应该始终捕获 `DeadObjectException` 异常，它们是在连接中断时引发的。这是远程方法引发的唯一异常。

- 对象是跨进程数量的引用。

- 您通常应该在客户端生命周期的引入 (bring-up) 和退出 (tear-down) 时刻执行对应的绑定和取消绑定操作。 例如：

  - 如果您只需要在 Activity 可见时与service交互，则应在 `onStart()` 期间绑定，在 `onStop()` 期间取消绑定。
  - 如果您希望 Activity 在后台停止运行状态下仍可接收响应，则可在 `onCreate()` 期间绑定，在 `onDestroy()` 期间取消绑定。请注意，这意味着您的 Activity 在其整个运行过程中（甚至包括后台运行期间）都需要使用service，因此如果service位于其他进程内，那么当您提高该进程的权重时，系统终止该进程的可能性会增加。

  > **注**：通常情况下，**切勿**在 Activity 的 `onResume()` 和 `onPause()` 期间绑定和取消绑定，因为每一次生命周期转换都会发生这些回调，您应该使发生在这些转换期间的c操作数量保持在最少水平。此外，如果您的应用内的多个 Activity 绑定到同一service，并且其中两个 Activity 之间发生了转换，则如果当前 Activity 在下一个 Activity 绑定（恢复期间）之前取消绑定（暂停期间），系统可能会销毁service并重建service。 （[Activity](../Activity)文档中介绍了这种有关 Activity 如何协调其生命周期的 Activity 转换。）

如需查看更多显示如何绑定到service的示例代码，请参阅 [ApiDemos](https://developer.android.com/resources/samples/ApiDemos/index.html?hl=zh-cn) 中的 [`RemoteService.java`](https://developer.android.com/resources/samples/ApiDemos/src/com/example/android/apis/app/RemoteService.html?hl=zh-cn)类。

## <span id="Managing-the-lifecycle-of-a-bound-Service">管理bound service的生命周期</span>

当service与所有客户端之间的绑定全部取消时，Android 系统便会销毁service（除非解绑的同时又使用 `onStartCommand()` 启动了该service）。因此，如果您的service是纯粹的bound service，则无需对其生命周期进行管理 — Android 系统会根据它是否绑定到客户端代您管理。

不过，如果您实现了 `onStartCommand()` 回调方法，则您必须显式停止service，因为系统现在已将service视为*已启动*。在此情况下，service将一直运行，直到其通过 `stopSelf()` 自行停止，或其他组件调用 `stopService()` 为止（无论其是否绑定到任何客户端）。

此外，如果您的service已启动并接受绑定，则当系统调用您的 `onUnbind()` 方法时，如果您想在客户端下一次绑定到service时接收 `onRebind()` 调用，则可选择在onUnbind（）中返回 `true`。如果`onRebind()` 返回空值，客户端仍会在其 `onServiceConnected()` 回调中接收 `IBinder`。图 1 说明了这种生命周期的逻辑。



![1554033596512](https://ws2.sinaimg.cn/large/006tNc79gy1g2h6qlh8vgj30em0frq3u.jpg)



<center>图 1. 已启动且允许绑定的service的生命周期

如需了解有关已启动service生命周期的详细信息，请参阅[service](./Service——概述.md)文档。

