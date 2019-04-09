# 构建应用程序小部件主机(Host)

[原文(英文)地址](https://developer.android.com/guide/topics/appwidgets/host#host-state)

大多数Android设备上提供的Android主屏幕允许用户嵌入应用小部件以便快速访问内容。如果您正在构建一个Home替换程序或类似的应用程序，您还可以允许用户通过实现AppWidgetHost来嵌入应用程序小部件。这不是大多数应用程序需要做的事情，但如果您要创建自己的主机，那么理解主机隐含同意的合同义务非常重要。

本文档重点介绍实现自定义AppWidgetHost所涉及的责任。有关如何实现AppWidgetHost的示例，请参阅Android主屏幕 [Launcher](https://android.googlesource.com/platform/packages/apps/Launcher2/+/master/src/com/android/launcher2/Launcher.java)的源代码。

以下是实现自定义AppWidgetHost所涉及的关键类和概念的概述：

- **App Widget Host**—— AppWidgetHost为AppWidget服务提供与想要在其UI中嵌入app小部件的应用程序（如主屏幕）的交互。 AppWidgetHost必须在主机自己的包中有唯一的ID，该ID在主机的所有使用中保持不变， ID通常是您在应用程序中分配的硬编码值（hard-coded value ）。
- **App Widget ID**——每个应用程序小组件实例在绑定时都会分配一个唯一的ID（请参阅[bindAppWidgetIdIfAllowed()](https://developer.android.com/reference/android/appwidget/AppWidgetManager.html#bindAppWidgetIdIfAllowed(int,%20android.content.ComponentName))，在[构建应用程序小部件](./构建应用程序小部件.md)中有更详细的讨论），主机使用allocateAppWidgetId（）获取唯一ID，此ID在窗口小部件的生命周期内是持久的，也就是说，直到从主机中删除它为止，任何特定于主机的状态（例如窗口小部件的大小和位置）都应由托管包（hosting package）保留，并与应用程序窗口小部件ID关联。
- **App Widget Host View**——可以将AppWidgetHostView视为每当需要显示小部件时包装小部件的框架。每次小部件由主机充气（inflated）时，都会将App小部件分配给AppWidgetHostView。
- **Options Bundle**——  AppWidgetHost使用options bundle 将信息传递给AppWidgetProvider，以了解窗口小部件的显示方式（例如，大小范围，以及窗口小部件是在锁屏还是主屏幕上）。此信息允许AppWidgetProvider根据窗口的显示方式和位置定制窗口小部件的内容和外观。您可以使用updateAppWidgetOptions（）和updateAppWidgetSize（）来修改应用程序窗口小部件的options bundle 。这两种方法都会触发对AppWidgetProvider的回调。

## 绑定App小部件

当用户将app小部件添加到主机时，会发生称为绑定的进程，绑定是指将特定应用程序窗口小部件ID与特定主机和特定AppWidgetProvider相关联。根据您的应用运行的Android版本，有不同的实现方法。

### 在Android4.0及以下进行绑定

在Android 4.0及更低版本的设备上，用户通过一个允许用户选择窗口小部件的系统Activity添加应用程序窗口小部件。这隐式地进行了权限检查——也就是说，通过添加应用小部件，用户隐式授予应用将应用小部件添加到主机的权限。下面是一个示例，说明了这种方法，取自Launcher的源码。在此片段中，事件处理程序使用请求代码REQUEST_PICK_APPWIDGET调用startActivityForResult（）以响应用户操作：

```java
private static final int REQUEST_CREATE_APPWIDGET = 5;
private static final int REQUEST_PICK_APPWIDGET = 9;
...
public void onClick(DialogInterface dialog, int which) {
    switch (which) {
    ...
        case AddAdapter.ITEM_APPWIDGET: {
            ...
            int appWidgetId =
                    Launcher.this.mAppWidgetHost.allocateAppWidgetId();
            Intent pickIntent =
                    new Intent(AppWidgetManager.ACTION_APPWIDGET_PICK);
            pickIntent.putExtra
                    (AppWidgetManager.EXTRA_APPWIDGET_ID, appWidgetId);
            ...
            startActivityForResult(pickIntent, REQUEST_PICK_APPWIDGET);
            break;
    }
    ...
}
```

系统Activity完成（finish）后，会将用户选择的应用小部件结果返回给您的Activity。在以下示例中，Activity通过调用addAppWidget（）作为响应以添加应用小部件：

```java
public final class Launcher extends Activity
        implements View.OnClickListener, OnLongClickListener {
    ...
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        mWaitingForResult = false;

        if (resultCode == RESULT_OK && mAddItemCellInfo != null) {
            switch (requestCode) {
                ...
                case REQUEST_PICK_APPWIDGET:
                    addAppWidget(data);
                    break;
                case REQUEST_CREATE_APPWIDGET:
                    completeAddAppWidget(data, mAddItemCellInfo, !mDesktopLocked);
                    break;
                }
        }
        ...
    }
}
```

方法addAppWidget（）检查应用小部件是否需要在添加之前进行配置：

```java
void addAppWidget(Intent data) {
    int appWidgetId = data.getIntExtra(AppWidgetManager.EXTRA_APPWIDGET_ID, -1);

    String customWidget = data.getStringExtra(EXTRA_CUSTOM_WIDGET);
    AppWidgetProviderInfo appWidget =
            mAppWidgetManager.getAppWidgetInfo(appWidgetId);

    if (appWidget.configure != null) {
        // Launch over to configure widget, if needed.
        Intent intent = new Intent(AppWidgetManager.ACTION_APPWIDGET_CONFIGURE);
        intent.setComponent(appWidget.configure);
        intent.putExtra(AppWidgetManager.EXTRA_APPWIDGET_ID, appWidgetId);
        startActivityForResult(intent, REQUEST_CREATE_APPWIDGET);
    } else {
        // Otherwise, finish adding the widget.
    }
}
```

关于更多配置的内容，请参阅[创建一个配置App小部件的configuration Activity](./构建应用程序小部件.md#creating-an-app-widget-configuration-activity)

应用程序小部件准备就绪后，下一步是将实际工作添加到工作区。Launcher的源码是使用一个名为completeAddAppWidget（）的方法来执行此操作。

### 在Android4.1及以上进行绑定

Android 4.1添加了API，以实现更简化的绑定过程。这些API还使主机可以提供用于绑定的自定义UI。要使用此改进的方法，您的应用必须在其manifest中声明BIND_APPWIDGET权限：

```xml
<uses-permission android:name="android.permission.BIND_APPWIDGET" />
```

但这只是第一步，在运行时，用户必须明确授予您的应用程序权限，以允许它将应用程序小部件添加到主机。要测试您的应用是否有权添加小部件，请使用bindAppWidgetIdIfAllowed（）方法。如果bindAppWidgetIdIfAllowed（）返回false，则您的应用必须显示一个对话框，提示用户授予权限（“允许”或“始终允许”，以涵盖所有未来的应用小部件添加）。此代码段提供了如何显示对话框的示例：

```java
Intent intent = new Intent(AppWidgetManager.ACTION_APPWIDGET_BIND);
intent.putExtra(AppWidgetManager.EXTRA_APPWIDGET_ID, appWidgetId);
intent.putExtra(AppWidgetManager.EXTRA_APPWIDGET_PROVIDER, info.componentName);
// This is the options bundle discussed above
intent.putExtra(AppWidgetManager.EXTRA_APPWIDGET_OPTIONS, options);
startActivityForResult(intent, REQUEST_BIND_APPWIDGET);
```

主机还必须检查用户是否添加了需要配置的应用程序小部件。有关此主题的更多讨论，请参阅[创建一个配置App小部件的configuration Activity](./构建应用程序小部件.md#creating-an-app-widget-configuration-activity)。

## 主机的责任

窗口小部件开发人员可以使用[AppWidgetProviderInfo元数据](./构建应用程序小部件.md#adding-the-appwidgetproviderinfo-metadata)为窗口小部件指定许多设置。下面更详细地讨论了这些配置选项可以由主机（host）从与窗口小部件提供者（widget provider）相关联的AppWidgetProviderInfo对象中检索。

无论您的Android版本如何，所有主机都应该遵循以下原则：

- 添加窗口小部件时，必须按上述方法分配窗口小部件ID。您还必须确保在从主机中删除窗口小部件时，调用deleteAppWidgetId（）来取消分配窗口小部件ID。
- 添加窗口小部件时，请务必启动其配置Activity（如果存在），如[在配置（configuration）Activity中更新App小部件](./构建应用程序小部件.md#Updating-the-App-Widget-from-the-Configuration-Activity)中所述。对于许多应用程序小部件，这是能正确显示它们所必要的步骤。
- 每个app小部件都以dps为单位指定最小宽度和高度，如AppWidgetProviderInfo元数据中所定义（使用android：minWidth和android：minHeight）。确保窗口小部件的布局至少包含这么多dps。例如，许多主机将网格中的图标和小部件对齐。在这种情况下，默认主机应使用满足minWidth和minHeight约束的最小单元格数添加应用程序窗口小部件。

除了上面列出的要求之外，特定平台版本还引入了在主机上承担新职责的功能。

### 你针对哪一个版本？

您在实现主机时使用的方法应取决于您要的Android版本。本节中描述的许多功能都是在3.0或更高版本中引入的。例如：

- Android 3.0（API Level 11）引入了小部件的自动前进（auto-advance）行为。
- Android 3.1（API Level 12）引入了调整窗口小部件大小的功能。
- Android 4.0（API Level 15）引入了填充策略的更改，该策略将主机的职责放在管理填充上。
- Android 4.1（API Level 16）添加了一个API，允许窗口小部件提供程序获取有关托管其窗口小部件实例的环境的更详细信息。
- Android 4.2（API Level 17）引入了options bundle和bindAppWidgetIdIfAllowed（）方法。它还引入了锁屏小部件。

如果您要针对设备，请参阅 [Launcher](https://android.googlesource.com/platform/packages/apps/Launcher/+/master/src/com/android/launcher/Launcher.java)的源码作为示例。

以下部分提供了有关在主机上放置新职责的功能的其他详细信息。

- Android 3.0
  Android 3.0（API Level 11）引入了一个小部件指定autoAdvanceViewId（）的功能。此视图ID应指向Advanceable的实例，例如StackView或AdapterViewFlipper。这表明主机应该以主机认为合适的间隔调用此视图上的advance（）（考虑到推进小部件是否有意义——例如，如果它在另一页上，或者如果屏幕关闭了，则主机可能不希望推进小部件）

- Android 3.1
  Android 3.1（API Level 12）引入了调整窗口小部件大小的功能。窗口小部件可以使用AppWidgetProviderInfo元数据中的android：resizeMode属性指定它是否可以调整大小，并指示它是否支持水平和/或垂直大小调整。在Android 4.0（API Level 14）中引入了小部件还可以指定android：minResizeWidth和/或android：minResizeHeight。

  主机负责使小部件可以按小部件的指定水平和/或垂直调整大小。指定可调整大小的窗口小部件可以任意调整大小，但不应调整大小小于android：minResizeWidth和android：minResizeHeight指定的值。有关示例实现，请参阅Launcher中的[`AppWidgetResizeFrame`](https://android.googlesource.com/platform/packages/apps/Launcher2/+/master/src/com/android/launcher2/AppWidgetResizeFrame.java)。

- Android 4.0
  Android 4.0（API Level 15）引入了填充策略的更改，该策略将主机的职责放在管理填充上。从4.0开始，app小部件不再包含自己的填充。相反，系统根据当前屏幕的特征为每个小部件添加填充。这导致网格中小部件更均匀、一致的呈现。为了帮助托管app小部件的应用程序，该平台提供了方法getDefaultPaddingForWidget（）。应用程序可以调用此方法来获取系统定义的填充，并在计算要分配给窗口小部件的单元格数时对其进行说明。

- Android 4.1
  Android 4.1（API Level 16）添加了一个API，允许窗口小部件提供程序获取有关托管其窗口小部件实例的环境的更详细信息。具体而言，主机向窗口小部件提供者提示有关窗口小部件显示的大小。主办方有责任提供此尺寸信息。

  主机通过updateAppWidgetSize（）提供此信息。大小指定为dps的最小和最大宽度/高度。指定范围（而不是固定大小）的原因是因为窗口小部件的宽度和高度可能会随方向而变化。您不希望主机在轮换时更新其所有小部件，因为这可能会导致严重的系统速度下降。这些值应该在放置窗口小部件时，每次调整窗口小部件时以及启动器在给定引导中第一次膨胀窗口小部件时更新（因为值不会在引导期间保持不变）。

- Android 4.2
  Android 4.2（API级别17）增加了在绑定时指定选项包的功能。这是指定应用小部件选项（包括大小）的理想方式，因为它使AppWidgetProvider可以立即访问第一次更新时的选项数据。这可以通过使用bindAppWidgetIdIfAllowed（）方法来实现。有关此主题的更多讨论，请参阅绑定应用程序小部件。

  Android 4.2还引入了锁屏小部件。在锁定屏幕上托管小部件时，主机必须在应用小部件选项包中指定此信息（AppWidgetProvider可以使用此信息来适当地设置小部件的样式）。要将窗口小部件指定为锁屏窗口小部件，请使用updateAppWidgetOptions（）并包含值为WIDGET_CATEGORY_KEYGUARD的字段OPTION_APPWIDGET_HOST_CATEGORY。此选项默认为WIDGET_CATEGORY_HOME_SCREEN，因此不明确要求为主屏幕主机设置此选项。

  确保您的主机仅添加适合您的应用的应用小部件 - 例如，如果您的主机是主屏幕，请确保AppWidgetProviderInfo元数据中的android：widgetCategory属性包含标志WIDGET_CATEGORY_HOME_SCREEN。同样，对于锁屏，请确保该字段包含标志WIDGET_CATEGORY_KEYGUARD。有关此主题的更多讨论，请参阅在锁屏上启用App小部件。

