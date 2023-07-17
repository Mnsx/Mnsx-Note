# IOC

## IOC概念

1. 控制反转，把对象创建和对象之间得调用过程，交给Spring进行管理

2. 使用IOC目的：为了耦合度降低

## IOC底层原理

* XML解析、工厂模式、反射

## IOC基本实现

1. xml配置文件，配置创建对象

```xml
<bean id="dao" class="top.mnsx.springstudy"></bean>
```

2. 有service类和dao类，创建工厂类

```java
class UserFactory{
    public static UserDao getDao(){
        String classValue = class属性; // xml解析
        Class clazz = Class.forName(classValue); // 通过反射创建对象
        return (UserDao)clazz.newInstance();
    }
}
```

## IOC接口

1. IOC思想基于IOC容器完成，IOC容器底层就是对象工厂

2. Spring提供IOC容器实现方式：（两个接口）

    * BeanFactory——IOC容器基本实现，是Spring内部使用接口，不提供开发人员进行处理

      **加载配置文件时候不会创建对象，在获取对象（使用）才去创建对象**

    * ApplicationContext——BeanFactory接口得子接口，提供更强大得功能，一般由开发人员进行使用

​                **加载配置文件时候会把配置文件对象进行创建**

3. ApplicationContext

- ——ConfigurableApplicationContext

    * ——AbstractApplicationContext

        * ——AbstractRefreshableApplicationContext

            * ——AbstractXmlApplicationContext

            * **——FileSystemXmlApplicationContext——实现类**

            * **——ClassPathXmlApplicationContext——实现类**

## IOC操作Bean管理

1. Bean管理——两个操作
    1. Spring创建对象
    2. Spring注入属性
2. Bean管理操作有两种方式
    1. 基于xml配置文件方式实现
    2. 基于注解方式实现

## IOC操作Bean管理（基于XML方式）

1. 基于xml方式创建对象

    * 在spring配置文件中，使用bean标签，标签里面添加对应得属性，就可以实现对象创建

    * 在bean标签有很多属性，介绍常用得属性——

        * id属性——唯一标识

        - class属性——创建对象所在类的全路径

    * 创建对象时候，默认也是执行的无参构造方法完成对象创建

2. 基于xml方式注入属性

​       **DI：依赖注入，就是注入属性**

* 第一种注入方式——使用set方法进行注入

  1.创建类，定义属性和对应的set方法

```java
public class Book{
	private String bname;
    private String bauthor;
    
    public void setBname(String bname){
        this.bname = bname;
    }
    
    public void setBauthor(String bauthor){
        this.bauthor = bauthor;
    }
}
```

2. 在spring配置文件配置对象创建，配置属性注入

```xml
<!-- set方法注入属性 -->
<bean id="book" class="top.mnsx.springstudy.Book">
  <!-- 使用property完成属性注入
			 name：类里面属性名称
			 value：向属性注入的值
	-->
 <property name="bname" value="九阴真经"></property>
 <property name="bauthor" value="达摩神功"></property>
</bean>
```

* 第二种注入方式——使用有参构造方法进行注入

1. 创建类，定义属性和对应的构造方法

```java
public class Orders{
	private String oname;
    private String address;
    
    public Book(String oname, String address){
        this.oname = oname;
        this.address = address;
    }
}
```

2. 在spring配置文件配置对象创建，配置属性注入

```xml
<bean id="orders" class="top.mnsx.springstudy.Orders">
  <constructor-arg name="oname" value="电脑"></constructor-arg>
  <constructor-arg name="address" value="China"></constructor-arg>
</bean>
```

* set方法注入简化——p名称空间注入

1. 使用p名称注入，可以简化xml配置方式

2. 添加p名称空间

```xml
<beans xmlns:p="http://www.springgramework.org/schema/p"></beans>
```

3. 进行属性注入，在bean标签里面进行操作

```xml
<bean id="book" class="top.mnsx.springstudy.Book" p:bname="九阳神功" p:bauthor="无名氏"></bean>
```

## IOC操作Bean管理（xml注入其他类型属性）

1. 注入属性——字面量
    1. null值

```xml
<property name="address">
  <null/>
</property>
```

 2. 属性值含特殊符号

```xml
<!--使用转移符号-->
<property name="&lt; address &gt;"></property>
<!--使用CDATA-->
<property name="address">
  <value><![CDATA[<<南京>>]]></value>
</property>
```

2. 注入属性——外部Bean

    1. 创建两个类service类和dao类

    2. 在service调用dao里面的方法

    3. 在Spring配置文件中配置

```java
public class UserService {
    private UserDao userDao;
    public void setUserDao(UserDao userDao){
        this.userDao = userDao;
    }
   	public void add(){
        System.out.println("service add...");
        userDao.update();
    }
}

public interface UserDao{
    public void update();
}

public class UserDaoIpml{
    public void update(){
        System.out.println("dao add...");
    }
}
<bean id="userService" class="top.mnsx.springstudy.UserService">
  <property name="userDao" ref="userDaoImpl"></property>
</bean>
<bean id="userDao" class="top.mnsx.springstudy.UserDaoImpl"></bean>
```

3. 注入属性——内部Bean

    1. 一对多关系：部门与员工

    2. 再实体类之间表示一对多的关系，员工表示所属部门，使用对象类型属性进行表示

