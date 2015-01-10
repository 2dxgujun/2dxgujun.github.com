---
layout: post
title: Android装载器（Loaders）使用示例
category: Android
date: 2014-11-14
---

关于装载器的简介，可以参考我的另外一篇文章：[Android装载器（Loaders）框架简介](/post/2014/11/14/Use-Loaders-in-Android.html)。

本文主要围绕Android官方的ApiDemo中的LoaderThrottle.java从下到上展开一个完整的示例；包括实现一个简单的SQLite数据库保存数据，以及提供一个基于SQLite数据库的`ContentProvider`的实现，节流装载器的基本使用，以及利用AsyncTask模拟数据源发生变化来测试装载器的行为。

<!-- more -->

# API概述

由于本示例中使用了一些装载器框架之外的类和框架，下面简单介绍一下几个关键的类：

### Uri

URI即通用资源标识符（Universal Resource Identifier），URI代表要操作的数据，Android上可用的每种资源（包括图像，视频，音频资源等）都可以用URI来表示。
就Android平台而言，URI主要分三个部分：**scheme、authority、path**；其中authority又分为host和port。

格式如下：
> scheme://host:port/path

举个实际的例子：
![uri](/media/2014/11/14/uri.png)

但是在Android中一般不直接使用URI字符串来标识Content provider；我们通常定义常量来标识，例如，Android系统提供的Contacts provider，我们就用`Contacts.People.CONTENT_URI`来标识Contacts provider中的的People这个表。

那么要标识某个具体的人怎么办？

这就用到了`ContentUris.withAppendedId()`和`Uri.withAppendedPath()`。例如我们要标识`content://contacts/people/20`，那么我们就可以用如下语句：

> Uri uri = ContentUris.withAppendedId(People.CONTENT_URI, 20); 或者<br/>
> Uri uri = Uri.withAppendedPath(People.CONTENT_URI, "20");

在本例中也将遵循这种URI的规范。

### UriMatcher

这是一个用来在Content provider中负责URI匹配的工具类。

{% highlight java %}
private static final int PEOPLE = 1;
private static final int PEOPLE_ID = 2;
private static final int PEOPLE_PHONES = 3;
private static final int PEOPLE_PHONES_ID = 4;

private static final UriMatcher sURIMatcher = new UriMatcher(UriMatcher.NO_MATCH);

sURIMatcher.addURI("contacts", "people", PEOPLE);
sURIMatcher.addURI("contacts", "people/#", PEOPLE_ID);
sURIMatcher.addURI("contacts", "people/#/phones", PEOPLE_PHONES);
sURIMatcher.addURI("contacts", "people/#/phones/#", PEOPLE_PHONES_ID);
{% endhighlight %}

`addURI()`方法有下面几个参数：

- authority：URI中的authority部分
- path：URI中的path部分，可以使用两个通配符，*代表任意字符串，#代表数字
- code：当匹配这个URI时，返回的数字，必须是正数

上面那段代码中初始化了一个`UriMatcher`，当你调用`sURIMatcher.match()`方法并传入下面的这些URI时，它会返回匹配的Code。

> content://contacts/people<br/>
> content://contacts/people/4<br/>
> content://contacts/people/4/phones<br/>
> content://contacts/people/4/phones/1<br/>

### MIME Type

MIME即多功能Internet邮件扩充服务（Multipurpose Internet Mail Extensions），它是一种多用途网际邮件扩充协议。
MIME类型就是设定某种扩展名的文件用一种应用程序来打开的方式类型，当该扩展名文件被访问的时候，浏览器（Android系统）会自动使用指定应用程序来打开。

Content providers既可以返回标准MIME媒体类型，也可以使用自定义的MIME类型字符串。

MIME格式：
> type/subtype

例如，在`text/html`这个MIME类型中type是`text`，subtype是`html`；如果一个Content provider对传入的URI响应这个MIME类型，这意味着使用这个URI来查询Content provider会返回包含html标签的文本数据。

