# Activity的状态变化

[原文（英文）地址](https://developer.android.com/guide/components/activities/state-changes)

不同的事件(一些是用户触发的，一些是系统触发的)，可以导致Activity从一个状态转换到另一个状态。本节描述了发生此类转换的一些常见情况，以及如何处理这些转换。

有关Activity状态的更多信息，请参阅[理解Activity的生命周期](.Activity/Activity的生命周期.md)。要了解ViewModel类如何帮助您管理活动生命周期，请参阅[Understand the ViewModel Class](https://developer.android.com/topic/libraries/architecture/viewmodel.html).

## 发生配置更改（Configuration change occurs）

有很多事件的发生会导致配置被更改，最常见的例子就比如屏幕在横屏和竖屏之间切换，还有其他的一些事件比如语言发生改变或者输入设备发生改变等也会导致配置被更改。

当配置被更改时，Activity会被销毁并且重建，原来（旧）的Activity实例将会触发onPause（）、onStop（）、onDestroy（）的调用，而新Activity实例将会被创建并且调用onCreate（）、onStart（）以及onResume（）回调方法。

结合使用ViewModels、onSaveInstanceState（）方法，以及/或本地持久化存储可以跨配置（across configuration changes.）保留Activity的UI状态。你应该根据你UI数据的复杂性、检索速度、内存的占用情况去决定如何结合这些可选的方法去实现你Activity的UI持久化。想要了解更多的信息请参阅 [Saving UI States](https://developer.android.com/topic/libraries/architecture/saving-states.html).

### 处理多窗口的情况

当一个app处于多窗口模式时（该模式可存在于系统版本高于等于Android7.0即ApI为24的设备上），如果配置发生改变，系统会通知当前正在运行的Activity，这样就可以执行上述配置发生改变时Activity的不同生命周期。不同于普通模式，处于多窗口模式的应用调整窗口大小后也会发生配置的更改，你可以在你的Activity中自行处理配置更改的情况，也可以允许系统销毁原有Activity后重建Activity。

关于更多的Activity在多窗口模式下的生命周期，请参阅Multi-Window Support](https://developer.android.com/guide/topics/ui/multi-window.html) 页面中的 [Multi-Window Lifecycle](https://developer.android.com/guide/topics/ui/multi-window.html#lifecycle)节。

在多窗口模式下，尽管有两个app同时对用户可见，但是在同一时间用户只会关注一个前台的app而且只有该一个app占有焦点，该Activity处于Resumed状态，而其他的处于前台但是没有占有焦点的Activity处于Paused状态。

当用户从app A切换到app B，系统会调用app A中当前（前台）Activity的onPause（）方法，同时调用app B中当前（前台）Activity的onResume（）方法。每次用户切换app时都会分别调用两个app中前台Activity的onPause（）和onResume（）方法。

关于更多多窗口模式的内容，请参阅[Multi-Window Support](https://developer.android.com/guide/topics/ui/multi-window.html).

## Activity或者Dialog出现在前台

如果一个新的Activity或者Dialog出现在前台并且获取焦点，同时部分地覆盖了之前的Activity，那么那个被部分覆盖的Activity会失去焦点同时进入Paused 状态，接着系统会调用该Activity的onPause（）方法。

当被部分覆盖的Activity重新回到前台并获取到焦点时，系统会调用他的onResume（）方法。

如果一个新的Activity或者Dialog出现在前台并获取到焦点，同时完全覆盖了之前的Activity，那么系统会快速连续地调用那个被完全覆盖的Activity的onPause（）和onStop（）方法。

当之前那个被完全覆盖的Activity 的实例重新回到前台时，系统会调用该Activity的OnRestart（）、onStart（）以及onResume（）方法。

如果是之前被覆盖的Activity的新的实例从后台切换到前台，则只会调用它的onStart（）和onResume（）方法，不会调用onRestart（）。

> 注意：当用户按了home键后，系统所做对之前处于前台的Activity所做的操作和该Activity被完全覆盖时所做的操作一样。

## 用户点击了Back按键

如果一个Activity处于前台并且用户点击了Back按键，系统会调用该Activity的onPause（）、onStop（）、onDestroy（）方法，当Activity被Destroy后，该Activity也会被从回退栈中移除。

必须记住的很重要的一点是默认情况下onSateInstanceState（）方法在此种情况下不会被调用。This behavior is based on the assumption that the user tapped the **Back** button with no expectation of returning to the same instance of the activity. 

然而，你可以重写onBackPressed（）方法去实现一些自己的需求，比如弹出“确认退出”提示框。

在你重写了onBackPressed（）方法的同时我们强烈建议你在你重写的方法体中调用super.onBackPressed()方法，否则后悔按钮可能会刺激到？（ jarring to）用户。

## 系统杀死了APP进程

如果系统为了处于前台的app需要清理内存，那么处于后台的app可以被杀死以释放更多的内存。想要了解更多关于系统是如何决定是否杀死一个app的，可以参阅 [Activity state and ejection from memory](https://developer.android.com/guide/components/activities/activity-lifecycle.html#asem) 和 [Processes and Application Lifecycle](https://developer.android.com/guide/components/activities/process-lifecycle.html).

想要了解更多如何在系统杀死你的app的同时保存你Activity的UI状态，请参阅[Saving and restoring activity state](https://developer.android.com/guide/components/activities/activity-lifecycle.html#saras).

