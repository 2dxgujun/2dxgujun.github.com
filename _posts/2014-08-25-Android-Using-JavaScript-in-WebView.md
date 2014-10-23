---
layout: post
title: Android和JavaScript互相调用
category: Android Dev
date: 2014-08-25
---

最近在做一个App中需要通过`WebView`加载一个使用JavaScript的网页，并且要实现相互之间的调用。

首先如果如果你的`WebView`要加载使用JavaScript的网页，必须启用`WebView`对JavaScript的支持，一旦启用之后你就可以通过接口来实现本地代码和JavaScript代码的互相调用。

#启用JavaScript支持
---
`WebView`默认是禁用JavaScript的，你可以通过附加到`WebView`的设置类`WebSettings`开启它。
通过`getSettings()`函数获取`WebView`的`WebSettings`对象，然后你可以调用`setJavaScriptEnabled()`函数开启对JavaScript的支持。

例如：
{% highlight java %}
WebView myWebView = (WebView) findViewById(R.id.webview);
WebSettings webSettings = myWebView.getSettings();
webSettings.setJavaScriptEnabled(true);
{% endhighlight %}

<!-- more -->

#JavaScript调用Android代码
---
当针对Android应用设计网页时，你可以通过一个接口实现JavaScript调用Android本地代码完成一些平台特有的功能。例如，JavaScript可以调用Android中的代码来生成一个`Dialog`。

你可以通过`addJavaScriptInterface()`方法传入一个用来绑定JavaScript的对象和一个用来规范JavaScript访问的接口名（类似命名空间的一个字符串）。

例如，你可以在Android代码中定义这样一个类：
{% highlight java %}
public class WebAppInterface {
	Context mContext;

	/** Instantiate the interface and set the context */
	WebAppInterface(Context c) {
		mContext = c;
	}

	/** Show a toast from the web page */
	@JavascriptInterface
	public void showToast(String toast) {
		Toast.makeText(mContext, toast, Toast.LENGTH_SHORT).show();
	}
}
{% endhighlight %}

在这个例子中，`WebAppInterface`类允许网页通过`showToast()`方法发送一个`Toast`消息。
然后调用`WebView`的`addJavaScriptInterface()`方法把这个类和JavaScript绑定在一起，传入"Android"作为接口名。

例如：
{% highlight java %}
WebView webView = (WebView) findViewById(R.id.webview);
webView.addJavascriptInterface(new WebAppInterface(this), "Android");
{% endhighlight %}

上面的代码创建了一个名为"Android"的接口。这时，网页应用就可以通过这个"Android"接口访问`WebAppInterface`里的方法了。

例如，下面有段HTML和JavaScript的代码，当用户点击按钮，通过调用绑定的接口里的方法来创建一条Toast消息。
{% highlight html %}
<input type="button" value="Say hello" onClick="showAndroidToast('Hello Android!')" />

<script type="text/javascript">
    function showAndroidToast(toast) {
        Android.showToast(toast);
    }
</script>
{% endhighlight %}

这里不需要手动初始化"Android"接口，`WebView`会自动完成初始化工作。所以，点击按钮时，`showAndroidToast()`方法使用"Android"接口调用了`WebAppInterface.showToast()`方法。

__注意__：和JavaScript绑定的对象，运行在其他线程中而不是运行在构造它的线程（访问Android UI时需特别注意）。

__警告__：如果你的`targetSdkVersion`设置成17或者更高，必须为所有你希望对JavaScript有效的方法添加一个`@JavascriptInterface`注解（必须都是`public`方法）。如果你没有提供这个注解，那么在Android4.2或更高系统版本上，JavaScript就无法访问这些方法。

__PS__：在项目中我`targetSdkVersion`设成了19，`@JavascriptInterface`注解由Eclipse自动添加在调用`addJavaScriptInterface()`方法的成员函数上并没有为每一个接口中的方法都添加这个注解，测试时，我的Android4.0.4和朋友的一台Android4.4表现都正常；交付客户时，两台Android4.4出现接口内所有方法都失效的情况，我把`targetSdkVersion`降到了15并且去掉了那个注解，问题就解决了。

#Android调用JavaScript代码
---
不仅仅JavaScript可以调用Android本地的代码，`WebView`也提供了在Android本地调用JavaScript方法的功能。

例如，下面有段HTML和JavaScript的代码，调用他会在网页中插入一行文字。
{% highlight html %}
<script type="text/javascript">  
    function show(content){  
        document.getElementById("content").innerHTML=  
            "这是我的JavaScript调用，这是：" + content;  
    }
</script>  
{% endhighlight %}

在Android本地调用JavaScript的方法非常简单，只需要一行代码，如下：
{% highlight html %}
WebView.loadUrl("javascript:show('Android传过来的数据')");
{% endhighlight %}

<br/>
参考：

1. [Building Web Apps in WebView](https://developer.android.com/guide/webapps/webview.html#UsingJavaScript)
2. [wangkuifeng0118的Android与JavaScript相互调用](http://blog.csdn.net/wangkuifeng0118/article/details/7032247)

本文出自[2dxgujun](http://github.com/2dxgujun)，转载时请注明出处及相应链接。