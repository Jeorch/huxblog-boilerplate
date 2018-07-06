---
layout:     post
title:      "Java实现鸭子类型机制【转载】"
subtitle:   " \"动态类型\""
date:       2018-07-05 18:00:00
author:     "Jeorch"
header-img: "img/post-bg-java.jpg"
catalog: false
tags:
    - Java
    - 鸭子类型
---

> “Yeah It's Java. ”


## 前言

看到一篇有趣的文章，是关于“鸭子类型”的。

## 正文

在程序设计中，鸭子类型（英语：duck typing）是动态类型的一种风格。在这种风格中，一个对象有效的语义，不是由继承自特定的类或实现特定的接口，而是由当前方法和属性的集合决定。这个概念的名字来源于由James Whitcomb Riley提出的鸭子测试（见下面的“历史”章节），“鸭子测试”可以这样表述：

“当看到一只鸟走起来像鸭子、游泳起来像鸭子、叫起来也像鸭子，那么这只鸟就可以被称为鸭子。“【引用维基】

一般而言，鸭子类型机制常见/用于动态语言，如Python、Ruby、JS。来段Python代码实例：

```
class Duck:
    def quack(self):
        print "Quaaaaaack!"
    def feathers(self):
        print "The duck has white and gray feathers."

class Person:
    def quack(self):
        print "The person imitates a duck."
    def feathers(self):
        print "The person takes a feather from the ground and shows it."

def in_the_forest(duck):
    duck.quack()
    duck.feathers()

def game():
    donald = Duck()
    john = Person()
    in_the_forest(donald)
    in_the_forest(john)

game()
```

从上面代码片段可以看出，鸭子类型设计风格更关注对象的使用，如正则（非成员）函数in_the_forest的形参duck，只要有方法quack和feathers即可，不用关注具体类型，这样使设计更加”范化“。当然鸭子类型机制并不仅限动态语言，在静态语言中也可见，本文主要阐述在Java中的鸭子类型机制（亦称潜在类型机制或结构化类型机制）实现。
实现方式一： 使用一个类或接口，并用有边界泛型方法实现，见代码：

```
// interface Performs.class
public interface Performs {
    void quack();
    void feathers();
}

// class Duck.class
public class Duck implements Performs {
    public void quack() {
        System.out.println("Quaaaaak!");
    }

   public void feathers() {
        System.out.println("The duck has white and gry feathers!");
    }
}

// class Person.class
public class Person implements Performs {
    public void quack() {
        System.out.println("The person imitates a duck!");
    }

    public void feathers() {
         System.out.println("The person has yellow feather!");
    }
}

// class  MainClass.class
public class MainClass {
    public static <T extends Performs> void inTheForest(T duck) {
        duck.quack();
        duck.feathers();
    }

    public static void main(String[] args) {
         Duck d = new Duck();
         Person p = new Person();
         MainClass.inTheForest(d);
         MainClass.inTheForest(p);
    }
}
```

方式一使用的前提是都有共同的基类或者接口，但是当两个类之间没有这个条件仅有相同的方法签名时如何实现呢？这时可以采用如下两种方法：
方式二： 利用反射，见代码:

```
// class Duck.class
public class Duck {
    public void quack() {
        System.out.println("Quaaaaak!");
    }

   public void feathers() {
        System.out.println("The duck has white and gry feathers!");
    }
}

// class Person.class
public class Person {
    public void quack() {
        System.out.println("The person imitates a duck!");
    }

    public void feathers() {
         System.out.println("The person has yellow feather!");
    }
}

// class  MainClass.class
import java.lang.reflect.*;

public class MainClass {
    public static void inTheForest(Object duck) {
        Class<?> duk = duck.getClass();
        try {
            try {
                Method quack = duk.getMethod("quack");
                quack.invoke(duck);
            } catch(NoSuckMethodException e) {
                System.err.println(duck + " cannot quack");
            }
            try {
                Method feathers = duk.getMethod("feathers");
                feathers.invoke(duck);
            } catch(NoSuckMethodException e) {
                System.err.println(duck + " cannot feathers");
            }
        } catch(Exception e) {
            throw new RuntimeException(duck.toString(), e);
        }
    }

    public static void main(String[] args) {
         Duck d = new Duck();
         Person p = new Person();
         MainClass.inTheForest(d);
         MainClass.inTheForest(p);
    }
}
```

方式三： 模拟接口，使用适配器设计模式
```
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

// interface Performs.class
// 先前不包含这个接口，也不继承这个接口
interface Performs {
    void quack();
    void feathers();
}


// class Duck.class
class Duck {
    public void quack() {
        System.out.println("Quaaaaak!");
    }


    public void feathers() {
        System.out.println("The duck has white and gry feathers!");
    }
}


// class Person.class
class Person {
    public void quack() {
        System.out.println("The person imitates a duck!");
    }


    public void feathers() {
        System.out.println("The person has yellow feather!");
    }
}


// Adapter
class PerformsAdapter implements Performs {
    private Object d;
    public PerformsAdapter(Object d) {
        this.d = d;
    }
    public void quack() {
        Class<?> dk = d.getClass();
        try {
            Method qk = dk.getMethod("quack");
            qk.invoke(d);
        } catch (NoSuchMethodException e) {
            System.out.println(d + " cannot quack");
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
    }
    public void feathers() {
        Class<?> dk = d.getClass();
        try {
            Method ft = dk.getMethod("feathers");
            ft.invoke(d);
        } catch (NoSuchMethodException e) {
            System.out.println(d + " cannot have feathers");
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
    }
}


// class  MainClass.class
public class DuckTest {
    public static <T extends Performs> void inTheForest(T duck) {
        duck.quack();
        duck.feathers();
    }


    public static void main(String[] args) {
        Duck d = new Duck();
        Person p = new Person();
        DuckTest.inTheForest(new PerformsAdapter(d));
        DuckTest.inTheForest(new PerformsAdapter(p));
    }
}
```

参考文献：

1. 鸭子类型: http://zh.wikipedia.org/wiki/%E9%B8%AD%E5%AD%90%E7%B1%BB%E5%9E%8B

2. 《Thinking in Java》

*转自：[Java中实现鸭子类型机制](https://blog.csdn.net/bobozhengsir/article/details/20625727)*
