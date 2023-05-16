# Spring和SpringBoot

## 为什么使用SpringBoot

> Spring Boot makes it easy to create stand-alone, production-grade Spring based Applications that you can "just run".
>
>
>
> 能快速创建出生产级别的Spring应用

## SpringBoot优点

- Create stand-alone Spring applications

-
    - 创建独立Spring应用

- Embed Tomcat, Jetty or Undertow directly (no need to deploy WAR files)

-
    - 内嵌web服务器

- Provide opinionated 'starter' dependencies to simplify your build configuration

-
    - 自动starter依赖，简化构建配置

- Automatically configure Spring and 3rd party libraries whenever possible

-
    - 自动配置Spring以及第三方功能

- Provide production-ready features such as metrics, health checks, and externalized configuration

-
    - 提供生产级别的监控、健康检查及外部化配置

- Absolutely no code generation and no requirement for XML configuration

-
    - 无代码生成、无需编写XML

> SpringBoot是整合Spring技术栈的一站式框架
>
> SpringBoot是简化Spring技术栈的快速开发脚手架

## SpringBoot缺点

* 人称版本帝，迭代快需要时刻关注变化
* 封装太深，内部原理复杂，不容易精通

## 微服务

* 微服务是一种架构风格
* 一个应用拆分为一组小型服务
* 每个服务运行在自己的进程内，也就是可独立部署和升级

- 服务之间使用轻量级HTTP交互
- 服务围绕业务功能拆分
- 可以由全自动部署机制独立部署
- 去中心化，服务自治。服务可以使用不同的语言、不同的存储技术

## 分布式

![img](Picture\分布式模型.png)

**分布式的解决**

SpringBoot + SpringCloud

![img](Picture\Spring微服务解决.png)

## 云原生

原生应用如何上云。Cloud Native

# SpringBoot2入门

## 修改Maven镜像

```xml
<mirrors>
      <mirror>
        <id>nexus-aliyun</id>
        <mirrorOf>central</mirrorOf>
        <name>Nexus aliyun</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public</url>
      </mirror>
  </mirrors>
 
  <profiles>
         <profile>
              <id>jdk-1.8</id>
              <activation>
                <activeByDefault>true</activeByDefault>
                <jdk>1.8</jdk>
              </activation>
              <properties>
                <maven.compiler.source>1.8</maven.compiler.source>
                <maven.compiler.target>1.8</maven.compiler.target>
                <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
              </properties>
         </profile>
  </profiles>
```

## 创建Maven工程

## 引入依赖

```xml
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.4.RELEASE</version>
    </parent>


    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

    </dependencies>
```

## 创建主程序

```java
/**
 * 主程序类
 * @SpringBootApplication: 这是一个SpringBoot应用
 */
@SpringBootApplication
public class Boot01HelloworldApplication {

    public static void main(String[] args) {
        SpringApplication.run(Boot01HelloworldApplication.class, args);
    }

}
```

## 编写业务

```java
//@Controller
//@ResponseBody

@RestController
public class HelloController {

    @RequestMapping("/hello")
    public String handle01() {
        return "Hello Spring Boot!";
    }
}
```

## 测试

直接进入main方法运行

## 简化配置

```properties
server.port = 8888
```

## 简化部署

```xml
	<build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```

将项目打成jar包，直接在目标服务器执行即可

# 自动配置原理

## 依赖管理

```xml
<!--本地项目的父项目-->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.4.RELEASE</version>
</parent>

<!--本地项目的父项目的父项目-->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.3.4.RELEASE</version>
</parent>
```

**几乎声明了所有开发中常用的依赖的版本号,自动版本仲裁机制**

* 开发导入starter场景启动器

    * spring-boot-starter-\*
    * 只要引入starter，这个场景的所有常规依赖，就会自动引入
    * 通过Spring-boot官方文档可以查看官方提供的场景启动器
    * \*-spring-boot-start，是第三方提供的场景启动器

  ```xml
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-web</artifactId>
          </dependency>
  ```

* 无需关注版本号，自动仲裁

    * 使用非版本仲裁的依赖时需要自己写入引入版本号

* 可以修改版本号

    * 查看spring-boot-dependencies里面规定的当前依赖的版本
    * 在当前项目里面重写配置

  ```xml
  <!--重写案例-->
  <properties>
  	<mysql.version>5.1.43</mysql.version>
  </properties>
  ```

## 自动配置

* 自动配置Tomcat

    * 引入Tomcat依赖
    * 配置Tomcat

  ```xml
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-tomcat</artifactId>
        <version>2.6.7</version>
        <scope>compile</scope>
      </dependency>
  ```

* 自动配置Spring

    * 引入SpringMVC全套组件
    * 自动配置SpringMVC常用组件

  ```xml
      <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
        <version>5.3.19</version>
        <scope>compile</scope>
      </dependency>
      <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>5.3.19</version>
        <scope>compile</scope>
      </dependency>
  ```

    * SpringBoot配置好了所有Web开发常见场景

    * 默认包结构

        * 主程序所在包以及其下所有子包里面的组件都会默认扫描进来

        * 无需配置包扫描

        * 想要该表扫描路径——

          `@SpringBootApplication(scanBasePackages="top.mnsx")`

          `@ComponentScan 指定扫描路径`

          ```java
          @SpringBootApplication(scanBasePackages="top.mnsx")
          等同于
          @SpringBootConfiguration
          @EnableAutoConfiguration
          @ComponentScan("top.mnsx")
          ```

    * 各种配置拥有默认值

        * 默认配置最终都是映射到对应的properties文件中
        * 配置文件的值最中会绑定每个类上，这个类会在容器中创建对象

    * 按需加载所有自动配置项

        * 非常多的starter
        * 引入了场景，这个场景的自动配置才会开启
        * SpringBoot所有的自动配置功能都在`spring-boot-autoconfigure`包里面

