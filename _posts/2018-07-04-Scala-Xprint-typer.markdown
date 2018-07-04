---
layout:     post
title:      "Scala分析工具 使用-Xprint:typer看语法糖的背后【转载】"
subtitle:   " \"Scala内部世界\""
date:       2018-07-04 16:00:00
author:     "Jeorch"
header-img: "img/post-bg-scala.jpg"
catalog: false
tags:
    - Scala
    - JVM
---

> “Yeah It's Scala. ”


## 前言

scala中很多看似花哨的语法表达，其实背后并不是语言级别的支持，都是些语法糖。当我们拿不准一些语句最终被翻译为什么，可以通过一些编译时参数来帮助我们分析。

## 正文

先看一下scalac编译过程的各个阶段(phases):

```
$ scalac -Xshow-phases

         phase name  id  description
         ----------  --  -----------
             parser   1  parse source into ASTs, perform simple desugaring
              namer   2  resolve names, attach symbols to named trees
     packageobjects   3  load package objects
              typer   4  the meat and potatoes: type the trees
             patmat   5  translate match expressions
     superaccessors   6  add super accessors in traits and nested classes
         extmethods   7  add extension methods for inline classes
            pickler   8  serialize symbol tables
          refchecks   9  reference/override checking, translate nested objects
       selectiveanf  10
       selectivecps  11
            uncurry  12  uncurry, translate function values to anonymous classes
          tailcalls  13  replace tail calls by jumps
         specialize  14  @specialized-driven class and method specialization
      explicitouter  15  this refs to outer pointers, translate patterns
            erasure  16  erase types, add interfaces for traits
        posterasure  17  clean up erased inline classes
           lazyvals  18  allocate bitmaps, translate lazy vals into lazified defs
         lambdalift  19  move nested functions to top level
       constructors  20  move field definitions into constructors
            flatten  21  eliminate inner classes
              mixin  22  mixin composition
            cleanup  23  platform-specific cleanups, generate reflective calls
              icode  24  generate portable intermediate code
            inliner  25  optimization: do inlining
inlineExceptionHandlers  26  optimization: inline exception handlers
           closelim  27  optimization: eliminate uncalled closures
                dce  28  optimization: eliminate dead code
                jvm  29  generate JVM bytecode
           terminal  30  The last phase in the compiler chain
```

第一个parser阶段就已经做了部分简单的“脱糖”处理来去除语法糖，不过对我们最有帮助的还是 typer 和 jvm 两个阶段。一般我们可以通过 -Xprint:typer 来看一些语法糖表达式的背后

```
$ scalac -Xprint:typer A.scala
```

或者直接以脚本的方式：

```
$ scala -Xprint:typer -e 'def incr(i:Int) = i+1'
```

例子，看一下for表达式的背后：

```
$ scala -Xprint:typer -e 'val l=List(1,2,3); for(i<-l) {println(i)}'
    ...
    private[this] val l: List[Int] = immutable.this.List.apply[Int](1, 2, 3);
    <stable> <accessor> private def l: List[Int] = $anon.this.l;
    $anon.this.l.foreach[Unit](((i: Int) => scala.this.Predef.println(i)))
```

可见在上面的例子里是被翻译为了foreach操作。

*转自：[scala的诊断方法(1) 使用-Xprint:typer看语法糖的背后](http://hongjiang.info/scala-diagnose-1/)*
