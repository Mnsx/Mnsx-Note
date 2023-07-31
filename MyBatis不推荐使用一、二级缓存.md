# 为什么MyBatis一级和二级缓存都不建议使用

![image-20221106210238596](D:\WorkSpace\Note\Picture\image-20221106210238596.png)

**Executor的设计是一个典型的装饰着模式，SimpleExecutor，ReuseExecutor是具体实现类，而CachingExecutor是装饰器类**

BaseExecutor这个父类是一个模板模式的典型应用，操作一级缓存的操作都在这个类中，而具体的操作数据库的功能则让子类去实现

**二级缓存则是一个装饰器类，当开启二级缓存的时候，会使用CachingExecutor对具体实现类进行装饰，所以查询的时候一定是先查询二级缓存再查询一级缓存**

## 一级缓存

![image-20221106210628692](D:\WorkSpace\Note\Picture\image-20221106210628692.png)

```java
// BaseExecutor
protected PerpetualCache localCache;
```

一级缓存是BaseExecutor中的一个成员变量localCache（对HashMap的一个简单封装），因此一级缓存的声明周期与SqlSession相同，可以将SqlSession类比为JDBC编程中Connection，即数据库的一次会话

**一级缓存和二级缓存key的构建规则是一致的，都是一个CacheKey对象，因为MyBatis中涉及动态SQL等多方面的因素，缓存的Key不能仅仅通过String来表示**

当执行sql的4个条件都相同时，CacheKey才会相同

1. mappedStatement的id
2. 指定查询结构集的范围
3. 查询所使用的SQL语句
4. 用户穿的给SQL语句的实际参数

```java
@Override
public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException {
    BoundSql boundSql = ms.getBoundSql(parameter);
    CacheKey key = createCacheKey(ms, parameter, rowBounds, boundSql);
    return query(ms, parameter, rowBounds, resultHandler, key, boundSql);
}
```

```java
list = resultHandler == null ? (List<E>) localCache.getObject(key) : null;
if (list != null) {
    handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);
} else {
    list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
}
```

当使用一个SqlSession执行更新操作时，会先清空一级缓存，因此一级缓存中内容被使用的概率也很低

```java
@Override
public int update(MappedStatement ms, Object parameter) throws SQLException {
    ErrorContext.instance().resource(ms.getResource()).activity("executing an update").object(ms.getId());
    if (closed) {
        throw new ExecutorException("Executor was closed.");
    }
    clearLocalCache();
    return doUpdate(ms, parameter);
}
```

**MyBatis的一级缓存最大范围时SqlSession内部，有多个SqlSession或者分布式的环境下，数据库写操作会引起脏数据，建议设定缓存级别为Statement**

```xml
<setting name="localCacheScope" value="STATEMENT"/>
```

BaseExecutor的query方法，当配置成STATEMENT时，每次查询完都会清空缓存

```java
if (configuration.getLocalCacheScope() == LocalCacheScope.STATEMENT) {
    // issue #482
    clearLocalCache();
}
```

当mybatis和spring整合侯

1. 在未开启事务的情况下，每次查询，spring都会关闭旧的sqlSession而创建新的SqlSession，因此此时的一级缓存是没有作用的
2. 在开启事务的情况之下，spring使用threaddLocal获取当前线程绑定的同一个SqlSession，因此此时一级缓存是有效的，当事务执行完毕，会完毕sqlSession

**当mybatis和spring整合后，未开启事务的情况下，不会有任何问题，因为一级缓存没有生效。当开启事务的情况下，可能会有问题，由于一级缓存的存在，在事务内的查询隔离级别是可重复读，即使你数据库的隔离级别设置的是提交读**

## 二级缓存

![image-20221106213917231](D:\WorkSpace\Note\Picture\image-20221106213917231.png)

```java
// Configuration
protected final Map<String, Cache> caches = new StrictMap<>("Caches collection");
```

**而二级缓存是Configuration对象的成员变量，因此二级缓存的生命周期是整个应用级别的。并且是基于namespace构建的，一个namesapce构建一个缓存**

**二级缓存不像一级缓存那样查询完直接放入一级缓存，而是要等待事务提交时才会将查询出来的数据放到二级缓存中**

```xml
<settings>
	<setting name="cacheEnabled" value="true"/>
</settings>
```

这是二级缓存的总开关，只有当该配置项设置为true时，后面两项的配置才会有效果

```java
public Executor newExecutor(Transaction transaction, ExecutorType executorType) {
    executorType = executorType == null ? defaultExecutorType : executorType;
    executorType = executorType == null ? ExecutorType.SIMPLE : executorType;
    Executor executor;
    if (ExecutorType.BATCH == executorType) {
        executor = new BatchExecutor(this, transaction);
    } else if (ExecutorType.REUSE == executorType) {
        executor = new ReuseExecutor(this, transaction);
    } else {
        executor = new SimpleExecutor(this, transaction);
    }
    if (cacheEnabled) {
        executor = new CachingExecutor(executor);
    }
    executor = (Executor) interceptorChain.pluginAll(executor);
    return executor;
}
```

当设置了cacheEnabled参数侯，如果为true，将使用修饰器类CaChingExecutor

**通常为每个表建立单独的映射文件，由于MyBatis的二级缓存是基于namespace的，多表查询语句所在的namespace无法感应到其他namespace中语句对多表查询中涉及的表的修改，所以引发脏读问题**

