---
layout: post
title: Java组合模式（Composite）
category: Design Pattern
date: 2014-11-12
---

#定义
---
组合多个对象形成树形结构以表示具有“整体—部分”关系的层次结构。组合模式对单个对象（即叶子对象）和组合对象（即容器对象）的使用具有一致性；组合模式又可以称为“整体—部分”(Part-Whole)模式，它是一种对象结构型模式。

#概述
---
对于树形结构，当容器对象（如文件夹）的某一个方法被调用时，将遍历整个树形结构，寻找也包含这个方法的成员对象（可以是容器对象，也可以是叶子对象）并调用执行，牵一而动百，其中使用了递归调用的机制来对整个结构进行处理。由于容器对象和叶子对象在功能上的区别，在使用这些对象的代码中必须有区别地对待容器对象和叶子对象，而实际上大多数情况下我们希望一致地处理它们，因为对于这些对象的区别对待将会使得程序非常复杂。组合模式为解决此类问题而诞生，它可以**让叶子对象和容器对象的使用具有一致性**。

#适用性
---
在以下情况下可以考虑使用组合模式：

1. 在具有整体和部分的层次结构中，希望通过一种方式忽略整体与部分的差异，客户端可以一致地对待它们。
2. 在一个使用面向对象语言开发的系统中需要处理一个树形结构。
3. 在一个系统中能够分离出叶子对象和容器对象，而且它们的类型不固定，需要增加一些新的类型。

#结构
---
在组合模式中引入了抽象构件类`Component`，它是所有容器类和叶子类的公共父类，客户端针对`Component`进行编程。组合模式结构如图1所示：

![structure](/media/files/pattern/composite/structure.png)
<div align="center">图1 组合模式结构图</div>

<!-- more -->

#角色
---
**Component（抽象构件）**
它可以是接口或抽象类，为叶子构件和容器构件对象声明接口，在该角色中可以包含所有子类共有行为的声明和实现。在抽象构件中定义了访问及管理它的子构件的方法，如增加子构件，删除子构件，获取子构件等。

**Leaf（叶子构件）**
它在组合结构中表示叶子节点对象，叶子节点没有子节点，它实现了在抽象构件中定义的行为。对于那些访问及管理子构件的方法，可以通过异常等方式进行处理。

**Composite（容器构件）**
它在组合结构中表示容器节点对象，容器节点包含子节点，其子节点可以是叶子节点，也可以是容器节点，它提供一个集合用于存储子节点，实现了在抽象构件中定义的行为，包括那些访问及管理子构件的方法，在其业务方法中可以递归调用其子节点的业务方法。

#协作
---
**组合模式的关键是定义了一个抽象构件类，它既可以代表叶子，又可以代表容器，而客户端针对该抽象构件类进行编程，无须知道它到底表示的是叶子还是容器，可以对其进行统一处理**。同时容器对象与抽象构件类之间还建立一个聚合关联关系，在容器对象中既可以包含叶子，也可以包含容器，以此实现递归组合，形成一个树形结构。

如果不使用组合模式，客户端代码将过多地依赖于容器对象复杂的内部实现结构，容器对象内部实现结构的变化将引起客户代码的频繁变化，带来了代码维护复杂、可扩展性差等弊端。组合模式的引入将在一定程度上解决这些问题。

下面通过简单的示例代码来分析组合模式的各个角色的用途和实现。对于组合模式中的抽象构件角色，其典型代码如下所示：

{% highlight java %}
public abstract class Component {
    public abstract void add(Component c); // 增加成员
    public abstract void remove(Component c); // 删除成员
    public abstract void Component getChild(int i); // 获取成员
    public abstract void operation(); // 业务方法
}
{% endhighlight %}

一般将抽象构件类设计为接口或抽象类，将所有子类共有方法的声明和实现放在抽象构件类中。对于客户端而言，将针对抽象构件编程，而无须关心其具体子类是容器构件还是叶子构件。

如果继承抽象构件的是叶子构件，则其典型代码如下所示：

{% highlight java %}
public class Leaf extends Component {
    public void add(Component c) {
        // 异常处理或错误提示
    }

