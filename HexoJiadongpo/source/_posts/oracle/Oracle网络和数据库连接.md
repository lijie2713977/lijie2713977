---
title: Oracle网络和数据库连接
date: 2017-04-01 15:30:00
tags: [oracle,数据库]
categories: [数据库,oracle]
---
内容来自《Oracle Database 11g 数据库管理艺术》
# 网络概念：Oracle网络如何工作
在希望从客户机（不管是传统客户机还是基于浏览器的客户机）打开数据库会话时，需要通过网络连接到数据库。假如要将台式电脑通过现有网络连接到UNIX服务器上的一个Oracle数据库，则需要在电脑和Oracle数据库（它使用专门的软件）之间构造一个连接方法。需要某种界面来处理会话（在此例子中为SQL*Plus），并且需要某种与业内标准的网络协议（如TCP/IP）通信的方法。  
为方便配置和管理网络连接，Oracle提供了Oracle Net Services，它是一套在分布式异构计算环境中提供连接方案的组件。Oracle Net Services由Oracle Net、Oracle Net Listener、Oracle Connection Manager、Oracle Net Configuration Assistant和Oracle Net Manager组成。Oracle Net Services软件是在Oracle Database Server或Oracle Client软件安装的过程中自动安装的。  
Oracle Net是一个初始化、建立及维护客户机和服务器之间的连接的组件。这就是为什么必须在客户机和服务器上都安装Oracle Net的原因。Oracle Net主要由两个组件构成。  
Oracle Network Foundation Layer：负责建立和维护客户机应用程序与服务器之间的连接，以及它们之间的交换信息。  
Oracle Protocol Support： 负责映射Transparent Net Substrate（TNS）功能到连接使用的业内标准协议。
驻留Oracle数据库的所有服务器还运行一个名为Oracle Net Listener（通常也称为监听器）的服务，其主要功能是监听来自客户机服务登录Oracle数据库的请求。监听器在保证客户机服务具有与数据库匹配的信息（协议、端口和实例名）后，将客户机请求传递到数据库。假如用户名和密码通过认证，则数据库将允许客户机登录。一旦监听器把用户请求交付给数据库，客户机和数据库将直接连接，不再需要监听器的帮助。
Oracle提供了基于GUI的大量的实用程序，以帮助配置数据库的网络连接。这些实用程序包括Oracle Connection Manager、Oracle Net Manager和Oracle Net Configuragion Assistant等。这些工具帮助处理所有网络需求。在结束本章学习后，可单击这些程序的图标，开始测试连接的实验。  
# Web应用如何连接到Oracle数据库
为了构造Oracle数据库的一个Internet连接，客户机上的Web浏览器要与Web服务器通信并使用HTTP进行连接请求。Web服务器将此请求传递给一个应用，该应用处理收到的请求并用Oracle Net（配置在数据库服务器和客户机上）与Oracle数据库服务器通信  
	下面介绍Oracle网络中几个关键的术语  
## 数据库实例名
正如所知，Oracle实例由SGA和一组Oracle进程组成。数据库实例名在初始化文件（init.ora）中作为INSTANCE_NAME参数给出。在谈到Oracle SID（System identifier，系统标识符）时，指的是Oracle实例。  
	通常，每个数据库只有一个与其关联的实例。但在Oracle RAC配置中，单个数据库可关联到多个实例。  
## 全局数据库名
全局数据库名唯一地标识一个Oracle数据库，其格式为database_name.database_domain，如sales.us.acme.com。在这个全局数据库中，sales为数据库名，us.acme.com为数据库域。因为相同的域中两个数据库不会有相同的数据库名，所以每个全局数据库名都是唯一的。  
## 数据库服务名
对于客户机，数据库在逻辑上简单地表现为一个服务。在服务和数据库之间存在一个多对多的关系，因为一个数据库可被一个或多个服务所代表，每个服务都专用于一组不同的客户机，而一个服务可覆盖不止一个数据库实例。我们在自己的系统中用每个数据库的服务名来标识它，用初始化参数SERVICE_NAMES来指定数据库的服务名。服务名参数值默认为全局数据库名。  
请注意，一个数据库可由多个服务名来访问。如果希望不同的客户机组访问适合于它们的特定需求的不同数据库，应该这样做。例如，可对相同数据库创建如下两个服务名：  
Sales.us.acme.com  
Finance.us.acme.com  
销售人员使用sales.us.acme.com服务名，而财务人员则使用finance.us.acme.com服务名。  
## 连接描述符
为了将电脑连接到世界上的任何数据库服务，需要提供两个信息：  
	数据库服务名；  
	地址。   
