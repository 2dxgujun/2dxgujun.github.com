---
title: 国庆福利——Activity Testing系列
layout: post
guid: 2014100106
date: 2014-10-01 17:02:00
description: 
tags:
  - Android
---

今天是2014国庆节，天气很好，屌丝没有回家，一个人呆在宿舍里面。昨晚和朋友出去喝酒，喝了四瓶，第一次喝到吐了……今天一天都有点萎＞_＜真坑爹啊，头痛写不了代码就写写博客吧……不知道你们的国庆节是怎么过的。

这次的Android Testing系列结合实例完整地介绍了Android测试中的一些知识点，大部分内容是翻译自[Android官网文档](https://developer.android.com/training/activity-testing/index.html)，水平有限，如有问题欢迎与我交流讨论（我的博客是有评论系统的，不过在主页上不支持评论，打开具体的文章就可以评论了）。

#前言
---
编写和运行测试应该作为你Android应用开发周期的一部分，编写好的测试可以帮助你在开发过程中尽早发现Bug，并让提高你对自己的代码的信心。

测试用例定义了一系列对象和方法可以供你独立运行多个测试，测试用例可以编写成一套测试组，由测试框架组织成一个可以重复运行的测试者。

这节内容将会教你如何在Android上基于最流行的JUnit框架来自定义测试框架；你可以编写测试用例来测试你应用程序的特定行为，并在不同的Android设备上验证一致性。

#索引
---
**[建立测试环境](http://2dxgujun.github.io/10-01-2014/Activity-Testing-Setting-Up-Your-Test-Environment.html)：**
如何在Android Studio中创建测试环境。

**[创建和运行一个测试用例](http://2dxgujun.github.io/10-01-2014/Activity-Testing-Creating-and-Running-a-Test-Case.html)：**
如何编写测试用例来验证`Activity`中的一些属性，并且使用Android框架提供的`Instrumentation`测试者来运行测试。

**[测试UI组件](http://2dxgujun.github.io/10-01-2014/Activity-Testing-Testing-UI-Components.html)：**
如何测试`Activity`中指定UI组件的行为。

**[创建单元测试](http://2dxgujun.github.io/10-01-2014/Activity-Testing-Creating-Unit-Tests.html)：**
如何用单元测试来独立地验证一个`Activity`的行为。

**[创建功能测试](http://2dxgujun.github.io/10-01-2014/Activity-Testing-Creating-Functional-Tests.html)：**
如何用功能测试来验证多个`Activity`之间的交互作用。


<br/>

本文出自[2dxgujun](http://github.com/2dxgujun)，转载时请注明出处及相应链接。