# 术语

| MangoDB术语/概念 | 解释/说明                            |
| ---------------- | ------------------------------------ |
| database         | 数据库                               |
| collection       | 集合                                 |
| document         | 文档                                 |
| field            | 域                                   |
| index            | 索引                                 |
| primary key      | 主键，MongoDB自动将_id字段设置为主键 |

# 数据库操作

* 查看当前数据库名称

  ```sql
  db
  ```

* 切换数据库，**如果数据库不存在，则指向数据库，但不创建**，知道插入数据或者创建集合时数据库才被创建

  ```sql
  user 数据库名称
  ```

* 删除当前指向的数据库，如果不存在，则不操作

  ```sql
  db.dropDatabase()
  ```

# 集合操作

* 集合创建

  ```sql
  db.createCollection(name, options)
  ```

  * name是创建的集合的名称
  * options是一个文档，用于指定集合的配置
  * **选项参数是可选的，所以只需要到指定的集合名称**

* 展示所有集合

  ```sql
  show collections;
  ```

# 数据类型

* Object ID：文档ID

  每个文档都有一个属性，为_id，保证每个文档的唯一性

  ObjectID是一个12字节的十六进制数

  * 前4个字节为当前时间戳
  * 接下来三个字节为机器ID
  * 接下来两个字节是MongoDB的服务进程ID
  * 最后三个字节是简单的增量值

* String：字符串，最常用，必须是有效的UTF-8

* Boolean：存储一个布尔类型，true或false

* Integer：整数可以是32位或64位，这取决于服务器

* Double：存储浮点值

* Arrays：数组或列表，多个值存储到一个键

* Object：用于嵌入式文档，即一个值一个文档

* Null：存储Null值

* Timestamp：时间戳

* Date：存储当前日期或者时间UNIX时间格式

# 数据操作

```java
db.集合名称.insert(document)
```

插入文档时，如果不指定_id参数，MongoDB会为文档分配一个唯一的ObjectID

```sql
db.集合名称.find()
```

展示集合下的所有文档

```sql
db.集合名称.update(<query>, <update>, {multi: <boolean>})
```

更新集合下的文档

* query：查询条件，类似于where
* update：更新操作符，类似于set
* multi：可选，默认false，表示只更新找到数据的第一条记录，如果为true那么就是全部更新

**正常情况是全文档更新**

```sql
db.study.update({'name':'ysy'}, {'name':'xyl'});
```

![image-20221026214122196](D:\WorkSpace\Note\Picture\image-20221026214122196.png)

```sql
db.study.update({'name':'ysy'}, {$set:{'name':'xyl'}});
```

![image-20221026214325773](D:\WorkSpace\Note\Picture\image-20221026214325773.png)

```sql
db.集合名称.save(document);
```

如果\_id已经存在则修改，如果\_id不存在则添加

```sql
db.集合名称.remove(<query>, {justOne:<boolean>})
```

* query：可选，删除文档的条件
* justOne：可选，如果true，只删除一条，false多删

**固定集合**

```
db.createCollection("sub", {'capped': true, size:10});
```

表示固定长度为10的集合，超过十个，多出的将会环形覆盖

# 查询操作

```sql
db.sub.find();
```

查询所有

```sql
db.sub.findOne();
```

查询单个

> * 比较运算符
>   * 等于，默认
>   * 小于，$lt
>   * 小于等于，$lte
>   * 大于，$gt
>   * 大于等于，$gte
>   * 不等于，$ne
> * 逻辑运算符
>   * 逻辑与，默认，
>   * 逻辑或，$or
> * 范围运算符
>   * $in
>   * $nin
> * 正则表达式
>   * 使用//或$regex编写正则表达式

```sql
db.sub.find({$where:function() {return this.age<20}});
```

使用$where后面加一个函数，返回满足条件的数据

* limit用于获取几条

* skip用于跳过几条

* 在查询到返回结果中，只选择必要的字段展示

  ```sql
  db.集合名称.find({}, {字段名称:1, ...})
  ```

* 排序

  ```sql
  db.集合名称.find().sort({字段升序:1, 字段降序:-1})
  ```

* 统计个数

  ```sql
  db.集合名称.count({条件})
  ```

# 聚合操作

```sql
db.集合名称.aggregate([{管道:{表达式}}])
```

将前一次的操作结果放到下一次的操作中

* $group：将集合中的文档分组，可用于统计结果
* $match：过滤数据，只输出符合条件的文档
* $project：修改输入文档的结构（重命名、增加、删除字段、创建计算结果）,显示的数据为1，不显示的为0
* $sort：将输入文档的内容排序后返回
* $limit：限制聚合管道返回文档数
* $skip：跳过指定数量的文档，并返回余下的文档
* $unwind：将数组类型的字段进行拆分

表达式

* $sum：计算总和
* $avg：计算平均值
* $min：获取最小值
* $max：获取最大值
* $push：在结果文档中插入值到一个数组中
* $first：获取第一个文档资源
* $last：获取最后一个文档资源





