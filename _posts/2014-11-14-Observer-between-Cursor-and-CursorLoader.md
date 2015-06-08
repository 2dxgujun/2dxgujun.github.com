---
layout: post
title: Cursor和CursorLoader中的观察者模式
category: Android
date: 2014-11-14
---

在我之前的一篇文章：[节流装载器（Throttle Loaders）的使用示例](/post/2014/11/14/Demonstration-of-Loaders.html)中简单的提了下`Cursor`和`CursorLoader`中所应用的观察者模式，这篇文章将从源码角度详细的分析一下这个观察者模式的具体实现。

# 前言

首先我们要搞清楚这里为什么使用观察者模式，它被用来完成什么工作。

之前我在介绍装载器（Loaders）时，说到它有一个很重要的特性：**监视数据源的改变，并在数据源发生变化时传送新的结果**；参考[Android装载器（Loaders）框架简介](/post/2014/11/14/Use-Loaders-in-Android.html)。

装载器要实现监视数据源，并在数据源的数据发生改变时装载最新的数据，必须提供一种源/监听器（Source/Listener）的机制；这里就是应用的这种观察者模式；关于观察者模式的详细介绍，可以参考我的另一篇文章：[Java观察者模式（Observer）](/post/2014/11/14/Java-Observer-Pattern.html)。

<!-- more -->

# API概述

在之前那篇[节流装载器（Throttle Loaders）的使用示例](/post/2014/11/14/Demonstration-of-Loaders.html)中，我们总共用到了下面的几个类：

1. `ContentProvider`：它对数据库CRUD操作进行了封装，提供了对外分享数据的接口；
2. `ContentResolver`：它通过URI来访问`ContentProvider`；
3. `Cursor`：它可以对查询数据库返回的数据集提供读取访问，实现类是`AbstractCursor`；
4. `CursorLoader`：它是一个装载器，在后台查询`ContentResolver`，返回一个`Cursor`。

这几个类之间的相互作用就构成了我们的观察者模式，首先要搞清楚`Cursor`与`CursorLoader`之间的关系，以及`Cursor`与`ContentResolver`之间更深层次之间的关系，才能弄清楚谁是目标，如何注册观察者；在数据变化时，目标是如何通知到观察者的。

下面我们将采用倒推的方式一步一步的学习。


# 第一个观察者模式

首先我们要找到观察者模式的切入点，在之前的那篇[节流装载器（Throttle Loaders）的使用示例](/post/2014/11/14/Fully-Demonstration-of-Throttling-Loaders.html)中的`ContentProvider`的实现中我们在`query()`方法的最后调用了`c.setNotificationUri(getContext().getContentResolver(), uri)`；在`insert()、delete()、update()`方法的最后调用了`getContext().getContentResolver().notifyChange(noteUri, null)`。

我们看一下源码，`AbstractCursor#setNotificationUri()`：
{% highlight java %}
public void setNotificationUri(ContentResolver cr, Uri notifyUri) {
    ...
    
    mSelfObserver = new SelfContentObserver(this);
    mContentResolver.registerContentObserver(mNotifyUri, true, mSelfObserver, UserHandle.myUserId());
    mSelfObserverRegistered = true;
    
    ...
}
{% endhighlight %}

源码中马上就能找到猫腻了，`SelfContentObserver`是一个观察者类，这里把它给注册到了`ContentResolver`中。

`ContentResolver#registerContentObserver()`方法有三个参数：

- uri：代表我对这个URI感兴趣，我想要在这个URI标识的数据改变时，接收通知，可以标识一行也可以是整张表；
- notifyForDescendents：true表示以uri前缀开始的任何变化都会通知；false表示完全匹配时才进行通知；
- observer：观察者对象。

继续深入下去，现在我们查看`ContentResolver#registerContentObserver()`方法：
{% highlight java %}
public final void registerContentObserver(Uri uri, boolean notifyForDescendents,
            ContentObserver observer, int userHandle) {
    getContentService().registerContentObserver(uri, notifyForDescendents,
            observer.getContentObserver(), userHandle);
}
{% endhighlight %}

我们可以看到实际上`ContentResolver`还不是目标，它把注册观察者的任务交给`ContentService`进行的，这是个系统服务（System Service）；从名称看到是内容服务，主要是为数据库等提供解决方案。

Go on...

