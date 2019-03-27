---
title: Hive入门概念
date: 2017-05-02 10:30:00
tags: [大数据,hive]
categories: [大数据,hive]
---
#Hive
大数据生态下，通过Hadoop MapReduce，实现将计算分割成多个处理单元，然后分散到一群家用或服务器级别的硬件上，从而降低成本并提供可伸缩性；这个计算模型下是HDFS，这是个“可插拔的“文件系统。不过，这里存在一个问题，就是用户如何从一个现有的数据基础架构转移到Hadoop上，而这个基础架构是基于关系型数据库和结构化查询语句（SQL）？
这就是Hive出现的原因，Hive提供了被称为Hive查询语言的（或称为HiveQL或HQL）的SQL方言，来查询存储在Hadoop集群中的数据。Hive将大多数据的查询转换为MapRecue任务（ｊｏｂ）。

#Hive安装
Hive使用环境变量HADOOP_HOME来指定Hadoop的所有相关JAR和配置文件，因此在安装之前请确认下是否设置好了这个环境变量。
$cd ~
$curl -o http://archive.apache.org/dis/hive/hive-0.9.0/hive-0.9.0-bin.tar.gz
$tar -xzf hive-0.9.0.tar.gz
$sudo mkdir -p /user/hive/warehouse
$sudo chmod a+rwx /user/hive/warehouse

可以定义HIVE_HOME环境变量
$sudo echo "export HIVE_HOME=$PWD/hive-0.9.0" > /etc/profile.d/hive.sh
$sudo echo "PATH=$PATH:$HIVE_HOME/bin" >> /etc/profile.d/hive.sh
$. /etc/profile

#Hive组成
主要包含三个部分：
1.代码本身，在$HIVE_HOME/lib下可以看到许多jar，例如hive-exec*.jar，hive-metastore*.ja，每个jar文件都实现了hive功能中某个特定的部分。
2.可执行文件，在$HIVE_HOME/bin下，包含hive的命令行界面CLI，CLI是使用hive最常用的方式，一般会使用小写的hive代替。CLI用于提供交互式的界面供输入语句或用户执行hive语句的脚本。
3.metastoreservice（元数据服务），所有的hive客户端都需要元数据服务，hive使用这个服务来存储表模式信息和其他元数据信息。通常会使用关系型数据库来存储这些信息，默认使用内置的DerbySQL服务器，其可以提供有限的、单进程的存储服务。例如，当使用Derby时，用户不能执行2个并发的Hive CLI实例，然而，如果是在个人计算机上或某些开发任务上使用的话这样也没有问题。对于集群来说，需要使用MYSQL或类似的关系型数据库。
另外，hive还有一些组件，Thrift服务提供可远程访问的其他进程的功能，也提供JDBC和ODBC访问Hive的功能。Hive还提供了一个简单的网页界面HWI，提供远程访问Hive服务。

#Hive启动
使用$HIVE_HOME/bin/hive命令
$cd $HIVE_HOME
$bin/hive
hive>CREATE TABLE x (a INT);
hive>SELECT * from x;
hive>DROP TABLE x; 
hive>exit;

#Hive命令
[root@cdhmaster~]#hive--help  
Usage./hive<parameters>--serviceserviceName<serviceparameters>  
ServiceList:beelinecleardanglingscratchdirclihelphiveburninclienthiveserver2hiveserverhwijarlineagemetastoremetatoolorcfiledumprcfilecatschemaToolversion  
Parametersparsed:  
--auxpath:Auxillaryjars  
--config:Hiveconfigurationdirectory  
--service:Startsspecificservice/component.cliisdefault  
Parametersused:  
HADOOP_HOMEorHADOOP_PREFIX:Hadoopinstalldirectory  
HIVE_OPT:Hiveoptions  
Forhelponaparticularservice:  
./hive--serviceserviceName--help  
Debughelp:./hive--debug--help  
Youhavenewmailin/var/spool/mail/root  
需要注意ServiceList:后面的内容，这里提供了几个服务，包括我们绝大多数据时间将要使用的CLI。用户可以通过--servicename服务名称来启用某个服务。  

#常用SQL
显示数据库  
hive>showdatabases;  
OK  
Default  
hive>showdatabaselike'h.*';  
创建数据库  
hive>createdatabasetest_test001;  
use命令用于将某个数据库设置为用户当前的工作数据库  
hive>usetest_test001;  
设置当前工作数据库后，即可查询所有表  
hive>showtables；  
删除数据库  
hive>dropdatabaseifexiststest_test001;  

创建数据  
createtableifnotexistsmydb.employees(  
namestringcomment'emplyeename',  
Salaryfloat  
)  

删除表  
droptableifexiststest_test001;  

修改表  
altertable只会修改元数据  

表重命名  
altertabletest_test001renametotes;  

set hive.cli.print.header=true; // 打印列名  
set hive.cli.print.row.to.vertical=true; // 开启行转列功能, 前提必须开启打印列名功能  
set hive.cli.print.row.to.vertical.num=1; // 设置每行显示的列数  



