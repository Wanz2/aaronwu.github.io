---
title: VMware虚拟CentOS 6.5在NAT模式下配置静态IP地址及Xshell远程控制配置
date: 2017-04-01 03:29:51
tags: 
	- Linux 
	- 经验心得
categories: Linux
description: 由于研究需要，笔者在Win10下的VMware虚拟机中安装了CentOS6.5，但在CentOS的网络配置上却遇到了一些问题。在此将踩过坑之后的正确配置方法记录一下，希望能够帮助到一些同学。
---
#  一、前言
&emsp;&emsp;由于研究需要，笔者在Win10下的VMware虚拟机中安装了CentOS6.5，但在CentOS的网络配置上却遇到了一些问题。众所周知在主机上通过Xshell工具远程管理虚拟系统相当快捷方便，但在虚拟系统NAT网络模式下笔者的主机却始终ping不通虚拟机，折腾了很久才搞定。在此将踩过坑之后的正确配置方法记录一下，希望能够帮助到一些同学。本文内容主要分为两块：
1. VMware下CentOS6.5在NAT网络模式下静态IP地址的配置
2. Windows主机中配置Xshell工具以远程控制虚拟系统
<!-- more -->

# 二、VMware下CentOS6.5在NAT网络模式下静态IP地址的配置
1. 在VMware中配置NAT网络模式
    1.1 在虚拟机设置中将网络适配器设置为NAT模式
![](http://img.blog.csdn.net/20161015045000229?imageView/2/w/500)
    1.2 在VMware工具栏点击“编辑”-“虚拟网络编辑器”，在弹出的窗口中选择“更改设置”
![](http://img.blog.csdn.net/20161015045052680?imageView/2/w/500)
    1.3 在弹出的窗口中选择VMnet8，并将其VMnet信息选择为NAT模式，点击“NAT设置”
![](http://img.blog.csdn.net/20161015045131242?imageView/2/w/500)
    1.4 在弹出的窗口中，记住“网关IP”，之后配置IP地址会用到。在“端口转发”中选择“添加”
![](http://img.blog.csdn.net/20161015045215555?imageView/2/w/500)
    1.5 在弹出的窗口中进行如下配置，其中主机端口可以自行设定，推荐设定值范围在10000和65535之间；虚拟机IP地址的前三位与上一步中网关地址的前三位相同，表示网段，虚拟机IP地址的最后一位可自行设定；虚拟机端口设置为22，为SSH协议的默认接口；“描述”选项可自行设定。设定结束后选择“确定”，在上一级窗口中选择“应用”并确定，至此VMware中的NAT网络配置结束。
![](http://img.blog.csdn.net/20161015045308234?imageView/2/w/500)
2. 在CentOS中配置静态IP地址
    2.1 打开CentOS，输入
            `ifconfig`
          查看网卡信息，注意信息中的网卡名：
![](http://img.blog.csdn.net/20161015045354993?imageView/2/w/500)
        上图中“eth0”即为网卡名，表示第一块网卡
    2.2 静态IP地址配置主要用到以下三个文件：
            `/etc/sysconfig/network-scripts/ifcfg-eth0`
            `/etc/sysconfig/network`
            `/etc/resolv.conf`
          注意：第一个文件名中的“ifcfg-eth0”可能有所不同，实际使用时根据上一步中查看的网卡名进行设置
    2.3 首先编辑`/etc/sysconfig/network`，配置如下：
![](http://img.blog.csdn.net/20161015045435665?imageView/2/w/500)
          其中GATEWAY即为之前查看到的网关地址
    2.4 其次编辑`/etc/sysconfig/network-scripts/ifcfg-eth0`，若找不到该文件，则说明网卡eth0尚未启动，则可以创建该文件并配置如下：
![](http://img.blog.csdn.net/20161015045508291?imageView/2/w/500)
         其中IP地址为之前在NAT设置中配置的IP地址，NETMASK为子网掩码，GATEWAY即默认网关地址
         注意：DNS1和DNS2必须填写，若不填写则无法进行域名解析，其中DNS1与GATEWAY地址相同，而DNS2则可选择常用的DNS服务器地址，这里使用的8.8.8.8是谷歌的DNS服务器
    2.5 上一步配置完成后，系统会在`/etc/resolv.conf`文件中自动写入DNS服务器地址，如图：
![](http://img.blog.csdn.net/20161015045535228?imageView/2/w/500)
    2.6 配置完成后输入命令`service network restart`重启网络服务，至此静态IP配置完成，虚拟CentOS的IP地址为192.168.81.23
3. 打开ssh服务
    3.1 在配置Xshell前，要在系统中打开ssh服务，输入如下命令
    打开ssh服务：`service sshd start`
    设置开机运行ssh：`chkconfig sshd on`

# 三、配置Xshell工具
1. 打开Xshell工具，选择“文件”-“新建”，在弹出窗口选择“连接”，作如下配置：
![](http://img.blog.csdn.net/20161015045604525?imageView/2/w/500)
    a. 协议选择SSH
    b. 由于已经将主机的10023端口映射为虚拟机的22号端口，因此主机设置为127.0.0.1，端口号设置为步骤1.5中设置的“主机端口号”，即10023
2. 在“用户身份验证”中输入登录用帐号密码：
![](http://img.blog.csdn.net/20161015045633192?imageView/2/w/500)
3. 配置成功，点击连接即可接入虚拟系统