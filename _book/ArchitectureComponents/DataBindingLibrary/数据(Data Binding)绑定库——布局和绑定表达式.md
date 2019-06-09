# 数据(Data Binding)绑定库——布局和绑定表达式

表达式语言允许您编写处理视图调度（View dispatch）的事件的表达式。数据绑定库自动生成将布局中的视图与数据对象绑定所需的类。

支持数据绑定的布局文件与常规布局文件略有不同，支持数据绑定的布局文件以layout的作为根标签（root tag），里面包含data元素和view根元素，此view根元素是您没有使用数据绑定的布局文件中对应的根元素。以下代码展示了一个示例布局文件：

```xml
<?xml version="1.0" encoding="utf-8"?>
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
           android:text="@{user.firstName}"/>
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.lastName}"/>
   </LinearLayout>
</layout>
```

被`data`修饰的`user`变量描述了一个可在此布局文件中使用的属性：

```xml
<variable name="user" type="com.example.User" />
```

`layout`中的表达式使用`@{}`语法写入了属性值中，下面这个TextView的text被设置成了user的firstName属性值：

```xml
<TextView android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:text="@{user.firstName}" />
```

> 注意：布局表达式应该尽量保持简单，因为他们不能被执行单元测试，而且拥有有限的IDE支持。为了使布局表达式尽可能简单，你可以使用自定义[binding adapters](https://developer.android.com/topic/libraries/data-binding/binding-adapters)

## 数据对象

我们现在假设您有一个普通的对象来描述User实体：

- kotlin

  ```kotlin
  data class User(val firstName: String, val lastName: String)
  ```

- java

  ```java
  public class User {
    public final String firstName;
    public final String lastName;
    public User(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }
  }
  ```

该类型的对象拥有不能更改的属性。在应用程序中，只读取一次并且之后不会更改的数据是很常见的。也可以使用遵循一组约定的对象，例如Java中的访问器（accessor）方法的使用，如以下示例所示：

- kotlin

  ```kotlin
  // Not applicable in Kotlin.
  data class User(val firstName: String, val lastName: String)
  ```

- java

  ```java
  public class User {
    private final String firstName;
    private final String lastName;
    public User(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }
    public String getFirstName() {
        return this.firstName;
    }
    public String getLastName() {
        return this.lastName;
    }
  }
  ```

从数据绑定的角度来看，这两个类是等价的。用于android：text属性的表达式@ {user.firstName}会访问前一类中的firstName字段和后一类中的getFirstName（）方法。或者，如果存在一个firstName（）方法，它也会解析为firstName（）。

## 绑定数据

为每个布局文件生成绑定类。默认情况下，类的名称基于布局文件的名称，将其转换为Pascal大小写并向其添加Binding后缀。上面的布局文件名是activity_main.xml，因此相应的生成类是ActivityMainBinding，此类包含布局属性（例如，user变量）到布局视图的所有绑定，并知道如何为绑定表达式分配值。创建绑定的推荐方法是在扩展（inflating）布局时执行此操作，如下例所示：

- kotlin

  ```kotlin
  override fun onCreate(savedInstanceState: Bundle?) {
      super.onCreate(savedInstanceState)
  
      val binding: ActivityMainBinding = DataBindingUtil.setContentView(
              this, R.layout.activity_main)
  
      binding.user = User("Test", "User")
  }
  ```

- java

  ```java
  @Override
  protected void onCreate(Bundle savedInstanceState) {
     super.onCreate(savedInstanceState);
     ActivityMainBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_main);
     User user = new User("Test", "User");
     binding.setUser(user);
  }
  ```

在运行时，app会在UI上展示“Test” User，或者你也可以使用[LayoutInflater](https://developer.android.com/reference/android/view/LayoutInflater.html)得到View，如下代码所示：

- kotlin

- ```kotlin
  val binding: ActivityMainBinding = ActivityMainBinding.inflate(getLayoutInflater())
  ```

- java

- ```java
  ActivityMainBinding binding = ActivityMainBinding.inflate(getLayoutInflater());
  ```

如果你在Fragment、ListView、RecycleView的item（条目）中使用数据绑定，你可以向下面这样使用绑定类或者DataBindingUtil的inflate方法：

- kotlin

- ```kotlin
  val listItemBinding = ListItemBinding.inflate(layoutInflater, viewGroup, false)
  // or
  val listItemBinding = DataBindingUtil.inflate(layoutInflater, R.layout.list_item, viewGroup, false)
  ```

- java

- ```java
  ListItemBinding binding = ListItemBinding.inflate(layoutInflater, viewGroup, false);
  // or
  ListItemBinding binding = DataBindingUtil.inflate(layoutInflater, R.layout.list_item, viewGroup, false);
  ```

## 表达式语言

### 共同特征

表达式语言看起来很像托管（managed）代码中的表达式。您可以在表达式语言中使用以下运算符和关键字：

- 数学符号 `+ - / * %`
- 字符串拼接 `+`
- 逻辑符号 `&& ||`
- 二进制操作 `& | ^`
- 一元运算 `+ - ! ~`
- 移位 `>> >>> <<`
- 比较 `== > < >= <=` (注意 `<` 需要转义为 `&lt;`)
- `instanceof`
- 群 `()`
- 文字, String, 数字, `null`
- Cast
- Method calls
- Field access
- Array access `[]`
- 三元运算符`?:`

例子：

```xml
android:text="@{String.valueOf(index + 1)}"
android:visibility="@{age > 13 ? View.GONE : View.VISIBLE}"
android:transitionName='@{"image_" + id}'
```

### 不能使用的操作

以下是你在托管（managed）代码中可用但是不能再表达式中使用的操作：

- this
- super
- new
- 显示通用调用(Explicit generic invocation)

### Null 结合运算符

如果前操作数不为空，则空合并运算符（??）选择左操作数;如果前操作数为空，则选择右操作数：

```xml
android:text="@{user.displayName ?? user.lastName}"
```

该表达式和下面的表达式效果相同：

```xml
android:text="@{user.displayName != null ? user.displayName : user.lastName}"
```

### 属性的引用

表达式中可以使用以下格式引用类中的属性，对于字段、getter和ObservableField对象都是一样的方法：

```xml
android:text="@{user.lastName}"
```

### 避免空指针异常

自动生成的绑定代码会自动检查null值并且避免空指针异常。比如在表达式`@{user.name}`中，如果user是空值，那么user.name就会访问它的默认值即null，如果你引用数据类型为int的user.age，数据绑定会使用其默认值0.

### 集合

对于常见的集合，例如arrays，lists，sparse lists，还有maps，可以使用`[]`操作符方便地使用。

```xml
<data>
    <import type="android.util.SparseArray"/>
    <import type="java.util.Map"/>
    <import type="java.util.List"/>
    <variable name="list" type="List&lt;String>"/>
    <variable name="sparse" type="SparseArray&lt;String>"/>
    <variable name="map" type="Map&lt;String, String>"/>
    <variable name="index" type="int"/>
    <variable name="key" type="String"/>
</data>
…
android:text="@{list[index]}"
…
android:text="@{sparse[index]}"
…
android:text="@{map[key]}"
```

> 注意：为了保证xml文件语法上的正确，你必须避免使用字符`<`,比如，你应该使用`List&lt;String>`而不是使用`List<String>`

您还可以使用object.key表示法引用map中的值。例如，上面示例中的@ {map [key]}可以替换为@ {map.key}。

### String literals

您可以使用单引号括起属性值，这允许您在表达式中使用双引号，如以下示例所示：

```xml
android:text='@{map["firstName"]}'
```

也可以使用双引号来包围属性值。这样做时，字符串文字应该用后引号`包围：

```xml
android:text="@{map[`firstName`]}"
```

### 资源

你可以使用以下语法在表达式中访问resources：

```xml
android:padding="@{large? @dimen/largePadding : @dimen/smallPadding}"
```

可以通过使用字符串和复数（plural）提供参数：

```xml
android:text="@{@string/nameFormat(firstName, lastName)}"
android:text="@{@plurals/banana(bananaCount)}"
```

当复数（plural）带有多个参数时，应传递所有参数：

```xml
  Have an orange
  Have %d oranges

android:text="@{@plurals/orange(orangeCount, orangeCount)}"
```

某些资源需要显式类型评估（evaluation），如下表所示：

| Type              | Normal reference | Expression reference |
| :---------------- | :--------------- | :------------------- |
| String[]          | @array           | @stringArray         |
| int[]             | @array           | @intArray            |
| TypedArray        | @array           | @typedArray          |
| Animator          | @animator        | @animator            |
| StateListAnimator | @animator        | @stateListAnimator   |
| color int         | @color           | @color               |
| ColorStateList    | @color           | @colorStateList      |

## 处理事件

数据绑定允许你处理由View分发的事件(比如onClick()方法)。事件（Event）属性名称由侦听器（listener）方法的名称确定，但也有一些例外。比如，[View.OnClickListener](https://developer.android.com/reference/android/view/View.OnClickListener.html)有方法onClick()，所以event属性的值为android：onClick。

click事件有一些专门的事件处理程序需要使用不同于android：onClick的属性值以避免冲突。您可以使用以下属性来避免这些类型的冲突：

| Class          | Listener setter                                   | Attribute               |
| :------------- | :------------------------------------------------ | :---------------------- |
| `SearchView`   | `setOnSearchClickListener(View.OnClickListener)`  | `android:onSearchClick` |
| `ZoomControls` | `setOnZoomInClickListener(View.OnClickListener)`  | `android:onZoomIn`      |
| `ZoomControls` | `setOnZoomOutClickListener(View.OnClickListener)` | `android:onZoomOut`     |

你可以使用以下机制去处理事件：

- 方法引用([Method references](https://developer.android.com/topic/libraries/data-binding/expressions#method_references))：在你的表达式中，你可以引用符合listener方法签名的方法。当表达式最终求得的值为方法的引用时，数据绑定将方法引用和方法的所有者对象包装在一个侦听器中，并在目标视图上设置该侦听器。如果表达式求值为null，则数据绑定不会创建侦听器并改为设置空侦听器。
- 侦听器绑定([Listener bindings](https://developer.android.com/topic/libraries/data-binding/expressions#listener_bindings))：这些是在事件发生时计算的lambda表达式。数据绑定总是创建一个设置在View上的侦听器，调度事件时，侦听器将计算lambda表达式。

### <span id="method-references">方法引用</span>

事件可以直接绑定到方法上进行处理，就像android：onClick可以被分配到activity中的方法一样。方法引用和View的onClick方法相比主要的优势在于表达式是在编译时运行的，这样如果方法不存在或者方法签名是错误的，你将会收到一个编译时错误而不是运行发生错误。

方法引用和侦听器绑定的主要不同在于实际发挥作用的侦听器是在数据被绑定的时候才被实现的，而不是方法被触发的时候。如果你偏向于在事件发生的时候计算表达式的值，请使用[绑定侦听器](#listener-bindings)的方法。

要将事件分配给其处理程序，请使用普通绑定表达式，其值为要调用的方法名称。例如，请考虑以下示例布局数据对象：

- kotlin

- ```kotlin
  class MyHandlers {
      fun onClickFriend(view: View) { ... }
  }
  ```

- java

- ```java
  public class MyHandlers {
      public void onClickFriend(View view) { ... }
  }
  ```

绑定表达式可以通过以下方法将View的点击侦听器分配给onClickFriend()方法：

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
   <data>
       <variable name="handlers" type="com.example.MyHandlers"/>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"
           android:onClick="@{handlers::onClickFriend}"/>
   </LinearLayout>
</layout>
```

> 注意：表达式中的方法签名必须和侦听器对象中的方法签名一致。

### <span id="listener-bindings">侦听器绑定</span>

监听器绑定是运行在在event发生的的绑定表达式，其与方法引用相似，但是侦听器绑定可以让你运行任意数据绑定表达式，这个特性在Gradle 2.0及之后可被使用。

在方法引用中，方法的参数必须和事件侦听器的参数一致。在侦听器绑定中，只需要你返回的值和侦听器期待返回的值一致即可(除非期待返回void)。比如，考虑以下含有onSaveClick（）方法的presenter类：

- kotlin

  ```kotlin
  class Presenter {
      fun onSaveClick(task: Task){}
  }
  ```

- java

- ```java
  public class Presenter {
      public void onSaveClick(Task task){}
  }
  ```

你可以按照以下的方式将事件与onSaveClick()方法绑定起来：

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    <data>
        <variable name="task" type="com.android.example.Task" />
        <variable name="presenter" type="com.android.example.Presenter" />
    </data>
    <LinearLayout android:layout_width="match_parent" android:layout_height="match_parent">
        <Button android:layout_width="wrap_content" android:layout_height="wrap_content"
        android:onClick="@{() -> presenter.onSaveClick(task)}" />
    </LinearLayout>
</layout>
```

当表达式中使用了回调（callback）之后，数据绑定自动创建必要的侦听器并将该侦听器注册给event事件，当View触发事件之后，数据绑定计算给定表达式的值。就像普通的绑定表达式一样，在侦听器表达式被计算的过程中仍然可以保证null安全和线程安全。

在上面的例子中，我们没有定义传入onClick（View）中的View参数。侦听器绑定在侦听器参数这部分提供了两个选择：你可以忽略所有的参数或者为所有的参数命名。如果你为所有的参数进行了命名，你就可以在你的表达式中使用它们。比如，上面的表达式可以被写成下面这样：

```xml
//忽略了所有参数
android:onClick="@{(view) -> presenter.onSaveClick(task)}"
```

或者如果你想在表达式中使用该参数，可以使用：

- kotlin

  ```kotlin
  class Presenter {
      fun onSaveClick(view: View, task: Task){}
  }
  ```

  ```xml
  android:onClick="@{(theView) -> presenter.onSaveClick(theView, task)}"
  ```

- java

  ```java
  public class Presenter {
      public void onSaveClick(View view, Task task){}
  }
  ```

  ```xml
  //为参数命名并使用
  android:onClick="@{(theView) -> presenter.onSaveClick(theView, task)}"
  ```

你可以使用lambda表达式去使用不止一个参数：

- kotlin

  ```kotlin
  class Presenter {
      fun onCompletedChanged(task: Task, completed: Boolean){}
  }
  ```

  ```xml
  <CheckBox android:layout_width="wrap_content" android:layout_height="wrap_content"
        android:onCheckedChanged="@{(cb, isChecked) -> presenter.completeChanged(task, isChecked)}" />
  ```

- java

  ```java
  public class Presenter {
      public void onCompletedChanged(Task task, boolean completed){}
  }
  ```

  ```xml
  <CheckBox android:layout_width="wrap_content" android:layout_height="wrap_content"
        android:onCheckedChanged="@{(cb, isChecked) -> presenter.completeChanged(task, isChecked)}" />
  ```

如果你侦听的事件返回一个不为void类型的返回值，你的表达式也必须返回一个相同类型的值。比如，如果你想监听一个长按事件，你的表达式应该返回一个boolean值：

- kotlin

  ```kotlin
  class Presenter {
      fun onLongClick(view: View, task: Task): Boolean { }
  }
  ```

  ```xml
  android:onLongClick="@{(theView) -> presenter.onLongClick(theView, task)}"
  ```

- java

  ```java
  public class Presenter {
      public boolean onLongClick(View view, Task task) { }
  }
  ```

  ```xml
  android:onLongClick="@{(theView) -> presenter.onLongClick(theView, task)}"
  ```

如果表达式由于null对象的问题不能被执行，数据绑定会返回对应类型的默认值。比如，引用类型返回null，int类型返回0，booean类型返回false等。

如果你需要使用带有谓词（predicate）的表达式(比如三元组)，你可以使用void作为符号(symbol)：

```xml
android:onClick="@{(v) -> v.isVisible() ? doSomething() : void}"
```

#### 避免复杂的侦听器

侦听器表达式十分有效，而且可以让你的代码更加易读。另一方面，包含复杂表达式的侦听器会让你的布局难以阅读和维护。这些表达式应该像从UI中传输数据到你的回调方法中一样简单，你应该在侦听器表达式中调用的回调方法中实现复杂的逻辑。

## Imports, variables和 includes

数据绑定提供如Imports, variables和 includes这样的特性。Imports使你可以在你的布局中更加容易地引用类，Variables允许你描述一些可以在表达式中使用的属性，Includes让你可以在你的app中重用复杂的布局。

### Imports

Imports允许你在你的layout文件中更加容易地引用类，就像在托管代码（managed code）中一样，data element中会使用0个或者多个import元素，下面的代码import了Viewclass到layout文件中：

```xml
<data>
    <import type="android.view.View"/>
</data>
```

import View类允许你在你的绑定表达式中引用View，下面的代码展示了如何引用View类中的VISIBLE和GONE常量：

```xml
<TextView
   android:text="@{user.lastName}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"
   android:visibility="@{user.isAdult ? View.VISIBLE : View.GONE}"/>
```

#### Type aliases(输入别名)

当存在命名冲突的时候，一个类也许应该被重命名为别名。下面的代码将 `com.example.real.estate`包中的View类重命名为了Vista：

```xml
<import type="android.view.View"/>
<import type="com.example.real.estate.View"
        alias="Vista"/>
```

你可以使用Vista去引用com.example.real.estate.View，而View可以用来引用布局文件中的android.view.View.

#### 导入其他类

导入的类型可以用作变量和表达式中的类型引用。以下示例显示用作变量类型的User和List：

```xml
<data>
    <import type="com.example.User"/>
    <import type="java.util.List"/>
    <variable name="user" type="User"/>
    <variable name="userList" type="List<User>"/>
</data>
```

> 警告：Android Studio尚未处理导入，因此导入变量的自动完成功能可能无法在IDE中运行。您的应用程序仍可以编译，您可以通过在变量定义中使用完全限定（qualified）名称来解决IDE问题。

你也可以使用已经导入的类型作为表达式强制类型转化的一部分，下面的代码将connection属性转化为User属性：

```xml
<TextView
   android:text="@{((User)(user.connection)).lastName}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```

在表达式中引用静态字段和方法时，也可以使用导入的类型。下面的代码导入了MyStringUtils类并且引用了其capitalize方法：

```xml
<data>
    <import type="com.example.MyStringUtils"/>
    <variable name="user" type="com.example.User"/>
</data>
…
<TextView
   android:text="@{MyStringUtils.capitalize(user.lastName)}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```

就像在普通代码中一样，java.lang.*被自动导入了。

### Variables

你可以在data 元素里面使用多个variable元素，每一个variable元素描述了一个layout文件中的绑定表达式中被可能被使用的属性。下面的代码生命了user、image、note变量：

```xml
<data>
    <import type="android.graphics.drawable.Drawable"/>
    <variable name="user" type="com.example.User"/>
    <variable name="image" type="Drawable"/>
    <variable name="note" type="String"/>
</data>
```

在编译时变量的类型会被检查，所以如果一个变量实现了Observable接口或者是一个[可观察集合](https://developer.android.com/topic/libraries/data-binding/observability.html#observable_collections)，则应该在类型中反映出来。如果变量是未实现Observable接口的基类或接口，则不会观察变量。

当不同的配置（比如横屏和竖屏）对应不同的layout文件的时候，variables是公用的，所以在这些不同配置对应的不同layout文件中不能存在有冲突的变量。

生成的绑定类中的每一个被描述的变量都有getter和setter方法，变量会在自己对应的setter方法被调用之前一直保存自己的值为对应类型的默认值（null、0、false等）。

为了使用绑定表达式，根据需要生成了一个名为context的特殊变量。 context的值是来自根View的getContext（）方法的Context对象，使用该名称的显式变量声明覆盖上下文变量。

### includes

通过使用app命名空间和属性中的变量名，变量可以别的layout为文件中被传递到当前layout文件中。下面的示例代码展示了从name.xml和contact.xml文件中includeuser变量的方法：

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:bind="http://schemas.android.com/apk/res-auto">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <include layout="@layout/name"
           bind:user="@{user}"/>
       <include layout="@layout/contact"
           bind:user="@{user}"/>
   </LinearLayout>
</layout>
```

数据绑定不支持include作为merge元素的直接子元素，比如下面的布局文件是不被支持的：

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:bind="http://schemas.android.com/apk/res-auto">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <merge><!-- Doesn't work -->
       <include layout="@layout/name"
           bind:user="@{user}"/>
       <include layout="@layout/contact"
           bind:user="@{user}"/>
   </merge>
</layout>
```

## 其他资源

想了解数据绑定的更多内容，请参考以下内容：

### Samples

- [Android Data Binding Library samples](https://github.com/googlesamples/android-databinding)

### Codelabs

- [Android Data Binding codelab](https://codelabs.developers.google.com/codelabs/android-databinding)

### Blog posts

- [Data Binding — Lessons Learnt](https://medium.com/androiddevelopers/data-binding-lessons-learnt-4fd16576b719)