    public void remove(Component c) {
        // 异常处理或错误提示
    }
    
    public void Component getChild(int i) {
        // 异常处理或错误提示
        return null;
    }
    
    public void operation() {
        // 叶子构件具体业务方法的实现
    }
}
{% endhighlight %}

作为抽象构件类的子类，在叶子构件中需要实现在抽象构件类中声明的所有方法，包括业务方法以及管理和访问子构件的方法，但是叶子构件不能再包含子构件，因此**在叶子构件中实现子构件管理和访问方法时需要提供异常处理或错误提示**；当然，这无疑会给叶子构件的实现带来麻烦。

如果继承抽象构件的是容器构件，则其典型代码如下所示：

{% highlight java %}
public class Composite extends Component {
    private List<Component> list = new ArrayList<Component>();
    
    public void add(Component c) {
        list.add(c);
    }
    
    public void remove(Component c) {
        list.remove(c);
    }
    
    public void operation() {
        // 容器构件具体业务方法的实现
        // 递归调用成员构件的业务方法
        for (Component c : list) {
            c.operation();
        }
    }
}
{% endhighlight %}

在容器构件中实现了在抽象构件中声明的所有方法，既包括业务方法，也包括用于访问和管理成员子构件的方法，如`add()`、`remove()`和`getChild()`等方法。需要注意的是在实现具体业务方法时，由于容器构件充当的是容器角色，包含成员构件，因此它将调用其成员构件的业务方法。**在组合模式结构中，由于容器构件中仍然可以包含容器构件，因此在对容器构件进行处理时需要使用递归算法**，即在容器构件的`operation()`方法中递归调用其成员构件的`operation()`方法。


#效果
---
组合模式使用面向对象的思想来实现树形结构的构建与处理，描述了如何将容器对象和叶子对象进行递归组合，实现简单，灵活性好。由于在软件开发中存在大量的树形结构，因此组合模式是一种使用频率较高的结构型设计模式，Java SE中的AWT和Swing包的设计就基于组合模式，在这些界面包中为用户提供了大量的容器构件（如`Container`）和成员构件（如`Checkbox`、`Button`和`TextComponent`等）。

组合模式的主要优点如下：

- 组合模式可以清楚地定义分层次的复杂对象，表示对象的全部或部分层次，它让客户端忽略了层次的差异，方便对整个层次结构进行控制。
- 客户端可以一致地使用一个组合结构或其中单个对象，不必关心处理的是单个对象还是整个组合结构，简化了客户端代码。
- 在组合模式中增加新的容器构件和叶子构件都很方便，无须对现有类库进行任何修改，符合“开闭原则”。
- 组合模式为树形结构的面向对象实现提供了一种灵活的解决方案，通过叶子对象和容器对象的递归组合，可以形成复杂的树形结构，但对树形结构的控制却非常简单。

组合模式的主要缺点如下：

- 在增加新构件时很难对容器中的构件类型进行限制。有时候我们希望一个容器中只能有某些特定类型的对象，例如在某个文件夹中只能包含文本文件，使用组合模式时，不能依赖类型系统来施加这些约束，因为它们都来自于相同的抽象层，在这种情况下，必须通过在运行时进行类型检查来实现，这个实现过程较为复杂。

#实例：设计杀毒软件的框架结构
---
> Sunny软件公司欲开发一个杀毒(AntiVirus)软件，该软件既可以对某个文件夹(Folder)杀毒，也可以对某个指定的文件(File)进行杀毒。该杀毒软件还可以根据各类文件的特点，为不同类型的文件提供不同的杀毒方式，例如图像文件(ImageFile)和文本文件(TextFile)的杀毒方式就有所差异。现需要提供该杀毒软件的整体框架设计方案。

在介绍Sunny公司开发人员提出的初始解决方案之前，我们先来分析一下操作系统中的文件目录结构，例如在Windows操作系统中，存在如图2所示目录结构：

![windows_directory](/media/files/pattern/composite/windows_directory.png)
<div align="center">图2 Windows目录结构</div>

图2可以简化为如图3所示树形目录结构：

![directory](/media/files/pattern/composite/directory.png)
<div align="center">图3 树形目录结构示意图</div>

