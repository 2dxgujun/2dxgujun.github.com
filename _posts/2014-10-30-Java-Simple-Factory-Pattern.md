---
layout: post
title: Java简单（Simple Factory）工厂模式
category: Pattern
date: 2014-10-30
---

# 定义

定义一个工厂类，它可以根据参数的不同返回不同类的实例，被创建的实例通常都具有共同的父类。因为在简单工厂模式中用于创建实例的方法是静态的，因此简单工厂模式又被称为**静态工厂（Static Factory Method）模式**；它属于类创建型模式。

# 概述

工厂模式是最常用的一类创建型设计模式，通常我们所说的工厂模式是指工厂方法模式，它也是使用频率最高的工厂模式。本章将要学习的简单工厂模式是工厂方法模式的“小弟”，它不属于GoF 23种设计模式，但在软件开发中应用也较为频繁，通常将它作为学习其他工厂模式的入门。此外，工厂方法模式还有一位“大哥”——抽象工厂模式。这三种工厂模式各具特色，难度也逐个加大，在软件开发中它们都得到了广泛的应用，成为面向对象软件中常用的创建对象的工具。

<!-- more -->

简单工厂模式的设计思想很简单，其**基本流程**如下：

首先将需要创建的各种不同对象的相关代码封装到不同的类中，这些类称为具体产品类，而将它们公共的代码进行抽象和提取后封装在一个抽象产品类中，每一个具体产品类都是抽象产品类的子类；然后提供一个工厂类用于创建各种产品，在工厂类中提供一个创建产品的工厂方法，该方法可以根据所传入的参数不同创建不同的具体产品对象；客户端只需调用工厂类的工厂方法并传入相应的参数即可得到一个产品对象。

简单工厂模式的要点在于：**当你需要什么，只需要传入一个正确的参数，就可以获取你所需要的对象，而无须知道其创建细节**。

# 适用性

在以下情况下可以考虑使用简单工厂模式：

1. 工厂类负责创建的对象比较少，由于创建的对象较少，不会造成工厂方法中的业务逻辑太过复杂。
2. 客户端只知道传入工厂类的参数，对于如何创建对象并不关心。

# 结构

简单工厂模式结构比较简单，其核心是工厂类的设计，其结构如图1所示：
![structure](/media/pattern/simple factory/structure.png)
<div align="center">图1 简单工厂模式结构图</div>

# 角色

**Factory（工厂角色）**
工厂角色即工厂类，它是简单工厂模式的核心，负责实现创建所有产品实例的内部逻辑；工厂类可以被外界直接调用，创建所需的产品对象；在工厂类中提供了静态的工厂方法`factoryMethod()`，它的返回类型为抽象产品类型`Product`。

**Product（抽象产品角色）**
它是工厂类所创建的所有对象的父类，封装了各种产品对象的公有方法，它的引入将提高系统的灵活性，使得在工厂类中只需定义一个通用的工厂方法，因为所有创建的具体产品对象都是其子类对象。

**ConcreteProduct（具体产品角色）**
它是简单工厂模式的创建目标，所有被创建的对象都充当这个角色的某个具体类的实例。每一个具体产品角色都继承了抽象产品角色，需要实现在抽象产品中声明的抽象方法。

# 协作

在简单工厂模式中，客户端通过工厂类来创建一个产品类的实例，而无须直接使用new关键字来创建对象，它是工厂模式家族中最简单的一员。

在使用简单工厂模式时，首先需要对产品类进行重构，**不能设计一个包罗万象的产品类，而需根据实际情况设计一个产品层次结构**，将所有产品类公共的代码移至抽象产品类，并在抽象产品类中声明一些抽象方法，以供不同的具体产品类来实现，典型的抽象产品类代码如下所示：

{% highlight java %}
public abstract class Product {
    // 所有产品类的公共业务方法
    public void methodSame() {
        // 公共方法的实现
    }
    
    // 声明抽象业务方法
    public abstract void methodDiff();
}
{% endhighlight %}

在具体产品类中实现了抽象产品类中声明的抽象业务方法，不同的具体产品类可以提供不同的实现，典型的具体产品类代码如下所示：

{% highlight java %}
public class ConcreteProduct extends Product {
    // 实现业务方法
    public void methodDiff() {
        // 业务方法的实现
    }
}
{% endhighlight %}

简单工厂模式的核心是工厂类，在没有工厂类之前，客户端一般会使用new关键字来直接创建产品对象，而在引入工厂类之后，客户端可以通过工厂类来创建产品，在简单工厂模式中，工厂类提供了一个静态工厂方法供客户端使用，根据所传入的参数不同可以创建不同的产品对象，典型的工厂类代码如下所示：

{% highlight java %}
public class Factory {
    public static Product getProduct(String arg) {
        Product product = null;
        if (arg.equalsIgnoreCase("A")) {
            product = new ConcreteProductA();
            // 初始化设置product
        }
        else if (arg.equalsIgnoreCase("B")) {
            product = new ConcreteProductB();
            // 初始化设置product
        }
        return product;
    }
}
{% endhighlight %}

