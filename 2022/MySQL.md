# MySQL架构

## MySQL逻辑架构

连接——》处理——》引擎——》存储

## SQL执行顺序

From——》on ——》Where——》Group By——》Having——》Order By——》Limit

# 索引优化分析

## 7种join理论

![](Picture\内连接.jpg)

```sql
SELECT <select_list> FROM table a LEFT JOIN table b ON a.key = b.key
```

![](Picture\左连接.jpg)

```sql
SELECT <select_list> FROM table a LEFT JOIN table b on a.key = b.key
```

![](Picture\右连接.jpg)

```sql
SELECT <select_list> FROM table a RIGHT JOIN table b on a.key = b.key
```

![](Picture\5号Join.jpg)

```sql
SELECT <select_list> FROM table a LEFT JOIN table b ON a.key = b.key WHERE b.key IS NULL
```

![](Picture\6号join.jpg)

```sql
SELECT <select_list> FROM table a RIGHT JOIN table b ON a.key = b.key WHERE a.key IS NULL
```

<img src="Picture\全连接.jpg" style="zoom: 80%;" />

```sql
SELECT <select_list> FROM table a FULL OUTER JOIN table b ON a.key = b.key
```

![](Picture\7号join.jpg)

```sql
SELECT <select_list> FROM table a FULL OUTER JOIN table b ON a.key = b.key WHERE a.key IS NULL OR b.key IS NULL
```

**实例展示**

```sql
create database db01;

use db01;

CREATE TABLE t_dept(
id INT(11) NOT NULL auto_increment,
deptName VARCHAR(30) DEFAULT NULL,
iocAdd VARCHAR(40) DEFAULT NULL,
PRIMARY KEY(id)
)ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

CREATE TABLE t_emp(
id INT(11) NOT NULL AUTO_INCREMENT,
name VARCHAR(20) DEFAULT NULL,
deptId INT(11) DEFAULT NULL,
primary key(id),
KEY fk_dept_id(deptId)
)ENGINE=INNODB AUTO_INCREMENT=1 DEFAUlt CHARSET=utf8;

INSERT INTO t_dept(deptName, iocAdd) VALUES('RD', 11);
INSERT INTO t_dept(deptName, iocAdd) VALUES('HR', 12);
INSERT INTO t_dept(deptName, iocAdd) VALUES('MK', 13);
INSERT INTO t_dept(deptName, iocAdd) VALUES('MIS', 14);
INSERT INTO t_dept(deptName, iocAdd) VALUES('FD', 15);

INSERT INTO t_emp(name, deptId) VALUES('z3', 1);
INSERT INTO t_emp(name, deptId) VALUES('z4', 1);
INSERT INTO t_emp(name, deptId) VALUES('z5', 1);
INSERT INTO t_emp(name, deptId) VALUES('w5', 2);
INSERT INTO t_emp(name, deptId) VALUES('w6', 2);
INSERT INTO t_emp(name, deptId) VALUES('s7', 3);
INSERT INTO t_emp(name, deptId) VALUES('s8', 4);
INSERT INTO t_emp(name, deptId) VALUES('s9', 51);

SELECT * FROM t_emp, t_dept;

SELECT * FROM t_emp a INNER JOIN t_dept b on a.deptId = b.id; 

SELECT * FROM t_emp a LEFT JOIN t_dept b on a.deptId = b.id;

SELECT * FROM t_emp a RIGHT JOIN t_dept b on a.deptId = b.id;

SELECT * FROM t_emp a LEFT JOIN t_dept b on a.deptId = b.id WHERE b.id IS NULL;

SELECT * FROM t_emp a RIGHT JOIN t_dept b on a.deptId = b.id WHERE a.deptId IS NULL;

#MYSQL不支持FULL OUTER JOIN这种方法
#SELECT * FROM t_emp a FULL OUTER JOIN t_dept b on a.deptId = b.id;

SELECT * FROM t_emp a LEFT JOIN t_dept b on a.deptId = b.id 
UNION 
SELECT * FROM t_emp a RIGHT JOIN t_dept b on a.deptId = b.id WHERE a.deptId IS NULL;

SELECT * FROM t_emp a LEFT JOIN t_dept b on a.deptId = b.id WHERE b.id IS NULL
UNION
SELECT * FROM t_emp a RIGHT JOIN t_dept b on a.deptId = b.id WHERE a.deptId IS NULL;
```