Oracle使用术语连接描述符(connect descriptor)来表示数据库连接的两个必需的部分：数据库服务名和地址。连接描述符的地址部分包含三个部分，分别是：连接使用的通信协议、主机名和端口号。  
了解通信协议有助于保证使用合适的网络协议，以便建立连接。标准的协议为TCP/IP或带SSL（Secure Sockets Layer，安全套接层）的TCP/IP。UNIX服务器上的Oracle连接的标准端口为1521或1526.Windows机器上的默认端口为1521.因为任何主机上的数据库具有唯一服务名，所以一个Oracle数据库服务名和一个主机名将唯一地标识任何数据库。下面是一个典型的连接描述符的例子：  
(DESCRIPTION
(ADDRESS=(PROTOCOL=tcp)(HOST=sales-server)(PORT=1521))
(CONNECT_DATA=
(SERVICE_NAME=sales.us.acme.com)))  
在此连接描述符中，ADDRESS行指出网络通信将使用TCP协议。HOST指定UNIX（或Windows）服务器，服务器上的Oracle监听器正监听来自端口1521的连接请求。连接描述符的ADDRESS部分也称为协议地址(protocol address)。
希望连接数据库的客户机首先连接到Oracle监听器进程。监听器接收到达的请求并把它们交给数据库服务器。一旦客户机和数据库服务器通过监听器的引导连接上，它们就直接通信，在此客户机连接的通信过程中不再需要监听器。  
## 连接标识符
连接标识符（connect identifier）与连接描述符紧密关联。可把连接描述符作为连接标识符，或者可简单地映射一个数据库服务名为一个连接描述符。例如，可以把一个服务名(如sales)映射为11.2.5节所看到的连接描述符。下面是说明映射sales连接标识符的例子。  
Sales=
(DESCRIPTION
(ADDRESS=(PROTOCOL=tcp)(HOST=sales-server)(PORT=1521))
(CONNECT_DATA=
(SERVICE_NAME=sales.us.acme.com)))  
## 连接串 
通过提供一个连接串(connect string)连接到数据库。连接串包含用户名/密码组合及一个连接标识符。最常见的连接标识符之一是节点服务名，它是一个数据库服务的名字。  
下面的例子给出一个连接串，它把一个完整的连接描述符作为连接标识符  
	CONNECT scott/tiger@(DESCRIPTION=
	(ADDRESS=(PROTOCOL=tcp)
	(HOST=sales-server)
	(PORT=1521))
	(CONNECT_DATA=
	(SERVICE_NAME=sales.us.acme.com)))
下面是一个更简单的连接到相同数据库的方法，它使用连接标识符sales：  
	CONNECT scott/tiger@sales  
上面两个例子都能连接到sales数据库，但显然第二个连接串（使用sales连接标识符）简单得多。  
 
## 使用Oracle网络服务工具
Oracle Net提供了配置客户机与数据库服务之间的连接的几个GUI和命令行工具。最常用的命令行工具是isnrctl实用程序，它帮助管理Oracle监听器服务。下面是帮助管理Oracle Net Servcies的重要GUI工具。  
Oracle NCA（Net Configuration Assistant，Oracle网络配置助手）。此工具主要用于在安装中配置网络组件，它允许在配置客户机连接的几个选项（本章稍后介绍这些选项）中进行选择。其便于使用的GUI界面使你能在所选择的任何命名方法下快速配置客户机连接。在UNIX/Linux系统上，可通过从$ORACLE_HOME/bin目录执行netca来启动NCA。在Window系统上，选择Start|Programs|Oracle-HOME_NAME|Configuration and Migration Tools|Net ConfigurationAssistant。  
Oracle网络配置管理器（Oracle Net Manager）。Oracle Net Manager可在客户机和服务器上运行，它允许配置各种命名方法和监听器。利用此工具，可在本地tnsnames.ora文件或在集中式的OID中配置连接描述符，而且可以方便地增加和修改连接方法。  
为了从Oracle企业管理器控制台启动Oracle Net Manager，选择Tools|Service Management|Oracle Net Manager。为了在Unix上作为独立的应用启动Oracle Net Manager，在ORACLE_HOME/bin目录执行netmgr。在windown上，选择Start|Programs|Oracle-HOME_NAME|Configuration and Migration Tools|NetManager。  
Oracle企业管理器（Oracle Enterprise Manager）。Oracle Database 11g中的OEM可以完成Oracle Net Manager能完成的所有任务，但不能跨多个文件系统管理多个Oracle主目录。此外，使用OEM可导出目录命名项到tnsnames.ora文件。  
Oracle目录管理器（Oracle Directory Manager）。这个功能强大的工具允许创建使用OID必需的各种域和环境。用此工具还可以执行密码策略管理及完成许多Oracle高级安全任务。在UNIX/LINUX系统上，可从$ORACLE_HOME/bin目录执行oidadmin来启动OID。在Window系统上，选择Start|Programs|Oracle-HOME_NAME|Integrated Manager Tools|Oracle Directory Manager。  
# 即时客户机
之前说过Oracle客户机安装需要经历常规的Oracle数据库服务器软件安排的所有预备步骤。幸而为连接到Oracle数据库并不总是需要安装完整的Oracle客户机软件。Oracle的新Instant Client（即时客户机）软件允许执行应用程序而不必安装标准的Oracle客户机也不必具有ORACLE_HOME。不需要为访问Oracle数据库的每台机器安装Oracle客户机软件。所有现有的OCI、ODBC和JDBC应用程序都可以使用Instan Client。如果愿意，甚至可以用Instant Client使用SQL*Plus。  
	相对于完整的Oracle客户机，Instant Clients提供以下好处：  
