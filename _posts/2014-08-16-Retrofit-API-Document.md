---
layout: post
title: Retrofit中文文档
category: Android Dev
date: 2014-08-16
---

今天在看[Android学习之路](http://stormzhang.github.io/android/2014/07/07/learn-android-from-rookie/)的时候，看到[retrofit](http://github.com/square/retrofit)这个开源库。虽然我浏览过该项目的主页，大概了解这是一个网络请求库，但是因为我在Android上一直都是用的Apache的HttpClient和一个基于回调的异步Http请求库[android-async-http](http://github.com/loopj/android-async-http)，特别是后者，简单方便，我一度感觉在网络请求库上别无他求了。

但是今天要介绍的这款网络请求库，给我带来了一种不一样的体验。

<!-- more -->

首先上一段代码，大家一起感受下吧。下面代码实现的功能是获取Github上对于retrofit这个开源库的贡献者用户名和贡献次数。
{% highlight java %}
public class GitHubClient {
	private static final String API_URL = "https://api.github.com";

	static class Contributor {
		String login;
		int contributions;
	}

	interface GitHub {
		@GET("/repos/{owner}/{repo}/contributors")
		List<Contributor> contributors(@Path("owner") String owner,
				@Path("repo") String repo);
	}

	public static void main(String... args) {
		// Create a very simple REST adapter which points the GitHub API
		// endpoint.
		RestAdapter restAdapter = new RestAdapter.Builder()
				.setEndpoint(API_URL).build();

		// Create an instance of our GitHub API interface.
		GitHub github = restAdapter.create(GitHub.class);

		// Fetch and print a list of the contributors to this library.
		List<Contributor> contributors = github.contributors("square",
				"retrofit");
		for (Contributor contributor : contributors) {
			System.out.println(contributor.login + " ("
					+ contributor.contributions + ")");
		}
	}
}
{% endhighlight %}
个人认为有如下三大亮点是：  

- Java注解式的URL及参数
- 精简的代码
- 查询式的网络请求

下面就让我们深入学习retrofit的使用:)

#简介
---
Retrofit把REST API封装成一个Java接口
例如：
{% highlight java %}
interface GitHub {
	@GET("/repos/{owner}/{repo}/contributors")
	List<Contributor> contributors(@Path("owner") String owner,
			@Path("repo") String repo);
}
{% endhighlight %}
该接口定义了一个函数contributors，该函数会通过HTTP GET请求去访问服务器的`/repos/{owner}/{repo}/contributors`路径并把返回的结果封装为`List<Contributor>`Java对象返回。

其中URL中的`{owner}`和`{repo}`的值为`contributors`函数中的参数`owner`和`repo`的取值。

然后通过`RestAdapter`类来生成一个Github接口的实现。
{% highlight java %}
RestAdapter restAdapter = new RestAdapter.Builder().setEndpoint(API_URL).build();
GitHub github = restAdapter.create(GitHub.class);
{% endhighlight %}

获取接口的实现后就可以调用接口函数来和服务器交互了
{% highlight java %}
List<Contributor> contributors = github.contributors("square","retrofit");
{% endhighlight %}

从上面的示例可以看出，Retrofit使用注解来声明HTTP请求：

- 支持URL参数替换和查询函数
- 返回结果转换为Java对象（返回结果可以为JSON，protocol buffers）
- 支持Multipart请求和文件上传

# API Declaration
---
函数和函数参数上的注解声明了请求方式和URL

## 1. 请求方式
每个函数都必须带有HTTP注解来表明请求方式和请求的URL。类库中有5个HTTP请求方式的注解：`GET，POST，PUT，DELETE和HEA`；注解中的参数为请求的相对URL路径。
{% highlight java %}
@GET("/users/list")
{% endhighlight %}
在URL中也可以指定请求参数
{% highlight java %}
@GET("/users/list?sort=desc")
{% endhighlight %}

## 2. URL处理
请求的URL可以根据可替换区块和函数参数动态更新。一个可替换的区块为用 { 和 } 包围的字符串，而函数参数必须用`@Path`注解表明，并且注解的参数为同样的字符串。
{% highlight java %}
@GET("/group/{id}/users")
List<User> groupList(@Path("id") int groupId);
{% endhighlight %}

还支持查询参数
{% highlight java %}
@GET("/group/{id}/users")
List<User> groupList(@Path("id") int groupId, @Query("sort") String sort);
{% endhighlight %}

