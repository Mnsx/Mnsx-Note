# 2.0 MyBatis动态SQL

## trim用法

trim标签属性——

* prefix：当trim元素内包含内容时，会给内容增加prefix指定的前缀
* prefixOverrides：当trim元素内包含内容时，会把内容中匹配的前缀字符串去掉
* suffix：当trim元素内包含内容时，会给内容增加suffix指定的后缀
* suffixOverrides：当trim元素包含内容时，会把内容中匹配到的后缀字符串去掉

## foreach用法

foreach标签属性——

* collection：必填，值为要循环迭代循环的属性
* item：变量名，值为从迭代对象中取出的每一个值
* index：索引的属性名，在集合数组的情况下值为当前索引值，当迭代循环的对象是Map类型时，这个值为Map的key
* open：整个循环内容的开头的字符串
* close：整个循环内容的结束的字符串
* separator：每次循环的分隔符

## OGNL用法

* a or b
* a and b
* a == b
* a != b
* a lt/gt b 小于/大于
* a lte/gte b 小于等于/大于等于
* +、-、*、/、%
* !a
* a.method(args) 调用对象方法
* a.property对象属性
* a[index] 按索引取值
* @class@method(args) 调用类静态方法
* @class@field 调用类静态属性

# MyBatis代码生成器

```xml
<generatorConfiguration>
    
    <!--用户指定一个需要在配置中解析使用的外部属性文件，引入属性文件后，可以在配置中使用${property}这种形式的引用，
可以通过这种方式引用属性文件中的属性值-->
    <properties resource="top.mnsx.study.jdbc.properties" url="file:///D:/workspace/jdbc.properties">
		<!--包含resource和url两个属性，只能使用其中一个属性来指定-->
    	<!--resource：指定classpath下的属性文件-->
    	<!--url：指定文件系统上的特定位置-->
    </properties>
    
    <!--通过location指定驱动的路径，可以配置多个，也可以不配置-->
    <classPathEntry location="D:\mysql\mysql-connector-java-5.1.29.jar">
    	<!--location：指定驱动路径-->
    </classPathEntry>
    
    <!--至少配置1个，或者配置多个，context标签用于指定生成一组对象的环境-->
    <context id="context1" defaultModeType="flat" targetRuntim="MyBatis3Simple">
    	<!--只有一个必选属性id，用来唯一确定这个标签-->
        <!--defaultModelType：定义了MBG如何生成实体类-->
        	<!--conditional：默认值，如果一个表的主键只有一个字段，那么不会为该字段生成单独的实体类，而是会将该字段合并到基本实体类中-->
        	<!--flat：该模型只为每张表生成一个实体类，这个实体类包含表中的所有字段，推荐使用-->
        	<!--hierarchical：如果表有主键，那么该模型会产生一个单独的主键实体类，如果还有BLOB字段，则会为表生成一个包含所有BLOB字段的单独的实体类，然后所有其他字段另外生成一个单独的实体类，MBG会在所有生成实体类间维护一个继承关系-->
        <!--targetRuntime此属性用于指定生成的代码的运行时环境-->
        	<!--MyBatis3：默认值-->
        	<!--MyBatis3Simple：不会生成与Example相关的方法-->
        <!--introspectedColumnImpl：这个参数可以指定扩展org.mybatis.generator.api.Introspected Cloumn类的实现类-->
        
        <property name="autoDelimitKeywords" value="true">
        	<!--autoDelimitKeywords：自动给关键字添加分隔符的属性，当数据库字段或表与关键字列表中的关键字一样时，
MBG会自动给这些关键字或表添加分隔符-->
            <!--beginningDelimiter：前置分隔符-->
            <!--endingDelimiter：后置分隔符-->
            <!--javaFileEncoding：设置要使用的Java文件的编码，默认使用当前运行环境的编码-->
            <!--javaFormatter：略-->
            <!--xmlFormatter：略-->
        </property>
        <property name="beginningDelimiter" value="`"/>
        <property name="endingDelimiter" value="`"/>
        
        <!--plugin标签可以配置0个或者多个，个数不限制，plugin标签用来定义一个插件，用于扩展或者修改通过MBG生成的代码-->
        <plugin>
        </plugin>
        
        <!--该标签用来配置如何生成注释信息，最多可以配置一个-->
        <commentGenerator>
            <!--type：用来指定用户的实现类，该类需要实现org.mybatis.generator.api.CommentGenerator接口，而且必须提供一个默认空的构造方法，
type属性接收默认的特殊值DEFAULT，使用默认的实现类org.mybatis.generator.internal.DefaultCommnetGenerator-->
            <!--suppressAllComments：阻止生成注释，默认为false-->
            <!--suppressDate：阻止生成注解包含时间戳，默认为false-->
            <!--addRemarkComments：注释是否添加数据库表的备注信息，默认为false-->
            <property name="suppressDate" value="true"/>
        	<property anme="addRemarkComments" value="true"/>
        </commentGenerator>
        
        <!--jdbcConnection用来指定MBG要连接的数据库信息，该标签必选，而且只能有一个, 
如果jdbc驱动不再classpath下，那就要通过classPathEntry标签引入jar包-->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver" connectionURL="jdbc:mysql://localhost:3306/mybatis"
                        userId="root" passwrod="">
        	<!--driverClass：访问数据库的JDBC驱动程序的完全限定类名-->
            <!--connectionURL：访问数据库的JDBC连接URL-->
            <!--userId：访问数据库的用户id-->
            <!--password：访问数据库的密码-->
        </jdbcConnection>
        
        <!--该标签的配置用来指定JDBC类型和Java类型如何转换，最多配置一个-->
        <javaTypeResolver>
        	<!--type：类似commentGenerator，默认的实现DEFAULT，一般情况直接使用默认情况，不建议修改这个属性-->
        </javaTypeResolver>
        
        <!--用来控制生成的实体类，根据context标签中配置的defaultModelType属性值不同，一个表可能会对应生成多个不同的实体类
