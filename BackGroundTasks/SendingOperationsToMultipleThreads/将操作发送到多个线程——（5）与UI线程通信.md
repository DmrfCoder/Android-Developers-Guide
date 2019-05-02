# 将操作发送到多个线程——与UI线程通信

前一篇文章[将操作发送到多个线程——在线程池中的线程中运行代码](./将操作发送到多个线程——在线程池中的线程中运行代码.md)，向您展示如何在`ThreadPoolExecutor`管理的线程上启动任务。本文将向您展示如何将任务中的数据发送给在用户界面（`UI`）线程上运行的对象。这允许您的任务执行后台工作，然后将结果返回到`UI`元素（如`bitmap`）。

每个应用程序都有自己的特殊线程来运行`UI`对象（例如`View`对象），这个线程称为`UI`线程。只有在`UI`线程上运行的对象才能访问该线程上的其他对象。由于您在线程池中的线程中运行的任务未在`UI`线程上运行，因此它们无权访问`UI`中的对象。要将数据从后台线程发送到`UI`线程，请使用在`UI`线程上运行的`Handler`。

## 在UI线程定义一个Handler

`Handler`是`Android`系统`Framework`层的一部分，其作用是管理线程。 `Handler`对象接收消息并运行代码来处理消息。通常，您为新线程创建一个`Handler`，但您也可以创建一个连接到现有线程的`Handler`。将`Handler`连接到`UI`线程时，处理消息的代码会在`UI`线程上运行。

在构造创建线程池的类的过程中实例化`Handler`对象，并将对象存储在全局变量中。通过使用`Handler（Looper）`构造函数将其实例化，将其连接到`UI`线程。此构造函数使用`Looper`对象，这是`Android`系统的线程管理框架的另一部分。当您基于特定的`Looper`实例实例化`Handler`时，`Handler`在与`Looper`相同的线程上运行。例如：

- kotlin

- ```kotlin
  object PhotoManager {
  ...
      private val handler: Handler = Handler(Looper.getMainLooper())
      ...
  }
  ```

- java

- ```java
  private PhotoManager() {
  ...
      // Defines a Handler object that's attached to the UI thread
      handler = new Handler(Looper.getMainLooper()) {
      ...
  ```

在`Handler`中，重写`handleMessage()`方法，`Android`系统会在接收到新`message`的时候调用`handleMessage()`方法，一个线程的所有`Handler`都会接收到相同的消息。比如：

- kotlin

- ```kotlin
  object PhotoManager {
      private val handler: Handler = object : Handler(Looper.getMainLooper()) {
          /*
           * handleMessage() defines the operations to perform when
           * the Handler receives a new Message to process.
           */
          override fun handleMessage(inputMessage: Message) {
              // Gets the image task from the incoming Message object.
              val photoTask = inputMessage.obj as PhotoTask
              ...
          }
      }
      ...
  }
  ```

- java

- ```java
          /*
           * handleMessage() defines the operations to perform when
           * the Handler receives a new Message to process.
           */
          @Override
          public void handleMessage(Message inputMessage) {
              // Gets the image task from the incoming Message object.
              PhotoTask photoTask = (PhotoTask) inputMessage.obj;
              ...
          }
      ...
      }
  }
  ```

下一节将介绍如何通知`Handler`移动数据。

## 将数据从任务线程传送到UI线程

要把运行在后台线程的任务中的数据传送到`UI`线程上中，请首先将后台任务中数据的引用以及UI对象的引用存储下来。接下来，将要传送的数据和状态代码传递给实例化`Handler`的对象，在此对象中，将含有状态和要发送数据的`Message`通过Handler对象发送到其`handleMessage（）`中。因为`Handler`在`UI`线程上运行，所以它可以将数据传送到UI对象。

### 后台线程的对象中存储数据

例如，这是一个在后台线程上运行的`Runnable`，它解码`Bitmap`并将其存储在其父对象`PhotoTask`中， `Runnable`还存储状态代码`DECODE_STATE_COMPLETED`。

- kotlin

- ```kotlin
  const val DECODE_STATE_COMPLETED: Int = ...
  
  // A class that decodes photo files into Bitmaps
  class PhotoDecodeRunnable(
          private val photoTask: PhotoTask,
          // Gets the downloaded byte array
          private var imageBuffer: ByteArray = photoTask.getByteBuffer()
  ) : Runnable {
      ...
      // Runs the code for this task
      override fun run() {
          ...
          // Tries to decode the image buffer
          BitmapFactory.decodeByteArray(
                  imageBuffer,
                  0,
                  imageBuffer.size,
                  bitmapOptions
          )?.also { returnBitmap ->
              ...
              // Sets the ImageView Bitmap
              photoTask.image = returnBitmap
          }
          // Reports a status of "completed"
          photoTask.handleDecodeState(DECODE_STATE_COMPLETED)
          ...
      }
      ...
  }
  ```

- java

- ```java
  // A class that decodes photo files into Bitmaps
  class PhotoDecodeRunnable implements Runnable {
      ...
      PhotoDecodeRunnable(PhotoTask downloadTask) {
          photoTask = downloadTask;
      }
      ...
      // Gets the downloaded byte array
      byte[] imageBuffer = photoTask.getByteBuffer();
      ...
      // Runs the code for this task
      public void run() {
          ...
          // Tries to decode the image buffer
          returnBitmap = BitmapFactory.decodeByteArray(
                  imageBuffer,
                  0,
                  imageBuffer.length,
                  bitmapOptions
          );
          ...
          // Sets the ImageView Bitmap
          photoTask.setImage(returnBitmap);
          // Reports a status of "completed"
          photoTask.handleDecodeState(DECODE_STATE_COMPLETED);
          ...
      }
      ...
  }
  ...
  ```

