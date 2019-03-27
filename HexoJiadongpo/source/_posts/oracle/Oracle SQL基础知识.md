---
title: Oracle SQL基础知识
date: 2016-05-22 14:43:49
tags: [oracle,数据库]
categories: [数据库,oracle]
---
## Oracle SQL基本知识##
### 安装数据库###
#### 1）安装Oracle常用问题(常用”用户名/密码“规则)：####
超级管理员：sys /change_on_install
普通管理员：system/manager
普通用户：scott/tiger----->默认是被锁定的
大数据用户：sh/sh
#### 2）SQL,DDL...####
SQL：structured query language 结构化查询语言
1.file(文件)

SQL:DDL DML TCL DQL DCL
DDL(data definition language 数据定义语言): column(列)--structure
create table (创建表):
列名 data type(数据类型) width(宽度)
constraint (约束)      alter table(修改表结构)           drop table(删除表)

DML(data manipulation language 数据操作语言)
:row(行)--data
insert 增       update 改            delete 删数据,删表里的记录

TCL(transaction control language 事务控制语言)
commit(提交)         rollback(回滚)               savepoint(保留点)

DQL(data query language 数据查询语言)
select
DCL(data control language 数据控制语言)
grant(授权)  grant to       revoke(回收权限) revoke from 
#### 3）RDBMS关系型数据库管理系统####
RDBMS(relationship database management system 关系型数据库管理系统) software(软件) --->(create database)database--->login in database (登录数据库系统 )--->用SQL操作table

create database 创建空间存储表 (datafile 数据文件)
login in database
1 远程登录到数据库所在的机器上
  192.168.0.20 192.168.0.23 192.168.0.26
shell(终端) telnet 192.168.0.20  (跟操作系统建连接)
login:openlab
password:open123
sunv210% shell提示符,执行操作系统命令
#### 4） 登录该机器上的数据库系统####
sunv210% sqlplus (跟数据库建连接)
Enter user-name: openlab
Enter password:open123
SQL>sqlplus openlab/open123
SQL> 数据库提示符,执行SQL命令

#### 5）登录的是哪个数据库####
echo $ORACLE_SID(环境变量)<---DBA(database administrator 数据库管理员)
查看ORACLE_SID变量的取值,oracle提供
通过设置ORACLE_SID变量,sqlplus就知道跟哪个数据库建连接.
unix平台
%c shell
%echo $ORACLE_SID  (tarena)
%setenv ORACLE_SID hiloo
%setenv ORACLE_SID tarena

$ b shell
$ echo $ORACLE_SID  (tarena)
$ ORACLE_SID=hiloo
$ export ORACLE_SID

windows平台
D:\>set ORACLE_SID=hiloo (设置环境变量)
D:\>set ORACLE_SID (查看环境变量)
ORACLE_SID=hiloo
##### 数据表信息：##### 
dept(表名) department 部门信息   列名
deptno 部门号  dname  部门名称      location 位置(地区)
create table dept_hiloo
(deptno  number(2), dname char(20),  location char(20));
insert into dept_hiloo values (10,'developer','beijing');
insert into dept_hiloo values (20,'account','shanghai');
insert into dept_hiloo values (30,'sales','guangzhou');
insert into dept_hiloo values  ( 40,'operations','tianjin');
commit;
insert成功后的提示:1 rows inserted
emp(表名) employee 员工信息    列名
empno 员工 ename 员工名字  job   职位   salary  月薪   bonus   奖金  
hiredate  入职日期  mgr   manager 管理者    deptno  部门号
create table emp_hiloo(
empno number(4),	ename varchar2(20),  job  varchar2(15),  
salary number(7,2), bonus number(7,2),  hiredate date,
 mgr number(4),  deptno number(10));
alter session set nls_date_language='american';
insert into emp_hiloo values (1001,'zhangwuji','Manager',10000,2000,'12-MAR-10',1005,10);
insert into emp_hiloo values (1001,'zhangwuji','Manager',10000,2000,'12-MAR-10',1005,10);
insert into emp_hiloo values (1002,'liucangsong','Analyst',8000,1000, '01-APR-11',1001,10);
insert into emp_hiloo values (1003,'liyi','Analyst',9000,1000,'11-APR-10',1001,10);
insertinto emp_hiloo values (1004,'guofurong','Programmer',5000,null,'01-JAN-11',1001,10);
insertintoemp_hiloo values (1005,'zhangsanfeng','President',15000,null,'15-MAY-08',null,20);
insert into emp_hiloo values (1006,'yanxiaoliu','Manager',5000,400,'01-FEB-09',1005,20);
insert into emp_hiloo values (1007,'luwushuang','clerk',3000,500,'01-FEB-06',1006,20);
insert into emp_hiloo values (1008,'huangrong','Manager',5000,500,'1-MAY-09',1005,30);
insert into emp_hiloo values (1009,'weixiaobao','salesman',4000,null,'20-FEB-09',1008,30);
insert into emp_hiloo values (1010,'guojing','salesman',4500,500,'10-MAY-09',1008,30);
报错信息
ORA-00955: name is already used by an existing object(名字已经被一个存在的对象使用)
错误：ORA-01843:无效的月份（在中文的plsql控制台上月份要写成’10-3月-02’这种形式，必须是一个数字和一个汉语月。也可以把日期改成英文环境，在执行插入前执行alter session set nls_date_language='american';就可以 了。

DQL
select(选择)
源表  结果集
1 投影操作 select子句实现
2 选择操作 where子句实现
3 连接操作
 1  select ename,salary*12 ann_sal(列别名)
 2* from emp_hiloo

单引号 表达字符串 ''
双引号 表达列别名 "",别名中包含空格,大小写敏感


##### 1）null值的理解##### 
1 null值出现在算术表达式中,结果必为null,null可以看作无穷大.
2 函数(function) nvl功能空值转换函数
nvl是函数名,p1,p2是参数,数据类型必须一致,函数本身有返回值
nvl(p1,p2)  
nvl函数实现:
if p1 is null then
   return p2;
else
   return p1;
end if;

3 若有多个null值,distinct去重时,结果集保留一个null值.
4 null = null 不成立 null <> null 不成立
5 若用in运算符,集合中有null值跟没有null值结果一致的,结果集中不会出现跟null值有关的记录
  若用not in运算符,集合中有null值,这个结果集不包含记录.no rows selected.
##### 2）各个子句的功能##### 
1 select后面跟列名,列别名,函数,表达式
2 select后面的distinct:去重
3 where子句
  where 条件表达式 (列名 比较运算符 值)  
表达式 比较运算符 值(尽量不用,为了性能)
  where子句中的列为字符类型,放值的位置上不加单引号或加双引号当列名解释,加单引号当字符串解释.
  where子句中的列为字符类型,表达具体值时注意字符是大小写敏感的.
SQL提供的四个比较运算符
肯定形式
   between and 区间,范围
   in <=> =any  (= or = )(跟集合里的任意一个值相等就满足条件) 集合 离散值
   = 单值运算符
   in =any 多值运算符
   like 像...一样
   通配符: %表示0或任意多个字符 _任意一个字符
   'S' 'S%' 'S_'
   is null  如何判断一个列的取值是否为空
否定形式
= <> != ^=
between and   not between and
in	not in (<> and <>) <=> <>all(跟集合里的所有值都不能相等)
like 	not like
is null   is not null 
各个子句的执行顺序
from-->where-->select
##### 3）课堂练习##### 
1 列出每个员工的名字和他的工资
  select ename,salary from emp_hiloo;
2 列出每个员工的名字和他的职位
  select ename,job from emp_hiloo;
3 列出每个员工的名字和他的年薪
 select ename,salary*12 ann_sal from emp_hiloo;
4 列出每个员工的名字和他一年的总收入
  (salary+bonus)*12 (15000+null)*12=null
  select ename,(salary+nvl(bonus,0))*12 tol_sal
  from emp_hiloo;
5 输出结果如下:
  zhangwuji is in department 10.
  liucangsong is in department 10.
  .....
  guojing is in department 30.
select ename||'is in department'||deptno||'.'employee from emp_hiloo;
什么要加employee呢？Employee是列别名为了显示用的。
6 列出该公司有哪些职位
  select distinct(job) from emp_hiloo;
  select distinct job from emp_hiloo;
7 列出该公司不同的奖金
  select distinct bonus from emp_hiloo;
8 各个部门有哪些不同的职位?
  select distinct deptno,job from emp_hiloo;
  去重方式:deptno和job联合唯一.
  distinct之后和from之前的所有列联合唯一.
distinct是保证每一行的唯一性而非某一列的唯一性，所以必须紧跟在select后面。
所以distinct只能放在select后面，紧跟select不然会报缺失表达式错误。
9 哪些员工的工资高于5000?
  select ename,salary from emp_hiloo 
  where salary > 5000; 
10 列出员工工资高于5000的员工的年薪?
  select ename,salary*12 from emp_hiloo
  where salary > 5000;
11 列出员工年薪高于60000的员工的年薪?
  select ename,salary*12 from emp_hiloo
  where salary*12> 60000;
  select ename,salary*12 ann_sal from emp_hiloo
  where ann_sal > 60000(错误的写法)
  select ename,salary*12 from emp_hiloo
  where salary > 5000;
12 zhangwuji的年薪是多少?
select ename,salary*12 from emp_hiloo
where ename='zhangwuji';
  哪些员工的职位是Manager?
select ename,job from emp_hiloo
where job='Manager';
  哪些员工的职位是clerk?
  select ename,job from emp_hiloo
  where job = 'Manager'
   select ename,job from emp_hiloo
  where job = 'clerk'(效率高)
  clerk的大小写不清楚
  函数:upper(),lower()
  select ename,job from emp_hiloo
  where upper(job) = 'CLERK' (通用性好)
