---
title: win10上PyCharm选择本地docker镜像中解释器报NullPointerException解决方案
categories:
  - Python
  - Docker
tags:
  - Python
  - Docker
mathjax: true
description: 在win10中使用PyCharm进行Python开发，通过Docker搭建本地开发环境会比较方便。我在这个过程中遇到了一个纠结了很久的问题，docker-machine启动后，在PyCharm中选择项目解释器时会报异常，同时PyCharm中会显示docker-machine没有运行。
date: 2018-09-13 19:28:57
---
&emsp;&emsp;在win10中使用PyCharm进行Python开发，通过Docker搭建本地开发环境会比较方便。我在这个过程中遇到了一个纠结了很久的问题，docker-machine启动后，在PyCharm中选择项目解释器时，会报NullPointerException异常，同时PyCharm中会显示docker-machine没有运行。问题的现象如下，在PyCharm中选择Docker Machine时，会上报如下图的异常：  

![选择Docker Machine异常][1]  

而在选择TCP socket时，会上报如下图的异常：  

![选择TCP socket异常][2]  

&emsp;&emsp;这个问题只会在windows上出现，mac上不会存在，我困扰了好几天，加起来大概有10个小时左右，在网上各种搜索也没有找到类似问题，小伙伴有的觉得是path路径写错，有的觉得是docker安装有问题。。。最后我在导师的指点下找到了解决方案，那就是：**以管理员身份运行PyCharm**。  
&emsp;&emsp;对于win10，我还能说啥呢（摊手）。

[1]:http://odnk9as2f.bkt.clouddn.com/win10%E4%B8%8Apycharm%E6%97%A0%E6%B3%95%E8%BF%9E%E6%8E%A5%E6%9C%AC%E5%9C%B0docker1.png  
[2]:http://odnk9as2f.bkt.clouddn.com/win10%E4%B8%8Apycharm%E6%97%A0%E6%B3%95%E8%BF%9E%E6%8E%A5%E6%9C%AC%E5%9C%B0docker2.png