---
title: Intellij IDEA 操作笔记
categories:
  - 经验心得
tags:
  - 经验心得
mathjax: true
description: 本文为笔者在学习和使用Intellij IDEA过程中踩坑的记录。
date: 2017-08-11 22:54:49
---
&emsp;&emsp;Intellij IDEA被认为是目前最优秀的java开发IDE，但其使用习惯与Eclipse存在较大的差异，笔者将自己学习和使用过程中所用到的一些操作记录下来，以便查阅。  

- **IntelliJ导入jar包：**
 - File->Project Structure->Modules->Dependencies->add->JARs or directories  

- **IntelliJ连接 tomcat 服务器：**
 - Edit Configuration->Tomcat Server->add->Deployment->add->Artifact->选择 Module-> 在右边窗口填写 Application context

- **IntelliJ连接 MySQL 数据库：**
 - 导入 MySQL 数据库 jdbc 的 jar 包：File->Project Structure->Modules->Dependencies->add->JARs or directories选择jdbc 驱动 jar 包
 	- Web 项目连接数据库：导入 jdbc的 jar 包->File->Project Structure->Artifacts->将右边的 jar 包拖入左边的WEB-INF中的 lib 文件夹
 
- **IntelliJ查看 Web 项目编译输出地址：**
 - File->Project Structure->Project->查看Project compiler output路径
 
- **IntelliJ导入javax.servlet包：**
 - File->Project Structure->Modules->add->Libraries-> 选择 Tomcat文件夹下的lib文件夹->选择servlet-api.jar
 - Intellij中的web项目，jsp页面里面部分内置对象的方法无法解析，如request.getParameter()，需要导入tomcat下的servlet-api.jar   	 	

- **解决运行Web Server时控制台中文输出乱码问题：**
 - Run/Debug Configurations ->Server->VM options中填写-Dfile.encoding=UTF-8

- **IntelliJ生成javadoc：**
 - 在 “Tools->Gerenate JavaDoc” 面版的 “Other command line arguments:” 栏里输入 “-encoding utf-8 -charset utf-8”以更改生成的javadoc的字符集

- **IntelliJ为java程序输入运行时的参数：**
 - Edit Configurations —> 在Program arguments中输入参数
 - e.g. 在SortCompare 用例类的 Program arguments中输入参数"1000"效果等于在终端输入“java SortCompare 1000”

- **IntelliJ为项目添加JUnit：**
 - 方法一：自定义生成测试类 cmd+shift+T—>create new test…
 - 方法二：使用junit generator生成测试类 control+return—>JUnit Test
 - 方法三：使用junit generator生成测试类 工具栏code—>Generate，然后同方法二

- **IntelliJ运行Maven命令：**
 - 在Maven项目界面右边Maven Projects—>Lifecycle—>选择Maven命令

- **IntelliJ创建Maven管理的Web项目的两种方式：**
 - 第一种：先创建普通maven项目，再增加框架支持
 - 第二种：直接通过archetype创建maven项目（推荐）

- **IntelliJ自己创建命令工具宏：**
 - Preference -> Tools -> External Tools -> “+” -> 编辑各个参数
 - 具体参数内容查看[官方文档](https://www.jetbrains.com/help/idea/create-edit-copy-tool-dialog.html)

