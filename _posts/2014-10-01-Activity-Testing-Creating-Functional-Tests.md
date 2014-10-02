---
title: Activity Testing——创建功能测试
layout: post
guid: 2014100105
date: 2014-10-01 15:13:00
description: 
tags:
  - Android
---


功能测试包括验证独立的应用组件是否像用户期望的那样协同工作。比如，你可以创建一个功能测试验证在用户执行UI交互时`Activity`是否正确启动了目标`Activity`。

要为你的`Activity`创建功能测试，你的测试类应该扩展自`ActivityInstrumentationTestCase2`。**与`ActivityUnitTestCase`不同的是在`ActivityInstrumentationTestCase2`中可以与Android系统通信以及发送键盘输入和点击事件到UI**。

完整的测试用例代码，请参考实例工程[AndroidTestingFun.zip](/media/files/2014/10/01/AndroidTestingFun.zip)中的SenderActivityTest.java。

#添加测试方法验证函数的行为
---
你进行的功能测试目标应该包括：

- 验证在`SenderActivity`的UI交互是否正确启动了目标`Activity`。
- 验证目标`Activity`中呈现的数据是否是`SenderActivity`所提供的。

你应该这样实现你的测试方法：

{% highlight java %}
@MediumTest
public void testSendMessageToReceiverActivity() {
    final Button sendToReceiverButton = (Button) 
            mSenderActivity.findViewById(R.id.send_message_button);

    final EditText senderMessageEditText = (EditText) 
            mSenderActivity.findViewById(R.id.message_input_edit_text);

    // Set up an ActivityMonitor
    ...

    // Send string input value
    ...

    // Validate that ReceiverActivity is started
    ...

    // Validate that ReceiverActivity has the correct data
    ...

    // Remove the ActivityMonitor
    ...
}
{% endhighlight %}

这个测试会等待与这个监视器匹配的`Activity`，否则会在等待超时后返回null。如果`ReceiverActivity`被启动了，那么你之前设置的`ActivityMonitor`就会收到一次通知。你可以使用断言方法验证`ReceiverActivity`是否确实被启动了，并且`ActivityMonitor`上的通知计数会按预期增加。


#设置一个ActivityMonitor
---
为了在测试中监视单个`Activity`，你可以注册一个`ActivityMonitor`。每当一个`Activity`和你设置的条件匹配时，`ActivityMonitor`就会收到通知。每匹配到一次，监视器的通知计数就会被更新。


通常来说要使用`ActivityMonitor`，你应该这样做：

1. 调用`getInstrumentation()`方法获取一个`Instrumentation`对象。
2. 调用`addMonitor()`方法为这个的`Instrumentation`对象添加一个`Instrumentation.ActivityMonitor`类的实例。匹配规则可以通过`IntentFilter`或者一个类名来设置。
3. 等待Activity启动。
4. 验证监视器的撞击次数。
5. 移除监视器。

例如：

{% highlight java %}
// Set up an ActivityMonitor
ActivityMonitor receiverActivityMonitor =
        getInstrumentation().addMonitor(ReceiverActivity.class.getName(),
        null, false);

// Validate that ReceiverActivity is started
TouchUtils.clickView(this, sendToReceiverButton);
ReceiverActivity receiverActivity = (ReceiverActivity) 
        receiverActivityMonitor.waitForActivityWithTimeout(TIMEOUT_IN_MS);
assertNotNull("ReceiverActivity is null", receiverActivity);
assertEquals("Monitor for ReceiverActivity has not been called",
        1, receiverActivityMonitor.getHits());
assertEquals("Activity is of wrong type",
        ReceiverActivity.class, receiverActivity.getClass());

// Remove the ActivityMonitor
getInstrumentation().removeMonitor(receiverActivityMonitor);
{% endhighlight %}



#使用Instrumentation发送一个键盘输入
---
如果你的`Activity`有一个`EditText`，你想要测试用户是否可以给`EditText`对象输入内容。

通常在`ActivityInstrumentationTestCase2`中给`EditText`对象发送字符串，你可以这样做：

1. 调用`runOnMainSync()`方法在一个循环中以同步的方式调用`requestFocus()`方法。这样，你的UI线程就会在获得焦点前一直被阻塞。
2. 调用`waitForIdleSync()`方法等待主线程空闲（也就是说，没有其它事件需要被处理）。
3. 调用`sendStringSync()`方法给`EditText`对象发送一个你要输入的字符串。

例如：

{% highlight java %}
// Send string input value
getInstrumentation().runOnMainSync(new Runnable() {
    @Override
    public void run() {
        senderMessageEditText.requestFocus();
    }
});
getInstrumentation().waitForIdleSync();
getInstrumentation().sendStringSync("Hello Android!");
getInstrumentation().waitForIdleSync();
{% endhighlight %}



浏览本系列的其它文章：

1. [建立测试环境](http://2dxgujun.github.io/10-01-2014/Activity-Testing-Setting-Up-Your-Test-Environment.html)
2. [创建和运行一个测试用例](http://2dxgujun.github.io/10-01-2014/Activity-Testing-Creating-and-Running-a-Test-Case.html)
3. [测试UI组件](http://2dxgujun.github.io/10-01-2014/Activity-Testing-Testing-UI-Components.html)
4. [创建单元测试](http://2dxgujun.github.io/10-01-2014/Activity-Testing-Creating-Unit-Tests.html)

<br/>
参考：

1. [Creating Functional Tests](https://developer.android.com/training/activity-testing/activity-functional-testing.html)
2. [创建功能测试](http://hukai.me/android-training-course-in-chinese/testing/activity-testing/activity-function-testing.html)

本文出自[2dxgujun](http://github.com/2dxgujun)，转载时请注明出处及相应链接。