```java
public class Dept {
    private String dname;
    public void setDname(String dname) {
        this.dname = dname;
    }
}

public class Emp{
    private String ename;
    private String gender;
    private Dept dept;
    public void setEname(String ename) {
        this.ename = ename;
    }
    public void setGender(String gender) {
        this.gender = gender;
    }
    public void setDept(String dept){
        this.dept = dept;
    }
}
<bean id="emp" class="top.mnsx.springstudy.Emp">
  <property name="ename" value="lucy"></property>
  <property name="gender" value="女"></property>
  <property name="dept">
    <bean id="dept" class="top.mnsx.springstudy.Dept">
      <property name="dname" value="安保科"></property>
    </bean>
  </property>
```

4. 注入属性——级联赋值

   第一种写法

```java
public class Dept {
    private String dname;
    public void setDname(String dname) {
        this.dname = dname;
    }
}

public class Emp{
    private String ename;
    private String gender;
    private Dept dept;
    public void setEname(String ename) {
        this.ename = ename;
    }
    public void setGender(String gender) {
        this.gender = gender;
    }
    public void setDept(String dept){
        this.dept = dept;
    }
}
<bean id="emp" class="top.mnsx.springstudy.Emp">
  <property name="ename" value="lucy"></property>
  <property name="gender" value="女"></property>
  <property name="dept" ref="dept"></property>
</bean>
<bean id="dept" class="top.mnsx.springstudy.Dept">
  	<property name="dname" value="安保科"></property>
</bean>
```

第二种写法

```java
public class Dept {
    private String dname;
    public void setDname(String dname) {
        this.dname = dname;
    }
    public String getDname(){
        return dname;
    }
}

public class Emp{
    private String ename;
    private String gender;
    private Dept dept;
    public void setEname(String ename) {
        this.ename = ename;
    }
    public void setGender(String gender) {
        this.gender = gender;
    }
    public void setDept(String dept){
        this.dept = dept;
    }
}
<bean id="emp" class="top.mnsx.springstudy.Emp">
  <property name="ename" value="lucy"></property>
  <property name="gender" value="女"></property>
  <property name="dept.dname" ref="dept"></property>
</bean>
```

## IOC操作Bean管理（xml注入集合属性）

1. 注入数组类型属性
2. 注入List集合类型属性
3. 注入Map集合类型属性
4. 注入Set集合类型属性

```java
public class Stu {
    private String[] courses;
    
    private List<String> list;
    
    private Map<String, String> map;
    
    private Set<String> set;
    
    public void setSet(Set<String> set){
        this.set = set;
    }
    
    public void setCourses(String[] courses){
        this.courses = courses;
    }
    
    public void setList(List<String> list){
        this.list = list;
    }
    
    public void setMap(Map<String, Strintg> maps) {
        this.map = map;
    }
}
```

```xml
<bean id="stu" class="top.mnsx.springstudy.Stu">
    <property name="courses">
        <array>
    		<value>Java</value>
        	<value>Mysql</value>
    	</array>
    </property>
	<property name="list">
    	<list>
        	<value>张三</value>
            <value>校长</value>
        </list>
    </property>
    <property name="map">
    	<map>
        	<entry key="Java" value="java"></entry>
            <entry key="Mysql" value="mysql"></entry>
        </map>
    </property>
    <property name="set">
    	<set>
        	<value>Java</value>
            <value>Mysql</value>
        </set>
    </property>
</bean>
```

4. 在集合里卖弄设置对象类型值

```java
public class Course {
    private String cname;
    public void setCname(String cname){
        this.cname = cname;
    }
}

public class Stu {    
    private List<Course> courseList;

    public void setCourseList(List<Course> courseList) {
        this.courseList = courseList;
    }
}
```

```xml
<bean id="stu" class="top.mnsx.springstudy.Stu">
    <property name="courseList">
    	<list>
        	<ref bean="courseList1"></ref>
            <ref bean="courseList2"></ref>
        </list>
    </property>
</bean>
<bean id="courseList1" class="top.mnsx.springstudy.Course">
    <property name="cname" value="Spring5框架"></peroperty>
</bean>
<bean id="courseList2" class="top.mnsx.springstudy.Course">
    <property name="cname" value="Spring5框架"></peroperty>
</bean>
```

4. 把集合注入部分提取出来

```java
public class Book{
    private List<String> list;
    public setList(List<String> list){
    	this.list = list;
    }
}
```

* 引入util名称空间

```xml
<util:list id="bookList">
	<value>Java核心技术</value>
    <value>Java编程思想</value>
</util:list>

<bean id="book" class="top.mnsx.springstudy.Book">
	<property name="list" ref="bookList"></property>
</bean>
```

## IOC操作Bean管理（FactoryBean）

1. Spring有两种bean，一种普通bean，一种工厂bean（FacortyBean）
2. 普通bean：在配置文件中定义bean类型就是返回类型
3. 工厂bean：在配置文件定义bean类型可以和返回类型不一样

第一步：

第二步：

```java
public class MyBean implements FactoryBean<Course>{
    @Override
    public Course getObject() throws Exception{
        Course course = new Course();
        course.setCname("abc");
        return course;
    }
}
```

```xml
<bean id="myBean" class="top.mnsx.springstudy.MyBean">

</bean>
```

## IOC操作Bean管理（bean的作用域）

1. 在spring里面，设置创建bean实例可以是单实例、也可以是多实例对象

2. Spring中默认设置对象为单实例对象
    1. singleton 默认值 表示是单实例对象
    2. prototype 表示是多实例对象

```xml
<bean id="book" class="top.mnsx.springstudy.Book" scope="prototype">
	<property name="list" ref="bookList"></property>
</bean>
```

