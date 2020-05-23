---
title: Java——重写equals()方法
date: 2017-04-30 13:19:34
tags: Java
categories: Java
description: 本文为重写equals()方法的记录
---
&emsp;&emsp;在Java中，自定义类默认继承Object类，因此也继承了Object中的equals方法，Object类中的equals方法定义如下：

```java
     public boolean equals(Object obj) {
        return (this == obj);
    }
```

&emsp;&emsp;这里很简单地返回了两个对象的地址是否相等。但在通常的使用中，我们希望能够比较两个对象中间的内容，因此我们在自己的类中也可以重写equals方法。下面是笔者自己定义了Course类并重写了equals方法：
&emsp;&emsp;Course类包含两个成员变量id和name，笔者希望通过Course类实例的id来判断两个实例是否相等，若id相等，则两个实例相等。  

```java
public class Course {

    private String  id;

    private String name;

    public Course(String s) {
    }

    public String getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    /**
     * 有参构造函数
     * @param id
     * @param name
     */
    public Course(String id, String name) {
        this.id = id;
        this.name = name;
    }

    /**
     * 重写equals方法，通过id属性的值来比较两个对象是否相等，若id值相等，则两对象相等
     * @param obj
     * @return
     */
    public boolean equals(Object obj) {
        if (this == obj) { //判断地址是否相同
            return true;
        }
        if (obj == null) { //判断obj是否为空
            return false;
        }
        if (!(obj instanceof Course)) { //判断obj与this是否属于Course类
            return false;
        }
        Course course = (Course) obj; //若obj与this属于同类，则将obj强制转换为Course类
        if (this.getId() == null) { //判断this的id属性是否为空
            if (course.getId() == null) { //若this.id和course.id均为空，则两者相等
                return true;
            } else {
                return false;
            }
        } else {
            if (this.getId().equals(course.getId())) { //这里调用的是String类型的equals方法，若两者内容相同，则相等
                return true;
            } else {
                return false;
            }
        }
    }

    public static void main(String[] args) {
        Course c1 = new Course("1", "数据结构");
        Course c2 = new Course("1", "数据结构");
        Course c3 = new Course("1", "Java");
        Course c4 = new Course("2", "数据结构");
        System.out.println("c1与c2是否相等：" + c1.equals(c2));
        System.out.println("c1与c3是否相等：" + c1.equals(c3));
        System.out.println("c1与c4是否相等：" + c1.equals(c4));
    }
}
```