---
title: Activity Testing——创建和执行测试用例
layout: post
guid: 2014100102
date: 2014-10-01 15:11:00
description: 
tags:
  - Android
---

为了验证布局设计和功能行为是否符合预期，为应用中的每一个`Activity`创建测试是很有意义的。对于每一个测试，你需要在测试用例中创建一个个独立的单元，包括测试夹具、测试前提和测试`Activity`的方法。然后你就可以运行测试并得到测试报告；如果任何测试方法失败，这表明在你的代码中有潜在的缺陷。


**注意：**
在测试驱动开发（TDD）方法中，你应该编写足够的有效测试代码，不断更新你的测试用例，而不是在你写了大部分或全部前期代码后再开始测试。


#创建一个测试用例
---
`Activity`的测试都是以一种结构化的方式进行的，确保把`Activity`的测试代码放到一个单独的包内，和其他测试代码区分开。

如之前介绍的，在Android Studio中，工程中`/src目录下直接有一个`androidTest/`目录，这就是工程模版为你创建的用于放置测试用例的目录，里面的结构和应用代码路径`main/`中的一样；你创建的测试用例应该放置在被测试应用代码相同的包内，不过一个是在`andoidTest/`目录，一个在`main/`目录（通常情况下，`Activity`都位于一个单独的包，所以无须再为你的`Activity`测试用例单独建一个包）；同时，测试用例名也应遵循你要测试的Java或Android类的相同名称，但后缀为“Test”。

得力于我们强大的Android Studio:)我们可以方便地使用一个命令为我们的类建立测试用例，并且无须关心包和命名的问题；比如你想为`MyActivity`创建一个测试用例，打开`MyActivity`，然后右击编辑区，在弹出的上下文菜单中选择`Go To - Test（Ctrl + Shift + T）`，你就可以打开一个新建测试用例的对话框：
![go_to_test](/media/files/2014/10/01/p2_go_to_test.png)

一般来说，你只需要修改一个`Superclass`（使用Android提供的测试框架），再选择是否生成一些测试方法。

OK之后，出现一个对话框叫你选择测试用例目录：
![choose_destination](/media/files/2014/10/01/p2_choose_destination.png)
选择`androidTest/`目录（最后一条）。

OK，这样就在对应的测试用例目录中新建了一个测试用例`MyActivityTest`。


#设置测试夹具
---
测试夹具由一系列必须在运行一个或多个测试方法前被初始化的对象组成。要建立测试夹具，你可以在你的测试用例中重写`setUp()`和`tearDown()`方法；测试者会在运行任何其它测试方法之前自动运行`setUp()`方法并且在所有测试方法运行结束后运行`tearDown()`方法，你可以用这些方法来分离测试代码的初始化和清理工作。

下面通过一个实例介绍如何设置测试夹具：

1. 打开之前建立好的测试用例，然后修改测试用例使它继承自`ActivityTestCase`的子类；例如：

{% highlight java %}
public class MyFirstTestActivityTest
        extends ActivityInstrumentationTestCase2<MyFirstTestActivity> {
{% endhighlight %}

2. 下一步，给测试用例添加构造方法和`setUp()`方法，并声明被测试的`Activity`和其它一些测试变量；例如：

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

你可以使用`getActivity()`方法得到测试`Activity`的引用。


#添加测试前提（Add Test Preconditions）
---
作为一次完善的检查，确认测试夹具是否设置正确是很好的做法，那样你想要测试的对象就会保证被正确地实例化和初始化。这样，你的测试就不会因为你的测试夹具初始化错误而失败；按照惯例，验证你的测试夹具状态的方法被命名为`testPreconditions()`。

例如，你可能需要添加一个像这样的`testPreconditions()`方法：

{% highlight java %}
public void testPreconditions() {
    assertNotNull(“mFirstTestActivity is null”, mFirstTestActivity);
    assertNotNull(“mFirstTestText is null”, mFirstTestText);
}
{% endhighlight %}

断言方法是从Junit框架中的`Assert`类来的。通常，你可以使用断言验证你想测试一个特定的条件是否是真。

- 如果条件为假，断言方法会抛出一个`AssertionFailedError`异常，这是典型的测试者报告。你可以给你的断言方法添加一个字符串作为第一个参数从而在你的断言失败时给出一些详细信息。
- 如果条件为真，测试通过。

在这两种情况下，测试者将继续运行其它测试用例的测试方法。


#添加测试方法验证你的Activity（Add Test Methods to Verify Your Activity）
---
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

当做这种形式的字符串比较时，从你的资源文件中读取字符串比在你的代码中硬编码字符串好；这可以防止当资源文件被修改时轻易的打断你的测试。

为进行比较，预期的和实际的字符串都要做为`assertEquals()`方法的参数。如果值是不一样的，断言将抛出一个`AssertionFailedError`异常。

如果你添加了一个`testPreconditions()`方法，请把你的测试方法放在`testPreconditions()`方法之后。

完整的测试用例代码，请参考示例中的MyFirstTestActivityTest.java。


#构建和运行你的测试（Build and Run Your Test）
---
注意到我写了很多个测试用例，但是现在我只想运行其中一个测试用例，该怎么办？
![multi_test_case](/media/files/2014/10/01/p2_multi_test_case.png)

So Easy...参照之前建立测试环境中的内容，打开运行配置，配置成仅对一个Activity进行测试：
![run_one_testcase](/media/files/2014/10/01/p2_run_one_testcase.png)

可以看到测试用例通过测试，没有任何错误：
![test_app_run3](/media/files/2014/10/01/p2_test_app_run.png)


浏览本系列的其它文章：

1. [建立测试环境](http://2dxgujun.github.io/10-01-2014/Activity-Testing-Setting-Up-Your-Test-Environment.html)
2. [测试UI组件](http://2dxgujun.github.io/10-01-2014/Activity-Testing-Testing-UI-Components.html)
3. [创建单元测试](http://2dxgujun.github.io/10-01-2014/Activity-Testing-Creating-Unit-Tests.html)
4. [创建功能测试](http://2dxgujun.github.io/10-01-2014/Activity-Testing-Creating-Functional-Tests.html)

<br/>
参考：

1. [Creating and Running a Test Case](https://developer.android.com/training/activity-testing/activity-basic-testing.html)

本文出自[2dxgujun](http://github.com/2dxgujun)，转载时请注明出处及相应链接。