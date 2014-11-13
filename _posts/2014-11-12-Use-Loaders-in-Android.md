---
layout: post
title: Android Loaders异步加载框架简介
category: Android Dev
date: 2014-11-13
---

Loaders从Android3.0开始引进（Loaders被翻译为装载器，它是一个异步加载数据的框架），它能在`Activity`或`Fragment`中异步加载数据；装载器具有如下特性：

- 它们对每个`Activity`和`Fragment`都有效；
- 它们支持数据的异步加载；
- 它们监视数据源的改变，并在数据源改变时传送新的结果；
- 当由于配置改变而被重新创建后，它们会自动重连到上一个装载器的游标，所以不必重新查询数据。

<!-- more -->

#装载器API概述
---
在使用装载器时，会涉及多个类和接口，下面对它们简单介绍一下：

##[LoaderManager](https://developer.android.com/reference/android/app/LoaderManager.html)
一个和`Activity`或`Fragment`关联抽象类，管理一个或多个装载器的实例，它帮助应用管理那些与`Activity`或`Fragment`生命周期相关的长时间运行的操作。最常见的方式是与一个`CursorLoader`一起使用，你也可以实现自己的装载器以加载其它类型的数据。
**每个`Activity`或`Fragment`只有一个`LoaderManager`，但是一个`LoaderManager`可以拥有多个装载器**。

##[LoaderManager.LoaderCallbacks](https://developer.android.com/reference/android/app/LoaderManager.LoaderCallbacks.html)
一个用于客户端与`LoaderManager`交互的回调接口，例如，你可以在`onCreateLoader()`回调来创建一个新的装载器。

##[Loader](https://developer.android.com/reference/android/content/Loader.html)
一个执行异步数据加载的抽象类，它是加载器的基类。你可以使用典型的`CursorLoader`，但是你也可以实现你自己的子类。一旦装载器被激活，它们应该监视它们的数据源并且在数据改变时传送新的结果。

##[AsyncTaskLoader](https://developer.android.com/reference/android/content/AsyncTaskLoader.html)
一个使用`AsyncTask`来执行异步加载工作的抽象类。

##[CursorLoader](https://developer.android.com/reference/android/content/CursorLoader.html)
一个`AsyncTaskLoader`的子类，它查询`ContentResolver`然后返回一个`Cursor`。这个装载器类的实现遵循查询游标数据源的标准，它的游标查询是通过`AsyncTaskLoader`在后台线程中执行，从而不会阻塞UI线程。使用这个装载器是从`ContentProvider`异步加载数据的最好方式。

上面所列的类和接口们是在应用中实现装载器时的核心组件，你的装载器并不一定需要用到所有的组件，但是你总是需要使用`LoaderManager`来初始化一个装载器。

下面这张UML图简单地描述了装载器框架结构：
![loaders](/media/files/2014/11/12/loaders.png)


#使用加载器
---
一个使用加载器的典型的应用包含以下几个组件：

1. 一个`Activity`或`Fragment`；
2. 一个`LoaderManager`的实例；
3. 一个依靠`ContentProvider`加载数据的`CursorLoader`；当然，你也可以继承`Loader`或`AsyncTaskLoader`实现你自己的装载器来从其它数据源加载数据；
4. 一个`LoaderManager.LoaderCallbacks`的实现，这是你创建新的装载器以及管理已有装载器的地方；
5. 一个用于展示装载器的返回数据的方式，例如使用一个`SimpleCursorAdapter`；
6. 一个数据源，例如`ContentProvider`（使用`CursorLoader`加载数据）。

##启动装载器
`LoaderManager`管理一个或多个装载器，但每个`Activity`或`Fragment`只拥有一个`LoaderManager`。

通常要在`Activity`的`onCreate()`方法中或`Fragment`的`onActivityCreated()`方法中初始化一个装载器，你可以如下创建：

{% highlight java %}
// 准备装载器，可以重连一个已经存在的也可以启动一个新的
getLoaderManager().initLoader(0, null, this);
{% endhighlight %}

`initLoader()`方法有下面几个参数：

- id：一个唯一的ID用于标识这个装载器，在这个例子中是0；
- args：可选的参数，在装载器初始化时作为参数传入，本例中是null；
- callbacks：一个`LoaderManager.LoaderCallbacks`的实现，被`LoaderManager`调用以报告装载器事件；在这个例子中，当前类实现了这个接口，所以传的是它自己：this。