3. singleton和prototype的区别
    1. singleton是单实例、prototype是多实例
    2. 设置singleton时，加载spring配置文件的时候创建单实例对象，设置prototype时，调用getBean方法时，创建多实例对象

## IOC操作Bean管理（bean的生命周期）

1. 生命周期
    * 从对象创建到对象销毁的 过程
2. bean声明周期
    1. 通过构造器创建bean实例（无参数构造）
    2. 为bean属性设置值和对其他bean引用（调用set方法）
    3. 调用bean的初始化方法（需要配置初始化方法）
    4. bean可以使用（对象获取）
    5. 当容器关闭时候，调用bean销毁的方法（需要进行配置销毁的方法）

```java
//使用ApplicationContext的子类调用close方法手动关闭bean
ApplicationContext app = new ClassPathXmlApplicationContext("bean.xml");
(ClassPathXmlApplicationContext) app.close();
```

```xml
<!-- bean中使用init-method可以配置初始化方法，destroy-method可以配置终结方法 ——>
```

3. bean的后置处理器，bean的生命周期变成7步
    1. 通过构造器创建bean实例（无参数构造）
    2. 为bean属性设置值和对其他bean引用（调用set方法）
    3. 把bean的实例传递bean后置处理器中的方法
    4. 调用bean的初始化方法（需要配置初始化方法）
    5. 把bean的实例传递bean后置处理器中的方法
    6. bean可以使用（对象获取）
    7. 当容器关闭时候，调用bean销毁的方法（需要进行配置销毁的方法）

```java
// 创建类，实现接口
public class MyBeanPost implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }
    
    @Nullable
    public Ojbect postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }
}
```

```xml
<bean id="myBeanPost" class="top.mnsx.springstudy.MyBeanPost"></bean>
```

## IOC操作Bean管理（xml自动装配）

1. 自动装配
    * 根据指定装配规则（属性名称或者属性类型），Spring自动匹配属性值进行注入

```xml
<!--
	bean标签属性autowired，配置自动装配
	autowire属性常用两个值——
	byName根据属性名称注入 注入值bean的id和类属性名称相同
	byType根据属性类型注入
-->
```

## IOC操作Bean管理（外部属性文件）

1. 直接配置数据库信息

```xml
<bean id="dataSourse" class="com.alibaba.druid.pool.DruidDataSource">
	<property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
    <property name="url" value="jdbc:mySql://localhost:3306/userDb"></property>
    <property name="username" value="root"></property>
    <property name="password" value="root"></property>
</bean>
```

2. 引入外部属性文件

```properties
driver=com.mysql.jdbc.Driver
url=jdbc:mySql://localhost:3306/userDb"></property>
username=root
password=root
```

```xml
<!-- 配置context名称空间 -->
<context:property-placehoder location="classpath:jdbc.properties"/>
<bean id="dataSourse" class="com.alibaba.druid.pool.DruidDataSource">
	<property name="driverClassName" value="${jdbc.driver}"></property>
    <property name="url" value="${jdbc.url}"></property>
    <property name="username" value="${jdbc.username}"></property>
    <property name="password" value="${jdbc.password}"></property>
</bean>
```

## IOC操作Bean管理（基于注解方式）

1. 注解
    * 代码黎一些特殊的标记，格式：@注解名称（属性名称=属性值，属性名称=属性值...）
    * 使用注解，注解作用在类上，方法上，属性上
    * 使用目的：简化xml配置
2. Spring针对Bean管理中的创建对象提供注解
    1. @Component
    2. @Service
    3. @Controller
    4. @Repository

## IOC操作Bean管理（组件扫描配置）

1. 如果扫描多个包，多个包使用逗号隔开
2. 扫描包上层目录

```xml
<context:component-scan base-package="top.mnsx" user-default-filters="false">
	<context:include-filter type="annotation" expression="org.springframeword.stereotype.Controller"></context:include-filter>
</context:component-scan>
```

```xml
<context:component-scan base-package="top.mnsx">
	<context:exclude-filter type="annotation" expression="org.springframeword.stereotype.Controller"></context:include-filter>
</context:component-scan>
```

## IOC操作Bean管理（注入属性）

1. @Autowired —— 根据属性类型自动装配

   第一步：把Service和Dao对象创建，在Service和Dao中添加对象创建的注解

   第二步：在Service注入Dao对象，在Service类添加Dao类型属性，在属性上面使用注解

```java
public interface UserDao {
	public void add();
}

@Repostory
public class UserDaoImpl implements UserDao {
    public void add(){
        System.out.println("dao add...");
    }
}

public interface UserService {
    public void add();
}

@Service
public class UserServiceImpl implements UserService {
    @Autowired
    private UserDao userDao;

    public void add(){
        userDao.add();
        System.out.println("service add...");
    }
}
```

2. @Qualifier —— 根据属性名称注入

   @Qualifier必须要与@Autowired一起使用

```java
public interface UserDao {
	public void add();
}

@Repostory("userDao")
public class UserDaoImpl implements UserDao {
    public void add(){
        System.out.println("dao add...");
    }
}

public interface UserService {
    public void add();
}

@Service
public class UserServiceImpl implements UserService {
    @Autowired
    @Qualifier("userDao")
    private UserDao userDao;

    public void add(){
        userDao.add();
        System.out.println("service add...");
    }
}
```

3. @Resource —— 可以根据类型也可以根据名称注入

