# 理解Tasks和回退栈

[原文（英文）地址](https://developer.android.com/guide/components/activities/tasks-and-back-stack)

Task是用户在执行特定任务时与之交互的所有Activity的集合。Activity按堆栈排列 （回退堆栈 ）—— 按打开Activity的顺序排列。例如，电子邮件应用可能有一个Activity来显示新消息列表，当用户选择消息时，将打开一个新Activity以查看该消息，此新Activity将添加到后台堆栈中，如果用户按下“返回”按钮，则该新Activity将完成（finished）并从堆栈中弹出。以下[视频](https://youtu.be/MvIlVsXxXmY)概述堆栈背后的工作原理。

当多个应用程序在多窗口模式（在Android 7.0，API级别24及更高版本中受支持）中同时运行时，系统会为每个窗口单独管理Task，每个窗口可能有多个Task。对于在Chromebook上运行的Android应用程序也是如此：系统基于每个窗口管理Task或groups of tasks（task组）。

设备主屏幕是大多数Task的起始位置，当用户触摸应用程序启动器中的图标（或主屏幕上的快捷方式）时，该应用程序的Task将进入前台，如果应用程序不存在Task（最近未使用该应用程序），则会创建一个新Task，该应用程序的“主”Activity将作为堆栈中的根Activity打开。

当当前Activity开始另一个Activity时，新Activity将被推（push）到堆栈顶部并获得焦点，之前的Activity仍在堆栈中，但已停止。当Activity停止时，系统将保留其用户界面的当前状态，当用户按下“返回”按钮时，当前Activity将从堆栈顶部弹出（Activity被销毁），之前的Activity将恢复（其UI的先前状态将恢复）。堆栈中的Activity永远不会重新排列，只能在当前Activity启动时从堆栈中推送并弹出到堆栈中，并在用户使用“返回”按钮离开时弹出。因此，回退栈是一种“后进先出”的对象结构。图一显示了这种动作，时间轴显示了Activity之间的进度以及每个时间点的当前回退栈：

![diagram_backstack](https://ws1.sinaimg.cn/large/006tNc79gy1g1qtje6t6dj30h505f0t2.jpg)

<center>图一：说明Task中每个新的Activity如何被添加到后台堆栈中以及当用户按下“返回”按钮时会销毁当前Activity并恢复之前的Activity</center>

如果用户继续点击“返回”按键，堆栈中的Activity将会依次出栈（pop）以显示前一个Activity，知道用户返回到了主界面（或者任务栈在开始时运行的Activity）。当所有的Activity都被从堆栈中移除，该Task也就不再存在。

Task是一个内聚单元（cohesive unit），当用户开始新Task或通过Home按钮进入主屏幕时，之前的Task可以移动到“后台”。在后台，Task中的所有Activity都会停止，但Task的后台堆栈中仍然完好无损 ——当前Task在被新Task替代后就就失去了焦点，如图2所示。

![diagram_multitasking](https://ws3.sinaimg.cn/large/006tNc79gy1g1qtocxkrcj307p044q2z.jpg)

<center> 图二：两个Task，Task B在前台与用户进行交互，Task A在后台等待被恢复</center>

然后Task可以返回到“前台“，所以可以从被中断的地方继续与进行用户进行交互。例如，假设当前Task（TaskA），在TaskA堆栈中有三个Activity（在当前Activity下有两个Activity），此时用户按下Home按钮，然后从应用启动器启动新应用，显示主屏幕时，TaskA进入后台，当新应用程序启动时，系统会为该应用程序的一系列Activity启动新的Task（TaskB），在与该应用程序交互完成之后，用户再次返回Home并选择最初启动TaskA的应用程序，现在，TaskA进入前台，并且其堆栈中的所有三个Activity都是完整的，堆栈顶部的Activity将恢复。此时，用户还可以通过Back键返回主页并选择启动TaskB对应的应用程序图标（或按“Recent”按钮后从最近任务中选择TaskB对应的应用程序）切换回TaskB。这是Android上处理多Task的一个示例。

> 注意：可以同时在后台同时保存多个任务。但是，如果用户同时运行许多后台任务，系统可能会销毁后台Activity以回收内存，从而导致Activity状态丢失。



由于后台堆栈中的Activity永远不会重新排列，如果您的应用程序允许用户从多个Activity启动特定Activity，则会创建该Activity的新实例并将其推送到堆栈（而不是将任何先前的Activity实例调整到顶部）。因此，应用程序中的一个Activity可能会被多次实例化（甚至来自不同的Task），如图3所示。因此，如果用户使用“后退”按钮向后导航，则Activity的每个实例都按顺序显示并被打开（每个都有自己的UI状态）。但是，如果您不希望多次实例化Activity，则可以修改此行为。有关如何执行此操作将在下一节”管理Task“中讨论。

![diagram_multiple_instances](https://ws3.sinaimg.cn/large/006tNc79gy1g1qtyqko75j305p05eaa2.jpg)

<center>图三：一个Activity被多次实例化</center>

总结一下Activity和Task的默认操作：

- 当ActivityA启动ActivityB时，ActivityA停止，但系统保留其状态（例如滚动位置和输入到表单中的文本）。如果用户在ActivityB中按下“返回”按钮，则ActivityA将恢复其状态。
- 当用户通过按Home键离开任务时，当前Activity将停止，其Task将进入后台。系统保留task中每个Activity的状态。如果用户稍后通过选择开始任务的启动器图标来恢复任务，则任务将到达前台并在堆栈顶部恢复Activity。
- 如果用户按下“返回”按钮，则会从堆栈中弹出当前Activity并将其销毁，堆栈中的之前的Activity会被恢复，当Activity被销毁时，系统不会保留Activity的状态。
- Activity可以被多次实例化，甚至可以从其他Task实例化。

> 导航设计:有关Android导航如何在Android上运行的内容，请参阅Android Design's [Navigation](https://developer.android.com/design/patterns/navigation.html) guide.

## 管理Tasks

如上所述Android管理Task和后台堆栈的方式：通过将所有Activity连续启动到同一Task和“后进先出”堆栈 ，该方式适用于大多数应用程序，你不必关于您的Activity如何与Task相关联或它们如何存在于后台堆栈中。但是，您可能有需求改变这种默认的的Task操作方式，比如也许您希望应用程序中的Activity在启动时开始新Task（而不是放在当前Task中），或者，当你开始一个Activity时，你想要复用它已有的实例（而不是在堆栈顶部创建一个新的实例），或者，您希望在用户离开Task时清除除了根Activity之外的所有Activity的回退堆栈。

您可以使用<activity\>清单元素中的属性以及传递给startActivity（）的intent中的标记来设置上述操作。

在这方面，您可以使用的主要<activity\>属性有：

- [`taskAffinity`](https://developer.android.com/guide/topics/manifest/activity-element.html#aff)
- [`launchMode`](https://developer.android.com/guide/topics/manifest/activity-element.html#lmode)
- [`allowTaskReparenting`](https://developer.android.com/guide/topics/manifest/activity-element.html#reparent)
- [`clearTaskOnLaunch`](https://developer.android.com/guide/topics/manifest/activity-element.html#clear)
- [`alwaysRetainTaskState`](https://developer.android.com/guide/topics/manifest/activity-element.html#always)
- [`finishOnTaskLaunch`](https://developer.android.com/guide/topics/manifest/activity-element.html#finish)

你主要可以使用的Intent flag有：

- `FLAG_ACTIVITY_NEW_TASK`
- `FLAG_ACTIVITY_CLEAR_TOP`
- `FLAG_ACTIVITY_SINGLE_TOP`

在以下部分，您将了解如何使用这些清单属性和Intent标志来定义Activity与Task的关联方式以及它们在后台堆栈中的行为方式。

另外，会单独讨论如何在“最近”屏幕中表示和管理Task和Activity的注意事项。有关详细信息，请参阅[Recents Screen](https://developer.android.com/guide/components/recents.html) 。通常，您应该允许系统在“最近”屏幕中定义您的Task和Activity的表示方式，而不需要修改此行为。

> 警告：大多数应用程序不应该更改Activity和Task的默认行为。如果您确定您的Activity需要修改默认的行为，请谨慎操作并确保在启动期间使用“返回”按钮以及从其他Activity和Task导航回Activity时测试Activity的可用性。请务必测试可能与用户预期行为冲突的导航行为。

### 指定运行模式

运行模式允许你设置一个新的Activity实例如何与当前Task建立关联，你可以通过两种方式设置不同的运行模式：

- 通过manifest文件

  > 当你在manifest中注册了你的Activity，你可以设定该Activity的默认启动模式

- 使用Intent flags

  > 当你调用startActivitu（）时你可以在Intent中设置flag来指定新的Activity如何与当前Task建立关联

因此，如果Activity A启动Activity B，Activity B可以在其清单中定义它应该如何与当前Task相关联，Activity A也可以请求Activity B应该如何与当前Task相关联。如果两个Activity都定义了Activity B应该如何与Task相关联，则Activity A的发起的请求（如意图中所定义）将向Activity B定义的方式妥协（即如以Activity B的清单中所定义的方式为执行标准）。

> 注意：manifest文件可用的某些启动模式不可用作intent的标志，同样，某些启用模式可用作intent的标志，无法在manifest中定义。

#### 使用manifest文件

通过在manifest文件中指定<activity\>元素的lanchMode属性，你可以指定该Activity与Task建立关联的方式。

launchMode属性指定了对应Activity以何种方式运行进Task中。你可以为launchMode指定以下四种启动模式：

- ”standard“（默认的启动模式）

  > 默认情况下就是这个启动模式，当该Activity被启动或者一个Intent想要启动它时系统会新建一个该Activity的实例。该Activity可以被多次实例化，内一个实例可以属于不同的Task，同时一个Task也可以有多个该Activity的实例。

- ”singleTop“

  > 如果需要导航到某个Activity而当前Task的栈顶广告存在一个该Activity的实例，那么系统会通过调用onNewIntent()方法将Intent导航到该Activity 实例而不是去创建一个新的Activity实例。处于这种启动模式的Activity可以被实例化多次，每一个实例可以属于不同的Task，并且每一个Task也可以拥有多个实例，但是仅限于回退栈的栈顶的不是该Activity的实例。
  >
  > 比如，一个以Activity A为栈底，栈中拥有B、C，栈顶为D的回退栈(栈中Activity实例的顺序是A-B-C-D，D是栈顶)，当一份Intent想要导航到D，如果D的启动模式是默认的"Standard"，那么一个新的Activity D将会被实例化，回退栈也将变为A-B-C-D-D。然而，如果D Activity的启动模式时"singleTop"，那么已经存在在栈顶的D Activity实例将会通过被调用onNewIntent（）收到一个Intent意图而不会新实例化一个D Activity并将其push到栈顶，因为当前栈顶是D Activity。但是如果当前的Intent的目标是B Activity，那么系统仍然会实例化一个新的B Activity，及时B Activity的启动模式时"singleTop"(因为B Activity此时不在栈顶)。

  > 注意：当一个新的Activity实例被创建后，用户可以点击Back按钮去回退到他之前的Activity，但是当一个Activity实例正在处理一个新的Intent意图时，在新的Intent到达onNewIntent()之前用户无法通过按"Back"按键返回到之前的状态。

- ”singleTask“

  > 系统将会创建一个新的Task和一个新的Activity实例，并将该实例放置于搞Task对应栈的栈底。但是，如果一个Activity的实例已经存在于之前的Task栈，系统会通过调用已经存在的Activity实例的onNewIntent()方法将Intent传递到该Activity 实例中而不会去创建新的Activity实例，在同一时间内，只能存在一个该Activity的实例。

  > 注意：尽管该Activity的实例会启动在一个新的task中，但是用户按了Back按钮后依然可以返回到用户的前一个Activity。

  

- ”singleInstance“

  > 与"singleTask"类似，但是系统不会再存在该模式下Activity 实例的Task中再防止任何其他Activity的实例，该Activity对应的实例是对应Task的唯一元素，任何该模式下Activity的实例都会在一个单独的Task中启动。

另一个例子，Android浏览器应用程序声明Web浏览器的Activity应始终在其自己的Task中打开（通过在<activity\>元素中指定singleTask启动模式）。这意味着，如果您的应用发出打开Android浏览器的Intent，则其Activity不会与您的应用中的Activity放在同一Task中。相反，要么为浏览器启动新Task，要么如果浏览器已经在后台运行Task，则该Task将被拿到前台处理新Intent。

无论Activity是在新Task中启动还是在与启动它的Activity相同的Task中启动，“Back”按钮始终会将用户带到上一个Activity。但是，对于启动指定singleTask启动模式的Activity，如果后台Task中存在该Activity的实例，则会将后台的Task带到前台，此时，后台堆栈会是之前前台对应Task的堆栈。图4说明了这种情况：

![diagram_backstack_singletask_multiactivity](https://ws4.sinaimg.cn/large/006tNc79gy1g1rs8bq730j30fa08l0tc.jpg)

<center>图四：从当前Task启动具有"singleTask"启动模式的Activity，如果该Activity已经是后台Task的一部分，那么整个后台堆栈将在当前Task之上出现。</center>

想要了解更多关于启动模式的内容，请参阅[<activity\>](https://developer.android.com/guide/topics/manifest/activity-element.html)元素的文档，该文档中讨论了更多关于启动模式和其他启动属性的内容。

> 注意：你在manifest中通过制定launchMode属性的方法指定Activity启动模式的目的也可以通过设置启动Intent的flag标志做到，下节将具体介绍。



#### 使用Intent flag

在启动Activity时，你可以通过设置传递给startActivity()中Intent的flag标志来达到改变该Activity与Task建立关联的方式，具体的有下面几种标志(flags)：

- [FLAG_ACTIVITY_NEW_TASK](https://developer.android.com/reference/android/content/Intent.html#FLAG_ACTIVITY_NEW_TASK)

  > 在一个新的Task中启动Activity。如果你当前要启动的Activity已经有实例运行在已经存在的Task中，则对应的Task将会被带到前台，并且已经存在的Activity实例的onNewIntent()方法会被调用。
  >
  > 这个标志的作用和上节中讨论的"singleTask"启动模式的作用一样。

- [FLAG_ACTIVITY_SINGLE_TOP](https://developer.android.com/reference/android/content/Intent.html#FLAG_ACTIVITY_SINGLE_TOP)

  > 如果被启动的Activity已经有一个实例在Task对应堆栈的栈顶，则已经存在的栈顶Activity实例会被调用其onNewIntent()方法而不是重新创建新的Activity实例。
  >
  > 这个标志和上节中讨论的"singleTop"启动模式的作用一样。

- [FLAG_ACTIVITY_CLEAR_TOP](https://developer.android.com/reference/android/content/Intent.html#FLAG_ACTIVITY_CLEAR_TOP)

  > 如果要启动的Activity已经有实例A运行在当前的Task中，则堆栈中运行在A上面的所有Activity的实例将会被销毁（destroyed）这样A就会出现在栈顶，同时其onNewIntent（）方法也将会被调用。
  >
  > 这个flag标志没有对应的lunchMode属性值与其对应。
  >
  > `FLAG_ACTIVITY_CLEAR_TOP` 标志经常与FLAG_ACTIVITY_NEW_TASK`标志一起使用，当同时使用这两个标志时，可以定位到另一个Task中存在的现有Activity实例 。

  > 注意：如果一个Activity的launchMode是"standard"，那么即使使用本flag，系统还是会新建Activity是实例，而不是clear top。



### 处理affinity

affinity属性指明了Activity属于哪个Task。默认情况下，同一应用程序中的所有Activity都具有同一个affinity。因此，默认情况下，同一个应用中的所有Activity都会处于同一Task中。但是，您可以修改Activity的默认affinity属性，在不同应用中定义的Activity可以使用同一个affinity，或者可以为同一应用中的Activity分配不同的Task affinity。

您可以使用<activity\>元素的taskAffinity属性修改任何Activity的affinity。

taskAffinity属性采用字符串值，该值必须与<manifest\>元素中声明的默认包名唯一（which must be unique from the default package name declared in the [`<manifest>` ](https://developer.android.com/guide/topics/manifest/manifest-element.html)element,），因为系统使用该名称来标识应用程序的默认Task affinity。

affinity在两种情况下起作用：

- 启动Activity的intent包含FLAG_ACTIVITY_NEW_TASK标志。
  默认情况下，新Activity将启动在调用startActivity（）的Activity的Task中，即它被push到与调用者相同的后台栈中。但是，如果传递给startActivity（）的intent包含FLAG_ACTIVITY_NEW_TASK标志，则系统会查找另一个Task以容纳新Activity。如果已存在与新Activity具有相同affinity的Task，则会将Activity启动到该Task中，如果没有，它才新建一个新Task。

  如果此标志导致Activity开始在了新的Task中并且用户按下Home按钮以离开它，则必须有某种方式让用户导航回Task。某些实体（例如通知管理器）总是在外部Task中启动Activity，从不将其启动到自己的Task中，因此它们总是将FLAG_ACTIVITY_NEW_TASK置于它们传递给startActivity（）的Intent中。如果您的Activity有可能被使用此标志的外部实体调用，请注意用户有一种独立的方式来返回已启动的Task，例如使用启动图标（launcher icon），因为Task的根Activity有一个CATEGORY_LAUNCHER intent filter，请参阅下面的“启动Task”部分。

- 当一个Activity的allowTaskReparenting属性设置为“true”时。
  在这种情况下，当与Activity具有想用affinity属性的Task到达前台时，Activity可以从它开始的Task移动到它具有affinity属性的Task。

  例如，假设报告天气状况的Activity被定义为旅行应用程序的一部分，它与同一应用程序中的其他Activity（默认的应用程序关联）具有相同的affinity，并允许使用此属性re-parenting 。当您的某个Activity启动报告天气状况的Activity时，它最初属于与您的Activity相同的Task（Task A）。但是，当旅行应用程序的Task（Task B）到达前台时，天气报告者Activity将重新分配给该Task（Task B）并显示在其中。

>  提示：如果APK文件从用户的角度包含多个“app”，您可能希望使用taskAffinity属性为与每个“app”关联的Activity分配不同的affinity。

### <span id="Clearing-the-back-stack">清除回退栈</span>

如果用户长时间离开Task，系统将清除Task的除了根Activity之外的所有Activity。当用户再次返回Task时，仅恢复根Activity。系统以这种方式运行，因为在很长一段时间之后，用户可能已经放弃了之前正在做的事情，返回Task只是为了做新的事情。

您可以使用一些Activity属性来修改此行为：

- alwaysRetainTaskState
  如果在Task的根Activity中将此属性设置为“true”，则不会发生刚刚描述的默认行为。即使经过很长一段时间，Task仍会保留堆栈中的所有Activity。
- clearTaskOnLaunch
  如果在Task的根Activity中将此属性设置为“true”，则只要用户离开Task，就会将堆栈清除为仅剩根Activity。换句话说，它与alwaysRetainTaskState相反。即使在离开Task片刻之后，用户也始终返回初始状态的Task。
- finishOnTaskLaunch
  此属性类似于clearTaskOnLaunch，但它在单个Activity上起作用，而不是在整个Task上起作用。它可以导致任何Activity消失，包括根Activity。当它设置为“true”时，Activity仍然只是当前会话的Task的一部分。如果用户离开然后返回Task，则它不再存在（即设置了谁，谁就被清除）。

### 开启一个task

你可以将某一个Activity设置为Task的入口点，方法是为其指定intent filter，其中“android.intent.action.MAIN”指定操作，“android.intent.category.LAUNCHER”指定类别，比如：

```xml
<activity ... >
    <intent-filter ... >
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
    ...
</activity>
```

这种intent filter会导致Activity的icon和label显示在应用程序启动器中，从而为用户提供启动Activity的方法，并在启动后随时返回创建的Task。

第二种作用很重要：用户离开Task后，必须可以使用此Activity启动器返回该Task。因此，仅当Activity具有ACTION_MAIN和CATEGORY_LAUNCHER属性时，才能够将Activity标记为始终启动新Task的两种启动模式：“singleTask”和“singleInstance”。想象一下，例如，如果intent filter丢失会发生什么：一个intent启动一个“singleTask” Activity，启动一个新Task，并且用户花费一些时间在该Task中工作。然后用户按下主页按钮。该Task现在被调整到后台并且不可见。现在用户无法返回Task，因为它未在应用启动器中显示。

对于您不希望用户能够返回Activity的情况，请将<activity\>元素的finishOnTaskLaunch设置为“true”（请参阅[清除回退栈](#Clearing-the-back-stack)）。

有关如何在“**Overview**”屏幕中表示和管理Task和Activity的更多信息，请参阅 [Recents Screen](https://developer.android.com/guide/components/activities/recents.html).

## 更多资源

- [Android Design: Navigation](https://developer.android.com/design/patterns/navigation.html)
- [`` manifest element](https://developer.android.com/guide/topics/manifest/activity-element.html)
- [Recents Screen](https://developer.android.com/guide/components/recents.html)
- [Multitasking the Android Way](http://android-developers.blogspot.com/2010/04/multitasking-android-way.html)