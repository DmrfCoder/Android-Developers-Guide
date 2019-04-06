# 处理Android APP Links——创建指向您内容的Deep Links

[原文(英文)地址](https://developer.android.com/training/app-links/deep-linking)

当单击链接或编程请求调用Web URI Intent时，Android系统将按顺序尝试以下每个操作，直到请求成功为止：

- 打开用户首选的可以处理URI的应用程序（如果已指定）。
- 打开唯一可以处理URI的可用应用程序。
- 允许用户从对话框中选择应用程序。

请按照以下步骤创建和测试指向您的内容的链接。您还可以使用Android Studio中的 [App Links Assistant](https://developer.android.com/studio/write/app-link-indexing.html?hl=zh-cn) 添加Android App Links。

## 为传入的链接添加Intent filter

要创建一个导向你app内容的链接，你需要在你的manifest中添加intent filter并且设置如下三个属性：

- <action\>

  > 指定ACTION_VIEW Intent操作，以便可以从Google搜索访问Intent filter

- <data\>

  > 添加一个或多个<data\>标记，每个标记代表一种解析为Activity的URI格式。<data\>标记必须至少包含android：scheme属性。
  > 您可以添加更多属性以进一步优化Activity接受的URI类型。例如，您可能有多个Activity接受类似的URI，但这些Activity仅根据路径名称而有所不同。在这种情况下，使用android：path属性或其pathPattern或pathPrefix变体来区分系统应为不同的URI路径打开哪个Activity。

- <category\>

  > 包括BROWSABLE category。为了从Web浏览器访问intent filter而需要它。没有它，单击浏览器中的链接无法解析为您的应用程序。
  > 还包括DEFAULT category。这允许您的应用响应隐式Intent。如果没有这个，只有在intent指定您的应用程序组件名称时才能启动Activity。

以下XML代码段显示了如何在manifest中为Deep Links指定intent filter。“example://gizmos”` 和“http://www.example.com/gizmos”这两个URI均会指向次Activity：

```xml
<activity
    android:name="com.example.android.GizmosActivity"
    android:label="@string/title_gizmos" >
    <intent-filter android:label="@string/filter_view_http_gizmos">
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <!-- Accepts URIs that begin with "http://www.example.com/gizmos” -->
        <data android:scheme="http"
              android:host="www.example.com"
              android:pathPrefix="/gizmos" />
        <!-- note that the leading "/" is required for pathPrefix-->
    </intent-filter>
    <intent-filter android:label="@string/filter_view_example_gizmos">
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <!-- Accepts URIs that begin with "example://gizmos” -->
        <data android:scheme="example"
              android:host="gizmos" />
    </intent-filter>
</activity>
```

请注意，两个intent filter仅因<data\>元素而不同。虽然可以在同一个过滤器中包含多个<data\>元素，但是当您打算声明唯一的URL（例如scheme和host的特定组合）时，创建单独的过滤器非常重要，因为实际上在同一个Intent filter下的多个<data\>元素会合并在一起，以考虑其组合属性的所有变体。例如，请考虑以下代码：

```xml
<intent-filter>
  ...
  <data android:scheme="https" android:host="www.example.com" />
  <data android:scheme="app" android:host="open.my.app" />
</intent-filter>
```

看起来该Intent filter仅仅会支持 `https://www.example.com` 和`app://open.my.app`，但是实际上他还另外支持这两个：`app://www.example.com` 和 `https://open.my.app`.(也就是说会被组合)

## 从传入的Intent中读取数据

一旦系统通过Intent filter启动您的Activity，您就可以使用Intent提供的数据来确定您需要呈现的内容。调用getData（）和getAction（）方法来取出与传入的Intent关联的数据和操作。您可以在Activity的生命周期中随时调用这些方法，但通常应该在早期回调期间执行此操作，例如onCreate（）或onStart（）。

这里展示了如何从Intent中取到数据：

- kotlin

  ```kotlin
  override fun onCreate(savedInstanceState: Bundle?) {
      super.onCreate(savedInstanceState)
      setContentView(R.layout.main)
  
      val action: String? = intent?.action
      val data: Uri? = intent?.data
  }
  ```

- java

  ```java
  @Override
  public void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      setContentView(R.layout.main);
  
      Intent intent = getIntent();
      String action = intent.getAction();
      Uri data = intent.getData();
  }
  ```

请遵循以下最佳做法以改善用户体验：

- deep link应该将用户直接带到内容（content），而无需任何提示、插页式页面，或者登录。即使用户之前从未打开过该应用程序也要确保用户可以直接查看应用程序内容。可以在后续交互时或从Launcher打开应用程序时提示用户。这与网站第一次免费体验的原则相同。
- 遵守“[Navigation with Back and Up](https://developer.android.com/design/patterns/navigation.html?hl=zh-cn)”中所述的设计指南，以便您的应用在用户通过Deep link进入应用后满足用户向后导航的期望。

## 测试你的Deep Links

您可以将 [Android Debug Bridge](https://developer.android.com/tools/help/adb.html?hl=zh-cn) 与activity manager（am）工具结合起来测试您为Deep Links指定的intent filter 的URI是否解析为正确的app Activity。您可以对设备或模拟器运行adb命令。

使用adb测试  Intent filter URI的一般语法是：

```shell
$ adb shell am start
        -W -a android.intent.action.VIEW
        -d <URI> <PACKAGE>
```

例如，以下命令尝试查看与指定URI关联的目标应用程序的Activity。

```shell
$ adb shell am start
        -W -a android.intent.action.VIEW
        -d "example://gizmos" com.example.android
```

您在上面设置的manifest和Intent处理程序定义了您的应用程序和网站之间的连接以及如何处理传入链接。但是，为了让系统将您的应用视为一组URI的默认处理程序，您还必须请求系统验证此连接。下一节将介绍如何实施此验证。

想了解更多关于Intent和app links的内容，请参考：

- [Intents and Intent Filters](https://developer.android.com/guide/components/intents-filters.html?hl=zh-cn)
- [Allow Other Apps to Start Your Activity](https://developer.android.com/training/basics/intents/filters.html?hl=zh-cn)
- [Add Android App Links with Android Studio](https://developer.android.com/studio/write/app-link-indexing.html?hl=zh-cn)

