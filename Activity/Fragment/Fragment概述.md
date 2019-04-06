# Fragment概述

[原文(英文)地址](https://developer.android.com/guide/components/fragments#java)

[Fragment](https://developer.android.com/reference/android/support/v4/app/Fragment.html)表示[FragmentActivity](https://developer.android.com/reference/android/support/v4/app/FragmentActivity.html)中的行为或用户界面的一部分。您可以在单个Activity中组合多个以构建多窗格UI，并在多个Activity中重用Fragment。您可以将Fragment视为Activity的模块化部分，它具有自己的生命周期，接收自己的输入事件，并且可以在Activity运行时添加或删除（有点像“子Activity”，您可以在不同的Activity中重用）。

Fragment必须始终托管在Activity中，Fragment的生命周期直接受宿主Activity生命周期的影响。例如，当Activity暂停时，其中的所有Fragment也都会暂停，当Activity被破坏时，所有Fragment也都会被破坏。但是，当一个Activity正在运行时（它处于resumed的生命周期状态），您可以独立操作每个Fragment，例如添加或删除它们。执行此类Fragment事务时，还可以将其添加到由Activity管理的回退栈中—— Activity中的每个回退栈条目都是发生的Fragment事务的记录。回退栈允许用户通过按“返回”按钮来反转Fragment事务（向后导航）。

将Fragment添加为Activity布局的一部分时，它将位于Activity视图层次结构内的ViewGroup中，并且Fragment定义自己的视图布局。您可以通过将Activity的布局文件中的Fragment声明为<fragment\>元素，或者通过将其添加到现有ViewGroup，从应用程序代码中将Fragment插入到Activity布局中。

本文档描述了如何构建应用程序以使用Fragment，包括：Fragment添加到Activity的后台堆栈时如何维持其状态、在Activity中和别的Activity以及本Activity的其他Fragment通信，为Activity的应用栏做出贡献等等。

有关处理生命周期的信息，包括有关最佳实践的指导，请参阅以下资源：

- [Handling Lifecycles with Lifecycle-Aware Components](https://developer.android.com/topic/libraries/architecture/lifecycle.html)
- [Guide to App Architecture](https://developer.android.com/topic/libraries/architecture/guide.html)
- [Supporting Tablets and Handsets](https://developer.android.com/guide/practices/tablets-and-handsets.html)

## Fragment的设计哲学

Android在Android 3.0（API 11）中引入了Fragment，主要是为了在大屏幕（如平板电脑）上支持更加动态和灵活的UI设计。由于平板电脑的屏幕比手机屏幕大得多，因此组合和交换UI组件的空间更大。Fragment允许此类设计，而无需您管理对视图层次结构的复杂更改。通过将Activity的布局划分为Fragment，您可以在运行时修改Activity的外观，并将这些更改保留在由Activity管理的后台堆栈中。它们现在可以通过 [fragment support library](https://developer.android.com/topic/libraries/support-library/packages.html#v4-fragment)广泛使用。

例如，新闻应用程序可以使用一个Fragment显示左侧文章列表，另一个Fragment显示右侧文章 ——两个Fragment并排显示在一个Activity中，每个Fragment都有自己的生命周期及回调方法并处理自己的用户输入事件。因此，用户可以选择文章并在同一Activity中阅读，而不是使用一个Activity来选择文章和另一个Activity来阅读文章，如图1中的平板电脑布局所示。

![fragments](https://ws4.sinaimg.cn/large/006tNc79gy1g1ryz1ypw6j30fk08zjru.jpg)



<center>图一：将由Fragment定义的两个UI模块组合成平板电脑设计的一个Activity，但是在手机上分开到两个Activity。</center>

您应该将每个Fragment设计为模块化和可重用的Activity组件。也就是说，因为每个Fragment使用自己的生命周期回调定义自己的布局和自己的行为，所以可以在不同Activity中包含一个Fragment，因此您应该将您的Fragment设计为可重用的，并避免在一个Fragment中直接操作另一个Fragment。这一点尤其重要，因为模块化Fragment允许您根据不同屏幕大小更改Fragment的不同组合。在设计支持平板电脑和手机的应用程序时，您可以在不同的布局配置中重复使用Fragment，以根据可用的屏幕空间优化用户体验。例如，在手机上，当多个Fragment不能存在于同一Activity时，可能需要分离Fragment以提供单窗格UI。

我们还是以使用新闻应用程序为例，当在平板电脑大小级别的设备上运行时，应用程序可以在Activity A中嵌入两个Fragment。但是，在手机大小级别的屏幕上，两个Fragment都没有足够的空间，因此Activity A仅包含文章列表的片段，当用户选择文章时，它启动 Activity B，Activity B中包括要展示文章的第二个Fragment。因此，该应用程序通过重复使用不同组合的Fragment来支持平板电脑和手机，如图1所示。

有关使用不同屏幕配置的不同Fragment组合设计应用程序的详细信息，请参阅 [Screen Compatibility Overview](https://developer.android.com/guide/practices/screens_support.html)。

## 创建一个Fragment

![fragment_lifecycle](https://ws4.sinaimg.cn/large/006tNc79gy1g1rz6s42f0j308t0njjs8.jpg)

<center>图二：Fragment的生命周期（宿主Activity正在运行）</center>

为了创建一个Fragment，你必须创建一个Fragment的子类(或者已经存在的Fragment的子类)。Fragment和Activity有类似的代码逻辑，其回调方法与Activity的回调方法类似，比如onCreate()，onStart()，onPause()，onStop()。事实上，如果你想将一个没有使用Fragment的app转化为使用Fragment的app，你也许可以简单地将你Activity毁掉方法里的代码移动到Fragment对应的回调方法中去即可。

一般来说，你至少需要实现以下生命周期对应的方法：

- onCreate()

  > 创建Fragment的时候系统会调用此方法。在你的实现中你应该在此方法中初始化一些你想要在Fragment处于paused、stooped状态下需要的元素。

- onCreateView()

  > 在Fragment第一次绘制其用户界面的时候，系统会调用此方法。要绘制该Fragment的用户界面，必须在此方法中返回一个View对象，该View对象是Fragment布局的根View。如果该Fragment不提供UI，则可以返回null。

- onPause()

  > 系统将此方法称为用户离开Fragment的第一个标志(尽管不总是意味着该Fragment被Destroyed)。这通常是你应该保存在当前用户会话之外还想保存的数据(因为用户可能不再回来了)

大多数程序中的Fragment至少应该实现以上三个方法，但是你还可以使用一些其他的回调来处理Fragment生命周期的各个阶段。有关Fragment生命周期的详细介绍，请参考[Handling the Fragment Lifecycle](https://developer.android.com/guide/components/fragments#Lifecycle).

请注意，实现依赖组件的生命周期操作的代码应放在组件本身中，而不是直接放在Fragment回调实现中。请参阅 [Handling Lifecycles with Lifecycle-Aware Components](https://developer.android.com/topic/libraries/architecture/lifecycle.html)，以了解如何使您的从属组件感知生命周期。

这里再介绍几个Fragment的子类，你可能想要继承这些基类而不是直接继承Fragment：

- DialogFragment

  > 显示一个悬浮的dialog。使用该类创建对话框是在Activity类中使用dialog helper的一个很好的替代方法，因为您可以将Fragment对话框合并到由Activity管理的Fragment的回退栈中，从而允许用户返回到已解散（dismissed）的Fragment。

- ListFragment

  > 显示由adapter管理的list（例如SimpleCursorAdapter），类似于ListActivity。它提供了几种管理列表视图的方法，例如onListItemClick（）回调来处理点击事件。 （请注意，显示list的首选方法是使用RecyclerView而不是ListView。在这种情况下，您需要创建一个在其布局中包含RecyclerView的Fragmrnt。请参阅 [Create a List with RecyclerView](https://developer.android.com/guide/topics/ui/layout/recyclerview.html) 以了解更多内容。）



## 添加一个用户界面

Fragment通常用作Activity用户界面的一部分，为Activity提供自己的布局。

要为Fragment提供布局，必须实现onCreateView（）回调方法，当Fragment绘制其布局时，Android系统会调用该方法。您对此方法的实现必须返回一个View，它是Fragment布局的根（root view）。

> 注意：如果你的Fragment事ListFragment的子类，那么默认情况下你Fragment的onCreatView()返回一个ListView，所以你不再需要实现该方法。

为了从onCreateView()中返回一个View，你可以从layout resource下定义的xml文件中inflate到View。为了帮助你做这件事，onCreateView()中提供了一个LayoutInflater对象。

比如，有一个Fragment的子类需要从example_fragment.xml文件加载View：

- kotlin

  ```kotlin
  class ExampleFragment : Fragment() {
  
      override fun onCreateView(
              inflater: LayoutInflater,
              container: ViewGroup?,
              savedInstanceState: Bundle?
      ): View {
          // Inflate the layout for this fragment
          return inflater.inflate(R.layout.example_fragment, container, false)
      }
  }
  ```

- java

  ```java
  public static class ExampleFragment extends Fragment {
      @Override
      public View onCreateView(LayoutInflater inflater, ViewGroup container,
                               Bundle savedInstanceState) {
          // Inflate the layout for this fragment
          return inflater.inflate(R.layout.example_fragment, container, false);
      }
  }
  ```

> 注意：在上例中，R.layout.example_fragment是一个存储在应用resources资源下的名为example_fragment.xml的layout资源的引用。关于如何创建layout xml文件的更多信息，请参阅 [User Interface](https://developer.android.com/guide/topics/ui/index.html) 

onCreateView()中的container参数是父ViewGroup(来自Activity 的layout)，你的Fragment就插入在该ViewGroup中。saveInstanceState参数是一个Bundle对象，提供了Fragment前一个实例的数据(在 [Handling the Fragment Lifecycle](https://developer.android.com/guide/components/fragments#Lifecycle)中更详细地讨论了如何恢复Fragment之前的状态)



inflate()方法接收三个参数：

- 想要inflate的layout文件的id
- 作为父View的ViewGroup。传递container参数非常重要，该参数使得系统可以将布局参数应用于inflated布局的根视图，其所在的父视图指定。
- 一个布尔值，指示在inflate期间是否应将inflated的布局附加到ViewGroup（第二个参数）。 （在这种情况下，该参数的值为false，因为系统已经将inflated的布局插入到container中 - 传递true会在最终布局中创建冗余的ViewGroup）

现在你已经学会如何为创建的Fragment添加布局了，下一节介如何将Fragment添加到Activity中。

### 将Fragment添加到Activity中

通常，一个Fragment作为一个宿主Activity的一部分，这里有两种方法将Fragment添加到Activity中：

- 在activity的layout文件中声明fragment

  这种情况下，你可以将fragment当做一个view一样为他指定参数，比如：

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
      android:orientation="horizontal"
      android:layout_width="match_parent"
      android:layout_height="match_parent">
      <fragment android:name="com.example.news.ArticleListFragment"
              android:id="@+id/list"
              android:layout_weight="1"
              android:layout_width="0dp"
              android:layout_height="match_parent" />
      <fragment android:name="com.example.news.ArticleReaderFragment"
              android:id="@+id/viewer"
              android:layout_weight="2"
              android:layout_width="0dp"
              android:layout_height="match_parent" />
  </LinearLayout>
  ```

  在<fragment\>中的android：name属性指定要在布局中实例化的Fragment类。

  当系统创建此Activity布局时，它会实例化布局中指定的每个Fragment，并为每个Fragment调用onCreateView（）方法，以检索每个Fragment的布局。系统直接插入Fragment返回的View代替<fragment\>元素。

  > 注意：每个Fragment都需要一个唯一的标识符，如果重新启动Activity，系统可以使用该标识符来恢复Fragment（您可以使用它来捕获Fragment以执行事务，例如删除它）。有两种方法可以为片段提供ID：
  >
  > - android：id
  > - android：tag

- 使用代码将Fragment添加到已经存在的ViewGroup中

  在你Activity运行的任意时刻，你都可以将Fragment添加到你Activity的布局中，你只需要指定一个ViewGroup来防止Fragment。

  要在Activity进行Fragment事物操作(比如添加、删除、替换Fragment)，必须使用FragmentTransaction中的API，你可以从FragmentActivity中获取FragmentTransaction的实例，比如：

  - kotlin

    ```kotlin
    val fragmentManager = supportFragmentManager
    val fragmentTransaction = fragmentManager.beginTransaction()
    ```

  - java

    ```java
    ExampleFragment fragment = new ExampleFragment();
    fragmentTransaction.add(R.id.fragment_container, fragment);
    fragmentTransaction.commit();
    ```

  add（）方法中传递的第一个参数是ViewGroup对应的id，第二个参数是被添加的Fragment对象。

  当你使用FragmentTransaction作出更改后，你必须调用commit()去让你的更改生效。



## 管理Fragment

为了管理你Activity中的Fragment，你必须使用FragmentManager。为了得到它的实例，你应该在你的Activity中调用getSupportFragmentManager()方法。

你可以用FragmentManager做的操作包括：

- 获取Activity中已经存在的Fragment实例，使用findFragmentById()(对于声明在layout布局文件中的Fragment)，或者findFragmentByTag()(对于没有声明在layout布局文件中的Fragment)
- 从回退栈中弹出Fragment，使用popBackStack(就像用户按了Back键)
- 注册一个监听回退栈变化的监听器，使用addOnBackStackChangedListener()

更多的内容请参考[FragmentManager](https://developer.android.com/reference/android/support/v4/app/FragmentManager.html)类的文档。

​      

如上一节介绍了，你也可以使用FragmentManager开启一个FragmentTransaction以执行你的事物，比如添加或者移除Fragment。

## 执行Fragment事物

在您的Activity中使用Fragment的一个很棒的特点是能够添加，删除，替换和执行其他操作，以响应用户交互。您为Activity提交的每组更改都称为事务，您可以使用FragmentTransaction中的API执行一项更改。您还可以将每个事务保存到由Activity管理的后台，允许用户向后导航Fragment的更改（类似于向后导航活动）。

你可以使用FragmentManager开启一个FragmentTransaction事物：

- kotlin

  ```kotlin
  val fragmentManager = supportFragmentManager
  val fragmentTransaction = fragmentManager.beginTransaction()
  ```

- java

  ```java
  FragmentManager fragmentManager = getSupportFragmentManager();
  FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
  ```

每个事务都是您要同时执行的一组更改。您可以使用add（），remove（）和replace（）等方法设置要为给定事务执行的所有更改。然后，要将事务应用于Activity，必须调用commit（）。

但是，在调用commit（）之前，可能需要调用addToBackStack（），以便将事务添加到Fragment事务的回退栈中。此后回退栈由Activity管理，并允许用户通过按“Back”按钮返回到先前的Fragment状态。

比如，这里展示了如何使用一个fragment替换另一个fragment，同时保留前一个回退栈的状态：

- kotlin

  ```kotlin
  val newFragment = ExampleFragment()
  val transaction = supportFragmentManager.beginTransaction()
  transaction.replace(R.id.fragment_container, newFragment)
  transaction.addToBackStack(null)
  transaction.commit()
  ```

- java

  ```java
  // Create new fragment and transaction
  Fragment newFragment = new ExampleFragment();
  FragmentTransaction transaction = getSupportFragmentManager().beginTransaction();
  
  // Replace whatever is in the fragment_container view with this fragment,
  // and add the transaction to the back stack
  transaction.replace(R.id.fragment_container, newFragment);
  transaction.addToBackStack(null);
  
  // Commit the transaction
  transaction.commit();
  ```

在这个例子中，newFragment替换当前在R.id.fragment_container ID标识的布局容器中的任何Fragment（如果有）。通过调用addToBackStack（），替换事务将保存到回退栈，因此用户可以通过按“Back”按钮来反转事务并恢复前一个Fragment。

FragmentActivity然后通过onBackPressed（）自动从回退栈中检索Fragment。

如果向事务添加多个更改（例如另一个add（）或remove（））并调用addToBackStack（），则在调用commit（）之前应用的所有更改都将作为单个事务添加到回退栈中，点击Back按钮时的效果和之前的顺序相反。

将更改添加到FragmentTransaction的顺序无关紧要，除了：

- 你必须在最后调用commit()
- 如果要将多个Fragment添加到同一容器中，则添加它们的顺序将决定它们在视图层次结构中的显示顺序。

调用commit（）不会立即执行事务。相反，它会在线程能够执行时将其安排在activity的UI线程（“主”线程）上运行。但是，如果需要，您可以从UI线程调用executePendingTransactions（）以立即执行commit（）提交的事务。除非事务是其他线程中任务的依赖项，否则通常不需要这样做。

> 警告：只有在Activity保存其状态（用户离开Activity时）之前，才能使用commit（）提交事务。如果在该点之后尝试提交，则抛出异常。这是因为如果需要恢复Activity，则提交后的状态可能会丢失。对于可以丢失提交的情况，请使用commitAllowingStateLoss（）。

## 与Activity通信

尽管Fragment是作为独立于FragmentActivity的对象实现的，并且可以在多个Activity中使用，但是Fragment的给定实例直接与托管它的Activity相关联。

具体来说，Fragment可以使用getActivity（）访问FragmentActivity实例，并轻松执行任务，例如在Activity布局中查找View:

- Kotlin

  ```kotlin
  val listView: View? = activity?.findViewById(R.id.list)
  ```

- java

  ```java
  View listView = getActivity().findViewById(R.id.list);
  ```

类似的，你的Activity也可以由FragmentManager的findFragmentById()方法或者findFragmentByTag()方法获取Fragment实例，进而调用该Fragment的的方法，比如：

- kotlin

  ```kotlin
  val fragment = supportFragmentManager.findFragmentById(R.id.example_fragment) as ExampleFragment
  ```

- java

  ```java
  ExampleFragment fragment = (ExampleFragment) getSupportFragmentManager().findFragmentById(R.id.example_fragment);
  ```



### 创建Activity的事件回调

有时你也许需要从fragment中发送数据或事件给宿主activity或者其他寄存在宿主activity中的fragment。为了发送数据，可以创建一个共享的ViewModel，如[ViewModel guide](https://developer.android.com/topic/libraries/architecture/viewmodel.html)中fragment之间共享数据部分所述。如果需要传播无法使用ViewModel处理的事件，则可以在Fragment内定义回调接口，并要求宿主Activity实现它。当Activity通过接口收到回调时，它可以根据需要与将数据与布局中的其他Fragment共享。

例如，如果新闻应用程序在Activity中有两个Fragment，一个用于显示文章列表（FragmentA）而另一个用于显示文章详情（FragmentB），那么FragmentA必须在选择列表项时告知Activity FragmentB应该显示的文章。在这种情况下，可以在FragmentA中声明OnArticleSelectedListener接口：

- kotlin

  ```kotlin
  public class FragmentA : ListFragment() {
      ...
      // Container Activity must implement this interface
      interface OnArticleSelectedListener {
          fun onArticleSelected(articleUri: Uri)
      }
      ...
  }
  ```

  

- java

  ```java
  public static class FragmentA extends ListFragment {
      ...
      // Container Activity must implement this interface
      public interface OnArticleSelectedListener {
          public void onArticleSelected(Uri articleUri);
      }
      ...
  }
  ```



然后宿主Activity应该实现(implement) OnArticleSelectedListener接口并且实现onArticleSelected()以将Fragment A发送的事件通知给fragment B，为了确保宿主Activity实现了该接口，Fragment A的onAttach()方法(该方法会在系统将Fragment加入到Activity中时调用)中通过强制转化传递给onAttach()的Activity来实例化OnArticleSelectedListener的实例：

- kotlin

  ```kotlin
  public class FragmentA : ListFragment() {
  
      var listener: OnArticleSelectedListener? = null
      ...
      override fun onAttach(context: Context) {
          super.onAttach(context)
          listener = context as? OnArticleSelectedListener
          if (listener == null) {
              throw ClassCastException("$context must implement OnArticleSelectedListener")
          }
  
      }
      ...
  }
  ```

- java

  ```java
  public static class FragmentA extends ListFragment {
      OnArticleSelectedListener listener;
      ...
      @Override
      public void onAttach(Context context) {
          super.onAttach(context);
          try {
              listener = (OnArticleSelectedListener) context;
          } catch (ClassCastException e) {
              throw new ClassCastException(context.toString() + " must implement OnArticleSelectedListener");
          }
      }
      ...
  }
  ```

如果宿主Activity没有实现该接口，则Fragment A会抛出一个ClassCastException。如果宿主Activity实现了该接口，则FragmentA就会有一个listener就会作为实现了OnArticleSelectedListener接口的Activity的引用，所以Fragment A可以通过调用在OnArticleSelectedListener中定义的方法以发送消息到宿主Activity。比如，Fragment A是ListFragment的子类，当用户点击了list的item，系统会回调Fragment A中的onListItemClick()方法，在此时可以通过回调listener的onArticleSelected()方法以将事件传递给宿主Activity：

- kotlin

  ```kotlin
  public class FragmentA : ListFragment() {
  
      var listener: OnArticleSelectedListener? = null
      ...
      override fun onListItemClick(l: ListView, v: View, position: Int, id: Long) {
          // Append the clicked item's row ID with the content provider Uri
          val noteUri: Uri = ContentUris.withAppendedId(ArticleColumns.CONTENT_URI, id)
          // Send the event and Uri to the host activity
          listener?.onArticleSelected(noteUri)
      }
      ...
  }
  ```

  

- java

  ```java
  public static class FragmentA extends ListFragment {
      OnArticleSelectedListener listener;
      ...
      @Override
      public void onListItemClick(ListView l, View v, int position, long id) {
          // Append the clicked item's row ID with the content provider Uri
          Uri noteUri = ContentUris.withAppendedId(ArticleColumns.CONTENT_URI, id);
          // Send the event and Uri to the host activity
          listener.onArticleSelected(noteUri);
      }
      ...
  }
  ```

onListItemClick()中的形参id是被点击的item的缩在行的下标，宿主Activity(或者其他Fragment)可以通过它从应用的ContentProvider中获取文章详情。

关于ContentProvider的更多信息请参阅 [Content Providers](https://developer.android.com/guide/topics/providers/content-providers.html)。



## 向App Bar中添加条目(items)

您的Fragment可以通过实现onCreateOptionsMenu（）将menu items提供给Activity的“ [Options Menu](https://developer.android.com/guide/topics/ui/menus.html#options-menu)”（以及应用程序栏）。但是，为了使此方法可以被调用，您必须在onCreate（）中调用setHasOptionsMenu（），以指示该Fragment要将item添加到Options Menu。否则，Fragment不会接收对onCreateOptionsMenu（）的调用。

然后，您从Fragment添加到“Options Menu”的任何项目都将附加到现有menu items。当选择菜单项时，Fragment还接收对onOptionsItemSelected（）的回调。

您还可以在Fragment布局中注册View，以通过调用registerForContextMenu（）来提供上下文菜单（context menu）。当用户打开上下文菜单时，Fragment将接收对onCreateContextMenu（）的调用。当用户选择一个项目（item）时，该Fragment接收对onContextItemSelected（）的调用。

> 注意：虽然您的Fragment会为其添加的每个菜单项添加项目选择的回调，但当用户选择菜单项时，Activity首先接收相应的回调。如果Activity的项目选择回调的实现方法不处理所选项目，则事件将传递给Fragment的回调。Options Menu 和 context menus.也是如此。

关于菜单(menu)的更多信息，请参阅 [Menus](https://developer.android.com/guide/topics/ui/menus.html) 开发者手册以及 [App Bar](https://developer.android.com/training/appbar/index.html) 类.

## 监听Fragment的生命周期

![activity_fragment_lifecycle](https://ws4.sinaimg.cn/large/006tNc79gy1g1suyp424fj309g0iraac.jpg)

<center>图三：activity生命周期对fragment生命周期的影响</center>

管理Fragment的生命周期和管理Activity的生命周期类似，Fragment和Activity一样含有以下三个状态：

- Resumed

  Fragment可见（正在运行的宿主Activity中）

- Paused

  另一个Activity正处于前台且占有焦点但是宿主Activity依然可见(没有焦点，被部分覆盖)

- Stopped

  Fragment不可见。宿主Activity被停止(stooped)或者Fragment被从Activity中移除但已经被加入到了回退栈。被停止的Fragment依然是活着的(alive)(所有状态和成员信息由系统保留)，但是，他不在对用户可见，并且如果宿主Activity被杀死(kill)，则他也会被杀死。



和Activity一样，您可以使用onSaveInstanceState（Bundle），ViewModel和持久化本地存储的组合，跨配置更改（across configuration changes ）保留Fragment的UI状态。要了解有关保留UI状态的更多信息，请参阅[Saving UI States](https://developer.android.com/topic/libraries/architecture/saving-states.html)。

Activity和Fragment生命周期的最显着的差异是如何将其存储在其各自的后台堆栈中。默认情况下，Activity停止（stopped）时被置于由系统管理的Activity的后堆栈中（以便用户可以使用“Back”按钮导航回到它，如 [理解Task和回退栈](./Activity/理解Task和回退栈.md) 中所述）。但是，只有当您在执行删除Fragment的事务时通过调用addToBackStack（）显式请求保存实例时，系统才会将Fragment放入由宿主Activity管理的后台堆栈中。

请参阅[Activity生命周期](./Activity/Activity的生命周期.md)和[Handling Lifecycles with Lifecycle-Aware Components](https://developer.android.com/topic/libraries/architecture/lifecycle.html)，以了解有关Activity生命周期和管理的更多信息。

> 警告：如果你需要在你的Fragment中使用Context对象，你可以调用getContext()方法。但是，仅仅当你的Fragment附加到（attached）宿主Activity之后你才可以谨慎地调用getContext()。当Fragment尚未附加到Activity中，或者其在生命周期结束的时候被Detached，getContext()hi返回null；



## 与Activity的声明周期进行协调

Fragment所在的宿主Activity的生命周期会直接影响Fragment的生命周期，因此Activity的每个生命周期回调都会为每个Fragment产生类似的回调。例如，当Activity收到onPause（）时，Activity中的每个Fragment都会收到onPause（）。

但是，Fragment有一些额外的生命周期回调，它们处理与Activity的唯一交互，以便执行构建和销毁Fragment的UI等操作。这些额外的回调方法是：

- `onAttach()`

  当Fragment与Activity关联时调用(宿主Activity会被当做形参传入)

- `onCreateView()`

  调用以创建与Fragment关联的视图层次结构。

- `onActivityCreated()`

  当Activity的`onCreate()` 方法返回时被调用。

- `onDestroyView()`

  当与Fragment关联的视图层次结构被移除时调用。

- `onDetach()`

  当Fragment与Activity解除关联时调用。

图3说明了Fragment生命周期的流程，因为它受宿主Activity的影响。在此图中，您可以看到Activity的每个连续状态如何确定Fragment可以接收哪些回调方法。例如，当Activity收到onCreate（）回调时，Activity中的Fragment只接收onActivityCreated（）回调。

一旦Activity达到Resumed状态，您就可以自由地向Activity添加和删除Fragment。因此，只有当Activity处于Resumed状态时，Fragment的生命周期才能独立地改变。

但是，当Activity离开Resumed状态时，Fragment再次被Activity推送到其自己的生命周期。

## 例子

为了将本文档中讨论的所有内容放在一起，下面是使用两个Fragment创建双窗格布局的Activity示例。下面的Activity包括一个Fragment，用于显示莎士比亚戏剧标题列表，另一个用于显示从列表中选择的播放摘要。它还演示了如何根据屏幕配置提供不同的Fragment配置。

> 注意：完整的源码见 [sample app ](https://github.com/aosp-mirror/platform_development/blob/master/samples/ApiDemos/src/com/example/android/apis/app/FragmentLayout.java)，该源码展示了如何使用FragmentLayout

- kotlin

  ```kotlin
  override fun onCreate(savedInstanceState: Bundle?) {
      super.onCreate(savedInstanceState)
  
      setContentView(R.layout.fragment_layout)
  }
  ```

- java

  ```java
  @Override
  protected void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
  
      setContentView(R.layout.fragment_layout);
  }
  ```

fragment_layout.xml布局文件如下：

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="horizontal"
    android:layout_width="match_parent" android:layout_height="match_parent">

    <fragment class="com.example.android.apis.app.FragmentLayout$TitlesFragment"
            android:id="@+id/titles" android:layout_weight="1"
            android:layout_width="0px" android:layout_height="match_parent" />

    <FrameLayout android:id="@+id/details" android:layout_weight="1"
            android:layout_width="0px" android:layout_height="match_parent"
            android:background="?android:attr/detailsElementBackground" />

</LinearLayout>
```



使用此layout，系统会在Activity加载layout时实例化TitlesFragment（列出播放标题），而FrameLayout（用于显示播放摘要的Fragment）会占据屏幕右侧的空间，但是起初仍然是空的。正如您将在下面看到的那样，直到用户从列表中选择一个Fragment放入FrameLayout中。

但是，并非所有屏幕都足够宽，以便并排显示播放列表和摘要。因此，上面的布局仅用于横向屏幕配置，方法是将其保存在res / layout-land / fragment_layout.xml中。

因此，当屏幕处于纵向时，系统将应用以下布局，该布局保存在res / layout / fragment_layout.xml中：

```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent" android:layout_height="match_parent">
    <fragment class="com.example.android.apis.app.FragmentLayout$TitlesFragment"
            android:id="@+id/titles"
            android:layout_width="match_parent" android:layout_height="match_parent" />
</FrameLayout>
```

此布局仅包含TitlesFragment。这意味着，当设备处于纵向方向时，只能看到播放标题列表。因此，当用户单击此配置中的列表项时，应用程序将启动一个新Activity以显示摘要，而不是加载第二个Fragment。

接下来，您可以看到如何在Fragment类中完成此操作。首先是TitlesFragment，它显示了莎士比亚戏剧名单。此Fragment继承了ListFragment并依赖它来处理大多数列表视图工作。

在查看此代码时，请注意当用户单击列表项时有两种可能的行为：根据两个布局中的哪一个处于生效状态，它可以创建并显示一个新Fragment以显示同一Activity中的详细信息（添加片段到FrameLayout），或者开始一个新的Activity（Fragment中的内容可以显示在该Activity中）。

- kotlin

  ```kotlin
  class TitlesFragment : ListFragment() {
  
      private var dualPane: Boolean = false
      private var curCheckPosition = 0
  
      override fun onActivityCreated(savedInstanceState: Bundle?) {
          super.onActivityCreated(savedInstanceState)
  
          // Populate list with our static array of titles.
          listAdapter = ArrayAdapter<String>(
                  activity,
                  android.R.layout.simple_list_item_activated_1,
                  Shakespeare.TITLES
          )
  
          // Check to see if we have a frame in which to embed the details
          // fragment directly in the containing UI.
          val detailsFrame: View? = activity?.findViewById(R.id.details)
          dualPane = detailsFrame?.visibility == View.VISIBLE
  
          curCheckPosition = savedInstanceState?.getInt("curChoice", 0) ?: 0
  
          if (dualPane) {
              // In dual-pane mode, the list view highlights the selected item.
              listView.choiceMode = ListView.CHOICE_MODE_SINGLE
              // Make sure our UI is in the correct state.
              showDetails(curCheckPosition)
          }
      }
  
      override fun onSaveInstanceState(outState: Bundle) {
          super.onSaveInstanceState(outState)
          outState.putInt("curChoice", curCheckPosition)
      }
  
      override fun onListItemClick(l: ListView, v: View, position: Int, id: Long) {
          showDetails(position)
      }
  
      /**
       * Helper function to show the details of a selected item, either by
       * displaying a fragment in-place in the current UI, or starting a
       * whole new activity in which it is displayed.
       */
      private fun showDetails(index: Int) {
          curCheckPosition = index
  
          if (dualPane) {
              // We can display everything in-place with fragments, so update
              // the list to highlight the selected item and show the data.
              listView.setItemChecked(index, true)
  
              // Check what fragment is currently shown, replace if needed.
              var details = fragmentManager?.findFragmentById(R.id.details) as? DetailsFragment
              if (details?.shownIndex != index) {
                  // Make new fragment to show this selection.
                  details = DetailsFragment.newInstance(index)
  
                  // Execute a transaction, replacing any existing fragment
                  // with this one inside the frame.
                  fragmentManager?.beginTransaction()?.apply {
                      if (index == 0) {
                          replace(R.id.details, details)
                      } else {
                          replace(R.id.a_item, details)
                      }
                      setTransition(FragmentTransaction.TRANSIT_FRAGMENT_FADE)
                      commit()
                  }
              }
  
          } else {
              // Otherwise we need to launch a new activity to display
              // the dialog fragment with selected text.
              val intent = Intent().apply {
                  setClass(activity, DetailsActivity::class.java)
                  putExtra("index", index)
              }
              startActivity(intent)
          }
      }
  }
  ```

  

- java

  ```java
  public static class TitlesFragment extends ListFragment {
      boolean dualPane;
      int curCheckPosition = 0;
  
      @Override
      public void onActivityCreated(Bundle savedInstanceState) {
          super.onActivityCreated(savedInstanceState);
  
          // Populate list with our static array of titles.
          setListAdapter(new ArrayAdapter<String>(getActivity(),
                  android.R.layout.simple_list_item_activated_1, Shakespeare.TITLES));
  
          // Check to see if we have a frame in which to embed the details
          // fragment directly in the containing UI.
          View detailsFrame = getActivity().findViewById(R.id.details);
          dualPane = detailsFrame != null && detailsFrame.getVisibility() == View.VISIBLE;
  
          if (savedInstanceState != null) {
              // Restore last state for checked position.
              curCheckPosition = savedInstanceState.getInt("curChoice", 0);
          }
  
          if (dualPane) {
              // In dual-pane mode, the list view highlights the selected item.
              getListView().setChoiceMode(ListView.CHOICE_MODE_SINGLE);
              // Make sure our UI is in the correct state.
              showDetails(curCheckPosition);
          }
      }
  
      @Override
      public void onSaveInstanceState(Bundle outState) {
          super.onSaveInstanceState(outState);
          outState.putInt("curChoice", curCheckPosition);
      }
  
      @Override
      public void onListItemClick(ListView l, View v, int position, long id) {
          showDetails(position);
      }
  
      /**
       * Helper function to show the details of a selected item, either by
       * displaying a fragment in-place in the current UI, or starting a
       * whole new activity in which it is displayed.
       */
      void showDetails(int index) {
          curCheckPosition = index;
  
          if (dualPane) {
              // We can display everything in-place with fragments, so update
              // the list to highlight the selected item and show the data.
              getListView().setItemChecked(index, true);
  
              // Check what fragment is currently shown, replace if needed.
              DetailsFragment details = (DetailsFragment)
                      getSupportFragmentManager().findFragmentById(R.id.details);
              if (details == null || details.getShownIndex() != index) {
                  // Make new fragment to show this selection.
                  details = DetailsFragment.newInstance(index);
  
                  // Execute a transaction, replacing any existing fragment
                  // with this one inside the frame.
                  FragmentTransaction ft = getSupportFragmentManager().beginTransaction();
                  if (index == 0) {
                      ft.replace(R.id.details, details);
                  } else {
                      ft.replace(R.id.a_item, details);
                  }
                  ft.setTransition(FragmentTransaction.TRANSIT_FRAGMENT_FADE);
                  ft.commit();
              }
  
          } else {
              // Otherwise we need to launch a new activity to display
              // the dialog fragment with selected text.
              Intent intent = new Intent();
              intent.setClass(getActivity(), DetailsActivity.class);
              intent.putExtra("index", index);
              startActivity(intent);
          }
      }
  }
  ```

第二个Fragment，DetailsFragment展示了从TitlesFragment中选择的播放摘要：

- kotlin

  ```kotlin
      class DetailsFragment : Fragment() {
  
          val shownIndex: Int by lazy {
              arguments?.getInt("index", 0) ?: 0
          }
  
          override fun onCreateView(
                  inflater: LayoutInflater,
                  container: ViewGroup?,
                  savedInstanceState: Bundle?
          ): View? {
              if (container == null) {
                  // We have different layouts, and in one of them this
                  // fragment's containing frame doesn't exist. The fragment
                  // may still be created from its saved state, but there is
                  // no reason to try to create its view hierarchy because it
                  // isn't displayed. Note this isn't needed -- we could just
                  // run the code below, where we would create and return the
                  // view hierarchy; it would just never be used.
                  return null
              }
  
              val text = TextView(activity).apply {
                  val padding: Int = TypedValue.applyDimension(
                          TypedValue.COMPLEX_UNIT_DIP,
                          4f,
                          activity?.resources?.displayMetrics
                  ).toInt()
                  setPadding(padding, padding, padding, padding)
                  text = Shakespeare.DIALOGUE[shownIndex]
              }
              return ScrollView(activity).apply {
                  addView(text)
              }
          }
  
          companion object {
              /**
               * Create a new instance of DetailsFragment, initialized to
               * show the text at 'index'.
               */
              fun newInstance(index: Int): DetailsFragment {
                  val f = DetailsFragment()
  
                  // Supply index input as an argument.
                  val args = Bundle()
                  args.putInt("index", index)
                  f.arguments = args
  
                  return f
              }
          }
      }
  }
  ```

- java

  ```java
  public static class DetailsFragment extends Fragment {
      /**
       * Create a new instance of DetailsFragment, initialized to
       * show the text at 'index'.
       */
      public static DetailsFragment newInstance(int index) {
          DetailsFragment f = new DetailsFragment();
  
          // Supply index input as an argument.
          Bundle args = new Bundle();
          args.putInt("index", index);
          f.setArguments(args);
  
          return f;
      }
  
      public int getShownIndex() {
          return getArguments().getInt("index", 0);
      }
  
      @Override
      public View onCreateView(LayoutInflater inflater, ViewGroup container,
              Bundle savedInstanceState) {
          if (container == null) {
              // We have different layouts, and in one of them this
              // fragment's containing frame doesn't exist. The fragment
              // may still be created from its saved state, but there is
              // no reason to try to create its view hierarchy because it
              // isn't displayed. Note this isn't needed -- we could just
              // run the code below, where we would create and return the
              // view hierarchy; it would just never be used.
              return null;
          }
  
          ScrollView scroller = new ScrollView(getActivity());
          TextView text = new TextView(getActivity());
          int padding = (int)TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP,
                  4, getActivity().getResources().getDisplayMetrics());
          text.setPadding(padding, padding, padding, padding);
          scroller.addView(text);
          text.setText(Shakespeare.DIALOGUE[getShownIndex()]);
          return scroller;
      }
  }
  ```

回想一下TitlesFragment类，如果用户单击列表项并且当前布局不包含R.id.details视图（这是DetailsFragment所在的位置），则应用程序启动DetailsActivity以显示这个item的具体内容。

这是DetailsActivity，它嵌入在DetailsFragment中以在屏幕处于纵向时显示所选的播放摘要：

- kotlin

  ```kotlin
  class DetailsActivity : FragmentActivity() {
  
      override fun onCreate(savedInstanceState: Bundle?) {
          super.onCreate(savedInstanceState)
  
          if (resources.configuration.orientation == Configuration.ORIENTATION_LANDSCAPE) {
              // If the screen is now in landscape mode, we can show the
              // dialog in-line with the list so we don't need this activity.
              finish()
              return
          }
  
          if (savedInstanceState == null) {
              // During initial setup, plug in the details fragment.
              val details = DetailsFragment().apply {
                  arguments = intent.extras
              }
              supportFragmentManager.beginTransaction()
                      .add(android.R.id.content, details)
                      .commit()
          }
      }
  }
  ```

- java

  ```java
  public static class DetailsActivity extends FragmentActivity {
  
      @Override
      protected void onCreate(Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
  
          if (getResources().getConfiguration().orientation
                  == Configuration.ORIENTATION_LANDSCAPE) {
              // If the screen is now in landscape mode, we can show the
              // dialog in-line with the list so we don't need this activity.
              finish();
              return;
          }
  
          if (savedInstanceState == null) {
              // During initial setup, plug in the details fragment.
              DetailsFragment details = new DetailsFragment();
              details.setArguments(getIntent().getExtras());
              getSupportFragmentManager().beginTransaction().add(android.R.id.content, details).commit();
          }
      }
  }
  ```

请注意，如果配置是横向的，则此Activity将自动finish（），以便主Activity可以接管并在TitlesFragment旁边显示DetailsFragment。如果用户在纵向上启动DetailsActivity，但随后旋转到横向（重新启动当前活动），则会发生这种情况。



## 更多资源

Fragment在 [Sunflower](https://github.com/googlesamples/android-sunflower)这个Demo app中被广泛使用。