## 索引简介

**排好序的快速查找数据结构**

数据本身之外，数据库还维护着一个满足特定查找算法的数据结构，这些数据结构以某种方式指向数据，这样就可以在这些数据结构的基础上实现高级查找算法，这种算法结构就是索引

索引本身很大，不可能全部存储在内存种，因此索引往往以索引文件的形式存储在磁盘上

索引，如果没有特别声明，都是指B树（多路搜索树，并不一定是二叉树）结构组织的索引

## 索引优势于劣势

* 优势
  * 提高数据检索的效率，降低数据库的IO成本
  * 降低数据排序的成本，降低了CPU的消耗
* 劣势
  * 实际上索引也是一张表，该表保存了主键与索引字段，并指向实体表的记录，所以索引列也是要占用空间的
  * 提高了查询数据的速度，降低了更新表的速度
  * 需要研究花时间建立最优秀的索引

## 索引的分类

* 单指索引——即一个索引只包含单个列，一个表可以有多个单列索引
* 唯一索引——索引列的指必须唯一，但允许空值
* 复合索引——即一个索引包含多个列

**基本语法**

* 创建

  ```sql
  CREATE [UNIQUE] INDEX indexName ON mytable(columnname(length));
  
  ALTER mytable ADD [UNIQUE] INDEX [indexName] ON(columnname(length));
  ```

* 删除

  ```sql
  DROP INDEX [indexName] ON mytable
  ```

* 查看

  ```sql
  SHOW INDEX FROM tablename
  ```

**四种方式添加数据表的索引**

```sql
ALTER TABLE t_name ADD PRIMARY KEY(column_list)

ALTER TABLE t_name ADD UNIQUE index_name(column_list)

ALTER TABLE t_name ADD INDEX index_name(column_list)

ALTER TABLE t_name ADD FULLTEXT index_name(column_list)
```

## 索引结构

* BTree索引
* Hash索引
* full-text全文索引
* R-Tree索引

## 适合建索引的情况

* 主键自动建立唯一索引
* 频繁作为查询条件的字段应该创建索引
* 查询中与其他表关联的字段，外键关系建立索引
* 频繁更新的字段不适合创建索引
* Where条件里用不到的字段不创建索引
* 高并发下倾向创建组合索引
* 查询种排序的字段，排序字段若通过索引去访问将大大提高排序速度
* 查询中统计或分组字段

## 不适合建索引的情况

* 表记录太少
* 经常增删改的表
* 重复，且平均分布的值，没有必要建立索引

## MySQL Query Optimizer

通过计算分析系统中收集到的统计信息，为客户端请求的Query提供他认为最有对的执行计划，最耗时间

MySQL常见瓶颈

* CPU：CPU在饱和的时候一般发生在数据装入内存或从磁盘上读取数据的时候
* IO：磁盘I/O瓶颈发生在装入数据远大于内存容量的时候
* 服务器硬件的性能瓶颈：top，free，iostat和vmstat来查看系统的性能状态

## EXPLAIN功能

* 表的读取顺序
* 数据读取操作的操作类型
* 那些索引可以被使用
* 那些索引被实际使用
* 表之间的引用
* 每张表有多少行被优化器查询

## EXPLAIN结果分析——id

select查询的序列号，包含一组数字，表示查询中执行select子句或操作表的顺序

* id相同，执行顺序由上至下
* id不同，如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行
* id相同，可以认为是一组，从上往下顺序执行，在所有组中，id不同，id值越大，优先级越高

## EXPLAIN结果分析——select_type

simple、primary、subquery、derived、union、union result