该标签必须配置一个，并且最多配置一个-->
		<javaModelGenerator targetPackage="top.mnsx" targetProject="src\main\java">
            <!--targetPackage：必选，生成实体类存放的包名，一般存放在改包下，实际还是会受到其他配置的影响-->
            <!--targetProject：必选，指定目标项目的路径，可以使用相对路径或绝对路径-->
           	<!--constructorBased：该属性支队MyBatis3有效，如果为true那么使用构造函数入参，如果为false那么使用setter，默认false-->
            <!--enableSubPackages：如果为true，MBG会根据catelog和schema来生成子包，如果为false就直接使用targetPackage属性，默认false-->
            <!--immutable：用来配置实体类属性是否可变，如果设置为true，那么constructorBased不管设置成什么，都是用构造方法入参，并且不会生成setter方法，如果false实体类型就是可变，默认false-->
            <!--rootClass：设置所有实体类的基类，如果设置，则需要使用类的全限定名称，并且如果MBG能够加载rootClass，那么MBG不会覆盖和父类中完全匹配的属性-->
            	<!--属性名完全相同-->
            	<!--属性类型相同-->
            	<!--属性有getter方法-->
            	<!--属性有setter方法-->
            <!--trimStrings：判断是否对数据库查询结果进行trim操作，默认值为false，设置true会生成代码-->
            	<!--
					public void setUsername(String username) {
                        this.username = username == null ? null : username.trim();
                    }
                -->
        </javaModelGenerator>
        
        <!--用于配置SQL映射生成器（Mapper.xml）的属性，该标签可选，最多配置一个，如果targetRuntime设置为MyBatis3，则只有当javaClientGenerator配置需要XML时，该标签才必须配置一个-->
        <sqlMapGenerator targetPackage="top.mnsx.mybatis.xml" targetProject="D:\workspace\src\main\resources">
            <!--targetPackage：必选，生成SQL映射文件（XML文件）存放的包名，一个班就是放在改包下，实际还会受其他配置的影响-->
            <!--targetProject：必选，指定明确项目路径，可以使用相对路径或绝对路径-->
            <!--enableSubPackages：如果是true，那么MBG会根据catelog和schema来生成子包，如果false就直接使用targetPackages属性，默认false-->
        </sqlMapGenerator>
        
        <!--用于配置Java客户端生成器（Mapper接口）的属性，该标签可选，最多配置一个，如果不配置，那么不生成Mapper接口-->  
      	<javaClientGenerator type="XMLMAPPER" targetPackage="top.mnsx.dao" targetProject="src\main\java">
        	<!--type：必选，用于选择客户端代码生成器，用户自定义需要集成org.mybatis.generator.codegen.AbstractJavaClientGenerator类，必须有一个默认空的构造方法，预设代码生成器-->
            	<!--MyBatis3
					* ANNOTATEDMAPPER：基于注解的Mapper接口，不会有对应的XML映射文件
					* MIXEDMAPPER：XML和注解的混合形式
					* XMLMAPPER：所有方法都在XML中，接口调用依赖XML文件
				-->
            	<!--MyBatis3Simple
					* ANNOTATEDMAPPER：基于注解的Mapper接口，不会有对应的XML映射文件
					* XMLMAPPER：所有方法都在XML中，接口调用依赖XML文件
				-->
            <!--targetPackage：必选，生成Mapper接口存放的包名，一般就是放在该包下，实际还会收到其他配置的影响-->
            <!--targetObject：必选，指定目标项目路径，可以使用相对路径或绝对路径-->
            <!--implementationPackage：如果指定了该属性，Mapper接口的实现类就会生成在该属性指定的包中-->
        </javaClientGenerator>
        
        <!--非常重要，用于配置需要通过内省数据库的表，只有在table配置过的表，才能经过上述其他配置生成最终代码，该标签至少需要配置一个，可以配置多个-->
        <table tableName="tb_user" domainObjectName="User">
            <!--tableName：必选，执行生成的表名，可以使用SQL通配符匹配多个ibao-->
            <!--domainObjectName：生成对象的基本名称，如果没有指定，MBG会自动根据表明来生成名称-->
            <!--generateKey：指定自动生成主键的属性，MBG将在生成insert的SQL映射文件中插入一个selectKey标签，只能配置一个-->
            	<!--column：必选，生成列的列名-->
            	<!--sqlStatement：必选，返回新值的SQL语句，参考例子-->
            	<!--identity：设置为true时，该列会被标记为identity列，并且selectKey标签会被插入insert后面，设置为false，selectKey会插入insert之前，默认false-->
            	<!--type：type=post且identity=true时，生成的selectKey中order=AFTER，如果type=pre那么identity只能时false-->
            <generatedKey column="id" sqlStatement="MySql"/>
            <!--cloumnRenameingRule：该标签最多可以配置一个，使用该标签可以生成列之前对列进行重命名，这对于那些存在同一前缀的字段非常有用-->
            	<!--searchString：用于定义将要被替换的字符串的正则表达式-->
            	<!--replaceString：用于替换搜索字符串列每一个匹配列的字符串，如果没有指定就是用空-->
            <columnRenamingRule searchString="^tb_"/>
            <!--cloumnOberride：用于将某些默认计算的属性更改为指定值，标签可选，可以配置多个-->
            	<!--column：必选要重写的列名-->
            	<!--property：要使用Java属性的名称，如果灭有指定，MBG会根据列明生成-->
            	<!--javaType：列的属性值为完全限定的Java类型-->
            	<!--jdbcType：列的JDBC类型-->
            	<!--typeHandler：根据用户定义的需要用来处理列的类型处理器，必须集成TypeHandler接口的全限定类名，如果不设置或者为空，那么使用默认-->
            	<!--delimitedColumnName：指定是否应该在生成的SQL的里名称上增加分隔符-->
           	<!--ignoreColumn：用于屏蔽不许哟啊生成的列，可选，可以配置多个-->
            	<!--column：必选，表示要忽略的列名-->
            	<!--delimitedColumnName：表示是否区分大小写-->
        </table>
    </context>
</generatorConfiguration>
```

```java
// 自定义注解形式
/**
 * @BelongsProject: mnsx-mybatis
 * @User: Mnsx_x
 * @CreateTime: 2022/10/28 13:17
 * @Description: 自定义MBG注解类
 */
