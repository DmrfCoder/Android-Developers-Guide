# 处理Android APP Links——验证App Links

[原文(英文)地址](https://developer.android.com/training/app-links/verify-site-associations)

Android APP Links是一种特殊类型的Deep Links，可让您的website URLs立即在Android应用中打开相应的内容（无需用户选择应用）。

要将Android APP Links添加到您的应用程序，请定义使用HTTP URL打开应用程序内容的Intent filter（如[创建指向您内容的DeepLinks](./创建指向您内容的Deep-Links.md)中所述），并验证您是否是应用程序和网站URL的拥有者（如本文档中所述）。如果系统成功验证您拥有URL，系统会自动将这些URL Intent路由到您的应用程序。

要验证您的应用和网站的所有权，需要执行以下步骤：

- 在您的manifest中请求自动应用链接验证。这向Android系统发出信号，表明它应该验证您的应用是否属于您的Intent filter中使用的URL域。

- 通过在以下地址托管数字资产链接JSON文件来声明您的网站与您的Intent filter之间的关系：

  ```
  https://domain.name/.well-known/assetlinks.json
  ```

你可以在以下资源中找到相关信息：

- [Supporting URLs and App Indexing in Android Studio](https://developer.android.com/tools/help/app-link-indexing.html?hl=zh-cn)

- [Creating a Statement List](https://developers.google.com/digital-asset-links/v1/create-statement?hl=zh-cn)

## Deep Links和app Links的不同

Deep Links接是一种Intent filter，允许用户直接在Android应用中输入特定Activity。单击其中一个链接可能会打开一个消歧对话框，允许用户选择可以处理给定URL的多个应用程序（包括您的应用程序）中的一个。例如，图1显示了用户单击地图链接后的消歧对话框，询问是否在地图或Chrome中打开链接。

![app-disambiguation_2x](https://ws1.sinaimg.cn/large/006tNc79gy1g1t78hvhcjj30dw0s2afb.jpg)

<center>图一：消歧对话框</center>

Android App Link是基于您的网站网址的Deep Links，该网址已经过验证，你拥有所有权。因此，如果已安装，则单击其中一个会立即打开您的应用程序，不显示消歧对话框。虽然用户可能稍后改变他们处理这些链接的偏好。

下表描述了更具体的差别。

|                           | Deep Links                                       | App Links                                                    |
| :------------------------ | ------------------------------------------------ | ------------------------------------------------------------ |
| Intent URL scheme         | `http`, `https`,或者自定义的 scheme              | `http` 或者 `https`                                          |
| Intent action             | 任何 action                                      | 需要 `android.intent.action.VIEW`                            |
| Intent category           | 任何 category                                    | 需要 `android.intent.category.BROWSABLE`and `android.intent.category.DEFAULT` |
| Link verification         | None                                             | 需要使用HTTPS在您的网站上提供的数字资产链接（[Digital Asset Links](https://developers.google.com/digital-asset-links/v1/getting-started?hl=zh-cn)）文件 |
| User experience(用户体验) | 可以显示消歧对话框，供用户选择打开链接的应用程序 | 没有对话框：您的应用将打开您的网站链接进行处理               |
| Compatibility(兼容性)     | 所有Android 版本                                 | Android 6.0 及以上                                           |

## 请求app links验证

要为您的应用启用链接处理验证，请在应用manifest文件中包含`android.intent.action.VIEW` intent action和`android.intent.category.BROWSABLE`intent category，如以下清单代码段所示：

```xml
<activity ...>
    <!--注意下面这一句-->
    <intent-filter android:autoVerify="true">
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="http" android:host="www.example.com" />
        <data android:scheme="https" />
    </intent-filter>

</activity>
```

当您的任何一个Intent filter上都存在`android：autoVerify =“true”`时，在Android 6.0及更高版本的设备上安装您的应用会导致系统尝试验证与您应用的任何Intent filter中的网址相关联的所有host。验证涉及以下内容：

- 系统检查所有Intent filter，包括：
  - 动作(Action)：android.intent.action.VIEW
  - 分类(Category)：android.intent.category.BROWSABLE和android.intent.category.DEFAULT
  - 数据方案(Data scheme)：http或https
- 对于上述Intent filter中找到的每个唯一主机名，Android会在`https：//hostname/.well-known/assetlinks.json`上查询相应的数字资产链接文件网站。

仅当系统为manifest中的所有host找到匹配的数字资产链接文件时，它才会将您的应用程序建立为指定URL模式的默认处理程序。

## <span id="Supporting-app-linking-for-multiple-hosts">支持多个主机（host）的app links</span>

系统必须能够针对托管在所有相应Web域上的数字资产链接文件验证应用程序的URL intent过滤器数据元素中指定的每个host。如果任何验证失败，则应用程序不会被验证为应用程序的意Intent filter中定义的任何URL的默认处理程序。然后，系统默认使用其标准行为来解析Intent，如[创建指向您内容的DeepLinks](./创建指向您内容的Deep-Links.md)中所述。

比如，如果在`https://www.example.com/.well-known/assetlinks.json` 和 `https://www.example.net/.well-known/assetlinks.json`中均没有发现一个名为assetlinks.json的文件，则该intent filter的app将会验证失败：

```xml
<application>

  <activity android:name=”MainActivity”>
    <intent-filter android:autoVerify="true">
      <action android:name="android.intent.action.VIEW" />
      <category android:name="android.intent.category.DEFAULT" />
      <category android:name="android.intent.category.BROWSABLE" />
      <!--注意下面这两句-->
      <data android:scheme="http" android:host="www.example.com" />
      <data android:scheme="https" />
      
    </intent-filter>
  </activity>
  <activity android:name=”SecondActivity”>
    <intent-filter>
      <action android:name="android.intent.action.VIEW" />
      <category android:name="android.intent.category.DEFAULT" />
      <category android:name="android.intent.category.BROWSABLE" />
      <!--注意下面这一句-->
      <data android:scheme="https" android:host="www.example.net" />
    </intent-filter>
  </activity>

</application>
```

请记住，同一Intent filter中的所有<data\>元素将组合在一起，以考虑其组合属性的所有变体。例如，上面的第一个intent过滤器包含一个只声明HTTPS方案的<data\>元素。但它与其他<data\>元素结合使用，因此intent过滤器同时支持http://www.example.com和https://www.example.com。因此，当您要定义URI方案和域的特定组合时，必须创建单独的intent filter。

## 创建支持多个子域（subdomains）的App links

数字资产链接协议将Intent filter中的子域视为唯一的独立主机。因此，如果您的intent filter列出了具有不同子域的多个主机，则必须将`assetlinks.json`发布在每个域上才有效 。例如，以下Intent filter包括`www.example.com`并且 `mobile.example.com`作为已接受的Intent URL主机。所以有效的 `assetlinks.json`必须同时发布在`https://www.example.com/.well-known/assetlinks.json`和`https://mobile.example.com/.well-known/assetlinks.json`。

```xml
<application>
  <activity android:name=”MainActivity”>
    <intent-filter android:autoVerify="true">
      <action android:name="android.intent.action.VIEW" />
      <category android:name="android.intent.category.DEFAULT" />
      <category android:name="android.intent.category.BROWSABLE" />
      <!--注意下面这两句-->
      <data android:scheme="https" android:host="www.example.com" />
      <data android:scheme="https" android:host="mobile.example.com" />
    </intent-filter>
  </activity>
</application>
```

或者，如果使用通配符（例如`*.example.com`）声明主机名，则`assetlinks.json`必须在根主机名（`example.com`）处被发布。例如，具有以下intent filter的应用程序将通过对`example.com`任何子名称（例如`foo.example.com`）的验证(只要该`assetlinks.json`文件在`https://example.com/.well- known/assetlinks.json`上被发布)：

```xml
<application>
  <activity android:name=”MainActivity”>
    <intent-filter android:autoVerify="true">
      <action android:name="android.intent.action.VIEW" />
      <category android:name="android.intent.category.DEFAULT" />
      <category android:name="android.intent.category.BROWSABLE" />
      <!--注意这一句-->
      <data android:scheme="https" android:host="*.example.com" />
    </intent-filter>
  </activity>
</application>
```

## 声明网站关联(website associations)

一个[数字资产链接](https://developers.google.com/digital-asset-links/v1/getting-started?hl=zh-cn) JSON文件必须在你的网站上公布，以指示与网站关联的Android应用并验证应用的URL Intent。JSON文件使用以下字段来标识关联的应用程序：

- `package_name`：在应用程序的build.gradle`文件中声明的 [application ID](https://developer.android.com/studio/build/application-id.html?hl=zh-cn) 

- `sha256_cert_fingerprints`：应用程序签名证书的SHA256指纹。您可以使用以下命令通过Java keytool生成指纹：

  ```
  $ keytool -list -v -keystore my-release-key.keystore
  ```

​        此字段支持多个指纹，可用于支持应用程序的不同版本，例如调试和生产版本。

以下示例`assetlinks.json`文件授予`com.example`Android应用程序的链接开放权限 ：

```json
[{
  "relation": ["delegate_permission/common.handle_all_urls"],
  "target": {
    "namespace": "android_app",
    "package_name": "com.example",
    "sha256_cert_fingerprints":
    ["14:6D:E9:83:C5:73:06:50:D8:EE:B9:95:2F:34:FC:64:16:A0:83:42:E6:1D:BE:A8:8A:04:96:B2:3F:CF:44:E5"]
  }
}]
```

## 将网站与多个App关联

网站可以在同一`assetlinks.json` 文件中声明与多个应用的关联。以下文件列表显示了一个声明文件的示例，该文件声明与两个应用程序的关联，并驻留在`https://www.example.com/.well-known/assetlinks.json`：

```json
[{
  "relation": ["delegate_permission/common.handle_all_urls"],
  "target": {
    "namespace": "android_app",
    "package_name": "com.example.puppies.app",
    "sha256_cert_fingerprints":
    ["14:6D:E9:83:C5:73:06:50:D8:EE:B9:95:2F:34:FC:64:16:A0:83:42:E6:1D:BE:A8:8A:04:96:B2:3F:CF:44:E5"]
  }
  },
  {
  "relation": ["delegate_permission/common.handle_all_urls"],
  "target": {
    "namespace": "android_app",
    "package_name": "com.example.monkeys.app",
    "sha256_cert_fingerprints":
    ["14:6D:E9:83:C5:73:06:50:D8:EE:B9:95:2F:34:FC:64:16:A0:83:42:E6:1D:BE:A8:8A:04:96:B2:3F:CF:44:E5"]
  }
}]
```

不同的应用程序可以处理同一Web主机下的不同资源的链接。例如，app1可以声明一个intent过滤器`https://example.com/articles`，app2可以声明一个intent过滤器`https://example.com/videos`。

> **注意：**与域（domain）关联的多个应用程序可能使用相同或不同的证书进行签名。

## 将多个网站与单个app进行关联

多个网站可以在各自的`assetlinks.json`文件中声明与同一应用程序的关联。以下文件列表显示了如何使用app1声明example.com和example.net的关联的示例。第一个list显示了example.com与app1的关联：

`https://www.example.com/.well-known/assetlinks.json:`

```json
[{
  "relation": ["delegate_permission/common.handle_all_urls"],
  "target": {
    "namespace": "android_app",
    "package_name": "com.mycompany.app1",
    "sha256_cert_fingerprints":
    ["14:6D:E9:83:C5:73:06:50:D8:EE:B9:95:2F:34:FC:64:16:A0:83:42:E6:1D:BE:A8:8A:04:96:B2:3F:CF:44:E5"]
  }
}]
```

下一个list显示了example.net与app1的关联。只是托管这些文件的位置不同（`.com`和`.net`）：

`https://www.example.net/.well-known/assetlinks.json:`

```json
[{
  "relation": ["delegate_permission/common.handle_all_urls"],
  "target": {
    "namespace": "android_app",
    "package_name": "com.mycompany.app1",
    "sha256_cert_fingerprints":
    ["14:6D:E9:83:C5:73:06:50:D8:EE:B9:95:2F:34:FC:64:16:A0:83:42:E6:1D:BE:A8:8A:04:96:B2:3F:CF:44:E5"]
  }
}]
```

## 发布JSON验证文件

您必须在以下位置发布JSON验证文件：

```
https：// domain.name /.well-known/assetlinks.json
```

确保以下内容：

- 该`assetlinks.json`文件以 `application/json`格式提供。
- 无论您的应用程序的Intent filter是否将HTTPS声明为数据方案，都必须可以通过HTTPS连接访问`assetlinks.json`文件。
- 该`assetlinks.json`文件必须可以访问，不需要任何重定向（没有301或302重定向），并且可以通过机器人访问（您`robots.txt`必须允许抓取 `/.well-known/assetlinks.json`）。
- 如果您的应用链接支持多个主机域，则必须在每个域上发布`assetlinks.json`文件。请参阅 [支持多个主机（host）的app links](#Supporting-app-linking-for-multiple-hosts)。
- 不要在manifest文件中使用dev / test URL发布可能无法访问的应用程序（例如任何只能通过VPN访问的应用程序）。在这种情况下，解决方法是[configure build variants](https://developer.android.com/studio/build/build-variants.html?hl=zh-cn)以为开发构建生成不同的清单文件。

## 测试App links

在实现App links功能时，您应该测试链接功能，以确保系统可以将您的应用与您的网站相关联，并按照您的预期处理URL请求。

要测试现有语句文件，可以使用 [Statement List Generator and Tester](https://developers.google.com/digital-asset-links/tools/generator?hl=zh-cn)工具。

### 确定要验证的主机列表

测试时，您应确认系统应为您的应用验证的关联主机列表。列出其对应的intent filter，包含以下属性和元素的所有URL：

- `android:scheme`值为`http`或`https`
- `android:host` 具有域URL模式
- `android.intent.action.VIEW` category元素
- `android.intent.category.BROWSABLE` category元素

使用此列表检查每个命名主机和子域上是否提供了数字资产链接JSON文件。

### 确认数字资产链接文件

对于每个网站，使用Digital Asset Links API确认数字资产链接JSON文件已正确托管和定义：

```
https://digitalassetlinks.googleapis.com/v1/statements:list?
   source.web.site=https://domain.name:optional_port&
   relation=delegate_permission/common.handle_all_urls
```



### 测试一个URL Intent

确认要与您的应用关联的网站列表，并确认托管的JSON文件有效后，请在您的设备上安装该应用。等待至少20秒以完成异步验证过程。使用以下命令检查系统是否验证了您的应用程序并设置了正确的链接处理策略（policies）：

```
adb shell am start -a android.intent.action.VIEW \
    -c android.intent.category.BROWSABLE \
    -d "http://domain.name:optional_port"
```

### 检查链接策略

作为测试过程的一部分，您可以检查当前系统设置以进行链接处理。使用以下命令获取所连接设备上所有应用程序的现有链接处理策略列表：

```
adb shell dumpsys package domain-preferred-apps
```

或者也可以通过以下命令达到同样的目的：

```
adb shell dumpsys包d
```

> **注意：**确保在安装应用程序后至少等待20秒，以便系统完成验证过程。

该命令返回设备上定义的每个用户或配置文件的列表，前面带有以下格式的标头：

```
App linkages for user 0:
```

在此header之后，输出使用以下格式列出该用户的链接处理设置：

```
Package: com.android.vending
Domains: play.google.com market.android.com
Status: always : 200000002
```

此列表指出哪些应用与该用户的哪些域相关联：

- `Package` - 根据其清单中声明的包名称标识应用程序。
- `Domains` - 使用空格作为分隔符，显示此应用处理其Web链接的主机的完整列表。
- `Status` - 显示此应用的当前链接处理设置。已通过验证且其清单包含的应用程序`android:autoVerify="true"`显示状态为`always`。此状态后的十六进制数与Android系统的用户应用程序链接首选项记录相关。该值不表示验证是否成功。

> **注意：**如果用户在验证完成之前更改了应用的应用链接设置，即使验证失败，您也可能会看到成功验证的误报。但是，如果用户明确启用应用程序以打开支持的链接而不询问，则此验证失败无关紧要。这是因为用户首选项优先于编程验证（或缺少编程验证）。因此，该链接直接进入您的应用程序，而不显示对话框，就像验证成功一样。

### 测试示例

要使应用链接验证成功，系统必须能够使用您在应用的Intent filter中指定的所有网站验证您的应用，并且该网站符合应用链接的条件。以下示例显示了定义了多个应用程序链接的清单配置：

```xml
<application>

    <activity android:name=”MainActivity”>
        <intent-filter android:autoVerify="true">
            <action android:name="android.intent.action.VIEW" />
            <category android:name="android.intent.category.DEFAULT" />
            <category android:name="android.intent.category.BROWSABLE" />
            <data android:scheme="https" android:host="www.example.com" />
            <data android:scheme="https" android:host="mobile.example.com" />
        </intent-filter>
        <intent-filter>
            <action android:name="android.intent.action.VIEW" />
            <category android:name="android.intent.category.BROWSABLE" />
            <data android:scheme="https" android:host="www.example2.com" />
        </intent-filter>
    </activity>

    <activity android:name=”SecondActivity”>
        <intent-filter>
            <action android:name="android.intent.action.VIEW" />
            <category android:name="android.intent.category.DEFAULT" />
            <category android:name="android.intent.category.BROWSABLE" />
            <data android:scheme="https" android:host="account.example.com" />
        </intent-filter>
    </activity>

      <activity android:name=”ThirdActivity”>
        <intent-filter>
            <action android:name="android.intent.action.VIEW" />
            <category android:name="android.intent.category.DEFAULT" />
            <data android:scheme="https" android:host="map.example.com" />
        </intent-filter>
        <intent-filter>
            <action android:name="android.intent.action.VIEW" />
            <category android:name="android.intent.category.BROWSABLE" />
            <data android:scheme="market" android:host="example.com" />
        </intent-filter>
      </activity>

</application>
```

平台尝试从以上清单验证的主机列表是：

```
www.example.com
mobile.example.com
www.example2.com
account.example.com
```

平台不会尝试从上面的清单验证的主机列表是：

```
map.example.com (it does not have android.intent.category.BROWSABLE)
market://example.com (it does not have either an “http” or “https” scheme)
```

要了解有关语句列表的更多信息，请参阅[Creating a Statement List](https://developers.google.com/digital-asset-links/v1/create-statement?hl=zh-cn).