支持使用`Map`组合复杂的查询参数
{% highlight java %}
@GET("/group/{id}/users")
List<User> groupList(@Path("id") int groupId, @QueryMap Map<String, String> options);
{% endhighlight %}

## 3. 请求体
通过`@Body`注解可以声明一个对象作为请求体发送到服务器
{% highlight java %}
@POST("/user/new")
void createUser(@Body User user, Callback<User> cb);
{% endhighlight %}
这个对象将被`RestAdapter`使用对应的转换器转换成字符串或者字节流提交到服务器。

## 4. 表单和Multipart
函数也可以使用注解定义为发送表单数据和multipart数据。
使用`@FormUrlEncoded`注解来发送表单数据；使用`@Field`注解来指定每个表单项的Key，函数参数的值来指定Value。
{% highlight java %}
@FormUrlEncoded
@POST("/user/edit")
User updateUser(@Field("first_name")  String first, @Field("last_name") String last);
{% endhighlight %}

使用`@Multipart`注解来发送multipart数据，使用`@Part`注解定义要发送的每个文件。
{% highlight java %}
@Multipart
@PUT("/user/photo")
User updateUser(@Part("photo") TypedFile photo, @Part("description") TypedString description);
{% endhighlight %}
Multipart中的Part使用RestAdapter的转换器来转换，也可以实现TypedOutput来自己处理序列化。

## 5. 请求头
可以使用`@Headers`注解来设置静态HTTP请求头。
{% highlight java %}
@Headers("Cache-Control: max-age=640000")
@GET("/widget/list")
List<Widget> widgetList();
{% endhighlight %}
{% highlight java %}
@Headers({
	"Accept: application/vnd.github.v3.full+json",
	"User-Agent: Retrofit-Sample-App"
})
@GET("/users/{username}")
User getUser(@Path("username") String username);
{% endhighlight %}
__注意__：请求头不会互相覆盖，所有同名请求头都会被包含在请求中。

可以使用`@Header`注解动态的更新请求头，函数中需提供一个和注解参数同名的函数参数，如果函数参数为null，就不会发送该请求头。
{% highlight java %}
@GET("/user")
void getUser(@Header("Authorization") String authorization, Callback<User> cb);
{% endhighlight %}

可以使用`RequestInterceptor`为每个请求添加请求头，下面的代码创建了一个`RequestInterceptor`为每个请求都添加`User-Agent`请求头。
{% highlight java %}
RequestInterceptor requestInterceptor = new RequestInterceptor() {
	public void intercept(RequestFacade request) {
		request.addHeader("User-Agent", "Retrofit-Sample-App");
	}
};
RestAdapter restAdapter = new RestAdapter.Builder()
		.setEndpoint("https://api.github.com")
		.setRequestInterceptor(requestInterceptor)
		.build();
{% endhighlight %}

## 6. 异步 VS 同步 VS 观察模式
每个函数都可以定位为同步或异步执行。

具有返回值的函数为同步执行
{% highlight java %}
@GET("/user/{id}/photo")
Photo getUserPhoto(@Path("id") int id);
{% endhighlight %}

而异步执行的函数没有返回值，并且要求最后一个参数为`Callback`对象
{% highlight java %}
@GET("/user/{id}/photo")
void getUserPhoto(@Path("id") int id, Callback<Photo> cb);
{% endhighlight %}

在Android上，`Callback`对象会在主（UI）线程中调用；而在普通Java应用中，`Callback`在请求执行的线程中调用。

Retrofit集成了[RxJava](https://github.com/Netflix/RxJava/wiki)，支持请求方法返回`rx.Observable`类型的对象。
{% highlight java %}
@GET("/user/{id}/photo")
Observable<Photo> getUserPhoto(@Path("id") int id);
{% endhighlight %}
`Observable`请求是异步执行的，并且在请求执行的线程中观察请求执行状态；如果需要在其他线程（e.g. Android主线程）中观察执行状态，调用请求返回的`Observable`的`observeOn(Scheduler)`方法。

PS. 这个RxJava不太了解，看项目主页说是一个JVM的响应式扩展框架，结合了可观察集合和LINQ式查询以达到异步和基于事件的的编程效果。不过我看上面使用`Callback`实现的异步请求应该能够满足大部分需求了，有机会可以尝试一下RxJava:)

## 7. 响应对象
使用`RestAdapter`的转换器把HTTP请求结果（默认为JSON）转换为Java对象，Java对象通过返回值或者`Callback`接口和`Obserable`接口定义。
{% highlight java %}
@GET("/users/list")
List<User> userList();

