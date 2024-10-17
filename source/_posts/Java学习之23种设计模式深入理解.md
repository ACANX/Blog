---
title: Java之23种设计模式深入理解
date: 2019-11-25 15:55:48
categories: "Java" 
tags: #Java 
     - Java
     - 设计模式
description: Java23种设计模式  
---
# 设计模式（Design Pattern）

设计模式（Design pattern）是一套被反复使用、多数人知晓的、经过分类编目的、代码设计经验的总结。使用设计模式是为了可重用代码、让代码更容易被他人理解、保证代码可靠性。 毫无疑问，设计模式于己于他人于系统都是多赢的，设计模式使代码编制真正工程化，设计模式是软件工程的基石，如同大厦的一块块砖石一样。项目中合理的运用设计模式可以完美的解决很多问题，每种模式在现在中都有相应的原理来与之对应，每一个模式描述了一个在我们周围不断重复发生的问题，以及该问题的核心解决方案，这也是它能被广泛应用的原因。

<!--  - 本章系Java之美[从菜鸟到高手演变]系列之设计模式，我们会以理论与实践相结合的方式来进行本章的学习，希望广大程序爱好者，学好设计模式，做一个优秀的软件工程师！-->


#  一、设计模式的分类

- 总体来说设计模式分为三大类：

## 创建型模式，
- 共五种：工厂方法模式、抽象工厂模式、单例模式、建造者模式、原型模式。

## 结构型模式
- 共七种：适配器模式、装饰器模式、代理模式、外观模式、桥接模式、组合模式、享元模式。

## 行为型模式
- 共十一种：策略模式、模板方法模式、观察者模式、迭代子模式、责任链模式、命令模式、备忘录模式、状态模式、访问者模式、中介者模式、解释器模式。

### 其他模式
- 其实还有两类：并发型模式和线程池模式。用一个图片来整体描述一下：
![设计模式之间的关系](/Blog/static/image/20191125/RelationshipBetweenDesignPattern.jpg)

<!--  http://www.itmayun.com/upload_files/files/1/subject/995726589692643/1.jpg  -->

# 二、设计模式的六大原则

## 1、开闭原则（Open Close Principle）
- 开闭原则就是说对扩展开放，对修改关闭。在程序需要进行拓展的时候，不能去修改原有的代码，实现一个热插拔的效果。所以一句话概括就是：为了使程序的扩展性好，易于维护和升级。想要达到这样的效果，我们需要使用接口和抽象类，后面的具体设计中我们会提到这点。

##  2、里氏代换原则（Liskov Substitution Principle）
- 里氏代换原则(Liskov Substitution Principle LSP)面向对象设计的基本原则之一。 里氏代换原则中说，任何基类可以出现的地方，子类一定可以出现。 LSP是继承复用的基石，只有当衍生类可以替换掉基类，软件单位的功能不受到影响时，基类才能真正被复用，而衍生类也能够在基类的基础上增加新的行为。里氏代换原则是对“开-闭”原则的补充。实现“开-闭”原则的关键步骤就是抽象化。而基类与子类的继承关系就是抽象化的具体实现，所以里氏代换原则是对实现抽象化的具体步骤的规范。

##  3、依赖倒转原则（Dependence Inversion Principle）
- 这个是开闭原则的基础，具体内容：真对接口编程，依赖于抽象而不依赖于具体。

##  4、接口隔离原则（Interface Segregation Principle）
- 这个原则的意思是：使用多个隔离的接口，比使用单个接口要好。还是一个降低类之间的耦合度的意思，从这儿我们看出，其实设计模式就是一个软件的设计思想，从大型软件架构出发，为了升级和维护方便。所以上文中多次出现：降低依赖，降低耦合。

##  5、迪米特法则（最少知道原则）（Demeter Principle）
- 最少知道原则，就是说：一个实体应当尽量少的与其他实体之间发生相互作用，使得系统功能模块相对独立。

