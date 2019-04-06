# 处理Android APP Links——为Instant APPs创建App Links

[原文(英文)地址](https://developer.android.com/training/app-links/instant-app-links?hl=zh-cn)

Android Instant APP是您的应用程序的小版本（small version），无需安装即可运行。用户只需点击网址即可启动您的应用，而不是安装APK。因此，所有Instant APP都需要通过使用Android App Links声明的URL进行访问。本页介绍了如何为 [Android Instant Apps](https://developer.android.com/topic/instant-apps/index.html?hl=zh-cn)使用Android App Links。

> 注意：如果您没有Instant APP，那么您不需要阅读本指南,您应该通过阅读 [创建指向您内容的Deep Links](./创建指向您内容的Deep-Links.md) 来为您的可安装应用程序创建应用程序链接。

## App link概述

首先，这里是您应该了解的app links.的摘要：

- 当您为应用中的Activity创建一个intent filter，允许用户使用URL链接直接跳转到应用中的特定屏幕时，这称为“Deep Link”。但是，其他应用程序可以声明类似的URL  Intent filter，因此系统可能会询问用户打开哪个应用程序。要创建这些Deep Link，请阅读 [创建指向您内容的Deep Links](./创建指向您内容的Deep-Links.md) 。
- 当您在与应用程序的HTTP Deep Link对应的网站上发布assetlinks.json文件时，您就可以验证您的应用是否是这些URL的真正所有者。因此，您已将Deep Link转换为Android App Link，这可确保您的应用在用户点击此类网址时立即打开。要创建App Link，请阅读 [验证App Links](./验证App-Links.md)。
  因此，Android App Links只是您的网站经过验证的HTTP深层链接，因此用户无需选择要打开的应用程序。有关更具体的说明，请参阅深层链接和应用链接之间的差异。

但是，在这两种情况下，用户必须已安装您的应用程序。如果用户单击您的某个网站链接并且他们没有安装您的应用程序（并且没有其他应用程序处理该URL Intent），则会在Web浏览器中打开该URL。因此，创建Instant Apps可解决此问题 ，它允许用户通过简单地单击URL来打开您的应用程序，即使他们没有安装您的应用程序。

当最终用户对您的应用执行Google搜索时，Google搜索会显示带有“即时”徽章的网址。

## Instant App的App Link有何不同

如果你已经阅读了 [创建指向您内容的Deep Links](./创建指向您内容的Deep-Links.md) 和 [验证App Links](./验证App-Links.md)，那么你已经完成了使app link与你的Instant App一起工作所需的大部分工作。使用Instant App的App link时，还有一些额外的规定：

- 在您的Instant App中用作App link的所有Intent filter必须同时支持HTTP和HTTPS。例如：

  ```xml
  <intent-filter>
      <action android:name="android.intent.action.VIEW" />
      <category android:name="android.intent.category.DEFAULT" />
      <category android:name="android.intent.category.BROWSABLE" />
      <data android:scheme="http" android:host="www.example.com" />
      <data android:scheme="https" />
  </intent-filter>
  ```

  请注意，您不需要在第二个<data\>元素中包含host，因为在每个<intent-filter\>元素中，每个<data\>属性的所有组合都被视为有效（因此此intent过滤器也会解析`https：/ /www.example.com`）。

- 每个网站域(domain)只能声明一个Instant app。 （这与为可安装应用程序创建App link时不同，后者允许您将网站与多个应用程序相关联。）

## 创建App link时的其他提醒

- 您的Instant App中的所有HTTP URL Intent filter都应包含在您的可安装应用中。这很重要，因为一旦用户安装完整的应用程序，点击URL应始终打开已安装的应用程序，而不是Instant App
- 您必须在即时和可安装应用程序中的至少一个intent过滤器中设置autoVerify =“true”。 （了解 [enable automatic verification](https://developer.android.com/training/app-links/verify-site-associations.html?hl=zh-cn#config-verify)。）
- 您必须为每个域（domain）使用HTTPS协议（以及App link支持的子域）发布一个assetlinks.json。（请参阅 [验证App Links](./验证App-Links.md)中支持多个主机（host）的app links部分）。
- assetlinks.json文件必须是有效的JSON，无需重定向即可提供，并且可供机器人访问（您的robots.txt必须允许抓取/.well-known/assetlinks.json）。
- 建议不要在intent filter的host属性中使用通配符。 （了解 [验证App Links](./验证App-Links.md)中创建支持多个子域（subdomains）的App links部分）
- 应使用单独的intent filter声明自定义host/scheme URL。
- 确保您的App Link网址占据关键字词的热门搜索结果。