```java
public interface UserDao {
	public void add();
}

@Repostory("userDao")
public class UserDaoImpl implements UserDao {
    public void add(){
        System.out.println("dao add...");
    }
}

public interface UserService {
    public void add();
}

@Service
public class UserServiceImpl implements UserService {
    @Resource //属性注入
    @Resource(name = "userDao") //名称注入
    private UserDao userDao;

    public void add(){
        userDao.add();
        System.out.println("service add...");
    }
}
```

4. @Value —— 普通属性注入

```java
public interface UserService {
    public void add();
}

@Service
public class UserServiceImpl implements UserService {
	
    @Value(value = "hello");
    private String val;

    public void add(){
        System.out.println(val);
        System.out.println("service add...");
    }
}
```

## IOC操作Bean管理（完全注解开发）

1. 创建配置类，替代xml配置文件

```java
@Configuration //将当前类声明为配置类
@ComponentScan(backsePackage = "top.mnsx")
public class SpringConfig {
    
}
```

2. 编写测试类

```java
@Test
public void testService(){
    ApplicaitonContext context = new AnnotationConfigApplicationContext(SpringConfig.class);
    UserService userService = context.getBean("userService", UserService.class);
    System.out.println(userService);
    userService.add();
}
```

# AOP

## AOP（概念）

1. 面向切面编程（方面），利用AOP可以对业务逻辑得各个部分进行隔离，从而使得业务逻辑各部分之间得耦合度降低，提高程序得可重用性，同时提高了开发得效率
2. 不通过修改代码得方式，在主干功能里添加功能

## AOP（底层原理）

AOP底层使用动态代理

第一种 有接口得情况，使用JDK代理。创建接口实现类代理对象，增强类的方法

第二种 没有接口的情况，使用CGLIB动态代理。创建子类的代理对象，增强类的方法

## AOP（JDK动态代理）

1. 调用newProxyInstance方法
    * ClassLoader 类加载器
    * Class 增强方法所在类，这个类实现的接口，支持多个接口
    * 实现这个接口InvocationHandler，创建对象，写增强部分

2. 编写JDK动态代理

```java
public interface UserDao {
    public int add(int a, int b);
    
    public String update(String id);
}

public class UserDaoImpl implements UserDao{
	public int add(int a, int b){
        return a + b;
    }
    
    public String update(String id){
        return id;
    }
}

public class JDKProxy {
    public static void main(String[] args){
        Class[] interfaces = {UserDao.class};
        UserDao dao = (UserDao)Proxy.newProxyInstance(JDKProxy.class.getClassLoader, intefaces, new MyInvocationHandler());
        int result = dao.add(1, 2);
        System.out.println("result:" + result)
    }
}

public class MyIvocationHandler(){
    private UserDao userDao = new UserDaoImpl();
    
    public Object invoke(Object proxy, Mehtod, method, Object[] args) throws Throwable{
        		//可以通过Mehtod.getName判断使用的方法名
                System.out.println("方法执行之前...");
                
                Object res = method.invoke(userDao, args);
                
                System.out.println("方法执行之后...");
    }
}
```

## AOP(术语)

1. 连接点——类里可以被增强的方法

2. 切入点——实际被增强的方法就是切入点

3. 通知（增强）——实际增强的部分
    1. 前置通知——方法前执行
    2. 后置通知——方法后执行
    3. 环绕通知——方法前后都会执行
    4. 异常通知——报异常后通知
    5. 最终通知——finally，无论如何，结束时都会通知
4. 切面——切入点（切点） + 通知（增强）

## AOP操作（准备）

1. Spring框架一般基于AspectJ实现AOP操作

   AspectJ不是Spring的一部分，一般一起使用，而AspectJ是独立的AOP框架

2. 基于AspectJ实现AOP操作

    1. 基于XML配置文件实现
    2. 机遇与注解方式实现

3. 切入点表达式

   切入点表达式作用：知道对那个类的那个方法进行加强

   语法结构——execution(\[权限修饰符]\[返回值类型]\[类全路径]\[方法名称](\[参数列表]))

## AOP操作（AspectJ注解）

1. 创建类，在类里面定义方法

```java
public class User {
	public void add(){
        System.out.println("add...");
    }
}
```

2. 创建增强类（编写增强逻辑)

   在增强类里面，创建方法，让不同方法代表不同通知类型

```java
public class UserProxy {
	public void before() {
        System.out.println("before...");
    }
}
```

3. 进行通知的配置
    1. 在Spring的配置文件中，开启注解扫描
    2. 使用注解创建User和UserProxy对象
    3. 在增强类上面添加注释@Aspect
    4. 在spring配置文件中开启生成代理对象

4. 配置不同类型的通知

```java
@Component
@Aspect
public class UserProxy {
    @Before(value = "execution(* top.mnsx.springstudy.User.add(..))")
    public void before(){
        System.out.println("before...");
    }
    
    @After(value = "execution(* top.mnsx.springstudy.User.add(..))")
    public void after(){
        System.out.println("after...");
    }
    
    @AfterReturning(value = "execution(* top.mnsx.springstudy.User.add(..))")
    public void afterReturning(){
        System.out.println("afterReturning...");
    }
    
    @AfterThrowing(value = "execution(* top.mnsx.springstudy.User.add(..))")
    public void before(){
        int i = 1 / 0;
        System.out.println("afterThrowing...");
    }
    
    @Around(value = "execution(* top.mnsx.springstudy.User.add(..))")
    public void before(ProceedingJoinPoint proceedingJoinPoint) throws Throwable{
        System.out.println("around...");
        proceedingJoinPoint.proceed();
        System.out.println("after...");
    }
}
```

