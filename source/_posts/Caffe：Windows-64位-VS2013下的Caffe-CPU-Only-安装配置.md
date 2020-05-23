---
title: Caffe：Windows(64位)+VS2013下的Caffe(CPU Only)安装配置
date: 2017-04-01 03:27:21
description: 本文为Windows(64位)+VS2013下的Caffe(CPU Only)安装配置过程记，并非安装教程。
categories: 机器学习
tags: 机器学习
---
# 一、环境准备
### windows 10 64位 专业版（非必须）
### visual studio 2013（墙裂推荐此版本）
1. 笔者使用的操作系统为win10 64位，在其他版本的64位windows系统上应该同样可行。
2. visual studio 2013（以下简称vs2013）中需要安装NuGet，安装方法如下：
a. 打开vs2013，点击“工具”-“扩展和更新”
b. 在弹出的对话框中点击“联机”，在右上角搜索框中搜索NuGet，在搜索结果中“NuGet Package Manager”项上面的“下载”，开始下载NuGet，下载完后进行安装
c. 安装完后重启vs2013，重启后点击“工具”-“扩展和更新”，选择“已安装”，可以看到NuGet Package Manager已经安装好了。
<!-- more -->  

# 二、配置步骤
1. 下载代码 
从https://github.com/Microsoft/caffe  
得到文件“caffe-master.zip”
或从https://github.com/BVLC/caffe/tree/windows 
得到文件“caffe-windows.zip” 
二者的配置步骤相同，均能编译成功，本文以源码“caffe-master.zip”为例

2. 解压后进入如下路径：\caffe-master\windows 之后的操作均在该目录下进行，因此之后省略\caffe-master的路径

3. 在\windows目录下复制文件 CommonSettings.props.example(应该会以副本形式出现CommonSettings.props - 副本.example),并将该副本改名为CommonSettings.props。(请确认显示文件扩展名这个选项已生效)

4. 用vs（或其他编辑器）打开CommonSettings.props，将第7行`CpuOnlyBuild`标签中的值改为true，将第8行`UseCuDNN`标签中的值改为false，更改完毕后保存并退出。
（这两步是将caffe的GPU版本关闭，仅使用CPU版本。由于打开GPU会出现其他错误，作为初学者，我们先从CPU版本开始，等熟悉caffe之后再深入研究GPU版本）

5. 用vs打开\windows下的Caffe.sln，可以看到该解决方案中共有16个项目，请注意核对。（若下载的源码为BVLC的“caffe-windows.zip”，则该解决方案中只有15个项目，所缺少的项目为“caffe.managed”，但经过笔者测试，在编译时并没有影响）右击“解决方案‘Caffe’”，选择“属性”，将“配置属性”-“配置”修改成Release和x64，如下图所示
![这里写图片描述](http://odnk9as2f.bkt.clouddn.com/caffe1.png?imageView/2/w/500)
（这一步是使用Release来进行编译，若用Debug，则之后每次都要打开vs，会不方便）
注意：在上图顶部工具栏中的“解决方案配置”和“解决方案平台”框，若你的vs2013中将这两个框在工具栏中显示，则要在工具栏中将配置改成Release和x64，否则直接右击“解决方案Caffe”来更改配置是无效的。

6. 右击解决方案中的libcaffe项目，选择“属性”，在打开的属性页中选择“C/C++”-“常规”，将“将警告视为错误”设为“否”，如下图所示：
![这里写图片描述](http://odnk9as2f.bkt.clouddn.com/caffe2.png?imageView/2/w/500)

7. 右击“libcaffe”项目，选择“生成”，之后是一段时间的等待。
注意：点击“生成”后会出现如下图的窗口，此时vs正在使用NuGet对caffe的一些依赖文件进行自动还原
![这里写图片描述](http://odnk9as2f.bkt.clouddn.com/caffe3.png?imageView/2/w/500)
还原成功后，会在caffe-master的同级目录生成文件夹NugetPackages。

8. 右击“解决方案Caffe”，选择“生成解决方案”，之后又是一段时间的等待

9. 等待过后生成成功，到此windows下的caffe配置完成，此时在\caffe-master目录下会生成Build文件夹，即为我们编译成功的文件夹，而\caffe-master\Build\x64\Release目录下则会有我们编译出的caffe.exe执行文件，到此caffe配置完成。



主要参考
1. http://www.cnblogs.com/yixuan-xu/p/5858595.html
2. http://m.blog.csdn.net/article/details?id=51355143
3. http://blog.csdn.net/qq_14845119/article/details/52415090



