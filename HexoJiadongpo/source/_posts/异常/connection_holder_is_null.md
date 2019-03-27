---
title: connection_holder_is_null

date: 2017-08-18 09:55:00
tags: [异常,tomcat]
categories: [异常,tomcat]
---
#java.sql.SQLException: connection holder is null
##问题描述
使用druid连接池时出现的间歇性错误，偶尔会出现。

##原因分析
连接池可能对连接持有时间有限制，一种保护机制，如果某个连接很长时间不释放，其它应用就没有办法使用这个连接。

##解决方案
###方案一：
        延长这个超时时间，默认为300秒（推荐）
        <!--是否自动回收超时连接-->
   		<property name="removeAbandoned" value="true" />
   		<!--延长这个所谓的超时时间-->
   		<property name="removeAbandonedTimeout" value="1800" />
   		<!--将当前关闭动作记录到日志-->
   		<property name="logAbandoned" value="true" />        
###方案二：
        关闭这个超时保护
        /*直接关闭这个   自动回收超时连接*/
        <property name="removeAbandoned" value="false" />
