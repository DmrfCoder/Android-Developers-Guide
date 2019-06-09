# 数据(Data Binding)绑定库——生成的绑定类

数据绑定库生成用于访问布局的变量（variables）和视图（views）的绑定类。此文档介绍如何创建和自定义生成的绑定类。

生成的绑定类将布局变量与布局中的视图链接起来，绑定类的名称和包可以自定义，所有生成的绑定类都继承自ViewDataBinding类。

为每个布局文件生成绑定类。默认情况下，类的名称基于布局文件的名称，将其转换为aaccPascal大小写并向其添加Binding后缀。布局文件名是activity_main.xml，相应的生成类是ActivityMainBinding，此类包含布局属性（例如，user变量）到布局视图的所有绑定，并知道如何为绑定表达式指定值。

## 创建一个绑定对象

在对布局进行inflateing之后，应该很快创建绑定对象，以确保布局中的表达式绑定到视图之前不会修改视图的层次结构。将对象绑定到布局的最常用方法是使用绑定类上的静态方法。您可以通过使用绑定类的inflate（）方法来扩展视图层次结构并将对象绑定到该层次结构，如以下示例所示：

- kotlin

- ```kotlin
  override fun onCreate(savedInstanceState: Bundle?) {
      super.onCreate(savedInstanceState)
  
      val binding: MyLayoutBinding = MyLayoutBinding.inflate(layoutInflater)
  }
  ```

- java

  ```java
  @Override
  protected void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      MyLayoutBinding binding = MyLayoutBinding.inflate(getLayoutInflater());
  }
  ```

