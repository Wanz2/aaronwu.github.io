---
title: Java——通配符和泛型边界
categories:
  - Java
tags:
  - Java
  - 泛型
mathjax: true
description: Java泛型中有一个比较容易令人困惑的知识点：泛型的边界，而讨论泛型的边界则要涉及到通配符“？”的使用，本文将对此进行讲解。
date: 2017-07-14 20:23:23
---
&emsp;&emsp;本文源起于笔者在阅读《Thinking in Java》泛型章节的时候，遇到的关于泛型边界的时候思考的一个问题，笔者就该问题分别询问了三位硕士小伙伴，大家都没有回答正确，因此笔者觉得有必要对此进行一个辨析。问题本身并不复杂，且听我细细道来。  
&emsp;&emsp;首先，假设有四个类存在如下的继承关系：  
  
```java
class Fruit {
}

class Apple extends Fruit {
}

class Jonathan extends Apple {
}

class Orange extends Fruit {
}
```

&emsp;&emsp;`Fruit`类是超类，`Apple`和`Orange`继承了`Fruit`，而`Jonathan`继承了`Apple`。  

# 泛型不存在协变  

&emsp;&emsp;首先我们来看这样一段代码：  

```java
package generics;//: generics/NonCovariantGenerics.java
// {CompileTimeError} (Won't compile)

import java.util.ArrayList;
import java.util.List;

public class NonCovariantGenerics {
    // Compile Error: incompatible types:
    List<Fruit> flist = new ArrayList<Apple>();
} ///:~
```

&emsp;&emsp;从注释上可以看到，这段代码是无法通过编译的。你可能以为在第9行里，用`Apple`类型的ArrayList来初始化`Fruit`类型的List是向上转型，然而这并不是向上转型，`Apple`类型的List不是`Fruit`类型的List，因为`Apple`类型的List可以持有`Apple`对象及其子类对象，而`Fruit`类型的List将持有`Fruit`类型对象及其子类对象，包括`Apple`类型对象，它可持有的范围更大。  
&emsp;&emsp;因此**不能用`Apple`类型的List初始化`Fruit`类型的List**，这说明**泛型不存在协变类型**。  

# 通配符表示的泛型的下界

&emsp;&emsp;但你还是想建立某种向上转型的关系，于是你想到了通配符：`？`，你可能会想到像下面这样来使用它：  

```java
package generics;//: generics/GenericsAndCovariance.java

import java.util.ArrayList;
import java.util.List;

public class GenericsAndCovariance {
    public static void main(String[] args) {
        // Wildcards allow covariance:
        List<? extends Fruit> flist = new ArrayList<Apple>();
        // Compile Error: can't add any type of object:
        // flist.add(new Apple());
        // flist.add(new Fruit());
        // flist.add(new Object());
        flist.add(null); // Legal but uninteresting
        // We know that it returns at least Fruit:
        Fruit f = flist.get(0);
    }
} ///:~
```

&emsp;&emsp;很遗憾，从注释中你可以看到，像上面那样使用`extends`关键字，虽然可以通过编译，但在为List中添加对象的时候，是无法成功的，不论你添加的是`Fruit`本身，还是它的子类，甚至是`Object`，都无法添加成功。你唯一可以做的是从List中读取对象。  
&emsp;&emsp;那么为什么不可以往该容器中装对象呢？因为`List<? extends Fruit>`表示该List持有的是**“Fruit类的某个不确定的子类型”**，但编译器并不知道该List中持有的究竟是`Fruit`的哪个子类型，因此无法将诸如`Apple`的`Fruit`的某个具体子类型装入该List，同时`Fruit`本身也不能被装入，当然，`Fruit`的父类型`Object`就更加不可以了，装入这些类时的转型都不是类型安全的。因此，这个通配符表示的泛型下界可以说是无下限的。  
&emsp;&emsp;那么为什么可以调用`get()`方法来读取该List呢？我们已经知道该List中持有的一定是`Fruit`的某个子类型，因此它的上界是确定的，就是`Fruit`，因此当取出的时候，将它赋值给一个静态类型为`Fruit`的对象，就属于向上转型，因此是没有问题的。  

# 通配符表示的泛型上界
&emsp;&emsp;通配符还有一种用法，就是表示泛型的上界，如下面所示：    

```java
package generics;//: generics/SuperTypeWildcards.java

import java.util.List;

public class SuperTypeWildcards {
    static void writeTo(List<? super Apple> apples) {
        apples.add(new Apple());
        apples.add(new Jonathan());
        // apples.add(new Fruit()); // Error
    }
} ///:~
```

&emsp;&emsp;像上面这样使用`List<? super Apple>`，就可以向List中添加`Apple`及其子类`Jonathan`了，但向其中添加`Apple`的父类`Fruit`则是不安全的。原因是`List<? super Apple>`表示该List持有的是**Apple的某个不确定的超类型**，既然该List中持有的一定是`Apple`的某个超类型，那么使用`add()`方法向其中添加`Apple`及其子类型，一定是类型安全的；但由于编译器只知道这个List持有的是`Apple`的某个超类型，但不能确定究竟是哪个超类型，因此向其中添加`Fruit`类型是不安全的。

# 总结  
&emsp;&emsp;总结一下就是：  

 - **<? extends T>** 表示T的某个不确定的子类，因此不能存放具体的T类对象及其子类，更不能存放T的父类，因此`Object`类不能存放。
 - **<? super T>** 表示T的某个不确定的超类，因此可以存放T类对象及T的子类对象。

