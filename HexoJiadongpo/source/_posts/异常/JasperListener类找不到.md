---
title: JasperListener类找不到
date: 2017-08-18 09:59:00
tags: [异常,tomcat]
categories: [异常,tomcat]
---
#Tomcat java.lang.ClassNotFoundException: org.apache.catalina.core.JasperListener
##问题描述
Linux下启动tomcat8.0.45错误，windons下可用。

##原因分析
ClassNotFoundException大致可能有两种情况，一是找不到jar包，二是jar包冲突，不知用哪个。

##解决方案
JasperListener好像是一个报表支持，目前不需要，可以暂时去掉。
在tomcat目录 /conf/server.xml里注释如下内容
<Listener className="org.apache.catalina.core.JasperListener" />   