我们可以看出，在图3中包含文件（灰色节点）和文件夹（白色节点）两类不同的元素，其中在文件夹中可以包含文件，还可以继续包含子文件夹，但是在文件中不能再包含子文件或者子文件夹。在此，我们可以称文件夹为容器(Container)，而不同类型的各种文件是其成员，也称为叶子(Leaf)，一个文件夹也可以作为另一个更大的文件夹的成员。如果我们现在要对某一个文件夹进行操作，如查找文件，那么需要对指定的文件夹进行遍历，如果存在子文件夹则打开其子文件夹继续遍历，如果是文件则判断之后返回查找结果。

Sunny软件公司的开发人员通过分析，决定使用面向对象的方式来实现对文件和文件夹的操作，定义了如下图像文件类`ImageFile`、文本文件类`TextFile`和文件夹类`Folder`：

{% highlight java %}
// 图像文件类
public class ImageFile {
    private String name;
    
    public ImageFile(String name) {
        this.name = name;
    }
    
    public void killVirus() {
        System.out.println("对图像文件'" + name + "'进行杀毒");
    }
}
{% endhighlight %}

{% highlight java %}
// 文本文件类
public class TextFile {
    private String name;
    
    public TextFile(String name) {
        this.name = name;
    }
    
    public void killVirus() {
        System.out.println("对文本文件'" + name + "'进行杀毒");
    }
}
{% endhighlight %}

{% highlight java %}
// 文件夹类
public class Folder {
    private String name;
    
    // 定义集合folderList，用于存储Folder类型的成员
    private List<Folder> folderList = new ArrayList<Folder>();
    // 定义集合imageList，用于存储ImageFile类型的成员    
    private List<ImageFile> imageList = new ArrayList<ImageFile>();
    // 定义集合textList，用于存储TextFile类型的成员
    private List<TextFile> textList = new ArrayList<TextFile>();
    
    public Folder(String name) {
        this.name = name;
    }
    
    // 增加新的ImageFile类型的成员
    public void addFolder(Folder f) {
        folderList.add(f);
    }
    
    // 增加新的ImageFile类型的成员
    public void addImageFile(ImageFile image) {
        imageList.add(image);
    }
    
    // 增加新的TextFile类型的成员
    public void addTextFile(TextFile text) {
        textList.add(text);
    }
    
    //需提供三个不同的方法removeFolder()、removeImageFile()和removeTextFile()来删除成员，代码省略  
  
    //需提供三个不同的方法getChildFolder(int i)、getChildImageFile(int i)和getChildTextFile(int i)来获取成员，代码省略  
    
    public void killVirus() {
        System.out.println("对文件夹'" + name + "'进行杀毒");
        
        // 如果是Folder类型的成员，递归调用Folder的killVirus()方法
        for (Folder folder : folderList) {
            folder.killVirus();
        }
        
        // 如果是ImageFile类型的成员，调用ImageFile的killVirus()方法
        for (ImageFile img : imageList) {
            img.killVirus();
        }
        
        // 如果是TextFile类型的成员，调用TextFile的killVirus()方法
        for (TextFile txt : textList) {
            txt.killVirus();
        }
    }
}
{% endhighlight %}

编写如下客户端测试代码进行测试：
{% highlight java %}
public class Client {
    public static void main(String args[]) {
        Folder folder1, folder2, folder3;
        folder1 = new Folder("我的文件");
        folder2 = new Folder("图像文件");
        folder3 = new Folder("文本文件");
        
        ImageFile img1, img2;
        img1 = new ImageFile("sun.jpg");
        img2 = new ImageFile("fun.png");
        
        TextFile txt1, txt2;
        txt1 = new TextFile("cpp.doc");
        txt2 = new TextFile("plan.txt");

        folder2.addImageFile(image1);
        folder2.addImageFile(image2);
        folder3.addTextFile(text1);
        folder3.addTextFile(text2);
        folder1.addFolder(folder2);
        folder1.addFolder(folder3);
        
        folder1.killVirus();
    }
}
{% endhighlight %}

编译并运行程序，输出结果如下：

