---
title: 磁盘分区与Ext2文件系统
categories:
  - 系统架构
tags:
  - 系统架构
mathjax: true
description: 磁盘分区与Linux的Ext2文件系统简介。
date: 2018-06-1 21:18:17
---

# 磁盘分区与文件系统结构
&emsp;&emsp;磁盘分区的逻辑结构以及磁盘分区与Ext2文件系统的关系简图如下图所示：

![磁盘分区与Ext2文件系统关系][1]  

注意以下几点：  
  * 每个分区都有一个启动扇区（boot sector）。  
  * superblock一般只在每个分区的第一个block group中有，后续的block group中不一定有，即使有，也主要是作为第一个block group中superblock的备份。
  * 每个文件和目录都对应一个inode以及一些block。
  * 目录的inode中记录该目录的相关权限和属性，并记录分配到的block号码；目录的block中记录该目录下的**文件名**和该文件名占用的**inode号码**。  
 
# 软硬连接  

  * 硬连接（hard link）：多个文件名连接到同一个inode号码关联的记录
  * 软链接/符号连接（symbolic link）：创建一个独立的文件，该文件让数据读取指向它所连接的文件的文件名。数据读取时通过文件名连接到正确的目录，在目录block中找到该文件名对应的inode，最终读取正确数据

[1]:http://odnk9as2f.bkt.clouddn.com/%E7%A3%81%E7%9B%98%E5%88%86%E5%8C%BA%E4%B8%8EExt2%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F1.png