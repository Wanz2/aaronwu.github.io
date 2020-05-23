---
title: Java——对象的序列化与反序列化（1）
date: 2017-05-05 10:03:26
tags: Java
categories: Java
description: 关于序列化和反序列化将要写两篇博客，第一篇介绍基本序列化与反序列化方法，第二篇将介绍如何自己实现元素的序列化与反序列化问题。
---
&emsp;&emsp;关于序列化和反序列化将要写两篇博客，第一篇介绍基本序列化与反序列化方法，第二篇将介绍如何自己实现元素的序列化与反序列化问题。  

<!--more-->
## 概念 
&emsp;&emsp;什么是对象序列化和反序列化？《Thinking in Java》中的解释是这样的：  
> Java的*对象序列化*将那些实现了Serializable接口的对象转换成一个字节序列，并能够在以后将这个字节序列完全恢复为原来的对象。  

&emsp;&emsp;这就是序列化和反序列化的基本内涵。由于Java中的文件是一个一个字节组成的字节序列，因此序列化就是将一个对象转换成Byte序列；而反序列化就是讲Byte序列转换为一个对象。序列化可以将对象保存到文件中，我们也可以将序列化之后的对象通过网络传输，这可以弥补不同操作系统中数据可能表示的不同。  
## 基本操作
&emsp;&emsp;对象序列化和反序列化其实很简单，有两个关注点：  
&emsp;&emsp;&emsp;&emsp;1. 对象必须实现序列化接口Serializable，否则将出现异常。该接口中没有任何方法，只提供了一个标准。  
&emsp;&emsp;&emsp;&emsp;2. 使用序列化流**ObjectOutputStream**和反序列化流**ObjectInputStream**。  
&emsp;&emsp;以下为序列化和反序列化的一个示例：  

```Java
import java.io.Serializable;

/**
 * Created by wuwenan on 2016/12/5 0005.
 * 实现了Serializable接口的类
 */
public class Person implements Serializable{
    private String name;
    private String gender;

    //该元素不会进行jvm默认的序列化，也可以自己完成这个元素的序列化
    private transient int age;

    public Person() {}

    public Person(String name, String gender, int age) {
        super();
        this.name = name;
        this.gender = gender;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getGender() {
        return gender;
    }

    public void setGender(String gender) {
        this.gender = gender;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", gender='" + gender + '\'' +
                ", age=" + age +
                '}';
    }

}

```  

```Java
import java.io.*;

/**
 * 实现了基本的序列化与反序列化流程
 * Created by Wenan Wu on 2016/12/5 0005.
 */
public class ObjectSeriaDemo1 {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        File file = new File("/Users/wuwenan/Work/SerializationTest.txt");
        if(!file.exists()){
            file.createNewFile();
        }
        //1.对象的序列化
        //从文件创建序列化流对象
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(file));
        //创建新的实体对象
        Person stu = new Person("Aaron Wu", "male", 23);
        //将实例写入序列化流对象缓存
        oos.writeObject(stu);//writeObject()方法是调用自被序列化的对象中的writeObject()方法
        //将缓存内容输出
        oos.flush();
        oos.close();

        //2.对象的反序列化
        //从文件创建反序列化流对象
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream(file));
        //通过反序列化流对象读取实体对象，由于返回的是Object类型，因此要进行向下转型
        Person stu1 = (Person) ois.readObject();
        System.out.println(stu1);
        ois.close();

    }
}

```

