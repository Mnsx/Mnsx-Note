# 2.0 索引

## 存储引擎

存储引擎——数据在磁盘上的不同组织形式

* InnoDB
* MyISAM

> InnoDB与MyISAM区别
>
> 1. InnoDB支持事务，MyISAM不支持事务
> 2. InnoDB支持外键，MyISAM不支持外键
> 3. InnoDB支持表锁和行锁，MyISAM只支持表锁
> 4. 索引存储方式不同

**InnoDB和MyISAM索引都是使用的B+Tree**，将数据持久化在硬盘中

memory存储引擎使用的hash数据结构来作为索引的数据结构，将数据放在内存中

## 数据结构

Hash表作为数据结构的缺点——

1. 使用hash表必须保证具备好的hash算法，如果算法不合适，会造成hash冲突或hash碰撞，会导致数据结构散列不均匀，有可能会退化成一个链表
2. 使用hash表的时候不支持范围查询，当需要范围匹配的时候，必须要挨个对比，效率太低
3. 需要大量的内存空间

二叉树、AVL、红黑树作为数据结构的缺点——

1. 分支有且只有两个，大数据情况下，会造成树的层数较高，导致IO消耗过大

   > * 局部性原理
   >
   >   时间局部性、空间局部性
   >
   >   数据和程序都有聚集成群的倾向
   >
   >   之前被读取过的数据再次读取速率比之前的数据高
   >
   > * 磁盘预读
   >
   >   内存跟磁盘在进行数据交互的时候由一个最基本的逻辑单元，称之为页，或者叫做datapage，不同的操作系统，页的大小是不同的，一般是4k，或者8k
   >
   >   每次进行读取数据的时候需要读取4k的整数倍
   >
   >   InnoDB存储引擎默认读取的是16kb的数据
   >
   > **尽可能减少要读取的数据量，还需要减少数据访问的次数**

BTree作为数据结构的缺点——

![image-20221103120227379](D:\WorkSpace\Note\Picture\image-20221103120227379.png)

1. 每个page只有16k，相当于可以存储16条1k的数据，三层B树，只能存储16\*16\*16条数据，如果想要存放更多的数据，只能通过扩大页内存大小或者将存放data的部分从节点中移除

B+Tree作为数据结构——

![image-20221103120829329](D:\WorkSpace\Note\Picture\image-20221103120829329.png)

**千万级别数据量只需要使用3-4层B+Tree即可**，如果指针加键值的大小为10字节，那么一层可以存放1600个数据，3层也就是1600\*1600\*16——》4千万数据

**插入数据时要避免页分裂，所以尽量使用自增id**

> * 页分裂
>
>   向图中13、15之间插入一个14，但是页中已经没有空间存放14了，那么就会将这个页，分裂成两个页，同时改变非叶子节点的数据
>
> * 页合并
>
>   删除数据后，页的内存不会改变，因为删除只是添加一个失效标记，让有其他数据插入时，会顶替他的位置，但是当失效标记达到系统设置的阈值时（默认页内存的50%）就会选择最近的页进行页合并

**InnoDB存储引擎，在插入数据的时候，必须要将数据跟某一个索引绑定在一起，索引列可以是主键，也可以是唯一键，也可以是6字节的rowid**

一个表中可以包含多个索引列，那么数据文件只会保存一份，不会造成多分数据的数据冗余，会将跟数据绑定在一起的索引列的值放到其他索引的叶子节点

## 聚簇索引和非聚簇索引

数据跟索引放在一起的叫做聚簇索引，数据跟索引没有放在一起的叫做非聚簇索引

## MySQL索引原则

1. 回表

   ```sql
   select * from table where name = "mnsx"
   ```

   从某一个索引的叶子节点中获取聚簇索引的id值，根据id再去聚簇索引中获取全量记录，**尽量减少回表**

2. 索引覆盖

   ```sql
   select id, name from table where name = "mnsx"
   ```

   从索引的叶子节点中获取到全量查询列的过程叫做索引覆盖

