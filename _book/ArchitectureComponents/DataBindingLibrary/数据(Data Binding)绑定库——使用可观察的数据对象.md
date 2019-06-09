# 数据(Data Binding)绑定库——使用可观察的数据对象

观察能力指的是当对象中的数据发生改变时，它能通知其他对象该更改的能力。数据绑定库允许你使对象、字段、集合编程可观察的。

任何普通旧对象（ plain-old object ）都可用于数据绑定，但修改对象不会自动导致UI更新。数据绑定可用于为数据对象提供在数据更改时通知其他对象（称为侦听器）的能力。有三种不同类型的可观察类：对象，字段和集合( [objects](https://developer.android.com/topic/libraries/data-binding/observability#observable_objects), [fields](https://developer.android.com/topic/libraries/data-binding/observability#observable_fields), [collections](https://developer.android.com/topic/libraries/data-binding/observability#observable_collections)）。

当这些可观察的数据对象呗绑定到UI上并且数据对象发生改变，UI将会自动更新。

## 可观察的字段(fields)

一些需要的实现在创建实现(Implement)了 [`Observable`](https://developer.android.com/reference/android/databinding/Observable.html)接口的类时会被自动完成，如果你的类只有一些属性，这是不值得的。在这种情况下，您可以使用通用Observable类和以下基于原语的类来使字段可观察：

- [`ObservableBoolean`](https://developer.android.com/reference/android/databinding/ObservableBoolean.html)
- [`ObservableByte`](https://developer.android.com/reference/android/databinding/ObservableByte.html)
- [`ObservableChar`](https://developer.android.com/reference/android/databinding/ObservableChar.html)
- [`ObservableShort`](https://developer.android.com/reference/android/databinding/ObservableShort.html)
- [`ObservableInt`](https://developer.android.com/reference/android/databinding/ObservableInt.html)
- [`ObservableLong`](https://developer.android.com/reference/android/databinding/ObservableLong.html)
- [`ObservableFloat`](https://developer.android.com/reference/android/databinding/ObservableFloat.html)
- [`ObservableDouble`](https://developer.android.com/reference/android/databinding/ObservableDouble.html)
- [`ObservableParcelable`](https://developer.android.com/reference/android/databinding/ObservableParcelable.html)

可观察字段个本身就含有单个可观察对象字段的可观察对象。原始版本在访问操作期间避免装箱和拆箱。要使用此机制，请在Java编程语言中创建public final属性或在Kotlin中创建只读属性，如以下示例所示：

- kotlin

- ```kotlin
  class User {
      val firstName = ObservableField<String>()
      val lastName = ObservableField<String>()
      val age = ObservableInt()
  }
  ```

- java

- ```java
  private static class User {
      public final ObservableField<String> firstName = new ObservableField<>();
      public final ObservableField<String> lastName = new ObservableField<>();
      public final ObservableInt age = new ObservableInt();
  }
  ```

要访问字段值，请使用get()和set()访问方法，如下所示：

- kotlin

  ```kotlin
  user.firstName = "Google"
  val age = user.age
  ```

- java

  ```java
  user.firstName.set("Google");
  int age = user.age.get();
  ```

> 注意：Android Studio3.1及以上允许你使用LiveData对象替换可观察的字段，该做法为你的app带来了更多好处，想了解更多的信息，请参考[使用LiveData通知UI有关数据更改的信息](./数据(Data Binding)绑定库——将布局视图绑定到体系结构组件.md#use-livedata-to-notify-the-ui-about-data-changes)

## 可观察的集合(collections)

一些应用程序使用动态结构来保存数据。可观察集合（Observable collections）允许使用key访问这些结构，当键是引用类型（如String）时，[`ObservableArrayMap`](https://developer.android.com/reference/android/databinding/ObservableArrayMap.html)类很有用，如以下示例所示：

- kotlin

- ```kotlin
  ObservableArrayMap<String, Any>().apply {
      put("firstName", "Google")
      put("lastName", "Inc.")
      put("age", 17)
  }
  ```

- java

- ```java
  ObservableArrayMap<String, Object> user = new ObservableArrayMap<>();
  user.put("firstName", "Google");
  user.put("lastName", "Inc.");
  user.put("age", 17);
  ```

在layout文件中，map可以使用string类型的key被绑定：

```xml
<data>
    <import type="android.databinding.ObservableMap"/>
    <variable name="user" type="ObservableMap<String, Object>"/>
</data>
…
<TextView
    android:text="@{user.lastName}"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>
<TextView
    android:text="@{String.valueOf(1 + (Integer)user.age)}"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>
```

[`ObservableArrayList`](https://developer.android.com/reference/android/databinding/ObservableArrayList.html)在key是integer类型的时候很有用，比如：

- kotlin

  ```kotlin
  ObservableArrayList<Any>().apply {
      add("Google")
      add("Inc.")
      add(17)
  }
  ```

- java

  ```java
  ObservableArrayList<Object> user = new ObservableArrayList<>();
  user.add("Google");
  user.add("Inc.");
  user.add(17);
  ```

在layout文件中，该List可以使用index被访问，就像：

```xml
<data>
    <import type="android.databinding.ObservableList"/>
    <import type="com.example.my.app.Fields"/>
    <variable name="user" type="ObservableList<Object>"/>
</data>
…
<TextView
    android:text='@{user[Fields.LAST_NAME]}'
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>
<TextView
    android:text='@{String.valueOf(1 + (Integer)user[Fields.AGE])}'
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>
```

## 可观察的对象(objects)

实现[`Observable`](https://developer.android.com/reference/android/databinding/Observable.html)接口的类允许注册希望被观察对象属性改变时被通知的侦听器。

Observable接口有一个添加和删除侦听器的机制，但您必须决定何时发送通知。为了简化开发，数据绑定库提供了[`BaseObservable`](https://developer.android.com/reference/android/databinding/BaseObservable.html)类，该类实现了侦听器注册机制。实现BaseObservable的数据类负责通知属性何时更改，这是通过将一个[`Bindable`](https://developer.android.com/reference/android/databinding/Bindable.html)注释分配给getter并在setter中调用[`notifyPropertyChanged()`](https://developer.android.com/reference/android/databinding/BaseObservable.html#notifyPropertyChanged(int))方法来完成的，如以下示例所示：

- kotlin

- ```kotlin
  class User : BaseObservable() {
  
      @get:Bindable
      var firstName: String = ""
          set(value) {
              field = value
              notifyPropertyChanged(BR.firstName)
          }
  
      @get:Bindable
      var lastName: String = ""
          set(value) {
              field = value
              notifyPropertyChanged(BR.lastName)
          }
  }
  ```

- java

  ```java
  private static class User extends BaseObservable {
      private String firstName;
      private String lastName;
  
      @Bindable
      public String getFirstName() {
          return this.firstName;
      }
  
      @Bindable
      public String getLastName() {
          return this.lastName;
      }
  
      public void setFirstName(String firstName) {
          this.firstName = firstName;
          notifyPropertyChanged(BR.firstName);
      }
  
      public void setLastName(String lastName) {
          this.lastName = lastName;
          notifyPropertyChanged(BR.lastName);
      }
  }
  ```

数据绑定在模块包中生成一个名为BR的类，该类包含用于数据绑定的资源的ID， Bindable注释在编译期间在BR类文件中生成一个条目（entry），如果数据类的基类不能被更改，则可以使用[`PropertyChangeRegistry`](https://developer.android.com/reference/android/databinding/PropertyChangeRegistry.html)对象实现Observable接口，以有效地注册和通知侦听器。

## 其他资源

想了解更多有关数据绑定的内容，请参考：

### Samples

- [Android Data Binding Library samples](https://github.com/googlesamples/android-databinding)

### Codelabs

- [Android Data Binding codelab](https://codelabs.developers.google.com/codelabs/android-databinding)

### Blog posts

- [Data Binding — Lessons Learnt](https://medium.com/androiddevelopers/data-binding-lessons-learnt-4fd16576b719)