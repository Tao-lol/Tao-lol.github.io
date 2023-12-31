---
title: 设计模式
tags:
  - 程序设计
categories:
  - 编程
date: 2019-08-30 10:43:20
---

> 简介：https://www.cnblogs.com/www-zsl187-com/p/8834734.html  
> 正文：https://www.cnblogs.com/zhili/p/DesignPatternSummery.html  

一、创建型模式  
1、抽象工厂模式(Abstract factory pattern): 提供一个接口, 用于创建相关或依赖对象的家族, 而不需要指定具体类.  
2、生成器模式(Builder pattern): 使用生成器模式封装一个产品的构造过程, 并允许按步骤构造. 将一个复杂对象的构建与它的表示分离, 使得同样的构建过程可以创建不同的表示.  
3、工厂模式(factory method pattern): 定义了一个创建对象的接口, 但由子类决定要实例化的类是哪一个. 工厂方法让类把实例化推迟到子类.  
4、原型模式(prototype pattern): 当创建给定类的实例过程很昂贵或很复杂时, 就使用原型模式.  
5、单例模式(Singleton pattern): 确保一个类只有一个实例, 并提供全局访问点.  
6、多例模式(Multition pattern): 在一个解决方案中结合两个或多个模式, 以解决一般或重复发生的问题.  
二、结构型模式  
1、适配器模式(Adapter pattern): 将一个类的接口, 转换成客户期望的另一个接口. 适配器让原本接口不兼容的类可以合作无间. 对象适配器使用组合, 类适配器使用多重继承.  
2、桥接模式(Bridge pattern): 使用桥接模式通过将实现和抽象放在两个不同的类层次中而使它们可以独立改变.  
3、组合模式(composite pattern): 允许你将对象组合成树形结构来表现"整体/部分"层次结构. 组合能让客户以一致的方式处理个别对象以及对象组合.  
4、装饰者模式(decorator pattern): 动态地将责任附加到对象上, 若要扩展功能, 装饰者提供了比继承更有弹性的替代方案.  
5、外观模式(facade pattern): 提供了一个统一的接口, 用来访问子系统中的一群接口. 外观定义了一个高层接口, 让子系统更容易使用.  
6、亨元模式(Flyweight Pattern): 如想让某个类的一个实例能用来提供许多"虚拟实例", 就使用蝇量模式.  
7、代理模式(Proxy pattern): 为另一个对象提供一个替身或占位符以控制对这个对象的访问.  
三、行为型模式  
1、责任链模式(Chain of responsibility pattern): 通过责任链模式, 你可以为某个请求创建一个对象链. 每个对象依序检查此请求并对其进行处理或者将它传给链中的下一个对象.  
2、命令模式(Command pattern): 将"请求"封闭成对象, 以便使用不同的请求,队列或者日志来参数化其他对象. 命令模式也支持可撤销的操作.  
3、解释器模式(Interpreter pattern): 使用解释器模式为语言创建解释器.  
4、迭代器模式(iterator pattern): 提供一种方法顺序访问一个聚合对象中的各个元素, 而又不暴露其内部的表示.  
5、中介者模式(Mediator pattern) : 使用中介者模式来集中相关对象之间复杂的沟通和控制方式.  
6、备忘录模式(Memento pattern): 当你需要让对象返回之前的状态时(例如, 你的用户请求"撤销"), 你使用备忘录模式.  
7、观察者模式(observer pattern): 在对象之间定义一对多的依赖, 这样一来, 当一个对象改变状态, 依赖它的对象都会收到通知, 并自动更新.  
8、状态模式(State pattern): 允许对象在内部状态改变时改变它的行为, 对象看起来好象改了它的类.  
9、策略模式(strategy pattern): 定义了算法族, 分别封闭起来, 让它们之间可以互相替换, 此模式让算法的变化独立于使用算法的客户.  
10、模板方法模式(Template pattern): 在一个方法中定义一个算法的骨架, 而将一些步骤延迟到子类中. 模板方法使得子类可以在不改变算法结构的情况下, 重新定义算法中的某些步骤.  
11、访问者模式(visitor pattern): 当你想要为一个对象的组合增加新的能力, 且封装并不重要时, 就使用访问者模式.  

<!--more-->