主要用于区别普通查询、联合查询、子查询等的复杂查询

* SIMPLE——简单的select查询，查询中不包含子查询或者union
* PRIMARY——查询中若包含任何复杂的子部分，最外层查询则被标记为
* SUBQUERY——在SELECT或WHERE列表中包含了子查询
* DERIVED——在FROM列表中包含的子查询被标记为DERIVED，MySQL会递归执行这些子查询，把结果放在临时表中
* UNION——若第二个SELECT出现UNION之后，则被标记为UNION，若UNION包含在FROM子句的子查询中，外层SELECT将被标记为DERIVED

## EXPLAIN结果分析——type

all、index、range、ref、eq_ref、const，system、null

最好到最差——

system > const > eq_ref > ref > range > index > all

**一般需要至少达到range级别，最好达到ref**

* system——表只有一行记录（等于系统表），这是const类型地特性，平时不会出现，这个可以忽略不记
* const——表示通过索引一次就找到了，const用于比较primary key或者unique索引，因为只匹配一行数据，所以很快，如果将主键置于WHERE列表中，MySQL就能将该查询转换为一个常量
* eq_ref——唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配，常见于主键或唯一索引扫描

* ref——非唯一性索引扫描，返回匹配某个单独值得所有行，本质上也是一种索引访问，他返回所有匹配某个单独值得行，然而，它可能会找到多个复合条件得行，所以他应该属于查找和扫描得混合体
* range——只检索给定范围得行，使用一个索引来选择行。key列显示使用了哪个索引，一般就是在你得where语句中出现了between、<、>、in等得查询，这种范围扫描索引比全表扫描要好，因为它只需要开始于索引得某一点，而结束语另一点，不用扫描全部索引
* index——Full Index Scan，index于All区别为index类型只遍历索引树，这通常比All快，因为索引文件通常比数据文件小
* all——Full Table Scan，将遍历全表以找到匹配的行

## EXPLAIN结果分析——possible_keys

显示可能应用在这张表中的索引，一个或多个，查询设计到的字段上若存在索引，则将该索引将被列出，**但不一定被查询实际使用**

## EXPLAIN结果分析——key

实际使用的索引，如果为NULL，则没有使用索引

查询中若使用了覆盖索引，则该索引仅出现在key列表中

> 覆盖索引，就是查询的字段的个数和顺序与索引字段相同

## EXPLAIN结果分析——key_len

表示索引中使用的字节数，可通过该列计算查询中使用的索引长度。在不损失精确度的情况下，长度越短越好

key_len显示的值为索引字段的最大可能长度，并非实际使用长度

## EXPLAIN结果分析——ref

显示索引的那一列被使用，如果可能的话，是一个常数。那些列或者常量被用于查找索引列上的值

## EXPLAIN结果分析——rows

根据表统计信息及索引选用情况，大致估算出找到所需的记录所需要读取的行数

## EXPLAIN结果分析——extra

* Using filesort——说明MySQL会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取，MySQL中无法利用索引完成的排序操作称为文件排序

* Using temporary——使用了临时表保存中间结果，MySQL在对查询结果排序时使用临时表，常见于排序order by和分组查询group by

* Using index——表示相应的select操作中使用了覆盖索引，避免访问了表中的数据行，如果同时出现了using where 表明索引被用来执行索引键值的查找，如果没有同时出现using where，表明索引用来读取数据并非执行查找动作

  **要使用覆盖索引，一定不能使用select\***

* Using where

* Using join buffer——使用了连接缓存

* Impossible where——where的值总是false

* select table optimized away

* distinct

## 实战结论

* 左链接，索引加右边；右链接，索引加左边
* 尽可能减少join语句中的NestedLoop的循环次数（**永远使用小结果集驱动大的结果集**）
* 优先优化NestedLoop的内层循环
* 保证join语句中被驱动表上join条件字段已经被索引
* 当无法保证被驱动表的join条件字段被索引且内存资源充足的情况下，不要太吝啬joinbuffer的设置

