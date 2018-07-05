---
layout:     post
title:      "Scala运行时反射简单介绍"
subtitle:   " \"Scala内部世界\""
date:       2018-07-04 18:00:00
author:     "Jeorch"
header-img: "img/post-bg-scala.jpg"
catalog: false
tags:
    - Scala
    - Reflection
---

> “Yeah It's Scala. ”


## 前言

看了些Scala官网的文档，暂时没能直接转化成自己的语言[说白了就是没理解透彻]，结合别人的博客，做了一些简单的总结。

## 正文

运行期反射的作用:

观察对象的类型 包括泛型

实例化一个对象

访问或者调用对象的成员

要使用运行期反射的相关工具的话 导入以下的包:
```
import scala.reflect.runtime.universe._
```

TypeTags:由编辑器生成

只能通过隐式参数或者上下文绑定获取

可以有两种方式获取:
```
scala> import scala.reflect.runtime.universe._  
import scala.reflect.runtime.universe._  

//使用typeTag  
scala> def getTypeTag[T:TypeTag](a:T) = typeTag[T]  
getTypeTag: [T](a: T)(implicit evidence$1: reflect.runtime.universe.TypeTag[T])reflect.runtime.universe.TypeTag[T]  

//使用implicitly 等价的   
//scala>def getTypeTag[T:TypeTag](a:T) = implicitly[TypeTag[T]]  

scala> getTypeTag(List(1,2,3))  
res0: reflect.runtime.universe.TypeTag[List[Int]] = TypeTag[List[Int]]
```

通过TypeTag的tpe方法获得需要的Type(如果不是从对象换取Type 而是从class中获得 可以直接用 typeOf[类名])

得到Type之后 就可以have fun了。【杨总的JsonAPI】

用 universe下的runtimeMirror 返回运行期所有的类:
```
scala> runtimeMirror(getClass.getClassLoader)  
res6: reflect.runtime.universe.Mirror = JavaMirror with scala.tools.nsc.interpreter.IMain$TranslatingClassLoader@189c091e of type class scala.tools  
.nsc.interpreter.IMain$TranslatingClassLoader with classpath [(memory)] and parent being scala.tools.nsc.util.ScalaClassLoader$URLClassLoader@2c1a9  
d31 of type class scala.tools.nsc.util.ScalaClassLoader$URLClassLoader with classpath [file:/E:/java/jdk1.7.0_51/jre/lib/resources.jar,file:/E:/jav  
a/jdk1.7.0_51/jre/lib/rt.jar,file:/E:/java/jdk1.7.0_51/jre/lib/jsse.jar,file:/E:/java/jdk1.7.0_51/jre/lib/jce.jar,file:/E:/java/jdk1.7.0_51/jre/lib  
/charsets.jar,file:/E:/java/jdk1.7.0_51/jre/lib/jfr.jar,file:/E:/java/jdk1.7.0_51/jre/lib/ext/access-bridge-64.jar,file:/E:/java/jdk1.7.0_51/jre/li  
b/ext/dnsns.jar,file:/E:/java/jdk1.7.0_51/jre/lib/ext/jaccess....
```

Universe的种类:

1、scala.reflect.runtime.universe 运行时的反射

2、scala.reflect.marcos.universe编译期的反射

Mirrors的种类：

1、classloader等级的mirrors

staticClass staticModule staticPackage

2、调用级别的mirrors

MethodMirror.apply FieldMirror.get（set）

运行期的mirror:

runtimeMirror(<classloader>)（通过runtime.universe)

得到scala.reflect.api.JavaMirrors#runtimeMirror

classloader的mirror可以创造invoker的mirror：
```
scala.reflect.api.Mirrors#InstanceMirror
scala.reflect.api.Mirrors#MethodMirror
scala.reflect.api.Mirrors#FieldMirror
scala.reflect.api.Mirrors#ClassMirror
scala.reflect.api.Mirrors#ModuleMirror
```
InstanceMirror是用来创建方法和属性的invoker mirrors，使用reflect(实例)得到。

MethodMirror用来访问实例的方法，可以用InstanceMirror的reflectMethod(MethodSymbol)得到。

FieldMirror用来访问属性(getting/setting)，可以用InstanceMirror的reflectField。

ClassMirror用来创建构造器的调用mirror，可以通过reflectClass(ClassSymbol)得到。

ModuleMirror用来访问单例Object，使用reflectModule(ModuleSymbol)

获取Class的symbol 用Type的typeSymbol 再asClass

获取Filed和Method的symbol 用Type的declaration 参数是newTermName("属性或方法名") 再asTerm(属性可能需要asTerm.accessed.asTerm）

获取构造器的用Type的declaration 参数是nme.CONSTRUCTOR 再asMethod

获取Module的用Type的termSymbol再asModule

感谢fair_jm前辈的总结，要是仅看官网还是不能很好的理解scala运行时反射，英语阅读水平！！！

*参考：[scala运行时反射简单介绍](http://fair-jm.iteye.com/blog/2163746)*