3. 最左匹配

   ```sql
   select * from table where name = "mnsx" and age =20; // true
   select * from table where name = "mnsx" and age =20; // true
   select * from table where age =20; // false
   select * from table where name = "mnsx" and age =20; // true
   ```

   MySQL优化器会进行优化，选择合适的顺序执行SQL语句

4. 索引下推

   ```sql
   select * from table where name = "mnsx" and age = 10;
   ```

   在没有索引下推之前，sql语句执行过程，现根据name去存储引擎拿到全量的数据，然后将数据读取到server层，然后按照age根据数据过滤

   **有索引下推之后，根据name、age两个列区存储引擎筛选数据，将最终的结果返回给客户端**

   ![image-20221104221304724](D:\WorkSpace\Note\Picture\image-20221104221304724.png)

## 索引分类

* 主键索引

  主键是一种唯一性索引，但他必须指定PRIMARRY KEY，每个表只能有一个主键

* 唯一索引

  索引列的所有值都只能出现一次，即必须唯一，值可以为空

* 普通索引

  基本的索引类型，值可以为空，没有唯一性的限制（覆盖索引）

* 全文索引

  全文索引的索引类型为FULLTEXT，全文索引可以在varchar、char、text类型的列上创建

* 组合索引

  多列值组成一个索引，专门用于组合搜索（最左匹配原则）

## 索引维护

* 在索引插入新的值的时候，为了维护索引的有序性，必须要维护
  * 需要插入一个比较大的值，直接插入即可，几乎没有成本
  * 如果插入的是中间的某一个值，需要逻辑上移动后续的元素，空出位置
  * 如果需要插入的数据页满了，许哟啊单独申请一个新的数据页，然后移动部分数据过去，叫做页分裂，此时性能会收到同时空间的使用率也会减低，除了页分裂之外还包含页合并
* 尽量使用自增主键作为索引

# MySQL架构

![image-20221105121747492](D:\WorkSpace\Note\Picture\image-20221105121747492.png)

## 连接器

* 连接器负责跟客户端建立连接，获取权限，维持和管理连接
  * 用户名密码验证
  * 查询权限信息，分配对应的权限
  * 可以使用show processlist查看现在的连接
  * 如果太长时间没有动静，就会自动断开，通过wait_timeout控制，默认8小时
* 连接可以分为两类
  * 长连接：推荐使用，但是要周期性的断开长连接
  * 短链接

## 分析器

* 词法分析：MySQL需要把输入的字符串进行识别每部分代表的意思
* 语法分析：根据语法规则判断这个sql语句是否满足mysql的语法，如果不满足就会报错“you hava an error in your sql synta”

## 优化器

* 在具体执行语句之前，要先经过优化器的处理
  * 当表中有多个索引的时候，决定使用哪个索引
  * 当sql语句需要多表关联的时候，决定表的连接顺序
  * ...
* 不同的执行方式对SQL语句的执行效率影响很大
  * RBO：基于规则的优化
  * CBO：基于成本的优化

# 日志实现

## Redo日志

InnoDB存储引擎的日志文件

* 当发生数据修改的时候，InnoDB引擎会先将记录写道redo log中，并更新内存，此时更新就算是完成了，同时innodb引擎会在何时的时机将记录操作到磁盘中
* Redolog是固定大小的，是循环写的过程
* 有了redolog之后，innodb就可以保证即使数据库发生异常重启，之前的记录也该不会丢失，叫做crash-safe

write pos

![image-20221105123353647](D:\WorkSpace\Note\Picture\image-20221105123353647.png)

## Undo日志

* Undo log是为了实现事务的原子性，在MySQL数据库InnoDB存储引擎中还用UndoLog来实现多版本并发控制（MVCC）

* 在操作任何数据之前，首先将数据备份到一个地方（这个存储数据备份的地方称为UndoLog），然后进行数据的修改，如果出现了错误或者用户执行了ROLLBACK语句，系统可以利用Undo Log中备份嫁给你数据恢复到事务开始之前的状态

