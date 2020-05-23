---
title: Java——利用Collections.sort()对泛型为String的List排序
date: 2017-04-30 13:23:10
tags: Java
categories: Java
description: 本文利用Collections.sort()对泛型为String的List排序 
---
&emsp;&emsp;本文利用Collections.sort()对泛型为String的List排序  

练习：利用Collections.sort()方法对泛型为String的List进行排序，要求：  
1. 创建完List<String>之后，往其中添加十条随机字符串  
2. 每条字符串的长度为10以内的随机整数  
3. 每条字符串的每个字符都为随机生成的字符，字符可以重复  
4. 每条随机字符串不可重复  
 
&emsp;&emsp;以下是实现代码：

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.Random;

/**
 * Created by Wenan Wu on 2016/11/16 0016.
 */
public class Exercise6_5 {
    /**
     * 练习：利用Collections.sort()方法对泛型为String的List进行排序：
     * 1.创建完list<String>之后，往其中添加十条随机字符串
     * 2.每条字符串的长度为10以内的随机整数
     * 3.每条字符串的每个字符都为随机生成的字符，字符可以重复
     * 4.每条随机字符串不可重复
     */


    /**
     * 随机生成长度为10以内整数的字符串方法
     */
    public String getRandomString() {
        String base = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
        Random random = new Random();
        StringBuilder sb = new StringBuilder();
        int length; // 字符串长度值
        do { //确保字符串长度不为0
            length = random.nextInt(10);
        } while (length==0);
        for (int i = 0; i < length; i++) {
            int number = random.nextInt(base.length());
            sb.append(base.charAt(number));
        }
        return sb.toString();
    }


    /**
     * 测试字符串排序
     */
    public void testSort() {
        Exercise6_5 ex1 = new Exercise6_5();
        List<String> stringList = new ArrayList<String>();
        String string;
        for (int i = 0; i < 10; i++) {//确保List中没有重复字符串
            do {
                string = ex1.getRandomString();
            }while (stringList.contains(string));
            stringList.add(string);
            System.out.println("增加的字符串为：" + string);
        }
        System.out.println("------------排序前----------");
        for (String s :
                stringList) {
            System.out.println("元素：" + s);
        }
        Collections.sort(stringList);
        System.out.println("---------排序后---------");
        for (String s :
                stringList) {
            System.out.println("元素：" + s);
        }
    }

    public static void main(String[] args) {
        Exercise6_5 ex = new Exercise6_5();
        ex.testSort();
    }
}

```