@GET("/users/list")
void userList(Callback<List<User>> cb);

@GET("/users/list")
Obserable<List<User>> userList();
{% endhighlight %}

如果要获取HTTP返回的原始数据，使用`Response`对象
{% highlight java %}
@GET("/users/list")
Response userList();

@GET("/users/list")
void userList(Callback<Response> cb);

@GET("/users/list")
Observable<Response> userList();
{% endhighlight %}

# 配置RestAdapter
---
`RestAdapter`把你定义的API接口转换成可调用的对象。默认情况，Retrofit会做好完整的配置，但是也可以自定义部分模块。

## 1. JSON解析
Retrofit默认使用[Gson](https://code.google.com/p/google-gson/)解析JSON数据。如果你想要实现特定的解析行为（e.g. 命名策略，日期格式，自定义类型），需要在构建`RestAdapter`时提供一个包含这些解析行为的自定义`Gson`对象，关于如何自定义`Gson`参考[Gson documentation](https://sites.google.com/site/gson/gson-user-guide)。

## 2. 自定义Gson解析示例
下面的代码创建了一个新的`Gson`对象，它会把所有小写并且带下划线的字段转换成驼峰式大小写，反之亦然。同时，它也为`Date`类型注册了一个类型适配器`DateTypeAdapter`，当`Gson`解析时遇到`Date`字段就会调用它。

`gson`对象作为`GsonConverter`的参数传入，类似于一个包装类。
{% highlight java %}
Gson gson = new GsonBuilder()
	.setFieldNamingPolicy(
			FieldNamingPolicy.LOWER_CASE_WITH_UNDERSCORES)
	.registerTypeAdapter(Date.class, new DateTypeAdapter())
	.create();
RestAdapter restAdapter2 = new RestAdapter.Builder()
	.setEndpoint("https://api.github.com")
	.setConverter(new GsonConverter(gson))
	.build();
GithubService service = restAdapter.create(GithubService.class);
{% endhighlight %}

## 3. 解析多种数据格式
除了JSON，Retrofit还可以通过配置解析其他数据格式。Retrofit提供多种方式来解析XML和协议数据，参考[retrofit-converters](https://github.com/square/retrofit/tree/master/retrofit-converters)目录获取所有`Converters`的列表。

下面的代码教你如何使用`SimpleXMLConverter`来和使用XML的服务器进行通信。
{% highlight java %}
RestAdapter restAdapter = new RestAdapter.Builder()
	.setEndpoint("https://api.soundcloud.com")
	.setConverter(new SimpleXMLConverter())
	.build();

SoundCloudService service = restAdapter.create(SoundCloudService.class);
{% endhighlight %}

## 4. 自定义错误处理
如果你需要自定义请求的错误处理，你需要提供你自己的`ErrorHandler`。
下面的代码教你如何在HTTP返回一个401状态响应码时抛出一个自定义的异常。
{% highlight java %}
class MyErrorHandler implements ErrorHandler {
	@Override
	public Throwable handleError(RetrofitError cause) {
		Response r = cause.getResponse();
		if (r != null && r.getStatus() == 401) {
			return new UnauthorizedException(cause);
		}
		return cause;
	}
}

RestAdapter restAdapter = new RestAdapter.Builder()
	.setEndpoint("https://api.github.com")
	.setErrorHandler(new MyErrorHandler())
	.build();
{% endhighlight %}
__注意__：如果指定了返回的异常，就必须在接口方法中声明异常。

## 5. 打印日志
如果你想要跟踪请求和响应的状态，你只需要给`RestAdapter`设置一个`LogLevel`属性既可，有4个属性值可选：`BASIC，FULL，HEADERS和NONE`。

下面的代码教你如何设置一个`FULL`级别的日志打印，打印内容包括请求和响应时的headers，body，metadata。
{% highlight java %}
RestAdapter restAdapter = new RestAdapter.Builder()
	.setLogLevel(RestAdapter.LogLevel.FULL)
	.setEndpoint("https://api.github.com")
	.build();
{% endhighlight %}
日志打印可以在`RestAdapter`的生命周期的任何时刻调用`setLogLevel()`方法添加或者改变级别。

参考：

1. [Retrofit – Java(Android) 的REST 接口封装类库](http://blog.chengyunfeng.com/?p=491)
2. [Retrofit Github项目主页](http://square.github.io/retrofit/)

本文出自[2dxgujun](http://github.com/2dxgujun)，转载时请注明出处及相应链接。