---
title: jdk环境变量配置 
date: 2017-04-18 16:43:49
tags: [java]
categories: [java]
---
# jdk环境变量配置 

jdk环境变量配置 
进行java开发，首先要安装jdk，安装了jdk后还要进行环境变量配置：
1、下载jdk（http://java.sun.com/javase/downloads/index.jsp），我下载的版本是：jdk-6u14-windows-i586.exe
2、安装jdk-6u14-windows-i586.exe
3、配置环境变量：右击“我的电脑”-->"高级"-->"环境变量"
1）在系统变量里新建JAVA_HOME变量，变量值为：C:\Program Files\Java\jdk1.6.0_14（根据自己的安装路径填写）
2）新建classpath变量，变量值为：.;%JAVA_HOME%\lib;%JAVA_HOME%\lib\tools.jar
3）在path变量（已存在不用新建）添加变量值：%JAVA_HOME%\bin;%JAVA_HOME%\jre\bin（注意变量值之间用“;”隔开）
4、“开始”-->“运行”-->输入“javac”-->"Enter"，如果能正常打印用法说明配置成功！
补充环境变量的解析:
JAVA_HOME:jdk的安装路径
classpath:java加载类路径，只有类在classpath中java命令才能识别，在路径前加了个"."表示当前路径。
path：系统在任何路径下都可以识别java,javac命令。



