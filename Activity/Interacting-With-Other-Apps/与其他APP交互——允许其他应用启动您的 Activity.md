# 与其他APP交互——允许其他应用启动您的 Activity

[原文(英文)地址](https://developer.android.com/training/basics/intents/filters?#AddIntentFilter)

如果另一个应用想要通过您的应用执行某些操作，您的应用应准备好响应来自其他应用的操作请求。 例如，如果您构建一款可与用户的好友分享消息或照片的社交应用，您最关注的是支持 `ACTION_SEND` Intent 以便用户可以从另一应用发起“分享”操作并且启动您的应用执行该操作。

要允许其他应用启动您的 Activity，您需要在清单文件中为对应的<activity\> 元素添加一个 <intent-filter\>元素。

当您的应用安装在设备上时，系统会识别您的 Intent  filter 并添加信息至所有已安装应用支持的 Intent 内部目录。当应用通过隐含 Intent 调用 `startActivity()` 或 `startActivityForResult()` 时，系统会找到可以响应该 Intent 的 Activity。

## 添加 Intent 过滤器

为了正确定义您的 Activity 可处理的 Intent，您添加的每个 Intent filter在操作类型和 Activity 接受的数据方面应尽可能具体。

如果 Activity 具有满足以下 `Intent` 对象条件的 Intent filter，系统可能向 Activity 发送给定的 `Intent`：

- 操作

  对要执行的操作命名的字符串。通常是平台定义的值之一，比如 `ACTION_SEND` 或 `ACTION_VIEW`。使用<action\>元素在您的 Intent 过滤器中指定此值。您在此元素中指定的值必须是操作的完整字符串名称，而不是 API 常量（请参阅以下示例）。

- 数据

  与 Intent 关联的数据描述。用<data\>元素在您的 Intent 过滤器中指定此内容。使用此元素中的一个或多个属性，您可以只指定 MIME 类型、URI 前缀、URI 架构或这些的组合以及其他指示所接受数据类型的项。

  > **注意**：如果您无需声明关于数据的具体信息 `Uri`（比如，您的 Activity 处理其他类型的“额外”数据而不是 URI 时），您应只指定 `android:mimeType` 属性声明您的 Activity 处理的数据类型，比如 `text/plain` 或 `image/jpeg`。

- 类别

  提供另外一种表征处理 Intent 的 Activity 的方法，通常与用户手势或 Activity 启动的位置有关。 系统支持多种不同的类别，但大多数都很少使用。 但是，所有隐含 Intent 默认使用 `CATEGORY_DEFAULT` 进行定义。用<category\>元素在您的 Intent 过滤器中指定此内容。

在您的 Intent 过滤器中，您可以通过声明嵌套在<intent-filter\>元素中的具有相应 XML 元素的各项，来声明您的 Activity 接受的条件。

例如，此处有一个 Activity 与在数据类型为文本或图像时处理 `ACTION_SEND` Intent 的 Intent 过滤器：

```xml
<activity android:name="ShareActivity">
    <intent-filter>
        <action android:name="android.intent.action.SEND"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <data android:mimeType="text/plain"/>
        <data android:mimeType="image/*"/>
    </intent-filter>
</activity>
```

每个入站 Intent 仅指定一项操作和一个数据类型，但可以在每个 <intent-filter\>中声明 <action\>,<category\>,<data\>元素的多个实例。

如果任何两对操作和数据的行为相斥，您应创建单独的 Intent 过滤器指定与哪种数据类型配对时哪些操作可接受。

比如，假定您的 Activity 同时处理 `ACTION_SEND` 和 `ACTION_SENDTO` Intent 的文本和图像。在这种情况下，您必须为两个操作定义两种不同的 Intent 过滤器，因为 `ACTION_SENDTO` Intent 必须使用数据 `Uri` 指定使用 `send` 或 `sendto` URI 架构的收件人地址。例如：

```xml
<activity android:name="ShareActivity">
    <!-- filter for sending text; accepts SENDTO action with sms URI schemes -->
    <intent-filter>
        <action android:name="android.intent.action.SENDTO"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <data android:scheme="sms" />
        <data android:scheme="smsto" />
    </intent-filter>
    <!-- filter for sending text or images; accepts SEND action and text or image data -->
    <intent-filter>
        <action android:name="android.intent.action.SEND"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <data android:mimeType="image/*"/>
        <data android:mimeType="text/plain"/>
    </intent-filter>
</activity>
```

> **注意**：为了接收隐式 Intent，您必须在 Intent 过滤器中包含 `CATEGORY_DEFAULT` 类别。方法 `startActivity()` 和`startActivityForResult()` 将按照已声明 `CATEGORY_DEFAULT` 类别的方式处理所有 Intent。如果您不在 Intent 过滤器中声明它，则没有隐含 Intent 分解为您的 Activity。

如需了解有关发送和接收 `ACTION_SEND` 执行社交共享行为的 Intent 的详细信息，请参阅有关[Receiving simple data from other apps](https://developer.android.com/training/sharing/receive.html?hl=zh-cn)的文档。

## 处理您的 Activity 中的 Intent

为了决定在您的 Activity 执行哪种操作，您可读取用于启动 Activity 的 `Intent`。

当您的 Activity 启动时，调用 `getIntent()` 检索启动 Activity 的 `Intent`。您可以在 Activity 生命周期的任何时间执行此操作，但您通常应在早期回调时（比如，`onCreate()` 或 `onStart()`）执行。

例如：

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    setContentView(R.layout.main);

    // Get the intent that started this activity
    Intent intent = getIntent();
    Uri data = intent.getData();

    // Figure out what to do based on the intent type
    if (intent.getType().indexOf("image/") != -1) {
        // Handle intents with image data ...
    } else if (intent.getType().equals("text/plain")) {
        // Handle intents with text ...
    }
}
```

### 返回结果

如果您想要向调用您的 Activity 的 Activity 返回结果，只需调用 `setResult()` 指定结果代码和结果 `Intent`。当您的操作完成且用户应返回原始 Activity 时，调用 `finish()` 关闭（和销毁）您的 Activity。 例如：

```java
// Create intent to deliver some kind of result data
Intent result = new Intent("com.example.RESULT_ACTION", Uri.parse("content://result_uri"));
setResult(Activity.RESULT_OK, result);
finish();
```

您必须始终为结果指定结果代码。通常，它是 `RESULT_OK` 或 `RESULT_CANCELED`。您之后可以根据需要为 `Intent` 提供额外的数据。

**注**：默认情况下，结果设置为 `RESULT_CANCELED`。因此，如果用户在完成操作动作或设置结果之前按了*返回*按钮，原始 Activity 会收到“已取消”的结果。

如果您只需返回指示若干结果选项之一的整数，您可以将结果代码设置为大于 0 的任何值。如果您使用结果代码传递整数，且无需包括 `Intent`，则可调用 `setResult()` 且仅传递结果代码。例如：

```java
setResult(RESULT_COLOR_RED);
finish();
```

在这种情况下，只有几个可能的结果，因此结果代码是一个本地定义的整数（大于 0）。 当您向自己应用中的 Activity 返回结果时，这将非常有效，因为接收结果的 Activity 可引用公共常数来确定结果代码的值。

> **注意**：无需检查您的 Activity 是使用 `startActivity()` 还是 `startActivityForResult()` 启动的。如果启动您 Activity 的 Intent 可能需要结果，只需调用 `setResult()`。如果原始 Activity 已调用 `startActivityForResult()`，则系统将向其传递您提供给 `setResult()` 的结果；否则，会忽略结果。