在客户端代码中，我们通过调用工厂类的工厂方法即可得到产品对象，典型代码如下所示：

{% highlight java %}
public class Client {
    public static void main(String args[]) {
        Product product;
        product = Factory.getProduct("A"); // 通过工厂类创建产品对象
        product.methodSame();
        product.methodDiff();
    }
}
{% endhighlight %}

# 效果

简单工厂模式提供了专门的工厂类用于创建对象，将对象的创建和对象的使用分离开，它作为一种最简单的工厂模式在软件开发中得到了较为广泛的应用。

简单工厂模式主要优点如下：

- 工厂类包含必要的判断逻辑，可以决定在什么时候创建哪一个产品类的实例，**客户端可以免除直接创建产品对象的职责，而仅仅“消费”产品，简单工厂模式实现了对象创建和使用的分离**。
- 客户端无须知道所创建的具体产品类的类名，只需要知道具体产品类所对应的参数即可，对于一些复杂的类名，通过简单工厂模式可以在一定程度减少使用者的记忆量。
- **通过引入配置文件，可以在不修改任何客户端代码的情况下更换和增加新的具体产品类**，在一定程度上提高了系统的灵活性。

简单工厂模式主要缺点如下：

- 由于工厂类集中了所有产品的创建逻辑，**职责过重**，一旦不能正常工作，整个系统都要受到影响。
- 使用简单工厂模式势必会增加系统中类的个数（引入了新的工厂类），增加了系统的复杂度和理解难度。
- 简单工厂模式由于使用了静态工厂方法，**造成工厂角色无法形成基于继承的等级结构**。

# 实例：图表库的设计

> Sunny软件公司欲基于Java语言开发一套图表库，该图表库可以为应用系统提供各种不同外观的图表，例如柱状图、饼状图、折线图等。Sunny软件公司图表库设计人员希望为应用系统开发人员提供一套灵活易用的图表库，而且可以较为方便地对图表库进行扩展，以便能够在将来增加一些新类型的图表。

Sunny软件公司图表库设计人员提出了一个初始设计方案，将所有图表的实现代码封装在一个`Chart`类中，其框架代码如下所示：

{% highlight java %}
public class Chart {
    private String type; // 图表类型
    
    public Chart(Object[][] data, String type) {
        this.type = type;
        if (type.equalsIgnoreCase("histogram")) {
            // 初始化柱状图
        } else if (type.equalsIgnoreCase("pie")) {
            // 初始化饼状图
        } else if (type.equalsIgnoreCase("line")) {
            // 初始化折线图
        }
    }
    
    public void display() {
        if (type.equalsIgnoreCase("histogram")) {
            // 显示柱状图
        } else if (type.equalsIgnoreCase("pie")) {
            // 显示饼状图
        } else if (type.equalsIgnoreCase("line")) {
            // 显示折线图
        }
    }
}
{% endhighlight %}

客户端代码通过调用`Chart`类的构造函数来创建图表对象，根据参数`type`的不同可以得到不同类型的图表，然后再调用`display()`方法来显示相应的图表。

不难看出，`Chart`类是一个“巨大的”类，在该类的设计中存在如下几个问题：

1. 在`Chart`类中包含很多“if…else…”代码块，整个类的代码相当冗长，代码越长，阅读难度、维护难度和测试难度也越大；而且大量条件语句的存在还将影响系统的性能，程序在执行过程中需要做大量的条件判断。
2. `Chart`类的职责过重，它负责初始化和显示所有的图表对象，将各种图表对象的初始化代码和显示代码集中在一个类中实现，违反了“单一职责原则”，不利于类的重用和维护；而且将大量的对象初始化代码都写在构造函数中将导致构造函数非常庞大，对象在创建时需要进行条件判断，降低了对象创建的效率。
3. 当需要增加新类型的图表时，必须修改`Chart`类的源代码，违反了“开闭原则”。
4. 客户端只能通过new关键字来直接创建`Chart`对象，`Chart`类与客户端类耦合度较高，对象的创建和使用无法分离。
5. 客户端在创建`Chart`对象之前可能还需要进行大量初始化设置，例如设置柱状图的颜色、高度等，如果在`Chart`类的构造函数中没有提供一个默认设置，那就只能由客户端来完成初始设置，这些代码在每次创建`Chart`对象时都会出现，导致代码的重复。

面对一个如此巨大、职责如此重，且与客户端代码耦合度非常高的类，我们应该怎么办？本章介绍的简单工厂模式**将在一定程度上解决上述问题**。

**解决方案**

为了将`Chart`类的职责分离，同时将`Chart`对象的创建和使用分离，Sunny软件公司开发人员决定使用简单工厂模式对图表库进行重构，重构后的结构如图2所示：
![chart](/media/pattern/simple factory/chart.png)
<div align="center">图2 图表库结构图</div>

在图2中，`Chart`接口充当抽象产品类，其子类`HistogramChart`、`PieChart`和`LineChart`充当具体产品类，`ChartFactory`充当工厂类。

