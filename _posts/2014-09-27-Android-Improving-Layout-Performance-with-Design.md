---
layout: post
title: Android布局的优化——改良设计和代码
category: Android Dev
---

今天被个推SDK的给坑的，哎，个推把推送后发送`Notification`的方法给封装起来了，且在外部没有给第三方留下任何处理推送内容的接口；只给我留了一个`notification.xml`布局文件作为通知的布局，默认的UI效果看了简直就想吐……无奈，只好翻看Android源码找到原生`Notification`的布局文件，拿来凑活着用：布局文件名是`notification_template_base.xml`，

在这个原生的布局中遇到了一个`ViewStub`，看着好像挺有用的样子……嗯

{% highlight xml %}
<ViewStub
    android:id="@+id/time"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_weight="0"
    android:visibility="gone"
    android:layout="@layout/notification_template_part_time" />
{% endhighlight %}

今天时间不是太紧，最近的项目也是在做布局工作，所以就查了点资料写了这篇学习性质的博文。

<!-- more -->

#使用`<include/>`标签重用Layout
---
Android提供了很多小的可重用的组件，但是你可能需要重用一些Layout，Android提供了`<include/>`标签可以把其他Layout嵌入到当前Layout。

它可以让你创建复杂的可重用Layout；比如，一个yes/no的按钮面板。这意味着，任何在多个Layout中重复出现的元素可以被提取出来，单独设计管理，再添加到Layout中。


如果你已经知道你需要重用的Layout，先创建一个新的Layout的XML文件。比如下面是一个来自G-Kenya codelab的Layout，定义了一个需要添加到每个`Activity`中的标题栏（titlebar.xml）:

{% highlight xml %}
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width=”match_parent”
    android:layout_height="wrap_content"
    android:background="@color/titlebar_bg">

    <ImageView android:layout_width="wrap_content"
               android:layout_height="wrap_content"
               android:src="@drawable/gafricalogo" />
</FrameLayout>
{% endhighlight %}

根节点就是你想添加入的Layout。当需要显示这个标题栏时，就应该添加这个Layout。

比如，这里有一个来自G-Kenya codelab的Layout需要包含上面的那个标题栏：

{% highlight xml %}
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width=”match_parent”
    android:layout_height=”match_parent”
    android:background="@color/app_bg"
    android:gravity="center_horizontal">

    <include layout="@layout/titlebar"/>

    <TextView android:layout_width=”match_parent”
              android:layout_height="wrap_content"
              android:text="@string/hello"
              android:padding="10dp" />
    ...
</LinearLayout>
{% endhighlight %}

你也可以覆写被include元素的根元素的所有layout参数（任何`android:layout_*`属性）。比如：

{% highlight xml %}
<include android:id="@+id/news_title"
         android:layout_width="match_parent"
         android:layout_height="match_parent"
         layout="@layout/title"/>
{% endhighlight %}

然而，如果你要在`<include/>`中覆写这些属性，你必须先覆写`android:layout_width`和 `android:layout_height`属性。

**注意**：`<include/>`标签若指定了id属性，而你的Layout也定义了id，则你的Layout的id会被覆盖。


**使用`<include/>`标签时的陷阱：**
如果我们在同一个Layout文件中多次包含引用同一个Layout的`<include/>`标签，那么当我们通过`findViewById()`这个方法去获取被include的Layout中的某一个`View`时，哪一个`<include/>`标签中的`View`会被获取到？（这个id产生了歧义）

**答案：**
我们总是会获得第一个`<include/>`标签引用的Layout中的`View`。

**解决方法：**
首先通过`<include/>`标签中的id获取对应的的`View`，然后再通过Layout中的id获取的`View`上的那个具体控件。代码示例如下：

{% highlight java %}
View container = findViewById(<include/>中的id); 
container.findViewById(被include的Layout中的id);
{% endhighlight %}

