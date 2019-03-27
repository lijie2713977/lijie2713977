---
title: 分布式SQL查询引擎-Presto
date: 2019-02-01 09:55:00
tags: [Presto]
categories: [大数据技术,Presto]
---

Presto是一个开源的，基于内存的分布式实时计算框架，它出自Facebook，国内大厂现在已有很多应用案例，如：京东、美团、携程等。

## PRESTO应用场景

Presto在大数据量的查询上有很好的性能，并且数据源具有完成解耦、高性能，以及对SQL的支持等特性，使其有很多的应用场景，如ETL、Ad-Hoc查询等。

### ETL

由于Presto支持多种数据源且支持数据源定制化的开发，所以使其在ETL方式有其应用场景，如下。
![Presto应用场景-ETL.jpg](https://upload-images.jianshu.io/upload_images/3109530-96f1cfbdf4a8018e.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过azkaban定时调度Presto从不同数据源同步数据到HDFS，然后通过Hive提供批量的查询，Presto提供实时的查询需要。

### Ad-Hoc查询

Ad-Hoc查询就是即席查询，即席查询允许用户根据自己的需求随时调整和选择查询条件，计算平台或者系统能够根据用户的查询条件返回查询结果或者生成相应的报表。
某公司使用Presto完成Ad-Hoc查询，实际的Ad-Hoc使用场景包括以下两种。
（1）使用BI工具进行报表展现。
BI工具通过ODBC驱动连接至Presto集群，BI工程师使用BI工具进行不同维度的报表设计和展现。

（2）使用Cli客户端进行数据分析
Presto使用Hive作为数据源，对Hive中的数据进行查询和分析。众所周知，hive使用Map-Reduce框架进行计算，由于Map-Reduce的优势在于进行大数据量的批运算和提供强大的集群计算吞吐量，但是对稍小数据量的计算和分析会花费相当长的时间，因此在进行GB-TB级别数据量的计算和分析时，Hive并不能满足实时性要求。
Presto是专门针对基于Ad-Hoc的实时查询和计算进行设计的，其平均性能是Hive的10倍，因此presto更适合于稍小数据量的计算和差异性分析等Ad-Hoc查询。

## 核心概念

### Presto服务进程

Presto集群中一共有两种服务器进程：Coordinator服务进程和Worker服务进程，其中Coordinator服务进程的主要作用是：接收查询请求、解析查询语句、生成查询执行计划、任务调度和Worker管理。而Worker服务进程则执行被分解后的查询执行任务:Task。

1.Coordinator
Coordinator服务进程部署于集群中一个单独的节点上，是整个Presto集群的管理节点。Coordinator服务进程主要用于接收客户端提交的查询，查询语句解析，生成查询执行计划、Stage和Task并对生成的Task进行调度。除此之外，Coordinator还对集群中的所有Worker进行管理。Coordinator进程是整个Presto集群的Master进程，该进程即与Worker进程通信从而获得最新的Worker信息，又与Client进行通信，从而接受查询请求，而所有的这些工作都是通过Coordinator上的StatementResource类提供的RESTful服务来完成的。

2.Worker
在一个Prestor集群中，存在一个Coordinator节点和多个Worker节点。Coordinator节点是管理节点，而Worker节点就是工作节点。在每个Worker节点上都会存在一个Worker服务进程，该服务进程主要进行数据的处理以及Task的执行。Worker服务进程每隔一定的时间都会向Coordinator上的RESTful服务发关心跳，从而告知Coordinator：我还活着，并接收你的调度。当客户端提交一个查询的时候，Coordinator则会从当前存活的Worker列表中选择出合适的Worker节点去运行Task。而Worker在执行每个Task的时候又会进一步对当前Task读入的每个Spli进行一系列的操作和处理。

### Presto模型

Presto可以通过多种不同类型的Connector访问多种数据源，目前支持的Connector包括：Hive、JMX、MySQL、Cassandra、PostgreSQL以及Kafka。下面介绍Presto是如何访问不同类型的数据源的，并对Presto中的模型和概念进行描述。

1.Connector
Presto是通过多种多样的Connector来访问多种不同的数据源的。你可以将Connector当作Presto访问各种不同数据源的驱动程序。一般情况下，Presto针对每种数据源都有与之对应的Connector。每种Connector都实现了Presto中标准的SPI接口，因此只要你实现Presto中的标准的SPI接口，就可以轻易地实现使用适合自己特定需求的Connector来访问特定的数据源。Presto目前支持的Connector 有Hive、JMX、MySQL、Cassandra、PostgreSQL等，都有其对应的Build-In Connector(内置的Connector)。
当你需要使用某种Connector访问特定的数据源时，需要在$PRESTO_HOME/etc/catalog、中创建一个配置文件：example.properties）（文件名字限制，但是其后缀名必须为.properties），在该配置文件中必须要设置一个属性:connector.name，该属性是必须设置的。Presto中Connector Manager就是通过该配置属性来决定使用哪个Connector去访问相应的数据源的。例如，你现在需要访问一个hive数据源，那么你在配置文件中就需要将属性connect.name设置为Hive-cdh5或者Hive-cdh4，这样Presto就会使用内置的Hive connector去访问Hive数据仓库中相应的数据。

2.Catalog
Presto中的Catalog类型于Mysql中的数据库实例。而Schema就类似于Mysql中的一个Database。通过使用特定的Connector访问Catalog中指定的数据源，一个Catalog中可以包含多个Schema。那么怎么定义一个Catalog呢？其实你不需要特意去指定Catalog。正如之前说的，假设你想访问Hive中的数据，则需要在$PRESTO_HOME/etc/catalog中创建一个配置文件：example.properties。该配置文件中定义了诸如Hive store的URI等访问Hive中的数据所需要的所有配置项，并且配置文件的名字就是Catalog名字:example。从这里可以看出Presto中配置文件的名字（不带.properties）就是Catalog的名字。
当你访问Catalog中的某个表时，该表的全名总是以Catalog的名字开始。例如名字为example.schema1.tables1的表，指的是表table1位于名为schema1的schema中，而schema1又位于名为example的Catalog中。

3.Schema
Presto中的Schema就类似于Mysql中的Database。一个Catalog名称和一个Schema名称唯一确定了可以查询的一系列表的集合。当通过Presto去查询Hive或者Mysql中的数据时，你会发现Presto中的Schema与Hive或者Mysql中的Database是相对应的。

4.Table
Presto中的Table与传统数据库中的Table的含义是一样的。

### Preto查询执行模型

Preto在执行SQL语句时，将这些SQL语句解析为相应的查询，并在分布式集群中执行这些查询。

1.Statement
Statement语句。其实就是指我们输入的SQL语句。Presto支持需要ANSI标准的SQL语句。这种语句由子句(Clause)、表达式（Expression）和断言(Predicate)组成。

Presto为什么将语句(Statement)和查询(Query)的概念分开呢？
因为在Presto中，语句和查询本身就是不同的概念。语句指的是终端用户输入的用文字表示的SQL语句；当Presto执行输入的SQL语句时，会根据SQL语句生成查询执行计划，进而生成可以执行的查询(Query)，而查询代表的是分布到所有的Worker之间执行的实际查询操作。

2.Query
Query即查询执行。当Presto接收一个SQL语句并执行时，会解析该SQL语句，将其转变成一个查询执行和相关的查询执行计划。一个查询执行代表可以在Presto集群中运行的查询，是由运行在各个Worker上且各自之间相互关联的阶段（Stage）组成的。

那么SQL语句与查询执行之间有什么不同呢？
其实很简单，你可以认为SQL语句就是提交给Presto的用文字表示的SQL执行语句。而查询执行则是为了完成SQL语句所表述的查询而实例化的配置信息、组件、查询执行计划和优化信息等。一个查询执行由Stage、Task、Driver、Split、Operator和DataSource组成。这些组件之间通过内部联系共同组成了一个查询执行，从而得到SQL语句表述的查询，并得到相应的结果集。

3.Stage
Stage即查询执行阶段。当Presto运行Query时，Presto会将一个Query拆分成具有层级关系的多个Stage，一个Stage就代表查询执计划的一部分。例如，当我们执行一个查询，从Hive的一张具有1亿条记录的表中查询数据并进行聚合操作时，Presto会创建一个Root Stage（后面会介绍，该Stage就是Single Stage），该Stage聚合其上游Stage的输出数据，然后将结果输出给Coordinator，并由Coordinator将结果输出给终端用户。
通常情况下Stage之间是树状的层次结构。每个Query都有一个Root Stage。该Stage用于聚集所有其他Stage的输出数据，并将最终的数据反馈给终端用户。需要注意的是，Stage并不会在集群中实际执行，它只是Coordinator用于对查询计划进行管理和建模的逻辑概念。每个Stage（除了Single Stage和Source Stage）都会有输入和输出，都会从上游Stage读取数据，然后将产生结果输出给下游Stage。需要注意的是；Source Stage没有上游，它从Connector获取数据，Single Stage没有下游，它的结果直接输出给Coordinator，它由Coordinator输出给终端用户。
Presto中的Stage共有4种，具体介绍如下：
Coordinator_Only：这种类型的Stage用于执行DDL或者DML语句中最终的表结构创建或者更改。
Single：这种类型的Stage用于聚合子Stage的输出，并将结果数据输出给终端用户。
Fixed：这种类型的Stage用于接受其子Stage产生的数据并在集群中对这些数据进行分布式的聚合或者分组计算。
Source：这种类型的Stage用于直接连接数据源，从数据源读取数据，在读取数据的同时，该阶段也会根据Presto对查询执行计划的优化完成相关的断言下发(Predicate PushDown)
和条件过滤等。

说明：一个SQL查询可以被分解 多个前后关联的Stage,在这里我们约定：按照数据的流向，越靠近数据源的Stage越处于上游，越远离数据源的Stage越处于下游。

4.Exchange
Exchange的字面意思就是“交换”。Presto的Stage是通过Exchange来连接另一个Stage的。Exchange用于完成有上下游关系的Stage之间的数据交换。在Presto中有两种Exchange：Output Buffer和Exchange Client。生产数据的Stage通过名为Output Buffer的Exchange将数据传送给其下游的Stage。消费数据的Stage通过名为Exchange从上游Stage读取数据。
如果当前Stage是Surce类型的Stage，那么该Stage则是直接通过相应的Connector从数据源读取数据的。而该Stage则是通过名为Source Operator的Operator与Connector进行交互的，例如，一个Source Stage直接从HDFS获取数据，那么这种操作不是通过Exchange Client来完成的，而是通过运行于Driver中的Source Operator来完成的。

5.Task
从前面的介绍中可以知道，Stage并不会在Presto集群中实际运行，它仅代表针对于一个SQL语句查询执行计划中的一部分查询的执行过程，只是用来对查询执行计划进行管理和建模。Stage在逻辑上又被分为一系列的Task，这些Task则是需要实际运行在Presto的各个Worker节点上的。
在Presto集群中，一个查询执行被分解成具有层次关系的一系列的Stage，一个Stage又被拆分为一系列的Task。每个Task处理一个或者多个Split。每个Task都有对应的输入和输入。一个Stage被分解为多个Task，从而可以并行地执行一个Stage。Task也采用了相应的机制：一个Task也可以被分解为一个或者多个Driver，从而并行地执行一个Task。

6.Driver
一个Task包含一个或者多个Driver。一个Driver其实就是作用于一个Split的一系列Operator的集合。因此一个Driver用于处理一个Split，并且生成相应的输出，这些输出由Task收集并且传送给下游Stage中的Task。一个Driver拥有一个输入和一个输出。

7.Operator
一个Operator代表对一个Spit的一种操作，例如过滤、加权、转换等。一个Operator依次读取一个Split中的数据，将Operator所代表的计算和操作作用于Split的数据上，并产生输出。每个Operator均会以Page为最小处理单元分别读取输入数据和产生输出数据。Operator每次只会读取一个Page对象，相应地，每次也只会产生一个Page对象。

8.Split
Split即分片，一个分片其实就是一个大的数据集中的一个小的子集。而Driver则是作用于一个分片上的一系列操作的集合，而每个节点上运行的Task，又包含多个Driver,从而一个Task可以处理多个Split。其中每一种操作均由一个Operator表示。分布式查询执行计划的源Stage(Source Stage)通过Connector从数据源获取多个分片。Source Stage对Split处理完毕之后，会将输出传递给其下游Stage（通常其下游Stage的类型为Fixed或者Single）。
当Presto执行一个查询的时候，首先会从Coordinator得到一个表对应的所有Split。然后Presto就会根据查询执行计划，选择合适的节点运行相应的Task处理Split。

9.Page
Page是Presto中处理的最小数据单元。一个Page对象包含多个Block对象，而每个Block对象是一个字节数组，存储一个字段的若干行。多个Block横切的一行是真实的一行数据。一个Page最大为1MB，最多16*1024行数据。

## 安装与部署

Presto目前只能部署在Linux中。以集群模式部署进行说明 。

### 环境说明

需要3台服务器，1台作为Coordinator，2台作为Worker。

### 准备工作

#### 建立SSH

Coordinator到各个Worker节点的ssh信任关系，以方便后续的Presto集群的管理和维护。

#### Hive

默认hive已经创建好。如果没有可自行安装。

#### 源码编译

下载官方编译好的安装包：https://github.com/prestodb/presto/archive/0.107.tar.gz。
执行mvn -T2C install -DskipTests
-T2C:说明一个CPU核心启动两个线程进行编译，可以加快源码编译的速度。
install：将编译完成的Jar包直接安装到Maven库中。
-DskipTests：跳过测试工程和测试类。

#### 服务部署

首先将二进制压缩包同步到各个节点上并解压，然后在各个节点上修改相应的配置项，最后启动Presto集群。

##### 修改配置

（1）config.properties

（2） jvm.config

（3）log.properties

（4）node.properties

### 启动和停止服务

sh /opt/presto/presto-server/bin/launcher start
sh /opt/presto/presto-server/bin/launcher stop

### 查询

```
package com.jd.PrestoJDBCClient;

import com.facebook.presto.jdbc.PrestoConnection;
import com.facebook.presto.jdbc.PrestoStatement;

import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.TimeZone;

public class PrestoJDBCTestClient {
    public static void printRow(ResultSet rs, int[] types) throws SQLException {
        for (int i = 0; i < types.length; i++) {
            System.out.print(" ");
            System.out.print(rs.getObject(i + 1));
        }
        System.out.println("");

    }

    public static void connect() throws SQLException {
        //设置时区，这里必须设置
        TimeZone.setDefault(TimeZone.getTimeZone("Asia/Shanghai"));
        try {
            //加载Presto JDBC驱动类
            Class.forName("com.facebook.presto.jdbc.PrestoDriver");

        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }

        PrestoConnection connection = null;
        try {
            //Presto连接串，在连接串中指定了默认的catalog为：system,默认的schema为:runtime，使用的用户名为：guest1，这个用户名根据实际业务自己设定，用来标识执行SQL的用户，虽然不会通过用户名进行身份认证，但是必须要写。密码直接指定为null，或者可以随便指定一个做生意密码，Presto是不会对密码进行验证的。
            connection = (PrestoConnection) DriverManager.getConnection("jdbc:presto://172.16.154.38:8001/system/runtime", "guest1", null);

        } catch (SQLException e) {
            e.printStackTrace();
        }

        PrestoStatement statement = null;
        try {
            statement = (PrestoStatement) connection.createStatement();
        } catch (SQLException e) {
            e.printStackTrace();
        }

        String query = "select * from nodes";
        ResultSet rs = null;
        try {
            rs = statement.executeQuery(query);
        } catch (SQLException e) {
            e.printStackTrace();
        }

        int cn = rs.getMetaData().getColumnCount();
        int[] types = new int[cn];
        for (int i = 1; i < cn; i++) {
            types[i - 1] = rs.getMetaData().getColumnType(i);
        }

        try {
            while (rs.next()) {
                printRow(rs, types);
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) throws ClassNotFoundException, SQLException {
        connect();
    }

}

```

## 参考

《Presto技术内幕》由京东研发团队出版。

中文社区：http://prestodb-china.com/
Presto 官网： http://prestodb.io/
Presto Github 主页： https://github.com/facebook/presto
京东修改版（推荐）： https://github.com/CHINA-JD/presto
Presto 文档： http://prestodb-china.com/docs/current/