A. 它是免费的；  
B. 战胜磁盘空间较少  
C. 安装更快（5分钟左右）  
D. 不需要CD  
E. 它具有Oracle客户机的所有特性，如果有必要甚至包括使用SQL*Plus。  

# 安装Instant Client
以下是安装新Instant Client软件并快速连接到Oracle数据库的步骤。  
(1) 从OTN Web站点下载Instant Client软件。你必须安装基本的客户机程序包，还可以包括其他高级可选的程序包。此程序包含以下内容：    
	a) Basic：运行OCI、OCCI和JDBC-OCI应用程序所需的文件。  
	b) SQL*Plus：为用Instant Client运行SQL*Plus需要的库和可执行文件。  
	c) JDBC Supplement：另外支持XA、国际化及JDBC下的RowSet操作。  
	d) ODBC Supplement：启用带Instant Client的ODBC应用的另外的库（仅对Windows）。  
	e) SDK：用于Instant Client开发Oracle应用程序所需的其他文件。  
(2) 将选择的程序包解压到某个目录，将些目录命名为instantclient或其它类似的名称。  
(3) 在UNIX和Linux系统中，将环境变量LD_LIBRARY_PATH设置为instantclient（从而保证此参数的设置与程序包所有所在的目录名匹配）。在Winddows系统上，将环境变量PATH设置为instantclient。  
(4) 测试对Oracle服务器的连接。  
# 监听器和连接
Oracle监听器是一个只运行在服务器上并监听连接请求的服务。Oracle提供一个名为lsnrctl的实用程序来管理监听器进程。以下是监听器如何配合Oracle网络的概述。  
a. 数据库用监听器记录关于服务、实例及服务处理器的信息  
b. 客户机与监听器进行初步连接  
c. 监听器接收和验证客户机连接请求并把此请求交给数据库服务的服务处理器。一旦交付了客户机请求，监听器在该连接中不再起作用。  
Listener.ora文件默认位置在UNIX系统上为$ORACLE_HOME/network/admin目录，在Windows系统上为$ORACLE_HOME\network\admin目录，它包含监听器的配置信息。因为监听器服务只运行在服务器上，因此在客户机上没有listener.ora文件。代码清单11-1给出了一个典型的listener.ora文件。  
Listener中的所有配置参数都具有默认值，不需要手动配置监听器服务。在服务器上创建了第一个数据库后，监听器服务自动启动，并且将监听器配置文件listener.ora放于默认目录中。新数据库创建后，数据库的网络和服务信息自动添加到监听器的配置文件中。实例启动后，数据库自动向监听器注册，并且监听器开始监听对此数据库的连接请求。  
代码清单11-1 典型的监听器配置文件  
代码清单11-1 典型的监听器配置文件   
	#LISTENRE.ORA Network Configuration file 
	/u01/app/oracle/product/11.1.0.6.0/db_1/network/admin/listener.ora
	SID_LIST_LISTENER = 
	(DESCRIPTION_LIST =
		(DESCRIPTION = 
			  (ADDRESS_LIST = 
				(ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC4))
			)
			(ADDRESS_LIST = 
				(ADDRESS = (PROTOCOL = TCP)(HOST = NTL-ALAPATISAM)(PORT = 1521))
			)
		)
	)
	SID_LIST_LISTENER = 
		(SID_LIST = 
		(SID_DESC = 
			(SID_NAME = PLSExtProc)
			(ORACLE_HOME = /u01/app/oracle/product/11.1.0/db_1)
			(PROGRAM = extproc)
		)
		(SID_DESC = 
			(GLOBAL_DBNAME = remorse.world)
			(ORACLE_HOME = /u01/app/oracle/product/11.1.0/db_1)
			(SID_NAME = remorse)
		)
		(SID_DESC = 
			(GLOBAL_DBNAME = finance.world)
			(ORACLE_HOME = /u01/app/oracle/product/11.1.0/db_1)
			(SID_NAME = finance)
		)
	)
	 
