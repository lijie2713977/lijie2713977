---
title: 分布式存储 
date: 2017-04-16 23:43:49
tags: [分布式,分布式存储]
categories: [分布式,分布式存储]
---


##Bigtable
Bigtable是非关系型数据库，是一个稀疏的、分布式的、持久化存储的多维度排序map。Bigtable设计的目的是快速且可靠地处理PB级别的数据，并且能够部署到上千台机器上。

Bigtable是闭源的，Cloud Bigtable是Google提供的大数据存储云服务。业界相关的Bigtable模型的开源实现为Apache HBase。

##HBase
HBase是一个高可靠、高性能、面向列、可伸缩的分布式存储系统，利用HBase技术可以在廉价PC上搭建起大规模结构化存储集群。
HBase是Google Bigtable的开源实现，类似于Google Bigtable利用GFS作为其文件存储系统，HBase利用Hadoop HDFS作为其文件存储系统；Google运行MapRecue来处理Bigtable中的海量数据，HBase同样利用Hadoop MapRecue来处理HBase中的海量数据；Google Bigtable利用Chubby作为协同服务，HBase利用Zookeeper作为对应。
###特性：
强一致性读写：HBase不是“Eventual Consistentcy（最终一致性）”数据存储，这让它很适合高速计数聚合类任务；
自动分片（Automatic sharding）：HBase表通过region分布在集群中，数据增长时，region会自动分割并重新分布；
RegionServer自动故障转移
Hadoop/HDFS集成：HBase支持开箱即用HDFS作为它的分布式文件系统；
MapRecue：HBase通过MapRecue支持大并发处理；
Java客户端API：HBase支持易于使用的Java API进行编程访问;
Thrift/REST API：HBase也支持Thrift和Rest作为非Java前端访问；
Block Cache和Bloom Filter：对于大容量查询优化，HBase支持Block Cache和Bloom Filter;
运维管理：HBase支持JMX提供内置网页用于运维。
###HBase应用场景
HBase不适合所有场景。
首先，确信有足够多数据，如果有上亿或上千亿行数据，HBase是很好的备选。如果只有上千或上百万行，则用传统的RDBMS可能是更好的选择。因为所有数据如果只需要在一两个节点进行存储，会导致集群其他节点闲置。
其次，确信可以不依赖于RDBMS的额外特性。例如，列数据类型、第二索引、事务、高级查询语言等
最后，确保有足够的硬件。因为HDFS在小于5个数据节点时，基本上体现不出来它的优势。
虽然HBase能在单独的笔记本上运行良好，但这应仅当成是开发阶段的配置 。
###HBase的优点
列可以动态增加，并且列为空就不存储数据，节省存储空间；
HBase可以自动切分数据，使得数据存储自动具有水平扩展功能；
HBase可以提供高并发读写操作的支持；
与Hadoop MapRecue相结合有利于数据分析；
容错性；
版权免费；
非常灵活的模式设计（或者说没有固定模式的限制）；
可以跟Hive集成，使用类SQL查询；
自动故障转移；
客户端接口易于使用；
行级别原子性，即PUT操作一定是完全成功或者完全失败。
###HBase的缺点
不能支持条件查询，只支持按照row key来查询；
容易产生单点故障（在只使用一个HMaster的时候）；
不支持事务；
JOIN不是数据库层支持的，而需要用MapRecue；
只能在主键上索引和排序；
没有内置的身份和权限认证；
###HBase与Hadoop/HDFS的差异
HDFS是分布式文件系统，适合保存大文件。官方宣称它并非普通用途的文件系统，不提供文件的个别记录的快速查询。另一方面，HBase基于HDFS，并能够提供大表的记录快速查询和更新。HBase内部将数据放到索引好的“StoreFiles”存储文件中，以便提供高速查询，而存储文件位于HDFS中。


##Cassandra
Cassandra是Facebook于2008年7月在Google Code上开源的项目。Cassandra实现了Dynamo风格的副本复制模型和没有单点失效的架构，增加了更加强大的column family数据模型。


##Memcached
Memcached可以更好利用内存


##Redis
Redis是一个key-value模型的内在数据存储系统。


##MongoDB
MongoDB是一个介于关系型数据库和非关系性数据库之间的产品，是非关系型 数据库中功能最丰富、最像关系型 数据库的，旨在为Web应用提供可扩展的高性能数据存储解决方案。













