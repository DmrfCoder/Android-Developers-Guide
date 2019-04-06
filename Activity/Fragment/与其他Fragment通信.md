# 与其他Fragment通信

[原文(英文)地址](https://developer.android.com/training/basics/fragments/communicating)

为了重用Fragment UI组件，您应该将每个组件构建为一个完全独立的模块化组件，以定义自己的布局和行为。一旦定义了这些可重用的Fragment，就可以将它们与Activity关联，并将它们与应用程序逻辑相连接，以实现整个复合UI。

通常，您会希望一个Fragment与另一个Fragment进行通信，例如根据用户事件更改内容。所有Fragment-to-Fragment通信都应该通过共享的ViewModel或通过关联的Activity完成。两个Fragment永远不应该直接通信。

Fragment之间通信的推荐方法是创建共享的ViewModel对象。两个Fragment都可以通过其宿主Activity访问ViewModel。 Fragments可以更新ViewModel中的数据，如果使用LiveData公开数据，只要从ViewModel观察LiveData，新状态就会被推送到另一个Fragment。要了解如何实现此类通信，请阅读[ViewModel guide](https://developer.android.com/topic/libraries/architecture/viewmodel.html).中的“在Fragment之间共享数据”部分。

如果您无法使用共享ViewModel在Fragments之间进行通信，则可以使用接口手动实现通信流。然而，这最终需要更多的工作来实现，并且不容易在其他Fragment中重用。

## 定义接口

要允许Fragment与其宿主Activity进行通信，您可以在Fragment类中定义接口并在Activity中实现它。 Fragment在其onAttach（）生命周期方法中捕获（attach）接口实现，然后可以调用Interface方法以与Activity通信。

以下是Fragment to Activity通信的示例：

- HeadlinesFragment

  ```java
  public class HeadlinesFragment extends ListFragment {
      OnHeadlineSelectedListener callback;
  
      public void setOnHeadlineSelectedListener(OnHeadlineSelectedListener callback) {
          this.callback = callback;
      }
  
      // This interface can be implemented by the Activity, parent Fragment,
      // or a separate test implementation.
      public interface OnHeadlineSelectedListener {
          public void onArticleSelected(int position);
      }
  
      // ...
  }
  ```

- MainActivity

  ```java
  public static class MainActivity extends Activity
          implements HeadlinesFragment.OnHeadlineSelectedListener{
      // ...
  
      @Override
      public void onAttachFragment(Fragment fragment) {
          if (fragment instanceof HeadlinesFragment) {
              HeadlinesFragment headlinesFragment = (HeadlinesFragment) fragment;
              headlinesFragment.setOnHeadlineSelectedListener(this);
          }
      }
  }
  ```

现在，Fragment可以通过使用OnHeadlineSelectedListener接口的mCallback实例调用onArticleSelected（）方法（或接口中的其他方法）来向Activity传递消息。

例如，当用户单击列表项时，将调用Fragment中的以下方法。该Fragment使用回调接口将事件传递给父Activity。

```java
    @Override
    public void onListItemClick(ListView l, View v, int position, long id) {
        // Send the event to the host activity
        callback.onArticleSelected(position);
    }
```

## 实现接口

为了从Fragment接收事件回调，托管它的Activity必须实现Fragment类中定义的接口。

例如，以下Activity实现了上面示例中的接口:

```java
public static class MainActivity extends Activity
        implements HeadlinesFragment.OnHeadlineSelectedListener{
    ...

    public void onArticleSelected(int position) {
        // The user selected the headline of an article from the HeadlinesFragment
        // Do something here to display that article
    }
}
```

## 发送Message到其他Fragment

宿主Activity可以通过使用findFragmentById（）捕获Fragment实例，然后直接调用Fragment的public方法来将消息传递给Fragment。

例如，假设上面显示的Activity可能包含另一个Fragment，该Fragment用于显示由上述回调方法返回的数据指定的项目。在这种情况下，Activity可以将回调方法中收到的信息传递给将显示该项目的另一个Fragment：

```java
public static class MainActivity extends Activity
        implements HeadlinesFragment.OnHeadlineSelectedListener{
    ...

    public void onArticleSelected(int position) {
        // The user selected the headline of an article from the HeadlinesFragment
        // Do something here to display that article

        ArticleFragment articleFrag = (ArticleFragment)
                getSupportFragmentManager().findFragmentById(R.id.article_fragment);

        if (articleFrag != null) {
            // If article frag is available, we're in two-pane layout...

            // Call a method in the ArticleFragment to update its content
            articleFrag.updateArticleView(position);
        } else {
            // Otherwise, we're in the one-pane layout and must swap frags...

            // Create fragment and give it an argument for the selected article
            ArticleFragment newFragment = new ArticleFragment();
            Bundle args = new Bundle();
            args.putInt(ArticleFragment.ARG_POSITION, position);
            newFragment.setArguments(args);

            FragmentTransaction transaction = getSupportFragmentManager().beginTransaction();

            // Replace whatever is in the fragment_container view with this fragment,
            // and add the transaction to the back stack so the user can navigate back
            transaction.replace(R.id.fragment_container, newFragment);
            transaction.addToBackStack(null);

            // Commit the transaction
            transaction.commit();
        }
    }
}
```

关于更多Fragment的实现，请参阅[Fragment](./),你也可以在 [relevant sample app](http://developer.android.com/shareables/training/FragmentBasics.zip)中了解更多信息。