13 员工工资在5000到10000之间的员工的年薪
   select ename,salary*12
   from emp_hiloo
   where salary >= 5000 
   and   salary <= 10000;
   select ename,salary*12
   from emp_hiloo
   where salary between 5000 and 10000;
14 哪些员工的工资是5000或10000.
   select ename,salary
   from emp_hiloo
   where salary = 5000
   or salary = 10000
   select ename,salary
   from emp_hiloo
   where salary in (5000,10000)
   select ename,salary
   from emp_hiloo
   where salary =any (5000,10000)
15 哪个员工的名字的第二个字符是a.
   select ename
   from emp_hiloo
   where ename like '_a%';
16 哪个员工的名字的第二个字符是_.
   select ename
   from emp_hiloo
   where ename like '_\_%' escape '\';
   第一个_表示任意一个字符,代表通配符
   \_必须连起来看,表示下划线本身,escape定义哪个字符可以定义转义'\'
17 哪些员工没有奖金?
   select ename,bonus
   from emp_hiloo
   where bonus is null
18 哪些员工有奖金?
   select ename,bonus
   from emp_hiloo
   where bonus is not null
19哪些员工的工资不是5000也不是10000.
  select ename,salary
  from emp_hiloo
  where salary not in (5000,10000);
  select ename,salary
  from emp_hiloo
  where salary <> 5000
  and salary <> 10000