## 6、合成复用原则（Composite Reuse Principle）
- 原则是尽量使用合成/聚合的方式，而不是使用继承。


# 模式分类Demo源码
- 参考：http://www.cnblogs.com/foryang/p/5849402.html
- 以下是学习过程中查询到的资料,比较容易理解(毕竟站在巨人的肩膀上...)

## 创建型
- 抽象工厂模式 http://www.cnblogs.com/java-my-life/archive/2012/03/28/2418836.html
- 工厂方法 http://www.cnblogs.com/java-my-life/archive/2012/03/25/2416227.html
- 建造者模式  http://www.cnblogs.com/java-my-life/archive/2012/04/07/2433939.html
- 原型模式 http://www.cnblogs.com/java-my-life/archive/2012/04/11/2439387.html
- 单态模式 http://www.cnblogs.com/java-my-life/archive/2012/03/31/2425631.html

## 结构型

- 适配器模式 http://www.cnblogs.com/java-my-life/archive/2012/04/13/2442795.html
- 桥接模式 http://blog.csdn.net/jason0539/article/details/22568865
- 组合模式 http://blog.csdn.net/jason0539/article/details/22642281
- 外观模式 http://blog.csdn.net/jason0539/article/details/22775311
- 装饰者模式 http://www.cnblogs.com/java-my-life/archive/2012/04/20/2455726.html
- 享元模式 http://www.cnblogs.com/java-my-life/archive/2012/04/26/2468499.html
- 代理模式 http://www.cnblogs.com/java-my-life/archive/2012/04/23/2466712.html

## 行为型

- 责任链模式 http://blog.csdn.net/zhouyong0/article/details/7909456
- 命令模式 http://www.cnblogs.com/java-my-life/archive/2012/06/01/2526972.html
- 解释器模式 http://www.cnblogs.com/java-my-life/archive/2012/06/19/2552617.html
- 迭代模式 http://www.cnblogs.com/java-my-life/archive/2012/05/22/2511506.html
- 中介者模式 http://blog.csdn.net/chenhuade85/article/details/8141831
- 备忘录模式 http://www.cnblogs.com/java-my-life/archive/2012/06/06/2534942.html
- 观察者模式 http://www.cnblogs.com/java-my-life/archive/2012/05/16/2502279.html
- 状态模式 http://www.cnblogs.com/java-my-life/archive/2012/06/08/2538146.html
- 策略模式 http://www.cnblogs.com/java-my-life/archive/2012/05/10/2491891.html
- 模板方法模式 http://www.cnblogs.com/java-my-life/archive/2012/05/14/2495235.html
- 访问者模式 http://www.cnblogs.com/java-my-life/archive/2012/06/14/2545381.html

## 题外: http://www.ourlove1314.com/


## 设计模式学习资料汇总
- [我的设计模式Demo代码](https://gitee.com/ACANX/ACANX-JavaSEBase/tree/master/ACANX-JavaSE-Core/src/main/java/com/acanx/designpattern)
- [Java 23种设计模式 深入理解](https://acanx.gitee.io/blog/2018/07/12/Java%2023%E7%A7%8D%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/) 
- [feixiao/DesignPattern](https://github.com/feixiao/DesignPattern/tree/master/Java)
- [设计模式-菜鸟教程](https://www.runoob.com/design-pattern/design-pattern-tutorial.html)
- [bjmashibing/DesignPatterns](https://github.com/bjmashibing/DesignPatterns)
- [非常全的23种设计模式详解，收藏了](https://blog.csdn.net/u013829202/article/details/52513029)
- [《Spring 5企业级开发实战》-设计模式源码](https://github.com/online-demo/spring5projectdemo/tree/master/chapter20/src/main/java/com/test)
- [设计模式-C语言中文网](http://c.biancheng.net/view/1364.html)
- [设计模式总结](https://www.cnblogs.com/chenssy/p/3357683.html)
