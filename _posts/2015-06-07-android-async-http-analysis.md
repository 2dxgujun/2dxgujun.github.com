---
layout: post
title: android-async-http 源码分析
category:
- Android
- Analysis
date: 2015-06-07
---

[android-async-http](https://github.com/loopj/android-async-http)是一个为Android量身定制的异步HTTP请求框架，它基于Apache的[HttpClient](http://hc.apache.org/httpcomponents-client-ga/)开发。本文针对v1.4.6版本进行分析。

<!-- more -->

## 1. 功能介绍

android-async-http中所有的HTTP请求都在主线程外的其他线程中执行，所有的请求回调都在创建该回调对象的线程中进行，所以不论在什么地方发起请求，系统都能够正确地进行处理。android-async-http的特性总结如下：

- 异步网络请求，在匿名回调中处理响应
- 使用线程池来限制并发请求的数量
- GET/POST请求参数的builder构造
- Multipart文件上传
- 较小的体积，仅60KB
- 自动请求重发优化
- 支持gzip格式自动解压缩
- 可选的内置Json解析
- 可选的Cookie持久化存储

### 1.1 基本使用

android-async-http的使用比较简单，实例化一个`AsyncHttpClient`对象，然后调用其`get()/post()`等方法传入URL和请求参数还有一个回调处理对象，就可以发送HTTP请求。

```java
AsyncHttpClient client = new AsyncHttpClient();
client.get("http://www.google.com", new AsyncHttpResponseHandler() {

    @Override
    public void onStart() {
        // called before request is started
    }

    @Override
    public void onSuccess(int statusCode, Header[] headers, byte[] response) {
        // called when response HTTP status is "200 OK"
    }

    @Override
    public void onFailure(int statusCode, Header[] headers, byte[] errorResponse, Throwable e) {
        // called when response HTTP status is "4XX" (eg. 401, 403, 404)
    }

    @Override
    public void onRetry(int retryNo) {
        // called when request is retried
	}
});
```

## 2. 总体设计

![structure](http://ww1.sinaimg.cn/large/bce2dea9jw1eswil4ev25j20hv0exmyl.jpg)

### 2.1 概述

android-async-http的结构比较简单，整体可以分解成三层，最上一层负责由客户端代码调用发送请求，第二层负责请求的封装和执行，最下一层负责处理HTTP响应。

- **`AsyncHttpClient`**：调用此客户端类中的方法发送异步HTTP请求。
- **`RequestParams`**：HTTP请求参数集合，可包含文件和数据流。
- **`AsyncHttpRequest`**：代表异步HTTP请求，作为线程执行对象。
- **`AsyncHttpResponseHandler`**：提供基本的HTTP响应处理。
- **`RequestHandle`**：HTTP请求的句柄，用于取消请求。

### 2.2 工作流程

客户端调用`AsyncHttpClient`中的方法，传入URL和请求参数`RequestParams`发送请求，请求被封装成一个`AsyncHttpRequest`对象，这个对象代表一个异步请求，把请求加入到队列中等待执行，执行请求的过程中通过调用客户端传入的回调接口，发送消息通知客户端请求执行的情况。

## 3. 详细设计

android-async-http框架建立在Apache的HttpClient库之上，对HttpClient的类库进行了包装和升级，诸如HTTP请求失败重试机制，multipart/form-data和Json请求体的支持，HTTP响应的处理与包装。

此外还针对Android平台的一些特性进行优化，包括利用线程池技术执行HTTP请求以避免阻塞UI线程，在请求响应时利用Android的消息机制在UI线程执行处理代码，还利用了`SharedPreferences`持久化存储Cookie。

### 3.1 发送HTTP请求

`AsyncHttpClient`是框架中的核心类，所有的HTTP请求都通过这个类发送。它有一系列`get()`, `post()`, `put()`, `delete()`, `head()`方法，分别代表HTTP协议的GET，POST，PUT，DELETE，HEAD请求。这些方法最终都会调用`sendRequest()`方法，这个方法会把一个HTTP请求包装成一个线程对象放入请求队列等待执行。

```java
AsyncHttpRequest request = newAsyncHttpRequest(client, httpContext, uriRequest, contentType, responseHandler, context);
threadPool.submit(request);
```

`newAsyncHttpRequest()`方法创建一个`AsyncHttpRequest`对象，`AsyncHttpRequest`类实现了`Runnable`接口，它代表一个异步请求，在`run()`方法中实现了具体的HTTP访问。

发送请求的流程图如下：
![send request](http://ww2.sinaimg.cn/large/bce2dea9jw1eswiljwcgij207l0f374t.jpg)

### 3.2 取消HTTP请求

在`sendRequest()`方法中，当把`AsyncHttpRequest`对象提交之后，会创建一个HTTP请求的句柄`RequestHandle`，之后作为返回值返回。

通过这个句柄可以操纵这个HTTP请求，句柄类中提供了`cancel()`方法可以取消这个请求，这个方法委托调用`AsyncHttpRequest`的`cancel()`方法，在HTTP请求的执行过程中，设置了许多检查点，如果在执行过程中检测到取消标识，则发送通知给回调对象，并且终止HTTP请求的执行。

除了可以取消某个具体的请求，android-async-http还支持把一个HTTP请求对象和Android中的`Context`绑定，通过绑定`Context`，可以对一个`Context`相关的所有请求执行一些操作，例如，可以在`Activity`的`onDestroy()`方法中可以取消与在`Activity`中创建的所有请求。

```java
private final Map<Context, List<RequestHandle>> requestMap;
```

在`AsyncHttpClient`类中声明了一个成员`requestMap`，它关联了`Context`和HTTP请求的句柄，在构造方法中对其进行初始化。

每次调用`sendRequest()`方法发送HTTP请求，如果传入的`Context`参数不为null，都会把这个请求句柄添加到`Context`相关的句柄链上。

在`AsyncHttpClient`中有两个方法：`cancelAllRequest(boolean)`、`cancelRequests(Context, booolean)`，前一个方法取消系统中所有的请求，后一个方法取消一个`Context`下的所有请求。

### 3.3 HTTP请求参数

`RequestParams`类封装了HTTP请求参数的集合，在这个类中有大量`put()`方法，用于添加各种请求参数和请求实体。

发送GET或DELETE请求时，调用`getParamString()`方法把请求参数拼装成URL中的查询字符串；而发送POST或PUT请求时，调用`getEntity()`方法获取请求实体，`RequestParams`支持三种类型的请求实体：

1. Json
2. multipart/form-data
3. url-encoded pairs

#### 3.3.1 multipart/form-data请求体

multipart/form-data请求的基础方法是POST，它的**请求头**和**请求体**与普通的POST请求不同。普通的POST请求只能发送简单的name-value对，而使用multipart/form-data可以对请求体进行包装可以发送多个不同种类的请求体内容。

android-async-http对多文件上传的支持，就是使用multipart/form-data格式对文件数据进行包装。

`SimpleMultipartEntity`类继承自`HttpEntity`类，这个类实现了对multipart/form-data格式请求体的封装。

关于multipart/form-data请求的协议细节不多作介绍。

#### 3.3.2 Json请求体

android-async-http还支持把请求参数包装成Json格式的请求体，`JsonStreamerEntity`类继承自`HttpEntity`类，这个类实现了把请求参数包装成Json格式的数据，可以用来上传文件和二进制数据。

### 3.4 执行HTTP请求

`AsyncHttpRequest`类代表一个异步HTTP请求，它封装了执行HTTP请求的具体细节，并实现了`Runnable`接口，作为线程执行对象，以异步的方式进行HTTP访问。

执行请求的流程图如下：
![request running](http://ww4.sinaimg.cn/large/bce2dea9jw1eswilr9zrwj20es0i1aar.jpg)

`makeRequest()`方法中完成HTTP访问，`makeRequestWithRetries()`方法调用`makeRequest()`方法，封装了对异常的处理逻辑，并实现了请求的重试机制。

```java
HttpRequestRetryHandler retryHandler = client.getHttpRequestRetryHandler();
```

`HttpRequestRetryHandler`是Apache HttpClient提供的HTTP请求重发的处理接口，调用`retryRequest()`方法申请一次HTTP请求重发，如果`retryRequest()`返回true即表示申请通过可以发送，false反之。

android-async-http实现了自己的重发处理器`RetryHandler`，在`AsyncHttpClient`的构造方法做了初始化，默认最多重发5次，每次间隔1.5s。

```java
httpClient.setHttpRequestRetryHandler(new RetryHandler(DEFAULT_MAX_RETRIES, DEFAULT_RETRY_SLEEP_TIME_MILLIS));
```

#### 3.4.1 重发处理机制

`RetryHandler`类中维护了一个黑白名单，黑白名单中存储了异常类型，每次请求发送失败都会抛出一个异常，根据这个黑白名单判断是否重发请求。

调用`retryRequest()`方法传入一个异常对象和一个整数，异常对象是在进行HTTP访问时抛出的，整数表示请求重发的次数。在`RetryHandler`类的实现中，`retryRequest()`方法根据重发的次数和抛出的异常还有其它一些属性，响应重发请求。

在`makeRequestWithRetries()`方法的实现中，利用一个while循环实现请求重发的机制。

请求重发的流程图如下：
![make request with retries](http://ww1.sinaimg.cn/large/bce2dea9jw1eswil585w6j20kt0l8wfy.jpg)

### 3.5 处理HTTP响应事件

`ResponseHandlerInterface`接口用于发送消息通知客户端代码处理请求执行过程发送的事件（请求的过程中会发送很多事件，包括开始事件、结束事件、成功事件、失败事件、重发事件、取消事件、更新进度事件），它是一个处理HTTP响应的回调接口。`AsyncHttpResponseHandler`类是这个接口的直接实现，这个类实现了最基本的事件处理逻辑，还有很多其它具体的处理类都是基于这个类的包装了几个接口，以满足一些特殊的需求。

HTTP请求事件的处理有两种方式：同步和异步，同步方式会在执行请求的线程中直接调用回调方法，而异步方式则会在创建这个处理对象的线程中调用回调方法。

`ResponseHandlerInterface`接口中定义了一系列`sendXXX()`方法，这些方法在请求执行过程中被调用，用于发送事件，在`AsyncHttpResponseHandler`的实现中，这些`sendXXX()`方法调用了`sendMessage()`方法，`sendMessage()`方法中封装了选择同步或异步处理的逻辑。

```java
protected void sendMessage(Message msg) {
    if (getUseSynchronousMode() || handler == null) {
        handleMessage(msg);
    } else if (!Thread.currentThread().isInterrupted()) { // do not send messages if request has been cancelled
        AssertUtils.asserts(handler != null, "handler should not be null!");
        handler.sendMessage(msg);
    }
}
```

`handleMessage()`方法模仿了`Handler`中的处理方法，在这个方法里面实现了具体的事件分发，根据事件类型调用相应的回调接口。

同步方式直接调用`handleMessage()`方法在当前线程中执行处理；异步方式使用了`Handler`，在构造方法中使用一个构造线程的`Looper`对象初始化了这个`Handler`，事件的处理委托给`Handler`进行。

处理请求事件的流程图如下：
![response handle](http://ww2.sinaimg.cn/large/bce2dea9jw1eswil616t6j20gt0g30tk.jpg)

框架中其它处理类都继承自`AsyncHttpResponseHandler`，例如：`TextHttpResponseHandler`、`FileAsyncHttpResponseHandler`，前者把二进制数据流编码成字符串作为结果返回，后者把HTTP请求返回的结果写入到文件，把文件对象作为结果返回。

### 3.6 持久化Cookie存储

`PersistentCookieStore`类实现了Apache HttpClient的`CookieStore`接口，它通过`SharedPreferences`实现了`Cookie`在Android设备上的持久化存储。

`Cookie`持久化的技术上，利用了Java的序列化机制，提供了一个`Cookie`包装类：`SerializableCookie`，这个包装类实现了`Serializable`接口。`PersistentCookieStore`类提供了对二进制字节数组和十六进制字符串的编码和解码方法。

添加时利用序列化机制把`Cookie`转换成字节数组然后进行十六进制编码，再存储到`SharedPreferences`中；读取时先进行逆向解码，再利用对象输入流`ObjectInputStream`获取`Cookie`对象。

<br/>
本文出自[2dxgujun](/)，转载时请注明出处及相应链接。