public class MnsxCommentGenerator extends DefaultCommentGenerator {

    private boolean suppressAllComments;

    private boolean addRemarkComments;

    /**
     * 从注解中获取对应参数属性
     * @param properties 解析的数据
     */
    @Override
    public void addConfigurationProperties(Properties properties) {
        super.addConfigurationProperties(properties);
        this.suppressAllComments = Boolean.parseBoolean(properties.getProperty(PropertyRegistry.COMMENT_GENERATOR_SUPPRESS_ALL_COMMENTS));
        this.addRemarkComments = Boolean.parseBoolean(properties.getProperty(PropertyRegistry.COMMENT_GENERATOR_ADD_REMARK_COMMENTS));
    }

    /**
     * 给字段添加注解信息
     * @param field 字段
     * @param introspectedTable 数据库表信息封装
     * @param introspectedColumn 数据库字段信息封装
     */
    @Override
    public void addFieldComment(Field field, IntrospectedTable introspectedTable, IntrospectedColumn introspectedColumn) {
        // 如果阻止生成所有注释，那么直接返回
        if (suppressAllComments) {
            return;
        }

        // 文档注释开始
        field.addJavaDocLine("/**");

        // 获取数据库字段的备注信息
        String remarks = introspectedColumn.getRemarks();

        // 根据参数和备注信息判断是否添加备注信息
        if (addRemarkComments && StringUtility.stringHasValue(remarks)) {
            field.addJavaDocLine(" * " + remarks);
        }

        // 由于Java对象名和数据库字段名可能不一样，注释中保存数据库字段名
        field.addJavaDocLine(" * " + introspectedColumn.getActualColumnName());
        field.addJavaDocLine(" */");
    }
}
```

# MyBatis高级查询

## 一对一映射

* 使用resultMap的association标签配置一对一映射'

  ```xml
  <resultMap id="userRoleMap" extends="userMap"
             type="top.mnsx.test.entity.Sysuser">
  	<result property="id" column="id"/>
      <result property="roleName" column="role_name"/>
  </resultMap>
  ```

  association参数分析——

  * property：对应实体类中的属性名，必填项

  * javaType：属性对应的Java类型

  * resultMap：可以直接使用现有的resultMap，而不需要在这里配置

  * columnPrefix：查询类的前缀，配置前缀后，在子标签配置result的column时，可以省略前缀

* association标签的嵌套查询

  ```xml
  <resultMap id="userRoleMapSelect" extends="userMap" type="top.mnsx.test.entity.SysUser">
  	<association property="role" column="{id = role_id}" select="top.mnsx.test.mapper.RoleMapper.selectRoleById"</association>
  </resultMap>
  ```

  association标签嵌套查询常用属性——

  * select：另一个映射查询的id，MyBatis会额外执行这个查询获取嵌套对象的结果
  * column：列名，将主查询中列的结果作为嵌套查询的参数，配置方式column={prop1=col1, prop2=col2}
  * fetchType：数据加载方式，可选值lazy和eager，分别为延迟加载和积极加载

## 一对多映射

* collection集合的嵌套结果映射

  ```xml
  <resultMap id="userRoleListMap" extends="userMap" type="top.mnsx.test.entity.SysUser">
  	<collection property="roleList" columnPrefix="role_" resultMap="top.mnsx.test.mapper.roleMapper"/>
  </resultMap>
  ```

  id的唯一作用时**在嵌套的映射配置时判断数据是否相同**

  如果没有配置id，MyBatis就会把resultMap中配置所有字段进行比较，**如果所有字段的值都相同就合并，只要有一个字段值不同，既不合并**

* collection标签的嵌套查询

  collection标签的嵌套查询属性——

  * select：另一个映射查询的id，MyBatis会额外执行这个查询获取嵌套对象的结果
  * column：列名，将主查询中列的结果作为嵌套查询的参数，配置方式column={prop1=col1, prop2=col2}
  * fetchType：数据加载方式，可选值lazy和eager，分别为延迟加载和积极加载

# MyBatis缓存配置

## 一级缓存

MyBatis的以及缓存存在于SqlSession的生命周期中，**在同一个SqlSession中查询时**，MyBatis会**把执行的方法和参数通过算法生成缓存的键值**，将键**值和查询结果存放进入一个Map中**，如果**同一个SqlSession**中执行的方法和参数完全一致，那么通过算法生成相同的键值，当Map缓存对象中已经存在该键值时，则返回缓存中的对象

如果当第一次获取数据后，通过setter方法修改对象的数据，那么下一次方法参数相同的获取数据，将不会获取数据库中的数据，而是获取缓存中的已经被setter修改过的数据

```xml
<!--解决方法-->
<select id="selectById" flushCache="true" resultMap="userMap">
	select * from sys_user where id = #{id}
</select>
```

为方法添加一个flushCache="true"，这个属性配置为true后，会在查询数据前当空当前的一级缓存，因此该方法每次都会重新从数据库总查询数据

或者使用insert、update、delete都会清空一级缓存

## 二级缓存

二级缓存存在于SqlSessionFactory的生命周期中

在保证二级缓存的全局配置开启的情况下，会给mapper.xml开启二级缓存只需要在mapper.xml中添加\<cache/>元素即可

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="top.mnsx.mapper.testMapper">
	<cache/>
</mapper>
```

默认的二级缓存配置——

* 映射文件中所有的SELECT语句都会被缓存
* 映射文件中所有INSERT、UPDATE、DELETE语句都会刷新缓存
* 缓存会使用Least Recently Userd算法来回收
* 根据时间表，缓存不会以任何时间顺序来刷新
* 缓存会存储集合或对象的1024个引用
* 缓存会被视为read/write的，意味着对象检索不是共享的，而且可以安全地被调用者修改，而不干扰其他调用者或线程所作的潜在修改

```xml
<cache
       eviction="FIFO"
       flushInterval="60000"
       size="512"
       readOnly="true"/>
```

二级缓存高级属性——

