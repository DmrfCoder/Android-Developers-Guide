# 使用RecyclerView创建列表

[原文(英文)地址](https://developer.android.com/guide/topics/ui/layout/recyclerview)

如果您的应用需要显示含有大量数据（或经常更改的数据）的滚动元素列表，你应该使用本文描述的[RecyclerView](https://developer.android.com/reference/androidx/recyclerview/widget/RecyclerView.html)。

> 提示：你可以在Android studio中点击File->New->Fragment->Fragment(List)创建特定的模板代码，然后将该fragment添加到你的activity中。

<div class="cols">
<div class="col-1of2" style="width:260px">
  <img src="http://picture-pool.oss-cn-beijing.aliyuncs.com/2019-05-20-025619.png" width="250">
  <p class="img-caption"><b>图1.</b> 一个使用<code translate="no">RecyclerView</code>的列表</p>
</div>
<div class="col-1of2" style="max-width:260px">
  <img src="http://picture-pool.oss-cn-beijing.aliyuncs.com/2019-05-20-025539.png" width="250">
  <p class="img-caption"><b>图 2.</b> 一个同时使用了 <code translate="no">CardView</code>的列表</p>
</div>
</div>

如果你想要创建一个像图2那样具有卡片的列表，请同时使用[Create a Card-based Layout](https://developer.android.com/guide/topics/ui/layout/cardview.html)中描述的`CardView`。

如果你想阅读更多关于`RecycleView`的示例代码，请访问[RecyclerView Sample App](https://github.com/googlesamples/android-RecyclerView/)。

## RecyclerView概述

RecyclerView控件是一个更高级、更灵活版本的ListView。

在RecyclerView中，若干不同的组件可以工作在一起以展示你的数据，用户界面的整体容器是您添加到layout的RecyclerView对象。 RecyclerView使用您提供的布局管理器中提供的View填充自己，这里的布局管理器指的是我们的标准布局管理器之一（例如LinearLayoutManager或GridLayoutManager），或您自己自定义的布局管理器。

列表中的View由ViewHolder对象表示。这些对象是您通过继承RecyclerView.ViewHolder定义的类的实例。每个ViewHolder对象负责显示View中的单个项目（item）。例如，如果您的列表显示音乐集，则每个ViewHolder对象可能代表一个专辑。 RecyclerView仅会创建当前显示在屏幕上的列表部分所需的ViewHolder，以及一些附加的View，当用户滚动列表时，RecyclerView才会获取屏幕外的View并将它们重新绑定到滚动到屏幕上的数据。

ViewHolder对象是被一个adapter对象所管理的，你可以通过继承RecyclerView.Adapter类来自定义你自己的Adapter类，这个Adapter会根据需要创建ViewHolder，同时将ViewHolder和对应的数据绑定起来。它通过将ViewHolder分配到一个位置，并调用Adapter类的onBindViewHolder（）方法来完成此操作，该方法使用ViewHolder的位置（基于其列表位置）来确定需要显示的内容应该是什么。

这个RecyclerView模型做了很多优化工作，所以你不必：

- 首次填充列表时，它会创建并绑定列表的某些ViewHolder对象。例如，如果当前View显示位置0到9的列表，则RecyclerView会创建并绑定这些ViewHolder对象，也可能会创建并绑定位置10的ViewHolder对象，这样，如果用户滚动列表，则下一个元素已经准备好显示了。
- 当用户滚动列表时，RecyclerView会根据需要创建新的ViewHolder对象。它还可以保存已滚动到屏幕外的ViewHolder对象，因此可以重复使用。如果用户改变他们正在滚动的方向，则可以向后移动从屏幕滚动出来的ViewHolder对象。另一方面，如果用户继续向相同方向滚动，则距离可显示屏幕距离最长的ViewHolder对象可以重新绑定到新的数据。ViewHolder对象不需要创建或填充View，相反，应用程序只更新View的内容以匹配它绑定的新项目。
- 当显示的项目发生更改时，您可以通过调用适当的RecyclerView.Adapter.notify ...（）方法来通知适配器。然后，适配器的内置代码仅重新绑定受影响的项目。

## 添加依赖库

要使用RecyclerView，您需要将v7支持库添加到项目中，如下所示：

1. 打开你app的build.gradle文件

2. 添加支持库

   ```groovy
   dependencies {
       implementation 'com.android.support:recyclerview-v7:28.0.0'
   }
   ```

## 添加RecyclerView到你的布局文件中

现在，您可以将RecyclerView添加到布局文件中。例如，以下布局使用RecyclerView作为整个布局的唯一View：

```xml
<?xml version="1.0" encoding="utf-8"?>
<!-- A RecyclerView with some commonly used attributes -->
<android.support.v7.widget.RecyclerView
    android:id="@+id/my_recycler_view"
    android:scrollbars="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"/>
```

将RecyclerView添加到布局文件后，获取对象的句柄，将其连接到布局管理器，并为要显示的数据添加适配器：

- kotlin

  ```kotlin
  class MyActivity : Activity() {
      private lateinit var recyclerView: RecyclerView
      private lateinit var viewAdapter: RecyclerView.Adapter<*>
      private lateinit var viewManager: RecyclerView.LayoutManager
  
      override fun onCreate(savedInstanceState: Bundle?) {
          super.onCreate(savedInstanceState)
          setContentView(R.layout.my_activity)
  
          viewManager = LinearLayoutManager(this)
          viewAdapter = MyAdapter(myDataset)
  
          recyclerView = findViewById<RecyclerView>(R.id.my_recycler_view).apply {
              // use this setting to improve performance if you know that changes
              // in content do not change the layout size of the RecyclerView
              setHasFixedSize(true)
  
              // use a linear layout manager
              layoutManager = viewManager
  
              // specify an viewAdapter (see also next example)
              adapter = viewAdapter
  
          }
      }
      // ...
  }
  ```

- Java

  ```java
  public class MyActivity extends Activity {
      private RecyclerView recyclerView;
      private RecyclerView.Adapter mAdapter;
      private RecyclerView.LayoutManager layoutManager;
  
      @Override
      protected void onCreate(Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          setContentView(R.layout.my_activity);
          recyclerView = (RecyclerView) findViewById(R.id.my_recycler_view);
  
          // use this setting to improve performance if you know that changes
          // in content do not change the layout size of the RecyclerView
          recyclerView.setHasFixedSize(true);
  
          // use a linear layout manager
          layoutManager = new LinearLayoutManager(this);
          recyclerView.setLayoutManager(layoutManager);
  
          // specify an adapter (see also next example)
          mAdapter = new MyAdapter(myDataset);
          recyclerView.setAdapter(mAdapter);
      }
      // ...
  }
  ```

## 添加Adapter

要将所有数据提供给列表，必须继承RecyclerView.Adapter类。此对象为项（item）创建View，并在原始项不再可见时用新数据项替换某些View的内容。

以下代码示例显示了一个数据集的简单实现，该数据集由使用TextView小部件显示的字符串数组组成：

- kotlin

  ```kotlin
  class MyAdapter(private val myDataset: Array<String>) :
          RecyclerView.Adapter<MyAdapter.MyViewHolder>() {
  
      // Provide a reference to the views for each data item
      // Complex data items may need more than one view per item, and
      // you provide access to all the views for a data item in a view holder.
      // Each data item is just a string in this case that is shown in a TextView.
      class MyViewHolder(val textView: TextView) : RecyclerView.ViewHolder(textView)
  
  
      // Create new views (invoked by the layout manager)
      override fun onCreateViewHolder(parent: ViewGroup,
                                      viewType: Int): MyAdapter.MyViewHolder {
          // create a new view
          val textView = LayoutInflater.from(parent.context)
                  .inflate(R.layout.my_text_view, parent, false) as TextView
          // set the view's size, margins, paddings and layout parameters
          ...
          return MyViewHolder(textView)
      }
  
      // Replace the contents of a view (invoked by the layout manager)
      override fun onBindViewHolder(holder: MyViewHolder, position: Int) {
          // - get element from your dataset at this position
          // - replace the contents of the view with that element
          holder.textView.text = myDataset[position]
      }
  
      // Return the size of your dataset (invoked by the layout manager)
      override fun getItemCount() = myDataset.size
  }
  ```

- Java

  ```java
  public class MyAdapter extends RecyclerView.Adapter<MyAdapter.MyViewHolder> {
      private String[] mDataset;
  
      // Provide a reference to the views for each data item
      // Complex data items may need more than one view per item, and
      // you provide access to all the views for a data item in a view holder
      public static class MyViewHolder extends RecyclerView.ViewHolder {
          // each data item is just a string in this case
          public TextView textView;
          public MyViewHolder(TextView v) {
              super(v);
              textView = v;
          }
      }
  
      // Provide a suitable constructor (depends on the kind of dataset)
      public MyAdapter(String[] myDataset) {
          mDataset = myDataset;
      }
  
      // Create new views (invoked by the layout manager)
      @Override
      public MyAdapter.MyViewHolder onCreateViewHolder(ViewGroup parent,
                                                     int viewType) {
          // create a new view
          TextView v = (TextView) LayoutInflater.from(parent.getContext())
                  .inflate(R.layout.my_text_view, parent, false);
          ...
          MyViewHolder vh = new MyViewHolder(v);
          return vh;
      }
  
      // Replace the contents of a view (invoked by the layout manager)
      @Override
      public void onBindViewHolder(MyViewHolder holder, int position) {
          // - get element from your dataset at this position
          // - replace the contents of the view with that element
          holder.textView.setText(mDataset[position]);
  
      }
  
      // Return the size of your dataset (invoked by the layout manager)
      @Override
      public int getItemCount() {
          return mDataset.length;
      }
  }
  ```

布局管理器会调用Adapter的onCreateViewHolder（）方法，该方法需要构建一个RecyclerView.ViewHolder对象并设置它用于显示其内容的View。 ViewHolder对象的类型必须与Adapter类签名中声明的类型匹配。通常，它会通过inflate XML布局文件来设置View。由于ViewHolder对象尚未分配给任何特定数据，因此该方法实际上并未设置View的内容。

然后，布局管理器将ViewHolder对象绑定到其数据，它通过调用适配器的onBindViewHolder（）方法将ViewHolder的位置传递给RecyclerView来完成此操作，onBindViewHolder（）方法需要获取适当的数据，并使用它来填充ViewHolder的布局。例如，如果RecyclerView正在显示名称列表，则该方法可能会在列表中找到相应的名称，并填写ViewHolder的TextView。

如果列表需要更新，请在RecyclerView.Adapter对象上调用通知方法，例如notifyItemChanged（），布局管理器会重新绑定所有受影响的ViewHolder，允许他们的数据进行更新。

> 提示：ListAdapter类可用于确定列表更改时列表中的哪些项目需要更新。

## 自定义你的RecyclerView

您可以自定义RecyclerView以满足您的特定需求。标准类提供了大多数开发人员所需的所有功能，在许多情况下，您需要做的唯一定制是为每个ViewHolder对象设计View（View），并编写代码以使用适当的数据更新这些View。但是，如果您的应用具有特定要求，则可以通过多种方式修改标准行为。以下部分描述了一些常见的自定义。

### 修改layout

RecyclerView使用布局管理器（layout manager）在屏幕上定位各个项目（item），并确定何时重用用户不再可见的itemView。要重用（或回收）View，布局管理器可能会要求适配器使用与数据集不同的元素替换View的内容。以这种方式回收View可以避免创建不必要的View或执行昂贵的findViewById（）查找，从而提高性能。 Android支持库包括三个标准布局管理器，每个管理器提供许多自定义选项：

- LinearLayoutManager将项目排列在一维列表中。使用带有LinearLayoutManager的RecyclerView提供的功能类似于旧的ListView布局。
- GridLayoutManager将项目排列在二维网格中，就像棋盘上的方块一样。使用带有GridLayoutManager的RecyclerView提供的功能类似于较旧的GridView布局。
- StaggeredGridLayoutManager将项目排列在二维网格中，每列稍微偏离前一个，就像美国国旗中的星星一样。

如果这些布局管理器都不适合您的需求，您可以通过继承RecyclerView.LayoutManager抽象类来创建自己的布局管理器。

### 添加item动画

每当item发生变化时，RecyclerView都会使用动画师（*animator*）来改变其外观。这个动画师（*animator*）是一个继承了RecyclerView.ItemAnimator抽象类的对象。默认情况下，RecyclerView使用DefaultItemAnimator来提供动画。如果要提供自定义动画，可以通过继承RecyclerView.ItemAnimator来定义自己的动画对象。

### 允许item被选中

recyclerview-selection库使用户可以使用触摸或鼠标输入选择RecyclerView列表中的项目。您可以保持对所选项目的可视化表示的控制。您还可以保留对控制选择行为的策略的控制，例如可以选择的项目以及可以选择的项目数。

要向RecyclerView实例添加选择支持，请按照下列步骤操作：

1. 确定要使用的选择键类型，然后构建ItemKeyProvider。
   您可以使用三种键类型来标识所选项：Parcelable（以及所有子类，如Uri），String和Long。有关选择键类型的详细信息，请参阅[`SelectionTracker.Builder`](https://developer.android.com/reference/androidx/recyclerview/selection/SelectionTracker.Builder.html)。

2. 实现ItemDetailsLookup。
   ItemDetailsLookup使选择库能够在给定MotionEvent的情况下访问有关RecyclerView项的信息。它实际上是由RecyclerView.ViewHolder实例备份（或从中提取）的ItemDetails实例的工厂。

3. 在RecyclerView中更新项目视图以反映用户已选择或取消选择它。
   选择库不为所选项目提供默认的可视化装饰。您必须在实现onBindViewHolder（）时提供此功能。建议的方法如下：
   - 在onBindViewHolder（）中，使用true或false调用View对象上的setActivated（）（而不是setSelected（））（具体取决于是否选中了该项）。
   - 更新视图样式以表示激活状态。我们建议您使用颜色状态列表资源来配置样式。
4. 使用ActionMode为用户提供工具以对选择执行操作。
   注册SelectionTracker.SelectionObserver，以便在选择更改时收到通知。首次创建选择时，启动ActionMode以将其表示给用户，并提供特定于选择的操作。例如，您可以向ActionMode栏添加删除按钮，并连接栏上的后退箭头以清除选择。当选择变空时（如果用户最后一次清除了选择），请不要忘记终止操作模式。

5. 执行任何解释的辅助操作
   在事件处理流水线的末尾，库可以通过点击它来确定用户正在尝试激活项目，或者正在尝试拖放项目或一组所选项目。通过注册适当的监听器来对这些解释做出反应。有关更多信息，请参阅SelectionTracker.Builder。

6. 使用SelectionTracker.Builder组装所有内容
   以下示例显示如何使用Long选择键将这些片段放在一起：

   - kotlin

     ```kotlin
     var tracker = SelectionTracker.Builder(
         "my-selection-id",
         recyclerView,
         StableIdKeyProvider(recyclerView),
         MyDetailsLookup(recyclerView),
         StorageStrategy.createLongStorage())
             .withOnItemActivatedListener(myItemActivatedListener)
             .build()
     ```

   - Java

     ```java
     SelectionTracker tracker = new SelectionTracker.Builder<>(
             "my-selection-id",
             recyclerView,
             new StableIdKeyProvider(recyclerView),
             new MyDetailsLookup(recyclerView),
             StorageStrategy.createLongStorage())
             .withOnItemActivatedListener(myItemActivatedListener)
             .build();
     ```

   为了构建SelectionTracker实例，您的应用必须提供用于将RecyclerView初始化为SelectionTracker.Builder的相同RecyclerView.Adapter。因此，在创建RecyclerView.Adapter之后，您很可能需要在创建后将SelectionTracker实例注入RecyclerView.Adapter。否则，您将无法从onBindViewHolder（）方法检查项目的选定状态。

7. 在活动生命周期事件中包括选择。
   为了在活动生命周期事件中保留选择状态，您的应用程序必须分别从活动的onSaveInstanceState（）和onRestoreInstanceState（）方法调用选择跟踪器的onSaveInstanceState（）和onRestoreInstanceState（）方法。您的应用还必须为SelectionTracker.Builder构造函数提供唯一的选择ID。此ID是必需的，因为活动或片段可能具有多个不同的可选列表，所有这些列表都需要以其保存状态保留。

## 其他资源

RecyclerView在[Sunflower](https://github.com/googlesamples/android-sunflower)示例项目中有示例用法。