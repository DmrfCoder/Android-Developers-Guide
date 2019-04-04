# Activity简介（介绍）

[原文（英文）地址](https://developer.android.com/guide/components/activities/intro-activities)

Activity类是Android应用程序的重要组成部分，Activity的启动和组合方式是应用程序的基本组成部分。与使用main（）方法启动应用程序的编程范例不同，Android系统通过调用与其生命周期的特定阶段相对应的特定回调方法来启动Activity实例中的代码。

本文档介绍了Activity的概念，然后提供了有关如何使用它们的一些轻量级指导。有关构建应用程序的最佳实践的其他信息，请参阅 [Guide to App Architecture](https://developer.android.com/topic/libraries/architecture/guide.html)。

## Activity的概念

移动应用与桌面应用不同，用户与移动应用的交互并不总是在同一个地方开始。相反，用户打开同一个应用的目的通常是非确定性的。例如，如果您从主屏幕打开电子邮件应用程序，则可能会看到电子邮件列表。但是，如果您在使用第三方社交媒体应用程序时从该应用内的入口启动您的电子邮件应用程序，您可能会直接进入电子邮件应用程序的屏幕撰写电子邮件。

Activity类旨在促进（方便化）这种应用场景。当一个应用程序调用另一个应用程序时，调用应用程序将调用另一个应用程序中的Activity，而不是一个整体的应用程序。通过这种方式，Activity可以作为应用与用户交互的入口点。您通常需要将Activity类作为父类去实现您自己的Activity（`MYActivity extends Activity`）。

Activity提供应用程序绘制其UI的Window。此Window通常填充屏幕，但可能小于屏幕并浮动在其他窗口的顶部。通常，一个Activity在应用程序中实现一个Window。例如，应用程序的某个Activity可能会implement（实现）一个*Preferences* （首选项）screen，而另一个会implement（实现）  *Select Photo* （选择照片）screen。

大多数应用程序包含多个页面（screens），这意味着它们通常包含多个Activity，应用程序中的一个Activity被指定为MainActivity，这是用户启动应用程序时显示的第一页面。然后，每个Activity可以启动另一个Activity以执行不同的操作。例如，电子邮件应用程序中的MainActivity可能会提供显示电子邮件收件箱的界面，从那个页面开始，MainActivity可能会启动其他Activity，去展示编写电子邮件和打开个人电子邮件的界面。

虽然在一个应用中一般会有多个Activity紧密地在一起工作，但每个Activity仅与其他Activity轻耦合（依赖性不大），一个应用程序中不同的Activity之间通常存在很小的依赖关系。实际上，Activity通常会启动属于其他应用程序的Activity。例如，浏览器应用可能会启动社交媒体应用的共享。

要在应用程序中使用Activity，您必须在应用程序的manifest文件中注册有关它们的信息，并且必须合理地管理Activity的生命周期。本文档的其余部分介绍了这些主题。

## 配置manifest文件

为了使您的应用能够使用Activity，您必须在manifest文件中声明Activity及其某些属性。

### 声明Activity

打开你应用的manifest文件，在\<application\>…<application\>中间添加<activity\>元素，比如：

```xml
<manifest ... >
  <application ... >
      <activity android:name=".ExampleActivity" />
      ...
  </application ... >
  ...
</manifest >
```

对于该元素唯一必须的属性是android:name，该属性唯一确定了一个Activity的类名。同样，你也可以在manifest中为该activity添加其他属性，比如label、icon或者UI theme等。想要了解更多activity的其他属性，可以参考[ the element reference documentation](https://developer.android.com/guide/topics/manifest/activity-element.html) .

> 注意：当你发布了你的APP之后，你就不应该再更改你Activity的名字（name），如果你在发布APP之后更改了某个Activity的name属性，你app的某些功能可能被破坏，比如你应用的快捷启动方式（app shortcuts），有关发布APP后应该禁止更改的属性，可以参见 [Things That Cannot Change](http://android-developers.blogspot.com/2011/06/things-that-cannot-change.html)

### 声明intent filter

Intent filter是Android平台一个非常强大的功能。其可以基于显式请求和隐式请求来启动Activity。例如，显式请求可能会告诉系统“在Gmail应用中启动发送电子邮件的Activity”。相反，隐式请求会告诉系统“在任何可以执行此任务的Activity中启动发送电子邮件的界面”。当系统UI询问用户在执行任务时使用哪个应用程序时，这是一个工作中的intent filter。

您可以通过在<activity\>元素中声明<intent-filter\>属性来利用此功能。该元素的定义包括<action\>元素，以及可选的<category\>元素和/或<data\>元素。这些元素组合在一起以指定您的Activity可以响应的Intent类型。例如，以下代码段显示了如何配置发送文本数据的Activity，并从其他Activity接收请求以执行此操作：

```xml
<activity android:name=".ExampleActivity" android:icon="@drawable/app_icon">
    <intent-filter>
        <action android:name="android.intent.action.SEND" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="text/plain" />
    </intent-filter>
</activity>
```

在此例中，<action\>元素指定此Activity可以发送数据。将<category\>元素声明为DEFAULT可使Activity接收启动的请求。 <data\>元素指定此Activity可以发送的数据类型。以下代码段展示了如何调用上述Activity：

- kotlin

```kotlin
val sendIntent = Intent().apply {
    action = Intent.ACTION_SEND
    type = "text/plain"
    putExtra(Intent.EXTRA_TEXT, textMessage)
}
startActivity(sendIntent)
```

- Java

```java
// Create the text message with a string
Intent sendIntent = new Intent();
sendIntent.setAction(Intent.ACTION_SEND);
sendIntent.setType("text/plain");
sendIntent.putExtra(Intent.EXTRA_TEXT, textMessage);
// Start the activity
startActivity(sendIntent);
```

如果您想使某一个Activity只能在本应用内启动（不需要其他应用启动该Activity），则不需要任何其他intent filter。您不希望其他应用程序可见的Activity应该没有intent filter，您可以使用显式意图自行启动它们。有关Activity如何响应Intent的更多内容，请参阅[Intents and Intent Filters](https://developer.android.com/guide/components/intents-filters.html)。

### 声明权限

您可以使manifest中的<activity\>元素（element）来控制哪些应用可以启动特定Activity。除非两个Activity在其manifest文件中具有相同的权限（permissions），否则父Activity无法启动子Activity。如果为特定Activity声明<uses-permission\>元素（element），则调用其的Activity必须具有匹配的<uses-permission\>元素(element)。

例如，加入您的应用想要通过名为SocialApp的应用中的Activity在社交媒体上分享帖子，SocialApp中的Activity本身必须定义调用它的应用必须具有的权限：

```xml
<manifest>
<activity android:name="...."
   android:permission=”com.google.socialapp.permission.SHARE_POST”

/>
```

同时，为了被允许调用SocialAPP，你自己的APP必须匹配SocialAPP的manifest文件中声明的权限：

```xml
<manifest>
   <uses-permission android:name="com.google.socialapp.permission.SHARE_POST" />
</manifest>
```

有关权限和安全性的更多信息请参考：[Security and Permissions](https://developer.android.com/guide/topics/security/security.html)。

## 管理Activity的生命周期

一个Activity从创建到销毁经历和很多不同的周期，你可以使用一系列的回调方法去控制不同生命周期应该执行的逻辑代码，接下来的一节介绍这些回调方法。

### onCreate（）

您必须实现此回调，该回调在系统创建Activity时触发。您的实现应初始化Activity中的基本组件：例如，您的应用程序应创建视图并将数据绑定到列表。最重要的是，您必须调用setContentView（）来定义Activity用户界面的布局（xml布局文件）。

当onCreate（）完成时，进入Created状态，下一个回调一定是onStart（）。

### onStart（）

当onCreate（）执行完成后，会调用onStart（），此回调主要包含Activity最终准备到达前台并变为交互式所需执行的逻辑。

执行完onStart（）之后Activity进入Started状态，Activity对用户可见，接着会调用onResume（）方法。

### onResume（）

系统在Activity开始与用户交互之前回调此方法。此时，Activity位于Activity Stack（堆栈）的顶部，并捕获（captures）所有用户输入。应用程序的大多数核心功能都是在onResume（）方法中实现的。

onResume（）之后一般会调用onPause（）方法。

### onPause（）

当Activity失去焦点并进入暂停状态时，系统回调onPause（）方法。例如，当用户点击“Back（后退键）”或“Recents（最近任务）”按钮时，会出现此状态。当系统回调您的Activity的onPause（）方法时，它在意味着您的Activity仍然部分可见，但通常表示用户正在离开Activity，并且Activity很快将进入“已停止（Stopped）”或“已恢复（Resumed）”状态。

如果用户期望UI继续更新，则处于暂停状态的Activity可以继续更新UI。这种场景的Activity包括导航地图的Activity或媒体播放器播放的Activity。即使这些Activity失去焦点，用户也希望他们的UI继续更新。

您不应使用onPause（）来保存应用程序或用户数据、进行网络调用或执行数据库事务。有关保存数据内容，请参阅 [Saving and restoring activity state](https://developer.android.com/guide/components/activities/activity-lifecycle.html#saras).

一旦onPause（）完成执行，下一个回调就是onStop（）或onResume（），具体取决于进入Paused状态后发生的情况。

### onStop（）

当Activity不再对用户可见时，系统会回调onStop（）。这中情况一般是因为旧Activity正在被销毁（being destroyed），新Activity正在启动（Starting），或者现有Activity正在进入恢复状态（Resumed state）并且正在覆盖已停止的Activity。在这些场景下，已经停止的Activity不再可见。

之后如果Activity返回与用户交互（比如按了最近任务键之后又点击了之前的Activity），系统调用的下一个回调是onRestart（），如果此Activity完全终止，则会回调onDestroy（）。

### onRestart（）

当一个处于Stopped状态的Activity被重新启动（start）时系统会回调onRestart（）方法。onRestart（）方法会恢复之前Activity停止（stop）时保存的状态。

onRestart（）方法执行完成之后一般会调用onStart（）方法。

### onDestroy（）

系统在销毁Activity之前调用此方法。

此回调是Activity的最后一个回调。通常实现onDestroy（）以确保在Activity或包含该Activity的进程被销毁时释放所有Activity占用的资源。

有关Activity生命周期及其回调的更详细的内容，请阅读 [Activity的生命周期](./Activity/Activity的生命周期.md)。