`ContentService#registerContentObserver()`方法：
{% highlight java %}
public void registerContentObserver(Uri uri, boolean notifyForDescendents,
              IContentObserver observer) {
    mRootNode.addObserverLocked(uri, observer, notifyForDescendents, mRootNode,
            Binder.getCallingUid(), Binder.getCallingPid());
}
{% endhighlight %}

好了，到这里你会发现我们在客户端调用`AbstractCursor#setNotificationUri()`方法一路通过`ContentResover#registerContentObserver()`、`ContentService#registerContentObserver()`方法才真正注册了观察者；到这里目标和观察者就很明确了。

## 目标和观察者

- 目标：`ContentService`
- 观察者：`AbstractCursor$SelfContentObserver`

上面的代码通过指定URI可以仅对数据库中感兴趣的数据有变化时，进行监听。具体实现细节可以去看`ContentService`的源码是如何构建一个树形结构来管理观察者对感兴趣数据的监听，树的根节点就是上述的`mRootNode`；注册之后就是等待数据有变化时，进行监听了；下面我们就来看看这个观察者是如何被通知的。


当数据改变时就需要通知这个观察者，什么时候数据改变呢？不要忘了我们前面还有一个方法`getContext().getContentResolver().notifyChange(noteUri, null)`没有介绍呢；这个方法在`ContentProvider`中修改数据的方法最后被调用，干什么的？看名字也能猜个大概了。

扒它源码：
{% highlight java %}
public void notifyChange(Uri uri, ContentObserver observer, boolean syncToNetwork,
        int userHandle) {
    getContentService().notifyChange(...);
}
{% endhighlight %}

我就不Go on了...这个`ContentService#notifyChange()`方法比较复杂，我就描述一下它做的工作：因为我们注册观察者时对感兴趣的数据变化时才需要被通知到，所以此处遍历树形结构通知对变化感兴趣的观察者。

到此，这个观察者模式已经水落石出了！在`ContentProvider#query()`方法的最后注册的观察者，在`insert()、delete()、update()`方法的最后被通知；这样就实现了当数据发生改变时发送通知更新数据。

