---
layout: post
title: 当Android Studio遇到Maven打包的apklib
category: Android Dev
date: 2014-11-03
---

众位看官容我娓娓道来……

我今天把[Github Android](https://github.com/github/android)给扒了下来，准备研究研究的，我就按照套路把整个Project给改成了Android Studio的Structure，导入没问题，接下来，就是添加依赖了，Github Android工程的依赖是用Maven管理，依赖关系都在那个pom.xml里。

{% highlight xml %}
<dependencies>
    <dependency>
        <groupId>com.actionbarsherlock</groupId>
        <artifactId>library</artifactId>
        <version>4.1.0</version>
        <type>apklib</type>
    </dependency>
    <dependency>
        <groupId>com.github.rtyley</groupId>
        <artifactId>roboguice-sherlock</artifactId>
        <version>1.4</version>
    </dependency>
    <dependency>
        <groupId>org.roboguice</groupId>
        <artifactId>roboguice</artifactId>
        <version>2.0</version>
    </dependency>
<dependencies>
{% endhighlight %}

我对Maven不太熟悉，但是依赖关系还是看得懂的，按照套路，给工程添加依赖，比如这个ActionBarSherlock开源库我就这么写：

<!-- more -->

{% highlight xml %}
compile 'com.actionbarsherlock:library:4.1.0'
{% endhighlight %}

结果写完，Sync一下，各种错啊有没有，看那个错误就是没有找到这个库中包含的类和资源；这我就不能理解了，依赖命名已经添加了，Gradle也把依赖的包下载下来了，此处省略1W字……

仔细查看pom.xml文件，发现那些提示没有找到类或资源的Lib的`<dependency>`标签中都有一个`<type>apklib</type>`，这是什么玩意？

直接看下面一张图，Android Studio提供的依赖库的列表：
![external libraries](/media/files/2014/11/3/external libraries.png)
注意到我展开了两个Lib，一个是`http-request`，另一个是`wishlist`；前一个工程能正常引用到，后一个引用不到，而它们的依赖中唯一的区别就是`wishlist`的依赖中有一个`<type>apklib</type>`。

找到问题所在了，查了些资料，这个apklib是类似于aar文件的东西，因为这方面资料实在是少，我把StackOverflow上的问题翻译一下，简单了解一下：

**apklib vs aar files**<br/>
apklib由Maven打包生成，.aar文件由Gradle打包生成，两者都可以包含代码和资源文件；aar文件和apklib的一个主要区别是：aar中包含一个编译出来的classes.jar（aar可以封闭源码），然而，apklib中不能包含编译好的类和jar文件（apklib在使用时还需要进行编译）。

**Can Maven use aar? Can Gradle use apklib?**<br/>
Maven的Android插件从Version 3.7.0开始支持aar文件，Gradle暂不支持使用apklib的依赖。

问题已经很清楚了，Gradle不支持以apklib的方式添加依赖，但你总不能说不能用就不用了吧；我这里介绍一种另辟蹊径的方法：

#使用本地Library Module作为工程依赖
---
丫的，叫你给我上传apklib，我直接把你整个库的源码给扒下来，添加到我的Android Studio工程里面，具体怎么做呢，比如那个ActionBarSherlock库，大概有以下几个步骤：

1. Download or Clone 这个库的源码
2. 分离这个库的核心代码，重新组织代码结构，添加build.gradle文件
3. 复制到需要添加这个依赖库的工程目录下，比如{project}/extras/ActionBarSherlock
4. 修改工程的settings.gradle文件，把你添加的那个依赖的库include进来，例如`include ':extras:ActionBarSherlock'`
5. 修改工程模块中的build.gradle文件，添加依赖关系，例如`compile project(':extras:ActionBarSherlock')`
6. Sync一下Project

OK了，有点麻烦，但是没办法；你也可以这个库给打包成jar或者aar再添加进依赖。

不知道有没有更加好的方法，如果你知道其它更加好的方法可以添加此类依赖，欢迎与我交流。

参考：

1. [Android dependencies : apklib vs aar files](http://stackoverflow.com/questions/22657466/android-dependencies-apklib-vs-aar-files)
2. [How can I add a “Library Project” in IntelliJ IDEA?](http://stackoverflow.com/questions/8884662/how-can-i-add-a-library-project-in-intellij-idea)
3. [Add support for apklib dependencies](https://github.com/rgladwell/m2e-android/issues/8)

本文出自[2dxgujun](http://2dxgujun.com/)，转载时请注明出处及相应链接。