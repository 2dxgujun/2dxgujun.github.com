---
title: Activity Testing——创建单元测试
layout: post
guid: 2014100104
date: 2014-10-01 15:12:00
description: 
tags:
  - Android
---

对`Activity`进行单元测试可以快速验证一个`Activity`的状态以及它和与其它独立组件间的交互方式。一个单元测试通常只测试应用代码中尽可能小的代码块（可以是一个方法，类，或者组件），而且不应该依赖于系统或网络资源。比如说，你可以写一个单元测试去检查一个`Activity`是否有正确的布局或者触发它的`Intent`对象是否正确。

**单元测试一般不适合测试与系统有复杂交互事件的UI**。相反，你应该使用像ActivityInstrumentationTestCase2这样的类，参考Activity Testing系列的另一篇文章：[测试UI组件](http://2dxgujun.github.io/10-01-2014/Activity-Testing-Testing-UI-Components.html)

这节内容将会教你编写一个单元测试来验证一个`Intent`是否正确触发了另一个`Activity`。由于测试运行在一个独立的环境中，所以`Intent`没有被发送到`Android`系统中（不会真正启动`Activity`），但你还是可以验证`Intent`对象中携带的有效数据的正确性。

完整的测试用例代码，请参考实例工程[AndroidTestingFun](/media/files/2014/10/01/AndroidTestingFun.zip)中的LaunchActivityTest.java。

**注意：**
要针对系统或外部依赖进行测试，你可以使用来自Mocking框架的`Mock`类，并把它们注入到你的单元测试中。要了解更多关于Android提供的Mocking框架，请参看[Mock Object Classes](http://developer.android.com/tools/testing/testing_android.html#MockObjectClasses})。


#为Activity的单元测试创建一个测试用例
---
`ActivityUnitTestCase`类支持对单个`Activity`进行测试。要为你的`Activity`创建一个单元测试，你的测试类应该继承自[ActivityUnitTestCase](http://developer.android.com/reference/android/test/ActivityUnitTestCase.html)。

`ActivityUnitTestCase`中的`Activity`不会被Android Instrumentation测试框架自动启动；要单独启动`Activity`，你需要显式的调用`startActivity()`方法，并传递一个`Intent`来启动你的目标`Activity`。

例如:

{% highlight java %}
public class LaunchActivityTest
        extends ActivityUnitTestCase<LaunchActivity> {
    ...

    @Override
    protected void setUp() throws Exception {
        super.setUp();
        mLaunchIntent = new Intent(getInstrumentation()
                .getTargetContext(), LaunchActivity.class);
        startActivity(mLaunchIntent, null, null);
        final Button launchNextButton =
                (Button) getActivity()
                .findViewById(R.id.launch_next_activity_button);
    }
}
{% endhighlight %}

<br/>
#验证另一个Activity的启动
---
你进行测试的目的可能包括：

- 验证当`Button`被按下时启动的`LaunchActivity`是否正确。
- 验证用于启动`Activity`的`Intent`是否包含有效数据。

为了验证当`Button`按下时是否发送了一个`Intent`，你可以调用`getStartedActivityIntent()`方法；通过使用断言，你可以验证返回的`Intent`是否为空，以及是否包含了预期的数据来启动下一个`Activity`；如果两个断言值都是true，那么就成功的验证了你的`Activity`发送的`Intent`的正确性。

你应该像这样实现你的测试方法：

{% highlight java %}
@MediumTest
public void testNextActivityWasLaunchedWithIntent() {
    startActivity(mLaunchIntent, null, null);
    final Button launchNextButton =
            (Button) getActivity()
            .findViewById(R.id.launch_next_activity_button);
    launchNextButton.performClick();

    final Intent launchIntent = getStartedActivityIntent();
    assertNotNull("Intent was null", launchIntent);
    assertTrue(isFinishCalled());

    final String payload =
            launchIntent.getStringExtra(NextActivity.EXTRAS_PAYLOAD_KEY);
    assertEquals("Payload is empty", LaunchActivity.STRING_PAYLOAD, payload);
}
{% endhighlight %}

因为`LaunchActivity`是独立运行的，所以不可以使用`TouchUtils`库来操作UI。你可以调用`preformClick()`方法来触发`Button`的点击事件。


浏览本系列的其它文章：

1. [建立测试环境](http://2dxgujun.github.io/10-01-2014/Activity-Testing-Setting-Up-Your-Test-Environment.html)
2. [创建和运行一个测试用例](http://2dxgujun.github.io/10-01-2014/Activity-Testing-Creating-and-Running-a-Test-Case.html)
3. [测试UI组件](http://2dxgujun.github.io/10-01-2014/Activity-Testing-Testing-UI-Components.html)
4. [创建功能测试](http://2dxgujun.github.io/10-01-2014/Activity-Testing-Creating-Functional-Tests.html)

<br/>
参考：

1. [Creating Unit Tests](https://developer.android.com/training/activity-testing/activity-unit-testing.html)

本文出自[2dxgujun](http://github.com/2dxgujun)，转载时请注明出处及相应链接。