下面这张图大概描述了这个观察者的框架：
![framework_1](http://ww3.sinaimg.cn/large/bce2dea9jw1esvuwdwftwj20m80j640e.jpg)

下面我们就要来看看这个观察者被通知后做了什么工作。

## 观察者类

前面已经提到了这个观察者类是`AbstractCursor#SelfContentObserver`，这是一个非静态内部类：
{% highlight java %}
protected static class SelfContentObserver extends ContentObserver {
    ...

    @Override
    public void onChange(boolean selfChange) {
        // 这里调用了其依赖类的onChange()方法
        cursor.onChange(false); // false
    }
}
{% endhighlight %}

其中使用了弱引用来避免`Context`泄漏，我这边就不介绍什么`Context`泄漏了，有兴趣可以看看我的另一篇文章：[简析Handler造成的Context泄漏，以及解决方法](/post/2014/09/11/Handler-Leaks-Solution.html)。

当初这个观察者被注册到`ContentService`，现在数据源改变了，`ContentService`就会调用观察者的`onChange()`方法作为一个通知，`onChange()`方法调用了其外部类的`onChange()`方法。

Go on...

`AbstractCursor#onChange()`方法的源码：
{% highlight java %}
protected void onChange(boolean selfChange) {
    // 通知所有注册的观察者
    mContentObservable.dispatchChange(selfChange, null);
    // 这些代码不会执行
    if (mNotifyUri != null && selfChange) {
        mContentResolver.notifyChange(mNotifyUri, mSelfObserver);
    }
}
{% endhighlight %}

`ContentObservable`是什么呢？
{% highlight java %}
public class ContentObservable extends Observable<ContentObserver> {
    public void registerObserver(ContentObserver observer) {...}
    public void dispatchChange(boolean selfChange) {...}
    public void notifyChange(boolean selfChange) {...}
}
{% endhighlight %}

`ContentObservable`继承自android.database.Observable抽象类，从上面的方法可以看出它是专门用来注册`ContentObserver`的。

What... 

这里代码怎么又像是在通知观察者...没错，它的确是在通知观察者，这又是另一个观察者模式了，晕乎，下面我们来介绍第二个观察者模式。

# 第二个观察者模式

Ctrl + F搞起来... 一共找到下面几处（除了上面的`onChange()`方法）：
{% highlight java %}
ContentObservable mContentObservable = new ContentObservable();
...

public void registerContentObserver(ContentObserver observer) {
    mContentObservable.registerObserver(observer);
}

public void unregisterContentObserver(ContentObserver observer) {
    ...
    mContentObservable.unregisterObserver(observer);
}

public void close() {
    ...    
    mContentObservable.unregisterAll();
    ...
}
{% endhighlight %}

可以看出这里使用的是一种关联复用，`ContentObservable`负责管理`AbstractCursor`作为目标时接收注册观察者，此处`AbstractCursor`就是目标。


目标有了，观察者在哪呢？或者换一种角度，当数据源发生改变后，我们一路奔走相告通知到`AbstractCursor#onChange()`方法，那么接下来我们应该通知谁呢？通知我们的装载器！！

好吧，扒开`CursorLoader`的源码，在`CursorLoader`中负责后台加载数据的`loadInBackground()`方法找到了注册观察者的代码：
{% highlight java %}
ForceLoadContentObserver mObserver = new ForceLoadContentObserver();
...

public Cursor loadInBackground() {
    ...
    
    Cursor cursor = getContext().getContentResolver().query(mUri, mProjection, mSelection,
            mSelectionArgs, mSortOrder, mCancellationSignal);
    if (cursor != null) {
        try {
            // Ensure the cursor window is filled.
            cursor.getCount();
            cursor.registerContentObserver(mObserver);
        } catch (RuntimeException ex) {
            cursor.close();
            throw ex;
        }
    }
    
    ...
    return cursor;
}
{% endhighlight %}

`ContentObserver`是用来接收数据变化时的观察者，可以异步派发接收到的通知，在`Loader`中实现了它的子类`ForceLoadContentObserver`。

## 目标与观察者

- 目标：`AbstractCursor`
- 观察者：`Loader$ForceLoadContentObserver`

到此，第二个观察者模式也已经水落石出了！在`CursorLoader#loadInBackground()`方法中注册的观察者，在第一个观察者接收到通知后接着被通知；这样当数据发生改变时这个通知就传递到了真正干活的兄弟身上。

下面这张图大概描述了这个观察者的框架：
![framework_2](http://ww1.sinaimg.cn/large/bce2dea9jw1esvuwd8s1nj20m80le0vs.jpg)

下面我们就要来看看这个观察者里做了什么工作。

## 观察者

这个观察者类定义在`Loader`中，这也是一个非静态内部类：
{% highlight java %}
/**
 * 它实现了ContentObserver，当它被通知时负责调用装载器的方法重新加载数据；
 * 你通常不需要使用它；它被用在CursorLoader中，当游标装载的数据发生改变时执行
 * 更新操作。
 */
public final class ForceLoadContentObserver extends ContentObserver {
    ...
    
    @Override
    public void onChange(boolean selfChange) {
        onContentChanged();
    }
}
{% endhighlight %}

注意到它重写了`onChange()`方法，调用了`Cursor#onContentChanged()`方法，然后就是一层一层的实现，最终调用到了`CursorLoader#loadInBackground()`方法。

从上面的整个代码流程可以看到这个过程中使用两个层次的观察者模式：

下面是注册观察者的时候的大致流程：
![sequence](http://ww4.sinaimg.cn/large/bce2dea9jw1esvuwcdmg3j20ua0kk770.jpg)

下面是数据变化时通知更新流程图：
![sequence_2](http://ww4.sinaimg.cn/large/bce2dea9jw1esvuwcdmg3j20ua0kk770.jpg)

整个过程大致如上所述，有两个层次的观察者模式的应用。

<br/>
参考：

1. [Android装载器（Loaders）框架简介](/post/2014/11/14/Use-Loaders-in-Android.html)
2. [Android学习ContentProvider数据更新与Observer模式](http://www.cnblogs.com/bastard/archive/2012/06/02/2531663.html)
3. [What is cursor.setNotificationUri() used for?](http://stackoverflow.com/questions/21623714/what-is-cursor-setnotificationuri-used-for)
4. [CursorLoader not updating after notifyChange call](http://stackoverflow.com/questions/21273898/cursorloader-not-updating-after-notifychange-call)

本文出自[2dxgujun](/)，转载时请注明出处及相应链接。