## 自动服务注册
Oracle PMON进程负责向监听器动态服务注册新Oracle数据库服务名，也就是说，在创建新Oracle数据库时，它们将自动向监听器服务注册。PMON进程将在每个新数据库在服务器上创建之后更新listener.ora文件。
为自动服务注册，ini.ora文件或SPFILE应该包含如下参数：  
a. SERVICE_NAMES(如sales.us.oracle.com)  
b. INSTANCE_NAME(如sales)  
如果不指定SERVICE_NAMES参数的值，它默认为全局数据库名，全局数据库名是DB_NAME和DB_DOMAIN参数的组合。INSTANCE_NAME参数的默认值为Oracle安装或数据库创建时输入的SID。  
可使用lsnrctl实用程序查看服务器上监听器的状态，如代码清单11-2所示。相应的输出说明监听器启动了多长时间，监听器服务的配置文件位于何处。它还给出监听器为连接请求而监听的数据库的名称。  
代码清单11-2 使用lsnrctl实用程序查看监听器的状态  
$ lsnrctl status  
	 
在代码清单11-2的Services Summary部分，相应的状态可具有如下的某个值。  
a. READY：此实例可接受连接  
b. BLOCKED：此实例不能接受连接  
c. UNKNOWN：此实例在listener.ora文件中注册而不是通过动态服务注册，因而不知道其状态  
## 监听器命令
在调用lsnrctl实用程序后，除了status命令外还可以执行其他一些重要的命令。例如，service命令允许查看监听器正为连接请求而监控的是什么服务。  
注解：还可以从Oracle企业管理器的Net Services Administration页面查看监听器服务的状态。  
代码清单11-2 使用lsnrctl help列出lsnrctl命令  
$lsnrctl help  
可以调用lsnrctl实用程序后，使用start命令启动监听器，使用stop命令停业监听器。如果希望从操作系统命令行发布这些命令，可使用lsnrctl start和lsnrctl stop命令执行这两个任务。  
如果对listener.ora文件做了更改，为使更改起作用的一种方法是重启监听器。另一种安全的方法是重新装载监听信息，包括对监听器配置文件所做的最新更改。Lsnrctl reload命令允许在运行中重新装载监听器，而不用重新启动它。在监听器重装载（甚至是重启）的过程中，当前连接的客户机将继续保持连接，因为监听器已经将连接“交付”给数据库，在客户和数据库服务之间不起作用。  
注意：我的忠告是，如非绝对有必要，不要修改listener.ora文件，而且对于动态自动服务注册，几乎没有必要修改此文件。不过，有时可能需要修改监听器文件的某些部分，此文件由监听器监控连接请求的所有服务的网络配置信息组成。  
## 命名和连接
在前面连接描述符和连接标识符的例子中，使用sales连接标识符来连接sales服务。连接标识符可以是连接描述符本身，也可以是一个能解析为连接描述符的简单名字(如sales)。一般使用的简单连接标识符称为net service name(网络服务名)。因此前面例子中的sales连接标识符就是一个net service name。  
因为每次进行连接时都需要提供一下完整的连接描述符非常令人厌烦，使用网络服务名是明智的。但这需要维护网络服务名和连接描述信息之间所有映射的一个中心信息库(central repository)，以便Oracle验证这些网络服务名。因此，在一个用户使用网络服务名sales启动连接进程时，Oracle将搜索中心信息库查找sales的连接描述符。找到连接描述符后，Oracle Net会为指定服务器上的数据库初始化一个连接。  
Oracle允许几种类型的命名信息库，可用下列4种命名方法访问存储在这些位置中的映射信息。  
a. 本地命令(local naming )：使用存储在每个客户机上的名为tnsnames.ora的文件连接到数据库服务器。  
b. 简易连接命名(easy connect naming)：允许连接而无需任何服务名配置。  
c. 外部命名(external naming)：使用第三方命名服务来解析服务名。  
d. 目录命令(directory naming)：使用一个集中式的符合LDAP的目录服务器来解析服务名。  
	不管使用何种命名方法，名字解析过程都是相同的。每种命名法都遵循以下步骤将连接描述符解析为网络服务名：
i. 选择命令方法—本地、简易连接、外部命名或目录服务命名  
ii. 映射连接描述符到服务名；  
iii. 配置客户机以使用步骤1中选择的命名方法  
## 本地命名方法
本地命令是建立Oracle连接最简单、最容易的方法。使用这种方法，在名为tnsnames.ora的本地化配置文件中存储服务名及其连接描述符。此文件默认存储在$ORACLE_HOME/network/admin目录中。  