# 一、创建型模式  
&emsp;&emsp;创建型模式就是用来创建对象的模式，抽象了实例化的过程。所有的创建型模式都有两个共同点。第一，它们都将系统使用哪些具体类的信息封装起来；第二，它们隐藏了这些类的实例是如何被创建和组织的。创建型模式包括单例模式、工厂方法模式、抽象工厂模式、建造者模式和原型模式。  
* 单例模式：解决的是实例化对象的个数的问题，比如抽象工厂中的工厂、对象池等，除了Singleton之外，其他创建型模式解决的都是 new 所带来的耦合关系。  
* 抽象工厂：创建一系列相互依赖对象，并能在运行时改变系列。  
* 工厂方法：创建单个对象，在Abstract Factory有使用到。  
* 原型模式：通过拷贝原型来创建新的对象。  

&emsp;&emsp;工厂方法、抽象工厂、建造者都需要一个额外的工厂类来负责实例化“一个对象”，而Prototype则是通过原型（一个特殊的工厂类）来克隆“易变对象”。  
&emsp;&emsp;下面详细介绍下它们。  

## 1.1  单例模式  
&emsp;&emsp;单例模式指的是确保某一个类只有一个实例，并提供一个全局访问点。解决的是实体对象个数的问题，而其他的建造者模式都是解决new所带来的耦合关系问题。其实现要点有：  
* 类只有一个实例。问：如何保证呢？答：通过私有构造函数来保证类外部不能对类进行实例化  
* 提供一个全局的访问点。问：如何实现呢？答：创建一个返回该类对象的静态方法  

&emsp;&emsp;单例模式的结构图如下所示：  

![ ](单例模式.gif)
 
## 1.2 工厂方法模式  
&emsp;&emsp;工厂方法模式指的是定义一个创建对象的工厂接口，由其子类决定要实例化的类，将实际创建工作推迟到子类中。它强调的是”单个对象“的变化。其实现要点有：  
* 定义一个工厂接口。问：如何实现呢？答：声明一个工厂抽象类  
* 由其具体子类创建对象。问：如何去实现呢？答：创建派生于工厂抽象类，即由具体工厂去创建具体产品，既然要创建产品，自然需要产品抽象类和具体产品类了。  

&emsp;&emsp;其具体的UML结构图如下所示：  

![ ](工厂方法模式.png)

&emsp;&emsp;在工厂方法模式中，工厂类与具体产品类具有平行的等级结构，它们之间是一一对应关系。  

## 1.3 抽象工厂模式  
&emsp;&emsp;抽象工厂模式指的是提供一个创建一系列相关或相互依赖对象的接口，使得客户端可以在不必指定产品的具体类型的情况下，创建多个产品族中的产品对象，强调的是”系列对象“的变化。其实现要点有：  
* 提供一系列对象的接口。问：如何去实现呢？答：提供多个产品的抽象接口  
* 创建多个产品族中的多个产品对象。问：如何做到呢？答：每个具体工厂创建一个产品族中的多个产品对象，多个具体工厂就可以创建多个产品族中的多个对象了。  

&emsp;&emsp;具体的UML结构图如下所示：  

![ ](抽象工厂模式.gif)
 
## 1.4 建造者模式  
&emsp;&emsp;建造者模式指的是将一个产品的内部表示与产品的构造过程分割开来，从而可以使一个建造过程生成具体不同的内部表示的产品对象。强调的是产品的构造过程。其实现要点有：  
* 将产品的内部表示与产品的构造过程分割开来。问：如何把它们分割开呢？答：不要把产品的构造过程放在产品类中，而是由建造者类来负责构造过程，产品的内部表示放在产品类中，这样不就分割开了嘛。  

&emsp;&emsp;具体的UML结构图如下所示：  

![ ](建造者模式.png)

## 1.5 原型工厂模式  
&emsp;&emsp;原型模式指的是通过给出一个原型对象来指明所要创建的对象类型，然后用复制的方法来创建出更多的同类型对象。其实现要点有：  
* 给出一个原型对象。问：如何办到呢？答：很简单嘛，直接给出一个原型类就好了。  
* 通过复制的方法来创建同类型对象。问：又是如何实现呢？答：.NET可以直接调用MemberwiseClone方法来实现浅拷贝  

&emsp;&emsp;具体的UML结构图如下所示：  

![ ](原型工厂模式.png)

# 二、结构型模式  
&emsp;&emsp;结构型模式，顾名思义讨论的是类和对象的结构 ，主要用来处理类或对象的组合。它包括两种类型，一是类结构型模式，指的是采用继承机制来组合接口或实现；二是对象结构型模式，指的是通过组合对象的方式来实现新的功能。它包括适配器模式、桥接模式、装饰者模式、组合模式、外观模式、享元模式和代理模式。  
* 适配器模式注重转换接口，将不吻合的接口适配对接   
* 桥接模式注重分离接口与其实现，支持多维度变化   
* 组合模式注重统一接口，将“一对多”的关系转化为“一对一”的关系   
* 装饰者模式注重稳定接口，在此前提下为对象扩展功能   
* 外观模式注重简化接口，简化组件系统与外部客户程序的依赖关系   
* 享元模式注重保留接口，在内部使用共享技术对对象存储进行优化   
* 代理模式注重假借接口，增加间接层来实现灵活控制  

