---
layout:     post
title:      "Scala Reflect 使用reify方法查看表达式树"
subtitle:   " \"Hello Scala, Hello Reflect\""
date:       2018-07-03 14:00:00
author:     "Jeorch"
header-img: "img/post-bg-scala.jpg"
catalog: true
tags:
    - Scala
    - Reflect
    - 表达式树
---

> “Yeah It's Scala. ”


## 前言

本文的故事背景是老齐在群里问了一个问题：

```
class User1(id: String, name: String)

class User2(var id: String, var name: String)

class User3(val id: String, val name: String)

class User4(private var id: String,private  var name: String)
```
上面代码定义了四个User类,每个类都有两个参数`id`和`name`,当然,他们构造函数的区别也很明显.那么这几种不同的定义方式,有什么区别呢?

总的来说，可以这么回答：</br>在类中基本构造函数的参数定义前加上val或var后，对应的实例属性就会被定义。</br>
如果构造函数参数声明为val，Scala只为它生成一个getter方法。</br>
如果构造函数参数声明为var，Scala将生成访问器和mutator方法。

参考：
  - [Scala讲座：类定义和构造函数](http://developer.51cto.com/art/200912/166814.htm)，这篇博文都快十年了。。。
  - [W3Cschool Scala 构造函数](https://www.w3cschool.cn/scala/scala-constructors.html)
*PS：简单的来说accessor和mutator就是Scala对pojo的getter/setter一种约定，可以通过反射被大量使用，有兴趣的可以自行Google。*

## 正文

杨总说上面那个关于类的构造函数的问题，使用表达式树看一下定义就一目了然了。

那么现在就该介绍一下scala诊断工具reify了，reify是scala.reflect.runtime.universe抽象类的一个方法，用于对一段表达式参数生成语法树。

Talk is cheap, show me the code:
```
scala> import reflect.runtime.universe._

scala> reify( for(i <- 1 to 10) println(i) )
res0: reflect.runtime.universe.Expr[Unit] = Expr[Unit](scala.this.Predef.intWrapper(1).to(10).foreach(((i) => scala.this.Predef.println(i))))

scala> show(res0)
res1: String = Expr[Unit](scala.this.Predef.intWrapper(1).to(10).foreach(((i) => scala.this.Predef.println(i))))

scala> show(res0.tree)
res2: String = scala.this.Predef.intWrapper(1).to(10).foreach(((i) => scala.this.Predef.println(i)))
```
```
scala> reify{ class User(id : String, name: String) }.tree
res3: reflect.runtime.universe.Tree =
{
  class User extends AnyRef {
    <paramaccessor> private[this] val id: Predef.String = _;
    <paramaccessor> private[this] val name: Predef.String = _;
    def <init>(id: Predef.String, name: Predef.String) = {
      super.<init>();
      ()
    }
  };
  ()
}
```
reify对定位隐式转换有帮助，比如我们不确定一个类型在哪儿被隐式转换了：
```
scala> 1.max(2) //max在哪定义的呢？
res1: Int = 2

scala> reify(1.max(2)).tree
res2: reflect.runtime.universe.Tree = Predef.intWrapper(1).max(2)
```

参考：
  - [scala的诊断方法(2) 在repl下用reify查看表达式的翻译结果](https://hongjiang.info/scala-diagnose-2/)

## 传送门
  - [IBMdeveloper 告诉你 理解Scala的类语法和语义](https://www.ibm.com/developerworks/cn/java/j-scala02198.html)
  - [Daniel认为case class很酷，但是尽量少用var](http://www.codecommit.com/blog/scala/case-classes-are-cool)
  - [Scala官网 告诉你 Scala风格的Accessors/Mutators](https://docs.scala-lang.org/style/naming-conventions.html#accessorsmutators)
