---
layout: post
title: 我在Android开发中遇到的问题和解决方案
category: Android Dev
date: 2014-10-05
---

在开发中遇到问题的时候，我一直有一种我之前好像遇到过的感觉，但是怎么解决的我已经忘了……（PS.我在写这篇文章前刚遇到过这种情况）

然后我们只能再次百度或Google，当然还有Stack Overflow；这不是坑爹么……？然而如果每次解决问题后都发一篇文章总结一下这个问题，又觉得不太可取。

所以这篇文章记录了我在Android开发中遇到的各种各样的问题，以及通过网络搜索找到的能够帮助我解决这些问题的文章或贴子，我会不定期的更新这篇文章。

废话不多说，开始吧！

#ScrollView嵌套AbsListView的问题
---
通常，`ScrollView`嵌套`ListView`会存在两个问题：

1. 里面的`ListView`高度无法计算出来，通常只能显示`ListView`中的一行
2. `ListView`不能滚动

参考：
[四种方案解决ScrollView嵌套ListView问题](http://www.apkbus.com/android-161576-1-1.html)中的第四种方案

<!-- more -->

<br/>
#ListView中Item失去焦点，无法点击
---
在Item的Layout中如果放置了像`Button`，`EditText`之类的控件，这些子控件会把Item的焦点给抢掉。

参考：
[关于ListView的Item失去焦点不能点击](http://blog.csdn.net/beijingshi1/article/details/10431589)

<br/>
#Android软键盘弹出时主窗口整体上移的问题
---
这个问题，网上没找到写的像样的文章，就自己写了篇，针对自己遇到的问题，做了一些分析。

参考：
[关于Android软键盘弹出时主窗口整体上移的问题](/post/2014/10/05/Android-Problem-and-Solution-Collection.html)

<br/>
#Android Studio无法导入apklib依赖
---
关于Android Studio导入Maven apklib时遇到问题的分析，介绍了一种另辟蹊径的解决方法。

参考：
[当Android Studio遇到Maven打包的apklib](/post/2014/11/03/Maven-APKLIB-with-Android-Studio.html)


<br/>
本文出自[2dxgujun](http://github.com/2dxgujun)，转载时请注明出处及相应链接。