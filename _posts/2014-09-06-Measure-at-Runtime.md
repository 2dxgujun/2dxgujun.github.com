---
title: 运行时获取控件的尺寸
layout: post
guid: 2014090501
date: 2014-09-05 12:57:00
description: 介绍几种在运行时获得控件大小的方法
tags:
  - Android
---


今天在项目中遇到一个问题，需要使用`GridView`来显示一组图片，图片要求长宽相等，而且图片之间有一定的间距，类似于QQ里面在写动态时添加图片的网格；需要适配不同分辨率的屏幕，每行固定显示4个Item。

这就需要在运行时计算`GridView`的宽度，由此计算出`GridView`中每个Item的宽度和高度。这里遇到一个很关键的问题：**如果你在`onCreate`等负责初始化的方法内调用`GradView`的`getWidth`方法，你将得到一个0！**

查找了相关的资料之后得知这是因为太早调用`getWidth`方法，这时控件还未完成布局尺寸的测量。当界面中的控件全部完成初始化并被添加到它们的父控件之后，整个窗口内界面的布局才算完成。

下面我将介绍几种不同的方法来获取控件的尺寸，各有利弊，最佳解决方案请参考[Best Solution](#best_solution)。

#Override View's layoutChildren Method
---
[API文档](https://developer.android.com/reference/android/widget/AbsListView.html#layoutChildren())中说明`AbsListView`的子类必须覆盖这个方法来布局子Item，而在布局子Item时，父控件的尺寸一定已经完成计算了，所以可以通过覆盖这个方法来获取父控件的尺寸。

**注意：**这个回调方法会在子Item布局发生改变时多次回调，所以应该避免多次执行相同的代码。

#Override View's onWindowFocusChanged Method
---
查阅相关资料得知，`Activity`生命周期中，`onCreate`，`onStart`，`onResume`，都不是真正的visable的时间点，真正的visable时间点是`onWindowFocusChanged`函数被执行时，`onWindowFocusChanged`在当前`Activity`得到或者失去焦点时调用。

**注意：**在某些情况下`onWindowFocusChanged`不会被调用，所以**尽量不要使用这种方法来获取控件尺寸**。

#Override View's onSizeChanged Method
---
**注意：**这个方法最初几次回调可能会获取到0，需要实现一些判断的逻辑，避免在计算布局尺寸时出现异常，**不推荐使用**。

#Listen to Draw/Layout Events: ViewTreeObserver
---
`ViewTreeObserver`用来接收全局布局事件，使用`ViewTreeObserver`来获取控件尺寸，可以不用自定义控件。
**注意：**`ViewTreeObserver`会在各种布局事件触发时调用（例如，当一个控件可见或不可见），所以不要忘记在不需要使用的时候移除这个监听器。

{% highlight java %}
mGridView.getViewTreeObserver().addOnGlobalLayoutListener(
		new ViewTreeObserver.OnGlobalLayoutListener() {
			@Override
			public void onGlobalLayout() {
				Log.i(LOG_TAG, "onGlobalLayout called, width: "
						+ mGridView.getWidth());

                // Ensure you call it only once
				if (Build.VERSION.SDK_INT < Build.VERSION_CODES.JELLY_BEAN) {
					mGridView.getViewTreeObserver().removeGlobalOnLayoutListener(this);
				} else {
					mGridView.getViewTreeObserver().removeOnGlobalLayoutListener(this);
				}
			}
		});
{% endhighlight %}


#Add a Runnable to the Layout queue: View.post() <a id="best_solution"></a>
---
这个方法不太为人所知，但我认为这是最好的解决方案。调用控件的`post()`方法，传入自定义的`Runnable`对象，这个`Runnable`对象会控件完成布局之后添加到消息队列中。

UI事件队列会按顺序处理消息，在调用`setContentView()`方法之后，这个事件队列中就会包含一个请求界面重新进行布局的消息，所以你之后发送给这个事件队列的消息会在布局完成之后执行。

{% highlight java %}
mGridView.post(new Runnable() {
	@Override
	public void run() {
		Log.i(LOG_TAG, "Runnale called, width: " + mGridView.getWidth());
	}
});
{% endhighlight %}

这种方式比`ViewTreeObserver`更好：你的代码将只会被执行一次，你不用在代码执行完之后注销观察者对象。

<br/>
参考：

1. [getWidth() returns 0](http://stackoverflow.com/questions/3591784/getwidth-returns-0)
2. [onWindowFocusChanged的作用](http://blog.csdn.net/pi9nc/article/details/9237031)

本文出自[2dxgujun](http://github.com/2dxgujun)，转载时请注明出处及相应链接。