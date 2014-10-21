---
layout: post
title: Measured Dimensions和Drawing Dimensions的不同
category: Android Dev
date: 2014-09-08
---


[Android Documentation](http://developer.android.com/reference/android/view/View.html#SizePaddingMargins)中说明一个View有两种尺寸：Measured Dimensions和Drawing Dimensions。

#Measured Dimensions
---
Measured Dimensions是在`onMeasure`方法中计算得到的，它是对`View`上的内容进行测量后得到的**`View`上的内容占据的尺寸**。

Documentation中有句话：
> The width of this view as measured in the most recent call to measure(). This should be used during measurement and layout calculations only.

这个尺寸是最近一次调用`measure`方法计算宽度后View的尺寸，它应该仅仅用在测量和布局的计算中。
但是如果你自己在实现布局，你可能要用到Measured Dimensions。

<!-- more -->

#Drawing Dimensions
---
Drawing Dimensions是`View`在设定好布局后**整个`View`的实际尺寸**。

Documentation中有一句话：
> These values may, but do not have to, be different from the measured width and height.

这个值有可能，但不是必须要和Measured Dimensions不同。



那么什么情况下这个Measured Dimensions和Drawing Dimensions会不同呢？

下面我们来看一个例子：

{% highlight java %}
public class MainActivity extends ActionBarActivity {
	private static final String LOG_TAG = MainActivity.class.getSimpleName();

	LinearLayout mLayout;
	MyTextView mTextView;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);

		mLayout = new LinearLayout(this);
		mLayout.setLayoutParams(new LinearLayout.LayoutParams(
				LinearLayout.LayoutParams.MATCH_PARENT,
				LinearLayout.LayoutParams.MATCH_PARENT));

		mTextView = new MyTextView(this);

		mLayout.addView(mTextView);
		setContentView(mLayout);
	}

	static class MyTextView extends TextView {
		public MyTextView(Context context) {
			super(context);
			setText("Hello World");
		}

		@Override
		protected void onDraw(Canvas canvas) {
			super.onDraw(canvas);
			Log.i(LOG_TAG, "width: " + getWidth() + ", height: " + getHeight());
			Log.i(LOG_TAG, "measuredWidth: " + getMeasuredWidth()
					+ ", measuredHeight: " + getMeasuredHeight());
		}
	}
}
{% endhighlight %}

Log Info: 
> width: 107, height: 29

> measuredWidth: 107, measuredHeight: 29

这里在`LinearLayout`里添加了一个`TextView`，如果要得到`TextView`的宽，那么是要在`TextView`添加到`LinearLayout`后再获取值，这里放在`onDraw`方法中。在一个`View`初始化时，即在构造函数当中我们是得不到`View`的实际大小的；`getWidth`和`getMeasuredWidth`方法得到的结果都是0，这是因为这个`View`还未完成布局尺寸的测量，当然也就没有完成布局了。关于如何正确地获取控件布局尺寸的方法，请参考[我的另一篇博文](http://2dxgujun.github.io/09-05-2014/Get-Drawing-Dimensions-after-Layout.html)。

可以看到Measured Dimensions和Drawing Dimensions是一样的，说明这个`TextView`实际尺寸和根据内容测量出来的尺寸是一致的。事实也正是如此，这个`TextView`的宽和高默认被初始化成`ViewGroup.LayoutParams.WRAP_CONTENT`，它的Measured Dimensions就是"Hello World"这个字符串的所占的宽高；因为这个`TextView`被完整的放置在`LinearLayout`中，Drawing Dimensions也就是内容的宽高。



下面将对上面的例子做一下修改。
{% highlight java %}
public class MainActivity extends ActionBarActivity {
	private static final String LOG_TAG = MainActivity.class.getSimpleName();

	MyTextView mTextView;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		mTextView = new MyTextView(this);
		setContentView(mTextView);
	}

	static class MyTextView extends TextView {
		public MyTextView(Context context) {
			super(context);
			setText("Hello World");
		}

		@Override
		protected void onDraw(Canvas canvas) {
			super.onDraw(canvas);
			measure(0, 0);
			Log.i(LOG_TAG, "width: " + getWidth() + ", height: " + getHeight());
			Log.i(LOG_TAG, "measuredWidth: " + getMeasuredWidth()
					+ ", measuredHeight: " + getMeasuredHeight());
		}
	}
}
{% endhighlight %}

Log Info: 
> width: 480, height: 690

> measuredWidth: 107, measuredHeight: 29

这里直接把`TextView`设置成`Activity`的`ContentView`。
看一下`setContentView`方法的文档：

> Set the activity content to an explicit view. This view is placed directly into the activity's view hierarchy. It can itself be a complex view hierarchy. **When calling this method, the layout parameters of the specified view are ignored. Both the width and the height of the view are set by default to ViewGroup.LayoutParams.MATCH_PARENT.** To use your own layout parameters, invoke setContentView(android.view.View, android.view.ViewGroup.LayoutParams) instead.

调用这个方法，指定的`View`的`LayoutParams`会被忽略，并且宽和高都会被设置成`ViewGroup.LayoutParams.MATCH_PARENT`。
也就是说这个`TextView`添加到`Activity`后，它的布局方案就是填充满整个`Activity`。


此时的Drawing Dimensions是`TextView`的实际尺寸，也就是屏幕的大小；而Measured Dimensions是`TextView`内的文字"Hello World"占的大小。


<br/>
参考：

1. [Android 一张图理解getWidth和getMeasuredWidth](http://blog.sina.com.cn/s/blog_6e519585010152s5.html)
2. [Android getWidth和getMeasuredWidth的正解](http://blog.csdn.net/wotoumingzxy/article/details/7760935)
3. [What is the difference between getWidth/heigth() and getMeasuredWidth/Heigth() in Android SDK?](http://stackoverflow.com/questions/8657540/what-is-the-difference-between-getwidth-heigth-and-getmeasuredwidth-heigth-i)
4. [Size, padding and margins](http://developer.android.com/reference/android/view/View.html#SizePaddingMargins)

本文出自[2dxgujun](http://github.com/2dxgujun)，转载时请注明出处及相应链接。