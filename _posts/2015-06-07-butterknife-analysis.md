---
layout: post
title: butterknife 源码分析
category:
- Android
- Analysis
date: 2015-06-07
---

[butterknife](https://github.com/JakeWharton/butterknife)是一个快速Android视图注入框架，它的开发者是大名鼎鼎的[Jake Wharton](http://jakewharton.com/)。本文针对v6.1.0版本进行分析。

<!-- more -->

## 1. 功能介绍

Android开发中经常要获取各种各样的view，比如我们一般在activity的`onCreate()`方法中调用`findViewById()`方法获取view，然后再转换成我们需要的类型。这种做法会在类中包含大量“样板代码”，不利于维护和升级。

butterknife用来解决此类问题，它通过给view字段上添加一个Java注解，框架会自动注入这些字段。

### 1.1 注入方式

butterknife的注入方法与众不同，Java有很多依赖注入框架，比较有名的像Guice和Spring，此类注入框架非常强大，它们通过在运行时读取注解实现注入，依赖的生成和注入都需要依靠Java的反射机制，反射对于性能敏感的Android来说是一个硬伤，因此此类注入框架普遍应用于JavaEE开发。

butterknife同样使用注解来实现依赖注入，但它利用APT（Annotation Process Tool）在编译时生成辅助类，这些类继承特定父类或实现特定接口，程序运行时加载这些辅助类，调用相应接口完成依赖注入。

使用butterknife实现依赖注入的开销仅仅是在编译时刻做的注解处理，程序运行时的开销几乎可以忽略不计。

### 1.2 基本使用

给view字段添加`@InjectView`注解，参数为view的ID，butterknife会根据ID找到这个view，然后自动完成类型转换：

```java
class ExampleActivity extends Activity {
  @InjectView(R.id.title) TextView title;
  @InjectView(R.id.subtitle) TextView subtitle;
  @InjectView(R.id.footer) TextView footer;

  @Override public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.simple_activity);
    ButterKnife.inject(this);
    // TODO Use "injected" views...
  }
}
```
在`onCreate()`方法中调用`ButterKnife.inject(this)`委托生成的辅助类进行注入，辅助类的代码类似：

```java
public void inject(ExampleActivity activity) {
  activity.subtitle = (android.widget.TextView) activity.findViewById(2130968578);
  activity.footer = (android.widget.TextView) activity.findViewById(2130968579);
  activity.title = (android.widget.TextView) activity.findViewById(2130968577);
}
```

### 1.3 集合注入

butterknife可以把多个视图注入一个数组或`List`。

```java
@InjectViews({ R.id.first_name, R.id.middle_name, R.id.last_name })
List<EditText> nameViews;
```
可以对集合元素执行批量操作：

```java
ButterKnife.apply(nameViews, DISABLE);
ButterKnife.apply(nameViews, ENABLED, false);
```

`Action`和`Setter`接口可以设置简单的操作：

```java
static final Action<View> DISABLE = new Action<>() {
  @Override public void apply(View view, int index) {
    view.setEnabled(false);
  }
}
static final Setter<View, Boolean> ENABLED = new Setter<>() {
  @Override public void set(View view, Boolean value, int index) {
    view.setEnabled(value);
  }
}
```

### 1.4 监听器注入

butterknife还可以注入监听器，例如，给方法添加一个`@OnClick`注解，参数中的ID为目标view的ID，当这个view被点击时，就会调用这个方法。

```java
@OnClick(R.id.submit)
public void submit(View view) {
  // TODO submit data to server...
}
```

## 2. 总体设计

![butterknife structure](http://ww3.sinaimg.cn/large/bce2dea9jw1eswif6qc5fj20nk0i9acd.jpg)

### 2.1 概述

butterknife在编译时刻利用APT分析程序代码，扫描每一个有注解的类，找出类中带有注解的字段生成`ViewBinding`，带有注解的方法生成`ListenerBinding`，最终组合成一个`ViewInjector`，再利用Java的`Filer`API生成一个包含注入代码的辅助类，程序中调用`ButterKnife.inject()`方法时加载这个辅助类实现依赖注入。

### 2.2 butterknife注解

butterknife中有两种类型的注解，第一种注解表示程序代码中的注入点；第二种注解在框架内部使用，为第一种注解定义元数据。

第一种注解又分成两类：

- **视图注入注解**：`@InjectView`和`@InjectViews`，标记字段，用于注入视图。
- **监听器注入注解**：`@OnTouch`和`@OnClick`等等，标记方法，用于注入视图监听器。

第二种注解位于*butterknife.internal*包中：`@ListenerClass`和`@ListenerMethod`，它们为监听器方法注解提供元数据，例如下面定义了一个`@OnCheckedChanged`注解：

```java
@Target(METHOD)
@Retention(CLASS)
@ListenerClass(
    targetType = "android.widget.CompoundButton",
    setter = "setOnCheckedChangeListener",
    type = "android.widget.CompoundButton.OnCheckedChangeListener",
    method = @ListenerMethod(
        name = "onCheckedChanged",
        parameters = {
            "android.widget.CompoundButton",
            "boolean"
        }
    )
)
public @interface OnCheckedChanged {
  /** View IDs to which the method will be bound. */
  int[] value() default { View.NO_ID };
}
```
`@ListenerClass`注解表示一个监听器，它是一个元注解，其中声明了一系列参数；修改这些参数，就可以代表不同的监听器，比如上面的`@ListenerClass`定义了一个`OnCheckedChangeListener`监听器，定义一个监听器需要设置目标类型、setter方法、监听器类型、回调方法、等等。

`@ListenerMethod`注解表示监听器中的一个回调方法，例如`OnItemClickListener`中的`onItemClick()`方法，其中声明了一系列参数，修改这些参数，就可以表示代表不用的回调方法，这个注解和`@ListenerClass`注解中的`method()`和`callbacks()`一起工作，代表监听器中的一个或多个回调方法。

此外还有一个注解：`@Optional`，这是一个独立的注解。默认情况下注入时如果找不到目标视图，会抛出一个异常，使用`@Optional`注解就可以抑制这种行为。

### 2.3 注入单元

butterknife使用APT处理代码中的注解时，会边处理边建立一个“模型”，模型是一个数据结构，最终的辅助类就是根据这个模型生成的。

`ViewInjection`类是这个模型当中的的重要组成部分，我把它叫做注入单元。模型中包含大量注入单元，每一个注入单元把一个类中同一个id的注入点信息封装起来，即把一个view的字段注入点的信息和它的监听器注入信息都放在这个`ViewInjection`中。

```java
private final int id;
private final Set<ViewBinding> viewBindings = new LinkedHashSet<ViewBinding>();
private final LinkedHashMap<ListenerClass, Map<ListenerMethod, Set<ListenerBinding>>>
    listenerBindings = new LinkedHashMap<ListenerClass,
    Map<ListenerMethod, Set<ListenerBinding>>>();
```

`ViewBinding`和`ListenerBinding`是都是`Binding`类型，bindings封装了这些注入点信息，一共有三种类型的注入点，也就存在三类binding：
- `ViewBinding`：表示视图注入点，封装了字段名和类型。
- `CollectionBinding`：表示集合注入点，封装了字段名、集合类型和集合元素类型。
- `ListenerBinding`：表示监听器注入点，封装了方法名和方法参数。

每个binding中还有一个`required`标记用于表明是否是“可选注入”（注入点是否有`@Optional`注解）。

注入单元由这些bindings组成，APT处理过程中，会把每一个带有注解的类中同一个id的注入点信息封装成一个`ViewInjection`对象。

那么由什么来封装`ViewInjection`对象呢？

### 2.4 注入器

`ViewInjector`类代表注入器，它是注入单元的外层包装，它和一个`TypeElement`类型的映射关系组成这个“模型”，也就是APT处理得到的最终数据。

`TypeElement`是一个程序元素，表示一个接口或类，它在模型中表示一个带有注解的类，换句话说，这个模型就是带有注解的类和注入器组成的`Map`。

每一个注入器都对应一个辅助类，即每个带有注解的类最后都会生成一个辅助类来实现注入。

```java
...
private final Map<Integer, ViewInjection> viewIdMap = new LinkedHashMap<Integer, ViewInjection>();
private final Map<CollectionBinding, int[]> collectionBindings =
      new LinkedHashMap<CollectionBinding, int[]>();
```
注入单元`ViewInjection`把一个类中同一个id的注入点信息封装起来，而注入器`ViewInjector`又把视图id和注入单元封装成一个`Map`，集合注入点的信息被单独封装成另一个`Map`。

![class diagram](http://ww1.sinaimg.cn/large/bce2dea9jw1eswif61ylpj20e60ci0tl.jpg)

### 2.5 工作流程

编译时，APT会扫描所有的代码文件，根据代码中的注解生成一些辅助类，然后将这些新生成的类文件同项目文件一起编译成字节码文件。
![compile-time flow chart](http://ww1.sinaimg.cn/large/bce2dea9jw1eswifdymnwj20bw0fmwf2.jpg)


运行时，在适当的位置调用`ButterKnife#inject()`方法，调用这个方法会加载之前生成的辅助类实现依赖注入。
![run-time flow chart](http://ww2.sinaimg.cn/large/bce2dea9jw1eswif5h59kj20cc0d6wew.jpg)

## 3. 详细设计

### 3.1 监听器注入注解

butterknife中有很多监听器注入注解，如`@OnClick`、`@OnItemClick`、`@OnItemSelect`等等。

这些注解用来标注方法，butterknife会为id指定的视图设置监听器，例如下面的代码为`example_list`设置了`OnItemClickListener`监听器。

```java
@OnItemClick(R.id.example_list)
void onItemClick(int position) {
   Toast.makeText(this, "Clicked position " + position + "!", LENGTH_SHORT).show();
}
```

监听器注入和视图注入都是通过辅助类中的注入代码实现，注入`OnItemClickListener`监听器的代码如下所示：

```java
((android.widget.AdapterView<?>) view).setOnItemClickListener(
  new android.widget.AdapterView.OnItemClickListener() {
    @Override public void onItemClick(
      android.widget.AdapterView<?> p0,
      android.view.View p1,
      int p2,
      long p3
    ) {
      target.onItemClick(p2);
    }
  });
```
一个`@OnItemClick`注解就能生成上面的注入代码，这是因为butterknife为这些注解定义了生成注入代码所需的全部数据，把设置监听器的代码查分开来，可以得到以下几个组成部分：

- 目标类型：`android.widget.AdapterView<?>`
- setter方法名：`setOnItemClickListener`
- 监听器类名：`android.widget.AdapterView.OnItemClickListener`
- 监听器回调方法

这些数据由`@ListenerClass`注解提供：

```java
@ListenerClass(
    targetType = "android.widget.AdapterView<?>",
    setter = "setOnItemClickListener",
    type = "android.widget.AdapterView.OnItemClickListener",
    method = @ListenerMethod(
        name = "onItemClick",
        parameters = {
            "android.widget.AdapterView<?>",
            "android.view.View",
            "int",
            "long"
        }
    )
)
```

监听器回调方法由另外一个注解（`@ListenerMethod`）提供。

一个监听器中可能包含多个回调方法，比如`OnItemSelectedListener`中有两个回调方法，butterknife提供了解决方案。`@ListenerClass`注解中有两个参数用来设置回调方法：

- `method`：定义一个回调方法；
- `callbacks`：定义多个回调方法，使用一个枚举来定义这些回调方法。

这两个参数只能设置其中一个。

当只有一个回调方法时，只需要给`method`参数指定一个`@ListenerMethod`注解既可；当有多个回调方法时，首先需要定义一个枚举类，为枚举常量添加`@ListenerMethod`注解表示一个回调方法，给`callbacks`参数指定枚举类类实例，代码片段如下：

```java
...
@ListenerClass(
    ...
    callbacks = OnItemSelected.Callback.class
)
public @interface OnItemSelected {
  ...
  /** Listener callback to which the method will be bound. */
  Callback callback() default Callback.ITEM_SELECTED;

  /** {@link OnItemSelectedListener} callback methods. */
  enum Callback {
    /**
     * {@link OnItemSelectedListener#onItemSelected(android.widget.AdapterView, android.view.View,
     * int, long)}
     */
    @ListenerMethod(...)
    ITEM_SELECTED,

    /** {@link OnItemSelectedListener#onNothingSelected(android.widget.AdapterView)} */
    @ListenerMethod(...)
    NOTHING_SELECTED
  }
}
```

给方法添加注解时，需要指定其`callback`参数：

```java
@OnItemSelected(value = R.id.example_list, callback = NOTHING_SELECTED)
void onNothingSelected() {
  Toast.makeText(this, "Nothing selected!", LENGTH_SHORT).show();
}
```

如果没有指定`callback`参数，采用定义的默认值。

### 3.2 处理注解

APT中有一个`AbstractProcessor`抽象类，这个类中有一个抽象方法`process()`。`ButterKnifeProcessor`类继承`AbstractProcessor`类，这个类即为一个注解处理器，程序在编译时刻会自动调用它的`process()`方法，在这个方法中实现所有的处理逻辑。

APT会扫描所有的代码文件，每找到一个注解，butterknife就会执行一系列检查，当检查通过后，这个注解的信息就会被记录到“模型”中。

*javax.lang.model*是用来为Java编程语言建立模型的包和类的层次结构，此包及其子包的成员适用于语言建模、语言处理任务。处理注解时需要大量用到这个包和其子包中的类。

#### 3.2.1 处理`@InjectView`注解

`Element`代表Java语言中的一个程序元素，`env.getElementsAnnotatedWith(InjectView.class)`方法返回一个带有`@InjectView`注解的字段集合。

调用`parseInjectView()`方法对这些字段进行处理，处理过程中包含一系列检查：

- 第一步，判断目标字段的定义类型是否是`View`的子类型或者是一个接口类型；
- 第二步，检查目标字段的可访问性，要求目标字段必须是类字段且字段和类的访问修饰符都不能为`private`，这是因为butterknife生成的注入代码直接给目标字段赋值实现依赖注入，字段或类修饰成`private`后，这个字段就无法在外部访问了，也就无法注入了；
- 第三步，检查这个类的包名，包名不能以`android.`和`java.`开头，butterknife不可以在Android Framework和JDK框架内部使用；
- 第四步，检查是否有多余的注解：`@InjectView`和`@InjectViews`注解不能同时标注一个字段；
- 第五步，检查是否存在同一个id的多个`@InjectView`注解，butterknife把这种行为当作错误来处理。

以上五步检查通过之后，这个注入点的信息会被放置到模型中，这样就完成了一个`@InjectView`注解的处理。

#### 3.2.2 处理`@InjectViews`注解

`@InjectViews`注解和`@InjectView`的处理逻辑相似，执行的检查稍有不同，首先要检查目标字段声明类型，声明类型只能是数组或者是`List`，然后检查数组元素类型或者集合元素类型。`@InjectViews`的注解id参数不能为空，也不能包含重复的id。最后把这个集合注入点的信息放入到模型中。

#### 3.2.3 处理监听器注入注解

监听器注入注解的处理比较复杂，这个注解标注在方法上，除了要对目标方法进行一般的检查外，还需要把目标方法同元数据（`@ListenerClass`注解提供的监听器数据）进行校对，绑定方法参数。

目标方法的一般检查包含以下五步：

- 第一步，检查目标类型，目标必须是一个方法，和`@Target(METHOD)`注解形成双重验证。
- 第二步，检查注解，注解必须包含一个类型为`int[]`的`value()`参数，表示视图的id。
- 第三步，检查目标的可访问性，类和方法的访问修饰符不能为`private`且方法不可以是静态的。
- 第四步，检查目标方法所在的类的包名，包名不能以`android.`和`java.`开头，butterknife不可以在Android Framework和JDK框架中使用。
- 第五步，检查注解参数中是否包含重复的ID。

获取注解参数（id数组）时，用到了反射：

```java
Annotation annotation = element.getAnnotation(annotationClass);
Method annotationValue = annotationClass.getDeclaredMethod("value");
...
int[] ids = (int[]) annotationValue.invoke(annotation);
```

此处无法像处理`@InjectView`注解那样通过`value()`参数直接取得ID，因为每个监听器注解都是独立的注解类型，由框架维护者负责维护这种独立性间的“协议”（每个监听器注解都有一个接收`int[]`类型的`value()`参数），此类注解只能使用其父类类型即`Annotation`进行一般化处理，遵守“协议”的注解都可以被正确的处理。

#### 3.2.4 处理“特殊情况”

在处理监听器注入注解的方法中有一段代码，这里执行了一个特殊的检查。

Android中有一种特殊的监听器——这个监听器监听的对象就是当前类（view内部定义的监听器），也就是说，不需要额外指定监听目标，监听目标就是其本身。

butterknife支持注入这种类型的监听器，注入此类监听器的注解不需要设置ID参数，也不能带有`@Optional`注解，同时还要对目标方法的enclosing element进行检查（方法的enclosing element即为方法所在类的类型）：目标方法所在类的类型必须是注解元数据中`targetType`参数所指定的目标类型的子类型，这样才能保证监听器被正确设置。

#### 3.2.5 校对元数据

处理监听器注入注解时，要对`@ListenerClass`注解定义的元数据进行解析。

然后把目标方法的参数列表和返回值同解析得到的回调方法对象`ListenerMethod`进行校对。

#### 3.2.6 绑定方法参数

定义目标方法时，不需要完整地声明参数列表，例如，`OnItemClickListener`监听器中的回调方法`onItemClick(android.widget.AdapterView<?>, android.view.View, int, long)`有四个参数，定义目标方法时，可以根据需要定义0个或多个参数，如下所示：

```java
@OnItemClick(R.id.list_of_things)
void onItemClick(int position) {
  Toast.makeText(this, "You clicked: " + adapter.getItem(position), LENGTH_SHORT).show();
}
```

butterknife利用*javax.lang.model*包和其子包内的类对方法参数进行分析处理，具体处理方法如下：

外层循环遍历目标方法的参数，内层循环遍历回调方法的参数，如果目标方法参数是回调方法参数的子类型或者目标方法参数是接口类型，就表示两个类型匹配，初始化一个`Parameter`对象完成参数的绑定；以后每次遍历回调方法参数时，都会跳过先前绑定过的参数位置，一轮下来如果这个目标方法参数没有找到匹配的回调方法参数与之绑定，编译就会报错而终止。

例如一个回调方法有四个参数：A、B、C、C，三种类型，目标方法定义有三个参数B、C、D，则绑定过程示意如下：

第一轮：B→A，B→B（绑定）
第二轮：C→A，C→B（跳过），C→C（绑定）
第三轮：D→A，D→B（跳过），D→C（跳过），D→C，报错终止

### 3.3 父级注入器

假如程序代码中有两个类：A和B，B继承自A，两个类都需要注入，butterknife会生成两个辅助类；当给B注入依赖时，不光仅仅注入B中的依赖，其位于父类中的依赖也应该要注入——依赖关系也要被继承。

此处的父级注入器就是以上问题的解决方案。

以上问题中，让B的辅助类继承A的辅助类，然后在B的注入代码前先调用父类的注入代码，就能保证父类的依赖先于子类被注入。

处理注解完之后，“模型”已经建立完成，此时对这个模型进行分析找到目标类之间的父子关系：

```java
// Try to find a parent injector for each injector.
for (Map.Entry<TypeElement, ViewInjector> entry : targetClassMap.entrySet()) {
  String parentClassFqcn = findParentFqcn(entry.getKey(), erasedTargetNames);
  if (parentClassFqcn != null) {
    entry.getValue().setParentInjector(parentClassFqcn + SUFFIX);
  }
}
```

父级注入器通过`ViewInjector`的`setParentInjector()`方法设置。

### 3.4 生成辅助类

“模型”已经建立，辅助类之间的继承关系也已经确定。此时代码还没有编译，还有最后一项工作要做——生成辅助类。

```java
JavaFileObject jfo = filer.createSourceFile(viewInjector.getFqcn(), typeElement);
Writer writer = jfo.openWriter();
writer.write(viewInjector.brewJava());
writer.flush();
writer.close();
```

使用`Filer`API创建辅助类文件，`ViewInjector`的`brewJava()`方法根据模型“酝酿”Java代码，之后使用Java IO流把代码写入文件。

### 3.5 酝酿注入代码

butterknife为“模型”中每一个注入器生成一个辅助类，辅助类实现了`Injector`接口，注入依赖时，程序通过接口调用辅助类方法。

#### 3.5.1 `Injector`接口

`Injector`接口位于`ButterKnife`类中，该接口中定义如下：

```java
public interface Injector<T> {
  void inject(Finder finder, T target, Object source);
  void reset(T target);
}
```

`inject()`、`reset()`分别表示注入和重置操作，所有辅助类都实现这个接口，调用`ButterKnife.inject()`方法会委托调用辅助类的`inject()`方法。

`inject()`方法中有三个参数：
- `target`：注入目标，它包含需要注入的依赖，可以是任何类的对象；
- `source`：注入源，butterknife仅支持三种注入源：`Activity`、`Dialog`、`View`，注入时会调用注入源的`findViewById()`来查找视图；
- `finder`：查找器，查找器负责适配在不同的注入源中查找视图的方法。

#### 3.5.2 辅助类代码框架

辅助类有两种代码框架，第一种辅助类代码框架用于注入不包含依赖继承关系的普通目标类：

```java
public class HelloActivity$$ViewInjector<T extends HelloActivity> implements Injector<T> {
  @Override public void inject(final Finder finder, final T target, Object source) {
    ...
  }
  @Override public void reset(T target) {
    ...
  }
}
```

辅助类的命名为目标类名后面附加`$$ViewInjector`，上面注入的目标类为`HelloActivity`，其还有个参数类型**`T`**表示注入目标的类型，然后实现`Injector`接口。

注入时，实例化这个辅助类对象，调用其`inject()`方法，第一个参数框架会根据注入源提供，如果注入源为`Activity`实例，那么就提供activity的`Finder`。

第二种辅助类代码框架应用于存在依赖继承关系的子类目标：

```java
public class SimpleActivity$$ViewInjector<T extends SimpleActivity> extends HelloActivity$$ViewInjector<T> {
  @Override public void inject(final Finder finder, final T target, Object source) {
    super.inject(finder, target, source);
    ...
  }

  @Override public void reset(T target) {
    super.reset(target);
    ...
  }
}
```

`SimpleActivity`继承`HelloActivity`，两个类中都存在需要注入的依赖。当注入子类`HelloActivity`中的依赖时，先要注入其父类的依赖，因此两个目标类的辅助类也引入继承关系。

#### 3.5.3 字段注入代码

字段注入代码如下所示：

```java
View view;
view = finder.findRequiredView(source, 2130968576, "field 'title'");
target.title = finder.castView(view, 2130968576, "field 'title'");
```
butterknife调用`Finder`的`findRequiredView()`方法查找视图对象，然后直接把视图对象赋值给目标注入点。

集合注入点的注入代码如下所示：
```java
target.headerViews = Finder.listOf(
	finder.<android.view.View>findRequiredView(source, 2130968576, "field 'headerViews'"),
    finder.<android.view.View>findRequiredView(source, 2130968577, "field 'headerViews'"),
    finder.<android.view.View>findRequiredView(source, 2130968578, "field 'headerViews'")
);
```

`Finder`的`listOf()`方法是一个工具方法，负责把数组类型转换成`List`类型，此处调用`findRequiredView()`方法中的参数化类型为注入点的集合元素类型。

#### 3.5.5 监听器注入代码

监听器注入代码如下：

```java
((android.widget.AdapterView<?>) view).setOnItemSelectedListener(
  new android.widget.AdapterView.OnItemSelectedListener() {
    @Override public void onItemSelected(
      android.widget.AdapterView<?> p0,
      android.view.View p1,
      int p2,
      long p3
    ) {
      target.onItemSelected(p2);
    }
    @Override public void onNothingSelected(
      android.widget.AdapterView<?> p0
    ) {
      target.onNothingSelected();
    }
  });
```

给目标视图设置一个指定的监听器，然后在监听器回调方法中委托调用目标类中的方法。

<br/>
本文出自[2dxgujun](/)，转载时请注明出处及相应链接。