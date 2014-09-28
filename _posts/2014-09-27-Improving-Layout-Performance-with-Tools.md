---
title: Android布局的优化——使用优化工具
layout: post
guid: 2014092701
date: 2014-09-27 15:15:00
description: 
tags:
  - Android
---

Layout是Android应用中直接影响用户体验的关键部分；如果实现的不好，可能会导致你的应用消耗大量内存，同时应用会非出现卡顿。Android SDK工具集里面提供了可以帮助你定位Layout性能缺陷的工具，通过这些工具可以让你用最小的内存消耗实现流畅的UI。


就像一个复杂的网页会减慢载入速度，你的Layout结构如果太复杂，也会造成性能问题。本文教你如何使用SDK自带工具来检查Layout并找到性能瓶颈。

一个常见的误区是，用最基础的Layout结构可以使布局性能提高。然而，你的程序的每个组件和Layout都需要初始化、布置位置和绘制。例如，嵌套的`LinearLayout`可能会使得`View`的层级结构很深。此外，嵌套使用了`layout_weight`参数的 `LinearLayout`的计算量会尤其大，因为每个子元素都需要被测量两次。这对需要多次重复inflate的Layout尤其需要注意，比如使用`ListView`或`GridView`时。


#Inspect Your Layout（检查Layout）
---
Android SDK带有一个叫做[Hierarchy Viewer](https://developer.android.com/tools/help/hierarchy-viewer.html)的工具，能够在程序运行时分析Layout。你可以用这个工具找到Layout的性能瓶颈。

Hierarchy Viewer会让你选择设备或者模拟器上正在运行的进程，然后显示其Layout的树形结构。**每个块上的交通灯分别代表了它在测量、布置和绘画时的性能**，帮你找出瓶颈部分。

比如，下图是`ListView`中的一个列表项的Layout。它的左边是一个位图，右边是两个层叠的文字；像这种需要被多次inflate的Layout，优化它们会有事半功倍的效果。

![layout_listitem](/media/files/2014/09/27/layout_listitem.png)

`hierarchyviewer`这个工具在`<sdk>/tools/`中；当打开时，它显示一张可使用设备的列表，和它正在运行的组件。选中你要查看的组件；点击`Load View Hierarchy`来查看所选组件的层级。比如，下图就是前一个图中所示Layout的层级关系。

![hierarchy_linearlayout](/media/files/2014/09/27/hierarchy_linearlayout.png)

这张图中，你可以看到一个三层结构，其中布置在右下角的`TextView`三个交通灯都是黄色的表示它的存在一些布局缺陷；点击其中的项就会显示每个步骤所花费的时间。这样，谁花了多长时间在什么步骤上，就清晰可见了。

![hierarchy_layouttimes](/media/files/2014/09/27/hierarchy_layouttimes.png)

可以看到，渲染一个完整的列表项的时间就是：

- 测量：0.977ms
- 布置：0.167ms
- 绘制：2.717ms


<br/>
**注意**：以上内容是在Android官方文档中介绍的使用Hierarchy Viewer来检查Layout的方法；但是我在用这个工具检查Layout时手边两台机都出现了以下错误：

> Unable to debug device.

百度了一会儿，发现[Android官方文档](https://developer.android.com/tools/debugging/debugging-ui.html)上有说明：

> To preserve security, Hierarchy Viewer can only connect to devices running a developer version of the Android system.

也就是说Hierarchy Viewer只能连接搭载Android开发版系统的手机或模拟器。
Hierarchy Viewer在连接手机时，手机会启动View Server与其进行Socket通信；但在我们平常用的商业机上，是无法开启View Server的，实现这个限制的Android源码目录如下：

> /frameworks/base/services/java/com/android/server/wm/WindowManageService.java

检查一台手机是否开启了View Server的方法为：
> adb shell service call window 3

若返回值是：Result: Parcel(00000000 00000000 '........')，说明View Server处于关闭状态；    
若返回值是：Result: Parcel(00000000 00000001 '........')，说明View Server处于开启状态。


如果要在自己的手机上正常地使用Hierarchy Viewer，有两种方法：

1. 直接刷一个开发版本的Android固件
2. 如果只需要查看自己开发的应用的UI变化，可以用Github上的一个项目[ViewServer](https://github.com/romainguy/ViewServer)。

第一种方法不适合我，我就写一下使用ViewServer这个开源项目的简略过程：
由于ViewServer这个库没有发布到Maven，所以不能使用Maven或Gradle的Dependencies来添加库:(

首先去Github Download下来，使用Android Studio打开这个工程，可能需要一些配置（如Gradle版本的配置）；
完成之后运行`viewserver-sample`这个Module，启动之后，会打开一个Activity，如下
![activity_screenshot](/media/files/2014/09/27/activity_screenshot.png)

之后使用前面介绍的方法，运行`hierarchyviewer`工具，在视图中能看到设备下面出现了一个组件，选中之后，点击`Load View Hierarchy`就能显示组件的层级了。

![hierarchy_viewer_screenshot](/media/files/2014/09/27/hierarchy_viewer_screenshot.png)


#Revise Your Layout（修正Layout）
---
因为上面的Layout性能太慢，原因在这个嵌套的LinearLayout，解决的办法可能是将Layout层级变浅变宽，而不是又窄又深。RelativeLayout作为根节点时就可以达到目的。所以，当换成基于RelativeLayout的设计时，你的Layout变成了两层。新的Layout长成这样：

![hierarchy_relativelayout](/media/files/2014/09/27/hierarchy_relativelayout.png)

现在渲染列表项的时间：

- 测量：0.598ms
- 布置：0.110ms
- 绘制：2.146ms

可能看起来是很小的进步，但是由于它对列表中的每个项都有效，这个时间要翻倍。

更明显的性能差距，是当使用在LinearLayout中使用layout_weight的时候，因为会减慢“测量”的速度。这只是一个正确使用各种Layout的例子，当你使用layout_weight时一定要慎重。


#Use Lint（使用Lint检查工具）
---
经常运行Lint工具来检查Layout可能的优化方法，是个很好的实践。Lint已经取代了layoutopt工具，它拥有更强大的功能。Lint中包含的一些检测规则有：

- 使用复合`drawable`：用一个`drawable`替代一个包含`ImageView`和`TextView`的`LinearLayout`时会更有效率。
- 合并根`FrameLayout`：如果`FrameLayout`是`Layout`的根节点，并且没有使用`padding`或者背景等，那么用`merge`标签替代他们会稍微高效些。
- 没用的子节点：一个没有子节点或者背景的`Layout`应该被去掉，来提高性能。
- 没用的父节点：一个节点如果只有一个子节点，并且它不是`ScrollView`或根节点，并且它没有背景，这样的节点应该直接被子节点取代。
- 太深的Layout：Layout的嵌套层数太深对性能有很大影响；尝试使用更扁平的Layout，比如`RelativeLayout`或`GridLayout`来提高性能；一般最多不超过10层。



<br/>
参考：

1. [Improving Layout Performance](https://developer.android.com/training/improving-layouts/index.html)
2. [Hierarchy Viewer](https://developer.android.com/tools/help/hierarchy-viewer.html)
3. [layoutopt](https://developer.android.com/tools/help/layoutopt.html)
4. [Hierarchy Viewer无法连接真机调试](http://xie2010.blog.163.com/blog/static/211317365201402893433577/)

本文出自[2dxgujun](http://github.com/2dxgujun)，转载时请注明出处及相应链接。