`initLoader()`保证一个装载器被初始化并激活，它具有两种可能的结果：

- **如果ID所指的装载器已经存在，那么上一次创建的装载器将被重用**；
- 如果装载器不存在，`initLoader()`就会触发`LoaderManager.LoaderCallbacks`中的`onCreateLoader()`回调，你需要在这里实例化并返回一个新的装载器。

在这两种情况下，传入的`LoaderManager.LoaderCallbacks`的实现都与装载器绑定在一起，并且会在装载器状态变化时被调用；**如果在调用这个`initLoader()`方法时，装载器已经处于启动状态（也就是说这个装载器已存在），并且所请求的装载器已产生了数据，那么系统会马上调用`onLoadFinished()`**，所以你必须为这种情况的发生做好准备。

我用ApiDemo中的`LoaderCursor`做了下测试，总结出以下**几个主要的生命周期过程**：

> 调用`initLoader()`方法，且装载器还没有被创建：
> `onCreateLoader()`→`onLoadFinished()`
> 
> 调用`restartLoader()`方法重启装载器：
> `onCreateLoader()`→`onLoadFinished()`
> 
> 按Back键回退（注意按Home键不会触发调用，Back与Home代表了不同的用户行为）：
> `onLoaderReset()`
> 
> 调用`initLoader()`方法，且装载器实例已经存在
> `initLoader()`→`onLoadFinished()`


**注意：**
`initLoader()`方法会返回所创建的装载器，但是你不需获取这个对象的引用；`LoaderManager`会自动管理装载器的生命，`LoaderManager`会在需要时开始和停止装载动作，并且维护装载器的状态和它所关联的内容；这意味着，你很少与装载器直接交互（虽然可以使用装载器的方法来微调装载器的行为，请参阅ApiDemo中的LoaderThrottle.java）。大多数情况都是使用`LoaderManager.LoaderCallbacks`中的回调方法在某个事件发生时介入到数据加载的过程中。

##重启装载器
当你使用`initLoader()`时，如果指定ID的装载器已经存在，则它使用这个装载器；如果不存在，它将创建一个新的。但是有的时候你却想丢弃旧的装载器然后开始一个新的装载器。

要想丢弃旧的装载器，你应使用`restartLoader()`。例如，下面这个`SearchView.OnQueryTextListener`的实现在用户查询发生改变时重启了装载器，**装载器需要重启从而才能使用新的搜索过滤词来进行一次新的查询**。

{% highlight java %}
public boolean onQueryTextChanged(String newText) {
    // Called when the action bar search text has changed.  Update
    // the search filter, and restart the loader to do a new query
    // with this filter.
    mCurFilter = !TextUtils.isEmpty(newText) ? newText : null;
    getLoaderManager().restartLoader(0, null, this);
    return true;
}
{% endhighlight %}

##使用LoaderManager中的回调
`LoaderManager.LoaderCallbacks`是一个回调接口，它使得客户端可以与`LoaderManager`进行交互。

装载器，尤其是`CursorLoader`，我们希望在它停止后依然保持数据，这让应用可以在`Activity`或`Fragment`的`onStop()`和`onStart()`方法之间保持数据；当用户返回应用时，他们不需要再等待数据加载，你可以使用`LoaderManager.LoaderCallbacks`中的方法，在需要时创建新的装载器，并且告诉应用什么时候要停止使用装载器中的数据。

`LoaderManager.LoaderCallbacks`包含以下方法：

- `onCreateLoader()`：根据传入的ID，初始化并返回一个新的装载器
- `onLoadFinished()`：当一个装载器完成了它的装载过程后调用
- `onLoaderReset()`：当一个装载器被重置，从而使得其数据无效时被调用

###onCreateLoader
当你试图去操作一个装载器时（比如，调用`initLoader()`方法），它会检查是否指定ID的装载器已经存在，如果不存在，将会触发`LoaderManager.LoaderCallbacks`中的`onCreateLoader()`，你需要在这里创建一个新装载器并返回。

在下面例子中，在`onCreateLoader()`方法中创建了一个`CursorLoader`，你必须使用构造方法来创建`CursorLoader`，构造方法需要向`ContentProvider`执行一次查询的完整信息作为参数，它需要以下参数：