5. 相同的切入点抽取

```java
@Pointcut(value = "execution(* top.mnsx.sprintstudy.User.add(..))")
public void pointdemo() {
    
}

@Before(value = "pointdemo()")
public void before() {
    System.out.println("before...");
}
```

6. 有多个增强类对同一个方法进行增强，设置增强优先级

   在增强类上面添加注解@Order（数字类型值），数字类型值越小优先级越高

```java
@Component
@Aspect
@Order(1)
public class PersonProxy
```

## AOP(ApectJ配置文件)

```java
public class Book{
    public void buy(){
        System.our.println("buy...");
    }
}

public class BookProxy {
    public void before() {
        System.out.println("before...");
    }
}
```

```xml
<bean id="book" class="top.mnsx.springstudy.Book"></bean>
<bean id="bookProxy" class="top.mnsx.springstudy.BookProxy"></bean>
<aop:config>
	<aop:pointcut id="p" expression="execution(* top.mnsx.springstudy.Book.buy(..))"/>
    <aop:aspect ref="bookProxy">
    	<aop:before methos="before" pointcut-ref="p"/>
    </aop:aspect>
</aop:config>
```

**全注解开发**

```java
@Configuration
@componentScan(basePackages = {"top.mnsx"})
@EnableAspectJAutoProxy(proxyTargetClass = true)
public class ConfigAop {
    
}
```

# JDBCTemplate

Spring框架对JDBC进行封装，使用JDBCTemplate方便实现对数据库操作

```xml
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" destroy-method="close">
      <property name="url" value="jdbc:mysql:///user_db"/>
      <property name="username" value="root"/>
      <property name="password" value="123123"/>
      <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
</bean>

<bean id="jdbcTemplate" class="org.Springframeword.jdbc.core.JdbcTemplate">
	<property name="dataSource" ref="dataSource"/>
</bean>
```

```java
@Service
public class BookService {
    @Autowired
    private BookDao bookDao;
}

@Repository
public class BookDaoImpl implements BookDao{
    @Autowried
    private JdbcTemplate jdbcTemplate;
    
    @Override
    public void add(Book book){
        String sql = "insert into t_book values(?, ?, ?)";
        jdbcTemplate.update(sql, book.getId(), book.getUsername, book.getUStatus);
        System.out.println(update);
    }

    @Override
    public void updateBook(Book book){
        String sql = "update t_book set username = ?, uStatus = ? where user_id = ?";
        Object[] args = {book.getUserName(), book.getUStatus(), book.getUserId()};
        int update = jdbcTemplate.update(sql, args);
        System.out.println(update);
    }
    
    @Override
    public void deleteBook(String id){
    	String sql = "delete from t_book where user_id = ?";
        int update = jdbcTemplate, update(sql, id);
        System.out.println(update);
    }
    
    @Override
    public void findCount(){
        String sql = "select count(*) from t_book";
        int count = jdbcTmemplate.queryForObject(sql, Integer.class);
        return count
    }
    
    @Override
    public Book findOne(String id){
        String sql = "select * from t_book where user_id = ?";
        Book book = jdbcTemplate.queryForObject(sql, new BeanPropertyRowMapper<Book>(Book.class), id);
        return book;
    }
    
    @Override
    public List<Book> findAllBook(){
        String sql = "select * from t_book";
        List<Book> bookList = jdbcTemplate.query(sql, new BeanPropertyRowMapper<Book>(Book.class));
        return bookList;
    }
    
    @Override
    public int[] batchAdd(List<Object[]>) batchArgs){
        String sql = "insert into t_book values(?, ?, ?)";
        int[] ints = jdbcTemplate.batchUpdate(sql, batchArgs);
        return ints;
    }
    
    @Override
    public int[] batchUpdate(List<Object[]> batchArgs){
        String sql = "update t_book set username = ?, uStatus = ? where user_id = ?";
        int[] ints = jdbcTemplate.batchUpdate(sql, batchArgs);
        return ints;
    }
    
    @Override
    public int[] batchDelete(List<Object[]> batchArgs){
        String sql = "delete from t_book where user_id = ?";
        int[] ints = jdbcTemplate.batchUpdate(sql, batchArgs);
        return ints;
    }
}

public interface BookDao {
    public void addBook(Book book);
    
    public void updateBook(Book book);
    
    public void deleteBook(Stirng id);
    
    public int findCount();
    
    public Book findOne();
    
    public List<Book> findAllBook(); 
    
    public int[] batchAdd(List<Object[]> batchArgs);
    
    public int[] batchUpdate(List<Object[]> bathcArgs);
    
    public int[] batchDelete(List<Object[]> bathcArgs);
}
```

```java
@Data
public class User {
    private String userId;
    private String username;
    private String uStatus
}
```

# Spring事务管理

## 事务概念

事务是数据库操作最基本单元，逻辑上一组操作，要么成功，如果有一个失败，所有操作都会失败

1. 事务四大特性（ACID）
    1. 原子性——过程中不可分割，要成成功要么失败
    2. 一致性——开始和结束，总量不变
    3. 隔离性——多事务操作，事务之间不会有影响
    4. 持久性——事务提交后，表中数据发生变化

```xml
<context:component-scan base-package="top.mnsx"></context:component-scan>

<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" destroy-method="close">
    <property name="url" value="jdbc:mysql:///user_db"/>
    <property name="username" value="root"/>
    <property name="password" value-"123123"/>
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
</bean>

<bean id="jdbcTemplate" class="org.springframeword.jdbc.core.JdbcTemplate">
	<property name="dataSource" ref="dataSource"></property>
</bean>
```

