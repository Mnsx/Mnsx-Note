# MyBatis简介

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