## 索引失效

1. 全值匹配我最爱

2. 最佳左前缀法则——查询从索引的最左前列开始并且不跳过索引中列

3. 不在索引列上做任何操作（计算、函数、（自动or手动）类型转换），会导致索引失效而转向全表扫描

4. 存储引擎不能使用索引中范围条件右边的列

5. 尽域使用覆盖索引（只访问索引的查询（索引列和杳询列一致）），减少select* 

6. mysql在使用不等于（!= 或者＜＞）的时候无法使用索引会导致全表扫描

7. is null，Js not null也无法使用索引

8. like以通配符开头（%abc..）mysql索引失效会变成全表扫描的操作

   > 解决like'%字符%'时索引不被使用的方法
   >

9. 字符串不加单引号索引失效

10. 少用or，用它来连接时会索引失效

# 查询截取分析

## 调优步骤

1. 观察，至少运行一天，查看生产状态的慢SQL情况
2. 开启慢查询日志，设置阈值
3. explain+慢SQL分析
4. show profile

## 小表驱动大表

当A表的数据大于B表的数据时，用exists优于in

当B表的数据大于A表的数据时，用in优于exists

## Order By语句调优

使用index方式调优，不要使用file sort的方式

Order By语句复合最左前缀原则

* Order by时select \* 是一个大忌
* 提高sort_buffer_size
* 提高max_length_for_sort_data

MySQL两种排列方式：文件排序或扫描有序索引排序

MySQL能为排序与查询使用相同的索引

* order by能使用索引最左前缀
* 如果WHERE使用索引的最左前缀定义为常量，则order by能使用索引

不能使用索引进行排序的情况——

* 排序不一致
* 丢失头索引
* 丢失中间索引
* 排序不是索引的部分
* 对于排序来说，多个相同条件也是范围查询

## Group By语句调优

与Order By语句调优类似

## 慢查询日志

默认情况下，MySQL数据库没有开启慢查询日志，需要我们手动来设置这个参数

当然，如果不是调优需要的话，一般不建议启动该参数

查看是否开启慢查询日志

`show variables like '%slow_query_log%'`

开启慢查询日志

`set global slow_query_log=1`

查看阈值时间

`show variables like '%long_query_time%'`

修改阈值时间

`set global long_query_time=3`

查看慢查询记录个数

`show global status like '%slow_queries%'`

> mysqldumpslow使用
>
> * 得到返回记录集最多的10个sql
>
>   `mysqldumpslow -s r -t 10 /var/lib/mysql/xxx-slow.log`
>
> * 得到访问次数最多的10个sql
>
>   `mysqldumpslow -s c -t 10 /var/lib/mysql/xxx-slow.log`
>
> * 得到按照时间顺序的前10条里面含有左连接的查询语句
>
>   `mysqldumpslow -s t -t 10 -g 'left join' /var/lib/mysql/xxx-slow.log`
>
> * 另外建议在使用这些命令时结合|和more使用，否则可能出现爆屏现象

## 批量插入数据脚本

**不如使用java插入**(bushi)

**存储过程不太熟练**

## 使用show profile

1. 是否支持，查看当前MySQL版本是否支持

   `show variables like 'profiling'`

2. 开启功能，默认是关闭，使用前需要开启

3. 运行sql

   > https://blog.csdn.net/weixin_43945983/article/details/113109009

4. 查看结果，show profiles

5. 诊断sql——show profile cpu, block io for query 上一步sql数字号码

   > ALL——显示所有开销信息
   >
   > BLOCK IO——显示块IO相关开销
   >
   > CONTEXT SWITCHES——上下文切换相关开销
   >
   > CPU——显示CPU相关开销信息
   >
   > IPC——显示发送和接受相关开销信息
   >
   > MEMORY——显示内存相关开销信息
   >
   > PAGE FAULTS——显示页面错误相关开销信息
   >
   > SOURCE——显示和source_function，source_file，source_line相关的开销信息
   >
   > SWAPS——显示交换次数相关开销信息