> 对文件夹'我的文件'进行杀毒<br/>
> 对文件夹'图像文件'进行杀毒<br/>
> 对图像文件'sun.jpg'进行杀毒<br/>
> 对图像文件'fun.png'进行杀毒<br/>
> 对文件夹'文本文件'进行杀毒<br/>
> 对文本文件'cpp.doc'进行杀毒<br/>
> 对文本文件'plan.txt'进行杀毒<br/>


Sunny公司开发人员“成功”实现了杀毒软件的框架设计，但通过仔细分析，发现该设计方案存在如下问题：

1. 文件夹类`Folder`的设计和实现都非常复杂，需要定义多个集合存储不同类型的成员，而且需要针对不同的成员提供增加、删除和获取等管理和访问成员的方法，存在大量的冗余代码，系统维护较为困难；
2. 由于系统没有提供抽象层，客户端代码必须有区别地对待充当容器的文件夹`Folder`和充当叶子的`ImageFile`和T`extFile`，无法统一对它们进行处理；
3. 系统的灵活性和可扩展性差，如果需要增加新的类型的叶子和容器都需要对原有代码进行修改，例如如果需要在系统中增加一种新类型的视频文件`VideoFile`，则必须修改`Folder`类的源代码，否则无法在文件夹中添加视频文件。

面对以上问题，Sunny软件公司的开发人员该如何来解决？这就需要用到本章介绍的组合模式。

**解决方案**

为了让系统具有更好的灵活性和可扩展性，客户端可以一致地对待文件和文件夹，Sunny公司开发人员使用组合模式来进行杀毒软件的框架设计，其基本结构如图4所示：

![antivirus](/media/files/pattern/composite/antivirus.png)
<div align="center">图4 杀毒软件框架设计结构图</div>

在图4中， `AbstractFile`充当抽象构件类，`Folder`充当容器构件类，`ImageFile`、`TextFile`和`VideoFile`充当叶子构件类。