#使用`<merge/>`标签优化使用`<include/>`标签产生的`ViewGroup`冗余
---
`<merge/>`标签在你嵌套Layout时取消了UI层级中冗余的`ViewGroup`。比如，如果你有一个Layout是一个垂直方向的`LinearLayout`，其中包含两个连续的`View`可以在别的`Layout`中重用，那么你会做一个`LinearLayout`来包含这两个`View`，以便重用。不过，当使用另一个`LinearLayout`来嵌套这个可重用的`LinearLayout`时，这种嵌套`LinearLayout`的方式除了减慢你的`UI`性能外没有任何意义。

为了避免这种情况，你可以用`<merge>`元素来替代可重用Layout的根节点。例如：

{% highlight xml %}
<merge xmlns:android="http://schemas.android.com/apk/res/android">
    <Button
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:text="@string/add"/>

    <Button
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:text="@string/delete"/>
</merge>
{% endhighlight %}

现在，当你要将这个Layout包含到另一个Layout中时（并且使用了`<include/>`标签），系统会直接把两个`Button`放到Layout中，而不会有多余的`ViewGroup`被嵌套。

#使用`ViewStub`按需加载`View`
---
除了简单的把一个Layout包含到另一个中，你可能还想在程序开始后，仅当你的Layout对用户可见时才开始载入一些视图。各种不常用的布局像进度条、显示错误消息等可以使用`<ViewStub/>`标签，以减少内存使用量，加快渲染速度。本节告诉你如何分步载入Layout来提高布局性能。