* eviction（回收策略）

  * LRU（最近最少使用）：移除最长时间不被使用的对象，默认
  * FIFO（先进先出）：按对象进入缓存的顺序来移除他们
  * SOFT（软引用）：移除基于垃圾回收器的状态和软引用规则的对象
  * WEAK（弱引用）：更积极的移除基于垃圾收集器状态的弱引用规则对象

* flushInterval（刷新间隔）

  可以被设置为任意正整数，而且他们代表一个合理的毫秒形式的时间段，默认情况不设置，没有刷新间隔，缓存仅在调用语句时刷新

* size（引用数目）

  可以被设置为任意正整数，要记住缓存的对象数目和运行环境的可用内存资源数目，默认1024

* readOnly（只读）

  属性可以被设置为true或false，只读缓存会给所有调用者返回缓存对象的相同实例，因此这些对象不能被修改，者提供了很重要的性能优势，可以读写的缓存会通过序列化返回缓存对象的拷贝，这种方式会慢一些但是能保证安全，因此默认false

**当调用close方法关闭SqlSession时，SqlSession才会保存查询数据到二级缓存中**

> **为什么避免使用二级缓存**
>
> 在UserMapper.xml中有大多数针对user表的操作，到那时在一个xxxMapper.xml中，还有针对user单表的操作
>
> 这会导致user在两个命名空间中的数据不一致

# MyBatis插件开发

## 拦截器接口介绍

```java
public interface Interceptor {
    Object intercept(Invocation invocation) throws Throwable;
    Object plugin(Object target);
  	void setProperties(Properties properties);
}
```

* setProperties方法，用来传递插件的参数，可以通过参数来改变插件的行为

* plugin方法，参数target就是拦截器要拦截的对象，该方法会在创建被拦截的接口实现类时被调用

  只需要调用MyBatis提供的Plugin类的warp静态方法就可以通过Java的动态代理拦截目标对象

  ```java
  @Override
  public Object plugin(Object target) {
      return Plugin.wrap(taret, this);
  }
  ```

* intercept方法时MyBatis运行时要执行的拦截方法，通过该方法的参数invocation可以得到很多有用的信息

  ```java
  @Override
  public Object intercept(Invocation invocation) trhows Throwable {
      Object target = invocation.getTarget();
      Method method = invocation.getMethod();
      Obejct[] args = invocation.getArgs();
      Object result = invocation.proceed();
      return result;
  }
  ```

## 拦截器签名介绍

@Intercepts注解中的属性是一个@Signature数组，可以在同一个拦截器中同时拦截不同的接口和方法

@Signature注解的关键属性：

* type：设置拦截的接口
* method：设置拦截接口中的方法名
* args：设置拦截方法的参数类型数组，通过方法名和参数类型可以确定唯一一个方法

## Executor接口

```java
public interface Executor {

  ResultHandler NO_RESULT_HANDLER = null;

  int update(MappedStatement ms, Object parameter) throws SQLException;

  <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey cacheKey, BoundSql boundSql) throws SQLException;

  <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException;

  <E> Cursor<E> queryCursor(MappedStatement ms, Object parameter, RowBounds rowBounds) throws SQLException;

  List<BatchResult> flushStatements() throws SQLException;

  void commit(boolean required) throws SQLException;

  void rollback(boolean required) throws SQLException;

  CacheKey createCacheKey(MappedStatement ms, Object parameterObject, RowBounds rowBounds, BoundSql boundSql);

  boolean isCached(MappedStatement ms, CacheKey key);

  void clearLocalCache();

  void deferLoad(MappedStatement ms, MetaObject resultObject, String property, CacheKey key, Class<?> targetType);

  Transaction getTransaction();

  void close(boolean forceRollback);

  boolean isClosed();

  void setExecutorWrapper(Executor executor);

}

```

* **常用方法**

```java
int update(MappedStatement ms, Object parameter) throws SQLException;
```

**该方法会在所有INSERT、UPDATE、DELETE执行时被调用**，因此如果想要拦截这3类操作，可以拦截这个方法

```java
<E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException;
```

**该方法会在所有SELECT查询方法执行时被调用**，通过这个接口参数可以获取很多有用的信息，因此这是最长被拦截的一个方法，使用该方法需要注意的是，*虽然接口中还有一个参数更多的同名接口，但是MyBatis的设计，这个参数多的接口不能被拦截*

```java
List<BatchResult> flushStatements() throws SQLException;
```

**该方法只在同故宫SqlSession方法调用flushStatements方法或执行的接口方法中带有@Flush注解时才被调用**

```java
void commit(boolean required) throws SQLException;
```

**该方法只在通过SqlSession方法调用commit方法时才被调用**

```java
void rollback(boolean required) throws SQLException;
```

**该方法只在通过SqlSesson方法调用rollback方法时才被调用**

```java
Transaction getTransaction();
```

**该方法只在通过SqlSession方法获取数据库连接时才被调用**

```java
void close(boolean forceRollback);
```

## ParameterHandler接口

```java
public interface ParameterHandler {

    Object getParameterObject();

    void setParameters(PreparedStatement ps) throws SQLException;

}
```

* 常用方法

```java
Object getParameterObject();
```

**该方法只在执行存储过程处理出参的时候被调用**

```java
void setParameters(PreparedStatement ps) throws SQLException;
```

**该方法在所有数据库方法设置SQL参数时被调用**

## ResultSetHandler接口

```java
public interface ResultSetHandler {

  <E> List<E> handleResultSets(Statement stmt) throws SQLException;

  <E> Cursor<E> handleCursorResultSets(Statement stmt) throws SQLException;

  void handleOutputParameters(CallableStatement cs) throws SQLException;

}
```

* 常用方法

```java
<E> List<E> handleResultSets(Statement stmt) throws SQLException;
```

**该方法会在除存储过程及返回值类型为Cursor\<T>以外的查询方法中被调用**

```java
void handleOutputParameters(CallableStatement cs) throws SQLException;
```

**该方法只在使用存储过程处理出参时被调用**

## StatementHandler接口

