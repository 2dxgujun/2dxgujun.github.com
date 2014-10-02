---
title: Activity Testing——建立测试环境
layout: post
guid: 2014100101
date: 2014-10-01 15:10:00
description: 学习如何使用Android Studio进行测试。
tags:
  - Android
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


浏览本系列的其它文章：

1. [创建和运行一个测试用例](http://2dxgujun.github.io/10-01-2014/Activity-Testing-Creating-and-Running-a-Test-Case.html)
2. [测试UI组件](http://2dxgujun.github.io/10-01-2014/Activity-Testing-Testing-UI-Components.html)
3. [创建单元测试](http://2dxgujun.github.io/10-01-2014/Activity-Testing-Creating-Unit-Tests.html)
4. [创建功能测试](http://2dxgujun.github.io/10-01-2014/Activity-Testing-Creating-Functional-Tests.html)

<br/>
参考：

1. [AndroidStudio单元测试——Instrumentation](http://blog.csdn.net/harvic880925/article/details/38060361)
2. [Unit Testing with Android Studio](http://rexstjohn.com/unit-testing-with-android-studio/)

本文出自[2dxgujun](http://github.com/2dxgujun)，转载时请注明出处及相应链接。