自定义的MIME类型，通常有更多复杂的type和subtype，type的值通常是：
> vnd.android.cursor.**dir**：对应多行数据<br/>
> vnd.android.cursor.**item**：对应一行数据

subtype通常由Content provider指定，Android内建的Content providers通常使用一些简单subtype。例如，联系人应用标识一个联系人时，它会使用如下的MIME类型：
> vnd.android.cursor.item/phone_v2
注意到subtype是`phone_v2`，

其它的Content provider可能会使用它们的authority字符串或表名来指定subtype。例如，有一个火车时刻表的Content provider，authority是`com.example.trains`，它有三个表：Line1、Line2、Line3，它的Content URI类似如下：
> content://com.example.trains/Line1

对于表Line1，Content provider应该对应如下MIME：
> vnd.android.cursor.**dir**/vnd.example.line1

如果使用如下Content URI：
> content://com.example.trains/Line2/5

标识表Line2中的第5行，则应该对应如下MIME：
> vnd.android.cursor.**item**/vnd.example.line2

大多数Content provider都会使用一些常量来定义它们所使用的MIME类型；例如在Contacts provider中定义了`CONTENT_ITEM_TYPE`作为其联系人表中一行数据的MIME。

本例中的MIME类型也将遵照上面的规范。

# SQLite数据库的设计

首先定义数据库中唯一一张表的结构，本例中的`MainTable`继承自`BaseColums`，`BaseColums`中提供了两个默认的字段`_ID`和`_COUNT`作为表中的列名；在`MainTable`中还定义了其它一些成员。

{% highlight java %}
public static final class MainTable implements BaseColumns {
    ...

	public static final String TABLE_NAME = "main"; // 表名
	public static final String DEFAULT_SORT_ORDER = "data COLLATE LOCALIZED ASC"; // 数据排序命令
	public static final String COLUMN_NAME_DATA = "data"; // 列名

	...
}
{% endhighlight %}

然后，你还需要一个SQLite数据库的帮助类，负责数据库的创建，版本管理。

{% highlight java %}
static class DatabaseHelper extends SQLiteOpenHelper {
	private static final String DATABASE_NAME = "loader_throttle.db";
	private static final int DATABASE_VERSION = 2;

	DatabaseHelper(Context context) {
		// 调用父类构造器，使用默认游标工厂
		super(context, DATABASE_NAME, null, DATABASE_VERSION);
	}

	@Override
	public void onCreate(SQLiteDatabase db) {
	    // 建表
		db.execSQL("CREATE TABLE " + MainTable.TABLE_NAME + " ("
				+ MainTable._ID + " INTEGER PRIMARY KEY,"
				+ MainTable.COLUMN_NAME_DATA + " TEXT" + ");");
	}

	/**
	 * 在这个方法中处理数据库的升级工作，在本例中里升级操作将删除所有数据；
	 */
	@Override
	public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
		// 删除表和其中的所有数据
		db.execSQL("DROP TABLE IF EXISTS notes");

		// 重新建表
		onCreate(db);
	}
}
{% endhighlight %}

# 实现 Content provider

上面完成了SQLite数据库的设计，你还需要提供一种对数据库进行操作的方式，本例采用Content provider来实现对数据库的访问。

在那个`MainTable`中还定义了一些访问数据时所使用的URI信息：

{% highlight java %}
// 用于访问我们的示例provider的权限信息
public static final String AUTHORITY = "com.example.android.apis.app.LoaderThrottle";
...

public static final class MainTable implements BaseColumns {
    ...

	// 用于这个数据库表的 content:// 类型的URI
	// content://com.example.android.apis.app.LoaderThrottle/main
	public static final Uri CONTENT_URI = Uri.parse("content://" + AUTHORITY + "/main");

	// 用于某一行数据的URI，使用时必须在URI后面附加一个行号来检索一行
	// content://com.example.android.apis.app.LoaderThrottle/main/
	public static final Uri CONTENT_ID_URI_BASE = Uri.parse("content://" + AUTHORITY + "/main/");

	// CONTENT_URI的MIME类型
	public static final String CONTENT_TYPE = "vnd.android.cursor.dir/vnd.example.api-demos-throttle";