```java
public interface StatementHandler {

  Statement prepare(Connection connection, Integer transactionTimeout)
      throws SQLException;

  void parameterize(Statement statement)
      throws SQLException;

  void batch(Statement statement)
      throws SQLException;

  int update(Statement statement)
      throws SQLException;

  <E> List<E> query(Statement statement, ResultHandler resultHandler)
      throws SQLException;

  <E> Cursor<E> queryCursor(Statement statement)
      throws SQLException;

  BoundSql getBoundSql();

  ParameterHandler getParameterHandler();

}
```

* 常用方法

```java
Statement prepare(Connection connection, Integer transactionTimeout) throws SQLException;
```

**该方法会在数据库执行前被调用，优先于当前接口中的其他方法而被执行**

```java
void parameterize(Statement statement) throws SQLException;
```

**刚方法在prepare方法之后执行，用于处理参数信息**

```java
<E> List<E> query(Statement statement, ResultHandler resultHandler) throws SQLException;
```

**执行SELECT方法时调用**

# 1.0 MyBatis简介

## MyBatis特性

1. MyBatis是支持定制化SQL、存储过程以及高级映射得优秀得持久层框架
2. MyBatis避免了几乎所有得JDBC代码和手动设置参数以及获取结果集
3. MyBatis可以使用简单得XML或者注解用于配置和原始映射，将接口和java得POJO映射成数据库中得记录
4. MyBatis是一个半自动的ORM框架

## 和其他持久化层技术对比

* JDBC
    * SQL夹杂在java代码中耦合度高，导致硬编码内伤
    * 维护不易且实际开发需求中SQL有变化，频繁修改的情况多见
    * 代码冗长，开发效率低
* Hibernate和JPA
    * 操作简便，开发效率高
    * 程序中的长难复杂SQL需要绕过框架
    * 内部自动生产的SQL，不容易做特殊优化
    * 基于全映射的全自动框架，大量字段的POJO进行部分映射时比较困难
    * 反射操作太多，导致数据库性能下降
* MyBatis
    * 轻量级，性能出色
    * SQL和java编码分开，功能边界清晰，java代码专注业务、SQL语句专注数据
    * 开发效率稍逊于Hibernamte，但是完全能够接受

# 搭建MyBatis

## 引入依赖

```xml
<dependencies>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.7</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.3</version>
        </dependency>
    </dependencies>
```

## 创建MyBatis的核心配置文件

> 习惯上命名为mybatis-config.xml，这个文件名建议，并非强制要求。将来整合Spring之后，这个配置文件可以省略
>
> 核心配置文件主要用于配置连接数据库的环境以及MyBatis的全局配置信息
>
> 核心配置文件存放的位置是src/main/resources目录下

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
      </dataSource>
    </environment>
  </environments>
  <mappers>
    <mapper resource="org/mybatis/example/BlogMapper.xml"/>
  </mappers>
</configuration>
```

## 创建mapper接口

> MyBatis中的mapper接口相当于dao，但是不需要实现类

```java
public interface UserMapper {
    /**
     * 添加用户信息
     */
    int insertUser();
}
```

## 创建MyBatis的映射文件

1. 映射文件的命名规则：

   表所对应的实体类的类名+Mapper.xml

   因此一个映射文件对应一个实体类，对应一张表的操作

   MyBatis映射文件用于编写SQL，访问以及操作表中的数据

   MyBatis映射文件存放的位置是src/main/resources/mappers目录下

2. MyBatis中可以面向接口操作数据，要保证两个一致：

   mapper接口的全类名和映射文件的名称空间保持一致

   mapper接口中方法的方法名和映射文件中编写SQL的标签的id属性保持一致

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="top.mnsx.mapper.UserMapper">
    <insert id="insertUser">
        insert into t_user values(1, 'admin', '123456', 23, '男', '110@qq.com');
    </insert>
</mapper>   
```

## 测试功能

```java
public class MyBatisTest {

    @Test
    public void testMyBatis() throws IOException {
        InputStream resourceAsStream = Resources.getResourceAsStream("mybatis-config.xml");
        SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
        SqlSessionFactory build = sqlSessionFactoryBuilder.build(resourceAsStream);
        SqlSession sqlSession = build.openSession(true);
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
        int i = userMapper.insertUser();
        System.out.println(i);
        sqlSession.commit();
        sqlSession.close();
    }
}
```

# 核心配置文件详解

核心配置文件中标签必须按照固定的顺序——

properties——》setting——》typeAliases——》typeHandlers——》objectFactory——》objectWrapperFactory——》reflectorFactory——》plugins——》environments——》databaseIdProvider——》mappers

## environments属性

environments：配置多个连接数据库的环境

 default：设置默认使用的环境的id

environment：配置莫格具体的环境

 id：表示连接数据库的环境的唯一标识，不能重复

transactionManager：设置事务管理方式

 type：jdbc|managed

 jdbc：表示当前环境中，操作SQL时使用的时jdbc中的原生事务管理方式，事务的提交或者回滚需要手动处理

 managed：被管理

datasource：配置数据源

 type：设置数据源的类型（POOLED|UNPOOLED|JNDI）

 POOLED：表示使用数据库连接池，缓存数据库连接

 UNPOOLED：不使用数据库连接池

 JNDI：表示使用上下文中的数据源

## properties属性

```xml
<properties resource="jdbc.properties"/>
```

引入properties文件，通过使用${...}的方式来获取properties中的值

## typeAliases属性

typeAlias：设置某个类型的别名

 type：设置需要设置别名的类型

 alias：设置某个类型的别名

 不设置alias属性那就表示为该类的类名且不区分大小写

package：以包为单位，将包下所有的类型设置默认的类型别名，也就是类名且不区分大小写

```xml
    <typeAliases>
        <typeAlias type="top.mnsx.entity.User" alias="User"/>
    </typeAliases>
```

**类型别名不区分大小写**

## mappers属性

mappers

 package：以包为单位引入映射文件

要求：

1. mapper接口所在的包和映射文件所在的包一致
2. mapper接口要和映射文件名字一致

```xml
<package name="top.mnsx.mybatis.xml"/>
```

# SqlSession工具类

```java
public class SqlSessionUtils {
    public static SqlSession getSqlSession() {
        SqlSession sqlSession = null;
        try {
            InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
            SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
            sqlSession = sqlSessionFactory.openSession(true);
        } catch (IOException e) {
            e.printStackTrace();
        }
        return sqlSession;
    }
}
```

