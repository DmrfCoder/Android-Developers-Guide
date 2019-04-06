# 测试Fragment

[原文(英文)地址](https://developer.android.com/training/basics/fragments/testing#drive-fragment-new-state)

Fragment在您的应用程序中充当可重用容器，允许您在各种Activity和布局配置中呈现相同的用户界面布局。鉴于这些Fragment的多功能性，测试它们提供一致且资源有效的体验非常重要：

- 您的Fragment外观应该在布局配置中保持一致，包括支持更大屏幕尺寸或横向设备方向的配置。
- 除非Fragment对用户可见，否则不要创建Fragment的视图层次结构。

本文档描述了如何在测试Fragment时使用框架提供的API。

## 驱动Fragment的状态

为了帮助设置执行这些测试的条件，AndroidX提供了一个FragmentScenario库来创建Fragment并更改其状态。

> 注意：要成功运行包含FragmentScenario对象的测试，请在测试线程中运行API的方法。要了解有关Android测试中如何使用不同线程的更多信息，请参阅[Understand threads in tests](https://developer.android.com/training/testing/fundamentals#threads).

### 配置测试工件位置

为了使用FragmentScenario，在应用程序的测试APK中定义Fragment测试工件，如以下代码片段所示：

```
dependencies {
    // ...
    debugImplementation 'androidx.fragment:fragment-testing:1.1.0-alpha01'
}
```

### 创建Fragment

FragmentScenario包含用于启动以下类型的Fragment的方法：

该方法还支持以下类型的Fragment：

- 图形Fragment（*Graphical*），包含用户界面。要启动此类型的Fragment，请调用launchFragmentInContainer（）。 FragmentScenario将Fragment附加到Activity的根视图控制器。This containing activity is otherwise empty.
- 非图形Fragment（*Non-graphical*）（有时称为无头Fragment- *headless fragments*），用于存储或执行对多个Activity中包含的信息的短期处理（ short-term processing ）。要启动此类型的Fragment，请调用launchFragment（）。 FragmentScenario将这种类型的Fragment附加到一个完全空的Activity，一个没有根视图的Activity。

在启动其中一个Fragment类型后，FragmentScenario将测试中的Fragment驱动到RESUMED状态。此状态表示Fragment正在运行。如果您正在测试图形Fragment，那么用户也可以看到它，因此您可以使用[Espresso UI tests](https://developer.android.com/training/testing/espresso)评估有关其UI元素的信息。

以下代码段显示了如何启动每种类型的Fragment：

#### 图形Fragment

```kotlin
@RunWith(AndroidJUnit4::class)
class MyTestSuite {
    @Test fun testEventFragment() {
        // The "state" and "factory" arguments are optional.
        val fragmentArgs = Bundle().apply {
            putInt("selectedListItem", 0)
        }
        val factory = MyFragmentFactory()
        val scenario = launchFragmentInContainer<MyFragment>(
                fragmentArgs, factory)
        onView(withId(R.id.text)).check(matches(withText("Hello World!")))
    }
}
```

#### 非图形Fragment

```kotlin
@RunWith(AndroidJUnit4::class)
class MyTestSuite {
    @Test fun testEventFragment() {
        // The "state" and "factory" arguments are optional.
        val fragmentArgs = Bundle().apply {
            putInt("numElements", 0)
        }
        val factory = MyFragmentFactory()
        val scenario = launchFragment<MyFragment>(fragmentArgs, factory)
    }
}
```

### 重建Fragment

如果设备的资源不足，系统可能会销毁宿主Activity，当用户重新返回app时需要你的app重建Fragment，为了模拟这个场景，调用recreate()：

```kotlin
@RunWith(AndroidJUnit4::class)
class MyTestSuite {
    @Test fun testEventFragment() {
        val scenario = launchFragmentInContainer<MyFragment>()
        scenario.recreate()
    }
}
```

当FragmentScenario类重新创建被测试的Fragment时，Fragment将返回到重新创建之前所处的生命周期状态。

### 驱动Fragment到新的状态

在您的应用程序的UI测试中，通常只需启动并重新创建测试中的Fragment即可。但是，在细粒度单元测试中，您还可以评估Fragment在从一个生命周期状态转换到另一个生命周期状态时的行为。

要将Fragment驱动到不同的生命周期状态，请调用moveToState（）。此方法支持以下状态作为参数：CREATED，STARTED，RESUMED和DESTROYED。此操作模拟包含您Fragment的宿主Activity因为它被另一个应用程序或系统操作中断而更改其状态的情况。

注意：如果将Fragment转换为DESTROYED状态，则无法将Fragment驱动到其他状态，也无法将Fragment附加到其他Activity。
moveToState（）的示例用法显示在以下代码段中：

```kotlin
@RunWith(AndroidJUnit4::class)
class MyTestSuite {
    @Test fun testEventFragment() {
        val scenario = launchFragmentInContainer<MyFragment>()
        scenario.moveToState(State.CREATED)
    }
}
```

> 警告：如果您尝试将测试中的Fragment转换为其当前状态，FragmentScenario会将此请求视为无操作，而不是抛出异常。特别注意，API允许您连续多次将片段转换为DESTROYED状态。

### 触发Fragment的行为

要在测试中的Fragment中触发某个行为，请使用Espresso视图匹配器与视图中的元素进行交互：

```kotlin
@RunWith(AndroidJUnit4::class)
class MyTestSuite {
    @Test fun testEventFragment() {
        val scenario = launchFragmentInContainer<MyFragment>()
        onView(withId(R.id.refresh))
                .perform(click())
    }
}
```

如果需要在Fragment本身上调用方法，例如响应选项菜单中的选择，则可以通过实现FragmentAction安全地执行此操作：

```kotlin
@RunWith(AndroidJUnit4::class)
class MyTestSuite {
    @Test fun testEventFragment() {
        val scenario = launchFragmentInContainer<MyFragment>()
        scenario.onFragment(fragment ->
            fragment.onOptionsItemSelected(clickedItem) {
                // Update fragment's state based on selected item.
            }
        }
    }
}
```



> 注意：在测试类中，不要保留传递给onFragment（）的对象的引用。这些引用消耗系统资源，并且引用本身可能是陈旧的，因为框架可以重新创建传递给回调方法的片段。