`PhotoTask`还包含一个显示`Bitmap`的`ImageView`的句柄。但即使`Bitmap`和`ImageView`的引用位于同一对象中，也无法将`Bitmap`分配给`ImageView`，因为您当前没有在`UI`线程上运行。

所以，下一步是将数据发送到`PhotoTask`对象。

### 将数据发送到对象层

`PhotoTask`是层次结构中的下一个更高级的对象。它维护对已解码数据的引用以及显示数据的`View`对象。它从`PhotoDecodeRunnable`接收状态代码并将其传递给维护线程池的对象并实例化`Handler`：

- kotlin

- ```kotlin
  // Gets a handle to the object that creates the thread pools
  class PhotoTask() {
      ...
      private val photoManager: PhotoManager = PhotoManager.getInstance()
      ...
      fun handleDecodeState(state: Int) {
          // Converts the decode state to the overall state.
          val outState: Int = when(state) {
              PhotoDecodeRunnable.DECODE_STATE_COMPLETED -> PhotoManager.TASK_COMPLETE
              ...
          }
          ...
          // Calls the generalized state method
          handleState(outState)
      }
      ...
      // Passes the state to PhotoManager
      private fun handleState(state: Int) {
          /*
           * Passes a handle to this task and the
           * current state to the class that created
           * the thread pools
           */
          PhotoManager.handleState(this, state)
      }
      ...
  }
  ```

- java

- ```java
  public class PhotoTask {
      ...
      // Gets a handle to the object that creates the thread pools
      photoManager = PhotoManager.getInstance();
      ...
      public void handleDecodeState(int state) {
          int outState;
          // Converts the decode state to the overall state.
          switch(state) {
              case PhotoDecodeRunnable.DECODE_STATE_COMPLETED:
                  outState = PhotoManager.TASK_COMPLETE;
                  break;
              ...
          }
          ...
          // Calls the generalized state method
          handleState(outState);
      }
      ...
      // Passes the state to PhotoManager
      void handleState(int state) {
          /*
           * Passes a handle to this task and the
           * current state to the class that created
           * the thread pools
           */
          photoManager.handleState(this, state);
      }
      ...
  }
  ```

### 将数据传送到UI

`PhotoManager`对象从`PhotoTask`对象接收状态代码和`PhotoTask`对象的句柄。由于状态为`TASK_COMPLETE`，因此创建一个包含状态码和要传送数据的`Message`，并将其发送给`Handler`：

- kotlin

- ```kotlin
  object PhotoManager {
      ...
      // Handle status messages from tasks
      fun handleState(photoTask: PhotoTask, state: Int) {
          when(state) {
              ...
              TASK_COMPLETE -> { // The task finished downloading and decoding the image
                  /*
                   * Creates a message for the Handler
                   * with the state and the task object
                   */
                  handler.obtainMessage(state, photoTask)?.apply {
                      sendToTarget()
                  }
              }
              ...
          }
          ...
      }
  ```

- java

  ```java
  public class PhotoManager {
      ...
      // Handle status messages from tasks
      public void handleState(PhotoTask photoTask, int state) {
          switch (state) {
              ...
              // The task finished downloading and decoding the image
              case TASK_COMPLETE:
                  /*
                   * Creates a message for the Handler
                   * with the state and the task object
                   */
                  Message completeMessage =
                          handler.obtainMessage(state, photoTask);
                  completeMessage.sendToTarget();
                  break;
              ...
          }
          ...
      }
  ```

  最后，`Handler.handleMessage（）`检查每个传入消息的状态代码。如果状态代码为`TASK_COMPLETE`，则任务结束，`Message`中的`PhotoTask`对象包含`Bitmap`和`ImageView`，因为`Handler.handleMessage（）`在`UI`线程上运行，所以它可以安全地将`Bitmap`移动到`ImageView`：

- kotlin

- ```kotlin
      object PhotoManager {
          ...
          private val handler: Handler = object : Handler(Looper.getMainLooper()) {
  
              override fun handleMessage(inputMessage: Message) {
                  // Gets the image task from the incoming Message object.
                  val photoTask = inputMessage.obj as PhotoTask
                  // Gets the ImageView for this task
                  val localView: PhotoView = photoTask.getPhotoView()
                  ...
                  when (inputMessage.what) {
                      ...
                      TASK_COMPLETE -> localView.setImageBitmap(photoTask.image)
                      ...
                      else -> super.handleMessage(inputMessage)
                  }
                  ...
              }
              ...
          }
          ...
      ...
      }
  ...
  }
  ```

- java

- ```java
      private PhotoManager() {
          ...
              handler = new Handler(Looper.getMainLooper()) {
                  @Override
                  public void handleMessage(Message inputMessage) {
                      // Gets the task from the incoming Message object.
                      PhotoTask photoTask = (PhotoTask) inputMessage.obj;
                      // Gets the ImageView for this task
                      PhotoView localView = photoTask.getPhotoView();
                      ...
                      switch (inputMessage.what) {
                          ...
                          // The decoding is done
                          case TASK_COMPLETE:
                              /*
                               * Moves the Bitmap from the task
                               * to the View
                               */
                              localView.setImageBitmap(photoTask.getImage());
                              break;
                          ...
                          default:
                              /*
                               * Pass along other messages from the UI
                               */
                              super.handleMessage(inputMessage);
                      }
                      ...
                  }
                  ...
              }
              ...
      }
  ...
  }
  ```

## 更多信息

关于多线程的更多内容，请参阅[进程和线程——概述](…/BestPractices/Performance/进程和线程——概述.md)。