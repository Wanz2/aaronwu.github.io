---
title: Java——对象的序列化与反序列化（2）
date: 2017-05-09 09:47:52
categories: Java
tags: Java
description: 本文是Java对象的序列化与反序列化的第二篇，主要记录了自定义序列化与反序列化的方式，以及序列化与反序列化过程中继承的关系。
---
&emsp;&emsp;序列化与反序列化操作中，可能在某些情况下我们不希望对象中的所有元素都被序列化，如对象中的某个元素是密码一类的敏感信息，若经过序列化处理，可能被他人通过读取文件或拦截网络传输等方式访问到。防止自定义序列化的方式有两种：  
&emsp;&emsp;第一种是将类实现Externalizable接口，该接口继承了Serializable接口同时增添了两个方法：writeExternal()和readExternal()  
&emsp;&emsp;第二种是在实现Serializable接口时使用transient（瞬时）关键字。  
&emsp;&emsp;本文继承上文，介绍上面的第二种方法。
## Serizable类对象自定义序列化
&emsp;&emsp;自定义序列化有两个关键点，如下：  
&emsp;&emsp;&emsp;&emsp;1. 使用transient（瞬时）关键字，当使用该关键字时，可通过它逐个字段地关闭序列化。  
&emsp;&emsp;&emsp;&emsp;2. 重写writeObject()和readObject()方法。  
具体代码示例如下：  

```java
import java.io.*;

/**
 * 自定义序列化和反序列化操作
 * Created by wuwenan on 09/05/2017.
 */
public class Student implements Serializable {
    private String id;
    private transient String name;

    public Student(String id, String name) {
        this.id = id;
        this.name = name;
    }

    @Override
    public String toString() {
        return "Student{" +
                "id='" + id + '\'' +
                ", name='" + name + '\'' +
                '}';
    }

    private void writeObject(java.io.ObjectOutputStream s)
            throws java.io.IOException{
        //把jvm能默认序列化的元素进行序列化操作
        s.defaultWriteObject();
        //自定义序列化
        s.writeObject(name);
    }

    private void readObject(java.io.ObjectInputStream s)
            throws java.io.IOException, ClassNotFoundException {
        //把jvm能默认反序列化的元素进行反序列化操作
        s.defaultReadObject();
        //自己完成反序列化操作
        this.name = (String) s.readObject();
    }

    public static void main(String[] args) throws IOException, ClassNotFoundException {

        //1.对象序列化
        //创建对象实例
        Student student = new Student("123","娜美");
        //创建文件
        File file = new File("/Users/wuwenan/Work/customSerialization.txt");
        if(!file.exists()){
            file.createNewFile();
        }
        //从文件创建序列化流对象
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(file));
        //将对象实例写入序列化流缓存
        oos.writeObject(student);
        //将序列化流缓存输出
        oos.flush();
        //关闭序列化流
        oos.close();

        //2.对象的反序列化
        //从文件创建反序列化流对象
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream(file));
        //通过反序列化流对象读取实体对象，由于返回的是Object类型，因此要进行向下转型
        Student stu1 = (Student) ois.readObject();
        System.out.println(stu1);
        ois.close();
    }
}

```

## 序列化和反序列化中的继承问题
&emsp;&emsp;一个类实现了序列化接口，那么其子类都可以进行序列化；在对子类对象进行反序列化时，若其父类没有实现序列化接口，则从没有实现序列化接口的那一级父类开始，该类及其所有子类的构造方法会被显式调用。  
&emsp;&emsp;以下分别用两个demo来说明：  
&emsp;&emsp;一个类实现了序列化接口，那么其子类都可以进行序列化。  

```java
import java.io.*;

/**
 * Created by wuwenan on 2016/12/5 0005.
 * 对象序列化示例
 */
public class ObjectSeriaDemo2 {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        File file = new File("/Users/wuwenan/Work/serialization2.txt");
        if(!file.exists()){
            file.createNewFile();
        }
        //1.进行序列化
        ObjectOutputStream oos = new ObjectOutputStream(
                new FileOutputStream(file));
        Foo2 foo2 = new Foo2();
        oos.writeObject(foo2);
        oos.flush();
        oos.close();

        //2.进行反序列化
        //反序列化是否递归调用父类构造函数，这里控制台只打印了对象，但不能说明没有调用父类构造函数
        ObjectInputStream ois = new ObjectInputStream(
                new FileInputStream(file));
        Foo2 foo2temp = (Foo2) ois.readObject();
        System.out.println(foo2temp);
        ois.close();
    }
}

/*
一个类实现了序列化接口，那么其子类都可以进行序列化
 */
class Foo implements Serializable {
    public Foo() {
        System.out.println("Foo...");
    }
}

class Foo1 extends Foo {
    public Foo1() {
        System.out.println("Foo1...");
    }
}

class Foo2 extends Foo1 {
    public Foo2() {
        System.out.println("Foo2...");
    }
}

```

&emsp;&emsp;在对子类对象进行反序列化时，若其父类没有实现序列化接口，则从没有实现序列化接口的那一级父类开始，该类及其所有子类的构造方法会被显式调用。  

```java
import java.io.*;

/**
 * Created by wuwenan on 2016/12/5 0005.
 * 对象序列化示例
 */
public class ObjectSeriaDemo2 {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        //创建序列化测试文件
        File file = new File("/Users/wuwenan/Work/serializationBar.txt");
        if(!file.exists()){
            file.createNewFile();
        }

        //进行序列化
        ObjectOutputStream oos = new ObjectOutputStream(
                new FileOutputStream("/Users/wuwenan/Work/serializationBar.txt"));
        Bar2 bar2 = new Bar2();
        oos.writeObject(bar2);
        oos.flush();
        oos.close();

        //进行反序列化
        System.out.println("====反序列化的分隔线====");
        ObjectInputStream ois = new ObjectInputStream(
                new FileInputStream("/Users/wuwenan/Work/serializationBar.txt"));
        Bar2 bar2temp = (Bar2) ois.readObject();
        System.out.println(bar2temp);
        ois.close();
    }
}

class Bar {
    public Bar() {
        System.out.println("bar");
    }
}

class Bar1 extends Bar implements Serializable {
    public Bar1() {
        System.out.println("bar1...");
    }
}

class Bar2 extends Bar1 {
    public Bar2() {
        System.out.println("bar2...");
    }
}

```