```java
@Service
public class UserService {
    @Autowired
    private UserDao userDao;
    
    public void accountMoney() {
        userDao.reduceMoney();
        
        userDao.addMoney();
    }
}

@Repository
public class UserDao {
    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    @Override
    public void reduceMoney() {
        String sql = "update t_account set money = money - ? where username = ?";
        jdbcTemplate.update(sql, 100, "lucy");
    }
    
    @Override
    public void addMoney() {
        String sql = "update t_account set money = money - ? where username = ?";
        jdbcTemplate.update(sql, 100, "marry");
    }
}

public class TestBook {
    @Test
    public void testAccount() {
        ApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");
        UserService userService = context.getBean("userService", UserService.class);
        userService.accountMoney();
    }
}
```

## 事务操作（Spring事务管理介绍）

1. 事务添加到JavaEE三层结构里Service层（业务逻辑层）

2. 在Spring中进行事务管理操作
    1. 编程式事务管理——臃肿、冗余
    2. 声明式事务管理
3. 声明式事务管理
    1. 基于注解方式
    2. 基于xml配置文件
4. 在Spring进行声明式事务管理，底层使用AOP原理
5. Spring事务管理API
    * 提供一个接口，代表事务管理器，这个接口针对不同的框架提供了不同的实现类

## 事务操作（注解方式事务管理）

1. 在Spring配置文件中配置事务管理器

```xml
<bean id="transactionManager" class="org.springframeword.jdbc.datasource.DataSourceTransactionManager">
	<property name="dataSource" ref="dataSOurce"></property>
</bean>
```

2. 在Spring配置文件中，开启事务注释

```xml
//引入名称空间tx
<tx:annotation-driven manager="transactionManager"></tx:annotation-driven>
```

3. 在Service类上面加一个@Transaction注解

```java
@Transactional
@Service
public class UserService {
    @Autowired
    private UserDao userDao;
    
    public void accountMoney() {
        userDao.reduceMoney();
        
        userDao.addMoney();
    }
}
```

* 如果这个注解加到类上面，所有的方法都将被事务管理
* 如果加到方法上，该方法被事务管理

## 事务操作（声明式事务管理参数配置）

1. 在Service类上添加注解@Transactional，在这个注解里面可以配置事务的相关属性

2. propagation：事务传播行为

    * 多事务方法直接进行调用，这个过程事务是如何进行管理的

    * Spring框架事务传播行为有7种

        * REQUIRED：如果add本身有事务，调用update方法后，也是使用add方法黎面的事务，如果add本身没有事务，就会建一个新的事务

        * REQUIRED_NEW：使用add方法调用update方法，如果add无论是否有事务，都创建新的事务

          ![image-20220411144241700](..\Picture\Spring\事务传播行为.png)

3. ioslation：事务隔离级别

    * 事务有一个特性叫隔离性，多事务之间不会产生影响，不考虑隔离性会产生很多问题

    * 有三个读问题：脏读、不可重复读、虚（幻）读

        * 脏读：一个未提交的事务读取到另一个未提交事务的数据
        * 不可重复读：一个未提交的事务读取到另一个提交的事务的数据
        * 虚读：一个未提交事务读取到一个提交事务新添加的数据

    * 通过设置事务隔离性，来解决问题

      ![事务隔离级别](..\Picture\Spring\事务隔离级别.png)

4. timeout：超时时间
    * 事务需要在一定时间内进行提交，如果不提交进行回滚
    * 默认值为-1，设置时间以秒为单位进行计算

5. readOnly：是否只读
    * 读：查询操作， 写：添加修改删除操作
    * 默认值为false 表示可以查询，可以添加修改删除操作
    * 修改为true，不能进行写操作

6. rollbackFor：回滚
    * 设置出现了那些异常回滚

7. noRollbackFor：不回滚
    * 设置出现了那些异常不回滚

## 事务操作（XML声明式事务管理）

1. 创建事务管理器

```xml
<bean id="transactionManaget" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	<property name="dataSource" ref="dataSource"></property>
</bean>
```

2. 配置通知

```xml
<tx:advice id="txAdvice">
	<tx:attributes>
    	<tx:methods name="accountMoney" propagation="REQUIED"/>
        <tx:methods name="account*"/></tx:methods>
    </tx:attributes>
</tx:advice>
```

3. 配置切入点和切面

```xml
<aop:config>
	<aop:pointcout id="pt" expression="execution(* top.mnsx.springstudy.UserService.*(..))"/>
    <aop:advisor advice-ref="txAdvice" pointcout-ref="pt"></aop:advisor>
</aop:config>
```

## 事务操作（完全注解）

1. 创建配置类，替代Spring配置文件

```java
@Configuration
@ComponentScan(basePackage="top.mnsx")
@EnableTransactionManagement
public class TxConfig {
	@Bean
    pbulic DruidDataSource getDruidDataSource(){
        DuridDataSource dataSource = new DruidDataSource();
        dataSource.setDriverClassName("top.mnsx.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql:///user_db");
        dataSource.setUsername("root");
        dataSource.setPassword("123123");
        return dataSource;
    }
    
    @Bean
    public JdbcTemplate getJdbcTemplate(DataSource dataSource){
        JdbcTemplate jdbcTemplate = new JdbcTemplate();
        jdbc.setDataSource(dataSource);
        return jdbcTemplate;
    }
    
    @Bean
    public DataSourceTransactionManager getDataSoueceTransactionManager(){
        DataSourceTransactionManager dataSourceTransactionManager = new DataSourceTransactionManager();
        return dataSourceTransactionManager;
    }
}
```