- uri：要获取内容的URI；
- projection：要返回的列的集合，传入null将会返回所有的列，这样比较低效；
- selection：一个过滤器，表明哪些行将被返回，格式化成类似SQL WHERE子句的样子（除去WHERE），传入null将返回所有的行；
- selectionArgs：你可以在selection中包含一些'?'，它将被本参数的值替换掉，这些值出现的顺序与'?'在selection中出现的顺序一致，值将作为字符串；
- sortOrder：排序字符串，格式化成类似于SQL ORDER BY子句样子（除去ORDER BY），传入null将使用默认顺序，默认顺序可能是无序的。

例如：
{% highlight java %}
// If non-null, this is the current filter the user has provided.
String mCurFilter;
...

public Loader<Cursor> onCreateLoader(int id, Bundle args) {
    // This is called when a new Loader needs to be created.  This
    // sample only has one Loader, so we don't care about the ID.
    // First, pick the base URI to use depending on whether we are
    // currently filtering.
    Uri baseUri;
    if (mCurFilter != null) {
        baseUri = Uri.withAppendedPath(Contacts.CONTENT_FILTER_URI,
                  Uri.encode(mCurFilter));
    } else {
        baseUri = Contacts.CONTENT_URI;
    }

    // Now create and return a CursorLoader that will take care of
    // creating a Cursor for the data being displayed.
    String select = "((" + Contacts.DISPLAY_NAME + " NOTNULL) AND ("
            + Contacts.HAS_PHONE_NUMBER + "=1) AND ("
            + Contacts.DISPLAY_NAME + " != '' ))";
    return new CursorLoader(getActivity(), baseUri,
            CONTACTS_SUMMARY_PROJECTION, select, null,
            Contacts.DISPLAY_NAME + " COLLATE LOCALIZED ASC");
}
{% endhighlight %}

###onLoadFinish
这个方法会在已创建的装载器完成其加载过程后被调用；这个方法保证会在装载器上的数据被释放之前被调用。在此方法中，你**必须移除所有对旧数据的使用（因为它们将很快会被删除），但是不要自己去释放它们，因为它们的装载器会去做这些事情**。

装载器一旦了解到应用不再使用这些数据时，将马上释放这些数据。例如，如果数据是一个从`CursorLoader`来的游标，你不应该自己调用游标的`close()`方法；如果游标被放置在一个`CursorAdapter`中，你可以使用`swapCursor()`方法，这样，旧的游标就不会被关闭（`CursorLoader`的实现中会自动帮你关闭旧的游标）；例如：

{% highlight java %}
// This is the Adapter being used to display the list's data.
SimpleCursorAdapter mAdapter;
...

public void onLoadFinished(Loader<Cursor> loader, Cursor data) {
    // Swap the new cursor in.  (The framework will take care of closing the
    // old cursor once we return.)
    mAdapter.swapCursor(data);
}
{% endhighlight %}

###onLoaderReset
当一个已创建的装载器被重置，从而使其数据无效时，此方法会被调用。这个回调告诉你什么时候数据将被释放，所以你可以释放对它的引用。

下面这个实现调用参数为null的`swapCursor()`：

{% highlight java %}
// This is the Adapter being used to display the list's data.
SimpleCursorAdapter mAdapter;
...

public void onLoaderReset(Loader<Cursor> loader) {
    // This is called when the last Cursor provided to onLoadFinished()
    // above is about to be closed.  We need to make sure we are no
    // longer using it.
    mAdapter.swapCursor(null);
}
{% endhighlight %}

#实例
---
Android官方提供了三个关于装载器的ApiDemo，分别是：

1. LoaderCursor.java：使用`CursorLoader`装载器从联系人content provider读取联系人集合，并显示在`Fragment`中。
2. LoaderThrottle.java：使用Throttle来减少对content provider查询的次数，非常全面的一个Demo。
3. LoaderCustom.java：实现了一个自定义的一个Loader，来实现对特定数据的加载。


<br/>
参考：

1. [Loaders](https://developer.android.com/guide/components/loaders.html)
2. [Android Loader详解](http://blog.csdn.net/niu_gao/article/details/7244117)
3. [LoaderManager及Loader初步探索](http://blog.sina.com.cn/s/blog_62c5894901014g5x.html)

本文出自[2dxgujun](http://2dxgujun.com/)，转载时请注明出处及相应链接。