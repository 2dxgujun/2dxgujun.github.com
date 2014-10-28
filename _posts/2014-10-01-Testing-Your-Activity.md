---
layout: post
title: Activity测试系列教程
category: Android Training
date: 2014-10-01
---

> 目录：
> 
> 1. [建立测试环境](#anchor_1)——教你在Android Studio下建立测试环境
> 2. [创建和运行一个测试用例](#anchor_2)——教你创建一个Activity测试用例，并用Instrumentation测试器来执行
> 3. [测试UI组件](#anchor_3)——教你测试Activity中特定UI组件
> 4. [创建单元测试](#anchor_4)——教你在封闭状态下对Activity进行测试
> 5. [创建功能测试](#anchor_5)——教你创建一个功能性测试，并用来测试多个Activity间的交互

今天是2014国庆节，天气很好，屌丝没有回家，一个人呆在宿舍里面。昨晚和朋友出去喝酒，喝了四瓶，第一次喝到吐了……今天一天都有点萎＞_＜真坑爹啊，头痛写不了代码就写写博客吧……不知道你们的国庆节是怎么过的。

这次的Android测试系列教程结合实例完整地介绍了Android测试中的一些知识点，大部分内容是翻译自[Android官网文档](https://developer.android.com/training/activity-testing/index.html)，水平有限，如有问题欢迎与我交流讨论。

#前言
---
编写和运行测试应该作为你Android应用开发周期的一部分，编写好的测试可以帮助你在开发过程中尽早发现Bug，并让提高你对自己的代码的信心。

测试用例定义了一系列对象和方法可以供你独立运行多个测试，测试用例可以编写成一套测试组，由测试框架组织成一个可以重复运行的测试者。

这节内容将会教你如何使用基于最流行的JUnit框架扩展的Android测试框架；你可以编写测试用例来测试你应用程序的特定行为，并在不同的Android设备上验证一致性。

<!-- more -->

<a id="anchor_1"></a>
#建立测试环境
---
在开始编写、运行你的测试之前，你应该先建立你的测试环境；本节将会教你如何在Android Studio上进行测试。

**注意：**
如果想要了解如何用Eclipse建立测试环境，请参考[Android官方文档](https://developer.android.com/training/activity-testing/preparing-activity-testing.html#eclipse)。


Android Studio新建的工程从一开始就已经支持测试了（我的Android Studio版本：0.8.9）；新建的Android工程目录结构如下：
![project_structure](/media/files/2014/10/01/p1_project_structure.png)

注意到`src/`目录下面有两个子目录`androidTest/`和`main/`，其中`main/`目录就是工程代码和资源文件放置的地方；而`androidTest/`目录就是测试代码应该放置的地方；可以看到测试目录里面已经有一个`ApplicationTest`类了。

在运行测试之前还需需要进行一些简单的配置：

点击Android Studio工具栏上“运行”按钮旁的运行配置下拉框，打开如下菜单：
![edit_configuration](/media/files/2014/10/01/p1_edit_configuration.png)

点击`Edit Configurations...`打开配置对话框之后，点击左上角的+号，选择`Android Tests`；在右边出现的面板上稍作配置：
![test_app_configuration](/media/files/2014/10/01/p1_test_app_configuration.png)

OK，切换运行模式到`test-app`模式，接下来就可以运行测试了。

运行中会弹出下面的工具窗口：
![test_app_run1](/media/files/2014/10/01/p1_test_app_run1.png)

这个窗口显示测试运行的各种状况，最右侧的窗口显示的就是那个`ApplicationTest`测试用例运行情况，其中包括运行花费的时间和测试方法的运行状况；`P:2`表示有两个方法通过了测试，双击即可查看具体测试通过的方法：
![test_app_run2](/media/files/2014/10/01/p1_test_app_run2.png)

这两个测试方法分别位于`ApplicationTestCase`和`AndroidTestCase`类；它们是Android测试框架进行的初始化测试。

<a id="anchor_2"></a>
#创建和运行一个测试用例
---
为了验证`Activity`中的布局设计和功能性行为是否符合预期，为应用中的每一个`Activity`创建测试是很有意义的。对于每一个测试，你需要在测试用例中创建一个个独立的单元，包括测试夹具、测试前提和测试方法。然后你就可以运行测试并得到测试报告；如果任何测试方法失败，这表明在你的代码中有潜在的缺陷。

**注意：**
在测试驱动开发（TDD）方法中，你应该编写足够的有效测试代码，不断更新你的测试用例，而不是在你写了大部分或全部前期代码后再开始测试。

##创建一个测试用例
`Activity`的测试都是以一种结构化的方式进行的，确保把`Activity`的测试代码放到一个单独的包内，和其他测试代码区分开。

如上一章介绍的，在Android Studio中，工程中`/src`目录下直接有一个`androidTest/`目录，这就是工程模版为你创建的用于放置测试用例的目录，里面的结构和应用代码路径`main/`中的一样；**你创建的测试用例应该放置在被测试应用代码相同的包内，不过一个是在`andoidTest/`目录，一个在`main/`目录（通常情况下，应用中的`Activity`都位于一个单独的包，所以无须再为你的`Activity`测试用例单独建一个包）；同时，测试用例名也应遵循你要测试的Java或Android类的相同名称，但后缀为“Test”**。

得力于我们强大的Android Studio:)我们可以方便地使用一个命令为我们的类建立测试用例，并且无须关心包和命名的问题；比如你想为`MyActivity`创建一个测试用例，打开`MyActivity`，然后右击编辑区，在弹出的上下文菜单中选择Go To - Test（Ctrl + Shift + T），你就可以打开一个新建测试用例的对话框：
![go_to_test](/media/files/2014/10/01/p2_go_to_test.png)

一般来说，你只需要修改那个Superclass（使用Android提供的测试框架），再选择是否生成一些测试方法。

OK之后，出现一个对话框叫你选择测试用例目录：
![choose_destination](/media/files/2014/10/01/p2_choose_destination.png)
选择`androidTest/`目录（最后一条）。

OK，这样就在对应的测试用例目录中新建了一个测试用例`MyActivityTest`。

###设置测试夹具
测试夹具由一系列必须在运行一个或多个测试方法前被初始化的对象组成。要建立测试夹具，你可以在你的测试用例中重写`setUp()`和`tearDown()`方法；测试者会在运行任何其它测试方法之前自动运行`setUp()`方法并且在所有测试方法运行结束后运行`tearDown()`方法，你可以用这些方法来分离测试代码的初始化和清理工作。

下面通过一个实例介绍如何设置测试夹具：

打开之前建立好的测试用例，然后修改测试用例使它继承自`ActivityTestCase`的子类；例如：

{% highlight java %}
public class MyFirstTestActivityTest
        extends ActivityInstrumentationTestCase2<MyFirstTestActivity> {
{% endhighlight %}

下一步，给测试用例添加构造方法和`setUp()`方法，并声明被测试的`Activity`和其它一些测试变量；例如：

{% highlight java %}
public class MyFirstTestActivityTest
        extends ActivityInstrumentationTestCase2<MyFirstTestActivity> {

    private MyFirstTestActivity mFirstTestActivity;
    private TextView mFirstTestText;

    public MyFirstTestActivityTest() {
        super(MyFirstTestActivity.class);
    }

    @Override
    protected void setUp() throws Exception {
        super.setUp();
        mFirstTestActivity = getActivity();
        mFirstTestText =
                (TextView) mFirstTestActivity
                .findViewById(R.id.my_first_test_text_view);
    }
}
{% endhighlight %}

通常在`setUp()`方法中你需要做以下几个工作：

- 调用父类的`setUp()`方法（JUnit框架要求）
- 通过下面的方法初始化测试夹具
    - 定义用于存储夹具状态的实例变量
    - 为正在测试的`Activity`创建和存储引用实例
    - 获取`Activity`中需要测试的UI组件的引用

**你可以使用`getActivity()`方法得到当前正在测试`Activity`的引用**。

###添加测试前提
一次完善的检查，确认测试夹具是否设置正确是很有必要的，那样你想要测试的对象就会保证被正确地实例化和初始化。这样，你的测试就不会因为你的测试夹具初始化错误而失败；按照惯例，验证你的测试夹具状态的方法被命名为`testPreconditions()`。

例如，你可能需要添加一个像这样的`testPreconditions()`方法：

{% highlight java %}
public void testPreconditions() {
    assertNotNull("mFirstTestActivity is null", mFirstTestActivity);
    assertNotNull("mFirstTestText is null", mFirstTestText);
}
{% endhighlight %}

断言方法是从Junit框架中的`Assert`类来的。通常，你可以使用断言验证你想测试一个特定的条件是否是真。

- 如果条件为假，断言方法会抛出一个`AssertionFailedError`异常，这是典型的测试报告。你可以给你的断言方法添加一个字符串作为第一个参数从而在你的断言失败时给出一些详细信息；
- 如果条件为真，测试通过。

在这两种情况下，测试器将继续运行其它测试用例的测试方法。

###添加方法测试Activity
下一步，添加一个或多个测试方法来验证`Activity`的布局和功能性行为。

例如，如果你的`Activity`中有一个`TextView`，你可以添加一个像这样的测试方法来检查它是否有正确的标签文本：

{% highlight java %}
public void testMyFirstTestTextView_labelText() {
    final String expected =
            mFirstTestActivity.getString(R.string.my_first_test);
    final String actual = mFirstTestText.getText().toString();
    assertEquals(expected, actual);
}
{% endhighlight %}


这个`testMyFirstTestTextView_labelText()`方法只是简单的检查`TextView`的默认文本是否和`strings.xml`资源中定义的字符串一致。

**注意：**
在命名测试方法时，可以使用下划线把被测试的内容从具体测试用例中分离出来；这种风格可以让你很容易的看清楚哪部分正在被测试。

**当做这种形式的字符串比较时，从你的资源文件中读取字符串比在你的代码中硬编码字符串好；这可以防止当资源文件被修改时轻易的打断你的测试**。

为进行比较，预期的和实际的字符串都要做为`assertEquals()`方法的参数。如果值是不一样的，断言将抛出一个`AssertionFailedError`异常。

**如果你添加了一个`testPreconditions()`方法，请把你的测试方法放在`testPreconditions()`方法之后**。

完整的测试用例代码，请参考实例工程[AndroidTestingFun](/media/files/2014/10/01/AndroidTestingFun.zip)中的MyFirstTestActivityTest.java。


##构建和运行你的测试
注意到我写了很多个测试用例，但是现在我只想运行其中一个测试用例，该怎么办？
![multi_test_case](/media/files/2014/10/01/p2_multi_test_case.png)

So Easy...参照之前建立测试环境中的内容，打开运行配置，配置成仅对一个`Activity`进行测试：
![run_one_testcase](/media/files/2014/10/01/p2_run_one_testcase.png)

可以看到测试用例通过测试，没有任何错误：
![test_app_run3](/media/files/2014/10/01/p2_test_app_run.png)


<a id="anchor_3"></a>
#测试UI组件
---
通常情况下，你的`Activity`中会包含很多用户界面组件（如`Button`、`CheckBox`、`EditText`等等）允许你的用户和Android应用程序进行交互。本节将介绍如何测试一个简单的按钮交互界面。你可以使用相同的步骤来测试其它的，更复杂的UI组件。

**注意：**
这一节中介绍的UI测试叫做白盒测试，因为你拥有被测试应用程序的源码。**Android [Instrumentation](http://developer.android.com/tools/testing/testing_android.html#Instrumentation)测试框架适用于创建应用程序中UI部件的白盒测试**。

UI测试的另一种类型是黑盒测试，就是那种你无法得到应用程序源码的测试方法。这种类型的测试可以用来测试你的应用程序是如何和其它应用程序或与系统进行交互的。本节不包括黑盒测试，想要了解更多关于如何在你的Android应用程序中进行黑盒测试，请参看[UI Testing guide](http://developer.android.com/tools/testing/testing_ui.html)。

完整的测试用例代码，请参考实例工程[AndroidTestingFun](/media/files/2014/10/01/AndroidTestingFun.zip)中的ClickFunActivityTest.java。

##使用Instrumentation框架创建一个UI测试用例
当你在测试一个包含用户界面的`Activity`时，被测`Activity`运行在UI线程，但是测试程序运行在被测应用进程中的另外一个线程。这意味着，你的测试代码可以引用UI线程的对象，但是如果你尝试修改它们的属性或者给UI发送事件，你将会获得一个`WrongThreadException`异常。

为了安全的注入`Intent`对象到你的`Activity`或者在UI线程运行测试方法，你可以继承`ActivityInstrumentationTestCase2`这个类。想要了解跟多关于如何在UI线程运行测试的方法，请参看[Testing on the UI thread](http://developer.android.com/tools/testing/activity_testing.html#RunOnUIThread)。

###设置测试夹具
当你在设置UI测试的夹具时，你需要在`setUp()`方法中指定[touch mode](http://developer.android.com/guide/topics/ui/ui-events.html#TouchMode)。把touch mode设置成true可以防止UI组件抢夺你编程指定的点击方法的焦点事件（比如，一个按钮可以通过编程触发它的点击监听器），确保你在调用`getActivity()`方法之前调用了`setActivityInitialTouchMode()`方法。

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

##添加方法测试UI组件的行为
你的UI测试目标应该包括：

- 检验`Activity`启动后`Button`是否按照正确布局显示
- 检验`TextView`初始化时是否是隐藏的
- 检验`TextView`在`Button`点击时是否显示预期的字符串

接下来的部分会示范如何编写测试方法，完成上面的测试目标。

###验证Button的布局
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

`@MediumTest`注解用于把测试方法按规模大小归类，这和测试的绝对执行时间有关。

###验证TextView的布局参数
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


###验证Button的行为
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

##测试注解
下面的注解可以用于标识测试方法的规模：

[@SmallTest](https://developer.android.com/reference/android/test/suitebuilder/annotation/SmallTest.html)：小规模测试方法

[@MediumTest](https://developer.android.com/reference/android/test/suitebuilder/annotation/MediumTest.html)：中规模测试方法

[@LargeTest](https://developer.android.com/reference/android/test/suitebuilder/annotation/LargeTest.html)：大规模测试方法


通常情况下，只需要几毫秒的时间的测试方法应该被标记为`@SmallTest`；长时间运行的测试（100毫秒或更多）通常被标记为`@MediumTest`或`@LargeTest`，测试时间主要取决于该测试是否访问了网络或本地系统的资源。可以参看[Android Tools Protip](http://plus.sandbox.google.com/+AndroidDevelopers/posts/TPy1EeSaSg8)指导你更好的使用测试注解。

你可以使用其它测试注解来控制测试的组织和运行。要了解更多关于其他注解的信息，见[Annotation](https://developer.android.com/reference/java/lang/annotation/Annotation.html)类参考。


<a id="anchor_4"></a>
#创建单元测试
---
对`Activity`进行单元测试可以快速验证一个`Activity`的状态以及它和与其它独立组件间的交互方式。一个单元测试通常只测试应用代码中尽可能小的代码块（可以是一个方法，类，或者组件），而且不应该依赖于系统或网络资源。比如说，你可以写一个单元测试去检查一个`Activity`是否有正确的布局或者触发它的`Intent`对象是否正确。

**单元测试一般不用于测试与系统有复杂交互事件的UI；相反，你应该使用像`ActivityInstrumentationTestCase2`这样的类，参考[测试UI组件](#anchor_3)。**

这节内容将会教你编写一个单元测试来验证一个`Intent`是否正确触发了另一个`Activity`。由于测试运行在一个独立的环境中，所以`Intent`没有被发送到`Android`系统中（不会真正启动`Activity`），但你还是可以验证`Intent`对象中携带的有效数据的正确性。

完整的测试用例代码，请参考实例工程[AndroidTestingFun](/media/files/2014/10/01/AndroidTestingFun.zip)中的LaunchActivityTest.java。

**注意：**
要针对系统或外部依赖进行测试，你可以使用来自Mocking框架的`Mock`类，并把它们注入到你的单元测试中。要了解更多关于Android提供的Mocking框架，请参看[Mock Object Classes](http://developer.android.com/tools/testing/testing_android.html#MockObjectClasses})。

##为Activity的单元测试创建一个测试用例
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

##测试被启动的Activity
你进行测试的目的包括：

- 验证当`Button`被按下时启动的`LaunchActivity`是否正确。
- 验证用于启动`Activity`的`Intent`是否携带有效数据。

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


<a id="anchor_5"></a>
#创建功能测试
---
功能测试包括验证独立的应用组件是否像用户期望的那样协同工作。比如，你可以创建一个功能测试验证在用户执行UI交互时`Activity`是否正确启动了目标`Activity`。

要为你的`Activity`创建功能测试，你的测试类应该继承自`ActivityInstrumentationTestCase2`。**与`ActivityUnitTestCase`不同的是，在`ActivityInstrumentationTestCase2`中可以与Android系统通信，并且还可以给UI组件发送键盘输入和点击事件**。

完整的测试用例代码，请参考实例工程[AndroidTestingFun](/media/files/2014/10/01/AndroidTestingFun.zip)中的SenderActivityTest.java。

##添加测试方法验证函数的行为
你进行的功能测试的目标包括：

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


##设置一个ActivityMonitor
为了在测试中监视单个`Activity`，你可以注册一个`ActivityMonitor`。每当一个`Activity`和你设置的条件匹配时，`ActivityMonitor`就会收到一次通知。每匹配到一次，监视器的通知计数就会被更新。


通常来说要使用`ActivityMonitor`，你应该这样做：

1. 调用`getInstrumentation()`方法获取一个`Instrumentation`对象。
2. 调用`addMonitor()`方法为这个的`Instrumentation`对象添加一个`Instrumentation.ActivityMonitor`类的实例；匹配规则可以通过`IntentFilter`或者一个类名来设置。
3. 等待`Activity`启动。
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

##使用Instrumentation发送一个键盘输入
如果你的`Activity`有一个`EditText`，你想要测试用户是否可以给`EditText`对象输入内容。

通常在`ActivityInstrumentationTestCase2`中给`EditText`对象发送字符串，你可以这样做：

1. 调用`runOnMainSync()`方法在主线程中以同步的方式调用`requestFocus()`方法。这样，你的UI线程就会在`EditText`获得焦点前一直被阻塞。
2. 调用`waitForIdleSync()`方法等待主线程空闲（也就是说，没有其它事件需要被处理）。
3. 调用`sendStringSync()`方法给`EditText`对象发送一个用于模拟键盘输入的字符串。

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


<br/>
参考：

1. [AndroidStudio单元测试——Instrumentation](http://blog.csdn.net/harvic880925/article/details/38060361)
2. [Unit Testing with Android Studio](http://rexstjohn.com/unit-testing-with-android-studio/)
3. [Creating and Running a Test Case](https://developer.android.com/training/activity-testing/activity-basic-testing.html)
4. [Testing UI Components](https://developer.android.com/training/activity-testing/activity-ui-testing.html)
5. [Creating Unit Tests](https://developer.android.com/training/activity-testing/activity-unit-testing.html)
6. [Creating Functional Tests](https://developer.android.com/training/activity-testing/activity-functional-testing.html)

本文出自[2dxgujun](http://2dxgujun.com/)，转载时请注明出处及相应链接。