    // 其中一行数据的MIME类型
	public static final String CONTENT_ITEM_TYPE = "vnd.android.cursor.item/vnd.example.api-demos-throttle";
	
    ...
}
{% endhighlight %}

接下来需要实现自己的Content provider类，负责管理数据库的连接，对数据进行操作：

{% highlight java %}
public static class SimpleProvider extends ContentProvider {
	// A projection map used to select columns from the database
	private final HashMap<String, String> mNotesProjectionMap;
	// Uri matcher to decode incoming URIs.
	private final UriMatcher mUriMatcher;

	private static final int MAIN = 1; // Code，表示匹配main数据表的URI
	private static final int MAIN_ID = 2; // Code，表示匹配main数据表中一条记录的URI

	private DatabaseHelper mOpenHelper;

	public SimpleProvider() {
		// 创建和初始化URI的匹配器
		mUriMatcher = new UriMatcher(UriMatcher.NO_MATCH);
		mUriMatcher.addURI(AUTHORITY, MainTable.TABLE_NAME, MAIN);
		mUriMatcher.addURI(AUTHORITY, MainTable.TABLE_NAME + "/#", MAIN_ID);

		// 创建和初始化数据列映射表
		mNotesProjectionMap = new HashMap<String, String>();
		mNotesProjectionMap.put(MainTable._ID, MainTable._ID);
		mNotesProjectionMap.put(MainTable.COLUMN_NAME_DATA,
				MainTable.COLUMN_NAME_DATA);
	}

	@Override
	public boolean onCreate() {
	    // 在Content provider创建时，实例化SQLite帮助类的实例
		mOpenHelper = new DatabaseHelper(getContext());
		return true;
	}

	@Override
	public Cursor query(Uri uri, String[] projection, String selection,
			String[] selectionArgs, String sortOrder) {
		// 构造一次数据库查询
		SQLiteQueryBuilder qb = new SQLiteQueryBuilder();
		qb.setTables(MainTable.TABLE_NAME);

		switch (mUriMatcher.match(uri)) { // URI匹配
			case MAIN : // 查询整张表
				qb.setProjectionMap(mNotesProjectionMap);
				break;

			case MAIN_ID : // 查询一条记录
				qb.setProjectionMap(mNotesProjectionMap);
                // 构造WHERE子句
				qb.appendWhere(MainTable._ID + "=?");
				selectionArgs = DatabaseUtils.appendSelectionArgs(
						selectionArgs,
						new String[]{uri.getLastPathSegment()});
				break;

			default :
				throw new IllegalArgumentException("Unknown URI " + uri);
		}

		if (TextUtils.isEmpty(sortOrder)) {
			sortOrder = MainTable.DEFAULT_SORT_ORDER;
		}

		SQLiteDatabase db = mOpenHelper.getReadableDatabase();

		Cursor c = qb.query(db, projection, selection, selectionArgs,
				null /* no group */, null /* no filter */, sortOrder);
        // 往下看
		c.setNotificationUri(getContext().getContentResolver(), uri);
		return c;
	}

	@Override
	public String getType(Uri uri) { // 返回传入的URI的MIME
		switch (mUriMatcher.match(uri)) {
			case MAIN :
				return MainTable.CONTENT_TYPE;
			case MAIN_ID :
				return MainTable.CONTENT_ITEM_TYPE;
			default :
				throw new IllegalArgumentException("Unknown URI " + uri);
		}
	}

	@Override
	public Uri insert(Uri uri, ContentValues initialValues) {
		if (mUriMatcher.match(uri) != MAIN) { // 只能对main URI进行插入操作
			throw new IllegalArgumentException("Unknown URI " + uri);
		}

        // 构造插入的数据
		ContentValues values;
		if (initialValues != null) {
			values = new ContentValues(initialValues);
		} else {
			values = new ContentValues();
		}

		if (values.containsKey(MainTable.COLUMN_NAME_DATA) == false) {
			values.put(MainTable.COLUMN_NAME_DATA, "");
		}

		SQLiteDatabase db = mOpenHelper.getWritableDatabase();

		long rowId = db.insert(MainTable.TABLE_NAME, null, values);

		if (rowId > 0) { // 如果插入成功，就会返回行号
			Uri noteUri = ContentUris.withAppendedId(MainTable.CONTENT_ID_URI_BASE, rowId);
			// 往下看
			getContext().getContentResolver().notifyChange(noteUri, null);
			return noteUri;
		}

		throw new SQLException("Failed to insert row into " + uri);
	}