工厂类代码如下：
{% highlight java %}
public class ChartFactory {
    // 静态工厂方法
    public static Chart getChart(String type) {
        Chart chart;
        if (type.equalsIgnoreCase("histogram")) {
            chart = new HistogramChart();
        } else if (type.equalsIgnoreCase("pie")) {
            chart = new PieChart();
        } else if (type.equalsIgnoreCase("line")) {
            chart - new LineChart();
        }
        return chart;
    }
}
{% endhighlight %}
你只需要调用`getChart()`方法传入正确的参数，工厂就会给你创建一个对应的图表对象。

## 方案的改进

Sunny软件公司开发人员发现在创建具体`Chart`对象时，每更换一个`Chart`对象都需要修改客户端代码中静态工厂方法的参数，客户端代码将要重新编译，这对于客户端而言，违反了“开闭原则”，有没有一种方法能够在不修改客户端代码的前提下更换具体产品对象呢？答案是肯定的，下面将介绍一种常用的实现方式。

我们可以将静态工厂方法的参数存储在XML或properties格式的配置文件中，如下config.xml所示：
{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<config>
    <chartType>histogram</chartType>
</config>
{% endhighlight %}

再通过一个工具类`XMLUtil`来读取配置文件中的字符串参数，`XMLUtil`类的代码如下所示：
{% highlight java %}
public class XMLUtil {
    /** 
	 * 该方法用于从XML配置文件中提取图标类型，并返回类型名
	 */
	public static String getChartType() {
		try {
			DocumentBuilderFactory dFactory = DocumentBuilderFactory
					.newInstance();
			DocumentBuilder builder = dFactory.newDocumentBuilder();
			Document document = builder.parse(new File(
					"src/creational/simplefactory/chart/config.xml"));

			NodeList nList = document.getElementsByTagName("chartType");
			Node classNode = nList.item(0).getFirstChild();
			String chartType = classNode.getNodeValue().trim();
			
			return chartType;
		} catch (Exception e) {
			e.printStackTrace();
			return null;
		}
	}
}
{% endhighlight %}


在引入了配置文件和工具类`XMLUtil`之后，客户端代码修改如下：
{% highlight java %}
public class Client {
    public static void main(String args[]) {
        Chart chart;
        String type = XMLUtil.getChartType(); // 读取配置文件中的参数
        chart = ChartFactory.getChart(type); // 创建产品对象
        chart.display();
    }
}
{% endhighlight %}

不难发现，在上述客户端代码中不包含任何与具体图表对象相关的信息，如果需要更换具体图表对象，只需修改配置文件config.xml，无须修改任何源代码，符合“开闭原则”。

## 再次改进

Sunny软件公司开发人员发现在新增具体`Chart`类型时，需要修改工厂类中的工厂方法，客户端代码将要重新编译，这对于客户端而言，违反了“开闭原则”，下面将介绍一种常用的实现方式。

我们可以将系统中的具体产品以key-value的形式存储在XML或properties格式的配置文件中，key为静态工厂方法的参数，value为该具体产品的完整类名，如下properties.xml所示：
{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
    <entry key="histogram">creational.simplefactory.chart.HistogramChart</entry>
    <entry key="pie">creational.simplefactory.chart.PieChart</entry>
    <entry key="Line">creational.simplefactory.chart.LineChart</entry>
</properties>
{% endhighlight %}

然后我们需要修改工厂方法的逻辑，从配置文件中读取传入参数对应的具体产品类的类名，然后利用Java的反射机制，生成这个具体产品对象，工厂类修改如下：
{% highlight java %}
public class ChartFactory {
	public static Chart getChart(String type) {
		Properties properties = null;
		Class<?> c = null;
		Object object = null;
		try {
			// 读取配置文件，在不修改代码的情况下实现具体产品类的修改
			properties = new Properties();
			properties.loadFromXML(new FileInputStream(
					"src/creational/simplefactory/chart/properties.xml"));

			// 利用反射机制创建具体产品类
			c = Class.forName(properties.getProperty(type));
			object = c.newInstance();
			return (Chart) object;
		} catch (Exception ex) {
			ex.printStackTrace();
			return null;
		}
	}
}
{% endhighlight %}

当系统中需要增加新的具体`Chart`类时，只需要在properties.xml中添加一条新的key-value记录，然后在config.xml中替换成新增的图表类型的key，客户端代码无须重新编译，符合“开闭原则”。

完整代码托管在Github：[Chart](https://github.com/2dxgujun/java-design-patterns/tree/master/src/creational/simplefactory/chart)

# 简单工厂模式的简化

有时候，为了简化简单工厂模式，我们可以将抽象产品类和工厂类合并，将静态工厂方法移至抽象产品类中，如图3所示：
![simplify](/media/pattern/simple factory/simplify.png)
<div align="center">图3 简化的简单工厂模式结构图</div>

在图3中，客户端可以通过产品父类的静态工厂方法，根据参数的不同创建不同类型的产品子类对象，这种做法在JDK等类库和框架中也广泛存在。

<br/>
参考：

1. [史上最强设计模式导学目录](http://blog.csdn.net/lovelion/article/details/17517213)
2. 《设计模式——可复用面向对象软件的基础》

本文出自[2dxgujun](/)，转载时请注明出处及相应链接。