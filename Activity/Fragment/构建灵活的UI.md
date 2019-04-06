# 构建灵活的UI

[原文(英文)地址](https://developer.android.com/training/basics/fragments/fragment-ui)

在设计应用程序以支持各种屏幕尺寸时，您可以在不同的布局配置中重复使用Fragment，以根据可用的屏幕空间优化用户体验。

例如，在手机设备上，一次只显示一个Fragment用于单窗格用户界面可能是合适的。相反，您可能希望在具有更宽屏幕尺寸的平板电脑上并排设置Fragment，以向用户显示更多信息。

![fragments-screen-mock](https://ws1.sinaimg.cn/large/006tNc79gy1g1sxcmsejmj30ga06dq3h.jpg)

<center>图一：两个Fragment，以不同的配置显示，用于不同屏幕尺寸的相同Activity。在大屏幕上，两个Fragment并排放置，但在手机设备上，一次只能放入一个Fragment，因此Fragment必须在用户导航时互相替换。</center>

FragmentManager提供的方法允许你在Activity运行时添加、移除、替换Fragment以创造动态的体验。

关于Fragment的更多实现信息请参阅一下资源：

- [Fragment](./)
- [Supporting Tablets and Handsets](https://developer.android.com/guide/practices/tablets-and-handsets.html)

## 在Activity运行时添加Fragment

相比于上一个文档[创建Fragment](./创建Fragment.md)中所述的在布局文件中定义Activity的Fragment，您可以在Activity运行时期间向Activity添加Fragment。如果您计划在Activity期间更改Fragment，则必须采用此种方法（在Activity运行时期间向Activity添加Fragment）。

要执行诸如添加或删除Fragment之类的事务，必须使用FragmentManager创建FragmentTransaction，它提供API以添加，删除，替换和执行其他Fragment事务。

如果您的Activity允许删除和替换Fragment，则应在Activity的onCreate（）方法中将初始Fragment添加到Activity中。

处理Fragment时的一个重要规则 （特别是在运行时添加Fragment时）： 您的Activity布局必须包含一个容器视图，您可以在其中插入Fragment。

以下布局是上一个文档中显示的布局的替代方案，一次只显示一个Fragment。为了将一个Fragment替换为另一个Fragment，Activity的布局包括一个空的FrameLayout，它充当Fragment容器。

请注意，文件名与上一课中的布局文件相同，但布局目录没有"large"限定符，因此当设备屏幕小于"大(large)"时，将使用此布局，因为屏幕不适合同时放置两个Fragment。

res/layout/news_articles.xml:

```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/fragment_container"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

在您的Activity中，调用getSupportFragmentManager（）以使用Support Library APIs获取FragmentManager。然后调用beginTransaction（）创建FragmentTransaction并调用add（）来添加Fragment。

您可以使用相同的FragmentTransaction为Activity执行多个Fragment事务，准备好进行更改后，必须调用commit（）。

例如，以下是如何将Fragment添加到先前的布局：

```java
import android.os.Bundle;
import android.support.v4.app.FragmentActivity;

public class MainActivity extends FragmentActivity {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.news_articles);

        // Check that the activity is using the layout version with
        // the fragment_container FrameLayout
        if (findViewById(R.id.fragment_container) != null) {

            // However, if we're being restored from a previous state,
            // then we don't need to do anything and should return or else
            // we could end up with overlapping fragments.
            if (savedInstanceState != null) {
                return;
            }

            // Create a new Fragment to be placed in the activity layout
            HeadlinesFragment firstFragment = new HeadlinesFragment();

            // In case this activity was started with special instructions from an
            // Intent, pass the Intent's extras to the fragment as arguments
            firstFragment.setArguments(getIntent().getExtras());

            // Add the fragment to the 'fragment_container' FrameLayout
            getSupportFragmentManager().beginTransaction()
                    .add(R.id.fragment_container, firstFragment).commit();
        }
    }
}
```

因为Fragment已在运行时添加到FrameLayout容器中(而不是使用<fragment\>元素在Activity的布局中定义它) ，则该Activity可以删除Fragment并将其替换为另一个Fragment。

## 用另一个Fragment替换当前Fragment

替换Fragment的过程类似于添加Fragment，但需要使用replace（）方法而不是add（）。

请记住，当您执行Fragment事务（例如替换或删除Fragment事务）时，通常允许用户向后导航并“撤消”更改。要允许用户在Fragment事务中向后导航，必须在提交FragmentTransaction之前调用addToBackStack（）。

>  注意：删除或替换Fragment并将事务添加到后台堆栈时，被移除的Fragment将会Stopped而不是Destroyed。如果用户导航回还原Fragment，则会重新启动。如果不将事务添加到后台堆栈，则在删除或替换时Destroy该Fragment。

用另一个Fragment替换一个Fragment的示例：

```java
// Create fragment and give it an argument specifying the article it should show
ArticleFragment newFragment = new ArticleFragment();
Bundle args = new Bundle();
args.putInt(ArticleFragment.ARG_POSITION, position);
newFragment.setArguments(args);

FragmentTransaction transaction = getSupportFragmentManager().beginTransaction();

// Replace whatever is in the fragment_container view with this fragment,
// and add the transaction to the back stack so the user can navigate back
transaction.replace(R.id.fragment_container, newFragment);
transaction.addToBackStack(null);

// Commit the transaction
transaction.commit();
```

addToBackStack（）方法采用可选的字符串参数，该参数指定事务的唯一名称。除非您计划使用FragmentManager.BackStackEntry API执行高级Fragment操作，否则不需要该名称。

> 注意：ragment add replace 区别
>
> 1. replace 先删除容器中的内容，再添加
> 2. add直接添加，可以配合hide适用