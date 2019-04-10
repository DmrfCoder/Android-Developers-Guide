# 数据（Data Binding）绑定库——开始

了解如何使您的开发环境随时可以使用数据绑定库（ Data Binding Library），包括在Android Studio中支持数据绑定的代码。

数据绑定库提供了灵活性和广泛的兼容性 - 它是一个支持库，因此您可以将其用于运行Android 4.0（API级别14）或更高版本的设备。

建议在项目中使用最新的Gradle Android插件。但是，1.5.0及更高版本支持数据绑定。有关更多信息，请参阅如何更新Gradle的Android插件（[update the Android Plugin for Gradle](https://developer.android.com/studio/releases/gradle-plugin.html#updating-plugin)）。

## 构建环境

为了使用数据绑定库，请从Android Sdk manager的Support Repository中下载库文件，关于更多信息请参考[Update the IDE and SDK Tools](https://developer.android.com/studio/intro/update.html).

为了配置你的app可以使用数据绑定库，请添加"dataBinding"元素到你app module的build.gradle文件中，就像下面这样：

```xml
android {
    ...
    dataBinding {
        enabled = true
    }
}
```

> 注意：您必须为依赖于使用数据绑定的库的应用程序模块配置数据绑定，即使应用程序模块不直接使用数据绑定也是如此。

## Android Studio对于数据绑定的支持

Android Studio支持许多用于数据绑定代码的编辑功能。例如，它支持数据绑定表达式的以下功能：

- 语法突出显示
- 标记表达式语言语法错误
- XML代码自动完成
- 参考，包括导航（[navigation](https://www.jetbrains.com/help/idea/2017.1/navigation-in-source-code.html)）（例如导航到声明）和快速文档([quick documentation](https://www.jetbrains.com/help/idea/2017.1/viewing-inline-documentation.html))



> 警告：数组和泛型类型（如[`Observable`](https://developer.android.com/reference/android/databinding/Observable.html)类）显示的错误可能是不正确的。

布局编辑器中的“预览（preview）”窗格显示数据绑定表达式的默认值（如果提供）。例如，“预览”窗格在以下示例中声明的TextView小部件上显示my_default值：

```xml
<TextView android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@{user.firstName, default=my_default}"/>
```

如果只需要在项目的设计阶段显示默认值，则可以使用"tools"属性而不是默认表达式值，如“[Tools Attributes Reference](https://developer.android.com/studio/write/tool-attributes.html)”中所述。

## 用于绑定类的新数据绑定编译器

Android Gradle插件版本3.1.0-alpha06包含一个生成绑定类的新的数据绑定编译器。新编译器会逐步创建绑定类，这在大多数情况下会加快构建过程。要了解有关绑定类的更多信息，请[生成的绑定类](./数据(Data Binding)绑定库——生成的绑定类.md)。

以前版本的数据绑定编译器在编译托管代码的同时生成绑定类，如果您的托管代码无法编译，您可能会收到多个报告未找到绑定类的错误，新的数据绑定编译器通过在托管编译器构建应用程序之前生成绑定类来防止这些错误。

要启用新的数据绑定编译器，请将以下选项添加到gradle.properties文件中：

```groovy
android.databinding.enableV2=true
```

您还可以通过在gradle命令中添加以下参数启用新编译器：

```
-Pandroid.databinding.enableV2=true
```

> 注意：Android插件版本3.1中的新数据绑定编译器不向后兼容。您需要生成所有绑定类，并启用此功能以利用增量编译( incremental compilation)。但是，Android插件版本3.2中的新编译器与先前版本生成的绑定类兼容。默认情况下会启用版本3.2中的新编译器。

启用新数据绑定编译器时，默认会遵守以下行为：

- 在编译托管代码之前，Android Gradle插件会为您的布局生成绑定类。
- 如果布局（layout）被多个目标资源配置（resource configuration）包含，则数据绑定库使用android.view.View作为共享相同资源ID的所有视图（View）的默认视图类型，而不是作为所有类型相同的视图默认视图类型。
- 编译库模块（ library modules）的绑定类会被编译并打包到相应的Android Archive（AAR）文件中。依赖于这些库模块的应用程序模块不再需要重新生成绑定类。有关AAR文件的更多信息，请参阅[Create an Android Library](https://developer.android.com/studio/projects/android-library)。
- 模块（module）的绑定适配器不能更改模块依赖项的适配器的行为。绑定适配器仅影响其自己的模块中的代码和模块的使用者。

## 其他资源

想了解关于数据绑定的更多信息，请参阅：

Samples

- [Android Data Binding Library samples](https://github.com/googlesamples/android-databinding)

Codelabs

- [Android Data Binding codelab](https://codelabs.developers.google.com/codelabs/android-databinding)

Blog posts

- [Data Binding — Lessons Learnt](https://medium.com/androiddevelopers/data-binding-lessons-learnt-4fd16576b719)