---
title: Java_heap_space_OutOfMemoryError
date: 2017-08-18 20:30:00
tags: [异常,tomcat]
categories: [异常,tomcat]
---
#java.lang.OutOfMemoryError: Java heap space
##问题描述
Tomcat 8.0.45在linux环境下启动错误

##原因分析
JVM堆是指java程序运行过程中JVM可以调配使用的内存空间,主要用于存放Instance。JVM在启动的时候会自动设置Heap size的值，其初始空间(即-Xms)是物理内存的1/64，最大空间(-Xmx)是物理内存的1/4。可以利用JVM提供的-Xmn -Xms -Xmx等选项可进行设置。Heap size 的大小是Young Generation 和Tenured Generaion 之和。在JVM中如果98％的时间是用于GC且可用的Heap size 不足2％的时候将抛出此异常信息。

##解决方案
手动设置Heap size
a.如果tomcat是以bat方式启动的，则如下设置：
修改TOMCAT_HOME/bin/catalina.sh
JAVA_OPTS="-server -Xms1024m -Xmx1024m    -XX:MaxNewSize=256m"