## 容器功能

1. 组件添加

    * @Configuration

      Full模式和Lite模式

      **最佳实战**

        * 配置 类组件之间无依赖关系用Lite模式加速容器启动过程，减少判断
        * 配置类组件之间有依赖关系，方法会被调用得到之前单实例组件，用Full模式

      ```java
      /**
       * 配置类里面使用@Bean标注在方法上给容器注册主键，默认也是单实例的
       * 配置类本身也是组件
       * proxyBeanMethods：代理bean方法
       * Full(proxyBeanMethods = true，
       * Lite(proxyBeanMethods = false
       */
      @Configuration(proxyBeanMethods = true) // 告诉SpringBoot这是一个配置类 == 配置文件
      public class MyConfig {
      
          /**
           * 外部无论对配置类中的这个组件注册方法调用多少次获取的都是之前注册容器中的单实例对象
           */
          @Bean // 给容器添加组件。以方法名作为组件的id，返回类型就是组件类型，返回的值就是组件在容器中的实例
          public User user01() {
              User user = new User("mnsx", 20);
              user.setPet(tomcatPet());
              return user;
          }
      
          @Bean("pet01")
          public Pet tomcatPet() {
              return new Pet("tomcat");
          }
      }
      ```

      ```java
      /**
       * 主程序类
       * @SpringBootApplication: 这是一个SpringBoot应用
       */
      //@ComponentScan("")
      //@SpringBootApplication(scanBasePackages="top.mnsx")
      @SpringBootConfiguration
      @EnableAutoConfiguration
      @ComponentScan("top.mnsx")
      public class Boot01HelloworldApplication {
      
          public static void main(String[] args) {
              // 1.返回IOC容器
              ConfigurableApplicationContext run = SpringApplication.run(Boot01HelloworldApplication.class, args);
      
              // 2.查看容器里的组件
              String[] names = run.getBeanDefinitionNames();
              for (String name : names) {
                  System.out.println(name);
      
              }
              
              // 3.从容器中获取组件
      
              Pet pet01 = run.getBean("pet01", Pet.class);
              Pet pet02 = run.getBean("pet01", Pet.class);
              System.out.println("组件" + (pet01 == pet02));
      
              // top.mnsx.boot_01_helloworld.confit.MyConfig$$EnhancerBySpringCGLIB$$3c423deb@6eafb10e
              MyConfig bean = run.getBean(MyConfig.class);
              System.out.println(bean);
      
              // 如果@Configuration(proxyBeanMethods = true)代理对象调用方法。SpringBoot总会检查这个组件是否在容器中有
              // 保持组件单例
              User user01 = bean.user01();
              User user02 = bean.user01();
              System.out.println(user01 == user02);
      
              User user03 = run.getBean("user01", User.class);
              Pet pet03 = run.getBean("pet01", Pet.class);
      
              System.out.println("用户的宠物：" + (user03.getPet() == pet03));
          }
      
      }
      ```

      ```java
      @Data
      @AllArgsConstructor
      @NoArgsConstructor
      public class User {
          private String name;
          private Integer age;
          private Pet pet;
      
          public User(String name, Integer age) {
              this.name = name;
              this.age = age;
          }
      }
      ```

      ```java
      @Data
      @AllArgsConstructor
      @NoArgsConstructor
      public class Pet {
          private String name;
      }
      ```


* @Bean、@Component、@Contoller、@Service、@Repository

* @ComponentScan、@Import

  ```java
  /**
   * 配置类里面使用@Bean标注在方法上给容器注册主键，默认也是单实例的
   * 配置类本身也是组件
   * proxyBeanMethods：代理bean方法
   * Full(proxyBeanMethods = true，
   * Lite(proxyBeanMethods = false
   *
   *
   * @Import: 给容器中自动创建出指定类型的组件，默认组件的名字就是全类名
   */
  @Import({User.class})
  @Configuration(proxyBeanMethods = true) // 告诉SpringBoot这是一个配置类 == 配置文件
  public class MyConfig {
  ```

* @Conditional

  条件装配：满足Conditional指定的条件，则进行组件注入

  ```java
  /**
   * 配置类里面使用@Bean标注在方法上给容器注册主键，默认也是单实例的
   * 配置类本身也是组件
   * proxyBeanMethods：代理bean方法
   * Full(proxyBeanMethods = true，
   * Lite(proxyBeanMethods = false
   *
   *
   * @Import: 给容器中自动创建出指定类型的组件，默认组件的名字就是全类名
   */
  @Import({User.class})
  @Configuration(proxyBeanMethods = true) // 告诉SpringBoot这是一个配置类 == 配置文件
  public class MyConfig {
  
      /**
       * 外部无论对配置类中的这个组件注册方法调用多少次获取的都是之前注册容器中的单实例对象
       */
      @Bean("user01") // 给容器添加组件。以方法名作为组件的id，返回类型就是组件类型，返回的值就是组件在容器中的实例
      public User user01() {
          User user = new User("mnsx", 20);
          user.setPet(tomcatPet());
          return user;
      }
  
      @Bean("pet01")
      @ConditionalOnBean(name = "user01")
      public Pet tomcatPet() {
          return new Pet("tomcat");
      }
  }
  ```

  ```java
          System.out.println("容器中pet01组件" + run.containsBean("pet01"));
  
          System.out.println("容器中user01组件" + run.containsBean("user01"));
  ```

