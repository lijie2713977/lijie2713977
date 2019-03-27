---
title: 开源ETL工具-kettle
date: 2017-04-15 14:43:49
tags: [开源项目,kettle]
categories: [开源项目,kettle]
---
#开源ETL工具-kettle
说明：本文部分内容参考网络的资料，如果侵权之处请告知一下，不胜感激！

Kettle是Pentaho公司开发的一款ETL产品，以工作流为核心，强调面向解决方案而非工具的，基于java平台的商业智能(Business Intelligence,BI)套件。Kettle的开源协议是LGPL，该协议来自GNU，因功能强大，被FSF(Free Software Foundation)列为首选协议。LGPL协议允许Kettle作为商业（非开源）代码的链接库，使用Kettle的商业代码无须开源。LGPL带来的不仅是Kettle API，你还可以对它进行拓展对外提供商业软件或服务。
##  ##ETL是什么

ETL早期作为数据仓库的关键环节，负责将分布的、异构数据源中的数据如关系数据、平面数据文件等抽取到临时中间层后进行清洗、转换、集成，最后加载到数据仓库（Data Warehouse）或数据集市（Data Mart）中，成为联机分析处理（On-Line Analytical Processing，OLAP）、数据挖掘（Data Mining）的基础。


Exract：从多种异构数据源中抽取数据

Transform：经过清洗，统一化和转换

Load：将数据加载到目的数据源中



##  ##Kettle产品特点

适用于将多个应用系统的大批量的、异构的数据进行整合，有强大的数据转换功能。
高效适配多种类型的异构数据库、文件和应用系统。
快速构建复杂数据大集中应用、无需编码。

Kettle构成

TODO以Github上的名词进行定义Spoon，Cart等


左边是集成开发工具（Spoon），可以进行流程的开发、配置、调试、部署、执行(转换、任务)，也可以对运行情况进行监控、对处理过程的日志进行查看、也可以通过接口调用方式进行远程管理。

中间是服务器(Carte)，包括实际执行转换和任务的ETL引擎、监控管理的接口、认证授权接口，还有一个可以拓展的接口。

下面是在开发过程中，用于保存集成开发工具中创建的转换、任务、数据库等项的，资源库包含两类，一个是数据库资源库，一个是文件资源库。

右边个是是第三方平台，可以基于kettle提供的接口实现相应的功能包括状态监控、启停控制、日志查看等功能。


组成部分
名称
描述
Spoon
一个基于swt开发的流式处理客户端，用户开发转换、任务、创建数据库、集群、分区等
Pan
一个独立的命令行程序，支持通过命令行实现界面的功能，如果转换启停、任务启停。状态查看等
Kitchen
一个独立的命令行程序，用于执行由Spoon编辑的作业。
Carte
Carte是一个轻量级的Web容器，用于建立专用、远程的ETL Server。


PDI相关术语和概念
Job(任务)、Transformation(转换)是kettle的两个最重要的概念。任务做的一件完整的事，包含开始、结束等整个生命周期；而转换是要做这件事的某一个小的功能。比如你要从A数据源中解析数据后放入B数据源，那么你可以创建两个转换，一个是从A数据源加载数据->处理数据->放入存储中；另一个是把数据放入B数据源，然后在一个任务中处理他们。

下面我们通过集成开发工具去了解一个转换和任务

Transformation（转换）
Transformation（转换)是由step(步骤)和hops(节点连接线)组成，一个转换，可以看成一段数据流，每一个步骤完成一项数据处理的工作，节点连接线用于数据的流动。

转换可以单独运行完成某一项工作，文件的扩展名为.ktr

Steps（步骤）
Steps（步骤）是转换的重要组件部分，在Spoon中步骤根据功能分为输入类、输出类、脚本类等，每一个步骤完成一种特定的功能，比如excel输出组件，用于把数据流输出为excel文件格式。参考如下：


Hops（节点连接）
Hops（节点连接）是数据传输的通道，用于连接两个步骤，使数据从一个步骤传递到另一个步骤，支持分发、复制等方式。注意数据处理的顺序并不是按照节点连接箭头的顺序，因为第个步骤都是单独的线程。

Jobs（工作）
Jobs（工作）是基于工作流模型的，顺序处理。把步骤、转换组织在一起完成一件完整的事情。
文件扩展名为.kjb


下载使用
kettle下载 目前最新版7.0
https://sourceforge.net/projects/pentaho/files/Data%20Integration/
 
下载解压后是一个如：pdi-ce-7.0.0.0-25的文件，目录内容如下

windown下直接双拼Spoon.bat、linux下直接运行./spoon.sh即可。
注： could not find the main class:org.pentaho.commons.launcher.Launcher. Program will exit. 表示jdk版本错误 。7.0版本只支持jdk1.8，可以单独配置kettle的jdk，添加配置到系统中即可：
名称：PENTAHO_JAVA_HOME
值：C:\Program Files\Java\jdk1.8.0_45 
    mac系统下/Library/Java/JavaVirtualMachines/jdk1.8.0_111.jdk/Contents/Home/
启动后界面如下

新建转换：快捷键Ctrl+N


首先从核心对象区选择“生成记录组件，编辑：


然后选择Excel输出组件到工作区，创建生成记录步骤到Excel输出步骤的连接线，编辑excel输出目录和字段




最后生成如下，点击运行：




运行后的结果是输出excel文件，并可以查看每个步骤的处理情况，读、写、输入、输出等


其它参考链接
kettle源码下载，可以选择各个版本下载，自己编译。
https://github.com/pentaho/pentaho-kettle
大数据插件源码
https://github.com/pentaho/big-data-plugin
kettle支持的大数据环境源码，主要是hdp,cdh。
https://github.com/pentaho/pentaho-hadoop-shims
kettle nexus
http://repo.pentaho.org/content/groups/omni/pentaho/  
http://repository.pentaho.org/artifactory/repo/
所有组件实现说明
http://wiki.pentaho.com/display/EAI/Pentaho+Data+Integration+Steps  
所有组件测试说明
http://wiki.pentaho.com/display/EAI/test 
帮助
http://help.pentaho.com/Documentation 
























