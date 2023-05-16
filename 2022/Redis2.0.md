# 数据安全与性能保障

## 持久化选项

Redis提供了两种不同的持久化方法来讲数据存储到硬盘中

* 快照——**可将讲存在于某一时刻的所有数据都写入硬盘中**
* 只追加文件——**执行写命令时，将被执行的写命令赋值到硬盘里面**

```shell
# 快照持久化选项
save 60 1000
stop-writes-on-bgsave-error no
rdbcompression yes
dbfilename dump.rdb

# AOF持久化选项
appendonly on
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-precentage 100
auto-aof-rewrite-min-size 64mb

# 共享选项，这个选项决定了快照文件和AOF文件的保存位置
dir ./
```

### 快照持久化

Redis可以通过创建快照来获得**存储在内存中的数据在某个时间点的副本**

根据配置，快照将被写入dbfilename选项指定的文件里面，并储存在dir选项指定的路径上面

Redis、系统或者硬件三者之一的任意一个崩溃后，那么Redsi**将丢弃最近一次创建快照之后写入的所有数据**

创建快照的方法——

* BGSAVE，并发进行

* SAVE，阻塞进行

* 设置save配置选项

  `save 60 10000`如果60s之内有10000次写入，那么就会触发BGSAVE

  **可以设置多个save配置选项**

* SHUTDOWN关闭服务器时，将调用SAVE阻塞的进行快照保存

* 一个redis服务器连接另一个redis服务器时，并向对方发送SYNC命令来开始一次赋值操作时

### AOF持久化

AOF持久化会将被执行的写命令**写到AOF文件的末尾**，以此来记录数据发生的变化

**appendfsync选项及同步频率**

* always——**每次redis写命令都要同步写入磁盘**，这样严重降低redis速度
* everysec——**每秒执行一次同步，显式将多个写命令同步到磁盘中**
* no——**让操作系统决定应该何时进行同步**

**当磁盘忙于执行写操作的时候，Redis还会优雅的放慢自己的速度以便适应磁盘的最大写入速度**

AOF持久化缺陷——**AOF文件的体积大小**

### 重写/压缩AOF文件

体积不断增大的AOF文件可能会占据所有的磁盘空间

* 可以通过BGREWRITEAOF来移除AOF文件中的冗余命令来重写AOF文件
* 设置auto-aof-rewrite-percentage和auto-aof-rewrite-min-size来自动进行BGREWRITEAOF，**当aof文件大小超过min-size并且是之前aof文件的percentage倍**时开始rewrite

## 复制

### 对Redis复制相关选项进行配置

**指定slaveof host port选项**来连接主服务器

### 主从链

因为Redis的主服务器和从服务器么有特别不同的地方，所以**从服务器也可以拥有自己的从服务器**，形成主从链

## Redis事务

使用MULTL命令开启事务

使用EXEC命令提交事务

使用WATCH对键进行监视，**如果有其他客户端抢先对任何被监视的字段进行了替换、更新或删除等操作，那么执行EXEC时，就会失败并返回错误**

使用UNWATCH取消监视

使用DISCARD取消事务

