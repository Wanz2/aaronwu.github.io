---
title: JVM 类加载机制解析
categories:
  - Java
tags:
  - Java
mathjax: true
description: 从jdk 源码层面谈谈 jvm 类加载机制。
date: 2020-06-27 22:16:09
---

​	当JVM 执行某个类的方法时，首先需要通过**类加载器**把类加载到JVM。

## 类加载器

​	JVM 类加载阶段分为加载、连接、初始化三个阶段，加载阶段JVM通过一个类的全限定名来获取定义此类的二进制字节流。在加载阶段， JVM 会创建一个引导类加载器（BootstrapClassLoader）的实例，然后 C++代码会调用 Javadiamante 创建 JVM 启动器类 sun.misc.Launcher，然后引导类加载器加载 Laucher 类，JVM 实例化该类，由该类去创建其他类加载器，然后由这些类加载器去加载所有的类。

​	由于类加载器的存在，Java 中任意一个类都是由类加载器+类本身信息来确定该类在 JVM 中的唯一性。即使两个类有的字节码文件完全相同，只要两者的类加载器不同，那么在 JVM 中也会被认为是不同的类。

​	JVM 中共有以下几种类加载器：

* 引导类加载器(BootstrapClassLoader)：由 C++代码实现，负责加载支撑JVM运行的位于JRE的lib目录下的核心类库，比如rt.jar、charsets.jar等。它是所有类加载器的双亲。
* 扩展类加载器(ExtClassLoader)：负责加载支撑JVM运行的位于JRE的lib目录下的ext扩展目录中的JAR类包
* 应用程序类加载器(AppClassLoader)：负责加载ClassPath路径下的类包，主要负责加载用户类路径上的类库，也就是用户自己实现的那些类。它是Java 程序中的默认类加载器。
* 自定义加载器：负责加载用户自定义路径下的类包。

​	此处特别要提到一点，Java 类加载机制遵循“双亲委派模型”，上文中也说到引导类加载器是所有类加载器的双亲，**但并不意味着上述的几个类加载器有继承关系**，事实上，这几个类加载器在 jdk 源码中的实现，存在的是关联关系，“双亲委派”也只是通过程序的调用逻辑上去实现的。真正的类加载器类之间的关系如下图：

![JVM 类加载器模型类图](ClassLoader Class Diag.png)

## 双亲委派机制

​	双亲委派机制是JVM 决定由哪一个类加载器去加载指定类的机制。

​	双亲委派机制的具体流程是：那么首先它会把这个类请求委派给父加载器去完成，每一层都是如此。一直递归到顶层，如果所有父加载器在自己的加载类路径下都找不到目标类，则自己的类加载路径中查找并载入目标类。这里的双亲其实就指的是父类。这里把重要的事再强调一遍：这里的父加载器与当前加载器之间并非继承关系，只是程序的调用关系。

​	所有的类加载器都继承自 ClassLoader 类，他们都使用 loadClass 方法加载类，而双亲委派机制也是在 loadClass 方法中实现。当类加载器发现父加载器无法加载而需要自己去加载类的时候（在 loadClass 方法体内的逻辑），会调用 findClass 方法去加载指定类，这个方法最终会调用到由 C++实现的 native 方法。

​	双亲委派机制的流程图如下：

![双亲委派机制](parent delegation.png)

## ClassLoader.loadClass 双亲委派机制实现流程

​	用源码说话，我将实现双亲委派机制的核心方法 loadClass 方法的处理流程画成了活动图，以供参考。

![ClassLoader.loadClass 方法活动图](ClassLoader.loadClass Activity Diag.png)

## 打破双亲委派机制

1. 继承 ClassLoader 类，重写 loadClass方法
2. 使用线程上下文类加载器（ContextClassLoader）