# Spring5

## 整合日志框架

1. 整个Spring5框架代码基于Java8，运行时兼容JDK9，许多不建议使用的类和方法在代码库种被删除
2. Spring5.0框架自带通用的日志封装
    1. Spring5已经移除Log4jConfigListener， 官方建议使用Log4j2
    2. Spring5框架整合Log4j2

## Nullable注解

1. @Nullable注解可以使用在方法上面，属性上面，参数上面，表示方法返回可以为空，属性值可以为空，参数可以为空
2. 注解用在方法上，方法返回值可以为空

```java
@Nullable
String getId();
```

3. 注解使用在方法参数里，方法参数可以为空

```java
public <T> void registerBean(@Nullable String beanName){
    this.reader.registerBean(beanClass, beanName, supplication);
}
```

4. 注解使用在属性上面，属性可以为空

```java
@Nullable
private String bookName;
```

## 函数式风格 GenericApplicationContext

```java
@Test
public void testGenericApplicationContext() {
    //1. 创建GenericApplicationContext对象
    GenericApplicationContext context = new GenericApplicationContext();
    //2. 调用context的方法对象注册
    context.refresh();
    context.registerBean(User.class, ()-> new User());
    //3. 获取在Spring注册的对象
    context.getBean("user");
}
```

## Webflux——基本概念

1. Spring5新添加的模块，用于web开发，功能SpringMVC类似，Webflux使用当前比较流行的响应式编程

2. 使用传统web框架，比如SpringMVC，这些基于Servlet容器，Webflux是一种异步非阻塞的框架，异步非阻塞的框架，异步非阻塞的框架在Servlet3.1以后才支持，核心是基于Reactor的相关API实现的。

3. 异步非阻塞

    * 异步和同步

    * 非阻塞和阻塞

      针对的对象不同

      异步同步针对调用者，调用者发送请求，如果要等对方回应后做其他的事情就是同步，如果发送请求之后不等着对方回应就去做其他的事情就是异步

      阻塞和非阻塞针对被调用者，被调用者收到请求后，昨晚请求任务后才给出反馈就是阻塞，收到请求之手马上给出反馈然后就去做其他事情就是非阻塞

4. webflux特点

    1. 非阻塞式：在有限资源下，提高系统吞吐量和伸缩性，以Reacrot为基础实现响应式编程
    2. 函数式：Spring5框架基于java8框架，Webflux使用java8函数式编程方式实现路由请求

5. 比较SpringMVC和SpringWebFlux

   ![image-20220411225144683](..\Picture\Spring\image-20220411225144683.png)

    * 两个框架都可以使用注解方式，都运行在tomcat等容器中
    * SpringMVC采用命令式编程，Webflux采用异步响应式编程

## Webflux——响应式编程

响应式编程是一种面向数据流和变化传播的编程范式。这意味着可以在编程语言中很方便地表达静态或动态的数据流，而相关的计算模型会自动将变化的值通过数据流进行传播

Java8及其之前版本

提供的观察者模式两个类Observer和Observable

```java
public class ObserverDemo extends Observable {

    public static void main(String[] args) {
        ObserverDemo observerDemo = new ObserverDemo();
        //添加一个观察者
        observerDemo.addObserver((o, arg) -> {
            System.out.println("发送变化");
        });
        observerDemo.addObserver((o, arg) -> {
            System.out.println("手动被观察者通知，准备改变");
        });
        observerDemo.setChanged(); //监控到数据变化
        observerDemo.notifyObservers(); //通知
    }
}
```

**基于Reactor实现**

1. 响应式编程操作中，满足Reactor式满足Reactive规范框架
2. Reactor有两个核心类，Mono和Flux，这两个类实现了接口Publisher,提供了丰富操作符。Flux对象实现发布者，返回N个元素，Mono实现发布者，返回0或者1个元素
3. Flux和Mono都是数据流的发布者，使用Flux和Mono可以发出三种数据信号：元素值、错误信号、完成信号都代表终止信号，终止信号用户告诉订阅者数据流结束了，错误信号终止数据流同时把错误信息传递给订阅者

```xml
        <dependency>
            <groupId>io.projectreactor</groupId>
            <artifactId>reactor-core</artifactId>
            <version>3.3.14.RELEASE</version>
        </dependency>
```

```java
public class TestReactor {
    public static void main(String[] args) {
        //just方法直接声明
        Flux.just(1, 2, 3, 4);
        Mono.just(1);

        //其他方法
        Integer[] arr = {1, 2, 3, 4};
        Flux.fromArray(arr);

        List<Integer> list = Arrays.asList(arr);
        Flux.fromIterable(list);

        Stream<Integer> stream = list.stream();
        Flux.fromStream(stream);
    }
}
```

4. 三种信号特点
    * 错误信号和完成信号都是终止信号，不能共存的
    * 如果没有发送任何元素值，而是直接发送错误或者完成信号，表示空数据流
    * 如果没有错误信号，没有完成信号，表示是无限数据流

5. 调用just或者其他方法只是声明了数据流，数据流并没有发出，只是进行订阅之后才会触发数据流，不订阅什么都不会发生的

```java
        Flux.just(1, 2, 3, 4).subscribe(System.out::print);
        Mono.just(1).subscribe(System.out::print);
```

7. 操作符——对数据进行一道道操作成为操作符，比如工厂流水线
    * map 元素映射成新的元素
    * flatMap 元素映射成新的数据流——把每个元素转换成流，在把所有流合成大流