# MyBatis获取参数数值的两种方法

MyBatis获取参数的两种方法：${}和#{}

${}的本质就是字符串的拼接，#{}的本质就是占位符赋值

${}使用字符串拼接的方式拼接sql，若为字符串类型或日期类型的字段进行赋值时，需要手动添加单引号；但是#{}使用占位符赋值的方式拼接sql，此时为字符串类型或日期类型的字段进行赋值时，可以自动添加单引号

## 单个字面量类型

mapper接口方法的参数为单个字面量类型

可以通过${}和#{}以任意的名称获取参数值，但是需要注意${}的单引号问题

## 多个字面量类型——方式一

mapper接口方法参数为多个时

1. 以arg0，arg1...为键，以参数为值
2. 以param1，param2...为键，以参数为值

因此只需要通过${}和#{}以键的方式访问值即可，但是需要注意${}的单引号问题

## 多个字面量类型——方式二

mapper接口方法的参数有多个时，可以手动将这些参数放在一个map中存储

只需要通过${}和#{}以键的方式访问值即可，但是需要注意${}的单引号问题

## 实体类类型

mapper接口方法的参数时实体类类型的参数

只需要通过${}和#{}以属性的方式访问属性值即可，但是需要注意${}的单引号问题

## 使用@Param命名参数

此时MyBatis会将这些参数放到一个map集合中，以两种方式进行存储

* 以@Param为键，以参数为值
* 以param1， param2...为键，以参数为值

因此只需要通过${}和#{}以键的方式访问值即可，但是需要注意${}的单引号问题

# MyBatis的各种查询功能

## 查询一个实体类对象

```java
/**
 * 根据用户id查询用户信息
 * @Param id
 * @return
 */
User getUesrBuId(@Param("id") int id)
```

```xml
<!--User getUesrBuId(@Param("id") int id)-->
<select id="getUserById" resultType="user">
	select * from t_user where id = #{id} 
</select>
```

## 查询一个list集合

```java
/**
 * 查询所有用户信息
 * @Return
 */
List<User> getUserList();
```

```xml
<!--List<User> getUserList()-->
<select id = "getUserList" resultType="User">
	select * from t_user
</select>
```

**注意：一定不能通过实体类对象接收，此时会抛异常TooManyResultsException**

## 查询单个数据

```java
/**
 * 查询用户的总记录数
 * @return
 * 在MyBatis中，对于Java中常用的类型都设置了类型别名
 */
int getCount();
```

```xml
<!--int getCount()-->
<select id-"getCount" resultType="integer">
	select count(id) from t_user
</select>
```

## 查询一条数据为map集合

```java
/**
 * 根据用户id查询用户信息为map集合
 * @param id
 * @return
 */
Map<String, Object> getUserToMap(@Param("id") int id);
```

```xml
<!--Map<String, Object> getUserToMap(@Param("id") int id)-->
<select id="getUserToMap" resultType="map">
	select * from t_user where id = #{id}
</select>
```

## 查询多条数据为map集合

```java
/**
 * 查询所有用户信息为Map集合
 * @return
 */
 @MapKey("id")
Map<String, Object> getAllUserToMap();
```

```xml
<!--Map<String, Object> getAllUserToMap()-->
<select id="getAllUserToMap" resultType="map">
	select * from t_user
</select>
```

# 特殊SQL的执行

## 模糊查询

```java
/**
 * 模糊查询
 * @param username
 * @return
 */
List<User> getUserByLike(@Param("username") String username);
```

```xml
<!--List<User> getUserByLike(@Param("username") String username)-->
<select id="getUserByLike" resultType="User">
	<!--select * from t_user where username like '%${username}%'-->
    <!--select * from t_user where username like concat('%', #{username}, '%')-->
    select * from t_user where username like "%"#{username}"%"
</select>
```

## 批量删除

```java
/**
 * 批量删除
 * @param ids
 * @return
 */
int deleteMore(@Param("ids") String ids);
```

```xml
<!--int deleteMore(@Param("ids") String ids)-->
<delete id="deleteMore">
	delete from t_user where id in (${ids})
</delete>
```

## 动态设置表名

```java
/**
 * 动态设置表明，查询所有的用户信息
 * @param tableName
 * @return
 */
List<User> getAllUser(@param("tableName") Strign tableName);
```

```xml
<!--List<User> getAllUser(@Param("tableName") String tablename)-->
<select id="getAllUser" resultType="User">
	select * from ${tableName}
</select>
```

## 获取添加功能自增的主键

```java
/**
 * 添加用户
 * @param user
 * @return
 */
void insertUser(User user);
```

```xml
<!--void insertUser(User user)-->
<insert id="insertUser" useGeneratedKeys="true" keyProperty="id">
	insert into t_user values(null, #{username}, #{password}, #{age}, #{sex}, #{email})
</insert>
```

# 自定义映射resultMap

## 字段名和实体类属性名不一致

1. 通过为字段起别名，保持和属性名一致

```xml
<select id="getAllEmp" resultType="Emp">
	select eid, emp_name empName, age, sex, email from t_emp
</select>
```

2. 通过全局配置mapUnderScoreToCamelCase

```xml
<settings>
    <setting name="mapUnderScoreToCamelCase" value="true"/>
<settings/>
```

3. 通过resultMap设置自定义映射关系

```xml
<resultMap id="empResultMap" type="Emp">
	<id property="eid" column="eid"></id>
    <result property="empName" column="emp_name"></result>
    <result property="age" column="age"></result>
    <result property="sex" column="sex"></result>
    <result property="email" column="email"></result>
</resultMap>
```

resultMap：设置自定义映射关系

 id：唯一标识，不能重复

 type：设置映射关系中的实体类类型

 id：设置主键的映射关系

 result：设置其他属性的映射关系

 property：设置映射关系中的属性名，必须时type属性所设置的实体类类型中的属性名

 column：设置映射关系中的字段名，必须是sql语句查询出的字段名

## 处理多对一的映射关系

1. 级联属性赋值解决多对一映射关系