	@Override
	public int delete(Uri uri, String where, String[] whereArgs) {
		SQLiteDatabase db = mOpenHelper.getWritableDatabase();
		String finalWhere;
		int count;

		switch (mUriMatcher.match(uri)) {
			case MAIN : // 如果URI对应main table，则使用参数中传入的参数构造WHERE子句
				count = db.delete(MainTable.TABLE_NAME, where, whereArgs);
				break;

			case MAIN_ID : // 如果URI对应一条记录，就在传入的参数的基础上修改WHERE子句，删除对应ID的记录
				finalWhere = DatabaseUtils.concatenateWhere(MainTable._ID
						+ " = " + ContentUris.parseId(uri), where);
				count = db.delete(MainTable.TABLE_NAME, finalWhere, whereArgs);
				break;

			default :
				throw new IllegalArgumentException("Unknown URI " + uri);
		}
		// 往下看
		getContext().getContentResolver().notifyChange(uri, null);

		return count;
	}

	@Override
	public int update(Uri uri, ContentValues values, String where, String[] whereArgs) {
		SQLiteDatabase db = mOpenHelper.getWritableDatabase();
		int count;
		String finalWhere;

		switch (mUriMatcher.match(uri)) {
			case MAIN : // 如果URI对应main table，则使用参数中传入的参数构造WHERE子句
				count = db.update(MainTable.TABLE_NAME, values, where, whereArgs);
				break;

			case MAIN_ID : // 如果URI对应一条记录，就在传入的参数的基础上修改WHERE子句，更新对应ID的记录
				finalWhere = DatabaseUtils.concatenateWhere(MainTable._ID
						+ " = " + ContentUris.parseId(uri), where);
				count = db.update(MainTable.TABLE_NAME, values, finalWhere, whereArgs);
				break;

			default :
				throw new IllegalArgumentException("Unknown URI " + uri);
		}
		// 往下看
		getContext().getContentResolver().notifyChange(uri, null);

		return count;
	}
}
{% endhighlight %}

上面就是Content provider的实现，其中有四个关键的方法：`query()`、`insert()`、`delete()`、`update()`，负责数据的CRUD操作。

注意在`query()`方法最后有如下一句代码：
> c.setNotificationUri(getContext().getContentResolver(), uri);

在`insert()`、`delete()`、`update()`方法的最后有如下一句代码：
> getContext().getContentResolver().notifyChange(noteUri, null);

这两句代码分别是做什么的呢？
我这里只做一下简单的介绍，想要详细了解其内部的原理，请参考我的另一篇文章：[Cursor和CursorLoader中的观察者模式](/post/2014/11/14/Observer-between-Cursor-and-CursorLoader.html)。

简单来说，在`query()`方法中查询得到一个游标，你就持有了这个游标，然后你对这个游标做什么操作，比如使用`CursorAdapter`把结果显示到`ListView`中，都没问题。
但是当你通过`insert()`、`delete()`、`update()`方法修改了数据之后，你持有的游标就算作是过期了，你就需要重新获取这个游标；Android框架中应用了一种源/监听器的模式，会在数据源改变时发送通知自动更新这个游标。

那么上面的两句代码的作用就是实现当数据源发生改变之后，自动加载新数据的关键代码；具体到我们的`CursorLoader`中，就是当你对数据修改之后，`CursorLoader`会自动帮你加载新的数据，并调用`LoaderManager.LoaderCallbacks`中的`onLoadFinished()`传回最新的游标，你就可以更新界面等操作了。

# 装载器实现代码

