---
layout: post
title: otto 源码分析
category:
- Android
- Analysis
date: 2015-06-07
---

[otto](http://square.github.io/otto/)是[Square](http://square.github.io/)公司出品的一个发布-订阅模式框架，它基于Google [Guava](https://github.com/google/guava)项目中的event bus模块开发，针对Android平台做了优化和加强。本文针对v1.3.6版本进行分析。

<!-- more -->

## 1. 功能介绍

otto基于发布-订阅模式实现了组件间的通信，降低了类之间的耦合度，简化了组件间的通信，它被设计成用来替代Android中传统的进程间通信机制。

它与传统的发布-订阅模式不同，传统的发布-订阅模式会把发布者和订阅者耦合在一起（把订阅者注册到发布者），而otto中没有这种注册关系，它实现了一种基于事件类型的升级型发布-订阅模式，实现发布者和订阅者的完全解耦。

### 1.1 基本原理

otto中有一条“事件总线”，它统筹管理发布者和订阅者，并且负责事件调度。使用时需要把发布者和订阅者注册到总线，事件直接发送到总线，总线负责调度这些事件，分发给满足处理条件的订阅者处理。

otto中的发布者和传统的发布者不同，它负责初始化组件；有些组件可能会订阅一些特殊事件（例如，GPS坐标、登录用户，等），此类组件希望在注册之后马上获取这些事件对象（这些事件对象通常存在于其它组件中，且已经初始化完成）用来初始化一些属性，otto提供了解决方案——发布者。

组件在注册时，如果存在匹配新注册的订阅者订阅的事件类型的发布者，发布者会被直接调用来触发这些订阅者；同样，如果存在匹配订阅了新注册的发布者发布的事件类型的订阅者，发布者会被直接调用来触发这些订阅者。

传统的发布-订阅模式中的发布者和订阅者是类，订阅者必须要注册到发布者后，发布者才能给这个订阅者发送消息；而Otto中的发布者和订阅者都是方法，`@Produce`注解标记的方法为发布者，`@Subscribe`注解标记的方法为订阅者，一个类中可以有任意多个发布者和订阅者。

程序在运行时通过`register()`方法注册目标组件，框架会得到这个组件中的发布者和订阅者；当有事件产生时，框架会根据事件类型查找满足处理条件的订阅者，触发订阅者处理事件。

### 1.2 基本使用

使用时需要创建一个`Bus`实例，它代表一条总线，实际使用中最好作为一个单例依赖。

```java
Bus bus = new Bus();
```

事件在总线上面传递，任何一个类的实例都可以作为一个事件发送到总线上，调用`post()`方法，发布一个事件：

```java
bus.post(new AnswerAvailableEvent(42));
```

有发布就有订阅，订阅一个事件只需要定义一个方法，标记`@Subscribe`注解，这个方法就是一个订阅者，该方法只能有一个参数，参数类型即为订阅的事件类型。订阅上面事件的方法：

```java
@Subscribe
public void answerAvailable(AnswerAvailableEvent event) {
	// TODO: React to the event somehow!
}
```

定义好之后，还需要注册目标组件，让总线得到这个目标中的发布者和订阅者，假如`this`为订阅者所属类的实例，注册代码如下：

```java
bus.register(this);
```

otto同Guava event bus不同，	它不会把在父类或接口中定义的发布者和订阅者注册进来，这种设计方案不仅提升了性能，而且使代码尽量简单，避免出现模糊不清的语义。

最后不要忘了在恰当的时机调用`unregister()`方法，注销这个组件。

最后，定义发布者。发布者是一个标记了`@Produce`注解的方法，它没有参数，返回值类型即为发布的事件类型。例如上面事件的发布者：

```java
@Produce
public AnswerAvailableEvent produceAnswer() {
	// Assuming 'lastAnswer' exists.
    return new AnswerAvailableEvent(this.lastAnswer);
}
```

在一条总线上，针对每种事件类型只能存在一个发布者，否则会产生歧义。

## 2. 总体设计

![otto design diagram](http://ww3.sinaimg.cn/large/bce2dea9jw1eswiqj7ydoj20po0dx40f.jpg)
框架整体可以分成四个部分：

- **Annotations**：发布者和订阅者注解
- **Handler Finders**：分析程序代码，查找发布者和订阅者
- **Wrapper**：封装订阅者和发布者的数据结构
- **`Bus`类**：事件总线，注册/注销组件，维护发布-订阅模型，事件调度分发

### 2.1 概述

otto通过在运行时注册组件，分析目标类中声明的方法，查找带有`@Produce`和`@Subscribe`注解的方法，最终建立一个基于事件类型的发布-订阅模型。

`Bus`类中维护一个队列，用于表示总线，调用`post()`方法发布事件，事件调度依据之前建立的模型进行；满足条件的订阅者和事件对象被封装起来依次压入队列，最后触发这个队列分发处理事件。

### 2.2 工作流程

调用`register()`方法注册组件，发布者先于订阅者被注册，保证先注册的发布者被正确触发为后注册的订阅者提供初始值。
![register](http://ww3.sinaimg.cn/large/bce2dea9jw1eswiqoy5sjj20k10miwfq.jpg)

发布事件时，首先对事件的类型进行分析，根据事件类型和其所有父类型找到满足处理条件的订阅者，把这些订阅者和事件对象压入总线队列，之后依次分发处理这个队列中的所有元素。

![post event](http://ww2.sinaimg.cn/large/bce2dea9jw1eswiqk77ulj20jq0p9q4b.jpg)

## 3. 详细设计

### 3.1 发布者和订阅者

因为otto中的发布者和订阅者是方法，所以需要使用Java反射包中的`Method`类，`Method`类仅仅表示一个类方法，要执行这个方法，还需要提供一个该方法所属类的实例。

otto把一个`Method`和一个`Object`封装在一起，组成一个执行单元。分别定义了两个类`EventProducer`和`EventHandler`用于表示发布者和订阅者。

### 3.2 发布-订阅模型

发布-订阅模型是一个数据结构，它是实现这种基于事件类型的调度机制的核心。

`Bus`类中定义了如下两个`Map`：

```java
/** All registered event handlers, indexed by event type. */
private final ConcurrentMap<Class<?>, Set<EventHandler>> handlersByType =
        new ConcurrentHashMap<Class<?>, Set<EventHandler>>();

/** All registered event producers, index by event type. */
private final ConcurrentMap<Class<?>, EventProducer> producersByType =
        new ConcurrentHashMap<Class<?>, EventProducer>();
```

这两个`Map`即为发布-订阅模型，`Map`的key为事件类型，value分别为订阅者集合和发布者。

### 3.3 注册

运行时需要把组件注册到`Bus`总线，这个组件当中的发布者和订阅者就会被缓存在发布-订阅模型中。

首先注册发布者：调用`HandlerFinder`的`findAllProducers()`方法获取目标中的发布者，把发布者依次放入发布-订阅模型中；如果程序找到满足条件的订阅者，则触发新注册的发布者通知订阅者处理事件。

然后再注册订阅者，订阅者的注册和发布者相似，一个组件中可能有多个订阅者，所以注册完之后，要依次触发满足这些订阅者处理条件的发布者，通知刚注册的订阅者处理事件。

**注意**：发布-订阅模型中，每个事件类型只能有一个发布者，否则抛出异常。

### 3.4 注销

注销用于把一个组件中的发布者和订阅者从发布-订阅模型中移除，通常在一个组件生命周期的最后调用`unregister()`方法注销。

注销的顺序和注册的顺序相同，先注销发布者，再注销订阅者，防止调用发布者产生事件时，出现订阅者无效的情况，导致额外开销。

**注意**：注销时，除了把发布者和订阅者从发布-订阅模型中移除，还调用了`invalidate()`方法，因为在多线程环境中，事件的调度和注册/注销操作可能同步进行，注销操作还要中止在总线队列中的等待的事件后续的执行。

### 3.5 事件调度

otto的event bus模式事件调度有一个重要规则：**事件调度仅在发送线程域中进行**。

线程A中发送的事件，调度和处理都在线程A中进行，这样在其它线程中发送的事件就不会阻塞Android主线程的运行。

otto使用`ThreadLocal`维护事件队列，即每个线程域都有一条独立的总线；调用`post()`方法发送的事件会被放入当前线程域的总线中，即发送的事件仅存在于发送线程域。

```java
/** Queues of events for the current thread to dispatch. */
private final ThreadLocal<ConcurrentLinkedQueue<EventWithHandler>> eventsToDispatch =
    new ThreadLocal<ConcurrentLinkedQueue<EventWithHandler>>() {
      @Override protected ConcurrentLinkedQueue<EventWithHandler> initialValue() {
        return new ConcurrentLinkedQueue<EventWithHandler>();
      }
    };
```

在`post()`方法中调用`enqueueEvent()`方法把事件对象压入当前线程域的事件队列：

```java
protected void enqueueEvent(Object event, EventHandler handler) {
  eventsToDispatch.get().offer(new EventWithHandler(event, handler));
}
```

最后调用`dispatchQueuedEvents()`方法，依次执行当前线程域的事件队列中的事件对象。

### 3.6 线程控制

otto提供了线程控制策略，用于控制事件调度的执行线程，这些策略会在注册/注销组件和发送事件时实施。

下面的代码实现了一个控制事件在主线程执行的策略：

```java
/** A {@link ThreadEnforcer} that confines {@link Bus} methods to the main thread. */
ThreadEnforcer MAIN = new ThreadEnforcer() {
  @Override public void enforce(Bus bus) {
    if (Looper.myLooper() != Looper.getMainLooper()) {
      throw new IllegalStateException("Event bus " + bus + " accessed from non-main thread " + Looper.myLooper());
    }
  }
};
```
使用这个控制策略是为了强制事件调度在主线程进行，保证了事件的发送和处理都在主线程，如果在其它线程中，这个控制策略会抛出一个异常。

默认情况下，强制在主线程中分发调度事件：

```java
// Both of these are functionally equivalent.
Bus bus1 = new Bus();
Bus bus2 = new Bus(ThreadEnforcer.MAIN);
```

如果事件调度发生在其它线程中，程序就会抛出异常。

如果不关心在哪个线程中分发事件，则可以通过`ThreadEnforcer.ANY`参数来初始化`Bus`对象。
如果上面两个线程控制方式无法满足需求，还可以通过实现`ThreadEnforcer`接口来定义自己的线程控制策略。

### 3.7 处理注解

`AnnotatedHandlerFinder`类是一个工具类，它负责获取目标组件的发布者和订阅者。

`loadAnnotatedMethods()`方法负责查找目标类当中带有`@Produce`和`@Subscribe`注解的方法，把这些方法放入缓存。缓存是一个两级`Map`结构，外层key为组件类型，内层key为事件类型。

```java
/** Cache event bus producer methods for each class. */
private static final Map<Class<?>, Map<Class<?>, Method>> PRODUCERS_CACHE =
    new HashMap<Class<?>, Map<Class<?>, Method>>();

/** Cache event bus subscriber methods for each class. */
private static final Map<Class<?>, Map<Class<?>, Set<Method>>> SUBSCRIBERS_CACHE =
    new HashMap<Class<?>, Map<Class<?>, Set<Method>>>();
```

`findAllProducers()`和`findAllSubscribers()`方法返回目标对象中的发布者和订阅者，它们先在缓存中查找目标类型的发布者和订阅者方法，如果找不到就调用`loadAnnotatedMethods()`方法加载并缓存这些方法，如果找到就把`Method`对象和目标实例封装成发布者和订阅者并返回。

<br/>
本文出自[2dxgujun](/)，转载时请注明出处及相应链接。