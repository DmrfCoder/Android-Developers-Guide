# 与其他APP交互——概述

[原文(英文)地址](https://developer.android.com/training/basics/intents)

Android应用通常有几项Activity。每个Activity都显示一个用户界面，允许用户执行特定任务（例如查看地图或拍照）。要将用户从一个Activity带到另一个Activity，您的应用必须使用Intent来定义应用的“意图”以执行某些操作。使用startActivity（）等方法将Intent传递给系统时，系统会使用Intent来识别并启动相应的app组件。使用意图甚至允许您的应用启动包含在单独应用中的Activity。

Intent可以是显式的，以便启动特定组件（特定的Activity实例），也可以是隐式的，以启动任何可以处理预期操作的组件（例如“捕获照片”）。

本文档向您展示如何使用Intent与其他应用程序执行一些基本交互，例如启动另一个应用程序，从该应用程序接收结果，以及让您的应用程序能够响应来自其他应用程序的Intent。

## 主要内容

- 将用户导航到另一个APP

  > 介绍如何使用隐式Intent启动其他应用程序(APP)

- 接收另一个Activity的返回结果

  > 介绍如何启动另一个Activity并接收其返回的结果

- 允许其他APP启动你的Activity

  > 介绍如何通过声明应用程序接受的隐式意图的intent filter让你的App中的Activity可以被其他APP启动

关于本文档的额外参考信息，请参阅:

- [Sharing Simple Data](https://developer.android.com/training/sharing/index.html)
- [Sharing Files](https://developer.android.com/training/secure-file-sharing/index.html)
- [Integrating Application with Intents (blog post)](http://android-developers.blogspot.com/2009/11/integrating-application-with-intents.html)
- [Intents and Intent Filters](https://developer.android.com/guide/components/intents-filters.html)