2. 原生配置文件引入

    * ImportResource

      `@ImportResource("classpath:beans.xml")`

      导入Spring的配置文件

3. 属性值绑定

   使用原生Java读取到properties文件种的内容，并且把它封装到JavaBean中

   ```java
   public class getProperties {
       public static void main(String[] args) throws FileNotFoundException, IOException {
   		Properties prop = new Properties();
           prop.load(new FileInputStream("a.properties"));
           Enumeration enum = prop.propertyNames();
           while (enum.hasMoreElements()) {
   			String strKey = (String) enum1.nextElement();
               String strValue = prop.getProperty(strKey);
               System.out.println(strKey + "=" + strValue);
               // 封装到Bean
           }
       }
   }
   ```

    * @ConfigurationProperties + @Component

      ```java
      @Data
      @Component
      @ConfigurationProperties(prefix = "mycar")
      public class Car {
          private String brand;
          private String price;
      }
      ```

    * @EnableConfigurationProperties + @ConfigurationProperties

      ```java
      @SpringBootConfiguration
      @EnableAutoConfiguration
      @ComponentScan("top.mnsx")
      // 开启Car的属性功能
      // 将Car自动注册到其中
      @EnableConfigurationProperties(Car.class)
      public class Boot01HelloworldApplication {
      ```