create table emp_hiloo
( hiredate date）
insert into emp_hiloo values (1001,'zhangwuji','Manager',10000,2000,'12-MAR-10',1005,10);
解决方案：
	insert into emp_hiloo values (1001,'zhangwuji','Manager',10000,2000,'12-3月-10',1005,10);
##### 更改字段名字(mysql、orcle)：##### 
Oracle修改表
alter table 表名 rename column 原名 to 新名；
Mysql:
alter table 表名 change column(可写，可不写）原名 新名 字段类型；

ORA-00904：“ANN_SAL":invalid identifier
无效的标识符


index(索引) view(视图) sequence(顺序号/序列号) function(函数)
session altered.会话已更改
set feed on可以设置一个，显示操作数
connet tiger重新建立连接  show user查看当前用户是谁。
edit 用记事本编辑  /运行。
###Function (单行、多行)###
单行函数:表中的一列作为函数的参数,对于每一条记录函数都有一个返回值. 
例如:upper lower nvl
多行函数：表中的一列作为函数的参数,将记录分组,对于每组数据函数返回一个值. 
例如:avg
####1）单行函数####
 根据处理参数的数据类型分为
  ##### 1）字符函数:upper,lower##### 
   ##### 2）数值函数:##### 
     round 四舍五入
     round(12.345,2)-->12.35
     round(12.345,0)=round(12.345)-->12
     round(12,345,-1)-->10
     trunc 截取
     trunc(12.345,2)-->12.34
     trunc(12.345,0)=trunc(12.345)-->12
     trunc(12,345,-1)-->10
  ##### 3) 日期和日期函数##### 
    select sysdate from dual
    06-SEP-12 DD-MON-RR 
    alter session set
      nls_date_format = 'yyyy mm dd hh24:mi:ss'
    session 会话 connection(连接)
   日期类型的数据是用固定的字节7个字节来存储世纪,年,月,日,时,分,秒. 格式敏感
   会话级 alter session set nls_date_format
   语句级 select to_char(c1日期类型用7个字节来表达，日期类型的数据是用固定的字节7个字节来存储世纪，年，月，日，时，分，秒。四位年的前两位代表世纪20，后两位代表当前年12
如果不想修改sql语句运行的话，就需要在执行该语句之前，使用alter session 命令将nls_date_language修改为american，如下：
alter session set nls_date_language='american'    --以英语显示日期
如果不想修改sql语句运行的话，就需要在执行该语句之前，使用alter session 命令将

'01-JAN-08' 系统做了隐式数据类型转换,调用了to_date函数
'2008-01-01',用户做显式数据类型转换,自己调用
to_date('2008-01-01','yyyy-mm-dd'),第二个参数是对第一个参数的格式说明.
to_char的返回类型是字符类型,把date转换成了字符串类型,所以参数的数据类型是date.to_char函数可以获得日期的任何一部分信息,比如年,月,日等.
select c1 from ... 系统做了隐式数据类型转换,调用了to_char函数
select to_char(c1,.. 用户做显式数据类型转换,自己调用to_char(c1,'yyyy-mm-dd'),第二个参数是对第一个参数的格式说明.
日期的运算
   日期可以加减一个数值,单位为天.
   select sysdate-1,sysdate,sysdate+1 from dual
两个日期相减
   add_months 按月加 返回类型是date
   add_months(sysdate,6)
   select add_months(hiredate,6) from emp_hiloo
   add_months(sysdate,-6)
   months_between()  返回类型是number
   months_between(sysdate,hiredate) 两个日期之间相差多少个月
select months_between(sysdate,hiredate) from emp_hiloo;
   last_day(sysdate) 本月的最后一天
##### 4) 转换函数#####    
两个日期相减转换函数
to_date  char-->date
to_char  date-->char , number --> char
to_number  char-->number
##### 其他函数##### 
coalesce 类似nvl(oracle专有)
nvl(bonus,salary*0.1)
coalesce(bonus,salary*0.1,100)。输出所有员工的奖金，如果没有奖金就按工资的10%发放，如果奖金和工资都没有的临时工，就给100元。
不同的记录处理方式不一样时,用case when.
case when 条件表达式 then 返回结果
else
     返回结果
end
若没有else,当不匹配条件,表达式的返回值为null.
case deptno when 10 then(不建议该语法形式)
decode跟case when的功能类似.
decode(deptno,10,salary*1.1,
              20,salary*1.2,
              salary)
若没有最后一个参数,函数的返回值为null.
select语句
order by子句
select   from    where
order by
order by子句是select语句中的最后一个子句.
order by salary 缺省是升序 asc
order by salary desc 降序
order by子句后面可以跟列名,表达式(函数),列别名,在select子句中的位置.
ORDER BY 子句
ORDER BY 语句用于对结果集进行排序。
ORDER BY 语句
ORDER BY 语句用于根据指定的列对结果集进行排序。
ORDER BY 语句默认按照升序对记录进行排序。
如果您希望按照降序对记录进行排序，可以使用 DESC 关键字。
原始的表 (用在例子中的)：
Orders 表:
Company	OrderNumber
IBM	3532
W3School	2356
Apple	4698
W3School	6953
实例 1
以字母顺序显示公司名称：
SELECT Company, OrderNumber FROM Orders ORDER BY Company
结果：
Company	OrderNumber
Apple	4698
IBM	3532
W3School	6953
W3School	2356
实例 2
以字母顺序显示公司名称（Company），并以数字顺序显示顺序号（OrderNumber）：
SELECT Company, OrderNumber FROM Orders ORDER BY Company, OrderNumber
结果：
Company	OrderNumber
Apple	4698
IBM	3532
W3School	2356
W3School	6953
实例 3
以逆字母顺序显示公司名称：
SELECT Company, OrderNumber FROM Orders ORDER BY Company DESC
结果：
Company	OrderNumber
W3School	6953
W3School	2356
IBM	3532
Apple	4698
实例 4
以逆字母顺序显示公司名称，并以数字顺序显示顺序号：
SELECT Company, OrderNumber FROM Orders ORDER BY Company DESC, OrderNumber ASC
结果：
Company	OrderNumber
W3School	2356
W3School	6953
IBM	3532
Apple	4698
注意：在以上的结果中有两个相等的公司名称 (W3School)。只有这一次，在第一列中有相同的值时，第二列是以升序排列的。如果第一列中有些值为 nulls 时，情况也是这样的。

#### 2) 多行函数(哪两个函数里只能放number)####
avg()	平均值  函数的参数只能是number
sum()	求和	函数的参数只能是number
count()	计数 函数的参数可以是number date 字符
        count(*)统计记录,count(bonus)
max() 最大值 函数的参数可以是number date 字符
min() 最小值 函数的参数可以是number date 字符

组函数的缺省处理方式是处理所有的非空值.
avg(bonus) 所有有奖金的员工的平均值
count(bonus) 有奖金的员工个数
当所有的值都是null,count函数返回0,其他组函数返回null.

#### 3) group by子句####
若有group by子句,select后面跟组标识和组函数
组标识指group by后面的内容
from-->where-->group by-->select-->order by
若没有group by子句,select后面只要有一个是组函数,其余的都得是组函数.

#### having子句####
select deptno,round(avg(salary)) davg
from emp_hiloo
group by deptno
having round(avg(salary))> 5000

from-->where-->group by-->having-->select-->order by 
#### GROUP BY 语句####
GROUP BY 语句用于结合合计函数，根据一个或多个列对结果集进行分组。
SQL GROUP BY 语法
SELECT column_name, aggregate_function(column_name)
FROM table_name
WHERE column_name operator value
GROUP BY column_name
SQL GROUP BY 实例
我们拥有下面这个 "Orders" 表：
O_Id	OrderDate	OrderPrice	Customer
1	2008/12/29	1000	Bush
2	2008/11/23	1600	Carter
3	2008/10/05	700	Bush
4	2008/09/28	300	Bush
5	2008/08/06	2000	Adams
6	2008/07/21	100	Carter
现在，我们希望查找每个客户的总金额（总订单）。我们想要使用 GROUP BY 语句对客户进行组合。
我们使用下列 SQL 语句：
SELECT Customer,SUM(OrderPrice) FROM Orders
GROUP BY Customer
结果集类似这样：
Customer	SUM(OrderPrice)
Bush	2000
Carter	1700
Adams	2000
很棒吧，对不对？
让我们看一下如果省略 GROUP BY 会出现什么情况：
SELECT Customer,SUM(OrderPrice) FROM Orders
结果集类似这样：
Customer	SUM(OrderPrice)
Bush	5700
Carter	5700
Bush	5700
Bush	5700
Adams	5700
Carter	5700
上面的结果集不是我们需要的。
那么为什么不能使用上面这条 SELECT 语句呢？解释如下：上面的 SELECT 语句指定了两列（Customer 和 SUM(OrderPrice)）。"SUM(OrderPrice)" 返回一个单独的值（"OrderPrice" 列的总计），而 "Customer" 返回 6 个值（每个值对应 "Orders" 表中的每一行）。因此，我们得不到正确的结果。不过，您已经看到了，GROUP BY 语句解决了这个问题。
GROUP BY 一个以上的列
我们也可以对一个以上的列应用 GROUP BY 语句，就像这样：
SELECT Customer,OrderDate,SUM(OrderPrice) FROM Orders
GROUP BY Customer,OrderDate
#### 4) where和having比较####
共同点:都执行在select之前,都有过滤功能
区别
where执行在having之前
where过滤的是记录,任意列名都可以出现在where子句,单行函数可以用在where子句,组函数不能出现在where子句
having过滤的是组,组标识可以出现在having子句,其他列名不行,组函数用于having子句,单行函数不可以.
##### HAVING 子句##### 
在 SQL 中增加 HAVING 子句原因是，WHERE 关键字无法与合计函数一起使用。
SQL HAVING 语法
SELECT column_name, aggregate_function(column_name)
FROM table_name
WHERE column_name operator value
GROUP BY column_name
HAVING aggregate_function(column_name) operator value
SQL HAVING 实例
我们拥有下面这个 "Orders" 表：
O_Id	OrderDate	OrderPrice	Customer
1	2008/12/29	1000	Bush
2	2008/11/23	1600	Carter
3	2008/10/05	700	Bush
4	2008/09/28	300	Bush
5	2008/08/06	2000	Adams
6	2008/07/21	100	Carter
现在，我们希望查找订单总金额少于 2000 的客户。
我们使用如下 SQL 语句：
SELECT Customer,SUM(OrderPrice) FROM Orders
GROUP BY Customer
HAVING SUM(OrderPrice)<2000
结果集类似：
Customer	SUM(OrderPrice)
Carter	1700
现在我们希望查找客户 "Bush" 或 "Adams" 拥有超过 1500 的订单总金额。
我们在 SQL 语句中增加了一个普通的 WHERE 子句：
SELECT Customer,SUM(OrderPrice) FROM Orders
WHERE Customer='Bush' OR Customer='Adams'
GROUP BY Customer
HAVING SUM(OrderPrice)>1500
结果集：
Customer	SUM(OrderPrice)
Bush	2000
Adams	2000

#### 5) DCL#### 
connect openlab/open123
select count(*) from hiloo.emp_hiloo;

connect hiloo/hiloo123
grant select on emp_hiloo to openlab;

connect openlab/open123
select count(*) from hilool.emp_hiloo
10rows selected

connect hiloo/hiloo123
revoke select on emp_hiloo from openlab;

show user
select count(*) from hiloo.emp_hiloo

create synonym emp_hiloo for hiloo.emp_hiloo
#### 6) 关于null值的讨论####
1 case when在没有else和decode少一个参数时,返回null.
2order by bonus,asc升序时null值在最后,desc降序时null在最前.
3 组函数和null值的关系:1组函数的缺省处理方式是处理所有的非空值.2当所有的值都是null,count函数返回0,其他组函数返回null.
4若group by的列有null值,所有的null值分在一组.
课堂练习
1将每个员工的工资涨12.34567%,用round和trunc分别实现
select ename,nvl(trunc(round(salary+salary*0.1234567,2),1),0.0) from emp_hiloo;//自己写的。
2 将'2008-01-01'插入表中,
  再将'2008 08 08 08:08:08'插入表中
insert into test values
(to_date('01-JAN-08','DD-MON-RR'));

3找出3月份入职的员工.
select ename,hiredate
from emp_hiloo
where to_char(hiredate,'mm') = '03';
select ename,hiredate
from emp_hiloo
where to_char(hiredate,'mm') = 3;//可以正常输出winXP下
'03' = 3  ---> to_number('03') = 3
字符   数值  缺省系统将字符转成数值
select ename,hiredate
from emp_hiloo
where to_char(hiredate,'fmmm') = '03';(错，未选定行，无输出)

select ename,hiredate
from emp_hiloo
where to_char(hiredate,'fmmm') = '3';(对)
'03' = '3' (错)
fm表示去掉前导0或去掉两边的空格.
4 zhangsanfeng的mgr上显示boss,其他人不变.
select ename,empno,
       nvl(to_char(mgr),'boss') mgr
from emp_hiloo
函数nvl（“1”，“2”）:如果字符串1是空，就返回字符串”2”
#### 5十分钟之后####
 select sysdate,sysdate+1/144 from dual;
解释：Oracle 里面,

sysdate + 1 意思是 当前时间 + 1天

sysdate + 1/24  意思是 当前时间 + 1/24天  也就是1小时后

sysdate+1/144  意思是 当前时间 + 1/144天 （1/24*6）  也就是10分钟后
 6 若员工是10部门的,工资涨10%,20部门工资涨20%,其他员工工资不变.
select ename,salary,
       case when deptno = 10 then salary*1.1
            when deptno = 20 then salary*1.2
       else
            salary
       end new_sal
from emp_hiloo;

select ename,salary, 
       decode(deptno,10,salary*1.1,
                     20,salary*1.2,
                     salary) new_sal
from emp_hiloo;
7 列出每个员工的年薪,按年薪降序排列.
select ename,salary*12
from emp_hiloo
order by salary desc (好)
select ename,salary*12
from emp_hiloo
order by salary*12 desc
select ename,salary*12 n_sal
from emp_hiloo
order by n_sal desc

select ename,salary*12 n_sal from emp_hiloo order by 2 desc;
select salary*12,ename n_sal from emp_hiloo order by 2 asc;
8 列出员工的名字,部门号以及工资,按部门号从小到大的顺序,同一部门的工资按降序排列.
select ename,deptno,salary
from emp_hiloo
order by deptno,salary desc
9 列出奖金的平均值,和,个数,最大值,最小值.
AVG 函数返回数值列的平均值。NULL 值不包括在计算中
select avg(bonus),avg(nvl(bonus,0)),
       sum(bonus), sum(nvl(bonus,0)),
       count(bonus),count(nvl(bonus,0)),
       max(bonus),max(nvl(bonus,0)),
       min(bonus),min(nvl(bonus,0))
from emp_hiloo
10 各个部门的平均工资
ROUND 函数用于把数值字段舍入为指定的小数位数。
select deptno,round(avg(salary))
from emp_hiloo
group by deptno
11 求10部门的平均工资,只显示平均工资
   求10部门的平均工资,显示部门号,平均工资
   select round(avg(salary))
   from emp_hiloo
   where deptno = 10
   group by deptno

   select max(deptno),round(avg(salary))
   from emp_hiloo
   where deptno = 10 
12各个部门不同职位的平均工资
   select deptno,job,round(avg(salary))
   from emp_hiloo
   group by deptno,job
13 每种奖金有多少人?
   select bonus,count(empno)
   from emp_hiloo
   group by bonus
14 列出平均工资大于5000的部门的平均工资
   select deptno,round(avg(salary)) 
   from emp_hiloo
   group by deptno
   having round(avg(salary)) > 5000
15哪些员工的工资是最低的.
  select ename from emp_hiloo
  where salary = ( select min(salary)
                   from emp_hiloo)
报错信息
ORA-01861: literal does not match format string
文字值不匹配格式串
ORA-01722: invalid number 无效的数值 to_number
ORA-00937: not a single-group group function 不是一个组函数
ORA-00979: not a GROUP BY expression 不是一个group by表达式 GROUP BY expression指跟在group by后面的东西(列名),称之为组标识
detail 细节 summary 聚合

### 查询###
子查询定义
在SQL语句中嵌入select语句
create table new_tabname
as
select ename,salary*12 ann_sal from emp_hiloo;
新表的结构由select后面的项来决定,new_table包含两列ename,ann_sal.

#### 子查询####
  非关联子查询
    单列子查询
    多列子查询
  关联子查询

##### 子查询执行##### 
非关联子查询
子查询的表和主查询的表没有建关联
先执行子查询(只执行一遍),当返回多条记录,系统会将自动去重的结果返回给主查询,再执行主查询.

关联子查询
子查询的表和主查询的表建关联.所谓建关联指主查询表里的列和子查询表里的列写成一个条件表达式.

先执行主查询,判断表里的记录是否应该放入结果集.过程如下:拿到第一条记录,获得了各个列的值,将需要的列值带入子查询,执行后返回的结果再和主查询表里的列做比较,符合条件,该记录放入结果集,否则过滤掉.依次执行主查询表里的每条记录.子查询执行的次数由主查询表里的记录数决定.

1) exists和not exists
exists的执行过程
从主查询表里拿到第一条记录,按子查询里的关联条件在子查询的表里看是否能找到匹配的记录,当找到第一条匹配的记录后,立即返回(即不需要找出所有匹配的记录),exists条件满足,主查询表里的该记录放入结果集.若按子查询里的关联条件将子查询
表里的记录全部检查一遍后没有一条符合条件的记录,此时也返回, exists 条件不满足,主查询表里的该记录不能放入结果集,被过滤掉.

select ename from emp_afei o
where exists
             (select 1 from emp_afei i
              where o.empno = i.mgr)

##### 非关联子查询的分类##### 
单列子查询
select ename,salary
from emp_hiloo 
where salary = (select min(salary)
                from emp_hiloo 
                )
多列子查询:按键值对比较
select ename,salary,deptno
from emp_afei
where (deptno,salary) in
             (select deptno,round(avg(salary))
              from emp_afei
              group by deptno)

2) 课堂练习
1哪些人是领导?(非关联子查询)
如果一个员工的empno能出现在mgr里就说明他是领导.
select ename
from emp_hiloo
where empno in (select mgr from emp_hiloo)
select ename
fro emp_afei
where empno in (1001,1005,1006,1008,null)
2 哪些人是员工?
他的empno绝对不能出现在mgr中,他的empno跟mgr的出现的所有的值不能相等. <>all
select ename
from emp_hiloo
where empno not in (select mgr from emp_hiloo)
select ename
fro emp_afei
where empno not in (1001,1005,1006,1008,null)
select ename
from emp_hiloo
where empno not in (select mgr from emp_hiloo
                    where mgr is not null)

3哪些部门的平均工资比30部门的平均工资高?
select deptno,round(avg(salary))
from emp_hiloo
group by deptno
having round(avg(salary)) >
                    (select round(avg(salary))
                     from emp_hiloo
                     where deptno = 30)
4哪些员工的工资比zhangwuji的工资高?
select ename,salary
from emp_afei
where salary > (select salary from emp_afei
                where ename = 'zhangwuji')
ERROR at line 3:
ORA-01427: single-row subquery returns more than one row
单行子查询返回多条记录

比所有人高 > (select max(salary))
           >all
比任意人高 > (select min(salary)
           >any
5哪些员工的工资等于本部门的平均工资?
select ename,salary,deptno
from emp_afei
where (deptno,salary) in
             (select deptno,round(avg(salary))
              from emp_afei
              group by deptno)
5哪些员工的工资比本部门的平均工资高?
select ename,salary,deptno
from emp_afei o
where salary > (select round(avg(salary))
                from emp_afei i
                where i.deptno = o.deptno)
6哪些人是领导?(关联子查询)
select ename from emp_afei o
where exists
             (select 1 from emp_afei i
              where o.empno = i.mgr)
7哪些部门有员工?
select deptno,dname
from dept_afei o
where exists (select 1 from emp_afei i
              where o.deptno = i.deptno)

3) 课外练习day03am
1 zhangwuji的领导是谁,显示名称?
2 zangwuji领导谁,显示名称?
3 列出devoleper部门有哪些职位?
1) 课外练习day04am答案
1 zhangwuji的领导是谁,显示名称?
  select ename from emp_afei
  where empno in 
		(select mgr from emp_afei
                 where ename = 'zhangwuji')

zangwuji领导谁,显示名称?

 select ename from emp_afei
 where mgr in (select empno from emp_afei
               where ename = 'zhangwuji')

3 列出developer部门有哪些职位?
  select distinct job from emp_afei
  where deptno in 
           (select deptno from dept_afei
            where dname = 'developer')

2) 非关联子查询               
exists和not exists
not exists的执行过程
从主查询表里拿到第一条记录,按子查询里的关联条件在子查询的表里看是否能找到匹配的记录,当找到第一条匹配的记录后,立即返回(即不需要找出所有匹配的记录),not exists条件不满足,主查询表里的该记录不能放入结果集,被过滤掉.若按子查询里的关联条件将子查询表里的记录全部检查一遍后没有一条符合条件的记录,返回, not exists 条件满足,主查询表里的该记录放入结果集.

