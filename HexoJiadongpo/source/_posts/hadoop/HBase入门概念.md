---
title: HBase入门概念
date: 2017-05-02 10:30:00
tags: [分布式,hbase]
categories: [分布式,hbase]
---
#Hbase概念
HBase是一个分布式的、面向列的开源数据库。  

##Hbase术语
**行键Row Key**：主键是用来检索记录的主键，访问hbasetable中的行。  
**列族Column Family**：Table在水平方向有一个或者多个ColumnFamily组成，一个ColumnFamily中可以由任意多个Column组成，即ColumnFamily支持动态扩展，无需预先定义Column的数量以及类型，所有Column均以二进制格式存储，用户需要自行进行类型转换。  
**列column**：由Hbase中的列族ColumnFamily + 列的名称（cell）组成列。  
**单元格cell**：HBase中通过row和columns确定的为一个存贮单元称为cell。  
**版本version**：每个 cell都保存着同一份数据的多个版本。版本通过时间戳来索引。  


#Hbase安装
三种方式：单机、伪分布式、分布式
##单机模式
Hbase安装文件下载解压后，直接运行，在单机模式下HBase不使用HDFS。
##伪分布式
运行在单个节点上的分布式模式  
##全分布式
全分面式模式下的HBase集群需要ZooKeeper实例运行，并且需要所有的HBase节点都能够与ZooKeeper实例通信，默认情况下HBase自身维护着一组默认的ZooKeeper实例，不过用户也可以配置独立的ZooKeeper实例，这样能够使HBase更加健壮。

#运行HBase
##单机模式
start-hbase.sh
stop-hbase.sh
##伪分分面式
由于伪分布式运行基于HDFS，因此在期待运行HBase之前首先需要启动HDFS。
start-dfs.sh
然后start-hbase.sh
##全分布式
与伪分布式相同

#Hbase Shell
Hbase Shell提供了HBae命令，可以方便创建、删除及修改表，还可以向表中添加数据、列出表中相关信息等。  
在启动hbase之后，用户可以通过下面的命令进入Hbase Shell：  
hbase shell   
输入help获取帮助    
alter:修改列族模式  
count:统计表中行的数量  
create：创建表  
describe:显示表相关的详细信息  
delete:删除指定对象的值（可以为表、行、列对应的值，另外也可以指定时间戳的值）  
deleteall:删除指定行的所有元素值  
disable:使表无效  
drop:删除表  
enable:使表有效  
exists：测试表是否存在  
exit:退出Hbase Shell  
get:获取行或单元(cell)的值  
incr:增加指定表、行或列的值  
list:列出HBase中存在的所有表  
put:向指定的表单元添加值  
tools:列出HBase所支持的工具  
scan：通过对表的扫描来获取对应的值  
status:返回HBase集群的状态信息  
shutdown:关闭HBase集群  
truncate:重新创建指定表  
version:返回HBase版本信息  
下面介绍几个详细的：  
（1）create  
通过表名及用逗号做好事开的列族信息来创建表    
1）hbase>create 't1',{NAME=>'f1',VERSIONS=>5}  
2)hbase>create 't1',{NAME=>'f1'},{NAME=>'f2'},{NAME=>'f3'}  
hbase>#上面的命令可以简写为下面所示的格式：  
hbase>create 't1','f1','f2','f3'  
3)hbase>create 't1',{NAME='f1',VERSIONS=>1,TTL=>2592000,BLOCKCACHE=>true}  
以"NAME=>'f1'举例说明，其中，列族参数的格式是箭头左侧为参数变量，右侧为参数对应的值，并用“=>”分开。  

（2）list  
列出HBase中包含的表名称  
hbase>list   

(3)put  
向指定的HBase表单元添加值，例如向表t1的行r1、列c1:1添加值v1，并指定时间戳为ts的操作如下：  
hbase>put 't1','r1','c1:1','value',ta1  

(4)scan  
获取指定表的相关信息，可以通过逗号分隔来指定扫描参数  
例如：获取表test的所有值  
hbase>scan 'test'  
获取表test的c1列的所有值  
hbase>scan 'test',{COLUMNS=>'c1'}   
获取表test的c1列的前一行的所有值   
hbase>scan 'test',{COLUMNS=>'c1',limit=>1}  

(5)get  
获取行或单元的值，此命令可以指定表名、行值、以及可选的列值和时间戳。   
获取表test行r1的值  
hbase>get 'test','r1'  
获取表test行r1列c1:1的值  
hbase>get 'test','r1'{COLUMN=>'c1:1'}  
需要注意的是，COLUMN和COLUMNS是不同的，scan操作中的COLUMNS指定的是表的列族，get操作中的COLUMN指定的是特定的列，COLUMN的值实质上为“列族+：+列修饰符”。  
另外，在shell中，常量不需要用引号引起来，但二进制的值需要用双引号引起来，而其他值则用单引号引起来。  
HBase Shell的常量可以通过shell中输入“Object.constants”命令来查看。  




