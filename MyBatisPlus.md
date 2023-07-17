# MyBatis-Plus

## 配置日志

我们所有的sql是不可见的，所以需要配置日志才能直到它是如何执行的

```properties
#配置日志
mybatis-plus.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
```

## 主键生成策略

默认 ID_WORKER 全局的唯一ID

**雪花算法**：

雪花算法（Snowflake）是一种生成分布式全局唯一ID的算法，生成的ID称为Snowflake IDs或snowflakes。

一个Snowflake ID有64位元。前41位是时间戳，表示了自选定的时期以来的毫秒数。 接下来的10位代表计算机ID，防止冲突。 其余12位代表每台机器上生成ID的序列号，这允许在同一毫秒内创建多个Snowflake
ID。SnowflakeID基于时间生成，故可以按时间排序。**此外，一个ID的生成时间可以由其自身推断出来，反之亦然。该特性可以用于按时间筛选ID，以及与之联系的****对象****。**

**主键自增**

我们需要配置主键自增——

1. 实体类字段上 @TableId(type = IdType.AUTO)
2. 数据库字段一定要是自增的

其他的源码解释

```java
public enum IdType{
    AUTO(0), // 数据库id自增
    NONE(1), // 未设置主键
    INPUT(2), // 手动输入 
    ID_WORKER(3), // 默认全局ID
    UUID(4), // 全局ID
    ID_WORKER_STR(5);  // ID_WORKER的字符串形式
}
```

## 自动填充

创建时间、修改时间——这些操作都是自动化完成的额，我们不希望手动更新

阿里巴巴开发手册中——所有的数据库表：gmt_create、gmt_modified 这两个字段几乎所有的表都需要配置上！而且需要自动化

方式一：数据库级别

**失败**

方式二：代码级别

1. 删除数据库默认值、更新操作
2. 实体类字段属性上需要添加注解

```java
    @TableField(fill = FieldFill.INSERT)
    private Date createTime;
    @TableField(fill = FieldFill.UPDATE)
    private Date updateTime;
```

1. 编写处理器来处理这个注解即可

```java
@Slf4j
@Component // 一定需要将其加入IOC容器中
public class MyMetaObjectHandler implements MetaObjectHandler {
    // 插入时的填充策略
    @Override
    public void insertFill(MetaObject metaObject) {
        log.info("start insert fill...");

        this.setFieldValyName("createTime", new Date(), metaObject);
        this.setFieldValByName("updateTime", new Date(), metaObject);
    }

    // 更新时的填充策略
    @Override
    public void updateFill(MetaObject metaObject) {
        log.info("start update fill...");

        this.setFieldValByName("updateTime", new Date(), metaObject);
    }
}
```

## 乐观锁

乐观锁——总是认为不会出现问题，无论干什么都不会上锁，如果出现了问题，再次更新值测试

悲观锁——总是认为会出现问题，无论干什么都会上锁，再去操作

乐观锁实现方式——

- 取出记录时，获取当前version
- 更新时，带上这个version

- 执行更新时，set version = newVersion where version = oldVersion
- 如果version不对，就更新失败

测试MP乐观锁插件

1. 给数据库增加version字段
2. 实体类添加对应字段

```java
    @Version
    private Integer version;
```

1. 注册组件

## 批量条件查询

使用selectBatchIds(List list)、selectByMap(Map map)来进行批量条件查询

## 分页查询

1. 原始的limit进行分页
2. pageHelper 第三方插件

1. MP内置了分页插件

```java
@EnableTransactionManagement
@Configuration
public class MyBatisPlusConfiguration {

    // 注册乐观锁插件
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor mybatisPlusInterceptor = new MybatisPlusInterceptor();
        mybatisPlusInterceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
        mybatisPlusInterceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        return mybatisPlusInterceptor;
    }
}
```

## 逻辑删除

物理删除：从数据库中直接移除

逻辑删除：在数据库中没有被移除，而是通过变量让他失效——delete=0->1

防止数据丢失，类似于回收站的作用

只需要在实体类属性上加上@TableLogic，并且在application.yml中添加配置

```java
    @TableLogic
    private Integer deleted;
mybatis-plus:
  global-config:
    db-config:
      logic-not-delete-value: 0
      logic-delete-value: 1
```

## 条件查询器

查询常用——QueryWrapper

常见方法——

- isNotNull("columnName")——判断columnName不为空
- ge("columnName", val)——判断columnName的值大于var

- eq("columnName", val)——判断columnName的值与var相同
- between("columnName", val1, val2)——判断columnName的值在var1和var2之间的

- notLike("columnName", val)——判断columnName的值不包括val
- likeLeft("columnName", val)——模糊查询（%val）

- insql("columnName", sql)——子查询，通过sql得到值赋值到insql中
- orderByAsc('columnName")——按照columnName升序排序

- orderByDesc("columnName")——按照columnName降序排序

## 代码生成器

没学