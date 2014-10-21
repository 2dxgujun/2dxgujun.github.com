---
layout: post
title: Java工厂方法模式（Factory Method Pattern）——解决简单工厂模式的弊端
category: Design Pattern
date: 2014-10-06
---

#定义
---
定义一个用于创建对象的接口，让子类决定实例化哪个类。工厂方法模式让一个类的实例化延迟到其子类。工厂方法模式又被称为**虚构造器（Virtual Constructor）模式**或**多态工厂（Polymorphic Factory）模式**；工厂方法模式是一种类创建型模式。

#概述
---
简单工厂模式的最大的缺点是当有新产品要加入到系统中时，必须修改工厂类，需要在其中加入必要的业务逻辑，这违背了“开闭原则”。此外，在简单工厂模式中，所有的产品都由同一个工厂创建，工厂类职责较重，业务逻辑较为复杂，具体产品与工厂类之间的耦合度高，严重影响了系统的灵活性和扩展性，而工厂方法模式则可以很好地解决这一问题。

在工厂方法模式中**不再提供一个统一的工厂类来创建所有的产品对象，而是针对不同的产品提供不同的工厂，系统提供一个与产品等级结构对应的工厂等级结构**。

#适用性
---
以下情况可以考虑使用工厂方法模式：

1. 客户端不知道它所需要的对象的类，只需知道所对应的工厂。

#结构
---
工厂方法模式提供一个抽象工厂类来声明抽象工厂方法，而由其子类来具体实现工厂方法，创建具体的产品对象。工厂方法模式结构如图1所示：
![structure](/media/files/pattern/factory method/structure.png)
<div align="center">图1 工厂方法模式结构图</div>

<!-- more -->

#角色
---
**Product（抽象产品）**
它是定义产品的接口，是工厂方法模式所创建对象的超类型，也就是产品对象的公共父类。

**ConcreteProduct（具体产品）**
它实现了抽象产品接口，由专门的具体工厂创建，和具体工厂之间一一对应。

**Factory（抽象工厂）**
在抽象工厂类中，声明了工厂方法，用于返回一个产品；抽象工厂是工厂方法模式的核心，所有创建对象的工厂类都必须实现该接口。

**ConcreteFactory（具体工厂）**
它是抽象工厂类的子类，实现了抽象工厂中定义的工厂方法，并由客户端调用，返回一个具体产品类的实例。

#协作
---
与简单工厂模式相比，工厂方法模式最重要的区别是引入了抽象工厂角色，抽象工厂可以是接口，也可以是抽象类或者具体类，其典型代码如下所示：

{% highlight java %}
public interface Factory {
    public Product factoryMethod();
}
{% endhighlight %}

在抽象工厂中声明了工厂方法但并未实现工厂方法，具体产品对象的创建由其子类负责，客户端针对抽象工厂编程，可在运行时再指定具体工厂类，具体工厂类实现了工厂方法，不同的具体工厂可以创建不同的具体产品，其典型代码如下所示：

{% highlight java %}
public class ConcreteFactory implements Factory {  
    public Product factoryMethod() {  
        return new ConcreteProduct();  
    }  
}  
{% endhighlight %}

在实际使用时，具体工厂类在实现工厂方法时除了创建具体产品对象之外，还可以负责产品对象的初始化工作以及一些资源和环境配置工作，例如连接数据库、创建文件等。

在客户端代码中，只需关心工厂类既可，不同的具体工厂可以创建不同的产品，典型的客户端类代码片段如下：

{% highlight java %}
Factory factory;
factory = new ConcreteFactory(); // 可通过配置文件实现
Product product;
product = factory.factoryMethod();
{% endhighlight %}

**可以通过配置文件来存储具体工厂类ConcreteFactory的类名，更换新的具体工厂时无须修改源代码**，系统扩展更为方便。

#效果
---
工厂方法模式是简单工厂模式的延伸，它继承了简单工厂模式的优点，同时还弥补了简单工厂模式的不足。

工厂方法模式主要优点如下：

- **分离了产品的创建和使用**；工厂方法用来创建客户所需要的产品，同时还向客户隐藏了哪种具体产品类将被实例化这一细节；用户只需要关心所需产品对应的工厂，无须关心创建细节，甚至无须知道具体产品类的类名。
- **系统扩展性良好**；在系统中加入新产品时，无须修改抽象工厂和抽象产品提供的接口，无须修改客户端，而只需要添加一个具体工厂和具体产品就可以了，完全符合“开闭原则”。

