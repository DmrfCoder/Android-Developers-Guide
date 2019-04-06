# 处理Android APP Links——创建指向您内容的Deep Links

[原文(英文)地址](https://developer.android.com/training/app-links/deep-linking)

当单击链接或编程请求调用Web URI Intent时，Android系统将按顺序尝试以下每个操作，直到请求成功为止：

- 打开用户首选的可以处理URI的应用程序（如果已指定）。
- 打开唯一可以处理URI的可用应用程序。
- 允许用户从对话框中选择应用程序。

请按照以下步骤创建和测试指向您的内容的链接。您还可以使用Android Studio中的 [App Links Assistant](https://developer.android.com/studio/write/app-link-indexing.html?hl=zh-cn) 添加Android App Links。