## 2.1 适配器模式  
&emsp;&emsp;适配器模式意在转换接口，它能够使原本不能在一起工作的两个类一起工作，所以经常用来在类库的复用、代码迁移等方面。例如DataAdapter类就应用了适配器模式。适配器模式包括类适配器模式和对象适配器模式，具体结构如下图所示，左边是类适配器模式，右边是对象适配器模式。  

![ ](适配器模式.jpg)

## 2.2 桥接模式  
&emsp;&emsp;桥接模式旨在将抽象化与实现化解耦，使得两者可以独立地变化。意思就是说，桥接模式把原来基类的实现化细节再进一步进行抽象，构造到一个实现化的结构中，然后再把原来的基类改造成一个抽象化的等级结构，这样就可以实现系统在多个维度的独立变化，桥接模式的结构图如下所示。  

![ ](桥接模式.gif)

## 2.3 装饰者模式  
&emsp;&emsp;装饰者模式又称包装（Wrapper）模式，它可以动态地给一个对象添加一些额外的功能，装饰者模式较继承生成子类的方式更加灵活。虽然装饰者模式能够动态地将职责附加到对象上，但它也会造成产生一些细小的对象，增加了系统的复杂度。具体的结构图如下所示。  

![ ](装饰者模式.png)

## 2.4 组合模式  
&emsp;&emsp;组合模式又称为部分—整体模式。组合模式将对象组合成树形结构，用来表示整体与部分的关系。组合模式使得客户端将单个对象和组合对象同等对待。如在.NET中WinForm中的控件，TextBox、Label等简单控件继承与Control类，同时GroupBox这样的组合控件也是继承于Control类。组合模式的具体结构图如下所示。  

![ ](组合模式.gif)

## 2.5 外观模式  
&emsp;&emsp;在系统中，客户端经常需要与多个子系统进行交互，这样导致客户端会随着子系统的变化而变化，此时可以使用外观模式把客户端与各个子系统解耦。外观模式指的是为子系统中的一组接口提供一个一致的门面，它提供了一个高层接口，这个接口使子系统更加容易使用。如电信的客户专员，你可以让客户专员来完成冲话费，修改套餐等业务，而不需要自己去与各个子系统进行交互。具体类结构图如下所示：  

![ ](外观模式.gif)

## 2.6 享元模式  
&emsp;&emsp;在系统中，如何我们需要重复使用某个对象时，此时如果重复地使用new操作符来创建这个对象的话，这对系统资源是一个极大的浪费，既然每次使用的都是同一个对象，为什么不能对其共享呢？这也是享元模式出现的原因。  
&emsp;&emsp;享元模式运用共享的技术有效地支持细粒度的对象，使其进行共享。在.NET类库中，String类的实现就使用了享元模式，String类采用字符串驻留池的来使字符串进行共享。更多内容参考博文：http://www.cnblogs.com/artech/archive/2010/11/25/internedstring.html 。享元模式的具体结构图如下所示。  

![ ](亨元模式.gif)

## 2.7 代理模式  
&emsp;&emsp;在系统开发中，有些对象由于网络或其他的障碍，以至于不能直接对其访问，此时可以通过一个代理对象来实现对目标对象的访问。如.NET中的调用Web服务等操作。  
&emsp;&emsp;代理模式指的是给某一个对象提供一个代理，并由代理对象控制对原对象的访问。具体的结构图如下所示。  

![ ](代理模式.gif)

&emsp;&emsp;注：外观模式、适配器模式和代理模式区别？  
&emsp;&emsp;解答：这三个模式的相同之处是，它们都是作为客户端与真实被使用的类或系统之间的一个中间层，起到让客户端间接调用真实类的作用，不同之处在于，所应用的场合和意图不同。  
&emsp;&emsp;代理模式与外观模式主要区别在于，代理对象无法直接访问对象，只能由代理对象提供访问，而外观对象提供对各个子系统简化访问调用接口，而适配器模式则不需要虚构一个代理者，目的是复用原有的接口。外观模式是定义新的接口，而适配器则是复用一个原有的接口。  
&emsp;&emsp;另外，它们应用设计的不同阶段，外观模式用于设计的前期，因为系统需要前期就需要依赖于外观，而适配器应用于设计完成之后，当发现设计完成的类无法协同工作时，可以采用适配器模式。然而很多情况下在设计初期就要考虑适配器模式的使用，如涉及到大量第三方应用接口的情况；代理模式是模式完成后，想以服务的方式提供给其他客户端进行调用，此时其他客户端可以使用代理模式来对模块进行访问。  
&emsp;&emsp;总之，代理模式提供与真实类一致的接口，旨在用来代理类来访问真实的类，外观模式旨在简化接口，适配器模式旨在转换接口。  

