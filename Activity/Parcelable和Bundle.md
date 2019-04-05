# Parcelable和Bundle

Parcelable和Bundle对象旨在跨进程使用，例如IPC / Binder事务，具有Intent的Activity之间，以及跨配置更改存储瞬态。此文档介绍有关使用Parcelable和Bundle对象的建议和最佳实践。

> 注意：Parcel不是通用的序列化机制，您绝不应将任何Parcel数据存储在磁盘上或通过网络发送。

## 不同Activity之间发送数据

当app创建一个Intent对象以通过startActivity(android.content.Intent)去启动一个新的Activity时，app可以通过putExtra(java.lang.String,java.lang.String)传递参数给新的Activity。

以下代码演示了如何进行该操作：

- kotlin

  ```kotlin
  val intent = Intent(this, MyActivity::class.java).apply {
      putExtra("media_id", "a1b2c3")
      // ...
  }
  startActivity(intent)
  ```

- java

  ```java
  Intent intent = new Intent(this, MyActivity.class);
  intent.putExtra("media_id", "a1b2c3");
  // ...
  startActivity(intent);
  ```

系统包裹（编码，parcels）Intent的底层Bundle。然后，操作系统创建新Activity，之后解包（解码，un-parcels）Bundle，并将Intent传递给新Activity。

我们建议您使用Bundle类来传递操作系统已知的原语（primitives）对象。 Bundle类针对使用parcel的编码和解码进行了高度优化。

在某些情况下，您可能需要一种机制来跨Activity发送复合对象或复杂对象。在这种情况下，自定义类应该实现Parcelable，并提供正确的writeToParcel（android.os.Parcel，int）方法。它还必须提供一个名为CREATOR的非空字段，该字段实现Parcelable.Creator接口，其createFromParcel（）方法用于将Parcel转换回当前对象。有关更多信息，请参阅[Parcelable](https://developer.android.com/reference/android/os/Parcelable.html)对象的参考文档。

通过intent发送数据时，应注意将数据大小限制为几KB。发送过大数据可能导致系统抛出TransactionTooLargeException异常。

## 不同进程间发送数据

在进程之间发送数据和在Activity之间执行此操作类似。但是，在进程之间发送时，我们建议您不要使用自定义parcelables。如果您将自定义Parcelable对象从一个应用程序发送到另一个应用程序，则需要确保发送和接收应用程序上都存在完全相同的自定义类版本。通常，这可能是跨两个应用程序使用的公共库。如果您的应用尝试向系统发送自定义parcelable，则可能会发生错误，因为系统无法解码（unmarshal）它不知道的类。

例如，应用程序可能使用AlarmManager类设置alarm（警报），并在alarm Intent上使用自定义Parcelable。当闹钟响起时，系统会修改Intent的的Extras中的Bundle以添加重复计数。此修改可能导致系统从Extras中解析自定义的Parcelable对象。反过来，这种解析可能导致应用程序在收到修改后的alarm Intent时崩溃，因为应用程序希望接收不再存在的Extra数据。

Binder事务缓冲区具有有限的固定大小，目前是1MB，由进程正在进行的所有事务共享。由于此限制是在进程级别而不是在每个Activity级别，因此这些事务包括应用程序中的所有绑定事务，例如onSaveInstanceState，startActivity以及与系统的任何交互。超出大小限制时，将抛出TransactionTooLargeException。

对于savedInstanceState的特定情况，数据量应保持较小，因为只要用户可以导航回该Activity（即使Activity的进程被终止），系统进程就需要保持提供的数据。我们建议您将保存的状态保存在少于50k的数据中。

> 注意：在Android 7.0(API 24)及之后，系统会抛出TransactionTooLargeException异常，在之前版本的Android系统中，系统紧紧会在logcat打印一条警告。