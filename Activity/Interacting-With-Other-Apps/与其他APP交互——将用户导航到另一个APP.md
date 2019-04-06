#  与其他APP交互——将用户导航到另一个APP

[原文(英文)地址](https://developer.android.com/training/basics/intents/sending)

Android最重要的功能之一是应用程序能够根据其想要执行的“操作”将用户导航到另一个应用程序。例如，如果您的应用具有您希望在地图上显示的商家的地址，则您不必在应用中构建显示地图的Activity，您可以使用Intent创建查看地址的请求，然后Android系统会启动一个能够在地图上显示地址的应用程序。

您必须使用Intent在您自己的应用程序中的Activity之间导航。通常使用显式Intent来执行此操作，该Intent定义了要启动的组件的确切类名。但是，如果要让特定的应用程序执行操作（例如“查看地图”），则必须使用隐式意图。

本文档向您展示如何为特定操作创建隐式Intent，以及如何使用它来启动在另一个应用程序中执行操作的Activity。另请参阅此处嵌入的[视频](https://youtu.be/HGElAW224dE)，以了解为什么运行时对隐式Intent的检查很重要。

## 创建一个隐式Intent

隐式Intent不会声明要启动的组件的类名，而是声明要执行的操作。该声明指明了您要执行的操作，例如查看，编辑，发送或获取某些内容。Intent通常还包括与操作关联的数据，例如您要查看的地址或您要发送的电子邮件。根据您要创建的Intent，数据可能是Uri，其他几种数据类型之一，intent也可能根本不需要数据。

如果您的数据是Uri，那么可以使用一个简单的Intent（）构造函数来定义操作和数据。

例如，以下是如何使用Uri数据创建发起电话呼叫的Intent以指定电话号码：

- kotlin

  ```kotlin
  val callIntent: Intent = Uri.parse("tel:5551234").let { number ->
      Intent(Intent.ACTION_DIAL, number)
  }
  ```

- java

  ```java
  Uri number = Uri.parse("tel:5551234");
  Intent callIntent = new Intent(Intent.ACTION_DIAL, number);
  ```

当你的app通过startActivity()调用此Intent时，Phone app将会向传入的电话号码拨打电话。

这里是一些其他的Intent及其指定的动作和数据：

- 查看地图

  - kotlin

    ```kotlin
    // Map point based on address
    val mapIntent: Intent = Uri.parse(
            "geo:0,0?q=1600+Amphitheatre+Parkway,+Mountain+View,+California"
    ).let { location ->
        // Or map point based on latitude/longitude
        // Uri location = Uri.parse("geo:37.422219,-122.08364?z=14"); // z param is zoom level
        Intent(Intent.ACTION_VIEW, location)
    }
    ```

  - java

    ```java
    // Map point based on address
    Uri location = Uri.parse("geo:0,0?q=1600+Amphitheatre+Parkway,+Mountain+View,+California");
    // Or map point based on latitude/longitude
    // Uri location = Uri.parse("geo:37.422219,-122.08364?z=14"); // z param is zoom level
    Intent mapIntent = new Intent(Intent.ACTION_VIEW, location);
    ```

- 查看网页

  - kotlin

    ```kotlin
    val webIntent: Intent = Uri.parse("http://www.android.com").let { webpage ->
        Intent(Intent.ACTION_VIEW, webpage)
    }
    ```

  - java

    ```java
    Uri webpage = Uri.parse("http://www.android.com");
    Intent webIntent = new Intent(Intent.ACTION_VIEW, webpage);
    ```

其他类型的隐式Intent需要提供不同数据类型的“Extra”数据，例如字符串。您可以使用各种putExtra（）方法添加一个或多个额外数据。

默认情况下，系统根据包含的Uri数据确定intent所需的相应MIME类型。如果您没有在Intent中包含Uri，则通常应使用setType（）来指定与Intent关联的数据类型。设置MIME类型进一步指定应该接收Intent的活动类型。

以下是一些添加额外数据以指定所需操作的Intent：

- 发送带有附件的邮件

  - kotlin

    ```kotlin
    Intent(Intent.ACTION_SEND).apply {
        // The intent does not have a URI, so declare the "text/plain" MIME type
        type = HTTP.PLAIN_TEXT_TYPE
        putExtra(Intent.EXTRA_EMAIL, arrayOf("jon@example.com")) // recipients
        putExtra(Intent.EXTRA_SUBJECT, "Email subject")
        putExtra(Intent.EXTRA_TEXT, "Email message text")
        putExtra(Intent.EXTRA_STREAM, Uri.parse("content://path/to/email/attachment"))
        // You can also attach multiple items by passing an ArrayList of Uris
    }
    ```

  - java

    ```java
    Intent emailIntent = new Intent(Intent.ACTION_SEND);
    // The intent does not have a URI, so declare the "text/plain" MIME type
    emailIntent.setType(HTTP.PLAIN_TEXT_TYPE);
    emailIntent.putExtra(Intent.EXTRA_EMAIL, new String[] {"jon@example.com"}); // recipients
    emailIntent.putExtra(Intent.EXTRA_SUBJECT, "Email subject");
    emailIntent.putExtra(Intent.EXTRA_TEXT, "Email message text");
    emailIntent.putExtra(Intent.EXTRA_STREAM, Uri.parse("content://path/to/email/attachment"));
    // You can also attach multiple items by passing an ArrayList of Uris
    ```

- 创建一个日历事件

  - kotlin

    ```kotlin
    Intent(Intent.ACTION_INSERT, Events.CONTENT_URI).apply {
        val beginTime: Calendar = Calendar.getInstance().apply {
            set(2012, 0, 19, 7, 30)
        }
        val endTime = Calendar.getInstance().apply {
            set(2012, 0, 19, 10, 30)
        }
        putExtra(CalendarContract.EXTRA_EVENT_BEGIN_TIME, beginTime.timeInMillis)
        putExtra(CalendarContract.EXTRA_EVENT_END_TIME, endTime.timeInMillis)
        putExtra(Events.TITLE, "Ninja class")
        putExtra(Events.EVENT_LOCATION, "Secret dojo")
    }
    ```

  - java

    ```java
    Intent calendarIntent = new Intent(Intent.ACTION_INSERT, Events.CONTENT_URI);
    Calendar beginTime = Calendar.getInstance();
    beginTime.set(2012, 0, 19, 7, 30);
    Calendar endTime = Calendar.getInstance();
    endTime.set(2012, 0, 19, 10, 30);
    calendarIntent.putExtra(CalendarContract.EXTRA_EVENT_BEGIN_TIME, beginTime.getTimeInMillis());
    calendarIntent.putExtra(CalendarContract.EXTRA_EVENT_END_TIME, endTime.getTimeInMillis());
    calendarIntent.putExtra(Events.TITLE, "Ninja class");
    calendarIntent.putExtra(Events.EVENT_LOCATION, "Secret dojo");
    ```

  > 注意：此Intent只支持API14及更高的平台

> 注意：尽可能具体地定义Intent是非常重要的。例如，如果要使用ACTION_VIEW Intent显示图像，则应指定MIME类型image / *。这可以防止可以“view”其他类型数据（如地图应用程序）的应用程序被Intent触发。

## 验证是否有APP可以接收Intent

虽然Android平台保证某些Intent将解析为内置应用程序之一（例如电话，电子邮件或日历应用程序），但您应始终在调用Intent之前包含验证步骤。

>  警告：如果您调用intent并且设备上没有可以处理Intent的应用程序，则您的应用程序将崩溃。

要验证是否有可以响应Intent的活动，请调用queryIntentActivities（）以获取能够处理您的Intent的活动列表。如果返回的List不为空，则可以安全地使用intent。例如：

- kotlin

  ```kotlin
  val activities: List<ResolveInfo> = packageManager.queryIntentActivities(
          intent,
          PackageManager.MATCH_DEFAULT_ONLY
  )
  val isIntentSafe: Boolean = activities.isNotEmpty()
  ```

- java

  ```java
  PackageManager packageManager = getPackageManager();
  List<ResolveInfo> activities = packageManager.queryIntentActivities(intent,
          PackageManager.MATCH_DEFAULT_ONLY);
  boolean isIntentSafe = activities.size() > 0;
  ```

如果isIntentSafe为true，则至少有一个app可以相应该Intent，如果为false，则没有app可以相应该Intent。

> 注意：您应该在Activity首次启动时执行此检查，以防你需要在用户尝试使用该功能之前禁用使用Intent的功能。如果您知道可以处理意图的特定应用，您还可以给提供用户下载应用的链接（请参阅如何 [link to your product on Google Play](https://developer.android.com/distribute/tools/promote/linking.html)）。

## 使用Intent启动Activity

![intents-choice](https://ws4.sinaimg.cn/large/006tNc79gy1g1sz1g8qtcj30a8091mxl.jpg)

<center>图一：当有多个app可以相应Intent时的选择框</center>

当你创建完成Intent并且设置好Extra信息之后，请调用startActivity()以将该Intent发送到系统。如果系统有不知一个Activity可以处理该Intent，系统会显示一个dialog以供用户选择一个最终被使用的app，如图一。如果只有一个activity可以响应Intent，则系统会直接启动它。

这里是一个稍微复杂的例子，展示了如何创建一个展示地图的Intent，首先验证是否有应用可以响应，然后启动它：

- kotlin

  ```kotlin
  // Build the intent
  val location = Uri.parse("geo:0,0?q=1600+Amphitheatre+Parkway,+Mountain+View,+California")
  val mapIntent = Intent(Intent.ACTION_VIEW, location)
  
  // Verify it resolves
  val activities: List<ResolveInfo> = packageManager.queryIntentActivities(mapIntent, 0)
  val isIntentSafe: Boolean = activities.isNotEmpty()
  
  // Start an activity if it's safe
  if (isIntentSafe) {
      startActivity(mapIntent)
  }
  ```

- java

  ```java
  // Build the intent
  Uri location = Uri.parse("geo:0,0?q=1600+Amphitheatre+Parkway,+Mountain+View,+California");
  Intent mapIntent = new Intent(Intent.ACTION_VIEW, location);
  
  // Verify it resolves
  PackageManager packageManager = getPackageManager();
  List<ResolveInfo> activities = packageManager.queryIntentActivities(mapIntent, 0);
  boolean isIntentSafe = activities.size() > 0;
  
  // Start an activity if it's safe
  if (isIntentSafe) {
      startActivity(mapIntent);
  }
  ```

## 展示Activity的选择界面

![intent-chooser](https://ws2.sinaimg.cn/large/006tNc79gy1g1sz658hj7j308c0gogmz.jpg)

<center>图二：一个选择对话框</center>

请注意，当您通过将Intent传递给startActivity（）并且有多个应用程序可以响应intent时，用户可以选择默认使用哪个应用程序（通过选择对话框底部的复选框，见图1）。这在用户通常希望每次都使用相同应用去执行的动作时很好，例如当打开网页（用户可能只使用一个网络浏览器）或拍照时（用户可能更喜欢一个相机）。

但是，如果要执行的操作可以由多个应用程序处理，并且用户可能每次都更喜欢不同的应用程序——例如“分享”操作，用户可能拥有多个应用程序，通过这些应用程序可以分享项目，这时您应该明确显示一个选择器对话框，如图2所示。选择器对话框强制用户每次选择用于操作的应用程序（用户无法为操作选择默认应用程序）。

要显示选择器，请使用createChooser（）创建一个Intent并将其传递给startActivity（）。例如：

- kotlin

  ```kotlin
  val intent = Intent(Intent.ACTION_SEND)
  ...
  
  // Always use string resources for UI text.
  // This says something like "Share this photo with"
  val title = resources.getString(R.string.chooser_title)
  // Create intent to show chooser
  val chooser = Intent.createChooser(intent, title)
  
  // Verify the intent will resolve to at least one activity
  if (intent.resolveActivity(packageManager) != null) {
      startActivity(chooser)
  }
  ```

- java

  ```java
  Intent intent = new Intent(Intent.ACTION_SEND);
  ...
  
  // Always use string resources for UI text.
  // This says something like "Share this photo with"
  String title = getResources().getString(R.string.chooser_title);
  // Create intent to show chooser
  Intent chooser = Intent.createChooser(intent, title);
  
  // Verify the intent will resolve to at least one activity
  if (intent.resolveActivity(getPackageManager()) != null) {
      startActivity(chooser);
  }
  ```

这将显示一个对话框，其中包含可以响应传递给createChooser（）方法的Intent的应用程序列表，并使用提供的文本作为对话框标题。