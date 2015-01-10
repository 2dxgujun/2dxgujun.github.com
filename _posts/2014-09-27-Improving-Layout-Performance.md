---
layout: post
title: 三招优化Android布局
category: Android
date: 2014-09-27
---

Layout是Android应用中直接影响用户体验的关键部分，如果实现得不好，你的用户界面可能会出现卡顿。Android SDK提供了帮助你找到Layout性能瓶颈的工具，本节将会介绍如何使用这些工具，消耗小量的内存来实现流畅的UI。

# 优化Layout的层级

一个常见的误区是，用最基础的Layout结构可以使布局性能提高。然而，你的程序的每个组件和Layout都需要初始化、布置位置和绘制。例如，嵌套的`LinearLayout`可能会使得视图的层级结构很深。此外，**嵌套使用了`layout_weight`参数的 `LinearLayout`的计算量会尤其大，因为每个子元素都需要被测量两次。这对需要多次重复inflate的Layout尤其需要注意**，比如使用`ListView`或`GridView`时。

本节中，你将学习使用Hierarchy Viewer（层级浏览器）来检查和优化Layout。

<!-- more -->

## 检查Layout

Android SDK带有一个叫做[Hierarchy Viewer](https://developer.android.com/tools/help/hierarchy-viewer.html)的工具，能够在程序运行时分析Layout。你可以用这个工具找到Layout的性能瓶颈。

Hierarchy Viewer会让你选择设备或者模拟器上正在运行的进程，然后显示其Layout的树形结构。**每个块上的交通灯分别代表了它在测量、布置和绘画时的性能**，帮你找出瓶颈部分。

比如，下图是`ListView`中的一个列表项的Layout。它的左边是一个位图，右边是两个层叠的文字；像这种需要被多次inflate的Layout，优化它们会有事半功倍的效果。

![layout_listitem](/media/2014/09/27/layout_listitem.png)

hierarchyviewer这个工具在`<sdk>/tools/`中；当打开时，它显示一张可使用设备的列表，和它正在运行的组件。选中你要查看的组件；点击Load View Hierarchy来查看所选组件的层级。比如，下图就是前一个图中所示Layout的层级关系。

![hierarchy_linearlayout](/media/2014/09/27/hierarchy_linearlayout.png)

这张图中，你可以看到一个三层结构，其中布置在右下角的`TextView`三个交通灯都是黄色的表示它的存在一些布局缺陷；点击其中的项就会显示每个步骤所花费的时间。这样，谁花了多长时间在什么步骤上，就清晰可见了。

![hierarchy_layouttimes](/media/2014/09/27/hierarchy_layouttimes.png)

可以看到，渲染一个完整的列表项的时间就是：

- 测量：0.977ms
- 布置：0.167ms
- 绘制：2.717ms

**注意：**以上教程内容是在Android官方文档中介绍的使用Hierarchy Viewer来检查Layout的方法；但是我在用这个工具检查Layout时手边两台机都出现了以下错误：

> Unable to debug device.

百度了一会儿，发现[Android官方文档](https://developer.android.com/tools/debugging/debugging-ui.html)上有说明：

> To preserve security, Hierarchy Viewer can only connect to devices running a developer version of the Android system.

也就是说Hierarchy Viewer只能连接搭载Android开发版系统的手机或模拟器。
Hierarchy Viewer在连接手机时，手机会启动View Server与其进行Socket通信；但在我们平常用的商业机上，是无法开启View Server的，实现这个限制的Android源码目录如下：

> /frameworks/base/services/java/com/android/server/wm/WindowManageService.java

**检查一台手机是否开启了View Server的方法为**：
> adb shell service call window 3

若返回值是：Result: Parcel(00000000 00000000 '........')，说明View Server处于关闭状态；    
若返回值是：Result: Parcel(00000000 00000001 '........')，说明View Server处于开启状态。


如果要在自己的手机上正常地使用Hierarchy Viewer，有两种方法：

1. 直接刷一个开发版本的Android固件；
2. 如果只需要查看自己开发的应用的UI层级，可以用Github上的一个项目[ViewServer](https://github.com/romainguy/ViewServer)。

第一种方法不适合我，我就写一下使用ViewServer这个开源项目的简略过程：
由于ViewServer这个库没有发布到Maven，所以不能使用Maven或Gradle的Dependencies来添加库:(

首先去Github Download下来，使用Android Studio打开这个工程，可能需要一些配置（如Gradle版本的配置）；
完成之后运行`viewserver-sample`这个Module，启动之后，会打开一个Activity，如下
![activity_screenshot](/media/2014/09/27/activity_screenshot.png)

之后使用前面介绍的方法，运行`hierarchyviewer`工具，在视图中能看到设备下面出现了一个组件，选中之后，点击`Load View Hierarchy`就能显示组件的层级了。

![hierarchy_viewer_screenshot](/media/2014/09/27/hierarchy_viewer_screenshot.png)

## 修正Layout

因为上面的Layout性能太慢，原因在这个嵌套的`LinearLayout`，解决的办法可能是将Layout层级变浅变宽，而不是又窄又深。`RelativeLayout`作为根节点时就可以达到目的。所以，当换成基于`RelativeLayout`的布局时，你的Layout变成了两层。新的Layout长成这样：

![hierarchy_relativelayout](/media/2014/09/27/hierarchy_relativelayout.png)

现在渲染列表项的时间：

- 测量：0.598ms
- 布置：0.110ms
- 绘制：2.146ms

可能看起来是很小的进步，但是由于它对列表中的每个项都有效，这个时间要翻倍。

更明显的性能差距，是当使用在`LinearLayout`中使用`layout_weight`的时候，因为会减慢“测量”的速度。这只是一个正确使用各种Layout的例子，当你使用`layout_weight`时一定要慎重。

## 使用Lint工具
可以经常运行Lint工具来检查你的Layout，Lint已经取代了layoutopt工具，它拥有更强大的功能。Lint中包含的一些检测规则有：

- 使用复合`drawable`：用一个`drawable`替代一个包含`ImageView`和`TextView`的`LinearLayout`时会更有效率。
- 合并根`FrameLayout`：如果`FrameLayout`是`Layout`的根节点，并且没有使用`padding`或者背景等，那么用`merge`标签替代他们会稍微高效些。
- 没用的子节点：一个没有子节点或者背景的`Layout`应该被去掉，来提高性能。
- 没用的父节点：一个节点如果只有一个子节点，并且它不是`ScrollView`或根节点，并且它没有背景，这样的节点应该直接被子节点取代。
- 太深的Layout：Layout的嵌套层数太深对性能有很大影响；尝试使用更扁平的Layout，比如`RelativeLayout`或`GridLayout`来提高性能；一般最多不超过10层。


# 重用Layout

虽然Android提供很多小的可重用的交互组件，但是你仍然可能需要重用复杂一点的组件，这也许会用到Layout。为了高校重用整个Layout，你可以使用`<include>`和`<merge>`标签把其它Layout嵌入当前Layout。

重用Layout非常强大，它让你可以创建复杂的可重用Layout，比如，一个yes/no按钮面板，或者带有文字的自定义进度条。这还意味着，任何在多个Layout中重复出现的元素可以被提取出来，单独管理设计，再添加到Layout中。所以，当你要添加一个自定义View来实现单独的UI组件时，你可以更简单的直接重用某个Layout文件。

## 创建可重用的Layout

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

## include 标签

使用`<include>`标签，可以在Layout中添加可重用的组件。比如，这里有一个来自G-Kenya codelab的Layout需要包含上面的那个标题栏：
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

然而，如果你要在`<include>`中覆写这些属性，你必须先覆写`android:layout_width`和`android:layout_height`这两个属性。

**注意：**`<include>`标签若指定了id属性，而你的Layout也定义了id，则你的Layout的id会被覆盖。


**陷阱：**：如果我们在同一个Layout文件中多次包含引用同一个Layout的`<include>`标签，那么当我们通过`findViewById()`这个方法去获取被include的Layout中的某一个`View`时，哪一个`<include>`标签中的`View`会被获取到？（这个id产生了歧义）

**答案：**我们总是会获得第一个`<include>`标签引用的Layout中的`View`。

**解决方法：**首先通过`<include>`标签的id获取对应的的Layout，然后再通过Layout中的id获取的其中的`View`。代码示例如下：

{% highlight java %}
View container = findViewById(<include>标签的id); 
container.findViewById(Layout中View的id);
{% endhighlight %}

## merge 标签

`<merge>`标签在你嵌套Layout时取消了UI层级中冗余的`ViewGroup`。比如，如果你有一个Layout是一个垂直方向的`LinearLayout`，其中包含两个连续的`View`可以在别的`Layout`中重用，那么你会做一个`LinearLayout`来包含这两个`View`，以便重用。不过，当使用另一个`LinearLayout`来嵌套这个可重用的`LinearLayout`时，这种嵌套`LinearLayout`的方式除了减慢你的`UI`性能外没有任何意义。

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

# 按需载入Layout

除了简单的把一个Layout包含到另一个Layout中，你可能还想在程序开始后，仅当你的Layout对用户可见时才开始载入一些视图。各种不常用的布局像进度条、显示错误消息等可以使用`<ViewStub>`标签，以减少内存使用量，加快渲染速度。本节告诉你如何分步载入Layout来提高布局性能。

## 定义ViewStub

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

## 载入ViewStub

当你要载入用`ViewStub`声明的Layout时，要么用`setVisibility(View.VISIBLE)`设置它的可见性，要么调用其`inflate()`方法：

{% highlight java %}
((ViewStub) findViewById(R.id.stub_import)).setVisibility(View.VISIBLE);
// or
View importPanel = ((ViewStub) findViewById(R.id.stub_import)).inflate();
{% endhighlight %}

**注意：**如果你需要和这个Layout交互，`inflate()`方法会在渲染完成后返回被inflate的视图，所以你不需要再调用`findViewById()`去查找这个元素。

一旦`ViewStub`可见或是被inflate了，`ViewStub`元素就不存在了。取而代之的是被inflate的Layout，其id是`ViewStub`上的`android:inflatedId`属性。（`ViewStub`的`android:id`属性仅在`ViewStub`可见以前有用）

**注意：**`ViewStub`的一个缺陷是，它目前不支持使用`<merge>`标签的Layout。

**使用ViewStub时的技巧：**
如果你在Layout中放置了多个`ViewStub`，你可以使用下面的方法来简化Layout文件：
保留一个`ViewStub`，在`Activity`的`onCreate()`方法中执行如下代码：

{% highlight java %}
setContentView(R.layout.activity_main);
ViewStub = stub = (ViewStub) findViewById(R.id.stub);

stub.setInflateId(R.id.view1);
stub.setLayoutResource(R.layout.my_layout.xml);
View view1 = stub.inflate();
{% endhighlight %}

在运行时动态地设置`ViewStub`的资源文件和`inflateId`，然后调用`inflate()`方法生成`View`。

<br/>
参考：

1. [Improving Layout Performance](https://developer.android.com/training/improving-layouts/index.html)
2. [How many ViewStubs is too many for a single layout XML file?](http://stackoverflow.com/questions/3893669/how-many-viewstubs-is-too-many-for-a-single-layout-xml-file/3893821#3893821)
3. [Tech Stuff: Android <include/> layout pitfalls](http://www.coboltforge.com/2012/05/tech-stuff-layout/)
4. [Android抽象布局——include、merge、ViewStub](http://xyzlmn.blog.51cto.com/2532390/1344216)
5. [Improving Layout Performance](https://developer.android.com/training/improving-layouts/index.html)
6. [Hierarchy Viewer](https://developer.android.com/tools/help/hierarchy-viewer.html)
7. [layoutopt](https://developer.android.com/tools/help/layoutopt.html)
8. [Hierarchy Viewer无法连接真机调试](http://xie2010.blog.163.com/blog/static/211317365201402893433577/)

本文出自[2dxgujun](/)，转载时请注明出处及相应链接。