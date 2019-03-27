---
title: 固定IP上网方式
date: 2016-09-20 09:43:49
tags: [linux,网络]
categories: [linux,网络]
---
#固定IP上网方式#

**1. 修改主机名称：/etc/sysconfig/network**  
NETWORKING=yes
HOSTNAME=centos.dm.tsai

**2. 设置网络参数：/etc/sysconfig/network-scripts/ifcfg-eth0**  
请记得，这个ifcfg-eth0需与文件内的DEVICE名称设置相同，并且，在这个文件内的所有设置，基本上就是bash的变量设置规则

**[root@linux ~]# vi /etc/sysconfig/network-scripts/ifcfg-eth0**   
DEVICE=eth0                    网卡代号，需要ifcfg-eth0相对应   
BOOTPROTO=static			   开机协议，有dhcp及static,这里是static   
BROADCAST=192.168.1.255        广播地址  
HWADDR=00:40:D0:13:C3:46       网卡地址  
IPADDR=192.168.1.13            IP  
NETMASK=255.255.255.0          子屏蔽网络  
NETWORK=192.168.1.0           网段，该网段的第一个IP  
GATEWAY=192.168.1.2           默认路由  
ONBOOT=yes                  是否开机启动  
MTU=1500                    最大传输单元的设置值  
GATEWAYDEV=eth0            主要路由的设备，通常不用设置  

&emsp;&emsp;请注意每个变量（左边的英文）都应该要大写。否则我们的script会误判。关于IP的4个参数（IPADDR、NETMASK、NETWORK、BROADCAST），下面谈谈以下几个重要的设置值.  
&emsp;&emsp;DEVICE: 这个设置后面接的是设备代号必须与文件名（ifcfg-eht0）的设备代号相同，否则会显示找不到设备名称。
&emsp;&emsp;BOOTPROTO：启动该网络接口时，使用何种协议？如果是手动设置IP的环境，请输入static或none，如果是自动取得IP的情况，请输入dhcp。  
&emsp;&emsp;GATEWAY：代表的是整个主机系统的Default Gateway，所以，设置这个项目时，**请特别留意。不要有重复设置的情况发生。**也就是说，当您有ifcfg-eth0、Ifcfg-eht1等多个文件时，只要在其中一个文件里设置GATEWAY即可。  
&emsp;&emsp;GATEWAYDEV：如果您不是使用固定的IP作为Gateway，而是使用网络设备作为Gateway（通常Route最常有这样的设置），那也可以使用GATEWAYDEV来设置通信网关设备。不过这个设置项目很少使用。  
&emsp;&emsp;HWADDR：这是网卡的卡号。记得以前常常在讲，如果有两块一模一样的网卡存在，例如在一台主机上安装两张RealTek网卡，由于是相同的芯片，所以/etc/modprobe.conf内无法指定出明确的eth0与eth1的对应（因为模块使用相同），那么哪一个才是eth0?利用HWADDR指定网卡的卡号，就能够清楚定义出不同网卡的代号了。  
&emsp;&emsp;事实上，如果想了解每个变量的项目意义时，建议参考/sbin/ifup这个script的内容，script很清楚地记录了每个项目的应用。

**3. 启动与关闭网卡：ifup/ifdown**
启动与关闭网卡的方式有两种，下面分别介绍：
[root@linux~]#ifup eth0  
[root@linux~]#ifdown eth0 

上面的做法是针对eth0来进行启动(ifup)与关闭(ifdown)
[root@linux~]# /etc/init.d/network restart

针对这台主机的所有网络接口(包含lo)与通信闸进行重新启动所以网络会停止再连接

[root@linux~]#service network restart;

**4. 设置DNS的IP： /etc/resolv.conf**
这个文件会影响到您是否可以查询到主机名称与IP的对应。通常进行如果设置就可以了。
nameServer 168.95.1.1