对于exists和not exists,在子查询中找到第一条匹配的记录都会立即返回,exists将主查询表里的记录放入结果集,not exsits将主查询表里的记录过滤掉.
对于exists和not exists,如果子查询没有返回任何记录,即扫描全部记录后没有一条符合条件的记录,都返回,exists将主查询表里的记录过滤掉,not exists将主查询表里的记录放入结果集. 
not in ,<> all逻辑上跟not exists等价
in ,=any逻辑上跟exists等价

查询形式:集合操作
把结果集作为一个集合,结果集必须是同构的,列的个数及数据类型一致

3) 并集  union(去重)/union all(不去重)
select ename,deptno,salary,salary*1.1 new_sal
from emp_afei
where deptno = 10
union all
select ename,deptno,salary,salary*1.2 new_sal
from emp_afei
where deptno = 20
union all
select ename,deptno,salary,salary new_sal
from emp_afei
where deptno not in (10,20)

case when和decode可以实现类似功能.

4) 交集  intersect(去重)
select job from emp_afei
where deptno = 10
intersect
select job from emp_afei
where deptno = 20
10部门和20部门都有的职位是哪些?

5) 差  minus(去重)
select deptno from dept_afei
minus
select deptno from emp_afei
那些部门没有员工.

6) 多表查询
1) 交叉连接 cross join
select e.ename,d.dname
from emp_afei e cross join dept_afei d
结果集产生
10*4=40,组合操作,笛卡尔积

2) 内连接 inner join(匹配一个条件)
select e.ename,e.deptno,d.deptno,d.dname
from emp_afei e join dept_afei d
ORA-00905: missing keyword(丢失关键字)

如果把结果集的产生看成双层循环,驱动表是外层循环,匹配表是内层循环.
对于内连接哪张表做驱动表,哪张表做匹配表产生出的结果集是一样的,不同的是性能.
驱动表在匹配表的匹配情况如下:
一条记录找到一条匹配
一条记录找到多条匹配
一条记录找不到任何匹配.
内连接的核心是驱动表的记录要出现在结果集中必须在匹配表中能找到匹配的记录,否则该记录被过滤掉.

3) 内连接查询形式
等值连接 on e.deptno = d.deptno
两张表有表述同一属性的列,两张表都有deptno列.
自连接 on e.mgr = m.empno
同一张表的不同列能写成一个表达式,即同一张表的两条记录之间有关系.通过给表起别名的方式,将同一张表的两条记录之间的关系转化成不同表的两条记录之间的关系.
4) 外连接
外连接 outer join(驱动表的记录一个都不能少的出现在结果集里)
from t1 left join t2
on t1.c1 = t2.c2(t1驱动表,t2匹配表)
外连接结果集=内连接的结果集+t1表中匹配不上的记录和t2表中的null记录的组合
from t1 right join t2
on t1.c1 = t2.c2(t2驱动表,t1匹配表)
外连接结果集=内连接的结果集+t2表中匹配不上的记录和t1表中的null记录的组合
from t1 full join t2
on t1.c1 = t2.c2
外连接结果集=内连接的结果集+t1表中匹配不上的记录和t2表中的null记录的组合+t2表中匹配不上的记录和t1表中的null记录的组合

5) 外连接的应用场景
1 某张表的记录全部出现在结果集中,包括匹配不上的.
select e.ename,nvl(m.ename,'Boss')
from emp_afei e left join emp_afei m
on e.mgr = m.empno
2解决否定问题,匹配不上的记录找出来(跟所有的记录都不匹配.)(not in/not exists)
外连接 + where 匹配表.主键列 is null
select e.ename,d.dname
from emp_afei e right join dept_afei d
on e.deptno = d.deptno
where e.empno is null
(解决结果集只包含匹配不上的记录.where子句执行在外连接之后)哪些部门没有员工

select d.dname
from emp_afei e right join  dept_afei d
on e.deptno = d.deptno
and e.ename = 'zhangwuji'
where e.empno is null
如果希望在外连接之前过滤匹配表用and子句,如果想在外连接之后通过匹配表里的列过滤外连接的结果集时候用where.
过滤驱动表统计用where子句过滤.

6) 课内练习
1 哪些部门没有员工(not exists)
  select dname from dept_afei o
  where not exists 
        (select 1 from emp_afei i
         where o.deptno = i.deptno)
2 哪些人是员工?(not exists)
  select ename from emp_afei o
  where not exists 
            (select 1 from emp_afei i
             where o.empno = i.mgr)
他的empno和其他人的mgr相等是不可能存在的.即和所有人的mgr都不相等.
not in ,<> all逻辑上跟not exists等价
3 列出哪些员工在北京地区上班?
思路:确定表,两张表,匹配问题用inner join-->on(匹配条件)-->(对表是否过滤)
select e.ename,e.deptno,d.deptno,d.dname
from emp_afei e join dept_afei d
on e.deptno = d.deptno
and d.location = 'beijing'
4zhangwuji在哪个地区上班?
select e.ename,d.dname,d.location
from emp_afei e join dept_afei d
on e.deptno = d.deptno
and e.ename = 'zhangwuji'
5列出每个部门有哪些职位?部门名称,职位
 select distinct d.dname,e.job
 from emp_afei e join dept_afei d
 on e.deptno = d.deptno
 order by d.dname
6各个部门的平均工资,列出部门名称,平均工资.
select d.dname,round(avg(e.salary)) savg
from emp_afei e join dept_afei d
on e.deptno = d.deptno
group by d.dname
select max(d.dname),round(avg(e.salary)) savg
from emp_afei e join dept_afei d
on e.deptno = d.deptno
group by d.deptno
select min(deptno),round(avg(salary))
from emp_hiloo
where deptno = 10
7 列出每个员工的名字和他的领导的名字
select e.ename employee,
       m.ename manager
from emp_afei e join emp_afei m
on e.mgr = m.empno
结果集是9条.
e表中有10条记录,其中9条记录找到匹配,zhangsanfeng没匹配
m表中有10条记录,其中4条记录找到匹配,4条记录是领导,6条记录找不到匹配,他们是员工.
select e.ename employee,
       m.ename manager
