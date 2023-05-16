# MyBatis动态SQL

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



 
