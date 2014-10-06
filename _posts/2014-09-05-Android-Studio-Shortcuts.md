---
title: Android Studio常用快捷键整理
layout: post
guid: 2014090502
date: 2014-09-05 16:22:00
description: 
tags:
  - IDE
---

这篇文章将不定期更新一些Android Studio的常用的快捷键；留给自己备忘，也给大家做个参考。


索引：

- [Editor Actions（快速编辑）](#Editor Actions)
- [Main menu](#Main menu)
    1. [Find（查找文本）](#Find)
    2. [View（浏览、查看）](#View)
    3. [Navigate（导航、跳转、查找）](#Navigate)
    4. [Code（快速编码）](#Code)
        - [Completion（自动补全）](#Completion)
    5. [Analyze（工程扫描、错误检查）](#Analyze)
    6. [Refactor（重构）](#Refactor)
    7. [Run](#Run)
    8. [VCS（版本控制）](#VCS)
    9. [Window](#Window)
        - [Editor Tabs（操作工具窗口）](#Editor Tabs)
- [Other](#Other)

<br/>

<a id="Editor Actions"></a>
#Editor Actions
- Delete Line（删除行）: `Ctrl + Y`
- Select Word at Caret（选中代码，可连按）: `Ctrl + W`

<a id="Main menu"></a>
<a id="Find"></a>
#Main menu - Find
- Find...（查找文本）: `Ctrl + F`
- Replace...（替换文本）: `Ctrl + R`
- Highlight Usages in File（高亮选中文本）: `Ctrl + Shift + F7`

<a id="View"></a>
#Main menu - View
- Recent Files（最近打开的文件）: `Ctrl + E`
- Recent Changes（对比最近修改的代码）: `Alt + Shift + C`
- Paramter Info（方法参数提示）: `Ctrl + P`
- Quick Documentation（显示注释文档）: `Ctrl + Q`
- Recent Changes（查看最近的修改）: `Alt + Shift + C`

<a id="Navigate"></a>
#Main menu - Navigate
- Class...（查找类）: `Ctrl + N`
- File...（查找文件）:`Ctrl + Shift + N`
- Symbol...（查找类中的方法或变量）: `Ctrl + Shift + Alt + N`
- Select In...（跳转到工具窗口）: `Alt + F1`
- Back/Forward（返回上次浏览的位置）: `Ctrl + Alt + Left/Right`
- Previous/Next Method（在方法间快速移动定位）: `Alt + Up/Down`
- Previous/Next Highlighted Error（高亮错误快速定位）: `(Shift + )F2`
- Declartion（跳转到声明）: `Ctrl + B`
- Test（创建测试用例）：`Ctrl + Shift + T`

<a id="Code"></a>
#Main menu - Code
- Reformat Code...（格式化代码）: `Ctrl + Alt + L`
- Optimize Imports...（优化导入的包和类）: `Ctrl + Alt + O`
- Generate...（生成代码，如getter，setter方法）: `Alt + Insert`
- Comment with Line Comment（行注释）: `Ctrl + /`
- Comment with Block Comment（块注释）: `Ctrl + Shift + /`
- Insert Live Template...（自动代码）: `Ctrl + J`

<a id="Completion"></a>
#Main menu - Code - Completion
- SmartType（自动补全代码）[Custom]: `Alt + Shift + /`
- Basic（代码提示）[Custom]: `Alt + /`

<a id="Folding"></a>
#Main menu - Code - Folding
- Expand（展开代码）: `Ctrl + =`
- Collapse（折叠代码）: `Ctrl + -`

<a id="Analyze"></a>
#Main menu - Analyze
- Inspect Code...（Lint静态检查）[Custom]: `Ctrl + Alt + Shift + I`

<a id="Refactor"></a>
#Main menu - Refactor
- Rename...（重命名）: `Shift + F6`

<a id="Run"></a>
#Main menu - Run
- Run（运行）: `Shift + F10`
- Run...（选择运行）: `Alt + Shift + F10`

<a id="VCS"></a>
#Main menu - VCS
- VCS Operations Popup...（VCS操作弹窗）: `Alt + 后引号`

<a id="Window"></a>
<a id="Editor Tabs"></a>
#Main menu - Window - Editor Tabs
- Select Previous/Next Tab（切换代码Tab）: `Alt + Left/Right`
- Close（关闭当前Tab）: `Ctrl + F4`

<a id="Other"></a>
#Other
- Show Intention Actions（导入包，自动修正）: `Alt + CR`
- Class Name Completion（类名或接口名提示）: `Ctrl + Alt + Space`
- Fix doc comment（添加JavaDoc注释）[Custom]: `Alt + Shift + J`



<br/>
参考：

本文出自[2dxgujun](http://github.com/2dxgujun)，转载时请注明出处及相应链接。