* 注意UndoLog是逻辑日志，可以理解为：

  -当delete一条记录时，undo log中会记录一条对应的insert记录

  -当insert一条记录时，undo log中会记录一条对应的delete记录

  -当update一条记录时，它记录一条u敌营的update记录

## Bin日志

* bin log中会记录所有的逻辑，并且采用追加写的方式
* 一般在企业中数据库会有备份系统，可以定期执行备份，备份的周期可以自己设置
* 恢复数据的过程
  * 找到最近一次的全量备份数据
  * 从备份的时间点开始，将备份的binlog取出来，重放到要恢复的哪个时刻

> 将redolog的写入时拆分成两个阶段，prepare和submi阶段，为了保证redolog和binlog的内容同时写入到磁盘中

# MySQL执行计划

## id

select查询的序列号，包含一组数字，表示查询中执行selecct子句或者操作表的顺序

* 如果id相同，那么执行顺序从上到下
* 如果id不同，如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行
* id相同和不相同的，同时存在，相同的可以认为是一组，从上往下顺序执行，在所有组中，id值越大，优先级越高，越先执行

## select_type

主要用来分辨查询类型，是普通查询还是联合查询还是子查询

* SIMPLE——简单的select查询，查询中不包含子查询或者union
* PRIMARY——查询中若包含任何复杂的子部分，最外层查询则被标记为
* SUBQUERY——在SELECT或WHERE列表中包含了子查询
* DERIVED——在FROM列表中包含的子查询被标记为DERIVED，MySQL会递归执行这些子查询，把结果放在临时表中
* UNION——若第二个SELECT出现UNION之后，则被标记为UNION，若UNION包含在FROM子句的子查询中，外层SELECT将被标记为DERIVED
* DEPEND_UNION——如果UNION的查询语句，被外面所依赖，那么该查询语句就是DEPEND_UNION
* UNION_RESULT——从union表获取的结果的select
* DEPENDENT_SUBQUERY——subquery的子查询收到外部表查询的影响

 ## table

对应行正在访问的表，表明或者别名，可能是临时表或者union合并结果集

* 如果是具体的表名，则表明从实际的物理表中获取数据
* 表明是derivedN的形式，表示使用了id为N的查询产生的衍生表
* 当由union result时，表名是union n1,n2等的形式，n1、n2表示参数union的id

## type

type显示的访问类型，访问类型表示是以何种方式去访问数据

system > const > eq_ref > ref > range > index > all

**一般需要至少达到range级别，最好达到ref**

* system——表只有一行记录（等于系统表），这是const类型地特性，平时不会出现，这个可以忽略不记
* const——表示通过索引一次就找到了，const用于比较primary key或者unique索引，因为只匹配一行数据，所以很快，如果将主键置于WHERE列表中，MySQL就能将该查询转换为一个常量
* eq_ref——唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配，常见于主键或唯一索引扫描

* ref——非唯一性索引扫描，返回匹配某个单独值得所有行，本质上也是一种索引访问，他返回所有匹配某个单独值得行，然而，它可能会找到多个复合条件得行，所以他应该属于查找和扫描得混合体
* index_subquery：利用索引来关联子查询，不再扫描全表
* unique_subquery：该连接类型类似于index_subquery，使用的是唯一索引
* range——只检索给定范围得行，使用一个索引来选择行。key列显示使用了哪个索引，一般就是在你得where语句中出现了between、<、>、in等得查询，这种范围扫描索引比全表扫描要好，因为它只需要开始于索引得某一点，而结束语另一点，不用扫描全部索引
* index——Full Index Scan，index于All区别为index类型只遍历索引树，这通常比All快，因为索引文件通常比数据文件小
* all——Full Table Scan，将遍历全表以找到匹配的行

## possible_keys

显示可能应用在这张表中的索引，一个或多个，查询涉及到的字段上如哦存在索引，则该索引将被列出，但不一定被查询实际使用

