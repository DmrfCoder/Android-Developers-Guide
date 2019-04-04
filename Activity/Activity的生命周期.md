# Activity的生命周期（理解Activity的生命周期）

[原文（英文）地址](https://developer.android.com/guide/components/activities/activity-lifecycle)

当用户导航（navigates through），退出和返回您的应用时，您应用中的Activity实例会在其生命周期中的不同状态中进行转换。 Activity类提供了许多回调方法，这些回调方法允许Activity知道状态已被更改：系统正在创建（creating），停止（stopping）或恢复（resuming），或者破坏（destroying）Activity所在的进程。

在生命周期的回调方法中，您可以指定用户离开和重新进入Activity时的行为。例如，如果您正在构建流式视频播放器，则可能会在用户切换到另一个应用时暂停视频并终止网络连接。当用户返回时，您可以重新连接到网络并允许用户从同一位置观看视频。换句话说，每个回调允许您执行适合于给定状态更改的特定工作。在正确的时间做正确的工作并正确处理过渡可以使您的应用程序更加强大和高效。例如，良好地实现生命周期的回调可以帮助确保您的应用程序避免以下几种情况：

- 用户在使用您的应用时接到电话或切换到其他应用导致本应用崩溃。
- 当用户不主动使用它时，消耗宝贵的系统资源。
- 如果用户离开您的应用并稍后返回时丢失用户的进度。
- 当屏幕在横向和纵向之间旋转时，应用发生崩溃或丢失用户的进度。

本节详细说明了Activity的生命周期。首先描述了生命周期范例。接下来，解释了每个回调在执行时内部发生的逻辑，以及在你在不同的回调方法中应该实现的逻辑。然后简要介绍了Activity状态与进程被系统杀死的漏洞之间的关系。最后，它讨论了与Activity不同状态之间转换相关的一些内容。

有关处理生命周期的信息（包括有关最佳实践的指导），请参阅[Handling Lifecycles with Lifecycle-Aware Components](https://developer.android.com/topic/libraries/architecture/lifecycle.html) 和 [Saving UI States](https://developer.android.com/topic/libraries/architecture/saving-states.html). 。要了解如何使用与体系结构组件结合的Activity来构建强大的，生产质量的应用程序，请参阅[Guide to App Architecture.](https://developer.android.com/topic/libraries/architecture/guide.html)。

## Activity生命周期的概念

为了在Activity生命周期的各个阶段之间切换，Activity提供了6个核心的回调方法：onCreate（），onStart（），onResume（），onPause（），onStop（）和onDestroy（）。当Activity进入不同的状态时，系统会回调对应阶段的回调方法。

下图直观的画出了各个回调方法的执行逻辑：

![activity_lifecycle](https://ws4.sinaimg.cn/large/006tKfTcgy1g1pnxkgeq3j30e90ifjsc.jpg)

当用户开始离开Activity时，系统调用一些方法来移除Activity。在某些情况下，这种拆除只是部分的，Activity仍然驻留在内存中（例如当用户切换到另一个应用程序时），并且仍然可以返回到前台。如果用户返回该Activity，则Activity将恢复到用户停止的位置。系统是否杀死给定进程及其中的Activity取决于当时Activity的状态。 [Activity state and ejection from memory](https://developer.android.com/guide/components/activities/activity-lifecycle#asem) provides more information on the relationship between state and vulnerability to ejection.（这个应该如何翻译？）

根据Activity的不同的复杂程度，您可能不需要实现所有生命周期对应的回调方法。但是，了解每一个毁掉方法并掌握其使用对确保您的应用程序符合用户期望的行为非常重要。

本文档的下一部分介绍了不同状态间转换对应的不同回调的具体情况。

## 不同生命周期的回调

本节主要介绍有关在Activity生命周期中回调方法使用的概念和方法。

某些操作（如调用setContentView（））属于Activity生命周期方法本身。但是，实现依赖组件操作的代码应放在组件本身。要实现此目的，您必须使依赖组件生命周期可见。请参阅[Handling Lifecycles with Lifecycle-Aware Components](https://developer.android.com/topic/libraries/architecture/lifecycle.html)以了解如何保证您的依赖组件生命周期可见性。

这一段不理解，原文如下：

Some actions, such as calling [`setContentView()`](https://developer.android.com/reference/android/app/Activity#setcontentview_12), belong in the activity lifecycle methods themselves. However, the code implementing the actions of a dependent component should be placed in the component itself. To achieve this, you must make the dependent component lifecycle-aware. See [Handling Lifecycles with Lifecycle-Aware Components](https://developer.android.com/topic/libraries/architecture/lifecycle.html) to learn how to make your dependent components lifecycle-aware.

### onCreate（）

您必须实现此回调方法，该方法在系统首次创建Activity时触发。在Activity创建时，Activity进入创建状态（Created state）。在onCreate（）方法中，您应该执行应用程序基本的启动逻辑，该逻辑在Activity的整个生命周期中只应发生一次。例如，onCreate（）中可能会将数据绑定到列表（list），将Activity与ViewModel相关联，并实例化一些类级变量（class-scope variables）。此方法接收参数savedInstanceState，该方法的参数参数是Activity先前用于保存状态的Bundle对象。如果Activity以前从未存在过，则Bundle对象的值为null。

如果您有一个生命周期感知组件（lifecycle-aware component ）与您的Activity的生命周期相关联，它将收到ON_CREATE事件，同时将调用使用@OnLifecycleEvent注释的方法，以便您的生命周期感知组件可以执行创建状态所需的任何代码。

以下onCreate（）方法示例显示了Activity的基本设置，例如声明用户界面（在XML布局文件中定义），定义成员变量以及配置某些UI。在此示例中，通过将文件的资源ID R.layout.main_activity传递给setContentView（）来指定XML布局文件：

- Kotlin

  ```kotlin
  lateinit var textView: TextView
  
  // some transient state for the activity instance
  var gameState: String? = null
  
  override fun onCreate(savedInstanceState: Bundle?) {
      // call the super class onCreate to complete the creation of activity like
      // the view hierarchy
      super.onCreate(savedInstanceState)
  
      // recovering the instance state
      gameState = savedInstanceState?.getString(GAME_STATE_KEY)
  
      // set the user interface layout for this activity
      // the layout file is defined in the project res/layout/main_activity.xml file
      setContentView(R.layout.activity_main)
  
      // initialize member TextView so we can manipulate it later
      textView = findViewById(R.id.text_view)
  }
  
  // This callback is called only when there is a saved instance that is previously saved by using
  // onSaveInstanceState(). We restore some state in onCreate(), while we can optionally restore
  // other state here, possibly usable after onStart() has completed.
  // The savedInstanceState Bundle is same as the one used in onCreate().
  override fun onRestoreInstanceState(savedInstanceState: Bundle?) {
      textView.text = savedInstanceState?.getString(TEXT_VIEW_KEY)
  }
  
  // invoked when the activity may be temporarily destroyed, save the instance state here
  override fun onSaveInstanceState(outState: Bundle?) {
      outState?.run {
          putString(GAME_STATE_KEY, gameState)
          putString(TEXT_VIEW_KEY, textView.text.toString())
      }
      // call superclass to save any view hierarchy
      super.onSaveInstanceState(outState)
  }
  ```

  

- Java

  ```java
  TextView textView;
  
  // some transient state for the activity instance
  String gameState;
  
  @Override
  public void onCreate(Bundle savedInstanceState) {
      // call the super class onCreate to complete the creation of activity like
      // the view hierarchy
      super.onCreate(savedInstanceState);
  
      // recovering the instance state
      if (savedInstanceState != null) {
          gameState = savedInstanceState.getString(GAME_STATE_KEY);
      }
  
      // set the user interface layout for this activity
      // the layout file is defined in the project res/layout/main_activity.xml file
      setContentView(R.layout.main_activity);
  
      // initialize member TextView so we can manipulate it later
      textView = (TextView) findViewById(R.id.text_view);
  }
  
  // This callback is called only when there is a saved instance that is previously saved by using
  // onSaveInstanceState(). We restore some state in onCreate(), while we can optionally restore
  // other state here, possibly usable after onStart() has completed.
  // The savedInstanceState Bundle is same as the one used in onCreate().
  @Override
  public void onRestoreInstanceState(Bundle savedInstanceState) {
      textView.setText(savedInstanceState.getString(TEXT_VIEW_KEY));
  }
  
  // invoked when the activity may be temporarily destroyed, save the instance state here
  @Override
  public void onSaveInstanceState(Bundle outState) {
      outState.putString(GAME_STATE_KEY, gameState);
      outState.putString(TEXT_VIEW_KEY, textView.getText());
  
      // call superclass to save any view hierarchy
      super.onSaveInstanceState(outState);
  }
  ```

相比于直接设置xml文件作为视图层次结构，还有一个方法是在Activity中使用代码创建新的View对象，并通过将该View对象插入到ViewGroup中来构建视图层次结构。然后，通过将根ViewGroup传递给setContentView（）来使用该布局。有关创建用户界面的更多信息，请参阅 [User Interface](https://developer.android.com/guide/topics/ui/index.html) 文档。

 onCreate（）方法完成执行后Activity就不是Created状态了，Activity会进入Started状态，系统快速连续调用onStart（）和onResume（）方法。下一节将介绍onStart（）。

### onStart（）

当Activity进入Started状态时，系统将调用此回调方法。 onStart（）使Activity对用户可见，同时应用程序为Activity进入前台及Activity变成可交互的做准备工作。例如，此方法是应用程序初始化维护UI的代码的位置。

当Activity进入启动状态时，任何与Activity生命周期相关的生命周期感知组件（lifecycle-aware component）都将收到ON_START事件。

onStart（）方法非常快速地完成，并且与Created状态一样，Activity不会保持驻留在Started状态。一旦此回调结束，Activity就进入Resumed状态，系统将调用onResume（）方法。

### onResume（）

当Activity进入Resumed状态时，它进入前台，然后系统调用onResume（）回调方法。这是应用程序与用户交互的状态。该应用程序保持这种状态直到某些其他的事件使当前应用程序失去焦点。例如，这样的事件可能是接收电话，用户导航到另一个Activity，或者设备屏幕关闭。

当Activity进入恢复状态（Resume state）时，任何与Activity生命周期相关的生命周期感知组件都将收到ON_RESUME事件。这是生命周期组件可以启用任何需要在组件可见且在前台运行时运行的功能的位置，例如启动摄像头预览。

当发生中断事件时，Activity进入Paused状态，系统调用onPause（）回调。

如果Activity从Paused状态返回Resumed状态，则系统再次调用onResume（）方法。因此，您应该实现onResume（）来初始化在onPause（）期间释放的组件，并执行每次Activity进入Resumed状态时必须进行的任何其他初始化。

以下是在组件收到ON_RESUME事件时访问摄像头的生命周期感知组件的示例：

- Kotlin

  ```kotlin
  class CameraComponent : LifecycleObserver {
  
      ...
  
      @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
      fun initializeCamera() {
          if (camera == null) {
              getCamera()
          }
      }
  
      ...
  }
  ```

- Java

  ```java
  public class CameraComponent implements LifecycleObserver {
  
      ...
  
      @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
      public void initializeCamera() {
          if (camera == null) {
              getCamera();
          }
      }
  
      ...
  }
  
  ```

一旦LifecycleObserver收到ON_RESUME事件，上面的代码就会初始化相机。但是，在多窗口模式（multi-window mode）下，即使Activity处于暂停状态，您的Activity也可能完全可见。例如，当用户处于多窗口模式并点击不包含您的Activity的其他窗口时，您的Activity将移至暂停状态。如果您希望仅在应用程序在Resumed状态（在前台可见且拥有焦点）时使您的相机生效，则在上面演示的ON_RESUME事件后初始化相机。如果您想在Activity暂停（Paused状态）但是可见时（例如在多窗口模式下）保持相机处于Activity状态，则应在ON_START事件后初始化相机。但请注意，在Activity暂停时让相机处于生效状态可能会在多窗口模式下拒绝另一个已生效的应用程序（Resumed状态）对相机的访问。有时可能需要在Activity暂停（Paused）时保持相机处于生效状态，但如果这样做，实际上可能会降低整体用户体验。仔细考虑生命周期中的哪个位置更适合在多窗口环境中控制共享系统资源。要了解有关支持多窗口模式的更多信息，请参阅[Multi-Window Support](https://developer.android.com/guide/topics/ui/multi-window.html).

无论您选择在哪个构建事件中执行初始化操作，请确保使用相应的生命周期事件来释放资源。如果在ON_START事件之后初始化某些内容，则在ON_STOP事件之后释放或终止它。如果在ON_RESUME事件之后初始化，则在ON_PAUSE事件之后释放。

请注意，上面的代码片段将相机初始化代码放在生命周期感知组件中。您可以将此代码直接放入Activity生命周期回调中，例如onStart（）和onStop（），但不建议这样做。将此逻辑添加到独立的生命周期感知组件中，您可以跨多个Activity重用该组件，而无需重复代码。请参阅 [Handling Lifecycles with Lifecycle-Aware Components](https://developer.android.com/topic/libraries/architecture/lifecycle.html) 以了解如何创建生命周期感知组件。

### onPause（）

系统将此方法称为用户离开您的Activity的第一个标志（尽管并不总是意味着Activity正在被销毁），它表示Activity不再在前台（尽管如果用户处于多窗口模式，它仍然可见）。

使用onPause（）方法暂停或调整Activity处于Paused状态时不应继续（或应适度继续）并且您希望很快恢复的操作。

（这句感觉翻译的不太好，原文：Use the [`onPause()`](https://developer.android.com/reference/android/app/Activity.html#onpause) method to pause or adjust operations that should not continue (or should continue in moderation) while the [`Activity`](https://developer.android.com/reference/android/app/Activity.html) is in the Paused state, and that you expect to resume shortly. ）

例如：

- 某些事件会中断应用程序执行，如onResume（）部分所述。这是最常见的情况。
- 在Android 7.0（API级别24）或更高版本中，多个应用程序以多窗口模式运行。由于只有一个应用程序（窗口）可以占有焦点，系统会暂停所有其他应用程序。
- 将打开一个新的半透明Activity（如对话框）。只要Activity仍然部分可见但不是焦点，它仍然暂停。

当Activity进入暂停（paused）状态时，任何与Activity生命周期相关的生命周期感知组件都将收到ON_PAUSE事件。这是生命周期组件可以停止Activity不在前台时不需要运行的功能的地方，例如停止相机预览。

您还可以使用onPause（）方法释放系统资源，处理传感器（如GPS），或者在您的Activity暂停且用户不需要时可能影响电池寿命的任何资源。但是，如上面onResume（）部分所述，如果处于多窗口模式，PausedActivity仍然可以完全可见。因此，您应该考虑使用onStop（）而不是onPause（）来完全释放或调整与UI相关的资源和操作，以更好地支持多窗口模式。

下面对On_PAUSE事件做出反应的LifecycleObserver示例是上面ON_RESUME事件示例的对应示例，释放了在收到ON_RESUME事件后初始化的摄像头：

- Kotlin

  ```kotlin
  class CameraComponent : LifecycleObserver {
  
      ...
  
      @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
      fun releaseCamera() {
          camera?.release()
          camera = null
      }
  
      ...
  }
  ```

- java

  ```java
  public class JavaCameraComponent implements LifecycleObserver {
  
      ...
  
      @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
      public void releaseCamera() {
          if (camera != null) {
              camera.release();
              camera = null;
          }
      }
  
      ...
  }
  ```

注意，上面的代码片段在LifecycleObserver收到ON_PAUSE事件释放相机资源。如前所述，请参阅 [Handling Lifecycles with Lifecycle-Aware Components](https://developer.android.com/topic/libraries/architecture/lifecycle.html)以了解如何创建生命周期感知组件。

onPause（）执行非常简短，并不一定要有足够的时间来执行保存操作。因此，您不应使用onPause（）来保存应用程序或用户数据，进行网络调用或执行数据库事务，因为在方法完成之前，此类工作可能无法完成。您应该在onStop（）中执行重负载的关闭操作。有关在onStop（）期间执行的合适操作的更多信息，请参阅onStop（）。有关保存数据的更多信息，请参阅[Saving and restoring activity state](https://developer.android.com/guide/components/activities/activity-lifecycle#saras).

完成onPause（）方法并不意味着Activity离开Paused状态。相反，Activity保持在此状态，直到Activity恢复或变得对用户完全不可见。如果Activity恢复，系统将再次调用onResume（）回调。如果Activity从Paused状态返回到Resumed状态，系统会将Activity实例驻留在内存中，在系统调用onResume（）时调用该实例。在这种情况下，您无需重新初始化在任何导致Resumed状态的回调方法期间创建的组件。如果Activity变得完全不可见，则系统调用onStop（）。

### onStop（）

当您的Activity不再对用户可见时，它已进入Stopped状态，系统将调用onStop（）回调。例如，当新启动的Activity覆盖整个屏幕时，可能会发生这种情况。当Activity完成运行并且即将终止时，系统也可以调用onStop（）。

当Activity进入停止状态时，任何与Activity生命周期相关的生命周期感知组件都将收到ON_STOP事件。这是生命周期组件可以停止Activity在屏幕上不可见时不需要运行的功能的地方。

在onStop（）方法中，应用程序应释放或调整应用程序对用户不可见时不需要的资源。例如，您的应用可能会暂停动画或从细粒度位置更新切换到粗粒度位置更新。使用onStop（）而不是onPause（）可确保与UI相关的工作继续进行，即使用户在多窗口模式下查看您的Activity也是如此。

您还应该使用onStop（）执行相对CPU密集型的关闭操作。例如，如果找不到更合适的时间将信息保存到数据库，则可以在onStop（）期间执行此操作。以下示例显示onStop（）的实现，该实现将草稿注释（draft note）的内容保存到持久存储：

- kotlin

  ```kotlin
  override fun onStop() {
      // call the superclass method first
      super.onStop()
  
      // save the note's current draft, because the activity is stopping
      // and we want to be sure the current note progress isn't lost.
      val values = ContentValues().apply {
          put(NotePad.Notes.COLUMN_NAME_NOTE, getCurrentNoteText())
          put(NotePad.Notes.COLUMN_NAME_TITLE, getCurrentNoteTitle())
      }
  
      // do this update in background on an AsyncQueryHandler or equivalent
      asyncQueryHandler.startUpdate(
              token,     // int token to correlate calls
              null,      // cookie, not used here
              uri,       // The URI for the note to update.
              values,    // The map of column names and new values to apply to them.
              null,      // No SELECT criteria are used.
              null       // No WHERE columns are used.
      )
  }
  ```

- java

  ```java
  @Override
  protected void onStop() {
      // call the superclass method first
      super.onStop();
  
      // save the note's current draft, because the activity is stopping
      // and we want to be sure the current note progress isn't lost.
      ContentValues values = new ContentValues();
      values.put(NotePad.Notes.COLUMN_NAME_NOTE, getCurrentNoteText());
      values.put(NotePad.Notes.COLUMN_NAME_TITLE, getCurrentNoteTitle());
  
      // do this update in background on an AsyncQueryHandler or equivalent
      asyncQueryHandler.startUpdate (
              mToken,  // int token to correlate calls
              null,    // cookie, not used here
              uri,    // The URI for the note to update.
              values,  // The map of column names and new values to apply to them.
              null,    // No SELECT criteria are used.
              null     // No WHERE columns are used.
      );
  }
  ```

  

注意，上面的代码示例直接使用SQLite。您应该使用Room，这是一个提供SQLite抽象层的持久性库。要了解有关使用Room的好处以及如何在应用程序中实现Room的更多信息，请参阅 [Room Persistence Library](https://developer.android.com/topic/libraries/architecture/room.html) 指南。

当您的Activity进入Stopped状态时，Activity对象将保留在内存中：它维护所有状态和成员信息，但不会附加到窗口管理器。Activity恢复后，Activity会恢复此信息。您不需要重新初始化任何在导致Resumed状态发生的回调方法中创建的组件（也就是说如果某一个方法执行完成之后Activity进入了Resumed状态，则当Activity恢复时你不需要手动重新初始化该方法中初始化的组件）。系统还会跟踪布局中每个View对象的当前状态，因此，如用户在EditText小部件中输入的文本，则会保留该内容，因此您无需保存和还原它。

> 注意：停止Activity后，如果系统需要恢复内存，系统可能会破坏包含Activity的进程。即使系统在活动停止时销毁进程，系统仍会保留Bundle（一对键值对）中的View对象（例如EditText小部件中的文本）的状态，并在用户恢复它们时导航回Activity。有关还原用户返回的活动的详细信息，请参阅[Saving and restoring activity state](https://developer.android.com/guide/components/activities/activity-lifecycle#saras).。

Stopped状态之后，Activity要么返回与用户交互，要么Activity已完成运行并消失。如果Activity返回，系统将调用onRestart（）。如果Activity已完成运行，则系统调用onDestroy（）。

### onDestroy（）

onDestroy（）方法在Activity被系统销毁之前回调，一下场景会导致系统回调onDestroy（）：

- Activity正在结束（由于用户完全移除Activity或由于Activity调用完成）
- 由于配置更改（例如设备旋转或多窗口模式），系统暂时销毁Activity

当Activity进入销毁状态时，任何与Activity生命周期相关的生命周期感知组件都将收到ON_DESTROY事件。这是生命周期组件可以销毁在销毁Activity之前所需的任何内容的地方（释放所有资源）。

您应该使用ViewModel对象来包含Activity的相关视图数据，而不是在您的Activity中使用逻辑判断Activity被销毁的原因。如果由于配置更改而将重新创建Activity，则ViewModel不必执行任何操作，因为它将被保留并提供给下一个Activity实例。如果不重新创建Activity，则ViewModel将调用onCleared（）方法，以便在销毁之前清除所需的任何数据。您可以使用isFinishing（）方法区分这两种情况。

如果Activity正在完成（is finishing），则onDestroy（）是Activity接收到的最终生命周期回调。如果由于配置更改而调用onDestroy（），系统会立即创建新的Activity实例，然后在新配置中的新实例上调用onCreate（）。

onDestroy（）回调应该释放早期回调（例如onStop（））尚未释放的所有资源。

## Activity的状态与及内存回收（ejection from memory）

系统在需要释放RAM时会终止进程，系统杀死给定进程的可能性取决于当时进程的状态。反过来，进程状态取决于进程中运行的Activity的状态。表1显示了进程状态，Activity状态和系统终止进程的可能性之间的相关性：

| 进程被终止的可能性 | 进程状态                   | Activity状态              |
| ------------------ | -------------------------- | ------------------------- |
| 小                 | 前台（拥有或即将拥有焦点） | Created，Started，Resumed |
| 中                 | 后台（失去焦点）           | Paused                    |
| 大                 | 后台（失去焦点）           | Stopped                   |
| 大                 | 空                         | Destroyed                 |

系统永远不会直接杀死Activity以释放内存，它会杀死Activity运行的进程，不仅会破坏当前Activity，还会破坏进程中运行的所有其他Activity。要了解在系统启动的进程被销毁时如何保留和恢复Activity的UI状态，请参阅[Saving and restoring activity state](https://developer.android.com/guide/components/activities/activity-lifecycle#saras).

用户还可以使用“设置”下的“应用程序管理器”终止相应的应用程序。

有关一般进程的更多信息，请参阅 [Processes and Threads](https://developer.android.com/guide/components/processes-and-threads.html). 有关进程生命周期如何与其中Activity状态相关联的更多信息，请参阅该页面的“ [Process Lifecycle](https://developer.android.com/guide/components/processes-and-threads.html#Lifecycle)”部分。

## 保存和恢复瞬间UI状态

用户期望Activity的UI状态在整个配置更改期间保持不变，例如旋转或切换到多窗口模式。但是，当发生此类配置更改时，系统会默认销毁Activity，从而消除存储在Activity实例中的任何UI状态。同样，如果用户暂时从您的应用切换到其他应用，然后稍后再回到您的应用，用户同样希望UI状态保持不变。但是，当用户离开并且您的Activity停止时，系统可能会释放您的应用程序的进程。

当Activity因系统限制而被销毁时，您应该使用ViewModel，onSaveInstanceState（）和/或本地存储的组合来保留用户的瞬态UI状态。要了解有关用户期望与系统行为的更多信息，以及如何在系统启动的Activity和流程死亡中最好地保留复杂的UI状态数据，请参阅 [Saving UI State](https://developer.android.com/topic/libraries/architecture/saving-states.html).

下一节概述了实例状态（instance state）以及如何实现onSaveInstance（）方法，该方法是对Activity本身的回调。如果您的UI数据简单而轻量级，例如原始数据类型或简单对象（如String），则可以单独使用onSaveInstanceState（）来保持UI状态跨越配置更改和系统启动的进程死亡。但是，在大多数情况下，您应该使用ViewModel和onSaveInstanceState（）（如 [Saving UI State](https://developer.android.com/topic/libraries/architecture/saving-states.html)中所述），因为onSaveInstanceState（）会存在序列化/反序列化成本。

### 实例状态（Instance state）

在某些情况下，您的Activity会因应用程序的正常行为而被销毁，例如当用户按下“返回”按钮或您的Activity通过调用finish（）方法发出自己需要被破坏信号时。当您的Activity因用户按下Back或Activity自行完成而被销毁时，系统和用户对该Activity实例的概念将永远消失。在这些情况下，用户的期望与系统的行为相匹配，您没有任何额外的工作要做。

但是，如果系统因系统约束（例如配置更改或内存压力）而破坏Activity，则虽然实际的Activity实例已消失，但系统会记住它已存在。如果用户尝试导航回Activity，系统将使用一组已保存的数据创建该Activity的新实例，这些数据描述了Activity在销毁时的状态。

系统用于恢复先前状态的已保存数据称为实例状态，是存储在Bundle对象中的键值对的集合。默认情况下，系统使用Bundle实例状态来保存Activity布局中每个View对象的信息（例如输入EditText小部件的文本值）。因此，如果您的Activity实例被销毁并重新创建，则布局的状态将恢复到之前的状态，而您无需自己编写代码。但是，您的Activity可能包含您要恢复的更多状态信息，例如跟踪用户在Activity中的进度的成员变量。

> 注意：为了让Android系统恢复Activity中View的状态，每个View必须具有唯一的ID，由android：id属性提供。

Bundle对象不适合保留过大的数据，因为它需要在主线程上进行序列化并消耗系统进程内存。要保留较多的数据，您应该采用组合方法来保留数据：使用持久本地存储，onSaveInstanceState（）方法和ViewModel类，如保存和恢复瞬间UI状态（上一节）中所述。

### 使用onSaveInstanceState（）保存简单、轻量的UI状态

当您的Activity开始停止（begin stop）时，系统会调用onSaveInstanceState（）方法，以便您的Activity可以将状态信息保存到实例状态的bundle中。此方法的默认实现会保存有关Activity视图层次结构状态的瞬态信息，例如EditText小部件中的文本或ListView小部件的滚动位置。

要保存Activity的其他实例状态信息，必须重写onSaveInstanceState（）并将键值对添加到Bundle对象，该对象在您的Activity意外销毁时保存。如果重写onSaveInstanceState（）时希望同时实现默认的保存视图层次结构的状态则必须调用超类实现。例如：

- kotlin

  ```kotlin
  override fun onSaveInstanceState(outState: Bundle?) {
      // Save the user's current game state
      outState?.run {
          putInt(STATE_SCORE, currentScore)
          putInt(STATE_LEVEL, currentLevel)
      }
  
      // Always call the superclass so it can save the view hierarchy state
      super.onSaveInstanceState(outState)
  }
  
  companion object {
      val STATE_SCORE = "playerScore"
      val STATE_LEVEL = "playerLevel"
  }
  ```

- java

  ```java
  static final String STATE_SCORE = "playerScore";
  static final String STATE_LEVEL = "playerLevel";
  // ...
  
  
  @Override
  public void onSaveInstanceState(Bundle savedInstanceState) {
      // Save the user's current game state
      savedInstanceState.putInt(STATE_SCORE, currentScore);
      savedInstanceState.putInt(STATE_LEVEL, currentLevel);
  
      // Always call the superclass so it can save the view hierarchy state
      super.onSaveInstanceState(savedInstanceState);
  }
  ```

> 注意：当用户显式关闭Activity时或调用finish（），不会调用onSaveInstanceState（）。

要保存持久性数据（例如用户首选项或数据库的数据），您应该在Activity处于前台状态时寻找合适的时机进行保存，如果没有这样的时机，您应该在onStop（）方法中保存这些数据。

### 使用已经保存的实例状态恢复UI状态

之前销毁Activity后重新创建Activity时，可以从系统传递给Activity的Bundle中恢复已保存的实例状态。 onCreate（）和onRestoreInstanceState（）回调方法都接收包含实例状态信息的相同Bundle。

因为无论系统是创建Activity的新实例还是重新创建前一个实例，都会调用onCreate（）方法，因此在尝试读取之前必须检查状态Bundle是否为null。如果它为null，则系统正在创建Activity的新实例，而不是恢复已销毁的先前实例。

例如，以下代码段显示了如何在onCreate（）中恢复某些状态数据：

- kotlin

  ```kotlin
  override fun onCreate(savedInstanceState: Bundle?) {
      super.onCreate(savedInstanceState) // Always call the superclass first
  
      // Check whether we're recreating a previously destroyed instance
      if (savedInstanceState != null) {
          with(savedInstanceState) {
              // Restore value of members from saved state
              currentScore = getInt(STATE_SCORE)
              currentLevel = getInt(STATE_LEVEL)
          }
      } else {
          // Probably initialize members with default values for a new instance
      }
      // ...
  }
  ```

- java

  ```java
  @Override
  protected void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState); // Always call the superclass first
  
      // Check whether we're recreating a previously destroyed instance
      if (savedInstanceState != null) {
          // Restore value of members from saved state
          currentScore = savedInstanceState.getInt(STATE_SCORE);
          currentLevel = savedInstanceState.getInt(STATE_LEVEL);
      } else {
          // Probably initialize members with default values for a new instance
      }
      // ...
  }
  ```

  

您可以选择实现onRestoreInstanceState（），在该方法中恢复状态而不是在onCreate（）中，系统在onStart（）方法之后会调用该方法。仅当存在要还原的已保存状态时，系统才会调用onRestoreInstanceState（），因此您无需检查Bundle是否为null：

- kotlin

- ```kotlin
  override fun onRestoreInstanceState(savedInstanceState: Bundle?) {
      // Always call the superclass so it can restore the view hierarchy
      super.onRestoreInstanceState(savedInstanceState)
  
      // Restore state members from saved instance
      savedInstanceState?.run {
          currentScore = getInt(STATE_SCORE)
          currentLevel = getInt(STATE_LEVEL)
      }
  }
  
  
  
  ```

- java

  ```java
  public void onRestoreInstanceState(Bundle savedInstanceState) {
      // Always call the superclass so it can restore the view hierarchy
      super.onRestoreInstanceState(savedInstanceState);
  
      // Restore state members from saved instance
      currentScore = savedInstanceState.getInt(STATE_SCORE);
      currentLevel = savedInstanceState.getInt(STATE_LEVEL);
  }
  ```

> 警告：应该始终调用onRestoreInstanceState（）的超类实现（即super.onRestireInstanceState(saveInstanceState)），以便默认实现可以恢复视图层次结构的状态。

## 在不同Activity之间导航

在应用程序的生命周期中，应用程序很可能会多次进入和退出Activity。例如，用户可以点击设备的“后退”按钮，或者Activity可能需要启动不同的Activity。本节介绍了实现Activity跳转所需了解的内容。这些内容名包括从其他Activity启动Activity，保存Activity状态以及恢复Activity状态。

### 在另一个Activity启动当前Activity

Activity通常需要在某个时刻启动另一个Activity。例如，当应用程序需要从当前屏幕移动到新屏幕时，就会出现这种需求。

根据您的Activity是否希望从即将启动的新Activity返回结果，您可以使用startActivity（）或startActivityForResult（）方法启动新Activity。这两种方法都需要传入一个Intent对象。

Intent对象指定要启动的Activity，或描述您要执行的操作类型（系统为您选择适当的Activity，甚至可以来自不同的应用程序）。 Intent对象还可以携带由启动的Activity使用的少量数据。有关Intent类的更多信息，请参阅[Intents and Intent Filters](https://developer.android.com/guide/components/intents-filters.html).。

### startActivity()

如果新启动的Activity不需要返回结果，则当前Activity可以通过调用startActivity（）方法启动它。

在Activity工作在自己的应用程序中时（本应用的Activity），通常需要简单地启动已知Activity。例如，以下代码段显示了如何启动名为SignInActivity的Activity：

- kotlin

  ```kotlin
  val intent = Intent(this, SignInActivity::class.java)
  startActivity(intent)
  ```

- java

  ```java
  Intent intent = new Intent(this, SignInActivity.class);
  startActivity(intent);
  ```

  

您的应用程序可能还希望使用Activity中的数据执行某些操作，例如发送电子邮件，短信或状态更新。在这种情况下，您的应用程序可能没有自己的Activity来执行此类操作，因此您可以利用设备上其他应用程序提供的Activity，这些Activity可以为您执行操作。这是Intent非常有价值的地方：您可以创建描述您要执行的操作的Intent，系统从另一个应用程序启动相应的Activity。如果有多个Activity可以处理意图，那么用户可以选择使用哪个Activity。例如，如果要允许用户发送电子邮件，可以创建以下Intent：

- kotlin

  ```kotlin
  val intent = Intent(Intent.ACTION_SEND).apply {
      putExtra(Intent.EXTRA_EMAIL, recipientArray)
  }
  startActivity(intent)
  ```

- java

  ```java
  Intent intent = new Intent(Intent.ACTION_SEND);
  intent.putExtra(Intent.EXTRA_EMAIL, recipientArray);
  startActivity(intent);
  ```

添加到intent中的EXTRA_EMAIL Extra是电子邮件应发送到的电子邮件地址的字符串数组。当电子邮件应用程序响应此意图时，它会读取extra中提供的字符串数组，并将它们放在电子邮件撰写表单的“to”字段中。在这种情况下，电子邮件应用程序的Activity开始，当用户完成后，您的Activity将恢复。

### startActivityForResult()

有时您希望在Activity结束时从Activity中获取返回结果。例如，您可以启动一项Activity，让用户在联系人列表中选择一个人，当它结束时，它返回被选中的人。为此，请调用startActivityForResult（Intent，int）方法，其中integer参数标识该调用。此标识符用于消除来自同一Activity的多次startActivityForResult（Intent，int）调用之间的歧义。它不是全局标识符，不存在与其他应用程序或Activity冲突的风险。结果通过onActivityResult（int，int，Intent）方法返回。

当子Activity退出时，它可以调用setResult（int）将数据返回到其父级。子Activity必须始终提供结果代码，该结果代码可以是标准结果RESULT_CANCELED，RESULT_OK或从RESULT_FIRST_USER开始的任何自定义值。此外，子Activity可以选择性地返回包含所需的任何其他数据的Intent对象。父Activity使用onActivityResult（int，int，Intent）方法以及父Activity最初提供的整数标识符来接收信息。

如果子Activity因任何原因（例如崩溃）失败，则父Activity将收到代码为RESULT_CANCELED的结果。

- kotlin

  ```kotlin
  class MyActivity : Activity() {
      // ...
  
      override fun onKeyDown(keyCode: Int, event: KeyEvent?): Boolean {
          if (keyCode == KeyEvent.KEYCODE_DPAD_CENTER) {
              // When the user center presses, let them pick a contact.
              startActivityForResult(
                      Intent(Intent.ACTION_PICK,Uri.parse("content://contacts")),
                      PICK_CONTACT_REQUEST)
              return true
          }
          return false
      }
  
      override fun onActivityResult(requestCode: Int, resultCode: Int, intent: Intent?) {
          when (requestCode) {
              PICK_CONTACT_REQUEST ->
                  if (resultCode == RESULT_OK) {
                      startActivity(Intent(Intent.ACTION_VIEW, intent?.data))
                  }
          }
      }
  
      companion object {
          internal val PICK_CONTACT_REQUEST = 0
      }
  }
  ```

  

- java

  ```java
  public class MyActivity extends Activity {
       // ...
  
       static final int PICK_CONTACT_REQUEST = 0;
  
       public boolean onKeyDown(int keyCode, KeyEvent event) {
           if (keyCode == KeyEvent.KEYCODE_DPAD_CENTER) {
               // When the user center presses, let them pick a contact.
               startActivityForResult(
                   new Intent(Intent.ACTION_PICK,
                   new Uri("content://contacts")),
                   PICK_CONTACT_REQUEST);
              return true;
           }
           return false;
       }
  
       protected void onActivityResult(int requestCode, int resultCode,
               Intent data) {
           if (requestCode == PICK_CONTACT_REQUEST) {
               if (resultCode == RESULT_OK) {
                   // A contact was picked.  Here we will just display it
                   // to the user.
                   startActivity(new Intent(Intent.ACTION_VIEW, data));
               }
           }
       }
   }
  ```



### 协调Activity

当一个Activity启动另一个Activity时，它们都会经历生命周期转换，第一个Activity停止运行并进入暂停或停止状态，同时创建另一个Activity。如果这些Activity共享保存到磁盘或其他地方的数据，则必须知道第一个Activity在创建第二个Activity之前未完全停止。相反，启动第二个的过程与停止第一个过程重叠。

生命周期回调的顺序是明确定义的，特别是当两个Activity在同一个进程（app）中而另一个正在启动另一个时。以下是ActivityA启动ActivityB时发生的操作顺序：

1. `A.onPause()` 
2. `B.onCreate()`, `B.onStart()`, 和 `B.onResume()` 被顺序执行（Activity此时获得焦点）
3. A Activity不再可见， `A.onStop()` 被调用

这种可预测的生命周期回调序列允许您管理从一个Activity到另一个Activity的转换信息。