```xml
<resultMap id="empAndDeptResultMapOne" type="Emp">
	<id property="eid" column="eid"></id>
    <result property="empName" column="emp_name"></result>
    <result property="age" column="age"></result>
    <result property="sex" column="sex"></result>
    <result property="email" column="email"></result>
    <result property="dept.did" column="did"></result>
    <result property="dept.deptName" column="dept_name"></result>
</resultMap>
```

2. 通过association解决多对一的映射关系

```xml
<resultMap id="empAndDeptResultMapOne" type="Emp">
	<id property="eid" column="eid"></id>
    <result property="empName" column="emp_name"></result>
    <result property="age" column="age"></result>
    <result property="sex" column="sex"></result>
    <result property="email" column="email"></result>
    <association propety="dept" javaType="Dept">
    	<id property="did" column="did"></id>
        <result property="deptName" column="dept_name"></result>
    </association>
```

association：处理多对一的映射关系

 property：需要处理多对一的映射关系的属性名

 javaType：该属性的类型

3. **分布查询**

```java
/**
 * 通过分布查询，查询园中以及员工所对应的部门信息
 */
Emp getEmpAndDeptByStepOne(@Param("eid") Integer eid);
```

```xml
<select id="getEmpAndDeptByStepOne" resultMap="empAndDeptByStepResultMapTwo">
	select * from t_emp where eid=#{eid}
</select>
<resultMap id="empAndDeptByStepResultMapTwo" type="Emp">
	<id property="eid" column="eid"></id>
    <result property="empName" column="emp_name"></result>
    <result property="age" column="age"></result>
    <result property="sex" column="sex"></result>
    <result property="email" column="email"></result>
    <association property="dept"
                 select="top.mnsx.mybatisstudy.mapper.getEmpAndDeptByStepTwo"
                 column="did"></association>
</resultMap>
```

```java
/**
 * 通过分布查询，擦汗寻员工以及员工所对应的部门信息
 */
Dept getEmpAndDeptByStepTwo(@Param("did") Integer did);
```

```xml
<select id="getEmpAndDeptByStepTwo" resultType="Dept">
	select * from t_dept where did = #{did}
</select>
```

select：设置分布查询的sql的唯一标识

column：设置分布查询的条件

> 分布查询的优点：可以实现延迟加载，但是必须在社心配置文件中设置全局配置信息
>
> lazyLoadingEnabled：延迟加载的全局开关。当开启时，所有关联对象都会延迟加载
>
> aggressiveLazyLoading：当开启时，任何方法的调用都会加载该对象的所有属性。否则，每个属性会按需加载
>
> 此时就可以实现按需加载，获取的数据是什么，就会执行相应的sql。此时可以通过association和collection中的fetchType属性设置当前的分布查询是否使用延迟加载，fetchType=“lazy|eager“

## 处理一对多映射关系

1. 通过collection解决一对多映射关系

```xml
<resultMap id="deptAndEmpResultMap" type="Dept">
	<id property="did" column="did"></id>
    <result property="deptName" column="dept_name"></result>
    <collection property="emps" ofType="Emp">
   		<id property="eid" column="eid"></id>
    	<result property="empName" column="emp_name"></result>
    	<result property="age" column="age"></result>
    	<result property="sex" column="sex"></result>
    	<result property="email" column="email"></result>
    </collection>
</resultMap>
```

collection：处理一对多的映射关系

ofType：属性所对应的集合中存储数据的类型

2. 通过分布查询解决一对多映射关系

```java
/**
 * 通过分布查询，查询部门以及部门中所有员工的信息
 */
Dept getDeptAndEmpByStepOne(@Param("did") Integer did);
```

```xml
<select id="getDeptAndEmpByStepOne" resultMap="">
	select * from t_dept where did = #{did}
</select>
<resultMap id="empAndDeptByStepResultMapTwo" type="Emp">
    <id property="did" column="did"></id>
    <result property="deptName" column="dept_name"></result>
	<collection property="emps"
                select="top.mnsx.mybatisstudy.mapper.getDeptAndEmpByStepTwo"
                column="did"></collection>
</resultMap>
```

```java
/**
 * 通过分布查询，查询部门以及部门中所有员工的信息
 */
List<Emp> getDeptAndEmpByStepTwo(@Param("did") Integer did);
```

```xml
<select id="getDeptAndEmpByStepTwo" resultType="Emp">
	select * from t_emp where did = #{did}
</select>
```

# 动态SQL

MyBatis框架的动态SQL技术是一种根据特定条件动态拼装SQL语句的功能，它存在的意义是为了解决拼接SQL语句字符串是的痛点问题

## if

if标签可通过test属性的表达式进行判断，若表达式的结果为true，则标签中的内容会执行；反之标签中的内容不会执行

```xml
<select id="getEmpListByMoreTj" resultType="Emp">
	select * from t_emp where 1=1
    <if test="ename != '' and ename != null">
    	and ename = #{ename}
    </if>
    ...
</select>
```

## where

* 当where标签中有内容时，会自动生成where关键字，并且将内容前多余的and或or去掉
* 当where标签中没有内容时，此时where标签没有任何效果
* **注意：**where标签不能将其中内容后的多余的and去掉

```xml
<select id="getEmpListByMoreTj" resultType="Emp">
	select * from t_emp
    <where>
        <if test="ename != '' and ename != null">
    		and ename = #{ename}
    	</if>
    	...
    </where>
</select>
```

## trim

suffix/prefix：将trim标签中内容前面或后面添加指定内容

sffixOverrides/prefixOverrides：将trim标签中内容前面或后面删除指定内容

若标签中没有内容时，trim标签也没有任何效果

```xml
<select id="getEmpListByMoreTj" resultType="Emp">
	select * from t_emp
    <trim prefix="where" sufficOverrides="and|or">
        <if test="ename != '' and ename != null">
    		and ename = #{ename}
    	</if>
    	...
    </trim>
</select>
```

## choose、when、otherwise

相当于if...else if...else

when至少要有一个，otherwise最多只能有一个

