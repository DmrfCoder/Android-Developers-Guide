# 测试你的Activity

[原文（英文）地址](https://developer.android.com/guide/components/activities/testing)

Activity作为应用程序与用户交互的容器，因此在发生设备级事件时测试Activity的不同行为尤为重要，比如：

- 其他APP（比如设备来电APP）中断了你的Activity
- 设备销毁并重建了你的Activity
- 用户将你的Activity放置到了一个新的窗口环境，比如画中画（picture-in-picture ，PIP ）或者多窗口模式。

尤为重要的一点是保证你的Activity在响应文章 [Activity的生命周期 ](./Activity/Activity的生命周期.md ) 中描述的事件时依然可以表现正常。

这篇文章主要描述如何评估应用程序维护数据完整性的能力以及如何评估良好的用户体验，因为您应用程序的Activity在其生命中的不同阶段会处于不同的状态。

## 驱动（Drive）一个Activity的状态

测试应用程序Activity的一个关键点是将Activity置于特定的状态，请使用ActivityScenario的实例，它是AndroidX Test库的一部分。通过使用此类，您可以将Activity置于模拟本文章开头所描述的设备级事件的状态。

ActivityScenario是一个跨平台的API，您可以在本地单元测试（local unit tests ）和设备集成测试（on-device integration tests）中使用它们。假设你测试程序在运行在线程A，被测试的程序运行在线程B，无论是在真机还是虚拟机上，ActivityScenario都可以保证线程安全即可以在A线程和B线程之间同步事件。该API还特别适用于评估被测试的Activity在销毁或创建时的行为方式。

本节介绍与此API相关的常见用法。

### 创建一个Activity

为了创建一个运行在测试环境下的Activity，你应该添加以下代码：

```kotlin
@RunWith(AndroidJUnit4::class)
class MyTestSuite {
    @Test fun testEvent() {
        val scenario = launchActivity<MyActivity>()
    }
}
```

创建完Activity之后ActivityScenario会将Activity过渡到Resumed状态，该状态表名你的Activity正在运行且对用户可见，在这个状态下，你可以自由地使用 [Espresso UI tests](https://developer.android.com/training/testing/espresso)与你Activity上的View元素进行交互。

另外，你也可以使用ActivityScenarioRule在每一个test之前自动地调用ActivityScenario.launch，在测试结束的时候调用ActivityScenario.close。下面这个例子延时了如何定义一个规则（rule）并且获取scenario 的一个实例：

```kotlin
@RunWith(AndroidJUnit4::class)
class MyTestSuite {
  @get:Rule var activityScenarioRule = activityScenarioRule<MyActivity>()

  @Test fun testEvent() {
    val scenario = activityScenarioRule.scenario
    }
}
```



### 将Activity驱动到一个新的状态

为了将Activity转换到比如Created或者Started这类状态，你应该调用moveToState（）方法。该方法可以模拟您的Activity因为被另一个应用程序或者系统终端而被停止或暂停的情况。

下面是一个moveToState（）的使用示例：

```kotlin
@RunWith(AndroidJUnit4::class)
class MyTestSuite {
    @Test fun testEvent() {
        val scenario = launchActivity<MyActivity>()
        scenario.moveToState(State.CREATED)
    }
}
```

> 警告：如果你尝试将被测试中的Activity转换为其当前的状态（也就是Activity当前的状态时A，你又在此时调用了moveToState（A）），则ActivityScenario将会忽略此请求，而不是抛出异常。

### 重建Activity

如果一个设备资源不足，系统可能会销毁Activity，当用户重新进入你的Activity时将会重建Activity，为了模拟这个场景，你应该调用recreate（）：

```kotlin
@RunWith(AndroidJUnit4::class)
class MyTestSuite {
    @Test fun testEvent() {
        val scenario = launchActivity<MyActivity>()
        scenario.recreate()
    }
}
```

ActivityScenario类会维护已经保存的Activity的实例的状态和使用@NonConfigurationInstance注释的任何对象，这些对象将加载到你正在测试的Activity的心实例中。

### 接收Activity的返回结果

在你运行并且完成一个Activity之后，你可以使用以下代码接收到该Activity的返回结果：

```kotlin
@RunWith(AndroidJUnit4::class)
class MyTestSuite {
    @Test fun testResult() {
        val scenario = launchActivity<MyActivity>()
        onView(withId(R.id.finish)).perform(click())
        val resultCode = scenario.result.resultCode
        val resultData = scenario.result.resultData
    }
}
```



### 触发Activity中的操作

ActivityScenario中的所有方法都是阻塞调用，因此API需要您在测试线程中运行他们。

为了触发你被测试Activity中的操作，使用Espresso视图匹配器与视图中的元素进行交互：

```kotlin
@RunWith(AndroidJUnit4::class)
class MyTestSuite {
    @Test fun testEvent() {
        val scenario = launchActivity<MyActivity>()
        onView(withId(R.id.refresh)).perform(click())
    }
}
```

如果需要调用Activity本身的方法，你可以通过实现ActivityAction安全地达到这个目的：

```kotlin
@RunWith(AndroidJUnit4::class)
class MyTestSuite {
    @Test fun testEvent() {
        val scenario = launchActivity<MyActivity>()
        scenario.onActivity { activity ->
          activity.handleSwipeToRefresh()
        }
    }
}
```

> 注意：在测试类中，不要保留传递给onActivity（）的对象的引用。因为这些引用会消耗系统资源，并且引用本身可能是陈旧的（失效的），因为框架可以重新创建传递给回调方法的Activity。

想要了解更多关于线程如何在Android test中工作的内容，参考 [Understand threads in tests](https://developer.android.com/training/testing/fundamentals#threads).