from emp_afei e join emp_afei m
on e.mgr = m.empno
union all
select ename,'Boss'
from emp_afei
where mgr is null

select e.ename employee,
       decode(m.ename,e.ename,'Boss',
                  m.ename)   manager
from emp_afei e join emp_afei m
on nvl(e.mgr,e.empno) = m.empno

select e.ename,nvl(m.ename,'Boss')
from emp_afei e left join emp_afei m
on e.mgr = m.empno
10=9+1

8哪些人是领导?
select distinct m.ename
from emp_afei e join emp_afei m
on e.mgr = m.empno
9哪些部门没有员工?
select e.ename,d.dname
from emp_afei e right join dept_afei d
on e.deptno = d.deptno
where e.empno is null
(解决结果集只包含匹配不上的记录.where子句执行在外连接之后)
11=10+1
如果部门表里的某条记录的deptno在emp表找不到匹配,在内连接中,它被过滤,
e表的empno的特性是唯一且非空的(主键约束),居然e.empno is null,说明null是外连接时为了驱动表中那条匹配不上的记录出现在结果集中,在匹配表中模拟的null记录.
10哪些人是员工,哪些人不是领导?
select e.empno,m.ename
from emp_afei e right join emp_afei m
on e.mgr = m.empno
where e.empno is null

from emp_afei e right join emp_afei m
15=9+(10(m表中有10条记录)-4(m表中有4条匹配记录 ))
from emp_afei e left join emp_afei m
10(结果集)=9+(10(e表中有10条记录)-9(e表中有9条匹配记录))
11 哪些部门没有叫zhangwuji的?
select d.dname
from emp_afei e right join  dept_afei d
on e.deptno = d.deptno
and e.ename = 'zhangwuji'
where e.empno is null

7) 课外练习(day04)(答案在Day05)
1zhangwuji的领导是谁?(表连接)
2zhangwuji领导谁?(表连接)
3哪些人是领导?(in exists join)
4哪些部门没有员工?(not in/not exists/outer join)
5哪些人是员工,哪些人不是领导?(not in/not exists/outer join)
Day05.txt
Grade级别
Lowsal最低工资
Hisal最高工资
Create table salgrade_hiloo(
Grade 
)
cross join  inner join   outer join
inner join(匹配)
  等值连接
  自连接
  非等值连接
outer join(匹配+不匹配)
  等值连接

  自连接
  非等值连接

所谓非等值连接表示两张表里的列不能写成等值表达式,而是写成between and之类.所以两个表之间有关系是指表里的列可以写成表达式,而不是等值表达式.
salgrade
grade  级别
lowsal 最低工资
hisal  最高工资

from后面跟子查询
emp,各个部门的平均工资dept_avgsal(depnto,avgsal)
select e.ename,e.salary,e.deptno
from emp_afei e join
      (select deptno,round(avg(salary)) avgsal
       from emp_afei 
       group by deptno) a
on e.deptno = a.deptno
and e.salary > a.avgsal

各个部门的平均工资,列出部门名称,平均工资
select max(d.dname),round(avg(salary))
from emp_afei e join dept_afei d
on e.deptno = d.deptno
group by d.deptno

select d.dname,a.avgsal
from dept_afei d join
      (select deptno,round(avg(salary)) avgsal
       from emp_afei 
       group by deptno) a
on d.deptno = a.depto

DML
insert一条记录时,若某些列为null值,有哪些语法实现?
insert into tabname values (1,'a',null,sysdate)
insert into tabname(c1,c2,c4)
values (1,'a',sysdate)
insert语句的两种语法形式?
insert into tabname values () insert一条记录
insert into tabname
select * from tabname1  insert多条记录
连接图解：
    

### 数据类型###
1) 课外练习答案day04
1zhangwuji的领导是谁?(表连接)
 select m.ename
 from emp_afei e join emp_afei m
 on e.mgr = m.empno
 and m.ename = 'zhangwuji'
2 zhanghangwuji领导谁?(表连接)
 select e.ename
 from emp_afei e join emp_afei m
 on e.mgr = m.empno
 and m.ename = 'zhangwuji'
3哪些人是领导?(in exists join)
 select ename from emp_afei
 where empno in (select mgr from emp_afei)
 select ename from emp_afei o
 where exists
            (select 1 from emp_afei i
             where o.empno = i.mgr)
 select distinct m.ename
 from emp_afei e join emp_afei m
 on e.mgr = m.empno
4哪些部门没有员工?(not in/not exists/outer join)
 select dname from dept_afei
 where deptno not in 
               (select deptno from emp_afei)
 select dname from dept_afei o
 where not exists 
             (select 1 from emp_afei i
              where o.deptno = i.deptno)
 select d.dname
 from emp_afei e right join dept_afei d
 on e.deptno = d.deptno
 where e.empno is null
5哪些人是员工,哪些人不是领导?(not in/not exists/outer join)
 select ename from emp_afei
 where empno not in (
               select mgr from emp_afei
               where mgr is not null)
 select ename from emp_afei o
 where not exists
             (select 1 from emp_afei i
              where o.empno = i.mgr)
 select m.ename
 from emp_afei e right join emp_afei m
 on e.mgr = m.empno
 where e.empno is null
cross join (笛卡尔积)

rownum 伪列,记录号
若用rownum选择出记录,编号必须从1开始.
分页问题
第一页
select rownum,ename
from emp_afei
where rownum <= 3;
第二页
select rn,ename
from (
      select rownum rn,ename
      from emp_afei
      where rownum <= 6)
where rn between 4 and 6
排名问题
按工资排名的前三条记录
select rownum,ename,salary
from emp_hiloo
where rownum <=3
order by salary desc;(错)

select rownum,ename,salary
from ( select ename,salary
       from emp_afei
       order by salary desc)
where rownum <= 3

update语句的中set后面的=是什么含义?where后面的=是什么含义?
set c1 = null (= 赋值)
where c1 = null (= 等号)

update和delete语句中的where子句是什么含义?
用来确定对表里的哪些记录要进行update或delete操作,没有where子句多表里的所有记录update或delete
update
set
where c1 = (select ...)
rename 关键字 17
commit

1011 abc 1000 10 'clerk'
update 1001 1000-->2000
delete 1011
commit
如何编写和运行一个sql脚本(文本文件)
1 编辑文件
在linux环境下已经编写好了test.sql,做一个鼠标右键的copy

在20,23,26机器上,
vi test.sql
按a i o进入编辑模式,paste,按esc键,再按:wq!回车

2 运行文件
sun-server% sqlplus openlab/open123 @test.sql
@表示运行
SP2-0310: unable to open file "test.sql"在当前目录下没有test.sql文件
sqlplus openlab/open123 ../test.sql

cd ..
sun-server% sqlplus openlab/open123 @test.sql

SQL>@test.sql

数据库对象 PL/SQL
create or replace function test
insert into test values (1,1)
            *
ERROR at line 1:
ORA-04044: procedure(存储过程), function(函数), package(包), or type is not allowed here

事务(transaction 交易)
事务里包含的DML语句
事务的结束
commit 提交,(dml操作的数据入库了)
rollback 回滚 撤销(DML操作被取消)
sqlplus正常退出=commit
DDL语句自动提交
开始
上一个事务的结束是下一个事务的开始.
一致状态
数据库的数据被事务改变.
oltp online transaction processing联机事务处理系统 高并发系统

事务的隔离级别 read committed(读已经提交了的数据)


如果不commit----->commit rollback
1如果不commit,其他session是看不见你的操作
2如果不commit,会阻塞操作同一条记录的事务(session),commit才能释放所有DML加的锁.
3如果不commit,系统做DML操作,会将old data放入rollback segment(回滚段) ,所占用的回滚段资源不释放.

DML系统会自动给表及表里的记录加锁
表级共享锁
行级排他锁
	表级共享锁 	行级排他锁
s1	ok		ok
s2	ok		enqueue wait
s3	ok		ok

执行DDL语句,系统自动加DDL排他锁
SQL> drop table test purge;
drop table test purge
           *
ERROR at line 1:
ORA-00054: resource busy(资源忙 test表) and acquire with NOWAIT specified (dml wait,ddl nowait 如果加不上锁,报错退出)

DDL语句
字符类型
varchar2,必须带宽度, 按字符串的实际长度存,本身的数据是变化,对空格敏感
char,可以不带宽度,缺省宽度是1,按字符串的定义长度存,本身的数据是固定长度的.对空格不敏感
数值类型

number类型
create table test90
(c1 number,
 c2 number(6),
 c3 number(4,2),
 c4 number(2,4),
 c5 number(3,-3))

四舍五入
number(6) 表示6为整数 999999
number(4,2) 表示小数点后2位,整数位2位 99.99
number(2,4) 表示小数点后4位,能填数字的位数是2位 0.0099
number(3,-3) 999000 999123-->999000 
                    999511-->报错

user_tables 是一张系统表,里面记录当前用户所有的表的信息,里面没有记录表的创建日期.
user_objects 是一张系统表,里面记录当前用户所有的数据库对象的信息.created的列记录数据库对象(如表)的创建日期.
user_tables和user_objects这两张表的关系体现在table_name和object_name都记录的是表名.

data block 数据块,操作数据的最小逻辑(物理)单元,最少读一个block的数据

HWM high water mark 高水位线,表示曾经插入数据的最高位置
FTS full table scan 全表扫描,把表里的所有记录读一遍,把HWM之下的所有data block读一遍

truncate table 释放空间,HWM下移
delete 不释放空间,HWM不动
不适合用delete命令删大表.