完整代码托管在Github：[Antivirus](https://github.com/2dxgujun/java-design-patterns/tree/master/src/structural/composite/antivirus)

由于在本实例中使用了组合模式，在抽象构件类中声明了所有方法，包括用于管理和访问子构件的方法，如`add()`方法和`remove()`方法等，因此在`ImageFile`等叶子构件类中实现这些方法时必须进行相应的异常处理或错误提示。在容器构件类`Folder`的`killVirus()`方法中将递归调用其成员对象的`killVirus()`方法，从而实现对整个树形结构的遍历。

在具体实现时，我们可以创建图形化界面让用户选择所需操作的根节点，无须修改源代码，符合“开闭原则”，客户端无须关心节点的层次结构，可以对所选节点进行统一处理，提高系统的灵活性。

#透明组合模式与安全组合模式
---
通过引入组合模式，Sunny公司设计的杀毒软件具有良好的可扩展性，在增加新的文件类型时，无须修改现有类库代码，只需增加一个新的文件类作为`AbstractFile`类的子类即可，但是由于在`AbstractFile`中声明了大量用于管理和访问成员构件的方法，例如`add()`、`remove()`等方法，我们不得不在新增的文件类中实现这些方法，提供对应的错误提示和异常处理。为了简化代码，我们有以下两个解决方案：

**解决方案一：**
将叶子构件的`add()`、`remove()`等方法的实现代码移至`AbstractFile`类中，由`AbstractFile`提供统一的默认实现，代码如下所示：

{% highlight java %}
// 提供默认实现的抽象构件类
public abstract class AbstractFile {
    public void add(AbstractFile file) {
        System.out.println("对不起，不支持该方法！");
    }
    
    public void remove(AbstractFile file) {
        System.out.println("对不起，不支持该方法！");
    }
    
    public AbstractFile getChild(int i) {
        System.out.println("对不起，不支持该方法！");
        return null;
    }
    
    public abstract void killVirus();
}
{% endhighlight %}

如果客户端代码针对抽象类`AbstractFile`编程，在调用文件对象的这些方法时将出现错误提示。如果不希望出现任何错误提示，我们可以在客户端定义文件对象时不使用抽象层，而直接使用具体叶子构件本身，客户端代码片段如下所示：

{% highlight java %}
public class Client {
    public static void main(String[] args) {
        // 不能透明处理叶子构件
        ImageFile file1, file2;
        TextFile file3, file4;
        VideoFile file5;
        AbstractFile folder1, folder2, folder3, folder4;
        // 其它代码省略
    }
}
{% endhighlight %}

这样就产生了一种不透明的使用方式，即在客户端不能全部针对抽象构件类编程，需要使用具体叶子构件类型来定义叶子对象。

**解决方案二：**
除此之外，还有一种解决方法是在抽象构件`AbstractFile`中不声明任何用于访问和管理成员构件的方法，代码如下所示：

{% highlight java %}
public abstract class AbstractFile {
    public abstract void killVirus();
}
{% endhighlight %}

此时，由于在`AbstractFile`中没有声明`add()`、`remove()`等访问和管理成员的方法，其叶子构件子类无须提供实现；而且无论客户端如何定义叶子构件对象都无法调用到这些方法，不需要做任何错误和异常处理，容器构件再根据需要增加访问和管理成员的方法，但这时候也存在一个问题：客户端不得不使用容器类本身来声明容器构件对象，否则无法访问其中新增的`add()`、`remove()`等方法，如果客户端一致性地对待叶子和容器，将会导致容器构件的新增对客户端不可见，客户端代码对于容器构件无法再使用抽象构件来定义，客户端代码片段如下所示：

{% highlight java %}
public class Client {
    public static void main(String[] args) {
        AbstractFile file1, file2, file3, file4, file5;
        Folder folder1, folder2, folder3, folder4; // 不能透明的处理容器构件
        // 其他代码省略
    }
}
{% endhighlight %}

在使用组合模式时，根据抽象构件类的定义形式，我们可将组合模式分为透明组合模式和安全组合模式两种形式：

##透明组合模式
透明组合模式中，抽象构件`Component`中声明了所有用于管理成员对象的方法，包括`add()`、`remove()`以及`getChild()`等方法，这样做的好处是确保所有的构件类都有相同的接口。在客户端看来，叶子对象与容器对象所提供的方法是一致的，客户端可以相同地对待所有的对象。透明组合模式也是组合模式的标准形式，虽然上面的**解决方案一**在客户端可以有不透明的实现方法，但是由于在抽象构件中包含`add()`、`remove()`等方法，因此它还是透明组合模式，透明组合模式的完整结构如图5所示：

![transparent_composite](/media/files/pattern/composite/transparent_composite.png)
<div align="center">图5 透明组合模式结构图</div>

透明组合模式的缺点是不够安全，因为叶子对象和容器对象在本质上是有区别的。叶子对象不可能有下一个层次的对象，即不可能包含成员对象，因此为其提供`add()`、`remove()`以及`getChild()`等方法是没有意义的，这在编译阶段不会出错，但在运行阶段如果调用这些方法可能会出错（如果没有提供相应的错误处理代码）。

##安全组合模式
安全组合模式中，在抽象构件`Component`中没有声明任何用于管理成员对象的方法，而是在`Composite`类中声明并实现这些方法。这种做法是安全的，因为根本不向叶子对象提供这些管理成员对象的方法，对于叶子对象，客户端不可能调用到这些方法，这就是**解决方案二**所采用的实现方式。安全组合模式的结构如图6所示：

![safe_composite](/media/files/pattern/composite/safe_composite.png)
<div align="center">图6 安全组合模式结构图</div>

安全组合模式的缺点是不够透明，因为叶子构件和容器构件具有不同的方法，且容器构件中那些用于管理成员对象的方法没有在抽象构件类中定义，因此客户端不能完全针对抽象编程，必须有区别地对待叶子构件和容器构件。在实际应用中，安全组合模式的使用频率也非常高，在Java AWT中使用的组合模式就是安全组合模式。


<br/>
参考：

1. [史上最强设计模式导学目录](http://blog.csdn.net/lovelion/article/details/17517213)
2. 《设计模式——可复用面向对象软件的基础》

本文出自[2dxgujun](http://2dxgujun.com/)，转载时请注明出处及相应链接。