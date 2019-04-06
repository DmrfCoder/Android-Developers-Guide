# 与其他APP交互——接收另一个Activity返回的结果

[原文(英文)地址](https://developer.android.com/training/basics/intents/result)

启动另一项Activity不一定是单向的，您还可以启动另一项Activity并收到其返回的结果，要接收结果，请调用startActivityForResult（）（而不是startActivity（））。

例如，您的应用可以启动相机应用并接收拍摄的照片。或者，您可以启动People应用程序，以便用户选择联系人，您将收到联系人详细信息。

当然，响应的Activity必须设计为可返回结果的，它将结果作为另一个Intent对象发送，您的Activity在onActivityResult（）回调中接收它。

> 注意：调用startActivityForResult（）时可以使用显式或隐式Intent。在启动您自己的某个Activity以接收结果时，您应该使用显示的Intent来确保您收到预期的结果。

## 启动Activity

启动需要接收返回值得Activity时使用的Intent对象没有什么特别之处，但是您需要将一个整数参数传递给startActivityForResult（）方法。

整数参数是标识您的请求的“请求代码”，当您收到返回结果Intent时，回调会提供相同的请求代码，以便您的应用程序可以正确识别结果并确定如何处理它。

例如，以下是启动允许用户选择联系人的Activity：

- kotlin

  ```kotlin
  const val PICK_CONTACT_REQUEST = 1  // The request code
  ...
  private fun pickContact() {
      Intent(Intent.ACTION_PICK, Uri.parse("content://contacts")).also { pickContactIntent ->
          pickContactIntent.type = Phone.CONTENT_TYPE // Show user only contacts w/ phone numbers
          startActivityForResult(pickContactIntent, PICK_CONTACT_REQUEST)
      }
  }
  ```

- java

  ```java
  static final int PICK_CONTACT_REQUEST = 1;  // The request code
  ...
  private void pickContact() {
      Intent pickContactIntent = new Intent(Intent.ACTION_PICK, Uri.parse("content://contacts"));
      pickContactIntent.setType(Phone.CONTENT_TYPE); // Show user only contacts w/ phone numbers
      startActivityForResult(pickContactIntent, PICK_CONTACT_REQUEST);
  }
  ```

## 接收返回的结果

当用户完成第二个Activity并返回时，系统会调用您第一个Activity的onActivityResult（）方法。此方法包括三个参数：

- 您传递给startActivityForResult（）的请求代码。
- 由第二个Activity指定的结果代码。如果操作成功，则为RESULT_OK;如果用户由于某种原因退出或操作失败，则为RESULT_CANCELED。
- 包含结果数据的Intent。

例如，以下是如何处理“选择联系人”Intent的结果：

- kotlin

  ```kotlin
  override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent) {
      // Check which request we're responding to
      if (requestCode == PICK_CONTACT_REQUEST) {
          // Make sure the request was successful
          if (resultCode == Activity.RESULT_OK) {
              // The user picked a contact.
              // The Intent's data Uri identifies which contact was selected.
  
              // Do something with the contact here (bigger example below)
          }
      }
  }
  ```

- java

  ```java
  @Override
  protected void onActivityResult(int requestCode, int resultCode, Intent data) {
      // Check which request we're responding to
      if (requestCode == PICK_CONTACT_REQUEST) {
          // Make sure the request was successful
          if (resultCode == RESULT_OK) {
              // The user picked a contact.
              // The Intent's data Uri identifies which contact was selected.
  
              // Do something with the contact here (bigger example below)
          }
      }
  }
  ```

在此示例中，Android的Contacts或People应用程序返回的结果Intent提供了一个内容Uri，用于标识用户选择的联系人。

为了成功处理结果，您必须了解Intent的结果格式。当返回结果的Activity是您自己的Activity之一时，这样做很容易。 Android平台附带的应用程序提供了自己的API，您可以依赖这些API来获取特定的结果数据。例如，People应用程序始终返回带有标识所选联系人的内容URI的结果，而Camera应用程序在“data”extra中返回一个 `Bitmap`（请参阅 [Capturing Photos](https://developer.android.com/training/camera/index.html)）。

### 菜单一：发送返回数据的方法

主要是通过setResult()方法。

```kotlin
//数据是使用Intent返回
Intent intent = new Intent();
//把返回数据存入Intent
intent.putExtra("result", "My name is linjiqin");
//设置返回数据
OtherActivity.this.setResult(RESULT_OK, intent);
```

### 彩蛋二：读取联系人数据

上面展示了如何从People应用程序获取结果的代码，但是没有详细介绍如何从结果中实际读取数据，因为它需要有关ContentProvider的更多理论。但是，如果您感到好奇，可以使用以下代码查看如何查询结果数据以获取所选联系人的电话号码：

- kotlin

  ```kotlin
  override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent) {
      // Check which request it is that we're responding to
      if (requestCode == PICK_CONTACT_REQUEST) {
          // Make sure the request was successful
          if (resultCode == Activity.RESULT_OK) {
              // We only need the NUMBER column, because there will be only one row in the result
              val projection: Array<String> = arrayOf(Phone.NUMBER)
  
              // Get the URI that points to the selected contact
              data.data?.also { contactUri ->
                  // Perform the query on the contact to get the NUMBER column
                  // We don't need a selection or sort order (there's only one result for this URI)
                  // CAUTION: The query() method should be called from a separate thread to avoid
                  // blocking your app's UI thread. (For simplicity of the sample, this code doesn't
                  // do that.)
                  // Consider using <code><a href="/reference/android/content/CursorLoader.html">CursorLoader</a></code> to perform the query.
                  contentResolver.query(contactUri, projection, null, null, null)?.apply {
                      moveToFirst()
  
                      // Retrieve the phone number from the NUMBER column
                      val column: Int = getColumnIndex(Phone.NUMBER)
                      val number: String? = getString(column)
  
                      // Do something with the phone number...
                  }
              }
          }
      }
  }
  ```

- java

  ```java
  @Override
  protected void onActivityResult(int requestCode, int resultCode, Intent resultIntent) {
      // Check which request it is that we're responding to
      if (requestCode == PICK_CONTACT_REQUEST) {
          // Make sure the request was successful
          if (resultCode == RESULT_OK) {
              // Get the URI that points to the selected contact
              Uri contactUri = resultIntent.getData();
              // We only need the NUMBER column, because there will be only one row in the result
              String[] projection = {Phone.NUMBER};
  
              // Perform the query on the contact to get the NUMBER column
              // We don't need a selection or sort order (there's only one result for the given URI)
              // CAUTION: The query() method should be called from a separate thread to avoid blocking
              // your app's UI thread. (For simplicity of the sample, this code doesn't do that.)
              // Consider using <code><a href="/reference/android/content/CursorLoader.html">CursorLoader</a></code> to perform the query.
              Cursor cursor = getContentResolver()
                      .query(contactUri, projection, null, null, null);
              cursor.moveToFirst();
  
              // Retrieve the phone number from the NUMBER column
              int column = cursor.getColumnIndex(Phone.NUMBER);
              String number = cursor.getString(column);
  
              // Do something with the phone number...
          }
      }
  }
  ```

> 注意：在Android 2.3（API级别9）之前，对联系人ContentProvider执行查询（如上所示）需要您的应用程序声明READ_CONTACTS权限（请参阅 [Security and Permissions](https://developer.android.com/guide/topics/security/security.html)）。但是，从Android 2.3开始，Contacts / People应用程序授予您的应用程序临时权限，以便在返回结果时从联系人提供程序中读取数据。临时权限仅适用于请求的特定联系人，因此除非您声明READ_CONTACTS权限，否则无法查询intent的Uri指定的联系人之外的联系人。

关于更多与本文档有关的内容，请参考：

- [Sharing Simple Data](https://developer.android.com/training/sharing/index.html)
- [Sharing Files](https://developer.android.com/training/secure-file-sharing/index.html)