# 三、行为型模式  
&emsp;&emsp;行为型模式是对在不同对象之间划分责任和算法的抽象化。行为模式不仅仅关于类和对象，还关于它们之间的相互作用。行为型模式又分为类的行为模式和对象的行为模式两种。  
* 类的行为模式——使用继承关系在几个类之间分配行为。  
* 对象的行为模式——使用对象聚合的方式来分配行为。  

&emsp;&emsp;行为型模式包括11种模式：模板方法模式、命令模式、迭代器模式、观察者模式、中介者模式、状态模式、策略模式、责任链模式、访问者模式、解释器模式和备忘录模式。  
* 模板方法模式：封装算法结构，定义算法骨架，支持算法子步骤变化。  
* 命令模式：注重将请求封装为对象，支持请求的变化，通过将一组行为抽象为对象，实现行为请求者和行为实现者之间的解耦。  
* 迭代器模式：注重封装特定领域变化，支持集合的变化，屏蔽集合对象内部复杂结构，提供客户程序对它的透明遍历。  
* 观察者模式：注重封装对象通知，支持通信对象的变化，实现对象状态改变，通知依赖它的对象并更新。  
* 中介者模式：注重封装对象间的交互，通过封装一系列对象之间的复杂交互，使他们不需要显式相互引用，实现解耦。  
* 状态模式：注重封装与状态相关的行为，支持状态的变化，通过封装对象状态，从而在其内部状态改变时改变它的行为。  
* 策略模式：注重封装算法，支持算法的变化，通过封装一系列算法，从而可以随时独立于客户替换算法。  
* 责任链模式：注重封装对象责任，支持责任的变化，通过动态构建职责链，实现事务处理。  
* 访问者模式：注重封装对象操作变化，支持在运行时为类结构添加新的操作，在类层次结构中，在不改变各类的前提下定义作用于这些类实例的新的操作。  
* 备忘录模式：注重封装对象状态变化，支持状态保存、恢复。  
* 解释器模式：注重封装特定领域变化，支持领域问题的频繁变化，将特定领域的问题表达为某种语法规则下的句子，然后构建一个解释器来解释这样的句子，从而达到解决问题的目的。  

## 3.1 模板方法模式  
&emsp;&emsp;在现实生活中，有论文模板，简历模板等。在现实生活中，模板的概念是给定一定的格式，然后其他所有使用模板的人可以根据自己的需求去实现它。同样，模板方法也是这样的。  
&emsp;&emsp;模板方法模式是在一个抽象类中定义一个操作中的算法骨架，而将一些具体步骤实现延迟到子类中去实现。模板方法使得子类可以不改变算法结构的前提下，重新定义算法的特定步骤，从而达到复用代码的效果。具体的结构图如下所示（以生活中做菜为例子实现的模板方法结构图）。  

![以生活中做菜为例子实现的模板方法结构图](模板方法模式.png)

## 3.2 命令模式  
&emsp;&emsp;命令模式属于对象的行为模式，命令模式把一个请求或操作封装到一个对象中，通过对命令的抽象化来使得发出命令的责任和执行命令的责任分隔开。命令模式的实现可以提供命令的撤销和恢复功能。具体的结构图如下所示。  

![ ](命令模式.gif)

## 3.3 迭代器模式  
&emsp;&emsp;迭代器模式是针对集合对象而生的，对于集合对象而言，必然涉及到集合元素的添加删除操作，也肯定支持遍历集合元素的操作，此时如果把遍历操作也放在集合对象的话，集合对象就承担太多的责任了，此时可以进行责任分离，把集合的遍历放在另一个对象中，这个对象就是迭代器对象。  
&emsp;&emsp;迭代器模式提供了一种方法来顺序访问一个集合对象中各个元素，而又无需暴露该对象的内部表示，这样既可以做到不暴露集合的内部结构，又可以让外部代码透明地访问集合内部元素。具体的结构图如下所示。  

![ ](迭代器模式.png)

## 3.4 观察者模式  
&emsp;&emsp;在现实生活中，处处可见观察者模式，例如，微信中的订阅号，订阅博客和QQ微博中关注好友，这些都属于观察者模式的应用。  
&emsp;&emsp;观察者模式定义了一种一对多的依赖关系，让多个观察者对象同时监听某一个主题对象，这个主题对象在状态发生变化时，会通知所有观察者对象，使它们能够自动更新自己的行为。具体结构图如下所示：  