6. 日常开发需要注意的结论

   * converting heap to myisam——查询结果太大，内存都不够用了，开始往磁盘上搬了
   * create tmp table——创建临时表
   * copying to tmp table on disk——把内存中临时表复制到磁盘！
   * locked

## 全局查询日志

只能再测试时使用，不能在生产环境使用

* 配置启用

  ...

* 编码启用

  `set global general_log=1`

  `set global log_output='TABLE'`

  此后所有编写的sql语句，将会记录到mysql库里的general_log表

  `select * from mysql.general_log`

# MySQL锁机制

## 概述

分类——

* 对数据操作的类型分
  * 读锁（共享锁）——针对同一份数据，多个读操作可以同时进行而不会互相影响
  * 写锁（排他锁）——当前写操作没有完成之前，它会阻断其他写锁和读锁
* 对数据操作的粒度分
  * 表锁
  * 行锁

## 表锁（适用于MyISAM引擎）

[查看表上加过的锁]

`show open tables`

[手动增加表锁]

`lock table 表名字 read(write), 表名字2 read(write), ...`

[释放表锁]

`unlock tables`

**读锁会阻塞写，但是不会阻塞读，而写锁会把读和写都堵塞**

`show status like 'table%'`

参数table_locks_waited：出现表级锁定征用而发生等待的次数，此值高则说明存在着较重要的表级锁争用情况

## 行锁（适用于InnoDB引擎）

ACID——原子、一致、隔离、持久

并发事务带来的问题——

* 更新丢失
* 脏读
* 不可重复读
* 幻读

innodb事务过程中，自动上行锁，提交后自动解锁

**索引失效行锁将会变为表锁**

[间隙锁的危害](https://www.bilibili.com/video/BV1KW411u7vy?p=58&spm_id_from=pageDriver&vd_source=ef4cb053cb06864b58691c8387a45008)——

当我们用范围条件而不是等于条件检索数据时，并请求共享或排他锁时，InnoDB会给复合条件的已有数据的索引项枷锁：对于键值在条件范围内但并不存在的记录，兼做间隙

`show status like 'innodb_row_lock%'`

查看innodb状态

主要关注

innodb_row_lock_waits——系统启动后到现在总共等待的次数

innodb_row_lock_time_avg——等待平均时长

innodb_row_lock_time——等待总时长

**优化建议**

* 尽可能让所有数据检索都通过索引来完成，避免无行锁升级为表锁
* 合理设计索引，尽量缩小锁的范围
* 尽可能较少检索条件，避免间隙锁
* 尽量控制事务大小，减少锁定资源量和时间长度
* 尽可能低级别事务隔离

# 主从复制

MySQL版本一致且后台以服务运行

主从都配置在[mysqld]下

* 主机修改my.cnf

  ```sql
  server-id=1
  
  #启用二进制日志
  log-bin=自己本地的路径/mysqlbin
  
  #启用错误日志
  log-err=自己本地的路径/mysqlerr
  
  #根目录
  basedir=“自己本地路径”
  
  #临时目录
  tmpdir=“自己本地路径”
  
  #数据目录
  datadir=“自己本地路径/data/”
  
  #读写都可以
  read-only=0
  ```

* 从机修改my.cnf

  ```sql
  server-id=2
  
  #启动二进制日志
  log-bin=mysql-bin
  ```

主从服务器都需要重启服务

主从服务器都需要关闭防火墙（service iptables stop）

在windows主机上建立账户并授权slave

`grant replication slave on "*" to '用户名'@'主机ip' identified by '访问密码'`

刷新命令

`flush privileges`

查看master状态

`show master status`

在linux从机上配置需要复制的主机

`change master to master_host='主机ip', master_user='用户名', master_password='密码', master_log_file='file名字', master_log_pos=position数字`

启动主从关系

`start slave`

展示主从关系

`show slave status`

slave_io_running:yes + slave_sql_running:yes则主从关系复制成功

停止主从关系

`stop slave`
