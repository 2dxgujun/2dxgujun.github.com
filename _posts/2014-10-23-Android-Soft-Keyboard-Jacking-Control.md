---
layout: post
title: Android软键盘弹出时ActionBar和位于顶部的View被顶出去的问题
category: Android Dev
date: 2014-10-23
---

昨天在做一个页面，大概是这样子的：
![sample](/media/files/2014/10/23/sample.png)

<!-- more -->

顶部有一个原生的`ActionBar`和一个自定义的`ActionBar`，然后中间一段Content，最下面有一个`EditText`；当`EditText`获取焦点时，会弹出软键盘：
![soft_keyboard_pop](/media/files/2014/10/23/soft_keyboard_pop.png)
注意到顶部的原生`ActionBar`和我自定义的`ActionBar`被软键盘给顶出去了...我想要的效果是，顶部的两个`ActionBar`保持不动，然后下面的Content被顶上去，同时被`ActionBar`覆盖掉顶上去的那部分。

网上有一种解决方法说是给`Activity`设置`android:windowSoftInputMode="adjustPan"`，我试过，不管用。

百度了一下：**Android sdk目前提供的软键盘弹出模式接口只有两种，一是弹出时自动冲回界面，将所有元素上顶，另一种则是不重绘界面，直接将控件元素遮住，没有其他模式，如果想实现其他效果，光使用接口是不行的**。

但是实际上这个问题很容易解决，但是网上都是在贴子里讨论这个问题，众人给出来各种各样的Answer，大部分都是不可用，或者不适用于我这种情况的，所以我写下这篇文章总结一下这个问题。

#解决方法
---
解决问题的关键就在于`ScrollView`：给想要被顶上去的内容（包括下面那个`EditText`）嵌套一个`ScrollView`：
![scrollview_nested](/media/files/2014/10/23/scrollview_nested.png)

图中红色的框就是`ScrollView`应该在页面中所处的位置。

上布局代码：
{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical" >

    <TextView
        android:layout_width="match_parent"
        android:layout_height="48dp"
        android:background="#FFFF6702"
        android:gravity="center"
        android:text="Custom ActionBar"
        android:textColor="@android:color/white"
        android:textSize="20sp" />

    <ScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent" >

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:orientation="vertical" >

            <TextView
                android:layout_width="match_parent"
                android:layout_height="200dp"
                android:background="#FF33B5E5"
                android:gravity="center"
                android:text="Content"
                android:textColor="@android:color/white"
                android:textSize="50sp" />

            <EditText
                android:layout_width="match_parent"
                android:layout_height="50dp"
                android:layout_marginTop="50dp"
                android:text="EditText" />
        </LinearLayout>
    </ScrollView>

</LinearLayout>
{% endhighlight %}

如果你按照上面的写法，到这里这个问题已经解决了。

可是生活就是这样，到处充满了坑...我在项目中用了上面的方法还是没有解决问题，于是我开始一行行的对比代码，最后我剔除了所有不一样的地方，发现我给`ScrollView`设置了一个`android:scrollbars="none"`属性，把这个属性去掉问题就解决了...不太明白为什么，源码貌似涉及C++，看不到...

可是作为一个强迫症小青年，我偏不想去掉这个属性，怎么办？

我神奇的发现在`Activity`的`setContentView()`方法后面添加一条这样的语句，问题就解决了...
`getWindow().setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_ADJUST_RESIZE);`

下面我就来分析一下这个`android:windowSoftInputMode`属性

#android:windowSoftInputMode
---
这个属性用于设置`Activity`主窗口与软键盘的交互模式，可以用来避免软键盘面板遮挡内容的问题。
这个属性能影响两件事：

1. 当有焦点产生时，软键盘是隐藏还是显示
2. 是否调整`Activity`主窗口大小以便腾出空间放软键盘

它的设置必须是下面列表中的一个值，或一个"state"值加一个"adjust"值的组合，各个值之间用 | 分开。


- **stateUnspecified**：默认状态，系统将选择一个合适的状态或依赖于主题的设置。

- **stateUnchanged**：当这个`Activity`出现时，软键盘将一直保持上一个`Activity`里的状态，无论是隐藏还是显示。

- **stateHidden**：当这个`Activity`出现时，软键盘总是被隐藏。

- **stateAlwaysHidden**：当这个`Activity`主窗口获取焦点时，软键盘总是被隐藏。

- **stateVisible**：当这个`Activity`出现时（`Activity`中有可以获取输入焦点的控件），软键盘通常是可见的。

- **stateAlwaysVisible**：当这个`Activity`出现时，软键盘总是弹出。

- **adjustUnspecified**：默认行为，系统将根据窗口中的布局自动调整窗口大小。

- **adjustResize**：这个`Activity`主窗口总是调整屏幕的大小以便留出软键盘的空间；这个选项不能和adjustPan同时使用，如果这两个属性都没有被设置，系统会根据窗口中的布局自动选择其中一个。

- **adjustPan**：这个`Activity`主窗口并不调整屏幕的大小以便留出软键盘的空间；相反，当前窗口中获取输入焦点的控件会自动移动到软键盘上方以便用户总是能看到输入内容的部分；这个选项不能和adjustPan同时使用，如果这两个属性都没有被设置，系统会根据窗口中的布局自动选择其中一个。

<br/>
现在再回过头来看看我遇到的那个问题；我的猜测是没有设置`android:windowSoftInputMode`属性的主窗口会在弹出软键盘时检测当前需要做调整的布局中是否嵌套了`ScrollView`，再判断`ScrollView`是否设置了`android:scrollbars="none"`属性，如果设置了则不允许通过滚动`ScrollView`中的Content来达到调整屏幕内容留出用于放置软键盘空间的效果。


<br/>
参考：

1. [android:windowSoftInputMode属性使用](http://blog.csdn.net/gaomatrix/article/details/7057032)
2. [android:windowSoftInputMode属性详解](http://blog.csdn.net/twoicewoo/article/details/7384398)
3. [软键盘覆盖EditText，使用ScrollView的详解](http://www.eoeandroid.com/thread-53414-1-1.html)

本文出自[2dxgujun](http://2dxgujun.com/)，转载时请注明出处及相应链接。