[ViewStub](https://developer.android.com/reference/android/view/ViewStub.html)是一个轻量的视图，不需要大小信息，也不会在被加入的Layout中绘制任何东西。每个`ViewStub`只需要设置`android:layout`属性来指定需要被inflate的Layout。

下面的`ViewStub`是一个半透明的进度条覆盖层。功能上讲，它应该只在新的数据项被导入到应用程序时可见。

{% highlight xml %}
<ViewStub
    android:id="@+id/stub_import"
    android:inflatedId="@+id/panel_import"
    android:layout="@layout/progress_overlay"
    android:layout_width="fill_parent"
    android:layout_height="wrap_content"
    android:layout_gravity="bottom" />
{% endhighlight %}

当你要载入用`ViewStub`声明的Layout时，要么用`setVisibility(View.VISIBLE)`设置它的可见性，要么调用其`inflate()`方法：

{% highlight java %}
((ViewStub) findViewById(R.id.stub_import)).setVisibility(View.VISIBLE);
// or
View importPanel = ((ViewStub) findViewById(R.id.stub_import)).inflate();
{% endhighlight %}

**注意：**
如果你需要和这个Layout交互，`inflate()`方法会在渲染完成后返回被inflate的视图，所以你不需要再调用`findViewById()`去查找这个元素。

一旦`ViewStub`可见或是被inflate了，`ViewStub`元素就不存在了。取而代之的是被inflate的Layout，其id是`ViewStub`上的`android:inflatedId`属性。（`ViewStub`的`android:id`属性仅在`ViewStub`可见以前可用）

**注意：**
`ViewStub`的一个缺陷是，它目前不支持使用`<merge/>`标签的Layout。

**使用ViewStub时的技巧：**
如果你在Layout中放置了多个ViewStub，你可以使用下面的方法来简化Layout文件：
保留一个ViewStub，在Activity的onCreate()方法中执行如下代码：

{% highlight java %}
setContentView(R.layout.activity_main);
ViewStub = stub = (ViewStub) findViewById(R.id.stub);

stub.setInflateId(R.id.view1);
stub.setLayoutResource(R.layout.my_layout.xml);
View view1 = stub.inflate();
{% endhighlight %}

在运行时动态地设置`ViewStub`的资源文件和`inflateId`，然后调用`inflate()`方法生成`View`。

#通过优化，使`ListView`的滑动更加顺畅
---
如果你有一个包含复杂或者每个项（Item）包含很多数据的`ListView`，那么上下滚动的性能可能会降低。本节给你一些关于如何把滚动变得更流畅的提示。

**保持程序流畅的关键是让主线程（UI线程）不要进行大量运算**。你要确保在其他线程执行磁盘读写、网络读写或者SQL操作等。为了测试你的应用的状态，你可以启用[StrictMode](https://developer.android.com/reference/android/os/StrictMode.html)（严格模式）。

你应该把主线程中的耗时间的操作，提取到一个**后台线程**（也叫做“工人线程”，**Worker Thread**）中，使得主线程只关注UI绘画。很多时候，使用`AsyncTask`是一个简单的在主线程以外进行操作的方法。系统使用一个全局队列来管理所有的`execute()`请求。这个行为是全局的，这意味着你不需要考虑自己定义线程池的事情。

在以下的例子中，一个`AsyncTask`被用于在后台线程载入图片，并在载入完成后把图片放入UI。当图片正在载入时，它还会显示一个进度条。

{% highlight java %}
// Using an AsyncTask to load the slow images in a background thread
new AsyncTask<ViewHolder, Void, Bitmap>() {
    private ViewHolder v;

    @Override
    protected Bitmap doInBackground(ViewHolder... params) {
        v = params[0];
        return mFakeImageLoader.getImage();
    }

    @Override
    protected void onPostExecute(Bitmap result) {
        super.onPostExecute(result);
        if (v.position == position) {
            // If this item hasn't been recycled already, hide the
            // progress and set and show the image
            v.progress.setVisibility(View.GONE);
            v.icon.setVisibility(View.VISIBLE);
            v.icon.setImageBitmap(result);
        }
    }
}.execute(holder);
{% endhighlight %}

从Android 3.0(API level 11)开始，`AsyncTask`有个新特性，那就是它可以在多个CPU核心上运行。你可以调用`executeOnExecutor()`来自动根据核数，在多核上执行任务，而不是调用`execute()`。


你的代码可能在`ListView`滑动时经常使用`findViewById()`，这样会降低性能。即使是Adapter返回一个用于回收的inflate后的视图，你仍然需要查看这个元素并更新它。避免频繁调用`findViewById()`的方法之一，就是使用**View Holder（视图占位符）设计模式**。

一个`ViewHolder`对象存储了他的标签下的每个视图。这样你不用频繁查找这个元素。第一，你需要创建一个类来存储你会用到的视图。比如：

{% highlight java %}
static class ViewHolder {
    TextView text;
    TextView timestamp;
    ImageView icon;
    ProgressBar progress;
    int position;
}
{% endhighlight %}

然后，在Layout的类中生成一个`ViewHolder`对象：

{% highlight java %}
ViewHolder holder = new ViewHolder();
holder.icon = (ImageView) convertView.findViewById(R.id.listitem_image);
holder.text = (TextView) convertView.findViewById(R.id.listitem_text);
holder.timestamp = (TextView) convertView.findViewById(R.id.listitem_timestamp);
holder.progress = (ProgressBar) convertView.findViewById(R.id.progress_spinner);
convertView.setTag(holder);
{% endhighlight %}

这样你就可以轻松获取每个视图，而不是用`findViewById()`来不断查找视图，节省了宝贵的运算时间。


<br/>
参考：

1. [Improving Layout Performance](https://developer.android.com/training/improving-layouts/index.html)
2. [How many ViewStubs is too many for a single layout XML file?](http://stackoverflow.com/questions/3893669/how-many-viewstubs-is-too-many-for-a-single-layout-xml-file/3893821#3893821)
3. [Tech Stuff: Android <include/> layout pitfalls](http://www.coboltforge.com/2012/05/tech-stuff-layout/)
4. [Android抽象布局——include、merge、ViewStub](http://xyzlmn.blog.51cto.com/2532390/1344216)

本文出自[2dxgujun](http://github.com/2dxgujun)，转载时请注明出处及相应链接。