```xml
<select id="getEmpByChoose" resultType="Emp">
	select * from t_emp
    <where>
    	<choose>
        	<when test="empName != null and empName != ''">
            	emp_name = #{empName}
            </when>
            ...
            <otherwise>
            	did = 1
            </otherwise>
        </choose>
    </where>
</select>
```

## foreach

collection：设置需要循环的数组或集合

item：表示数组或集合中每一个数据

separator：循环体之间的分隔符

open：foreach标签所循环的所有内容的开始符

close：foreach标签所循环的所有内容的结束符

* 数组批量删除

```xml
<delete id="delteMoreByArray">
	delete from t_emp where eid in
    <foreach collection="eids" item="eid" separator=",">
    	#{eid}
    </foreach>
</delete>
```

```xml
<delete id="delteMoreByArray">
	delete from t_emp where eid in
    <foreach collection="eids" item="eid" separator="or">
    	eid = #{eid}
    </foreach>
</delete>
```

* 集合批量添加

```xml
<insert id="insertMoreByList">
	insert into t_emp values
    <foreach collection="emps" item="emp" separator=",">
    	(null, #{emp.empName}, #{emp.age}, #{emp.sex}, #{emp.email}, null)
    </foreach>
</insert>
```

## sql

```xml
<sql id="empColumns">eid, emp_name, age, sex, email</sql>

<select id="getEmpByCondition" resultType="Emp">
	select <include refid="empColumns"></include> from t_emp
    ...
</select>
```

# MyBatis缓存

## MyBatis的一级缓存

一级缓存是SqlSession级别的，通过同一个SqlSession查询的数据会被缓存，下一次查询相同的数据，就会从缓存中直接获取，不会从数据库中重新访问

使用一级缓存失效的四种情况：

1. 不同的SqlSession对应不同的一级缓存
2. 同一个SqlSession但是查询条件不同
3. 同一个SqlSession两次查询期间执行了任何一次增删改查操作
4. 同一个SqlSession两次查询期间手动清空了缓存

```java
clearCache() //清空缓存
```

## MyBatis的二级缓存

二级缓存是SqlSessionFactory级别，通过同一个SqlSessionFactory创建的SqlSession查询的结果会被缓存，此后若再次执行相同的查询语句，结果就会从缓存中获取

二级缓存开启的条件：

1. 在核心配置文件中，设置全局配置属性cacheEnable=true，默认为true，不需要设置
2. 在映射文件中设置标签`<cache/>`
3. 二级缓存必须在SqlSession关闭或提交之后才有效
4. 查询的数据所转换的实体类类型必须实现序列化接口

使二级缓存失效的情况：

两次查询之间执行了任意的增删改，会使一级和二级缓存同时失效

**二级缓存相关配置**

* eviction属性：缓存回收策略

  LUR——最近最少使用的：移除最长时间不被使用的对象

  FIFO——先进先出：按对象进入缓存的顺序来移除他们

  SOFT——软引用：移除基于垃圾回收器状态和软引用规则的对象

  WEAK——弱引用：更积极地移除基于垃圾回收器状态和弱引用规则的对象

  **默认LRU**

* flushInterval属性：刷新间隔，单位毫秒

  默认情况是不设置，也就是没有刷新间隔，缓存仅仅调用语句时刷新

* size属性：引用数目，正整数

* readOnly属性：只读，true/false

  true：只读缓存；会给所有调用者返回缓存对象的相同实例。因此这些对象不能被修改。这提供了很重要的性能优势

  false：读写数据；会返回缓存对象的拷贝，通过序列化。这会慢一些但是安全，默认为false

## MyBatis缓存查询的顺序

* 先查询二级缓存，因为二级缓存中可能会有其他程序已经查出来的数据可以直接使用
* 如果二级缓存没有命中，再查询一级缓存
* 如果一级缓存也没有命中，则查询数据库
* SqlSession关闭之后，一级缓存中的数据也会写入二级缓存

## 整合第三方缓存EHCache

1. 添加缓存

```xml
<dependency>
	<groupId>org.mybatis.caches</groupId>
    <artifactId>mybatis-ehcache</artifactId>
    <version>1.2.1</version>
</dependency>
<dependency>
	<groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.3</version>
</dependency>
```

2. 各jar包功能

   | jar包名称       | 作用                            |
      | --------------- | ------------------------------- |
   | mybatis-encache | Mybatis和EHCache的整合包        |
   | ehcache         | EHCache核心包                   |
   | slf4j-api       | SLF4J日志门面包                 |
   | logback-classic | 支持SLF4J门面接口的一个具体实现 |

3. 创建EHCache的配置文件ehcache.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="../config/ehcache.xsd">
	<diskStore path="D:\atguigu\ehcache"/>
    
    <defaultCache
                  maxElementsInMemory="1000"
                  maxElementsOnDisk="10000000"
                  eternal="false"
                  overflowToDisk="true"
                  timeToIdleSeconds="120"
                  timeToLiveSeconds="120"
                  diskExpiryThreadIntervalSeconds="120"
                  memoryStoreEvictionPolicy="LRU">
    </defaultCache>
</ehcache>
```

4. 设置二级缓存的类型

```xml
<cache type="org.mybatis.caches.ehcache.EhcadcheCache"/>
```

# MyBatis的逆向工程

https://www.bilibili.com/video/BV1VP4y1c7j7?p=62

略

# MyBatis分页插件

1. 添加依赖

```xml
<dependency>
	<groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper</artifactId>
    <version>5.2.0</version>
</dependency>
```

2. 配置分页插件

```xml
<plugins>
	<plugin interceptor="com.github.pageheloper.PageInterceptor"></plugin>
</plugins>
```

3. 使用分页插件

   index：当前页的起始索引

   pageSize：每页显示的条数

   pageNum：当前页得页码

   index = （pageNum -1) \* pageSize

```java
@Test
public void testPageHelper() {
    try {
        InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
        SqlSession sqlSession = sqlSessionFactory.openSession(true);
        EmpMapper mapper = sqlSession.getMapper(EmpMapper.class);
        List<Emp> list = mapper.selectByExample(null);
        PageInfo<Emp> page = new PageInfo<>(list, 5);
    } catch (IOException e) {
        e.printStackTrace()
    }
}
```