## 自动配置原理入门

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
```

1. 自动配置注解

   @SpringBootConfiguration

    * @Configuration代表当前是一个配置类

   @ComponentScan

   指定扫描组件的包

   @EnableAutoConfiguration

   ```java
   @AutoConfigurationPackage
   @Import({AutoConfigurationImportSelector.class})
   public @interface EnableAutoConfiguration {
   ```

    * @AutoConfigurationPackage

      ```java
      @Import({Registrar.class})
      public @interface AutoConfigurationPackage {
          
      ----> Registrar.class
          public static void register(BeanDefinitionRegistry registry, String... packageNames) {
              if (registry.containsBeanDefinition(BEAN)) {
                  AutoConfigurationPackages.BasePackagesBeanDefinition beanDefinition = (AutoConfigurationPackages.BasePackagesBeanDefinition)registry.getBeanDefinition(BEAN);
                  beanDefinition.addBasePackages(packageNames);
              } else {
                  registry.registerBeanDefinition(BEAN, new AutoConfigurationPackages.BasePackagesBeanDefinition(packageNames));
              }
      
          }
         
      // 利用Registrar给容器中导入一系列组件
      // 将指定的一个包下的所有组件导入到程序中（MainApplication 所在包下）
      ```

    * @Import({AutoConfigurationImportSelector.class})

      ```java
      1. 利用getAutoConfigurationEntry(annotationMetadata); 给容器中批量导入一些组件
      2. 调用List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes)获取所有需要导入到容器中的配置类
      3. 利用工厂加载Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader)得到所有的组件
      4. 从META-INF/spring.facotries位置来加载一个文件，默认扫描当前系统中所有META-INF/spring.facotries中的配置文件
      ```

      ```xml
      文件中写死了springboot一启动就要加载的所有配置类
      spring-boot-autoconfigure-2.3.4.RELEASE.jar包中的spring.factories
      # Auto Configure
      org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
      org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
      org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
      org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
      org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
      org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
      org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
      org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration,\
      org.springframework.boot.autoconfigure.context.LifecycleAutoConfiguration,\
      org.springframework.boot.autoconfigure.context.MessageSourceAutoConfiguration,\
      org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration,\
      org.springframework.boot.autoconfigure.couchbase.CouchbaseAutoConfiguration,\
      org.springframework.boot.autoconfigure.dao.PersistenceExceptionTranslationAutoConfiguration,\
      org.springframework.boot.autoconfigure.data.cassandra.CassandraDataAutoConfiguration,\
      org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveDataAutoConfiguration,\
      org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveRepositoriesAutoConfiguration,\
      org.springframework.boot.autoconfigure.data.cassandra.CassandraRepositoriesAutoConfiguration,\
      org.springframework.boot.autoconfigure.data.couchbase.CouchbaseDataAutoConfiguration,\
      org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveDataAutoConfiguration,\
      org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveRepositoriesAutoConfiguration,\
      org.springframework.boot.autoconfigure.data.couchbase.CouchbaseRepositoriesAutoConfiguration,\
      org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchDataAutoConfiguration,\
      org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchRepositoriesAutoConfiguration,\
      org.springframework.boot.autoconfigure.data.elasticsearch.ReactiveElasticsearchRepositoriesAutoConfiguration,\
      org.springframework.boot.autoconfigure.data.elasticsearch.ReactiveElasticsearchRestClientAutoConfiguration,\
      org.springframework.boot.autoconfigure.data.jdbc.JdbcRepositoriesAutoConfiguration,\
      org.springframework.boot.autoconfigure.data.jpa.JpaRepositoriesAutoConfiguration,\
      org.springframework.boot.autoconfigure.data.ldap.LdapRepositoriesAutoConfiguration,\
      org.springframework.boot.autoconfigure.data.mongo.MongoDataAutoConfiguration,\
      org.springframework.boot.autoconfigure.data.mongo.MongoReactiveDataAutoConfiguration,\
      org.springframework.boot.autoconfigure.data.mongo.MongoReactiveRepositoriesAutoConfiguration,\
      org.springframework.boot.autoconfigure.data.mongo.MongoRepositoriesAutoConfiguration,\
      org.springframework.boot.autoconfigure.data.neo4j.Neo4jDataAutoConfiguration,\
      org.springframework.boot.autoconfigure.data.neo4j.Neo4jReactiveDataAutoConfiguration,\
      org.springframework.boot.autoconfigure.data.neo4j.Neo4jReactiveRepositoriesAutoConfiguration,\
      org.springframework.boot.autoconfigure.data.neo4j.Neo4jRepositoriesAutoConfiguration,\
      org.springframework.boot.autoconfigure.data.r2dbc.R2dbcDataAutoConfiguration,\
      org.springframework.boot.autoconfigure.data.r2dbc.R2dbcRepositoriesAutoConfiguration,\
      org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration,\
      org.springframework.boot.autoconfigure.data.redis.RedisReactiveAutoConfiguration,\
      org.springframework.boot.autoconfigure.data.redis.RedisRepositoriesAutoConfiguration,\
      org.springframework.boot.autoconfigure.data.rest.RepositoryRestMvcAutoConfiguration,\
      org.springframework.boot.autoconfigure.data.web.SpringDataWebAutoConfiguration,\
      org.springframework.boot.autoconfigure.elasticsearch.ElasticsearchRestClientAutoConfiguration,\
      org.springframework.boot.autoconfigure.flyway.FlywayAutoConfiguration,\
      org.springframework.boot.autoconfigure.freemarker.FreeMarkerAutoConfiguration,\
      org.springframework.boot.autoconfigure.groovy.template.GroovyTemplateAutoConfiguration,\
      org.springframework.boot.autoconfigure.gson.GsonAutoConfiguration,\
      org.springframework.boot.autoconfigure.h2.H2ConsoleAutoConfiguration,\
      org.springframework.boot.autoconfigure.hateoas.HypermediaAutoConfiguration,\
      org.springframework.boot.autoconfigure.hazelcast.HazelcastAutoConfiguration,\
      org.springframework.boot.autoconfigure.hazelcast.HazelcastJpaDependencyAutoConfiguration,\
      org.springframework.boot.autoconfigure.http.HttpMessageConvertersAutoConfiguration,\
      org.springframework.boot.autoconfigure.http.codec.CodecsAutoConfiguration,\
      org.springframework.boot.autoconfigure.influx.InfluxDbAutoConfiguration,\
      org.springframework.boot.autoconfigure.info.ProjectInfoAutoConfiguration,\
      org.springframework.boot.autoconfigure.integration.IntegrationAutoConfiguration,\
      org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration,\
      org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
      org.springframework.boot.autoconfigure.jdbc.JdbcTemplateAutoConfiguration,\
      org.springframework.boot.autoconfigure.jdbc.JndiDataSourceAutoConfiguration,\
      org.springframework.boot.autoconfigure.jdbc.XADataSourceAutoConfiguration,\
      org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration,\
      org.springframework.boot.autoconfigure.jms.JmsAutoConfiguration,\
      org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration,\
      org.springframework.boot.autoconfigure.jms.JndiConnectionFactoryAutoConfiguration,\
      org.springframework.boot.autoconfigure.jms.activemq.ActiveMQAutoConfiguration,\
      org.springframework.boot.autoconfigure.jms.artemis.ArtemisAutoConfiguration,\
      org.springframework.boot.autoconfigure.jersey.JerseyAutoConfiguration,\
      org.springframework.boot.autoconfigure.jooq.JooqAutoConfiguration,\
      org.springframework.boot.autoconfigure.jsonb.JsonbAutoConfiguration,\
      org.springframework.boot.autoconfigure.kafka.KafkaAutoConfiguration,\
      org.springframework.boot.autoconfigure.availability.ApplicationAvailabilityAutoConfiguration,\
      org.springframework.boot.autoconfigure.ldap.embedded.EmbeddedLdapAutoConfiguration,\
      org.springframework.boot.autoconfigure.ldap.LdapAutoConfiguration,\
      org.springframework.boot.autoconfigure.liquibase.LiquibaseAutoConfiguration,\
      org.springframework.boot.autoconfigure.mail.MailSenderAutoConfiguration,\
      org.springframework.boot.autoconfigure.mail.MailSenderValidatorAutoConfiguration,\
      org.springframework.boot.autoconfigure.mongo.embedded.EmbeddedMongoAutoConfiguration,\
      org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration,\
      org.springframework.boot.autoconfigure.mongo.MongoReactiveAutoConfiguration,\
      org.springframework.boot.autoconfigure.mustache.MustacheAutoConfiguration,\
      org.springframework.boot.autoconfigure.neo4j.Neo4jAutoConfiguration,\
      org.springframework.boot.autoconfigure.netty.NettyAutoConfiguration,\
      org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration,\
      org.springframework.boot.autoconfigure.quartz.QuartzAutoConfiguration,\
      org.springframework.boot.autoconfigure.r2dbc.R2dbcAutoConfiguration,\
      org.springframework.boot.autoconfigure.r2dbc.R2dbcTransactionManagerAutoConfiguration,\
      org.springframework.boot.autoconfigure.rsocket.RSocketMessagingAutoConfiguration,\
      org.springframework.boot.autoconfigure.rsocket.RSocketRequesterAutoConfiguration,\
      org.springframework.boot.autoconfigure.rsocket.RSocketServerAutoConfiguration,\
      org.springframework.boot.autoconfigure.rsocket.RSocketStrategiesAutoConfiguration,\
      org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration,\
      org.springframework.boot.autoconfigure.security.servlet.UserDetailsServiceAutoConfiguration,\
      org.springframework.boot.autoconfigure.security.servlet.SecurityFilterAutoConfiguration,\
      org.springframework.boot.autoconfigure.security.reactive.ReactiveSecurityAutoConfiguration,\
      org.springframework.boot.autoconfigure.security.reactive.ReactiveUserDetailsServiceAutoConfiguration,\
      org.springframework.boot.autoconfigure.security.rsocket.RSocketSecurityAutoConfiguration,\
      org.springframework.boot.autoconfigure.security.saml2.Saml2RelyingPartyAutoConfiguration,\
      org.springframework.boot.autoconfigure.sendgrid.SendGridAutoConfiguration,\
      org.springframework.boot.autoconfigure.session.SessionAutoConfiguration,\
      org.springframework.boot.autoconfigure.security.oauth2.client.servlet.OAuth2ClientAutoConfiguration,\
      org.springframework.boot.autoconfigure.security.oauth2.client.reactive.ReactiveOAuth2ClientAutoConfiguration,\
      org.springframework.boot.autoconfigure.security.oauth2.resource.servlet.OAuth2ResourceServerAutoConfiguration,\
      org.springframework.boot.autoconfigure.security.oauth2.resource.reactive.ReactiveOAuth2ResourceServerAutoConfiguration,\
      org.springframework.boot.autoconfigure.solr.SolrAutoConfiguration,\
      org.springframework.boot.autoconfigure.sql.init.SqlInitializationAutoConfiguration,\
      org.springframework.boot.autoconfigure.task.TaskExecutionAutoConfiguration,\
      org.springframework.boot.autoconfigure.task.TaskSchedulingAutoConfiguration,\
      org.springframework.boot.autoconfigure.thymeleaf.ThymeleafAutoConfiguration,\
      org.springframework.boot.autoconfigure.transaction.TransactionAutoConfiguration,\
      org.springframework.boot.autoconfigure.transaction.jta.JtaAutoConfiguration,\
      org.springframework.boot.autoconfigure.validation.ValidationAutoConfiguration,\
      org.springframework.boot.autoconfigure.web.client.RestTemplateAutoConfiguration,\
      org.springframework.boot.autoconfigure.web.embedded.EmbeddedWebServerFactoryCustomizerAutoConfiguration,\
      org.springframework.boot.autoconfigure.web.reactive.HttpHandlerAutoConfiguration,\
      org.springframework.boot.autoconfigure.web.reactive.ReactiveMultipartAutoConfiguration,\
      org.springframework.boot.autoconfigure.web.reactive.ReactiveWebServerFactoryAutoConfiguration,\
      org.springframework.boot.autoconfigure.web.reactive.WebFluxAutoConfiguration,\
      org.springframework.boot.autoconfigure.web.reactive.WebSessionIdResolverAutoConfiguration,\
      org.springframework.boot.autoconfigure.web.reactive.error.ErrorWebFluxAutoConfiguration,\
      org.springframework.boot.autoconfigure.web.reactive.function.client.ClientHttpConnectorAutoConfiguration,\
      org.springframework.boot.autoconfigure.web.reactive.function.client.WebClientAutoConfiguration,\
      org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration,\
      org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration,\
      org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration,\
      org.springframework.boot.autoconfigure.web.servlet.HttpEncodingAutoConfiguration,\
      org.springframework.boot.autoconfigure.web.servlet.MultipartAutoConfiguration,\
      org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration,\
      org.springframework.boot.autoconfigure.websocket.reactive.WebSocketReactiveAutoConfiguration,\
      org.springframework.boot.autoconfigure.websocket.servlet.WebSocketServletAutoConfiguration,\
      org.springframework.boot.autoconfigure.websocket.servlet.WebSocketMessagingAutoConfiguration,\
      org.springframework.boot.autoconfigure.webservices.WebServicesAutoConfiguration,\
      org.springframework.boot.autoconfigure.webservices.client.WebServiceTemplateAutoConfiguration
      ```

2. 按需开启自动配置项

   虽然127个场景的自动配置启动时全部都要启动，但是按照条件装配规则，会进行按需装配

3. 修改默认配置

   ```java
   @Bean
   @ConditionalOnBean(MultipartResoulver.class) // 容器中有这个类型的组件
   @ConditionalOnMissingBean(name = DispatcherServlet.MUTIPART_RESOLVER_BEAN_NAME) // 容器中没有这个名字的组件
   public MultipartResolver multipartResolver(MultipartResolver resolver) {
       // 给@Bean标注的方法传入了对象参数，这个参数的值就会从容器中找
       // SpringMVC multipartResolver 防止有写用户配置的文件上传解析器不符合规范
   	return resolver;
   }
   给容器中加入了，文件上传解析器
   ```

   SpringBoot默认会在底层配置号所有的组件，但是如果用户已经配置了自己的组件，那么会以用户的优先

   **总结：**

    * SpringBoot先加载所有的自动配置类 xxxAutoConfiguration
    * 每个自动配置类按照条件进行装配，默认都会绑定配置文件指定的值 xxxProperties里面拿，xxxProperties和配置文件进行了绑定
    * 生效的配置类就会给容器中装配很多的组件
    * 只要容器中有这些组件，相当于这些功能有了
    * 如果用户有自己配置的，那么就以用户的优先
    * 定制化配置
        * 用户直接自己@Bean替换底层的组件
        * 用户去看这个组件是获取的配置文件的什么值，去配置文件中修改

4. 最佳实践

    * 引入场景依赖
        * https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.build-systems.starters
    * 查看自动配置的组件（选）
        * 自己分析，引入场景对应的自动配置一般都生效了
        * 配置文件中debug=true开启自动配置报告，Negative（不生效）、Positive（生效）
    * 是否需要修改配置
        * 参照文档修改配置
            * https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#appendix.application-properties
            * 自己分析。xxxProperties绑定了配置文件的那些
        * 自定义加入或者替换组件
            * @Bean @Component...
            * 自定义器 xxxCustomizer

## 简化开发

1. lombok

   @Slf4j——简化日志开发

   ...

2. dev-tools

   导入配置依赖后使用Ctrl + F9

   restart而不是reload

   不是纯正的热更新，只是自动重启

3. Spring Initailizr

    略

# 配置文件

## 文件类型

1. properties

2. yaml

   ### 简介

   YAML是“YAML Ain't Markup Language”（YAML不是一种标记语言）的递归缩写。在开发这种语言时，YAML的意思其实是：“Yet Another Markup Language”（仍是一种标记语言）

   **非常适合用来做以数据为中心的配置文件**

   ### 基本语法

    * key: value kv之间有空格

    * 大小写敏感

    * 使用缩进表示层级关系
    * 缩进不允许lab，只允许空格

    * 缩进的空格数不重要，只要相同层级的元素左对齐即可

    * ‘#’表示注解

    * ‘’与“”表示字符串内容转义/不转义

   ### 数据类型

    * 字面量：单个的、不可再分的值

      ```yaml
      k: v
      ```

    * 对象：键值对的集合

      ```yaml
      行内写法：
      k: {k1:v1,k2:v2}
      分行写法：
      k:
          k1:v1
          k2:v2
      ```

    * 数组：一组按次序排序的值

      ```yaml
      行内写法：
      k: [v1,v2,v3]
      分行写法：
      k:
          -v1
          -v2
          -v3
      ```

# web开发

## 静态资源访问

* 静态资源目录

  类路径下：/static（/public、/resource、/META-INF/resource）

  访问： 当前项目的根路径/ + 静态资源名

  原理：静态映射/\*\*

  请求进来，先去找Controller看能不能处理，不能处理的所有请求都交给静态资源处理器处理。静态资源也没找到，返回404

  ```yaml
  spring:
  	resources:
  		static-locations: classpath:/xxx
  ```

* 静态资源访问前缀

  默认无前缀

  ```yaml
  spring:
  	mvc:
  		static-path-pattern: /res/**
  ```

* 欢迎页支持

    * 静态资源路径下index.html

    * 自定义Favicon，静态资源路径下的favicon.ico

* 静态资源配置原理

    * SpringBoot启动默认加载 xxxAUtoConfiguration类（自动配置类）

    * SpringMVC功能的自动配置类 MVCAutoConfiguration、

      ```java
      @Configuration(
          proxyBeanMethods = false
      )
      @ConditionalOnWebApplication(
          type = Type.SERVLET
      )
      @ConditionalOnClass({Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class})
      @ConditionalOnMissingBean({WebMvcConfigurationSupport.class})
      @AutoConfigureOrder(-2147483638)
      @AutoConfigureAfter({DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class, ValidationAutoConfiguration.class})
      public class WebMvcAutoConfiguration {
      ```

    * 给容器中配置了的组件

      ```java
       @Configuration(
              proxyBeanMethods = false
          )
          @Import({WebMvcAutoConfiguration.EnableWebMvcConfiguration.class})
          @EnableConfigurationProperties({WebMvcProperties.class, WebProperties.class})
          @Order(0)
          public static class WebMvcAutoConfigurationAdapter implements WebMvcConfigurer, ServletContextAware {
      ```

    * 配置文件的相关属性和xxx进行了绑定。WebMvcProperties == spring.mvc、ResourceProperties == spring.resources

      > 配置类只有一个有参构造器，有参构造器所有参数的值都会从容器中获取
      >
      > ```java
    > public WebMvcAutoConfigurationAdapter(WebProperties webProperties, WebMvcProperties mvcProperties, ListableBeanFactory beanFactory, ObjectProvider<HttpMessageConverters> messageConvertersProvider, ObjectProvider<WebMvcAutoConfiguration.ResourceHandlerRegistrationCustomizer> resourceHandlerRegistrationCustomizerProvider, ObjectProvider<DispatcherServletPath> dispatcherServletPath, ObjectProvider<ServletRegistrationBean<?>> servletRegistrations) {
    >             this.resourceProperties = webProperties.getResources(); // ResourceProperties 获取和spring.resources绑定的所有的值的对象
    >             this.mvcProperties = mvcProperties; // WebMvcProperties获取和spring.mvc绑定的所有的值的对象
    >             this.beanFactory = beanFactory; // Spring的BeanFactory
    >             this.messageConvertersProvider = messageConvertersProvider; // 找到所有的HttpMessageConverters
    >             this.resourceHandlerRegistrationCustomizer = (WebMvcAutoConfiguration.ResourceHandlerRegistrationCustomizer)resourceHandlerRegistrationCustomizerProvider.getIfAvailable(); // 找到资源处理器的自定义器
    >             this.dispatcherServletPath = dispatcherServletPath; // 
    >             this.servletRegistrations = servletRegistrations; // 给应用注册Servlet、Filter...
    >             this.mvcProperties.checkConfiguration();
    >         }
    
* 资源处理的默认规则

  ```java
          public void addResourceHandlers(ResourceHandlerRegistry registry) {
              if (!this.resourceProperties.isAddMappings()) {
                  logger.debug("Default resource handling disabled");
              } else {
                  this.addResourceHandler(registry, "/webjars/**", "classpath:/META-INF/resources/webjars/");
                  this.addResourceHandler(registry, this.mvcProperties.getStaticPathPattern(), (registration) -> {
                      registration.addResourceLocations(this.resourceProperties.getStaticLocations());
                      if (this.servletContext != null) {
                          ServletContextResource resource = new ServletContextResource(this.servletContext, "/");
                          registration.addResourceLocations(new Resource[]{resource});
                      }
  
                  });
              }
          }
  ```

  ```yaml
  spring:
  	resources:
  		add-mappings: false 禁用所有静态资源规则
  ```

  ```java
  private static final String[] CLASSPATH_RESOURCE_LOCATIONS = new String[]{"classpath:/META-INF/resources/", "classpath:/resources/", "classpath:/static/", "classpath:/public/"};
  ```

* 欢迎页的处理规则

  HandlerMapping：处理器映射。保存了每一个handler能够处理那些请求

  ```java
          @Bean
          public WelcomePageHandlerMapping welcomePageHandlerMapping(ApplicationContext applicationContext, FormattingConversionService mvcConversionService, ResourceUrlProvider mvcResourceUrlProvider) {
              WelcomePageHandlerMapping welcomePageHandlerMapping = new WelcomePageHandlerMapping(new TemplateAvailabilityProviders(applicationContext), applicationContext, this.getWelcomePage(), this.mvcProperties.getStaticPathPattern());
              welcomePageHandlerMapping.setInterceptors(this.getInterceptors(mvcConversionService, mvcResourceUrlProvider));
              welcomePageHandlerMapping.setCorsConfigurations(this.getCorsConfigurations());
              return welcomePageHandlerMapping;
          }
  ```

## 请求参数处理

请求映射

* @xxxMapping
* Rest风格支持（使用HTTP请求方式动词来表达对资源的操作）
    * 核心filter：HiddenHttpMethodFilter
    * 用法：表单method=post，隐藏域\_method=put

### Rest原理（表单提交要使用Rest的时候）

```java
    @Bean
    @ConditionalOnMissingBean({HiddenHttpMethodFilter.class})
    @ConditionalOnProperty(
        prefix = "spring.mvc.hiddenmethod.filter",
        name = {"enabled"}
    )
    public OrderedHiddenHttpMethodFilter hiddenHttpMethodFilter() {
        return new OrderedHiddenHttpMethodFilter();
    }
```

* 表单提交会带上method=PUT
* 请求过来会被HiddenHttpMethodFilter拦截
    * 请求是否正常，并且是post
    * 获取到\_method的值
    * 兼容请求：PUT、DELETE、POST、GET、PETCH
    * 原生requet（post），包装模式requestWrapper重写了getMethod方法，返回\_method的值
    * 过滤器放行时使用的wrapper

### 请求映射原理

SpringMVC功能分析都从org.springframework.web.sevlet.DispatcherServlet -> doDispatch()

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
        HttpServletRequest processedRequest = request;
        HandlerExecutionChain mappedHandler = null;
        boolean multipartRequestParsed = false;
        WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

        try {
            try {
                ModelAndView mv = null;
                Object dispatchException = null;

                try {
                    processedRequest = this.checkMultipart(request);
                    multipartRequestParsed = processedRequest != request;
                    // 找到当前请求使用那个Handler处理
                    // HandlerMapping 处理器映射
                    mappedHandler = this.getHandler(processedRequest);
                    if (mappedHandler == null) {
                        this.noHandlerFound(processedRequest, response);
                        return;
                    }

                    HandlerAdapter ha = this.getHandlerAdapter(mappedHandler.getHandler());
                    String method = request.getMethod();
                    boolean isGet = HttpMethod.GET.matches(method);
                    if (isGet || HttpMethod.HEAD.matches(method)) {
                        long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
                        if ((new ServletWebRequest(request, response)).checkNotModified(lastModified) && isGet) {
                            return;
                        }
                    }

                    if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                        return;
                    }

                    mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
                    if (asyncManager.isConcurrentHandlingStarted()) {
                        return;
                    }

                    this.applyDefaultViewName(processedRequest, mv);
                    mappedHandler.applyPostHandle(processedRequest, response, mv);
                } catch (Exception var20) {
                    dispatchException = var20;
                } catch (Throwable var21) {
                    dispatchException = new NestedServletException("Handler dispatch failed", var21);
                }

                this.processDispatchResult(processedRequest, response, mappedHandler, mv, (Exception)dispatchException);
            } catch (Exception var22) {
                this.triggerAfterCompletion(processedRequest, response, mappedHandler, var22);
            } catch (Throwable var23) {
                this.triggerAfterCompletion(processedRequest, response, mappedHandler, new NestedServletException("Handler processing failed", var23));
            }

        } finally {
            if (asyncManager.isConcurrentHandlingStarted()) {
                if (mappedHandler != null) {
                    mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
                }
            } else if (multipartRequestParsed) {
                this.cleanupMultipart(processedRequest);
            }

        }
    }
```

RequestMappingHandlerMapping：保存了所有@RequestMapping和handler的映射规则

所有的请求映射都在HandlerMapping中

* SpringBoot自动配置欢迎页的WelcomePageHandlerMapping。访问/就访问到index.html
* SpringBoot自动配置了默认的RequestMappingHandlerMapping
* 请求进来，挨个尝试所有的HandlerMapping是否有请求信息
    * 如果有就找到这个请求对应的handler
    * 如果没有就时下一个HandlerMapping
* 我们需要一些自定义的映射处理，我们也可以自己给容器中放HandlerMapping，自动HandlerMapping

## 常用参数和基本注解

* 注解

  | 注解              | 功能               |
    | ----------------- | ------------------ |
  | @PathVariable     | 获取路径变量       |
  | @RequestHeader    | 获取请求头         |
  | @RequestParam     | 获取请求参数       |
  | @CookieValue      | 获取Cookie值       |
  | @RequestBody      | 获取请求体【post】 |
  | @RequestAttribute | 获取request域属性  |
  | @MatrixVariable   | 获取矩阵变量       |

> 矩阵变量使用方式——
>
> 1. 语法：/cars/sell;low=34;brand=byd,audi,yd
> 2. SpringBoot默认是禁用了矩阵变量的功能
     >
* 手动开启：原理——对于路径的处理。UrlPathHelper进行解析。removeSemicolonContent（移除分号内容）改为false来支持矩阵变量
> 3. 矩阵变量必须有url路径变量才能被解析

## 参数处理原理

* HandlerMapping 中找到能处理请求的Handler（Controller.Mehtod）
* 为当前Handler找一个适配器HandlerAdapter

1. HandlerAdapter

    1. 支持方法上标注@RequestMapping——> RequestMappingHandlerAdapter
    2. 支持函数式编程——> HandlerFunctionAdapter

2. 执行目标方法

   ```java
   // Actually invoke the handler
   mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
   ```

   ```java
   mav = invokeHandlerMthod(request, response, handlerMthod); // 执行目标方法
   
   
   Object returnValue = invokeForRequest(webRequest, mavContainer, provideArgs)
   ```

3. 参数解析器

   确定将要执行的目标方法的参数是什么类型

    * 判断当前解析器是否支持解析这种参数
    * 支持就调用resolveArgument

4. 返回值处理器

5. 如何确定目标方法每一个参数的值

   ```java
   protected Object[] getMethodArgumentValues(NativeWebRequest request, @Nullable ModelAndViewContainer mavContainer, Object... providedArgs) throws Exception {
           MethodParameter[] parameters = this.getMethodParameters();
           if (ObjectUtils.isEmpty(parameters)) {
               return EMPTY_ARGS;
           } else {
               Object[] args = new Object[parameters.length];
   
               for(int i = 0; i < parameters.length; ++i) {
                   MethodParameter parameter = parameters[i];
                   parameter.initParameterNameDiscovery(this.parameterNameDiscoverer);
                   args[i] = findProvidedArgument(parameter, providedArgs);
                   if (args[i] == null) {
                       if (!this.resolvers.supportsParameter(parameter)) {
                           throw new IllegalStateException(formatArgumentError(parameter, "No suitable resolver"));
                       }
   
                       try {
                           args[i] = this.resolvers.resolveArgument(parameter, mavContainer, request, this.dataBinderFactory);
                       } catch (Exception var10) {
                           if (logger.isDebugEnabled()) {
                               String exMsg = var10.getMessage();
                               if (exMsg != null && !exMsg.contains(parameter.getExecutable().toGenericString())) {
                                   logger.debug(formatArgumentError(parameter, exMsg));
                               }
                           }
   
                           throw var10;
                       }
                   }
               }
   
               return args;
           }
       }
   ```

   ### 遍历所有参数解析器，判断支持解析这个参数的解析器

   ```java
   @Nullable
       private HandlerMethodArgumentResolver getArgumentResolver(MethodParameter parameter) {
           HandlerMethodArgumentResolver result = (HandlerMethodArgumentResolver)this.argumentResolverCache.get(parameter);
           if (result == null) {
               Iterator var3 = this.argumentResolvers.iterator();
   
               while(var3.hasNext()) {
                   HandlerMethodArgumentResolver resolver = (HandlerMethodArgumentResolver)var3.next();
                   if (resolver.supportsParameter(parameter)) {
                       result = resolver;
                       this.argumentResolverCache.put(parameter, resolver);
                       break;
                   }
               }
           }
   
           return result;
       }
   ```

## 复杂参数

Map、Model（Map、model里面的所有数据会被放在request的请求域中
request.setAttribute）、Errors/BindingResult、RedirectAttribute（重定向携带数据）、ServletResponse（response）
