---
title: FreeMarker模板引擎实战
categories:
  - Web开发
tags:
  - Java
  - Web开发
  - 框架
mathjax: true
description: FreeMarker是一个模板引擎：它是一个Java库，基于模板和要改变的数据生成输出文本（HTML网页、电子邮件、配置文件、源代码等）。本文通过一个小demo来说明FreeMarker的使用方式。
date: 2017-09-04 15:40:05
---

&emsp;&emsp;FreeMarker是一个模板引擎：它是一个Java库，基于模板和要改变的数据生成输出文本（HTML网页、电子邮件、配置文件、源代码等）。  
&emsp;&emsp;笔者在项目开发过程中遇到了FreeMarker的使用场景，因此阅读了官方的文档。本文不会记录FreeMarker各种表达式、标签和指令的使用，更加侧重于模板引擎整体的使用。

## 基本概念和术语  
&emsp;&emsp;如官方手册中所言，`模板+数据模型=输出`。在FreeMarker中，首先要定义一个文件模板，如html，将其中需要动态传值的数据通过FreeMarker中的表达式、标签或自定义标签替换。再定义数据模型，通常是通过Java中的类来封装数据。使用时后台会读取模板和数据模型，将数据嵌入模板，然后生成新的html页面，提供给我们静态化访问。  
### 数据模型详解 
&emsp;&emsp;FreeMarker中的数据模型是一个树形的结构，如下所示:  
<pre> 
(root)
  |
  +- animals
  |   |
  |   +- mouse
  |   |   |
  |   |   +- size = "small"
  |   |   |
  |   |   +- price = 50
  |   |
  |   +- elephant
  |   |   |
  |   |   +- size = "large"
  |   |   |
  |   |   +- price = 5000
  |   |
  |   +- python
  |       |
  |       +- size = "medium"
  |       |
  |       +- price = 4999
  |
  +- message = "It is a test"
  |
  +- misc
      |
      +- foo = "Something"
</pre>  

&emsp;&emsp;其中有一些术语，上图中的根节点、mouse、elephant、python、misc这些非叶子结点，在上图中类似于一个目录，它们被称为**hash**，它们保存其他的叶子变量。  
&emsp;&emsp;上图中保存单个值的变量，如size、price、message、foo，被称为**scalar**，scalar支持的数据类型为：String、数字、日期时间、布尔值。要读取scalar，可以通过“.”符号，如要读取上图中的foo，可以写成misc.foo。  
&emsp;&emsp;如果上图中的animals目录下的mouse、elephant、python均没有具体的名字，则animals被称为**sequences**，sequence下的目录均为hash，但它们没有具体的名字，表示存在于一个列表中的项，可以通过索引来取到，如要取上图中的animals.python.size，也可以通过animals[2].size取到，就像Java中的列表一样。  
## 模板引擎的使用
### 使用步骤
&emsp;&emsp;使用FreeMarker主要有以下几个步骤：

1. 创建freemarker.template.Configuration类的实例并设置其属性。Configuration类的实例是用来存储freemarer应用级设置的核心容器，它负责创建和捕获预解析的模板（即Template对象）。  
2. 创建数据模型，数据模型中的hash（用来存储其他有名字的数据项）可以是Map也可以是JavaBeans，只要它拥有public的getXxx或isXxx方法。
3. 获取模板文件，模板由freemarker.template.Template类的实例所表示，Template类实例通过Configuration类实例的getTemplate()方法获得。Template实例用一种解析过后的格式来存储模板，而非纯文本。如果模板文件丢失或语法错误，则getTemplate()方法将会抛出异常。Configuration实例容器会缓存Template实例，下次通过getTemplate()获取同一个模板时，不会重新解析，而是返回缓存中的同一个Template实例。  
4. 将数据模型合并至模板中，并产生输出文件。通过Template实例对象的process()方法。一个Template实例对象一旦创建出来，它可以与多个数据模型进行多次合并。  

### 代码示例
&emsp;&emsp;下面就对应上面相应的步骤，给出代码示例：
&emsp;&emsp;主流程类Test.java：  

```java
package cn.edu.whu.test;

import freemarker.template.Configuration;
import freemarker.template.Template;
import freemarker.template.TemplateExceptionHandler;

import java.io.File;
import java.io.FileOutputStream;
import java.io.OutputStreamWriter;
import java.io.Writer;
import java.util.HashMap;
import java.util.Map;

/**
 * @author wawu
 * @create 2017/9/4
 */
public class Test {
    public static void main(String[] args) throws Exception {

        /* ------------------------------------------------------------------------ */
        /* Step1：实例化Configuration类并设置其属性。
            注意：
            1. 整个应用周期中只需要实例化一次
            2. Configuration类对象是单例的
        */
        Configuration cfg = new Configuration(Configuration.VERSION_2_3_25);
        cfg.setDirectoryForTemplateLoading(new File("/Users/wuwenan/Development/IdeaProjects/FreemarkerTest/src/main/resources"));
        cfg.setDefaultEncoding("UTF-8");
        cfg.setTemplateExceptionHandler(TemplateExceptionHandler.RETHROW_HANDLER);
        cfg.setLogTemplateExceptions(false);

        /* ------------------------------------------------------------------------ */
        /* Step2：创建数据模型
            注意：
            1. 在应用的生命周期中，可能会需要创建多次，每一个模板需要对应一个数据模型
        */
        Map<String, Object> root = new HashMap<>();
        root.put("user", "Big Joe");
        Product latest = new Product();
        latest.setUrl("products/greenmouse.html");
        latest.setName("green mouse");
        root.put("latestProduct", latest);

        /* Step3：从文件获得模板，文件为test.ftl*/
        Template temp = cfg.getTemplate("test.ftl");

        /*Step4：将数据模型合并至模板中*/
        Writer out = new OutputStreamWriter(new FileOutputStream("/Users/wuwenan/Development/IdeaProjects/FreemarkerTest/src/main/resources/test.html"));
        temp.process(root, out);

        /*注意：根据out对象的不同，可能需要调用out.close()方法（通常输出为文件时需要调用），
            但在Web应用中不需要调用*/
        out.close();
    }
}

```

&emsp;&emsp;代码中用到的模板test.ftl：

```html
<!DOCTYPE html>
<html>
<head>
    <title>Welcome!</title>
</head>
<body>
<h1>Welcome ${user}!</h1>
<p>Our latest product:
    <a href="${latestProduct.url}">${latestProduct.name}</a>!
</body>
</html>
```

&emsp;&emsp;代码中用到的实体类Product.java：

```java
package cn.edu.whu.test;

/**
 * @author wawu
 * @create 2017/9/4
 */
public class Product {

    private String url;
    private String name;

    // As per the JavaBeans spec., this defines the "url" bean property
    // It must be public!
    public String getUrl() {
        return url;
    }

    public void setUrl(String url) {
        this.url = url;
    }

    // As per the JavaBean spec., this defines the "name" bean property
    // It must be public!
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

}
```


