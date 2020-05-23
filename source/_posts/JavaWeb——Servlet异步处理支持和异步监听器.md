---
title: JavaWeb——Servlet异步处理支持和异步监听器
date: 2017-05-12 11:12:25
categories: Java
tags: Java
description: Servlet的异步支持是Servlet3.0中的新特性，它允许Servlet开启一个新的线程来异步处理业务，本文通过一个示例来介绍该特性。
---
&emsp;&emsp;Servlet的异步支持是Servlet3.0中的新特性，它允许Servlet开启一个新的线程来异步处理业务，本文通过一个示例来介绍该特性。
## Servlet异步处理支持
&emsp;&emsp;Servlet3.0中加入了线程的异步支持，在增加异步支持以前，Servlet的工作流程大致如下：  
  
1. 接收请求，对请求进行一些预处理。  
2. 调用业务逻辑接口进行完成业务的处理。  
3. 根据处理结果提交响应。  

&emsp;&emsp;通常在第二步会消耗大量处理时间，因为Servlet线程会等待业务方法处理完毕。  
&emsp;&emsp;在Servlet3.0中增加了异步支持之后，允许Servlet创建新的线程执行业务逻辑，使得Servlet在启动异步线程后自身可以立即返回容器。
## 异步监听器
&emsp;&emsp;Servlet3.0同时增加了异步监听器，该监听器实现AsyncListener接口，该监听器提供了四个方法：  

1. onStartAsync(AsyncEvent event) 异步线程开始时调用；  
2. onError(AsyncEvent event) 异步线程出错时调用
3. onTimeout(AsyncEvent event) 异步线程执行超时时调用  
4. onComplete(AsyncEvent event) 异步线程执行完毕时调用  


## 异步Servlet和异步监听器使用步骤
&emsp;&emsp;具体的异步Servlet和监听器使用共分为三步：  

1. 获得异步上下文  
2. 注册监听器
3. 开启异步子线程，子线程执行业务方法  

&emsp;&emsp;下面用一个示例来展示二者的使用：

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
  <head>
    <title>异步Servlet和Listener示例</title>
  </head>
  <body>
  <a href="/AsyncServlet">访问异步Servlet</a>
  </body>
</html>

```

```java
package servlet;

import javax.servlet.AsyncContext;
import javax.servlet.AsyncEvent;
import javax.servlet.AsyncListener;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * Created by wuwenan on 12/05/2017.
 */
@WebServlet(name = "AsyncServlet",
        urlPatterns = "/AsyncServlet",
        //开启异步支持
        asyncSupported = true
)
public class AsyncServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=utf-8");
        System.out.println("Servlet开始时间：" + new SimpleDateFormat("YYYY-MM-dd HH:mm:ss").format(new Date()));

        //step1:获得异步上下文
        AsyncContext ctx = request.startAsync();
        //step2:注册监听器
        ctx.addListener(new AsyncListener() {
            @Override
            public void onComplete(AsyncEvent asyncEvent) throws IOException {
                System.out.println("onComplete...");
            }

            @Override
            public void onTimeout(AsyncEvent asyncEvent) throws IOException {
                System.out.println("onTimeout...");
            }

            @Override
            public void onError(AsyncEvent asyncEvent) throws IOException {
                System.out.println("onError...");
            }

            @Override
            public void onStartAsync(AsyncEvent asyncEvent) throws IOException {
                System.out.println("onStartAsync...");
            }
        });
        //step3:开启异步子线程，子线程执行业务方法
        new Thread(new Executor(ctx)).start();

        System.out.println("Servlet结束时间：" + new SimpleDateFormat("YYYY-MM-dd HH:mm:ss").format(new Date()));
    }

    //子线程类
    class Executor implements Runnable{
        private AsyncContext ctx;

        Executor(AsyncContext ctx) {
            this.ctx = ctx;
        }

        @Override
        public void run() {
            try {
                System.out.println("业务开始时间："+new SimpleDateFormat("YYYY-MM-dd HH:mm:ss").format(new Date()));

                //线程等待十秒钟，以模拟业务方法执行
                Thread.sleep(10000);

                System.out.println("业务结束时间："+new SimpleDateFormat("YYYY-MM-dd HH:mm:ss").format(new Date()));

                //关闭异步上下文
                ctx.complete();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        }
    }

}

```

&emsp;&emsp;执行结果如下所示：  

Servlet开始时间：2017-05-12 11:49:54
Servlet结束时间：2017-05-12 11:49:54
业务开始时间：2017-05-12 11:49:54
业务结束时间：2017-05-12 11:50:04
onComplete...

