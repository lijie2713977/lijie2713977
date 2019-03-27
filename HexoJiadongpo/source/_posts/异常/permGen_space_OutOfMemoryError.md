---
title: permGen_space_OutOfMemoryError
date: 2017-08-18 09:55:00
tags: [异常,tomcat]
categories: [异常,tomcat]
---
#Tomcat java.lang.OutOfMemoryError: PermGen space
##问题描述
Tomcat 8.0.45在linux环境下启动错误

##原因分析
PermGen space是指内存的永久保存区域内存溢出。这块内存主要是JVM来来存放Class和Meta信息的，Class在被Load的时候被放入PermGen space区域，它和存放Instance的Heap区域不同,GC(Garbage Collection)不会在主程序运行期对PermGen space进行清理，所以如果你的APP会LOAD很多CLASS的话，就很可能出现PermGen space错误。这种错误常见在web服务器对JSP进行预编译的时候。如果你的WEB APP下都用了大量的第三方jar, 其大小超过了jvm默认的大小(4M)那么就会产生此错误信息了。

##解决方案
手动设置MaxPermSize大小,修改TOMCAT_HOME/bin/catalina.sh
JAVA_OPTS="-server -XX:PermSize=256M -XX:MaxPermSize=1024m"   