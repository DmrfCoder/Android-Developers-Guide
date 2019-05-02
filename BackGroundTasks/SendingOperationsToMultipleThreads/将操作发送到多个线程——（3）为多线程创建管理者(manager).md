# 将操作发送到多个线程——为多线程创建管理者(manager)

[原文(英文)地址](https://developer.android.com/training/multiple-threads/create-threadpool)

上一篇文章介绍了如何[指定线程中运行的代码](./将操作发送到多个线程——指定线程中运行的代码.md)，它说明了如何指定在单个线程上执行的任务，如果您只想运行一次任务，这可能就是您所需要的。如果要在不同的数据集上重复运行任务，但一次只需要执行一次，IntentService就能满足您的需求。要在资源可用时自动运行任务，或允许多个任务同时运行，您需要提供托管的线程集合。为此，请使用`ThreadPoolExecutor`实例，该实例在其池中的线程空闲时从队列取出任务运行。要运行任务，您只需将其添加到队列中即可。

线程池可以运行任务的多个并行实例，因此您应该确保代码是线程安全的。将可能存在多个线程同时访问的变量包含在`synchronized`块中，这种方法将阻止一个线程写入变量的时候另一个线程读取变量的行为。通常，这种情况会出现在静态变量中，但它也会出现在只实例化一次的任何对象中。要了解有关此内容的更多信息，请阅读[进程和线程——概述](…/BestPractices/Performance/进程和线程——概述.md)。

## 定义线程池类

在自己的类中实例化[ThreadPoolExecutor](https://developer.android.com/reference/java/util/concurrent/ThreadPoolExecutor.html)：

### 对线程池使用静态变量

您的应用程序中可能只需要一个线程池实例，以便为受限制的CPU或网络资源提供单个控制点。如果您有不同的Runnable类型，您可能希望为每个类型创建一个线程池，但每个类型都可以是一个实例。例如，您可以将其添加为全局字段声明的一部分（对于Kotlin，我们可以创建一个object）：

- kotlin

- ```kotlin
  // Creates a single static instance of PhotoManager
  object PhotoManager {
      ...
  }
  ```

- java

- ```java
  public class PhotoManager {
      ...
      static  {
          ...
          // Creates a single static instance of PhotoManager
          sInstance = new PhotoManager();
      }
      ...
  ```

### 使用私有构造方法

使构造函数私有可确保它只有一个单例，这意味着您不必在`synchronized`块中执行对类的访问（对于Kotlin，没有必要使用私有构造函数，因为object只会被初始化一次）：

- kotlin

- ```kotlin
  object PhotoManager {
      ...
  }
  ```

- java

- ```java
  public class PhotoManager {
      ...
      /**
       * Constructs the work queues and thread pools used to download
       * and decode images. Because the constructor is marked private,
       * it's unavailable to other classes, even in the same package.
       */
      private PhotoManager() {
      ...
      }
  ```

### 通过调用线程池类中的方法启动你的任务

在线程池类中定义一个方法，该方法将任务添加到线程池的队列中。例如：

- kotlin

- ```kotlin
  object PhotoManager {
      ...
      fun startDownload(imageView: PhotoView, downloadTask: DownloadTask, cacheFlag: Boolean) =
              decodeThreadPool.execute(downloadTask.getHTTPDownloadRunnable())
      ...
  }
  ```

- java

- ```java
  public class PhotoManager {
      ...
      // Called by the PhotoView to get a photo
      static public PhotoTask startDownload(
          PhotoView imageView,
          DownloadTask downloadTask,
          boolean cacheFlag) {
          ...
          // Adds a download task to the thread pool for execution
          sInstance.
                  downloadThreadPool.
                  execute(downloadTask.getHTTPDownloadRunnable());
          ...
      }
  ```

### 在构造方法中初始化Handler并将其attach到你的Ui线程

Handler允许您的应用程序安全地调用UI对象（如View对象）的方法。大多数UI对象只能从UI线程安全地更改，在[与UI线程通信](./将操作发送到多个线程——与UI线程通信.md)中有更详细的描述。例如：

- kotlin

- ```kotlin
  object PhotoManager {
      ...
      private val handler = object : Handler(Looper.getMainLooper()) {
  
          /*
           * handleMessage() defines the operations to perform when
           * the Handler receives a new Message to process.
           */
          override fun handleMessage(msg: Message?) {
              ...
          }
          ...
      }
  }
  ```

- java

- ```java
      private PhotoManager() {
      ...
          // Defines a Handler object that's attached to the UI thread
          handler = new Handler(Looper.getMainLooper()) {
              /*
               * handleMessage() defines the operations to perform when
               * the Handler receives a new Message to process.
               */
              @Override
              public void handleMessage(Message inputMessage) {
                  ...
              }
          ...
          }
      }
  ```

## 确定线程池参数

一旦拥有了整体类结构，就可以开始定义线程池。要实例化ThreadPoolExecutor对象，需要以下值：

### 初始池size和最大池size

分配给池的初始线程数，以及线程池中允许的线程的最大数量。线程池中可以拥有的线程数主要取决于设备可用的核心数。此数字可从系统环境获得：

- kotlin

- ```kotlin
  object PhotoManager {
      ...
      /*
       * Gets the number of available cores
       * (not always the same as the maximum number of cores)
       */
      private val NUMBER_OF_CORES = Runtime.getRuntime().availableProcessors()
  }
  ```

- java

- ```java
  public class PhotoManager {
  ...
      /*
       * Gets the number of available cores
       * (not always the same as the maximum number of cores)
       */
      private static int NUMBER_OF_CORES =
              Runtime.getRuntime().availableProcessors();
  }
  ```

此数字可能无法反映设备中的物理核心数量，因为某些设备具有根据系统负载停用一个或多个内核的CPU。对于这些设备，availableProcessors（）返回活动的核心数，这可能少于核心总数。

### 线程保持存活的时间以及单位

线程在关闭之前保持空闲的持续时间。持续时间由时间单位值解释，时间单位值是[TimeUnit](https://developer.android.com/reference/java/util/concurrent/TimeUnit.html)中定义的常量之一。

### 一个任务队列

ThreadPoolExecutor从中获取Runnable对象的队列。要在线程上启动代码，线程池管理器从先进先出队列中获取Runnable对象并将其附加到线程。当你创建线程池时可以提供实现了`BlockingQueue`接口的任何队列类的对象。要满足你的应用程序的需求，您可以从可用的队列类实现中进行选择，要了解有关它们的更多信息，请参阅[ThreadPoolExecutor](https://developer.android.com/reference/java/util/concurrent/ThreadPoolExecutor.html)。此示例使用[LinkedBlockingQueue](https://developer.android.com/reference/java/util/concurrent/LinkedBlockingQueue.html)类：

- kotlin

- ```kotlin
  object PhotoManager {
      ...
      // Instantiates the queue of Runnables as a LinkedBlockingQueue
      private val decodeWorkQueue: BlockingQueue<Runnable> = LinkedBlockingQueue<Runnable>()
      ...
  }
  ```

- java

- ```java
  public class PhotoManager {
      ...
      private PhotoManager() {
          ...
          // A queue of Runnables
          private final BlockingQueue<Runnable> decodeWorkQueue;
          ...
          // Instantiates the queue of Runnables as a LinkedBlockingQueue
          decodeWorkQueue = new LinkedBlockingQueue<Runnable>();
          ...
      }
      ...
  }
  ```

## 创建一个线程池

要创建线程池，请通过调用ThreadPoolExecutor（）来实例化线程池管理器。这会创建并管理一个受约束的线程组（threads group）。如果初始池大小和最大池大小相同，因此ThreadPoolExecutor在实例化时会创建所有线程对象。例如：

- kotlin

- ```kotlin
  object PhotoManager {
      ...
      // Sets the amount of time an idle thread waits before terminating
      private const val KEEP_ALIVE_TIME = 1L
      // Sets the Time Unit to seconds
      private val KEEP_ALIVE_TIME_UNIT = TimeUnit.SECONDS
      // Creates a thread pool manager
      private val decodeThreadPool: ThreadPoolExecutor = ThreadPoolExecutor(
              NUMBER_OF_CORES,       // Initial pool size
              NUMBER_OF_CORES,       // Max pool size
              KEEP_ALIVE_TIME,
              KEEP_ALIVE_TIME_UNIT,
              decodeWorkQueue
      )
  ```

- java

- ```java
      private PhotoManager() {
          ...
          // Sets the amount of time an idle thread waits before terminating
          private static final int KEEP_ALIVE_TIME = 1;
          // Sets the Time Unit to seconds
          private static final TimeUnit KEEP_ALIVE_TIME_UNIT = TimeUnit.SECONDS;
          // Creates a thread pool manager
          decodeThreadPool = new ThreadPoolExecutor(
                  NUMBER_OF_CORES,       // Initial pool size
                  NUMBER_OF_CORES,       // Max pool size
                  KEEP_ALIVE_TIME,
                  KEEP_ALIVE_TIME_UNIT,
                  decodeWorkQueue);
      }
  ```

或者，如果您不想详细管理线程池大小的调整，您可能会发现用于创建[single thread executor](https://developer.android.com/reference/java/util/concurrent/Executors#newSingleThreadExecutor())或[work stealing executor](https://developer.android.com/reference/java/util/concurrent/Executors.html#newWorkStealingPool())的[Executors](https://developer.android.com/reference/java/util/concurrent/Executors)工厂方法更方便。

## 更多内容

关于多线程的更多内容，请参阅[进程和线程——概述](…/BestPractices/Performance/进程和线程——概述.md)。

