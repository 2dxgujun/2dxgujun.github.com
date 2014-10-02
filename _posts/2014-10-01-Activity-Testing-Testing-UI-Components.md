---
title: Activity Testing——测试UI组件
layout: post
guid: 2014100103
date: 2014-10-01 15:12:00
description: 
tags:
  - Android
---

通常情况下，你的`Activity`中包含很多用户界面组件（如`Button`、`CheckBox`、`EditText`等等）允许你的用户和Android应用程序进行交互。本节将介绍如何测试一个简单的按钮交互界面。你可以使用相同的步骤来测试其它的，更复杂的UI组件。

**注意：**
这一节中介绍的UI测试叫做白盒测试，因为你拥有被测试应用程序的源码。Android [Instrumentation](https://developer.android.com/tools/testing/testing_android.html#Instrumentation)测试框架适用于创建应用程序中UI部件的白盒测试。用户界面测试的另一种类型是黑盒测试，就是那种你无法得到应用程序源码的测试方法。这种类型的测试可以用来测试你的应用程序是如何和其它应用程序或与系统进行交互的。本节不包括黑盒测试，想要了解更多关于如何在你的Android应用程序中进行黑盒测试，请参看[UI Testing guide](https://developer.android.com/tools/testing/testing_ui.html)。

完整的测试用例代码，请参考实例工程[AndroidTestingFun.zip](/media/files/2014/10/01/AndroidTestingFun.zip)中的ClickFunActivityTest.java。


#使用Instrumentation框架创建一个UI测试用例
---
当你在测试一个包含用户界面的`Activity`时，被测`Activity`运行在UI线程，但是测试程序运行在被测应用进程中的另外一个线程。这意味着，你的测试代码可以引用UI线程的对象，但是如果你尝试修改它们的属性或者给UI发送事件，你将会获得一个`WrongThreadException`异常。

为了安全的注入`Intent`对象到你的`Activity`或者在UI线程运行测试方法，你可以继承`ActivityInstrumentationTestCase2`这个类。想要了解跟多关于如何在UI线程运行测试的方法，请参看[Testing on the UI thread](https://developer.android.com/tools/testing/activity_testing.html#RunOnUIThread)。

##设置测试夹具
当你在设置UI测试的夹具时，你需要在`setUp()`方法中指定[touch mode](https://developer.android.com/guide/topics/ui/ui-events.html#TouchMode)。把touch mode设置成true可以防止UI组件抢夺你编程指定的点击方法的焦点事件（比如，一个按钮通过编程可以触发它的点击监听器），确保你在调用`getActivity()`方法之前调用了`setActivityInitialTouchMode()`方法。

例如：

{% highlight java %}
public class ClickFunActivityTest
        extends ActivityInstrumentationTestCase2 {
    ...
    @Override
    protected void setUp() throws Exception {
        super.setUp();

        setActivityInitialTouchMode(true);

        mClickFunActivity = getActivity();
        mClickMeButton = (Button) 
                mClickFunActivity
                .findViewById(R.id.launch_next_activity_button);
        mInfoTextView = (TextView) 
                mClickFunActivity.findViewById(R.id.info_text_view);
    }
}
{% endhighlight %}


#添加测试方法验证UI表现
---
你的UI测试目标应该包括：

- 检验`Activity`启动后`Button`按照正确布局显示
- 检验`TextView`初始化时是隐藏的
- 检验`TextView`在`Button`点击时显示预期的字符串

接下来的部分会示范如何编写测试方法，完成上面的测试目标。

##验证Button布局参数
你应该添加一个像这样的测试方法来验证按钮是否正确地显示在你的`Activity`中：

{% highlight java %}
@MediumTest
public void testClickMeButton_layout() {
    final View decorView = mClickFunActivity.getWindow().getDecorView();

    ViewAsserts.assertOnScreen(decorView, mClickMeButton);

    final ViewGroup.LayoutParams layoutParams =
            mClickMeButton.getLayoutParams();
    assertNotNull(layoutParams);
    assertEquals(layoutParams.width, WindowManager.LayoutParams.MATCH_PARENT);
    assertEquals(layoutParams.height, WindowManager.LayoutParams.WRAP_CONTENT);
}
{% endhighlight %}

在调用`assertOnScreen()`方法时，你应该传入一个根视图和你期望呈现在根视图上的`View`作为参数，如果你期望的`View`没有呈现在根视图上，该方法会抛出一个`AssertionFailedError`异常。

你也可以通过获取一个`ViewGroup.LayoutParams`对象的引用验证`Button`的布局是否正确，然后调用断言方法验证`Button`对象的宽高属性值是否和预期值一致。

`@MediumTest`注解用于把测试方法按规模大小归类，这和测试的绝对执行时间有关。要了解更多关于如何使用测试规模注解，请参考[使用测试注解](#使用测试注解)。


##验证TextView的布局参数
你应该像这样添加一个测试方法来验证一个`TextView`最初是隐藏在你的`Activity`中的：

{% highlight java %}
@MediumTest
public void testInfoTextView_layout() {
    final View decorView = mClickFunActivity.getWindow().getDecorView();
    ViewAsserts.assertOnScreen(decorView, mInfoTextView);
    assertTrue(View.GONE == mInfoTextView.getVisibility());
}
{% endhighlight %}

你可以调用`getDecorView()`方法得到一个`Activity`中decor view的引用，decor view是布局层次视图中最上层的`ViewGroup`（`FrameLayout`）。


##验证Button行为
你可以使用一个像这样的测试方法来验证当按下按钮时`TextView`变得可见：

{% highlight java %}
@MediumTest
public void testClickMeButton_clickButtonAndExpectInfoText() {
    String expectedInfoText = mClickFunActivity.getString(R.string.info_text);
    TouchUtils.clickView(this, mClickMeButton);
    assertTrue(View.VISIBLE == mInfoTextView.getVisibility());
    assertEquals(expectedInfoText, mInfoTextView.getText());
}
{% endhighlight %}

在你的测试代码中可以调用`TouchUtils`的`clickView()`方法以编程的方式点击一个按钮；你必须给这个方法传递一个正在运行的测试用例的引用和要操作的按钮的引用。

**注意：**
`TouchUtils`助手类提供一些可以方便地进行模拟触摸操作的方法；你可以使用这些方法来模拟点击、轻敲、拖动`View`或屏幕。

**警告：**
`TouchUtils`方法的目的是将事件安全地从测试线程发送到UI线程；你不应该把`TouchUtils`用在UI线程或任何标注`@UIThreadTest`的测试方法，这样做可能会抛出`WrongThreadException`异常。

<a id="使用测试注解"></a>
#使用测试注解
---
下面的注解可以用于标识测试方法的规模：

[@SmallTest](https://developer.android.com/reference/android/test/suitebuilder/annotation/SmallTest.html)：小规模测试方法

[@MediumTest](https://developer.android.com/reference/android/test/suitebuilder/annotation/MediumTest.html)：中规模测试方法

[@LargeTest](https://developer.android.com/reference/android/test/suitebuilder/annotation/LargeTest.html)：大规模测试方法


通常情况下，只需要几毫秒的时间的测试方法应该被标记为`@SmallTest`；长时间运行的测试（100毫秒或更多）通常被标记为`@MediumTest`或`@LargeTest`，测试时间主要取决于该测试是否访问了网络或本地系统的资源。可以参看[Android Tools Protip](https://plus.sandbox.google.com/+AndroidDevelopers/posts/TPy1EeSaSg8)指导你更好的使用测试注解。

你可以使用其它测试注解来控制测试的组织和运行。要了解更多关于其他注释的信息，见[Annotation](https://developer.android.com/reference/java/lang/annotation/Annotation.html)类参考。


浏览本系列的其它文章：

1. [建立测试环境](http://2dxgujun.github.io/10-01-2014/Activity-Testing-Setting-Up-Your-Test-Environment.html)
2. [创建和运行一个测试用例](http://2dxgujun.github.io/10-01-2014/Activity-Testing-Creating-and-Running-a-Test-Case.html)
3. [创建单元测试](http://2dxgujun.github.io/10-01-2014/Activity-Testing-Creating-Unit-Tests.html)
4. [创建功能测试](http://2dxgujun.github.io/10-01-2014/Activity-Testing-Creating-Functional-Tests.html)


<br/>
参考：

1. [Testing UI Components](https://developer.android.com/training/activity-testing/activity-ui-testing.html)

本文出自[2dxgujun](http://github.com/2dxgujun)，转载时请注明出处及相应链接。