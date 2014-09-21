---
title: Java状态模式（State Pattern）——处理对象的多种状态及其相互转换
layout: post
guid: 2014092001
date: 2014-09-20 22:07:00
description: 
tags:
  - Pattern
---

目录：

- [定义](#定义)
- [概述](#概述)
- [适用性](#适用性)
- [结构](#结构)
- [角色](#角色)
- [协作](#协作)
- [效果](#效果)
- [实现](#实现)
- [小试身手](#小试身手)
    1. [银行系统中账户类的设计：由具体状态类来实现状态转换](#小试身手_1)
    2. [开关的设计：实现共享状态](#小试身手_2)
    3. [屏幕放大镜工具：由环境类来实现状态转换](#小试身手_3)

<br/>

<a id="定义"></a>
#定义
---
允许一个对象在其内部状态改变时改变它的行为，对象看起来似乎修改了它的类。其别名为**状态对象（Objects for States）**，状态模式是一种对象行为型模式。

<a id="概述"></a>
#概述
---
状态模式用于**解决系统中复杂对象的状态转换以及不同状态下行为的封装问题**。状态模式**将一个对象的状态从该对象中分离出来，封装到专门的状态类中**，使得对象状态可以灵活变化，对于客户端而言，无须关心对象状态的转换以及对象所处的当前状态，无论对于何种状态的对象，客户端都可以一致处理。

<a id="适用性"></a>
#适用性
---
在以下情况可以使用State模式：

1. 系统中某个对象存在多个状态，这些状态之间可以进行转换，并且对象在不同状态下行为不同。
2. 对象的行为依赖于它的状态（如某些属性值），状态的改变将导致行为的变化。
3. 在代码中包含大量与对象状态有关的条件语句，这些条件语句的出现，会导致代码的可维护性和灵活性变差，不能方便地增加和删除状态，并且导致客户类与类库之间的耦合增强。

<a id="结构"></a>
#结构
---
在状态模式中引入了抽象状态类和具体状态类，它们是状态模式的核心，其结构如图1所示：
![structure](/media/files/pattern/state/structure.png)
<div align="center">图1 状态模式结构图</div>

<a id="角色"></a>
#角色
---
**Context（环境类）**
环境类又称为上下文类，它是拥有多种状态的对象。由于在环境类的状态存在多样性且在不同的状态下对象的行为有所不同，因此将状态独立出去形成单独的状态类。在环境类中维护一个抽象状态类State的实例，这个实例定义当前状态，在具体实现时，它是一个State子类的对象。

**State（抽象状态类）**
它用于定义一个接口以封装与环境类的一个特定状态相关的行为，在抽象状态类中声明了各种不同状态对应的方法，而在其子类中实现这些方法，由于不同状态下对象的行为可能不同，因此在不同子类中方法的实现可能存在不同，相同的方法可以写在抽象状态类中。

**ConcreteState（具体状态类）**
它是抽象状态类的子类，每一个子类实现一个与环境类的一个状态相关的行为，每一个具体状态类对应环境类的一个具体状态，不同的具体状态类其行为有所不同。

<a id="协作"></a>
#协作
---
在状态模式中，我们将对象在不同状态下的行为封装到不同的状态类中，为了让系统具有更好的灵活性和可扩展性，同时对个状态下的共有行为进行封装，我们需要对状态进行抽象，引入了抽象状态类角色，其典型代码如下所示：

{% highlight java %}
abstract class State {
    // 声明抽象业务方法，不同的具体状态类可以不同的实现
    public abstract void handle();
}
{% endhighlight %}

在抽象状态类的子类即具体状态类中实现了抽象状态类中声明的业务方法，不同的具体状态类可以提供完全不同的方法实现，在实际使用中，在一个状态类中可能包含多个业务方法，如果在具体状态类中某些业务方法的实现完全相同，可以将这些方法移至抽象状态类，实现代码的复用，典型的具体状态类代码如下所示：

{% highlight java %}
class ConcreteState extends State {
    public void handle() {
        // 方法具体实现代码
    }
}
{% endhighlight %}

环境类维持一个对抽象状态类的引用，通过`setState()`方法可以向环境类注入不同的状态对象，再在环境类的业务方法中调用状态对象的方法，典型代码如下所示：

{% highlight java %}
class Context {
    private State state; // 维持一个对抽象状态对象的引用
    private int value; // 其他属性值，该属性值的变化可能会导致对象状态发生变化

    // 设置状态对象
    public void setState(State state) {
        this.state = state;
    }

    public void request() {
        // 其他代码
        state.handle(); // 调用状态对象的业务方法
        // 其他代码
    }
}
{% endhighlight %}

环境类实际上是真正拥有状态的对象，我们只是将环境类中与状态有关的代码提取出来封装到专门的状态类中。在状态模式结构图中，环境类`Context`与抽象状态类`State`之间存在单向关联关系，在`Context`中定义了一个`State`对象。在实际使用中，它们之间可能存在更为复杂的关系，`State`与`Context`之间可能也存在依赖或者关联关系。

<a id="效果"></a>
#效果
---
状态模式的主要优点如下：

- **封装了状态的转换规则**；在状态模式中可以将状态的转换代码封装在环境类或者具体状态类中，可以对状态转换代码进行集中管理，而不是分散在一个个业务方法中。
- **所有与状态相关的行为都放到了一个类中**；只需要注入一个不同的状态对象即可使环境对象拥有不同的行为。
- **允许状态转换逻辑与状态对象合成一体，而不是提供一个巨大的条件语句块**；状态模式可以让我们避免使用庞大的条件语句来将业务方法和状态转换代码交织在一起。
- **多个环境对象可以共享一个状态对象**；从而减少系统中对象的个数

状态模式的主要缺点如下：

- **状态模式对“开闭原则”支持并不太好**；增加新的状态类需要修改那些负责状态转换的源代码，否则无法转换到新增状态；而且修改某个状态类的行为也需要修改对应类的源代码。


<a id="实现"></a>
#实现
---
在有些情况下，多个环境对象可能需要共享同一个状态，**如果希望在系统中实现多个环境对象共享一个或多个状态对象，那么需要将这些状态对象定义为环境类的静态成员对象**。
参考[小试身手——开关的设计：实现共享状态](#小试身手_2)


在状态模式的使用过程中，一个对象的状态之间还可以进行相互转换，通常有两种实现状态转换的方式：
一、统一由环境类来负责状态之间的转换

此时，环境类还充当了**状态管理器（State Manager）**角色，在环境类的业务方法中通过对某些属性值的判断实现状态转换，还可以提供一个专门的方法用于实现属性判断的状态转换，如下代码片段所示：

{% highlight java %}
public void changeState() {
    if (value == 0) { // 判断属性值，根据属性值进行状态转换
        this.setState(new ConcreteStateA());
    } else if (value == 1) {
        this.setState(new ConcreteStateB());
    }
    .....
}
{% endhighlight %}
参考[小试身手——屏幕放大镜工具：由环境类来实现状态转换](#小试身手_3)

二、由具体状态类来负责状态之间的转换

可以在具体状态类的业务方法中判断环境类的某些属性值再根据情况为环境类设置新的状态对象，实现状态转换。此时，状态类与环境类之间就将存在依赖或关联关系，因为状态类需要访问环境类中的属性值，如下代码片段所示：

{% highlight java %}
public void changeState(Context context) {
    if (context.getValue() == 1) { // 根据环境对象中的属性值进行状态转换
        context.setState(new ConcreteStateA());
    } else if (context.getValue() == 2) {
        context.setState(new ConcreteStateB());
    }
    .....
}
{% endhighlight %}
参考[小试身手——银行系统中的账户类的设计：由具体状态类来实现状态转换](#小试身手_1)


<a id="小试身手"></a>
#小试身手
---
<a id="小试身手_1"></a>
####一、银行系统中的账户类的设计：由具体状态类来实现状态转换
> Sunny软件公司欲为某银行开发一套信用卡业务系统，银行账户(Account)是该系统的核心类之一，通过分析，Sunny软件公司开发人员发现在该系统中，账户存在三种状态，且在不同状态下账户存在不同的行为，具体说明如下：
> 
> 1. 如果账户中余额大于等于0，则账户的状态为正常状态(Normal State)，此时用户既可以向该账户存款也可以从该账户取款；
> 2. 如果账户中余额小于0，并且大于-2000，则账户的状态为透支状态(Overdraft State)，此时用户既可以向该账户存款也可以从该账户取款，但需要按天计算利息；
> 3. 如果账户中余额等于-2000，那么账户的状态为受限状态(Restricted State)，此时用户只能向该账户存款，不能再从中取款，同时也将按天计算利息；
> 4. 根据余额的不同，以上三种状态可发生相互转换。

Sunny软件公司开发人员对银行账户类进行分析，绘制了如图2所示UML状态图：
![bank_account_activity](/media/files/pattern/state/bank_account_activity.png)
<div align="center">图2 银行账户状态图</div>

在图2中，`NormalState`表示正常状态，`OverdraftState`表示透支状态，`RestrictedState`表示受限状态，在这三种状态下账户对象拥有不同的行为，方法`deposit()`用于存款，`withdraw()`用于取款，`computeInterest()`用于计算利息，`stateCheck()`用于在每一次执行存款和取款操作后根据余额来判断是否要进行状态转换并实现状态转换，相同的方法在不同的状态中可能会有不同的实现。为了实现不同状态下对象的各种行为以及对象状态之间的相互转换，Sunny软件公司开发人员设计了一个较为庞大的账户类`Account`，其中部分代码如下所示：

{% highlight java %}
class Account {
    private String state; // 状态
    private int balance; // 余额
    .....
    
    // 存款操作
    public void deposit() {
        // 存款
        stateCheck();
    }
    
    // 取款操作
    public void withdraw() {
        if (state.equalsIgnoreCase("NormalState") || state.equalsIgnoreCase("OverdraftState")) {
            // 取款
            stateCheck();
        } else {
            // 取款受限
        }
    }
    
    // 计算利息操作
    public void computeInterest() {
        if (state.equalsIgoreCase("OverdraftState") || state.equalsIgnoreCase("RestrictedState")) {
            // 计算利息
        }
    }

    //状态检查和转换操作
    public void stateCheck() {
        if (balance >= 0) {
            state = "NormalState";
        }
        else if (balance > -2000 && balance < 0) {
            state = "OverdraftState";
        }
        else if (balance == -2000) {
            state = "RestrictedState";
        }
        else if (balance < -2000) {
            //操作受限
        }
    }
    .....
}
{% endhighlight %}

分析上述代码，我们不难发现存在如下几个问题：

1. 几乎每个方法中都包含状态判断语句，以判断在该状态下是否具有该方法以及在特定状态下该方法如何实现，导致代码非常冗长，可维护性较差；
2. 拥有一个较为复杂的`stateCheck()`方法，包含大量的if…else if…else…语句用于进行状态转换，代码测试难度较大，且不易于维护；
3. 系统扩展性较差，如果需要增加一种新的状态，如冻结状态`FrozenState`，在该状态下既不允许存款也不允许取款），需要对原有代码进行大量修改，扩展起来非常麻烦；
4. 为了解决这些问题，我们可以使用状态模式，在状态模式中，我们将对象在每一个状态下的行为和状态转移语句封装在一个个状态类中，通过这些状态类来分散冗长的条件转移语句，让系统具有更好的灵活性和可扩展性，**状态模式可以在一定程度上解决上述问题**。

**解决方案**

Sunny软件公司开发人员使用状态模式来解决账户状态的转换问题，客户端只需要执行简单的存款和取款操作，系统根据余额将自动转换到相应的状态，其基本结构如图3所示：

![bank_account](/media/files/pattern/state/bank_account.png)
<div align="center">图3 银行账户结构图</div>

在图3中，`Account`充当环境类角色，`AccountState`充当抽象状态角色，`NormalState`、`OverdraftState`和`RestrictedState`充当具体状态角色。
参考代码托管在Github：[Bank Account](https://github.com/2dxgujun/java-design-patterns/tree/master/src/behavioral/state/bankaccount)

<a id="小试身手_2"></a>
####二、开关的设计：实现共享状态
> 如果某系统要求两个开关对象要么都处于开的状态，要么都处于关的状态，在使用时它们的状态必须保持一致，开关可以由开转换到关，也可以由关转换到开。

可以使用状态模式来实现开关的设计，其结构如图4所示：
![switch](/media/files/pattern/state/switch.png)
<div align="center">图4 开关及其状态设计结构图</div>

参考代码托管在Github：[Switch](https://github.com/2dxgujun/java-design-patterns/tree/master/src/behavioral/state/switch1)

<a id="小试身手_3"></a>
####三、屏幕放大镜工具：由环境类来实现状态转换
下面通过一个包含**循环状态**的简单实例来说明如何使用环境类实现状态转换：

> Sunny软件公司某开发人员欲开发一个屏幕放大镜工具，其具体功能描述如下：用户单击“放大镜”按钮之后屏幕将放大一倍，再点击一次“放大镜”按钮屏幕再放大一倍，第三次点击该按钮后屏幕将还原到默认大小。

可以考虑使用状态模式来设计该屏幕放大镜工具，我们定义三个屏幕状态类`NormalState`、`LargerState`和`LargestState`来对应屏幕的三种状态，分别是正常状态、二倍放大状态和四倍放大状态，屏幕类`Screen`充当环境类，其结构如图5所示：

![screen](/media/files/pattern/state/screen.png)
<div align="center">图5 屏幕放大镜工具结构图</div>

参考代码托管在Github：[Screen](https://github.com/2dxgujun/java-design-patterns/tree/master/src/behavioral/state/screen)


<br/>
参考：

1. [史上最强设计模式导学目录](http://blog.csdn.net/lovelion/article/details/17517213)
2. 《设计模式——可复用面向对象软件的基础》

本文出自[2dxgujun](http://github.com/2dxgujun)，转载时请注明出处及相应链接。