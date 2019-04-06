# 创建一个Fragment

[原文(英文)地址](https://developer.android.com/training/basics/fragments/creating)

您可以将Fragment视为Activity的模块化部分，它具有自己的生命周期，接收自己的输入事件，并且可以在Activity运行时添加或删除（有点像“子Activity”，您可以在不同的Activity中重用）。本文档介绍如何使用 [Support Library](https://developer.android.com/tools/support-library/index.html)继承Fragment类，以便您的应用程序与运行系统版本低至Android 1.6的设备保持兼容。

您应该创建一个生命周期感知组件（lifecycle-aware component），而不是在Fragment的生命周期方法中设置依赖组件（dependent components）。当Fragment在其生命周期中移动时，该组件可以监听到你Fragment生命周期过程中发生的任何改变。然后，可以在其他Fragments和Activities中重用生命周期感知组件，以避免代码重复，并减少在Fragments / Activities本身中需要进行的设置。有关更多信息，请阅读 [Handling Lifecycles with Lifecycle-Aware Components](https://developer.android.com/topic/libraries/architecture/lifecycle.html).

在开始之前，您必须设置Android项目以使用Support Library.。如果您之前未使用过Support Library.，请按照 [Support Library Setup](https://developer.android.com/tools/support-library/setup.html)将项目设置为使用v4库。您也可以在Activity中包含[app bar](https://developer.android.com/training/appbar/index.html)，而不是使用与Android 2.1（API级别7）兼容的v7 appcompat库，还包括Fragment API。

有关实现Fragment的更多信息，请参阅[Fragment](./Activity/Fragment)。您还可以通过浏览相关的[sample app](http://developer.android.com/shareables/training/FragmentBasics.zip)了解更多信息。

## 创建一个Fragment类

为了创建一个Fragment，你应该继承Fragment类，重写其生命周期的回调方法，这和你使用Activity的方法类似。

你应该在onCreateView()中创建你Fragment的layout，事实上这是保证Fragment可以运行的基本回调方法，例如下面是一个指定Layout的示例代码：

```java
import android.os.Bundle;
import android.support.v4.app.Fragment;
import android.view.LayoutInflater;
import android.view.ViewGroup;

public class ArticleFragment extends Fragment {
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
        Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.article_view, container, false);
    }
}
```

就像Activity一样，Fragment应该实现其他生命周期回调，这样就可以允许您管理Fragment的状态（当该Fragment被添加到Activity或被从Activity中移除的时候，或者在Activity生命周期变化的时候）。例如，当调用activity的onPause（）方法时，Activity中的任何Fragment也会接收对onPause（）的调用。

有关Fragment生命周期和回调方法的更多信息，请参阅[Fragment](./Activity/Fragment)。

## 使用XML文件将Fragment添加到Activity

虽然Fragment是可重用的，模块化UI组件，但Fragment类的每个实例都必须与父FragmentActivity相关联。您可以通过定义Activity布局XML文件中的Fragment来实现此关联。

>  注意：FragmentActivity是支持库（ Support Library ）中提供的一个特殊Activity，用于处理早于API 11的系统版本上的Fragment。如果您支持的最低系统版本是API级别11或更高级别，则可以使用常规Activity。

下面是一个示例布局文件，当设备屏幕被视为“大”（由目录名中的大限定符指定）时，它会向Activity添加两个Fragment。

res/layout-large/news_articles.xml：

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="horizontal"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent">

    <fragment android:name="com.example.android.fragments.HeadlinesFragment"
              android:id="@+id/headlines_fragment"
              android:layout_weight="1"
              android:layout_width="0dp"
              android:layout_height="match_parent" />

    <fragment android:name="com.example.android.fragments.ArticleFragment"
              android:id="@+id/article_fragment"
              android:layout_weight="2"
              android:layout_width="0dp"
              android:layout_height="match_parent" />

</LinearLayout>
```

> 提示：关于更多如何根据屏幕不同大小创建布局文件的内容，请参阅 [Supporting Different Screen Sizes](https://developer.android.com/training/multiscreen/screensizes.html).

然后将该layout文件应用在你的Activity中：

```java
import android.os.Bundle;
import android.support.v4.app.FragmentActivity;

public class MainActivity extends FragmentActivity {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.news_articles);
    }
}
```

如果你在使用 [v7 appcompat library](https://developer.android.com/tools/support-library/features.html#v7-appcompat), 你的Activity应该继承自 `AppCompatActivity`,它是 FragmentActivity`的子类. 了解更多的信息请参考 [Adding the App Bar](https://developer.android.com/training/appbar/index.html)。

> 注意：通过在布局XML文件中定义Fragment将Fragment添加到Activity布局时，无法在运行时删除Fragment。如果您计划在用户交互期间替换Fragment，则必须在Activity首次启动时将Fragment添加到Activity中，如[构建灵活的UI](./构建灵活的UI.md)中所示。