![ ](观察者模式.png)

## 3.5 中介者模式  
&emsp;&emsp;在现实生活中，有很多中介者模式的身影，例如QQ游戏平台，聊天室、QQ群和短信平台，这些都是中介者模式在现实生活中的应用。  
&emsp;&emsp;中介者模式，定义了一个中介对象来封装一系列对象之间的交互关系。中介者使各个对象之间不需要显式地相互引用，从而使耦合性降低，而且可以独立地改变它们之间的交互行为。具体的结构图如下所示：  

![ ](中介者模式.png)

## 3.6 状态模式  
&emsp;&emsp;每个对象都有其对应的状态，而每个状态又对应一些相应的行为，如果某个对象有多个状态时，那么就会对应很多的行为。那么对这些状态的判断和根据状态完成的行为，就会导致多重条件语句，并且如果添加一种新的状态时，需要更改之前现有的代码。这样的设计显然违背了开闭原则，状态模式正是用来解决这样的问题的。  
&emsp;&emsp;状态模式——允许一个对象在其内部状态改变时自动改变其行为，对象看起来就像是改变了它的类。具体的结构图如下所示：  

![ ](状态模式.png)

## 3.7 策略模式  
&emsp;&emsp;在现实生活中，中国的所得税，分为企业所得税、外商投资企业或外商企业所得税和个人所得税，针对于这3种所得税，每种所计算的方式不同，个人所得税有个人所得税的计算方式，而企业所得税有其对应计算方式。如果不采用策略模式来实现这样一个需求的话，我们会定义一个所得税类，该类有一个属性来标识所得税的类型，并且有一个计算税收的CalculateTax()方法，在该方法体内需要对税收类型进行判断，通过if-else语句来针对不同的税收类型来计算其所得税。这样的实现确实可以解决这个场景，但是这样的设计不利于扩展，如果系统后期需要增加一种所得税时，此时不得不回去修改CalculateTax方法来多添加一个判断语句，这样明白违背了“开放——封闭”原则。此时，我们可以考虑使用策略模式来解决这个问题，既然税收方法是这个场景中的变化部分，此时自然可以想到对税收方法进行抽象，这也是策略模式实现的精髓所在。  
&emsp;&emsp;策略模式是对算法的包装，是把使用算法的责任和算法本身分割开，委派给不同的对象负责。策略模式通常把一系列的算法包装到一系列的策略类里面。用一句话慨括策略模式就是——“将每个算法封装到不同的策略类中，使得它们可以互换”。下面是策略模式的结构图：  

![ ](策略模式.png)
　　
## 3.8 责任链模式  
&emsp;&emsp;在现实生活中，有很多请求并不是一个人说了就算的，例如面试时的工资，低于1万的薪水可能技术经理就可以决定了，但是1万~1万5的薪水可能技术经理就没这个权利批准，可能需要请求技术总监的批准。  
&emsp;&emsp;责任链模式——某个请求需要多个对象进行处理，从而避免请求的发送者和接收之间的耦合关系。将这些对象连成一条链子，并沿着这条链子传递该请求，直到有对象处理它为止。具体结构图如下所示：  

![ ](责任链模式.png)

## 3.9 访问者模式  
&emsp;&emsp;访问者模式是封装一些施加于某种数据结构之上的操作。一旦这些操作需要修改的话，接受这个操作的数据结构则可以保存不变。访问者模式适用于数据结构相对稳定的系统， 它把数据结构和作用于数据结构之上的操作之间的耦合度降低，使得操作集合可以相对自由地改变。具体结构图如下所示：  

![ ](访问者模式.png)

## 3.10 备忘录模式  
&emsp;&emsp;生活中的手机通讯录备忘录，操作系统备份点，数据库备份等都是备忘录模式的应用。备忘录模式是在不破坏封装的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态，这样以后就可以把该对象恢复到原先的状态。具体的结构图如下所示：  

![ ](备忘录模式.png)

## 3.11 解释器模式  
&emsp;&emsp;解释器模式是一个比较少用的模式，所以我自己也没有对该模式进行深入研究，在生活中，英汉词典的作用就是实现英文和中文互译，这就是解释器模式的应用。  
&emsp;&emsp;解释器模式是给定一种语言，定义它文法的一种表示，并定义一种解释器，这个解释器使用该表示来解释器语言中的句子。具体的结构图如下所示：  

![ ](解释器模式.png)