## SpringWebflux执行流程和核心API

SpringWebFlux基于Reactor，默认使用容器是Netty，Netty是高性能的NIO框架，异步非阻塞的框架

1. SpringWebFlux执行流程和SpringMvc相似

   SpringWebflux核心控制器DispatchHandler实现接口WebHandler

```java
 Mono<Void> handle(ServerWebExchange exchange);
```

2. SpringWebFlux里面DispatcherHandler，负责请求的处理
    * HanddlerMapping——请求查询到处理的方法
    * HandlerAdapter——真正负责请求处理
    * HandlerResultHandler——响应结果处理
3. SpringWebFlux是心啊函数式编程，两个接口：RouterFunction(路由处理)和HandlerFunction(处理函数)

## SpringWebFulx（基于注解编程模型）

使用注解编程模型方式，和之前SpringMVC使用相似的，只需要把相关依赖配置到项目中，Springboot自动配置相关运行容器，默认情况使用Netty容器

1. 引入SpringFlux依赖

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-webflux</artifactId>
        </dependency>
```

2. 配置启动的端口号

```yaml
server:
  port: 8081
```

3. 创建实体类

```java
@Data
@AllArgsConstructor
public class User {
    private String username;
    private String gender;
    private Integer age;
}
```

4. 创建接口定义操作的方法

```java
public interface UserService {
    //根据Id查询
    Mono<User> getUserById(int id);

    //查询所有
    Flux<User> getAllUser();

    //添加用户
    Mono<Void> saveUserInfo(Mono<User> user);
}
```

5. 创建类实现了接口

```java
@Service
public class UserServiceImpl implements UserService {

    //创建map集合存储数据
    private final Map<Integer, User> users = new HashMap<>();

    public UserServiceImpl () {
        this.users.put(1, new User("lucy", "nan", 20));
        this.users.put(2, new User("mary", "nv", 30));
        this.users.put(3, new User("jack", "nv", 50));
    }

    @Override
    public Mono<User> getUserById(int id) {
        return Mono.justOrEmpty(this.users.get(id));
    }

    @Override
    public Flux<User> getAllUser() {
        return Flux.fromIterable(this.users.values());
    }

    @Override
    public Mono<Void> saveUserInfo(Mono<User> user) {
        return user.doOnNext(person -> {
            //向map集合里面放值
            int id = users.size() + 1;
            users.put(id, person);
        }).thenEmpty(Mono.empty());
    }
}
```

**说明**

SpringMvc方式实现，同步阻塞的方式，基于SpringMVC + Servlet + Tomcat

SpringWebFlux方式实现，异步非阻塞方式，基于SpringWebFlux + Reactor + Netty

## SpringWebflux（基于函数式编程模型）

1. 在使用函数式编程模型操作时候，需要自己初始化服务器
2. 基于函数式编程模式的时候有两个核心接口：RouterFunction（实现路由功能，请求转发给对应的handler）和HandlerFucntion（处理请求生成响应的数据）。核心任务定义两个函数式接口的实现并且启动需要的服务器

3. SpringWebflux请求和响应不再是ServletRequest和ServletResponse，而是serverRequest和ServerResponse

```java
public class UserHandler {
    private final UserService userService;
    public UserHandler(UserService userService){
        this.userService = userService;
    }

    //根据id查询
    public Mono<ServerResponse> getUserById(ServerRequest request) {
        //获取id
        int id = Integer.parseInt(request.pathVariable("id"));
        //空值处理
        Mono<ServerResponse> notFound = ServerResponse.notFound().build();
        //调用service方法获取数据
        Mono<User> userById = this.userService.getUserById(id);
        //把serById转换成流返回，使用Reactor操作符flatMap
        return userById.flatMap(person -> ServerResponse.ok().
                contentType(MediaType.APPLICATION_JSON).
                body(fromObject(person)).
                switchIfEmpty(notFound));
    }

    //查询所有
    public Mono<ServerResponse> getAllUsers() {
        //调用service得到结果
        Flux<User> users = this.userService.getAllUser();
        return ServerResponse.ok().contentType(MediaType.APPLICATION_JSON).body(users, User.class);
    }

    //添加
    public Mono<ServerResponse> saveUser(ServerRequest request) {
        Mono<User> userMono = request.bodyToMono(User.class);
        return ServerResponse.ok().build(this.userService.saveUserInfo(userMono));
    }
}

public class Server {

    //1 创建Router路由
    public RouterFunction<ServerResponse> routerFunction() {
        UserService userService = new UserServiceImpl();
        UserHandler handler = new UserHandler(userService);
        //设置路由
        return RouterFunctions.route(
                GET("/users/{id}").and(accept(APPLICATION_JSON)), handler::getUserById)
                .andRoute(GET("/users").and(accept(APPLICATION_JSON)), handler::getAllUsers);
    }

    //2 创建服务器完成适配
    public void createReactorServer() {
        RouterFunction<ServerResponse> route = routerFunction();
        HttpHandler httpHandler = toHttpHandler(route);
        ReactorHttpHandlerAdapter adapter = new ReactorHttpHandlerAdapter(httpHandler);

        //创建服务器
        HttpServer httpServer = HttpServer.create();
        httpServer.handle(adapter).bindNow();
    }
    
    public static void main(String[] args) throws IOException {
        Server server = new Server();
        server.createReactorServer();
        System.out.println("enter to exit");
        System.in.read();
    }
}
```