课内练习
1 列出工资级别为3级,5级的员工
  select e.ename,e.salary,s.grade
  from emp_afei e join salgrade_afei s
  on e.salary between s.lowsal and s.hisal
  and s.grade in (3,5)
2 列出各个工资级别有多少人?
  select s.grade,count(e.empno)
  from emp_afei e join salgrade_afei s
  on e.salary between s.lowsal and s.hisal
  group by s.grade
  order by s.grade
3 列出各个工资级别有多少人?(包含0级)
  select s.grade,count(e.empno)
  from emp_afei e right join salgrade_afei s
  on e.salary between s.lowsal and s.hisal
  group by s.grade
  order by s.grade
特别注意count不要写*或者s.grade

课外练习day05
1按工资排名的第4到第6名员工.
###关键点###
课外练习day05答案

按工资排名的第4到第6名员工.
select rn,ename,salary
from 
    (select rownum rn,ename,salary
     from (select ename,salary
           from emp_afei
           order by salary desc)
     where rownum <= 6
    )
where rn >= 4 

####1）事务####

####约束 constraint (安检)####
primary key(主键)
foreign key(外键)
unique key (唯一键)
not null(非空)
check (检查)

主键 (表中不会出现重复记录)
列级约束
create table test
(c1 number(2) 
    constraint test_c1_pk primary key,
 c2 number(3))

    constraint test_c1_pk primary key,
               *
ERROR at line 3:
ORA-02264: name already used by an existing constraint (名字被存在的约束使用了)

SQL> select table_name from user_constraints
  2  where constraint_name = 'TEST_C1_PK';
哪张表里有叫TEST_C1_PK这个约束名.

ORA-00001: unique constraint (HILOO(用户名) .TEST_C1_PK) violated(冲突)

PK=UK + NN

表级约束
create table test(
c1 number(2),
c2 number,
constraint test_c1_pk primary key(c1)
)
表中有三列c1,c2,c3,c1和c2做成联合主键
create table test(
c1 number,
c2 number,
constraint test_c1_c2_pk primary key(c1,c2),
c3 number
)
没有constraint关键字,系统用自动起名字sys_c数字.

not null
create table test
(c1 number constraint test_c1_pk primary key,
 c2 number not null);
not null约束没有表级形式

unique (pk)
相同点:都要保证唯一性
区别:uk允许为null,而且可以多个null值,一个表中只能有一个pk约束,可以有多个uk约束.
create table test
(c1 number constraint test_c1_pk primary key,
 c2 number constraint test_c2_uk unique)

create table test(
c1 number primary key,
c2 number primary key,
c3 number unique,
c4 number unique)  (报错,一张表只能有一个primary key)

create table test(
c1 number constraint test_c1_pk primary key,
c2 number constraint test_c2_uk unique,
c3 number constraint test_c3_uk unique,
c4 number ) 
c2上定义了一个唯一键 c3上定义了一个唯一键

create table test(
c1 number constraint test_c1_pk primary key,
c2 number,
c3 number,
constraint test_c2_c3_uk unique (c2,c3),
c4 number)
c2,c3联合唯一键

check
create table test(
c1 number(3) constraint test_c1_ck
             check (c1 > 100))

create table test(
c1 number(3),
constraint test_c1_ck check (c1 > 100))

外键
parent table(父表)上定义唯一列(pk/uk)
child table(子表)上定义外键列(fk)

1 先create parent table(pk/uk),再create child table(fk)
2 先insert into parent table,再insert into child table
3 先delete from child table,再delete from parent table
4 先drop child table,再drop parent table

reference 引用
create table parent
(c1 number(3))

create table child
(c1 number(2) constraint child_c1_pk
              primary key,
 c2 number(3) constraint child_c2_fk
              references parent(c1))

              references parent(c1))
                                *
ERROR at line 5:
ORA-02270: no matching unique or primary key for this column-list
在c1上没有定义uk或pk

alter table parent
add constraint parent_c1_pk primary key(c1);
给c1列增加主键约束

insert into child values (1,1)
ORA-02291: integrity constraint(完整性约束) (HILOO.CHILD_C2_FK) violated - parent key not found (父键值没发现)
违反fk约束

insert into parent values (1);
insert into child values (1,1)

delete from parent where c1 = 1;
ORA-02292: integrity constraint (HILOO.CHILD_C2_FK) violated - child record
found(子记录被发现)

delete from child where c2 = 1;
delete from parent where c1 = 1;

drop table parent purge;
ORA-02449: unique/primary keys in table referenced by foreign keys
在parent table上的pk/uk正在fk所引用

drop table child purge;
drop table parent purge;

drop table parent cascade constraints purge;
cascade constraints 级联约束,child table本身没被删除,只是先把子表上的fk约束删除,再删parent table.

表级约束
create table child
(c1 number(2) constraint child_c1_pk 
              primary key,
 c2 number(3),
 constraint child_c2_fk foreign key(c2)
            references parent(c1)
)

外键约束另外两种定义方法
create table child1
(c1 number(2) constraint child1_c1_pk
              primary key,
 c2 number(3) constraint child1_c2_fk
              references parent(c1)
              on delete cascade)
on delete cascade :级联删除会影响到对parent table的删除,先delete from child1,再delete from
parent

delete from parent where c1 = 1;
create table child2
(c1 number(2) constraint child2_c1_pk
              primary key,
 c2 number(3) constraint child2_c2_fk
              references parent(c1)
              on delete set null)

delete from parent where c1 = 1
等价于以下操作
SQL> update child2 set c2 = null
  2  where c2 = 1;
SQL> delete from parent where c1 = 1;

table 
DDL(数据类型 约束)
transaction (包含一堆DML)

4000
100 
1000
3100

视图(view)
create table test_t1
as
select * from test
where c1 = 1;
create or replace view test_v1
as
select * from test
where c1 = 1;
desc test_v1
selelct * from test_v1

insert into test values (1,3);
select * from test_v1 (1,3)
insert into test_v1 values (1,4)
select * from test_v1;
select * from test;
insert into test_v1 values (2,3);
select * from test_v1;(没有)
select * from test;(2,3)

drop table test purge;
select * from test_v1; 
SQL> desc test_v1
ERROR:
ORA-24372: invalid object for describe
无法描述无效对象的结构

SQL> select text from user_views
  2  where view_name = 'TEST_V1';

TEXT
-----------------------------------------
select "C1","C2" from test
where c1 = 1

view是一条select语句. select语句中包含的表为源表.通过view对源表做DML操作.

view作用
1 create view (deptno = 30)
  grant view to user
  限定用户查询的数据 子集
2 简化查询语句
3 create view beijing
  as
  select * from haidian
  union all
  select * from xicheng
...
  超集
view的类型
1 简单view (DML)
2 复杂view  (不能DML)

create or replace view avgscore_v
select s.name,a.avgscore
from student s,
     (select sid,round(avg(score)) avgscore
      from stu_cour
      group by sid) a
on s.id = a.sid

view的约束
create or replace view test_ck
as
select * from test
where c1 = 1
with check option;
c1=2,违反where条件,2,3记录insert时报错

create or replace view test_ro
as
select * from test
where c1 = 1
with read only;
只读视图
#### 索引####
create index test_c1_idx
on test(c1);
对索引不能做desc,select,DML操作
rowid 代表一条记录的物理位置
属于哪个数据对象(table)
属于哪个数据文件的
属于数据文件的第几个数据块
属于数据块里的第几条记录
#### index的结构####
index记录rowid
index的结构是一棵平衡树,有三类数据块组成,根节点,分支节点,叶子节点,数据块的数据是排序的.根节点和分支节点用于导航,里面记录下一级节点的物理位置以及该节点包含的数据范围.叶子节点里记录的是index entry(索引项),由key值和rowid组成,key值是建索引的列在每条记录上的取值,rowid是记录的物理位置,所有的叶子节点做成双向链表(升序/降序),适用于范围查询.
用索引查询的路线图,从根节点出发,找相应的分支节点,叶子节点,最后要找到index entry,通过rowid定位
表里所需要的数据块,避免了全表扫描.

索引为什么提高查询效率,为select语句
有效地降低了读取数据块的数量.读取数据块,一种从文件里读,物理读 physical read,一种从内存读,逻辑读 logical read /buffer gets

建索引代价
空间,DML变慢


#### 哪些列适合建索引####
1 经常出现在where子句的列
2 pk/uk列
3 经常出现在表连接的列
4 fk列 parent.pk列 = child.fk列
5 经常用于group by,order by的列
7 where c1 is null(全表扫描),索引里不记null值,
 该列有大量null值,找not null值用索引会快

#### 索引类型####
非唯一性索引,提高查询效率
唯一性索引,解决唯一性.等价建唯一性约束
create unique index test_c2_idx
on test(c2);

insert into test (c2) values (1)
*
ERROR at line 1:
ORA-00001: unique constraint(HILOO.TEST_C2_IDX ) violated

联合索引
create index test_c1_c2_idx
on test(c1,c2)
where c1 = 1 and c2 = 1

select ename from emp_hiloo
where salary*12 > 60000
where salary > 5000
如果salary建索引,where salary > 5000(用),where salary*12 > 60000(不能用)

where upper(ename) = 'ZHANGWUJI'

where c1 = 100 c1是varchar2类型
where to_number(c1) = 100

where ename like 'a%'
where substr(ename,1,1) = 'a'

deptno not in (20,30)
depotno in (10)

#### 函数索引####
create index test_c1_funidx
on test(round(c1));
where round(c1) = 10

create index student_name_idx
on student(name);