{% highlight java %}
public static class ThrottledLoaderListFragment extends ListFragment
			implements LoaderManager.LoaderCallbacks<Cursor> {
    ...

	SimpleCursorAdapter mAdapter; // 显示装载器结果用的Adapter

	AsyncTask<Void, Void, Void> mPopulatingTask; // 用于填充数据库的异步任务

	@Override
	public void onActivityCreated(Bundle savedInstanceState) {
		super.onActivityCreated(savedInstanceState);
        ...

        // 初始化装载器
		getLoaderManager().initLoader(0, null, this);
	}

	@Overide
	public boolean onOptionsItemSelected(MenuItem item) {
		ContentResolver cr = getActivity().getContentResolver();

		switch (item.getItemId()) {
			case POPULATE_ID : // 使用异步任务往数据库填充数据
				if (mPopulatingTask != null) {
					mPopulatingTask.cancel(false);
				}
				mPopulatingTask = new AsyncTask<Void, Void, Void>() {
					@Override
					protected Void doInBackground(Void... params) {
						for (char c = 'Z'; c >= 'A'; c--) {
							if (isCancelled()) {
								break;
							}
							StringBuilder builder = new StringBuilder(
									"Data ");
							builder.append(c);
							ContentValues values = new ContentValues();
							values.put(MainTable.COLUMN_NAME_DATA,
									builder.toString());
							cr.insert(MainTable.CONTENT_URI, values);
							// Wait a bit between each insert.
							try {
								Thread.sleep(250);
							} catch (InterruptedException e) {
							}
						}
						return null;
					}
				};
				mPopulatingTask.executeOnExecutor(
						AsyncTask.THREAD_POOL_EXECUTOR, (Void[]) null);
				return true;

			case CLEAR_ID : // 使用异步任务清空数据库中的数据
				if (mPopulatingTask != null) {
					mPopulatingTask.cancel(false);
					mPopulatingTask = null;
				}
				AsyncTask<Void, Void, Void> task = new AsyncTask<Void, Void, Void>() {
					@Override
					protected Void doInBackground(Void... params) {
						cr.delete(MainTable.CONTENT_URI, null, null);
						return null;
					}
				};
				task.execute((Void[]) null);
				return true;

			default :
				return super.onOptionsItemSelected(item);
		}
	}

	// These are the rows that we will retrieve.
	static final String[] PROJECTION = new String[]{MainTable._ID,
			MainTable.COLUMN_NAME_DATA,};

    // 构造节流装载器
	public Loader<Cursor> onCreateLoader(int id, Bundle args) {
		CursorLoader cl = new CursorLoader(getActivity(),
				MainTable.CONTENT_URI, PROJECTION, null, null, null);
		cl.setUpdateThrottle(2000); // update at most every 2 seconds.
		return cl;
	}

	public void onLoadFinished(Loader<Cursor> loader, Cursor data) {
		mAdapter.swapCursor(data);

		// The list should now be shown.
		if (isResumed()) {
			setListShown(true);
		} else {
			setListShownNoAnimation(true);
		}
	}

	public void onLoaderReset(Loader<Cursor> loader) {
		mAdapter.swapCursor(null);
	}
}
{% endhighlight %}

装载器初始化时，我们调用了`setUpdateThrottle(2000)`方法，这个方法的作用是设置装载器的节流更新周期，参数类型是毫秒。

╮(╯3╰)╭


<br/>
参考：

1. [Android装载器（Loaders）框架简介](/post/2014/11/14/Use-Loaders-in-Android.html)
2. [Android Uri简介](http://blog.csdn.net/sunny09290/article/details/7514963)
3. [Android MIME学习笔记](http://www.apkbus.com/forum.php?mod=viewthread&tid=88509)
4. [What is cursor.setNotificationUri() used for?](http://stackoverflow.com/questions/21623714/what-is-cursor-setnotificationuri-used-for)
5. [CursorLoader not updating after notifyChange call](http://stackoverflow.com/questions/21273898/cursorloader-not-updating-after-notifychange-call)

本文出自[2dxgujun](/)，转载时请注明出处及相应链接。