除了[LayoutInflater](https://developer.android.com/reference/android/view/LayoutInflater.html)对象之外，还有一个替换版本的inflate（）方法，它接受一个[ViewGroup](https://developer.android.com/reference/android/view/ViewGroup.html)对象，如下例所示：

- kotlin

  ```kotlin
  val binding: MyLayoutBinding = MyLayoutBinding.inflate(getLayoutInflater(), viewGroup, false)
  ```

- java

- ```java
  MyLayoutBinding binding = MyLayoutBinding.inflate(getLayoutInflater(), viewGroup, false);
  ```

如果使用不同的机制对布局进行了inflate，则可以单独绑定，如下所示：

- kotlin

- ```kotlin
  val binding: MyLayoutBinding = MyLayoutBinding.bind(viewRoot)
  ```

- java

- ```java
  MyLayoutBinding binding = MyLayoutBinding.bind(viewRoot);
  ```

有时不能事先知道绑定类型。在这种情况下，可以使用[`DataBindingUtil`](https://developer.android.com/reference/android/databinding/DataBindingUtil.html)类创建绑定，如以下代码段所示：

- kotlin

  ```kotlin
  val viewRoot = LayoutInflater.from(this).inflate(layoutId, parent, attachToParent)
  val binding: ViewDataBinding? = DataBindingUtil.bind(viewRoot)
  ```

- java

  ```java
  View viewRoot = LayoutInflater.from(this).inflate(layoutId, parent, attachToParent);
  ViewDataBinding binding = DataBindingUtil.bind(viewRoot);
  ```

如果在Fragment，ListView或RecyclerView适配器中使用数据绑定item项，则可能更喜欢使用绑定类或DataBindingUtil类的inflate（）方法，如以下代码示例所示：

- kotlin

  ```kotlin
  val listItemBinding = ListItemBinding.inflate(layoutInflater, viewGroup, false)
  // or
  val listItemBinding = DataBindingUtil.inflate(layoutInflater, R.layout.list_item, viewGroup, false)
  ```

- java

  ```java
  ListItemBinding binding = ListItemBinding.inflate(layoutInflater, viewGroup, false);
  // or
  ListItemBinding binding = DataBindingUtil.inflate(layoutInflater, R.layout.list_item, viewGroup, false);
  ```

## 带ID的View

数据绑定库为每一个layout文件中具有ID的view创建不可变的字段，比如，以下代码中数据绑定库创建名为firstName和lastName，类型为TextView的字段：

```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"
   android:id="@+id/firstName"/>
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.lastName}"
  android:id="@+id/lastName"/>
   </LinearLayout>
</layout>
```

该库在一次传递中从视图层次结构中提取具有ID的View，此机制比为布局中的每个视图调用findViewById（）方法更快。

如果没有数据绑定，则ID是非必要的，但仍有一些情况需要通过代码访问视图。

## 变量

数据绑定库为每一个声明在layout文件中的变量生成访问方法，比如，对于下面的layout文件，数据绑定类会为user，image和note生成setter和getter方法：

```xml
<data>
   <import type="android.graphics.drawable.Drawable"/>
   <variable name="user" type="com.example.User"/>
   <variable name="image" type="Drawable"/>
   <variable name="note" type="String"/>
</data>
```

## ViewStubs

和普通View不同的是，ViewStub对象开始时是不可见的，当其被手动置位visible或者被inflate，它才会inflate另一个layout以替换原来自己的位置。

由于ViewStub基本上是从视图层次结构中消失的，因此绑定对象中的视图也必须消失以允许通过垃圾回收声明。因为View是final的，所以ViewStubProxy对象取代了生成的绑定类中的ViewStub，使您可以在ViewStub存在时访问它，并在ViewStub inflate时访问inflated的视图层次结构。

在inflate另一个布局时，必须为新布局建立绑定。因此，ViewStubProxy必须侦听ViewStub 的OnInflateListener并在需要时建立绑定。由于在给定时间只能存在一个侦听器，因此ViewStubProxy允许您设置OnInflateListener，在建立绑定后它会被调用。

## 立即绑定(Immediate Binding)

当变量或可观察对象发生更改时，绑定会在下一帧（next frame）之前更改。但是，有时必须立即执行绑定，要强制立即执行，请使用[`executePendingBindings()`](https://developer.android.com/reference/android/databinding/ViewDataBinding.html#executePendingBindings())方法。

## 高级绑定

### 动态变量

有时，特殊的绑定类是未知的。例如，针对任意布局操作的RecyclerView.Adapter不知道特定的绑定类，它仍然必须在调用onBindViewHolder（）方法期间分配绑定值。

在以下示例中，RecyclerView绑定的所有布局都具有item变量， BindingHolder对象有一个getBinding（）方法返回ViewDataBinding基类。

- kotlin

- ```kotlin
  override fun onBindViewHolder(holder: BindingHolder, position: Int) {
      item: T = items.get(position)
      holder.binding.setVariable(BR.item, item);
      holder.binding.executePendingBindings();
  }
  ```

- java

- ```java
  public void onBindViewHolder(BindingHolder holder, int position) {
      final T item = items.get(position);
      holder.getBinding().setVariable(BR.item, item);
      holder.getBinding().executePendingBindings();
  }
  ```

> 注意：数据绑定库在模块包中生成一个名为BR的类，其中包含用于数据绑定的资源的ID。在上面的示例中，库自动生成BR.item变量。

## 后台线程

只要你的data model不是一个collection，你都可以将其变为后台线程。数据绑定在评估期间本地化每个变量/字段以避免任何并发问题。

## 自定义绑定类的类名

默认的，会基于layout文件的名字生成一个绑定类，以大写字母开头，删除下划线（_），大写开头字母，并在后面添加单词Binding，该类放在模块包下的databinding包中。例如，布局文件contact_item.xml生成ContactItemBinding类，如果模块包是com.example.my.app，则绑定类放在com.example.my.app.databinding包中。

通过调整数据元素的class属性，可以重命名绑定类或将绑定类放在不同的包中。例如，以下布局在当前模块的数据绑定包中生成ContactItem绑定类：

```xml
<data class="ContactItem">
    …
</data>
```

您可以通过在类名前加一个点来为不同的包生成绑定类。以下示例在模块包中生成绑定类：

```xml
<data class=".ContactItem">
    …
</data>
```

您还可以使用要在其中生成绑定类的完整包名称。以下示例在com.example包中创建ContactItem绑定类：

```xml
<data class="com.example.ContactItem">
    …
</data>
```

## 其他资源

了解更多数据绑定的内容，请参考：

### Samples

- [Android Data Binding Library samples](https://github.com/googlesamples/android-databinding)

### Codelabs

- [Android Data Binding codelab](https://codelabs.developers.google.com/codelabs/android-databinding)

### Blog posts

- [Data Binding — Lessons Learnt](https://medium.com/androiddevelopers/data-binding-lessons-learnt-4fd16576b719)