工厂方法模式主要缺点如下：

- **系统较复杂**；在添加新产品时，需要编写新的具体产品类，而且还要提供与之对应的具体工厂类，系统中类的个数将成对增加， 在一定程度上增加了系统的复杂度，带来额外的开销。
- **实现难度较大**；考虑到系统的可扩展性，需要引入抽象层，在客户端代码中均使用抽象层进行定义，增加了系统的抽象性和理解难度，且在实现时可能需要用到DOM、反射等技术，增加了系统的实现难度。

#实现
---
在工厂方法模式的使用过程中，发现需要通过多种方式来初始化一个产品。此时，可以**提供一组重载的工厂方法，以不同的方式创建产品对象**。

有些情况下，我们为了简化客户端的使用，还可以**对客户端隐藏工厂方法，在工厂类中将直接调用产品类的业务方法，客户端无须调用工厂方法创建产品，直接通过工厂即可使用所创建的对象中的业务方法**。

参考小试身手中的日志记录器的设计。

#小试身手
---
###一、日志记录器的设计
> Sunny软件公司欲开发一个系统运行日志记录器（Logger），该记录器可以通过多种途径保存系统的运行日志，如通过文件记录或数据库记录，用户可以通过修改配置文件灵活地更换日志记录方式。在设计各类日志记录器时，Sunny公司的开发人员发现需要对日志记录器进行一些初始化工作，初始化参数的设置过程较为复杂，而且某些参数的设置有严格的先后次序，否则可能会发生记录失败。如何封装记录器的初始化过程并保证多种记录器切换的灵活性是Sunny公司开发人员面临的一个难题。

Sunny公司的开发人员通过对该需求进行分析，发现该日志记录器有两个设计要点：

1. 需要封装日志记录器的初始化过程，这些初始化工作较为复杂，例如需要初始化其他相关的类，还有可能需要读取配置文件（例如连接数据库或创建文件），导致代码较长，如果将它们都写在构造函数中，会导致构造函数庞大，不利于代码的修改和维护；
2. 用户可能需要更换日志记录方式，在客户端代码中需要提供一种灵活的方式来选择日志记录器，尽量在不修改源代码的基础上更换或者增加新的日志记录方式。

Sunny公司开发人员最初使用简单工厂模式对日志记录器进行了设计，初始结构如图2所示：
![logger_simple_factory](/media/files/pattern/factory method/logger_simple_factory.png)
<div align="center">图2 基于简单工厂模式设计的日志记录器结构图</div>

在图2中，`LoggerFactory`充当创建日志记录器的工厂，提供了工厂方法`createLogger()`用于创建日志记录器，`Logger`是抽象日志记录器接口，其子类为具体日志记录器。其中，工厂类`LoggerFactory`代码片段如下所示：

{% highlight java %}
public class LoggerFactory {
    public static Logger createLogger(String args) {
        if (args.equalsIgnoreCase("db")) {
            // TODO 连接数据库
            Logger logger = new DatabaseLogger();
            // TODO 初始化数据库日志记录器
            return logger;
        } else if (args.equalsIgnoreCase("file")) {
            // TODO 创建日志文件
            Logger logger = new FileLogger();
            // TODO 初始化文件日志记录器
            return logger;
        } else {
            return null;
        }
    }
}
{% endhighlight %}

为了突出设计重点，我们对上述代码进行了简化，省略了具体日志记录器的初始化代码。在`LoggerFactory`类中提供了静态工厂方法`createLogger()`，用于根据所传入的参数创建各种不同类型的日志记录器。通过使用简单工厂模式，我们将日志记录器对象的创建和使用分离，客户端只需使用由工厂类创建的日志记录器对象既可，无须关心对象的创建过程，但是我们发现，虽然简单工厂模式实现了对象的创建和使用分离，但是仍然存在如下两个问题：

1. 工厂类过于庞大，包含了大量的if...else...代码，导致维护和测试难度增大；
2. 系统扩展不灵活，如果增加新类型的日志记录器，必须修改静态工厂方法的业务逻辑，违反了“开闭原则”。

**解决方法**

Sunny公司开发人员决定使用工厂方法模式来设计日志记录器，其基本结构如图3所示：
![logger](/media/files/pattern/factory method/logger.png)
<div align="center">图3 基于工厂方法模式设计的日志记录器结构图</div>