#### 序列号####
sequence
为table里的主键服务,产生主键值
唯一值产生器
sequence_name.nextval

为student表的id建sequence
insert into student(student_id.nextval...
为course表的id建sequence
insert into course (course_id.nextval...

创建序列如下：
create sequence SEQ_TEST100
minvalue 1
maxvalue 999999999999999999999999999
start with 11
increment by 1
cache 10;

函数
create or replace function dept_avgsal
(p_deptno number) --定义参数,数据类型不能有宽度
return number    --定义函数的返回类型
is
  v_salary emp_hiloo.salary%type;     --变量v_salary 的类型跟表emp_hiloo里的salary的类型定义一致
begin
  select round(avg(salary)) into v_salary
  from emp_hiloo
  where deptno = p_deptno;    --select当且仅当返回一条记录用select into语法,表示把select语句的执行结果赋值给v_salary
  return v_salary;       --返回函数值 
end;
.不运行,回到SQL>下
/表示运行
show error
SQL> select dept_avgsal(10) from dual;





练习
用语法实现多对多关系
student
id pk
name not null

course
id pk
name not null

stu_cour
sid fk -->student(id)
cid fk -->course(id)
pk(sid,cid)
score check [0,100](between and) 
#### 数据库日期比较####
Sql代码：
1	timesten内存数据库比较日期是不是同一天,低效的方法  
2	to_char(create_date,'yyyymmdd')=to_char(sysdate NUMTODSINTERVAL(60*60*24,'SECOND'),'yyyymmdd')  
3	oracle 数据库低效的方法  
4	to_char(create_date,'yyyymmdd')=to_char(sysdate-1,'yyyymmdd')   
5	2个数据库通用高效的方法  
6	trunc(create_date)=trunc(sysdate)-NUMTODSINTERVAL(1,'DAY')  
查找数据库里的表，索引等
支持oracle的模糊查询如select * from user_tables where table_name like '%_PROJECT';查表名以PROJECT结尾的表（注：区别大小写）
查所有用户的表在all_tables
主键名称、外键在all_constraints
索引在all_indexes
但主键也会成为索引，所以主键也会在all_indexes里面。
具体需要的字段可以DESC下这几个view，dba登陆的话可以把all换成dba。

查询用户表的索引(非聚集索引):
select * from user_indexes
where uniqueness = 'NONUNIQUE'

查询用户表的主键(聚集索引):
select * from user_indexes
where uniqueness = 'UNIQUE'

1、	查找表的所有索引（包括索引名，类型，构成列）：
select t.*,i.index_type from user_ind_columns t,user_indexes i where t.index_name = i.index_name and t.table_name = i.table_name and t.table_name = 要查询的表
2、查找表的主键（包括名称，构成列）：
select cu.* from user_cons_columns cu, user_constraints au where cu.constraint_name = au.constraint_name and au.constraint_type = 'P' and au.table_name = 要查询的表
3、查找表的唯一性约束（包括名称，构成列）：
select column_name from user_cons_columns cu, user_constraints au where cu.constraint_name = au.constraint_name and au.constraint_type = 'U' and au.table_name = 要查询的表
4、查找表的外键（包括名称，引用表的表名和对应的键名，下面是分成多步查询）：
select * from user_constraints c where c.constraint_type = 'R' and c.table_name = 要查询的表
查询外键约束的列名：
select * from user_cons_columns cl where cl.constraint_name = 外键名称
查询引用表的键的列名：
select * from user_cons_columns cl where cl.constraint_name = 外键引用表的键名
5、查询表的所有列及其属性
select t.*,c.COMMENTS from user_tab_columns t,user_col_comments c where t.table_name = c.table_name and t.column_name = c.column_name and t.table_name = 要查询的表
####数据唯一Id：####
1.	用Oracle来生成UUID，做法很简单，如下：select sys_guid() from dual;数据类型是 raw(16) 有32个字符。
create table test_guid3(
    id varchar(50)
)
select * from test_guid3;
insert into test_guid3(id) values(sys_guid())
----------- ----------------------------------------
       1000 7CD5B7769DF75CEFE034080020825436
       1100 7CD5B7769DF85CEFE034080020825436
       1200 7CD5B7769DF95CEFE034080020825436
       1300 7CD5B7769DFA5CEFE034080020825436
### 名词###

#### Oracle的方案（Schema）和用户（User）的区别####
 
从定义中我们可以看出方案（Schema）为数据库对象的集合，为了区分各个集合，我们需要给这个集合起个名字，这些名字就是我们在企业管理器的方案下看到的许多类似用户名的节点，这些类似用户名的节点其实就是一个schema，schema里面包含了各种对象如tables, views, sequences, stored procedures, synonyms, indexes, clusters, and database links。
 
   一个用户一般对应一个schema,该用户的schema名等于用户名，并作为该用户缺省schema。这也就是我们在企业管理器的方案下看到schema名都为数据库用户名的原因。Oracle数据库中不能新创建一个schema，要想创建一个schema，只能通过创建一个用户的方法解决(Oracle中虽然有create schema语句，但是它并不是用来创建一个schema的)，在创建一个用户的同时为这个用户创建一个与用户名同名的schem并作为该用户的缺省shcema。即schema的个数同user的个数相同，而且schema名字同user名字一一对应并且相同，所有我们可以称schema为user的别名，虽然这样说并不准确，但是更容易理解一些。
 
   一个用户有一个缺省的schema，其schema名就等于用户名，当然一个用户还可以使用其他的schema。如果我们访问一个表时，没有指明该表属于哪一个schema中的，系统就会自动给我们在表上加上缺省的sheman名。比如我们在访问数据库时，访问scott用户下的emp表，通过select * from emp; 其实，这sql语句的完整写法为select * from scott.emp。在数据库中一个对象的完整名称为schema.object，而不属user.object。类似如果我们在创建对象时不指定该对象的schema，在该对象的schema为用户的缺省schema。这就像一个用户有一个缺省的表空间，但是该用户还可以使用其他的表空间，如果我们在创建对象时不指定表空间，则对象存储在缺省表空间中，要想让对象存储在其他表空间中，我们需要在创建对象时指定该对象的表空间。
 
   oracle中的schema就是指一个用户下所有对象的集合，schema本身不能理解成一个对象，oracle并没有提供创建schema的语法，schema也并不是在创建user时就创建，而是在该用户下创建第一个对象之后schema也随之产生，只要user下存在对象，schema就一定存在，user下如果不存在对象，schema也不存在；这一点类似于temp tablespace group，另外也可以通过oem来观察，如果创建一个新用户，该用户下如果没有对象则schema不存在，如果创建一个对象则和用户同名的schema也随之产生。
####Oracle中User与Schema的简单理解####
技术积累（126）  
版权声明：本文为博主原创文章，未经博主允许不得转载。
方案（Schema）为数据库对象的集合，为了区分各个集合，我们需要给这个集合起个名字，这些名字就是我们在企业管理器的方案下看到的许多类似用户名的节点，这些类似用户名的节点其实就是一个schema，schema里面包含了各种对象如tables, views, sequences, stored procedures, synonyms, indexes, clusters, and database links。  一个用户一般对应一个schema,该用户的schema名等于用户名，并作为该用户缺省schema。
SQL Server中的Schema
SQL Server中一个用户有一个缺省的schema，其schema名就等于用户名，这也就是我们在企业管理器的方案下看到schema名都为数据库用户名的原因。当然一个用户还可以使用其他的schema。如果我们访问一个表时，没有指明该表属于哪一个schema中的，系统就会自动给我们在表上加上缺省的sheman名。比如我们在访问数据库时，访问scott用户下的emp表，通过select * from emp; 其实，这sql语句的完整写法为select * from scott.emp。在数据库中一个对象的完整名称为schema.object，而不属user.object。类似如果我们在创建对象时不指定该对象的schema，在该对象的schema为用户的缺省schema。这就像一个用户有一个缺省的表空间，但是该用户还可以使用其他的表空间，如果我们在创建对象时不指定表空间，则对象存储在缺省表空间中，要想让对象存储在其他表空间中，我们需要在创建对象时指定该对象的表空间。

Oracle中的Schema
Oracle中的schema就是指一个用户下所有对象的集合，schema本身不能理解成一个对象，oracle并没有提供创建schema的语法，schema也并不是在创建user时就创建，而是在该用户下创建第一个对象之后schema也随之产生，只要user下存在对象，schema就一定存在，user下如果不存在对象，schema也不存在；如果创建一个新用户，该用户下如果没有对象则schema不存在，如果创建一个对象则和用户同名的schema也随之产生。实际上在使用上，shcema与user完全一样，没有什么区别，在出现schema名的地方也可以出现user名。

Tablspace 
逻辑上用来放objects,，这是个逻辑概念，本质上是一个或者多个数据文件的集合，物理上对应磁盘上的数据文件或者裸设备。

数据文件
具体存储数据的物理文件，是一个物理概念。一个数据文件只能属于一个表空间，一个表空间可以包含一个或多个数据文件。一个数据库由多个表空间组成，一个表空间只能属于一个数据库。

下边是源自网络的一个形象的比喻
我们可以把Database看作是一个大仓库，仓库分了很多很多的房间，Schema就是其中的房间，一个Schema代表一个房间，Table可以看作是每个Schema中的床，Table（床）被放入每个房间中，不能放置在房间之外，那岂不是晚上睡觉无家可归了，然后床上可以放置很多物品，就好比 Table上可以放置很多列和行一样，数据库中存储数据的基本单元是Table，现实中每个仓库放置物品的基本单位就是床， User就是每个Schema的主人，（所以Schema包含的是Object，而不是User），user和schema是一一对应的，每个user在没有特别指定下只能使用自己schema（房间）的东西，如果一个user想使用其他schema（房间）的东西，那就要看那个schema（房间）的user（主人）有没有给你这个权限了，或者看这个仓库的老大（DBA）有没有给你这个权限了。换句话说，如果你是某个仓库的主人，那么这个仓库的使用权和仓库中的所有东西都是你的（包括房间），你有完全的操作权，可以扔掉不用的东西从每个房间，也可以放置一些有用的东西到某一个房间，你还可以给每个User分配具体的权限，也就是他到某一个房间能做些什么，是只能看（Read-Only），还是可以像主人一样有所有的控制权（R/W），这个就要看这个User所对应的角色Role了。
#### oracle的schema的含义####
在现在做的Kraft Catalyst 项目中，Cransoft其中有一个功能就是schema refresh. 一直不理解schema什么意思，也曾经和同事讨论过，当时同事就给我举过一个例子，下面会详细说的。其实schema是Oracle中的，其他数据库中不知道有没有这个概念。
首先,可以先看一下schema和user的定义：
A schema is a collection of database objects (used by a user).
Schema objects are the logical structures that directly refer to the database’s data.
A user is a name defined in the database that can connect to and access objects.
Schemas and users help database administrators manage database security.
从中我们可以看出,schema为数据库对象的集合，为了区分各个集合，需要给这个集合起个名字，这些名字就是在企业管理器的方案下看到的许多类似用户名的节点，这些类似用户名的节点其实就是一个schema。
schema里面包含了各种对象如tables, views, sequences, stored procedures, synonyms, indexes, clusters, and database links。
一个用户一般对应一个schema，该用户的schema名等于用户名，并作为该用户缺省schema。这也就是在企业管理器的方案下看到schema名都为数据库用户名的原因。
Oracle数据库中不能新创建一个schema，要想创建一个schema，只能通过创建一个用户的方法解决(Oracle中虽然有create schema语句，但是它并不是用来创建一个schema的)。在创建一个用户的同时，为这个用户创建一个与用户名同名的schem并作为该用户的缺省 shcema。即schema的个数同user的个数相同，而且schema名字同user名字一一 对应并且相同，所有我们可以称schema为user的别名，虽然这样说并不准确，但是更容易理解一些。
一个用户有一个缺省的schema，其schema名就等于用户名，当然一个用户还可以使用其他的schema。如果我们访问一个表时，没有指明该表属于 哪一个schema中的，系统就会自动给我们在表上加上缺省的sheman名。比如我们在访问数据库时，访问scott用户下的emp表，通过 select * from emp; 其实，这sql语句的完整写法为select * from scott.emp。在数据库中一个对象的完整名称为schema.object，而不属user.object。类似如果我们在创建对象时不指定该对象 的schema，在该对象的schema为用户的缺省schema。这就像一个用户有一个缺省的表空间，但是该用户还可以使用其他的表空间，如果我们在创 建对象时不指定表空间，则对象存储在缺省表空间中，要想让对象存储在其他表空间中，需要在创建对象时指定该对象的表空间。
有人举了个很生动的例子，来说明Database、User、Schema、Tables、Col、Row等之间的关系
“可以把Database看作是一个大仓库，仓库分了很多很多的房间，Schema就是其中的房间，一个Schema代表一个房间，Table可以看作是每个Schema中的床，Table（床）就被放入每个房间中，不能放置在房间之外，那岂不是晚上睡觉无家可归了。
然后床上可以放置很多物品，就好比Table上可以放置很多列和行一样，数据库中存储数据的基本单元是Table，现实中每个仓库放置物品的基本单位就是床， User就是每个Schema的主人（所以Schema包含的是Object，而不是User）。
其实User是对应与数据库的（即User是每个对应数据库的主人），既然有操作数据库（仓库）的权利，就肯定有操作数据库中每个Schema（房间）的 权利，就是说每个数据库映射的User有每个Schema（房间）的钥匙，换句话说，如果他是某个仓库的主人，那么这个仓库的使用权和仓库中的所有东西都 是他的（包括房间），他有完全的操作权，可以扔掉不用的东西从每个房间，也可以放置一些有用的东西到某一个房间。还可以给User分配具体的权限，也就是 他到某一个房间能做些什么，是只能看（Read-Only），还是可以像主人一样有所有的控制权（R/W），这个就要看这个User所对应的角色Role 了”
从定义中我们可以看出schema为数据库对象的集合，为了区分各个集合，我们需要给这个集合起个名字，这些名字就是我们在企业管理器的方案下看到的许多类似用户名的节点，这些类似用户名的节点其实就是一个schema，schema里面包含了各种对象如tables, views, sequences, stored procedures, synonyms, indexes, clusters, and database links。
一个用户一般对应一个schema,该用户的schema名等于用户名，并作为该用户缺省schema。这也就是我们在企业管理器的方案下看到schema名都为数据库用户名的原因。Oracle数据库中不能新创建一个schema，要想创建一个schema，只能通过创建一个用户的方法解决(Oracle中虽然有create schema语句，但是它并不是用来创建一个schema的)，在创建一个用户的同时为这个用户创建一个与用户名同名的schem并作为该用户的缺省shcema。即schema的个数同user的个数相同，而且schema名字同user名字一一 对应并且相同，所有我们可以称schema为user的别名，虽然这样说并不准确，但是更容易理解一些。
一个用户有一个缺省的schema，其schema名就等于用户名，当然一个用户还可以使用其他的schema。如果我们访问一个表时，没有指明该表属于哪一个schema中的，系统就会自动给我们在表上加上缺省的sheman名。比如我们在访问数据库时，访问scott用户下的emp表，通过select * from emp; 其实，这sql语句的完整写法为select * from scott.emp。在数据库中一个对象的完整名称为schema.object，而不属user.object。类似如果我们在创建对象时不指定该对象的schema，在该对象的schema为用户的缺省schema。这就像一个用户有一个缺省的表空间，但是该用户还可以使用其他的表空间，如果我们在创建对象时不指定表空间，则对象存储在缺省表空间中，要想让对象存储在其他表空间中，我们需要在创建对象时指定该对象的表空间。
咳，说了这么多，给大家举个例子，否则，一切枯燥无味！
SQL> Gruant dba to scott
SQL> create table test(name char(10));
Table created.
SQL> create table system.test(name char(10));
Table created.
SQL> insert into test values('scott');
1 row created.
SQL> insert into system.test values('system');
1 row created.
SQL> commit;
Commit complete.
SQL> conn system/manager
Connected.
SQL> select * from test;

NAME
----------
system
SQL> ALTER SESSION SET CURRENT_SCHEMA = scott; --改变用户缺省schema名
Session altered.
SQL> select * from test;

NAME
----------
scott
SQL> select owner ,table_name from dba_tables where table_name=upper('test');
OWNER TABLE_NAME
------------------------------ ------------------------------
SCOTT TEST
SYSTEM TEST
--上面这个查询就是我说将schema作为user的别名的依据。实际上在使用上，shcema与user完全一样，没有什么区别，在出现schema名的地方也可以出现user名。
表空间：
一个表空间就是一片磁盘区域,他又一个或者多个磁盘文件组成,一个表空间可以容纳许多表、索引或者簇等  
  每个表空间又一个预制的打一磁盘区域称为初始区间（initial   extent）用完这个区间厚在用下一个，知道用完表空间，这时候需要对表空间进行扩展，增加数据文件或者扩大已经存在的数据文件
 
 

instance是一大坨内存sga,pga....和后台的进程smon pmon.....组成的一个大的应用。
schema就是一个用户和他下面的所有对象。。
tablspace 逻辑上用来放objects.物理上对应磁盘上的数据文件或者裸设备。
 在Oracle中，结合逻辑存储与物理存储的概念，我们可以这样来理解数据库、表空间、SCHEMA、数据文件这些概念：
      数据库是一个大圈，里面圈着的是表空间，表空间里面是数据文件，那么schema是什么呢？schema是一个逻辑概念，是一个集合，但schema并不是一个对象，oracle也并没有提供创建schema的语法。
schema：
      一般而言，一个用户就对应一个schema,该用户的schema名等于用户名，并作为该用户缺省schema，用户是不能创建schema的，schema在创建用户的时候创建，并可以指定用户的各种表空间（这点与PostgreSQL是不同，PostgreSQL是可以创建schema并指派给某个用户）。当前连接到数据库上的用户创建的所有数据库对象默认都属于这个schema（即在不指明schema的情况下），比如若用户scott连接到数据库，然后create table test(id int not null)创建表，那么这个表被创建在了scott这个schema中；但若这样create kanon.table test(id int not null)的话，这个表被创建在了kanon这个schema中，当然前提是权限允许。
      创建用户的方法是这样的：
      create user 用户名 identified by 密码 
      default tablespace 表空间名 
      temporary tablespace 表空间名 
      quota 限额  （建议创建的时候指明表空间名）
由此来看，schema是一个逻辑概念。
      但一定要注意一点：schema好像并不是在创建user时就创建的，而是在该用户创建了第一个对象之后才将schema真正创建的，只有user下存在对象，他对应的schema才会存在，如果user下不存在任何对象了，schema也就不存在了；
 
数据库：
     在oracle中，数据库是由表空间来组成的，而表空间里面是具体的物理文件---数据文件。我们可以创建数据库并为其指定各种表空间。
 
表空间：
     这是个逻辑概念，本质上是一个或者多个数据文件的集合。
 
数据文件：
     具体存储数据的物理文件，是一个物理概念。
     一个数据文件只能属于一个表空间，一个表空间可以包含一个或多个数据文件。一个数据库由多个表空间组成，一个表空间只能属于一个数据库。