## key

实际使用的索引，如果为null，则没有使用索引，查询中若使用了覆盖索引，则该索引和查询的select字段重叠

## key_len

表示索引中使用的字节数，可以通过key_len计算查询中使用的索引长度，再不损失精度的情况下长度越短越好

## ref

显示索引的哪一行被使用了，如果可能的话，是一个常数

## rows

根据表的统计信息及索引使用情况，大致估算出找到所需记录需要读取的行数，此参数很重要，直接反映的sql找了多少条数据，在完成目的的情况下越少越好

## extra

* using filesort：说明mysql无法利用索引进行排序，只能利用排序算法进行排序，会消耗额外的位置
* using temporary：建立临时表来保存中间结果，查询完成之后把临时表删除
* using index：这个表示当前的查询用覆盖索引的，直接从索引中读取数据，而不用访问数据表，如果同时出现using where表名索引被用来执行索引键值的查找，如果没有，表明索引被用来读取数据，而不是真的查找
* using where：使用where进行条件过滤
* using join buffer：使用连接缓存
* impossible where：where语句的结果总是false

# MySQL的锁机制

## MySQL锁的基本介绍

**锁是计算机调节多个进程或线程并发访问某一资源的机制**

不同的存储引擎支持不同的锁机制

* MyISAM和MEMORY存储引擎采用表级锁
* InnoDB存储引擎及支持行级锁也支持表级锁

**表级锁：开销小，枷锁快，不会出现死锁，锁定粒度大，发生锁冲突的概率最高，并发度最低**

**行级锁：开销大，加锁慢，会出现死锁，锁定粒度肖，发生锁冲突的概率最低，并发度也最高**

> OLTP——在线事务处理
>
> OLAP——在线分析处理

## MyISAM表锁

MySQL的表级锁有两种模式：**表共享读锁（TableReadLock）**和**表独占写锁（TableWriteLock）**

```sql
create table mylock (
    id int(11) not null auto_increment,
    name varchar(20) DEFAULT NULL,
    PRIMARY KEY (id)
) ENGINE=MyISAM DEFAULT CHARSET=utf8;

insert into mylock values('1', 'a');
insert into mylock values('2', 'b');
insert into mylock values('3', 'c');
insert into mylock values('4', 'd');
```

![image-20221106160912837](D:\WorkSpace\Note\Picture\image-20221106160912837.png)

![image-20221106161209236](D:\WorkSpace\Note\Picture\image-20221106161209236.png)

## InnoDB锁

### 事务及其ACID属性

事务是由一组SQL语句组成的逻辑处理单元，事务具有4属性，通常称为事务的ACID属性

原子性：事务是由一个原子操作单元，其对数据的i需改，要么全都执行，要么都不执行

一致性：在事务开始和完成时，数据都必须保持一致状态

隔离性：数据库系统提供 一定的隔离机制，保证事务不受外部并发操作的影响的独立环境执行

持久性：事务完成之后，他对数据的修改是永久性的，即使出现系统故障也能够保障

### InnoDB的行锁模式及加锁方法

**共享锁**

**排他锁**

InnoDB行锁是通过给索引上的索引项加锁来实现的，**只有通过索引条件检索数据，InnoDB才使用行级锁，否则使用表锁**

```sql
create table tab_with_index(id int, name varchar(10))engine=innodb;
alter table tab_with_index add index id(id);
insert into tab_with_index values(1, '1'),(2,'2'),(3,'3'),(4,'4');
```

![image-20221106164144313](D:\WorkSpace\Note\Picture\image-20221106164144313.png)

![image-20221106164231079](D:\WorkSpace\Note\Picture\image-20221106164231079.png)

## 总结

![image-20221106164328526](D:\WorkSpace\Note\Picture\image-20221106164328526.png)

![image-20221106164346801](D:\WorkSpace\Note\Picture\image-20221106164346801.png)

# 1.0 MySQL架构

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