在图3中，`Logger`接口充当抽象产品，其子类`FileLogger`和`DatabaseLogger`充当具体产品，`LoggerFactory`接口充当抽象工厂，其子类`FileLoggerFactory`和`DatabaseLoggerFactory`充当具体工厂。

为了让系统具有更好的灵活性和可扩展性，在客户端代码中将不再使用new关键字来创建工厂对象，而是将具体工厂类的类名存储在配置文件（如XML文件）中，通过读取配置文件获取类名字符串，再使用Java的反射机制，根据类名字符串生成对象。

完整代码托管在Github：[Logger](https://github.com/2dxgujun/java-design-patterns/tree/master/src/creational/factorymethod/logger)

###二、日志记录器的设计：重载的工厂方法
Sunny公司开发人员通过进一步分析，发现可以通过多种方式来初始化日志记录器，例如可以为各种日志记录器提供默认实现；还可以为数据库日志记录器提供数据库连接字符串，为文件日志记录器提供文件路径；也可以将参数封装在一个`Object`类型的对象中，通过`Object`对象将配置参数传入工厂类。此时，可以提供一组重载的工厂方法，以不同的方式对产品对象进行创建。当然，对于同一个具体工厂而言，无论使用哪个工厂方法，创建的产品类型均要相同。如图4所示：
![logger_overload](/media/files/pattern/factory method/logger_overload.png)
<div align="center">图4 重载工厂方法后的日志记录器结构图</div>

引入重载方法后，抽象工厂`LoggerFactory`的代码修改如下：

{% highlight java %}
public interface LoggerFactory {  
    public Logger createLogger();  
    public Logger createLogger(String args);  
    public Logger createLogger(Object obj);  
}
{% endhighlight %}

具体工厂类`DatabaseLoggerFactory`代码修改如下：

{% highlight java %}
public class DatabaseLoggerFactory implements LoggerFactory {  
    public Logger createLogger() {  
        // TODO 使用默认方式连接数据库
        Logger logger = new DatabaseLogger();   
        // TODO 初始化数据库日志记录器
        return logger;
    }  
  
    public Logger createLogger(String args) {  
        // TODO 使用参数args作为连接字符串来连接数据库
        Logger logger = new DatabaseLogger();   
        // TODO 初始化数据库日志记录器
        return logger;
    }     
  
    public Logger createLogger(Object obj) {  
        // TODO 使用封装在参数obj中的连接字符串来连接数据库
        Logger logger = new DatabaseLogger();   
        // TODO 使用封装在参数obj中的数据来初始化数据库日志记录器
        return logger;
    }
}
{% endhighlight %}

在抽象工厂中定义多个重载的工厂方法，在具体工厂中实现了这些工厂方法，这些方法可以包含不同的业务逻辑，以满足对不同产品对象的需求。

###三、日志记录器的设计：工厂方法的隐藏
如果对客户端隐藏工厂方法，日志记录器的结构图将修改为图5所示：
![logger_hide](/media/files/pattern/factory method/logger_hide.png)
<div align="center">图5 隐藏工厂方法后的日志记录器结构图</div>

在图5中，抽象工厂类`LoggerFactory`的代码修改如下：

{% highlight java %}
public abstract class LoggerFactory {  
    // 在工厂类中直接调用日志记录器类的业务方法writeLog()  
    public void writeLog() {  
        Logger logger = this.createLogger();  
        logger.writeLog();  
    }

    public abstract Logger createLogger();    
}
{% endhighlight %}

客户端代码修改如下：

{% highlight java %}
public class Client {
    public static void main(String args[]) {
        LoggerFactory factory;  
        factory = (LoggerFactory)XMLUtil.getBean();  
        factory.writeLog(); // 直接使用工厂对象来调用产品对象的业务方法  
    }  
}  
{% endhighlight %}

通过将业务方法的调用移入工厂类，可以直接使用工厂对象来调用产品对象的业务方法，客户端无须直接使用工厂方法，在某些情况下我们也可以使用这种设计方案。

<br/>
参考：

1. [史上最强设计模式导学目录](http://blog.csdn.net/lovelion/article/details/17517213)
2. 《设计模式——可复用面向对象软件的基础》

本文出自[2dxgujun](http://github.com/2dxgujun)，转载时请注明出处及相应链接。