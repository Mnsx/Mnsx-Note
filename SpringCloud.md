# 学习环境

| Cloud | Boot    | Cloud Alibaba | Java | Maven | Mqsql |
| ----- | ------- | ------------- | ---- | ----- | ----- |
| H.SR1 | 2.2.2.R | 2.1.0.R       | 8    | 3.5   | 5.7   |

# Cloud组件升级

![](D:\WorkSpace\Note\Picture\Cloud组件.png)

# 微服务架构编码搭建

## 项目搭建

### 父工程搭建

**(使用SpringInitialzr)**

1. New Project

2. 聚合总父工程名称

3. Maven版本选择

   ![image-20221001122709290](D:\WorkSpace\Note\Picture\Maven版本选择.png)

4. 工程名称

5. 字符编码

   ![image-20221001122801981](D:\WorkSpace\Note\Picture\字符编码选择)

6. 注解生效激活

   ![image-20221001123156043](D:\WorkSpace\Note\Picture\注解生效激活.png)

7. java编译版本选择

   ![image-20221001123122342](D:\WorkSpace\Note\Picture\Java版本选择.png)

8. File Type过滤

   ![image-20221001122942019](D:\WorkSpace\Note\Picture\文件过滤.png)

### 父工程POM

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>sprintcloud</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>

    <!--统一管理jar包版本-->
    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <junit.version>5.8.2</junit.version>
        <log4j.version>2.17.1</log4j.version>
        <lombok.version>1.18.22</lombok.version>
        <mysql.version>8.0.30</mysql.version>
        <druid.version>1.2.8</druid.version>
        <mybatis.sspring.boot.version>2.2.2</mybatis.sspring.boot.version>
    </properties>

    <!--子模块继承之后，提供作用，锁定版本+子module不用写groupId和version-->
    <dependencyManagement>
        <dependencies>
            <!--spring boot 2.2.2-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.2.2.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--spring cloud H.SR1-->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Hoxton.SR1</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--spring cloud alibaba 2.1.0.RELEASE-->
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>2.1.0.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>${mysql.version}</version>
            </dependency>
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>druid</artifactId>
                <version>${druid.version}</version>
            </dependency>
            <dependency>
                <groupId>org.mybatis.spring.boot</groupId>
                <artifactId>mybatis-spring-boot-starter</artifactId>
                <version>${mybatis.sspring.boot.version}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

## DependencyManagement与Dependencies的区别

dependencyManagement元素来提供一种管理依赖版本号的方式

**通常会在一个组织或者项目的最顶层的父POM中看到dependencyManagement元素**

子项目不写version时就跟父项目中使用相同的version

* **dependencyManagement只是声明依赖，并不实现引入**，因此子项目需要显示的生命需要用的依赖
* 如果不在子项目中声明依赖，是不会从父项目中继承下来的，**只有再子项目中声明了依赖并且没有指定版本号时**，才会从父项目中继承该项，**并且version和scope都读取自父pom**
* **如果子项目指定版本，就是用子项目版本**

## 支付模块

**(使用SpringInitialzr)**

### POM文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>spring-cloud</artifactId>
        <groupId>top.mnsx</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloud-provider-payment8001</artifactId>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.2.13-SNSAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

### YML文件

```yml
server:
  port: 8001

spring:
  application:
    name: cloud-payment-service
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/cloud?useUnicode=true?characterEncoding=utf-8&useSSL=false
    username: root
    password: 123123

mybatis:
  mapper-locations: classpath:mapper/*.xml
  type-aliases-package: top.mnsx.springcloud.entity
```

### SQL建表

```sql
create table payment(
id bigint(20) not null auto_increment comment 'id',
serial varchar(200) default '流水号',
primary key(id)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8
```

### 实体类

* 支付类

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Payment implements Serializable {
    private Long id;
    private String serial;
}
```

* Json返回类

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class CommonResult<T> {
    private Integer code;
    private String message;
    private T data;

    public CommonResult(Integer code, String message) {
        this(code, message, null);
    }
}
```

### 持久层

* 接口

```java
@Mapper
public interface PaymentDao {
    public int create(Payment payment);

    public Payment getPaymentById(@Param("id") Long id);
}
```

* XML

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="top.mnsx.springcloud.dao.PaymentDao">
    <insert id="create" parameterType="payment" useGeneratedKeys="true" keyProperty="id">
        insert into payment(serial) values (#{serial});
    </insert>

    <resultMap id="BaseResultMap" type="payment">
        <id property="id" column="id" jdbcType="BIGINT"/>
        <result property="serial" column="serial" jdbcType="VARCHAR"/>
    </resultMap>
    <select id="getPaymentById" parameterType="long" resultMap="BaseResultMap">
        select * from payment where id = #{id};
    </select>
</mapper>
```

### 逻辑层

* 接口

```java
public interface PaymentService {
    public int create(Payment payment);

    public Payment getPaymentById(@Param("id") Long id);
}
```

* 实现类

```java
@Service
public class PaymentServiceImpl implements PaymentService {
    @Resource
    private PaymentDao paymentDao;

    @Override
    public int create(Payment payment) {
        return paymentDao.create(payment);
    }

    @Override
    public Payment getPaymentById(Long id) {
        return paymentDao.getPaymentById(id);
    }
}
```

### 控制层

```java
@RestController
@Slf4j
public class PaymentController {
    @Resource
    private PaymentService paymentService;

    @PostMapping("/payment/create")
    public CommonResult create(@RequestBody Payment payment) {
        int result = paymentService.create(payment);
        log.info("插入结果：" + result);
        if (result > 0) {
            return new CommonResult(200, "插入数据库成功", result);
        } else {
            return new CommonResult(444, "插入数据库失败", null);
        }
    }

    @GetMapping("/payment/get/{id}")
    public CommonResult getPaymentById(@PathVariable Long id) {
        Payment payment = paymentService.getPaymentById(id);
        log.info("获得结果：" + payment);

        if (payment != null) {
            return new CommonResult(200, "查询数据库成功", payment);
        } else {
            return new CommonResult(444, "查询数据库失败, 查询ID" + id + "失败");
        }
    }
}
```

## 热部署

1. 添加热部署依赖

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-devtools</artifactId>
       <scope>runtime</scope>
       <optional>true</optional>
   </dependency>
   ```

2. 添加热部署插件到父工程的pom.xml

   ```xml
   <build>
       <plugins>
           <plugin>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-maven-plugin</artifactId>
               <configuration>
                   <fork>true</fork>
                   <addResources>true</addResources>
               </configuration>
           </plugin>
       </plugins>
   </build>
   ```

3. 开启自动编译![image-20221001112910385](D:\WorkSpace\Note\Picture\开启自动编译.png)

4. 开启自动启动选项![image-20221001112814078](D:\WorkSpace\Note\Picture\开启自动启动.png)

5. 重启IEDA

## 订单模块

### 实例类

> 使用payment的entity

### RestTemplate

**创建配置类**

```java
@Configuration
public class ApplicationContextConfig {
    @Bean
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

### 控制层

```java
@RestController
@Slf4j
public class OrderController {
    public static final String PAYMENT_URL = "http://localhost:8001";

    @Resource
    private RestTemplate restTemplate;

    @SuppressWarnings("unchecked")
    @GetMapping("/consumer/payment/create")
    public CommonResult<Integer> create(Payment payment) {
        return restTemplate.postForObject(PAYMENT_URL + "/payment/create", payment, CommonResult.class);
    }

    @SuppressWarnings("unchecked")
    @GetMapping("/consumer/payment/get/{id}")
    public CommonResult<Payment> getPayment(@PathVariable("id") Long id) {
        return restTemplate.getForObject(PAYMENT_URL + "/payment/get/" + id, CommonResult.class);
    }
}
```

## 工程重构

将重复的内容提取到一个模块中，使用mavem的install方法将模块发布到仓库中，在需要使用的模块中添加依赖即可

**要求同一个项目能够使用相同的包结构，保证抽取时不用大量修改import**

# 服务注册中心——Eureka

## 服务治理

在传统的rpc远程调用框架中，管理每个服务与服务之间依赖关系比较复杂，管理比较复杂，所以需要使用服务治理，管理服务与服务之间依赖关系，可以实现服务调用、负载均衡、容错等，实现服务发现与注册

## Eureka组件

* Eureka Server提供服务注册服务

  **各个微服务节点通过配置启动后**，在EurekaServer中注册，**EurekaServer中的服务注册表中将会存储所有可用服务节点的信息**，服务节点的信息在界面中可以直观看到

* EurekaClient通过注册中心进行访问

  是一个Java客户端，用于简化EurekaServer的交互，**服务点具备一个内置的、使用轮询（round-robin）负载算法的负载均衡器**。应用启动后，**将会向EurekaServer发送心跳（默认周期30s）**。如果EurekaServer在多个心跳周期内没有接收到某个节点的心跳，**EurekaSerer将会从服务注册表中将这个服务节点移除（默认90s）**

## 单机Eureka

### 启动Eureka

* 插入依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

* YAML文件

```yaml
server:
  port: 7001

eureka:
  instance:
    hostname: localhost
  client:
    register-with-eureka: false
    fetch-registry: false
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

* 配置启动注解

```java
@SpringBootApplication
@EnableEurekaServer
public class CloudEurekaServer7001Application {

    public static void main(String[] args) {
        SpringApplication.run(CloudEurekaServer7001Application.class, args);
    }

}
```

### 配置Privoder注册Eureka

* 插入依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

* YAML文件

```yaml
eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://localhost:7001/eureka
```

* 配置启动注解

```java
@SpringBootApplication
@EnableEurekaClient
public class CloudProviderPayment8001Application {

    public static void main(String[] args) {
        SpringApplication.run(CloudProviderPayment8001Application.class, args);
    }

}
```

![image-20221001232147620](D:\WorkSpace\Note\Picture\Eureka启动成功.png)

### 配置Consumer注册Eureka

**同理Provider**

## Eureka集群

* 修改映射配置

  ![image-20221002080312232](D:\WorkSpace\Note\Picture\修改映射配置.png)

* 修改YAML文件

  * 7001

    ![image-20221002080400382](D:\WorkSpace\Note\Picture\Eureka7001yaml文件.png)

  * 7002

    ![image-20221002080430527](D:\WorkSpace\Note\Picture\Eureka7002yaml.png)

  * 80

    ![image-20221002081314005](D:\WorkSpace\Note\Picture\Eureka80yaml.png)

  * 8001

    ![image-20221002081341708](D:\WorkSpace\Note\Picture\Eureka8001yaml.png)

## Provider集群

* 修改ConsumerURL路径

  `public static final String PAYMENT_URL = "http://CLOUD-PAYMENT-SERVICE";`

* 为RestTemplate添加注解

  ```java
  @Configuration
  public class ApplicationContextConfig {
      @Bean
      @LoadBalanced
      public RestTemplate getRestTemplate() {
          return new RestTemplate();
      }
  }
  ```

## Actuator微服务信息完善

### 屏蔽主机名

![image-20221002090757380](D:\WorkSpace\Note\Picture\屏蔽主机名)

* 修改yaml文件

  ```yaml
  eureka:
    client:
      register-with-eureka: true
      fetch-registry: true
      service-url:
        defaultZone: http://eureka7002.com:7002/eureka,http://eureka7001.com:7001/eureka
    instance:
      instance-id: payment8001
  ```

  ```yaml
  eureka:
    client:
      register-with-eureka: true
      fetch-registry: true
      service-url:
        defaultZone: http://eureka7002.com:7002/eureka,http://eureka7001.com:7001/eureka
    instance:
      instance-id: payment8002
  ```

  ![image-20221002091610325](D:\WorkSpace\Note\Picture\image-20221002091610325.png)

### 显示ip地址

![image-20221002091627556](D:\WorkSpace\Note\Picture\image-20221002091627556.png)

```yaml
eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://eureka7002.com:7002/eureka,http://eureka7001.com:7001/eureka
  instance:
    instance-id: payment8001
    prefer-ip-address: true
```

```yaml
eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://eureka7002.com:7002/eureka,http://eureka7001.com:7001/eureka
  instance:
    instance-id: payment8002
    prefer-ip-address: true
```

![image-20221002092017647](D:\WorkSpace\Note\Picture\image-20221002092017647.png)

## 服务发现Discovery

**对于注册进eureka里面的微服务，可以通过服务发现来后的该服务的信息**

```java
@Resource
private DiscoveryClient discoveryClient;    

@GetMapping("/payment/discovery")
public Object discovery() {
    List<String> services = discoveryClient.getServices();
    for (String service : services) {
        log.info(".......element:" + service);
    }
    List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
    for (ServiceInstance instance : instances) {
        log.info(instance.getServiceId() + "\t" + instance.getHost() + "\t" + instance.getPort() + "\t" + instance.getUri());
    }
    return this.discoveryClient;
}
```

```java
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient
public class CloudProviderPayment8001Application {

    public static void main(String[] args) {
        SpringApplication.run(CloudProviderPayment8001Application.class, args);
    }

}
```

**注入DiscoveryClient需要使用Resource**

![image-20221002093314003](D:\WorkSpace\Note\Picture\image-20221002093314003.png)

![image-20221002093356362](D:\WorkSpace\Note\Picture\image-20221002093356362.png)

## Eureka自我保护机制

#### 自我保护简介

![image-20221002093440773](D:\WorkSpace\Note\Picture\image-20221002093440773.png)

保护模式主要用于一组客户端与Eureka Server之间存在网络分区场景下的保护。一旦进入保护模式**Eureka Server将会尝试保护其服务注册表中的信息，不再删除服务注册表中的数据**

**某时刻某一个微服务不可用了，Eureka不会立即清除，一九会对该微服务的信息进行保存**

### 禁止自我保护

```yaml
eureka:
  instance:
    hostname: eureka7001.com
  client:
    register-with-eureka: false
    fetch-registry: false
    service-url:
      defaultZone: http://eureka7002.com:7002/eureka
  server:
    enable-self-preservation: false
```

![image-20221002094122424](D:\WorkSpace\Note\Picture\image-20221002094122424.png)

```yaml
eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://eureka7002.com:7002/eureka,http://eureka7001.com:7001/eureka
  instance:
    instance-id: payment8001
    prefer-ip-address: true
    # Eureka客户端向服务端发送心跳的时间间隔，单位为秒（默认30s）
    lease-renewal-interval-in-seconds: 1
    # Eureka服务端在收到最后一次心跳后等待时间上线，单位为秒（默认90s），超时删除服务
    lease-expiration-duration-in-seconds: 2
```

# 服务注册中心——Zookeeper

**Eureka停更不停用，可以使用Zookeeper代替**

### 配置Provider

* YAML文件

  ```yaml
  server:
    port: 8004
  
  spring:
    application:
      name: cloud-provider-payment
    cloud:
      zookeeper:
        connect-string: 192.168.32.129:2181
  ```

* 启动类

  ```java
  @SpringBootApplication
  @EnableDiscoveryClient
  public class CloudProviderPayment8004Application {
  
      public static void main(String[] args) {
          SpringApplication.run(CloudProviderPayment8004Application.class, args);
      }
  
  }
  ```

* controller

  ```java
  @RestController
  @Slf4j
  public class PaymentController {
      @Value("${server.port}")
      private String serverPort;
  
      @RequestMapping("/payment/zk")
      public String paymentZK() {
          return "SpringCloud with Zookeeper：" + serverPort + "\t" + UUID.randomUUID().toString();
      }
  }
  ```

![image-20221002101651389](D:\WorkSpace\Note\Picture\image-20221002101651389.png)

![image-20221002101748608](D:\WorkSpace\Note\Picture\image-20221002101748608.png)

![image-20221002101947637](D:\WorkSpace\Note\Picture\image-20221002101947637.png)

**服务在Zookeeper中属于临时节点**

**服务关闭后，Zookeeper将会删除服务**

### 配置Consumer

* YAML

  ```yaml
  server:
    port: 80
  
  spring:
    application:
      name: cloud-consumer-order
    cloud:
      zookeeper:
        connect-string: 192.168.32.129:2181
  ```

* 启动类

  ```java
  @SpringBootApplication
  @EnableDiscoveryClient
  public class CloudConsumerzkOrder80Application {
  
      public static void main(String[] args) {
          SpringApplication.run(CloudConsumerzkOrder80Application.class, args);
      }
  
  }
  ```

* Config

  ```java
  @Configuration
  public class ApplicationContextConfig {
      @Bean
      @LoadBalanced
      public RestTemplate getRestTemplate() {
          return new RestTemplate();
      }
  }
  ```

* Controller

  ```java
  @RestController
  @Slf4j
  public class OrderController {
      public static final String INVOKE_URL = "http://cloud-provider-payment";
  
      @Resource
      private RestTemplate restTemplate;
  
      @GetMapping("/consumer/payment/zk")
      public String paymentInfo() {
          return restTemplate.getForObject(INVOKE_URL + "/payment/zk", String.class);
      }
  }
  ```

  ![image-20221002105518899](D:\WorkSpace\Note\Picture\image-20221002105518899.png)

![image-20221002105527017](D:\WorkSpace\Note\Picture\image-20221002105527017.png)

# 服务注册中心——Consul

## Consul简介

**Consul是一套开源的分布式服务发现和配置管理系统**

* 服务发现——提供HTTP和DNS两种发现方式
* 健康检测——支持多种方式：HTTP、TCP、Docker、Shell脚本定制化
* KV存储——Key、Value的存储方式
* 多数据中心——Consul支持多数据中心
* 可视化Web界面

## 安装运行Consul

> Docker 启动

![image-20221002165241017](D:\WorkSpace\Note\Picture\image-20221002165241017.png)

## 配置Provider

* YAML

  ```yaml
  server:
    port: 8006
  
  spring:
    application:
      name: cloud-provider-payment
    cloud:
      consul:
        host: 192.168.32.129
        port: 8500
        discovery:
          service-name: ${spring.application.name}
          heartbeat:
            enabled: true
  ```

* 启动类

  ```java
  @SpringBootApplication
  @EnableDiscoveryClient
  public class CloudProviderPayment8006Application {
  
      public static void main(String[] args) {
          SpringApplication.run(CloudProviderPayment8006Application.class, args);
      }
  
  }
  ```

* controller

  ```java
  @RestController
  @Slf4j
  public class PaymentController {
      @Value("${server.port}")
      private String serverPort;
  
      @RequestMapping("/payment/consul")
      public String paymentConsul() {
          return "SpringCloud with consul: " + serverPort + "\t" + UUID.randomUUID().toString();
      }
  }
  ```

## 配置Consumer

* YAML

  ```yaml
  server:
    port: 80
  
  spring:
    application:
      name: cloud-consumer-order
    cloud:
      consul:
        host: 192.168.32.129
        port: 8500
        discovery:
          service-name: ${spring.application.name}
          heartbeat:
            enabled: true
  ```

* Config

  ```java
  @Configuration
  public class ApplicationContextConfig {
      @Bean
      @LoadBalanced
      public RestTemplate getRestTemplate() {
          return new RestTemplate();
      }
  }
  ```

* Controller

  ```java
  @RestController
  @Slf4j
  public class OrderController {
      public static final String INVOKE_URL = "http://cloud-provider-payment";
  
      @Resource
      private RestTemplate restTemplate;
  
      @GetMapping("/consumer/payment/consul")
      public String paymentInfo() {
          return restTemplate.getForObject(INVOKE_URL + "/payment/consul", String.class);
      }
  }
  ```

* 启动类

  ```java
  @SpringBootApplication
  @EnableDiscoveryClient
  public class CloudConsumerconsulOrder80Application {
  
      public static void main(String[] args) {
          SpringApplication.run(CloudConsumerconsulOrder80Application.class, args);
      }
  
  }
  ```

![image-20221002171332644](D:\WorkSpace\Note\Picture\image-20221002171332644.png)

# 三个注册中心的异同点

| 组件名    | 语言 | CAP  | 服务健康检查 | 对外暴露接口 | SpringCloud集成 |
| --------- | ---- | ---- | ------------ | ------------ | --------------- |
| Eureka    | Java | AP   | 可配支持     | HTTP         | 集成            |
| Consul    | Go   | CP   | 支持         | HTTP/DNS     | 集成            |
| Zookeeper | Java | CP   | 支持         | 客户端       | 集成            |

> CAP
>
> C：Consistency（强一致性）
>
> A：Availability（可用性）
>
> P：Partition tolerance（分区容错性）
>
> **最多只能同时较好的满足两个**
>
> CAP理论的核心是：**一个分布式系统不可能同时很好的满足一致性，可用性和分区容错性这三个需求**
>
> CA——单点集群，满足一致性，可用性的系统，通常在可扩展性上不太强大
>
> CP——满足一致性，分区容忍性的系统，通常性能不是特别高
>
> AP——满足可用性，分区容忍性的系统，通常对一致性要求低一些

# 服务调用——Ribbon

## Ribbon简介

SpringCloudRibbon是基于Netfix Ribbon实现的一套**客户端** **负载均衡工具**

主要功能是提供**客户端的软件负载均衡算法和服务调用**

**LB负载均衡**

将用户的请求平摊的分配到多个服务上，从而达到系统的HA（高可用）

### Ribbon与Nginx负载均衡区别

* Nginx是服务器负载均衡，客户端所有请求都会交给nginx，然后由nginx实现转发请求。即负载均衡是由服务端实现的
* Ribbon 本地负载均衡，在调用微服务接口时候，会在注册中心上获取注册信息服务列表之后缓存到JVM本地，从而在本地实现RPC远程服务调用技术

Ribbon属于进程内LB、Nginx属于集中式LB

### Ribbon工作流程

* 选择EurekaServer，他优先选择在用同一区域内负载较少的server
* 根据用户指定的策略，在从server取到的服务注册列表中选择一个地址

![image-20221002174705280](D:\WorkSpace\Note\Picture\image-20221002174705280.png)

**Eureka中带有Ribbon的依赖**

## RestTemplate使用

* xxxForObject()——返回对象为响应体中数据转化成的对象，基本上可以理解为json
* xxxForEntity()——返回对象为ResponseEntity对象，包含了响应中的一些重要信息（响应头、响应状态码、响应体等）

```java
@GetMapping("/consumer/payment/getEntity/{id}")
public CommonResult<Integer> getPayment2(@PathVariable("id") Long id) {
    ResponseEntity<CommonResult> entity = restTemplate.getForEntity(PAYMENT_URL + "/payment/get/" + id, CommonResult.class);
    if (entity.getStatusCode().is2xxSuccessful()) {
        return entity.getBody();
    } else {
        return new CommonResult<>(444, "操作失败");
    }
}
```

## Ribbon负载均衡规则

### 使用核心组件IRule

```java
public interface IRule {
    Server choose(Object var1);

    void setLoadBalancer(ILoadBalancer var1);

    ILoadBalancer getLoadBalancer();
}
```

![image-20221002182731239](D:\WorkSpace\Note\Picture\image-20221002182731239.png)

* RoundRobinRule——轮询
* RandomRule——随机
* RetryRule——先按照RoundRobinRule的策略获取服务，如果服务失败则在指定时间内会重试
* WeightedResponseTimeRule——对RoundRobinRule的扩展，相应速度越快的实例选择权重越大，越容易被选中
* BesetAvailableRule——会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务
* AvailablilityFilteringRule——先过滤故障实例，在选择并发较小的实例
* ZoneAvoidanceRule——默认规则，复合判断server所在区域的性能和server的可用性选择服务器

### Ribbon负载规则替换

> **官方提示**
>
> 这个自定义配置类不能放在@ComponentScan所扫描的当前包下一级子包下
>
> 否则自定的配置类就会被所有的Ribbon客户端共享，达不到特殊化定制的目的

### Ribbon默认负载均衡算法

**负载均衡算法：rest接口第几次请求数%服务器集群总数量=实际调用服务器位置下标，每次服务重启后rest接口计数从1开始**

![image-20221002184824806](D:\WorkSpace\Note\Picture\image-20221002184824806.png)

### 自定义Ribbon负载均衡规则

* 取消使用Ribbon提供的lb

  ```java
  @Configuration
  public class ApplicationContextConfig {
      @Bean
  //    @LoadBalanced
      public RestTemplate getRestTemplate() {
          return new RestTemplate();
      }
  }
  ```

* 自定义LoadBanlance接口

  ```java
  public interface LoadBalancer {
      ServiceInstance instances(List<ServiceInstance> serviceInstances);
  }
  ```

* 实现接口

  ```java
  @Component
  @Slf4j
  public class MyLoadBalancer implements LoadBalancer {
  
      private AtomicInteger atomicInteger = new AtomicInteger(0);
  
      public final int getAndIncrement() {
          int current;
          int next;
          do {
              current = this.atomicInteger.get();
              next = current == Integer.MAX_VALUE ? 0 : current + 1;
          } while (!this.atomicInteger.compareAndSet(current, next));
          log.info("访问次数: {}", next);
          return next;
      }
  
      @Override
      public ServiceInstance instances(List<ServiceInstance> serviceInstances) {
          int index = getAndIncrement() % serviceInstances.size();
          return serviceInstances.get(index);
      }
  }
  ```

> 自旋锁+CAS
>
> 一直循环直到，当前值等于所期望的值
>
> 防止其他进程抢先修改当前值的大小，出现线程安全问题

# 服务调用——OpenFeign

## OpenFeign简介

Feign是一个声明式WebService客户端，使用Feign能让编写WebService客户端更加简单

使用方法是**定义一个服务接口然后在上面添加注解**

Feign也支持可拔插式的编码器和解码器

SpringCloud对Feign进行了封装，使其支持了SpringMVC标准注解和HttpMessageConverters

Feign可以与Eureka和Ribbon组合使用以支持负载均衡

## 构建OpenFeign项目

* YAML

  ```yaml
  server:
    port: 80
  
  eureka:
    client:
      register-with-eureka: false
      service-url:
        defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka/
  ```

* 启动类

  ```java
  @SpringBootApplication
  @EnableFeignClients
  public class CloudConsumerFeignOrder80Application {
  
      public static void main(String[] args) {
          SpringApplication.run(CloudConsumerFeignOrder80Application.class, args);
      }
  
  }
  ```

* Service

  ```java
  @Component
  @FeignClient("CLOUD-PAYMENT-SERVICE")
  public interface PaymentFeignService {
      @GetMapping("/payment/get/{id}")
      CommonResult<Payment> getPaymentById(@PathVariable("id") Long id);
  }
  ```

* Controller

  ```java
  @RestController
  @Slf4j
  public class OrderFeignController {
      @Resource
      private PaymentFeignService paymentFeignService;
  
      @GetMapping("/consumer/payment/get/{id}")
      public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id) {
          return paymentFeignService.getPaymentById(id);
      }
  }
  ```

## OpenFeign超时控制

### 超时设置，模拟超时情况

```java
@GetMapping("/payment/feign/timeout")
public String paymentFeignTimeout() {
    try {
        Thread.sleep(3L);
    } catch (InterruptedException e) {
        throw new RuntimeException(e);
    }
    return serverPort;
}
```

### 超时控制

**OpenFeign默认等待1s，超时后报错**

OpenFeign的超时控制通过Ribbon进行控制

```yaml
ribbon:
  ReadTimeout: 5000
  ConnectTimeout: 5000
```

* 配置允许超时

## OpenFeign日志打印

* NONE：默认的，不显示任何日志
* BASIC：仅记录请求方法、URL、响应状态码及执行时间
* HEADERS：除了BASIC中定义的信息意外，还有请求和响应的头信息
* FULL：除了HEADERS中定义的信息之外，还有请求和响应的正文及元数据

```java
@Configuration
public class FeignConfig {
    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }
}
```

```yaml
logging:
  level:
    top.mnsx.springcloud.service.PaymentFeignService: debug
```

![image-20221003110916945](D:\WorkSpace\Note\Picture\image-20221003110916945.png)

# 服务降级——Hystrix

## Hystrix简介

Hystrix是一个用于处理分布式系统的延迟和容错的开源库

Hystrix能够保证在一个以来出问题的情况下，**不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性**

*断路器*本身是一种开关装置，再某个服务单元发生故障之后，通过断路器的故障监控，**向调用方返回一个符合预期、可处理的备选响应，而不是长时间的等待或者抛出调用方无法处理的异常**，这样避免了故障再分布式中蔓延，乃至雪崩

## Hystrix重要概念

* 服务降级——fallback
  * 程序运行异常
  * 超时
  * 服务熔断出发服务降级
  * 线程池/信号量打满也会导致服务降级
* 服务熔断——break
  * 服务降级——》进而熔断——恢复调用链路
* 服务限流——limit

## Hystrix项目搭建

* YAML

  ```yaml
  server:
    port: 8007
  
  spring:
    application:
      name: cloud-provider-payment
  
  eureka:
    client:
      register-with-eureka: true
      fetch-registry: true
      service-url:
        defaultZone: http://eureka7001.com:7001/eureka
  ```

* 启动类

  ```java
  @SpringBootApplication
  @EnableEurekaClient
  public class CloudProviderPayment8007Application {
  
      public static void main(String[] args) {
          SpringApplication.run(CloudProviderPayment8007Application.class, args);
      }
  
  }
  ```

* Service

  ```java
  @Service
  public class PaymentService {
      public String paymentInfo_ok(Integer id) {
          return "线程池" + Thread.currentThread().getName() + " paymentInfo_ok: " + id + "\t" + "0v0";
      }
  
      public String paymentInfo_timeout(Integer id) {
          try {
              Thread.sleep(3000L);
          } catch (InterruptedException e) {
              throw new RuntimeException(e);
          }
          return "线程池" + Thread.currentThread().getName() + " paymentInfo_ok: " + id + "\t" + "0n0" + " 耗时3s";
      }
  }
  ```

* Controller

  ```java
  @Slf4j
  @RestController
  public class PaymentController {
      @Autowired
      private PaymentService paymentService;
  
      @Value("${server.port}")
      private Integer serverPort;
  
      @GetMapping("/payment/hystrix/ok/{id}")
      public String paymentInfo_ok(@PathVariable Integer id) {
          String result = paymentService.paymentInfo_ok(id);
          log.info("-------------result:" + result);
          return result;
      }
  
      @GetMapping("/payment/hystrix/timeout/{id}")
      public String paymentInfo_timeout(@PathVariable Integer id) {
          String result = paymentService.paymentInfo_timeout(id);
          log.info("-------------result:" + result);
          return result;
      }
  }
  ```


## JMeter压力测试

![image-20221003145719352](D:\WorkSpace\Note\Picture\image-20221003145719352.png)

**Tomcat固定的线程数全部被占用，导致请求ok也变得延迟**

* 加入80进行压测

```yaml
server:
  port: 80

eureka:
  client:
    register-with-eureka: false
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka

ribbon:
  ReadTimeout: 5000
  ConnectTimeout: 5000
```

```java
@Component
@FeignClient("CLOUD-PROVIDER-PAYMENT")
public interface PaymentHystrixService {
    @GetMapping("/payment/hystrix/ok/{id}")
    public String paymentInfo_ok(@PathVariable("id") Integer id);

    @GetMapping("/payment/hystrix/timeout/{id}")
    public String paymentInfo_timeout(@PathVariable("id") Integer id);

```

```java
@RestController
@Slf4j
public class OrderHystrixController {
    @Autowired
    private PaymentHystrixService paymentHystrixService;

    @GetMapping("/consumer/payment/hystrix/ok/{id}")
    public String order_paymentInfo_ok(@PathVariable("id") Integer id) {
        return paymentHystrixService.paymentInfo_ok(id);
    }

    @GetMapping("/consumer/payment/hystrix/timeout/{id}")
    public String order_paymentInfo_timeout(@PathVariable("id") Integer id) {
        return paymentHystrixService.paymentInfo_timeout(id);
    }
}
```

```java
@SpringBootApplication
@EnableFeignClients
public class CloudConsumerFeignHystrixOrder80Application {

    public static void main(String[] args) {
        SpringApplication.run(CloudConsumerFeignHystrixOrder80Application.class, args);
    }

}
```

## 服务降级

**@HystrixCommand**

### 配置8001服务降级

```java
@Service
public class PaymentService {
    public String paymentInfo_ok(Integer id) {
        return "线程池" + Thread.currentThread().getName() + " paymentInfo_ok: " + id + "\t" + "0v0";
    }

    @HystrixCommand(fallbackMethod="paymentInfo_TimeoutHandler", commandProperties = {
            @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds", value="3000")
    })
    public String paymentInfo_timeout(Integer id) {
        int timeNumber = 5;
        // int age = 10 / 0;
        try {
            TimeUnit.SECONDS.sleep(timeNumber);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        return "线程池" + Thread.currentThread().getName() + " paymentInfo_ok: " + id + "\t" + "0v0耗时" + timeNumber + "s";
    }

    public String paymentInfo_TimeoutHandler(Integer id) {
        return "线程池" + Thread.currentThread().getName() + " paymentInfo_ok: " + id + "\t" + "0n0";
    }
}
```

![image-20221003154821812](D:\WorkSpace\Note\Picture\image-20221003154821812.png)

```java
@Service
public class PaymentService {
    public String paymentInfo_ok(Integer id) {
        return "线程池" + Thread.currentThread().getName() + " paymentInfo_ok: " + id + "\t" + "0v0";
    }

    @HystrixCommand(fallbackMethod="paymentInfo_TimeoutHandler", commandProperties = {
            @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds", value="3000")
    })
    public String paymentInfo_timeout(Integer id) {
//        int timeNumber = 5;
        int timeNumber = 3;
        int age = 10 / 0;
        try {
            TimeUnit.SECONDS.sleep(timeNumber);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        return "线程池" + Thread.currentThread().getName() + " paymentInfo_ok: " + id + "\t" + "0v0耗时" + timeNumber + "s";
    }

    public String paymentInfo_TimeoutHandler(Integer id) {
        return "线程池" + Thread.currentThread().getName() + " paymentInfo_ok: " + id + "\t" + "0n0";
    }
}
```

![image-20221003155251544](D:\WorkSpace\Note\Picture\image-20221003155251544.png)

**超时和异常都被Handler处理**

### 配置80消费降级

**常用配置服务端**

```yaml
server:
  port: 80

eureka:
  client:
    register-with-eureka: false
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka

ribbon:
  ReadTimeout: 5000
  ConnectTimeout: 5000

feign:
  hystrix:
    enabled: true
```

```java
@SpringBootApplication
@EnableFeignClients
@EnableHystrix
public class CloudConsumerFeignHystrixOrder80Application {

    public static void main(String[] args) {
        SpringApplication.run(CloudConsumerFeignHystrixOrder80Application.class, args);
    }

}
```

```java
@RestController
@Slf4j
public class OrderHystrixController {
    @Autowired
    private PaymentHystrixService paymentHystrixService;

    @GetMapping("/consumer/payment/hystrix/ok/{id}")
    public String order_paymentInfo_ok(@PathVariable("id") Integer id) {
        return paymentHystrixService.paymentInfo_ok(id);
    }

    @GetMapping("/consumer/payment/hystrix/timeout/{id}")
    @HystrixCommand(fallbackMethod = "paymentTimeoutFallbackMethod", commandProperties = {
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "1500")
    })
    public String order_paymentInfo_timeout(@PathVariable("id") Integer id) {
        return paymentHystrixService.paymentInfo_timeout(id);
    }

    public String paymentTimeoutFallbackMethod(@PathVariable("id") Integer id) {
        return "I am Mnsx_x, I'm busy, you know?";
    }
}
```

### 存在问题及解决方法

* 代码耦合，处理方法和业务方法都在一个类中
* 代码膨胀，每个方法都需要添加handler

**正常情况除了个别重要业务由专属的解决方法，其他普通的可以通过@DefaultProperties(defaultFallback = "")统一处理结果页面**

### 代码膨胀解决方法

**使用全局处理方法**

* 为类添加注解

```java
@DefaultProperties(defaultFallback = "paymentGlobalFallbackMethod")\
public class OrderHystrixController {
```

* 为方法添加注解，没有特别声明callback方法，就调用全局方法

```java
@GetMapping("/consumer/payment/hystrix/timeout/{id}")
/*@HystrixCommand(fallbackMethod = "paymentTimeoutFallbackMethod", commandProperties = {
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "1500")
    })*/
@HystrixCommand
public String order_paymentInfo_timeout(@PathVariable("id") Integer id) {
    int age = 10 / 0;
    return paymentHystrixService.paymentInfo_timeout(id);
}
```

### 代码耦合解决方法

**实现service方法，对实现类进行处理，实现解耦合**

```java
@Component
@FeignClient(value = "CLOUD-PROVIDER-PAYMENT", fallback = PaymentFallbackService.class)
public interface PaymentHystrixService {
    @GetMapping("/payment/hystrix/ok/{id}")
    public String paymentInfo_ok(@PathVariable("id") Integer id);

    @GetMapping("/payment/hystrix/timeout/{id}")
    public String paymentInfo_timeout(@PathVariable("id") Integer id);
}
```

* 实现类

```java
@Component
public class PaymentFallbackService implements PaymentHystrixService {
    @Override
    public String paymentInfo_ok(Integer id) {
        return "-----PaymentFallbackService fall back-paymentInfo_ok, 0n0";
    }

    @Override
    public String paymentInfo_timeout(Integer id) {
        return "-----PaymentFallbackService fall back-paymentInfo_timeout, 0n0";
    }
}
```

![image-20221003163509239](D:\WorkSpace\Note\Picture\image-20221003163509239.png)

## 服务熔断

### 熔断机制概述

应对雪崩效应的一种微服务链路保护机制，当扇出链路的某个微服务出错不可用或者响应时间太长，会进行服务的降级，进而熔断该节点微服务的调用，快速返回错误的响应信息

**当检测到该节点微服务调用响应正常后，恢复调用链路**

> 1. 调用失败会触发降级，而降级会调用fallback方法
> 2. 无论如何降级的流程一定会先调用正常方法再调用fallback方法
> 3. 假如单位时间内调用失败次数过多，也就是降级次数过多，则触发熔断

```java
// =======服务熔断

@HystrixCommand(fallbackMethod = "paymentCircuitBreaker_fallback", commandProperties = {
    @HystrixProperty(name = "circuitBreaker.enabled", value = "true"), // 是否开启断路器
    @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"), // 请求次数
    @HystrixProperty(name = "circuitBreak.sleepWindowInMilliseconds", value = "10000"), // 时间窗口期
    @HystrixProperty(name = "circuitBreak.errorThresholdPercentage", value = "60") // 失败率到达多少跳闸
})
public String paymentCircuitBreaker(@PathVariable("id") Integer id) {
    if (id < 0) {
        throw new RuntimeException("***id不能为负数");
    }
    String serialNumber = UUID.randomUUID().toString();
    return Thread.currentThread().getName() + "\t" + "调用成功，流水号:" + serialNumber;
}

public String paymentCircuitBreaker_fallback(@PathVariable("id") Integer id) {
    return "id 不能为负数， 请稍后再试， TnT id:" + id;
}
```

```java
@GetMapping
public String paymentCircuitBreaker(@PathVariable("id") Integer id) {
    String result = paymentService.paymentCircuitBreaker(id);
    log.info("***result: " + result);
    return result;
}
```

### 熔断类型

* 熔断打开——请求不在进行调用当前服务，内部设置时钟一般为MTTR（平均故障处理时间），当打开时长达到所设时钟则进入半熔断状态
* 熔断关闭——熔断关闭不会对服务进行熔断
* 熔断半开——部分请求根据规则调用当前服务，如果请求成功则复合规则则认为当前服务恢复正常，关闭熔断

![img](https://martinfowler.com/bliki/images/circuitBreaker/state.png)

### 参数分析

```java
@HystrixCommand(fallbackMethod = "paymentCircuitBreaker_fallback", commandProperties = {
    @HystrixProperty(name = "circuitBreaker.enabled", value = "true"), // 是否开启断路器
    @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"), // 请求次数
    @HystrixProperty(name = "circuitBreak.sleepWindowInMilliseconds", value = "10000"), // 时间窗口期
    @HystrixProperty(name = "circuitBreak.errorThresholdPercentage", value = "60") // 失败率到达多少跳闸
})
```

1. 快照时间窗：断路器确定是否需要打开需要统计一些请求和错误数据，而统计的时间范围就是快照时间窗，默认为最近的10s
2. 请求总数阈值：在快照时间窗内，必须满足请求总数阈值才有资格熔断。默认为20，意**味着在10s内，如果该hystrix命令的调用次数不足20次即使所有的请求都超时或其他原因失败，断路器都不会打开**
3. 错误百分比阈值：当请求总数在快照时间窗内超过了阈值，并且如果错误占比超过阈值，断路器就会打开，在默认设置下阈值为50

### 断路器关闭的条件

* 当满足一定的阈值时
* 当失败率到达一定阈值时
* 到达以上阈值，断路器将会开启
* 当开启时，所有请求都不会进行转发
* 一段时间后（默认是5s），这个时候断路器是半开状态，会让其中一个请求进行转发
* 如果成功短路器会关闭，若失败，继续开启，重复验证

### 熔断器恢复逻辑

> 当断路器打开，对主逻辑进行熔断之后，hystrix会启动一个休眠时间窗，在这个时间窗内，降级逻辑是临时的成为主逻辑，当休眠时间到期，断路器将进入半开状态，释放一次请求到原来的主逻辑上，如果此次请求正常返回，那么熔断器将继续闭合主逻辑恢复，如果这次请求依然有问题，断路器继续进入打开状态，休眠时间重新计时

### Web界面——hystrix Dashboard

1. 加入依赖

2. 添加注解

   ```java
   @SpringBootApplication
   @EnableHystrixDashboard
   public class CloudConsumerHystrixDashboard9001Application {
   
       public static void main(String[] args) {
           SpringApplication.run(CloudConsumerHystrixDashboard9001Application.class, args);
       }
   
   }
   ```

3. 访问localhost:9001/hystrix

   ![image-20221004083846368](D:\WorkSpace\Note\Picture\image-20221004083846368.png)

**被监控者必须要具备依赖Actuator**

* 8001

  ```java
  @SpringBootApplication
  @EnableEurekaClient
  @EnableCircuitBreaker
  public class CloudProviderPayment8007Application {
  
      public static void main(String[] args) {
          SpringApplication.run(CloudProviderPayment8007Application.class, args);
      }
  
      @Bean
      public ServletRegistrationBean getServlet() {
          HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
          ServletRegistrationBean registrationBean = new ServletRegistrationBean(streamServlet);
          registrationBean.setLoadOnStartup(1);
          registrationBean.addUrlMappings("/hystrix.stream");
          registrationBean.setName("HystrixMetricsStreamServlet");
          return registrationBean;
      }
  
  }
  ```

* 9001

  ```yaml
  server:
    port: 9001
  hystrix:
    dashboard:
      proxy-stream-allow-list: "localhost"
  ```

![image-20221004091058877](D:\WorkSpace\Note\Picture\image-20221004091058877.png)

# 服务网关——Gateway

## Gateway简介

Gateway旨在提供一种简单而有效的方式来对API进行路由，以及提供一些强大的过滤器功能

SpringCloud Gateway基于WebFlux框架实现的，而WebFlux框架底层则使用了高性能的Reactor模式通信框架Netty

**异步非阻塞**

![image-20221004095612070](D:\WorkSpace\Note\Picture\image-20221004095612070.png)

## Gateway三大核心

* Route（路由）——路由是构建网关的基本模块，它由ID，目标URI，一系列的断言和过滤器组成，如果断言为true则匹配该路由
* Predicate（断言）——参考的Java8的java.util.function.Predicate，开发人员可以匹配HTTP请求中的所有内容，如果请求与断言相匹配则进行路由
* Filter（过滤）——指的是Spring框架中GatewayFilter的实例，使用过滤器，可以在请求被路由前或者之后对请求进行修改

## 项目搭建

* YAML
  ```yaml
  server:
    port: 9527
  
  spring:
    application:
      name: cloud-gateway
    cloud:
      gateway:
        routes:
          - id: payment_routh
            uri: http://localhost:8001
            predicates:
              - path = /payment/get/**
  
          - id: payment_routh2
            uri: http://localhost:8001
            predicates:
              - path = /payment/discovery
  
  eureka:
    instance:
      hostname: cloud-gateway-service
    client:
      service-url:
        defaultZone: http://eureka7001.com/eureka
      register-with-eureka: true
      fetch-registry: true
  ```

* 启动类

  ```java
  @SpringBootApplication
  @EnableEurekaClient
  public class CloudGatewayGateway9527Application {
  
      public static void main(String[] args) {
          SpringApplication.run(CloudGatewayGateway9527Application.class, args);
      }
  
  }
  ```

![image-20221004103410661](D:\WorkSpace\Note\Picture\image-20221004103410661.png)

![image-20221004103445265](D:\WorkSpace\Note\Picture\image-20221004103445265.png)

## 硬编码配置Gateway

```java
@Configuration
public class GatewayConfig {
    @Bean
    public RouteLocator customRouterLocator(RouteLocatorBuilder routeLocatorBuilder) {
        RouteLocatorBuilder.Builder routes = routeLocatorBuilder.routes();
        routes.route("path_route_pornhub", r -> r.path("/ps").uri("http://baidu.com/guonei")).build();
        return routes.build();
    }
}
```

## 配置动态路由

```yaml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true

      routes:
        - id: payment_routh
#          uri: http://localhost:8001
          uri: lb://cloud-payment-service
          predicates:
            - Path=/payment/get/**

        - id: payment_routh2
#          uri: http://localhost:8001
          uri: lb://cloud-payment-service
          predicates:
            - Path=/payment/discovery

eureka:
  instance:
    hostname: cloud-gateway-service
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka
```

## Predicate使用

```yaml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true

      routes:
        - id: payment_routh
#          uri: http://localhost:8001
          uri: lb://cloud-payment-service
          predicates:
            - Path=/payment/get/**
            - After=2022-10-05T08:56:29.815+08:00[Asia/Shanghai]

        - id: payment_routh2
#          uri: http://localhost:8001
          uri: lb://cloud-payment-service
          predicates:
            - Path=/payment/discovery

eureka:
  instance:
    hostname: cloud-gateway-service
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka
```

![image-20221005090607564](D:\WorkSpace\Note\Picture\image-20221005090607564.png)

## Filter使用

**路由器过滤器可用于修改进入的HTTP请求和返回的HTTP响应，路由过滤器只能指定路由进行使用**

SpringCloudGateway内置多种路由过滤器，由GatewayFilter的工厂类产生

* 生命周期
  * pre
  * post
* 分类
  * GatewayFileter
  * GlobalFilterl

### 自定义过滤器

`implements GlobalFilter, Odered`

```java
@Component
@Slf4j
public class MyLogGatewayFilter implements GlobalFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.info("******come in MyLogGatewayFilter: " + new Date());
        String uname = exchange.getRequest().getQueryParams().getFirst("uname");
        if (uname == null) {
            log.info("******用户名为null，非法用户，滚啦😒");
            exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);
            return exchange.getResponse().setComplete();
        }
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return 0;
    }
}
```

# 服务配置——Config

## Config简介

SpringCloud Config为微服务架构中的微服务提供集中化的外部配置支持，**配置服务器为各个不同微服务应用的所有环境提供了一个中心化的外部配置**

Config分为客户端和服务端

* 功能
  * 集中管理配置文件
  * 不同环境不同配置，动态的配置更新，分环境比如dev/test/prod/beta/release
  * 运行期间动态调整配置，不需要再每个服务部署的机器上编写配置文件，服务会配置中心同意拉去配置自己的信息
  * 当配置发生变动时，服务不需要启动即可感知到配置的变化并应用新的配置
  * 将配置信息以REST接口的形式暴露

## Config服务端配置

```yaml
server:
  port: 3344

spring:
  application:
    name: cloud-config-center
  cloud:
    config:
      server:
        git:
          uri: https://github.com/Mnsx/springcloud-config.git
          search-paths:
            - springcloud-config
          username: Mnsx
          password: xkq771212XKQ
          skip-ssl-validation: true
      label: master

eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka
```

```java
@SpringBootApplication
@EnableConfigServer
public class CloudConfigCenter3344Application {

    public static void main(String[] args) {
        SpringApplication.run(CloudConfigCenter3344Application.class, args);
    }

}
```

<img src="D:\WorkSpace\Note\Picture\image-20221005105834860.png" alt="image-20221005105834860" style="zoom:200%;" />

## Config客户端配置

* bootstrap.yml是系统级的，优先级更高
* Spring Cloud会创建一个bootsrap context，作为Spring应用搞得Application Context的父上下文，**初始化时”Bootsrap Context“负责从外部源加载配置属性并解析配置，这两个上下文共享一个从外部的‘Environment’**

```java
@EnableEurekaClient
@SpringBootApplication
public class CloudConfigClient3355Application {

    public static void main(String[] args) {
        SpringApplication.run(CloudConfigClient3355Application.class, args);
    }

}
```

```yaml
server:
  port: 3355

spring:
  application:
    name: config-client
  cloud:
    config:
      label: master
      name: config
      profile: dev
      uri: http://localhost:3344

eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka
```

```java
@RestController
public class ConfigClientController {
    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/configInfo")
    public String getConfigInfo() {
        return configInfo;
    }
}
```

## 动态刷新配置

```yaml
server:
  port: 3355

spring:
  application:
    name: config-client
  cloud:
    config:
      label: master
      name: config
      profile: dev
      uri: http://localhost:3344

eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka

management:
  endpoints:
    web:
      exposure:
        include: "*"
```

```java
@EnableEurekaClient
@SpringBootApplication
@RefreshScope
public class CloudConfigClient3355Application {

    public static void main(String[] args) {
        SpringApplication.run(CloudConfigClient3355Application.class, args);
    }

}
```

```shell
curl -X POST "http://localhost:3355/actuator/refresh"
```

# 消息总线——Bus

## Bus简介

**Bus支持两种消息代理——RabbitMQ和Kafka**

SpringCloudBus能够管理和传播分布式系统间的消息，可以用于广播状态更改，事件推送等，也可以当作微服务间的通信通道

在微服务架构的系统中，通常使用**轻量级的消息代理**来构建一个**共用消息主题**，并且让系统中所有微服务实例都链接上来，**由于该主题中产生的消息会被所有实例监听和消费，所有称为消息总线**

## Bus动态刷新全局广播配置

```java
@RestController
public class ConfigClientController {
    @Value("${config.info}")
    private String configInfo;

    @Value("${server.port}")
    private String serverPort;

    @GetMapping("/configInfo")
    public String getConfigInfo() {
        return configInfo + ":" + serverPort;
    }
}
```

```java
@SpringBootApplication
@EnableEurekaClient
public class CloudConfigClient3366Application {

    public static void main(String[] args) {
        SpringApplication.run(CloudConfigClient3366Application.class, args);
    }

}
```

```yaml
server:
  port: 3366

spring:
  application:
    name: config-client
  cloud:
    config:
      label: master
      name: config
      profile: dev
      uri: http://localhost:3344

eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka

management:
  endpoints:
    web:
      exposure:
        include: "*"
```

## 动态刷新全局广播的设计思想

* 利用消息总线触发一个客户端/bus/refresh，而刷新所有客户端配置
* 利用消息总线触发一个服务端ConfigServer的/bus/refresh断电，而刷新所有客户配置

第一种的缺陷——

* 打破了微服务的直接单一性，因为微服务本身时业务模块，它本身不应该承担配置刷新的职责
* 破坏了微服务个点的对等性
* 有一定局限性

## 配置动态刷新

* Server

```yaml
server:
  port: 3344

spring:
  application:
    name: cloud-config-center
  cloud:
    config:
      server:
        git:
          uri: https://github.com/Mnsx/SpringCloud-StudyProject.git
          search-paths:
            - springcloud-config
          username: Mnsx
          password: xkq771212XKQ
          skip-ssl-validation: true
      label: master
  rabbitmq:
    host: 192.168.32.129
    port: 5672
    username: root
    password: 123123
    virtual-host: center

eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka

management:
  endpoints:
    web:
      exposure:
        include: 'bus-refresh'
```

* Client

```yaml
server:
  port: 3355

spring:
  application:
    name: config-client
  cloud:
    config:
      label: master
      name: config
      profile: dev
      uri: http://localhost:3344
  rabbitmq:
    host: 192.168.32.129
    port: 5672
    username: root
    password: 123123
    virtual-host: center

eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka

management:
  endpoints:
    web:
      exposure:
        include: "*"
```

## 动态刷新定点通知

```http
http://localhost:配置中心端口号/actuator/bus-refresh/{destination}
```

# 消息驱动——Stream（ 配置有问题暂时跳过）

## Stream简介

**屏蔽底层消息中间件的差异，降低切换成本，统一消息的编程模型**

* Binder——链接中间件，屏蔽差异
* Channel——通道，是队列Queue的一种抽象，在消息系统中就是实现存储和转发的媒介，通过Channel对队列进行配置
* Source和Sink——参照对象是SpringCloudStream本身，从Stream发布消息就是输出，接收消息就是输入

# 分布式请求链路跟踪——Sleuth

* Server

  ```yaml
  server:
    port: 80
  
  eureka:
    client:
      register-with-eureka: true
      fetch-registry: true
      service-url:
        defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
  
  spring:
    application:
      name: cloud-order-service
    zipkin:
      base-url: http://192.168.32.129:9411
    sleuth:
      sampler:
        probability: 1
  ```

* Client

  ```yaml
  server:
    port: 8001
  
  spring:
    application:
      name: cloud-payment-service
    zipkin:
      base-url: http://192.168.32.129:9411
    sleuth:
      sampler:
        probability: 1
    datasource:
      type: com.alibaba.druid.pool.DruidDataSource
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://192.168.32.129:3306/cloud?useUnicode=true?characterEncoding=utf-8&useSSL=false
      username: root
      password: 123123
  
  eureka:
    client:
      register-with-eureka: true
      fetch-registry: true
      service-url:
        defaultZone: http://eureka7002.com:7002/eureka,http://eureka7001.com:7001/eureka
    instance:
      instance-id: payment8001
      prefer-ip-address: true
      lease-renewal-interval-in-seconds: 1
      lease-expiration-duration-in-seconds: 2
  
  mybatis:
    mapper-locations: classpath:mapper/*.xml
    type-aliases-package: top.mnsx.springcloud.entity
  ```

# SpringCloudAlibaba

## 服务注册和配置中心——Nacos

### Nacos简介

**一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台**

### Nacos安装和运行

Docker 安装 Nacos/Nacos-Server

```shell
sudo docker run -d --name=nacos_test -p 8848:8848 -e MODE=standalone nacos/nacos-server:v1.4.4
```

![image-20221006124215993](D:\WorkSpace\Note\Picture\image-20221006124215993.png)

### 注册中心测试项目搭建

* 服务端

  ```java
  @EnableDiscoveryClient
  @SpringBootApplication
  @EnableFeignClients
  public class CloudAlibabaConsumerPayment80Application {
  
      public static void main(String[] args) {
          SpringApplication.run(CloudAlibabaConsumerPayment80Application.class, args);
      }
  
  }
  ```

  使用@EnableDiscoveryClient注解修饰主启动类

  ```yaml
  server:
    port: 80
  
  spring:
    application:
      name: nacos-order-consumer
    cloud:
      nacos:
        discovery:
          server-addr: 192.168.32.129:8848
  ```

  将服务注册到nacos中

* 客户端

  ```java
  @EnableDiscoveryClient
  @SpringBootApplication
  public class CloudAlibabaProviderPayment9001Application {
  
      public static void main(String[] args) {
          SpringApplication.run(CloudAlibabaProviderPayment9001Application.class, args);
      }
  
  }
  ```

  ```yaml
  server:
    port: 9001
  
  spring:
    application:
      name: nacos-payment-provider
    cloud:
      nacos:
        discovery:
          server-addr: 192.168.32.129:8848
  
  management:
    endpoints:
      web:
        exposure:
          include: '*'
  ```

### Nacos与其他注册中心的对比

![image-20221006132356888](D:\WorkSpace\Note\Picture\image-20221006132356888.png)

Nacos支持CP、AP切换

> C是要求所有节点在同一时间看到的数据是一致的
>
> A是要求所有请求都会受到响应

**切换方法**

```shell
curl -X PUT '$NACOS_SERVER:8848/nacos/v1/ns/operator/switches?entry=serverMode&value=CP'
```

### 配置中心测试项目搭建

* 分为boostrap.yml和application.yml

```yaml
# application.yml
spring:
  profiles:
    active: dev
```

```yaml
# bootstrap.yml
server:
  port: 3377

spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.32.129:8848
      config:
        server-addr: 192.168.32.129:8848
        file-extension: yml
```

* 主启动类标注@EnableDiscoveryClient
* 业务类——使用@RefreshScope

```java
@RestController
@RefreshScope
public class ConfigClientController {
    @Value("${config.info}")
    private String configInfo;

    @GetMapping
    public String getConfigInfo() {
        return configInfo;
    }
}
```

### Nacos配置匹配规则

在 Nacos Spring Cloud 中，`dataId` 的完整格式如下：

`${prefix}-${spring.profiles.active}.${file-extension}`

- `prefix` 默认为 `spring.application.name` 的值，也可以通过配置项 `spring.cloud.nacos.config.prefix`来配置。
- `spring.profiles.active` 即为当前环境对应的 profile， **注意：当 `spring.profiles.active` 为空时，对应的连接符 `-` 也将不存在，dataId 的拼接格式变成 `${prefix}.${file-extension}`**
- `file-exetension` 为配置内容的数据格式，可以通过配置项 `spring.cloud.nacos.config.file-extension` 来配置。目前只支持 `properties` 和 `yaml` 类型。

![image-20221007080914864](D:\WorkSpace\Note\Picture\image-20221007080914864.png)

### Nacos分类配置

**多环境多项目管理**

Namespace + Group + DataID

Namespace默认是public，Namespace主要用来实现隔离

Group默认是DEFAULT_GROUP，Group可以把不同的微服务划分到同一个分组中

* DataID配置

![image-20221007082910906](D:\WorkSpace\Note\Picture\image-20221007082910906.png)

* Group配置

![image-20221007083748822](D:\WorkSpace\Note\Picture\image-20221007083748822.png)

* Namespace配置

![image-20221007084549241](D:\WorkSpace\Note\Picture\image-20221007084549241.png)

### Nacos集群

* 集群架构

  ![image-20221007084944770](D:\WorkSpace\Note\Picture\image-20221007084944770.png)

  默认Nacos使用嵌入式数据库实现数据的存储，所以，如果启动多个默认配置下的Nacos节点，数据存储是存在一致性问题

  为了解决这个问题，Nacos采用集中式存储的方式来支持集群化部署，目前只支持MySQL存储

  > Docker starter.md

## 服务降级和服务熔断——Sentinel

```yaml
server:
  port: 8401

spring:
  application:
    name: cloud-alibaba-sentinel-service
  profiles:
    active: dev
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.32.71:8848
    sentinel:
      transport:
        dashboard: 192.168.32.60:8858
        port: 8719


management:
  endpoints:
    web:
      exposure:
        include: '*'
```

### Sentinel流控规则

* QPS

  ![image-20221008105754416](D:\WorkSpace\Note\Picture\image-20221008105754416.png)

  当一秒钟的点击量，超过设定的qps，那么限流

  ![image-20221008105849367](D:\WorkSpace\Note\Picture\image-20221008105849367.png)

* 线程数

  ![image-20221008111217997](D:\WorkSpace\Note\Picture\image-20221008111217997.png)

  一秒钟内访问的线程数，超过设定，那么限流

* 关联

  ![image-20221008112414399](D:\WorkSpace\Note\Picture\image-20221008112414399.png)

  当关联资源超过设定，那么进行限流

* 链路

* WarmUp

  **公式：阈值除以coldFactor（默认值为3），经过预热时长后才会达到阈值**

  ![image-20221008113553693](D:\WorkSpace\Note\Picture\image-20221008113553693.png)

  **一开始为阈值除以coldFactor，过了预热时间，逐渐恢复为阈值**

* 排队等待

  匀速排队，让请求以均匀的速度通过，阈值类型必须设置成QPS，否者失效

  设置含义：超过限定，那么久，排队，可以设置排队超时时间

  ![image-20221008114358912](D:\WorkSpace\Note\Picture\image-20221008114358912.png)

### Sentinel降级规则

* RT（平均响应时间，秒）

  平均响应时间 **超出阈值且在时间窗口内通过请求大于5**，触发降级

  窗口期过后关闭断路器

  RT最大为4900（更大需要通过 -Dcsp.sentinel.statistic.max.rt=xxx设置生效）

* 异常比例（秒）

  **QPS >= 5 且 异常比例超过阈值**，出发降级，时间窗口结束后，关闭降级

* 异常数（分）

  异常数超过阈值时，触发降级，时间窗口结束后，关闭降级

**没有半开状态**

* 慢调用比例

![image-20221008122026092](D:\WorkSpace\Note\Picture\image-20221008122026092.png)

**当单位统计时长内，请求数目大于设置的最小请求数，并且慢调用的比例大于阈值，则接下来熔断时间内请求会熔断，经过熔断时长后熔断器会进入探测恢复状态，若接下来的一个请求响应时间小于设置的最大 RT 则结束熔断，若大于设置的最大 RT 则会再次被熔断。**

* 异常比例

![image-20221008122524531](D:\WorkSpace\Note\Picture\image-20221008122524531.png)

* 异常数

![image-20221008122721098](D:\WorkSpace\Note\Picture\image-20221008122721098.png)

### Sentinel热点key限流

```java
@GetMapping("/testHotKey")
@SentinelResource(value = "testHotKey", blockHandler = "deadHandler")
public String testHotKey(@RequestParam(value = "p1", required = false) String p1,
                         @RequestParam(value = "p2", required = false) String p2) {
    return p1 + p2;
}

public String deadHandler(String p1, String p2, BlockException exception) {
    return "烂了";
}
```

![image-20221008124618010](D:\WorkSpace\Note\Picture\image-20221008124618010.png)

![image-20221008124901506](D:\WorkSpace\Note\Picture\image-20221008124901506.png)

* 参数列外项

![image-20221008125116761](D:\WorkSpace\Note\Picture\image-20221008125116761.png)

**允许设置特殊值，qps可以自定义**

![image-20221008125436345](D:\WorkSpace\Note\Picture\image-20221008125436345.png)

### Sentinel系统规则

![image-20221008125815739](D:\WorkSpace\Note\Picture\image-20221008125815739.png)

### @SentinelResource面临的问题及解决

 ```java
 @GetMapping("/testC")
 @SentinelResource(value = "testC", blockHandler = "handlerException", blockHandlerClass = CustomerBlockHandler.class)
 public String testC() {
     return "---testC";
 }
 
 public class CustomerBlockHandler {
     public static String handlerException(BlockException exception) {
         return "error";
     }
 }
 ```

### Sentinel服务熔断

* @SentinelResource只配置value

  ```java
  @RestController
  public class CircleBreakerController {
      @Value("${service-url.nacos-user-service}")
      public String SERVICE_URL;
  
      @Autowired
      private RestTemplate restTemplate;
  
      @RequestMapping("/consumer/fallback/{id}")
      @SentinelResource("fallback")
      public String fallback(@PathVariable Long id) {
          String result = restTemplate.getForObject(SERVICE_URL + "/paymentSQL/" + id, String.class, id);
  
          if (id == 4) {
              throw new RuntimeException("非法参数异常");
          } else if (result == null) {
              throw new RuntimeException("没有对应数据");
          }
  
          return result;
      }
  }
  ```

  ![image-20221010090936815](D:\WorkSpace\Note\Picture\image-20221010090936815.png)

* @SentinelResource配置fallback

  ```java
  @RestController
  public class CircleBreakerController {
      @Value("${service-url.nacos-user-service}")
      public String SERVICE_URL;
  
      @Autowired
      private RestTemplate restTemplate;
  
      @RequestMapping("/consumer/fallback/{id}")
      @SentinelResource(value = "fallback", fallback="handleFallback")
  
      public String fallback(@PathVariable Long id) {
          String result = restTemplate.getForObject(SERVICE_URL + "/paymentSQL/" + id, String.class, id);
  
          if (id == 4) {
              throw new RuntimeException("非法参数异常");
          } else if (result == null) {
              throw new RuntimeException("没有对应数据");
          }
  
          return result;
      }
  
      public String handleFallback(@PathVariable Long id, Throwable e) {
          Payment payment = new Payment(id, null);
          return "handlerFallback，exception：" + e.getMessage() + "data: " + payment;
      }
  }
  ```

  ![image-20221010091109927](D:\WorkSpace\Note\Picture\image-20221010091109927.png)

* @SentinelResource配置blockHandler

  ```java
  @RestController
  public class CircleBreakerController {
      @Value("${service-url.nacos-user-service}")
      public String SERVICE_URL;
  
      @Autowired
      private RestTemplate restTemplate;
  
      @RequestMapping("/consumer/fallback/{id}")
      @SentinelResource(value = "fallback", blockHandler="blockHandler")
      public String fallback(@PathVariable Long id) {
          String result = restTemplate.getForObject(SERVICE_URL + "/paymentSQL/" + id, String.class, id);
  
          if (id == 4) {
              throw new RuntimeException("非法参数异常");
          } else if (result == null) {
              throw new RuntimeException("没有对应数据");
          }
  
          return result;
      }
  
      public String blockHandler(@PathVariable Long id, BlockException blockException) {
          Payment payment = new Payment(id, "null");
          return "blockHandler-sentinel限流，blockException：" + blockException.getMessage() + "data: " + payment;
      }
  }
  ```

  ![image-20221010091313171](D:\WorkSpace\Note\Picture\image-20221010091313171.png)

  * 单次点击

    ![image-20221010091341089](D:\WorkSpace\Note\Picture\image-20221010091341089.png)

  * 超过5次请求

    ![image-20221010091524379](D:\WorkSpace\Note\Picture\image-20221010091524379.png)

* @SentinelResource同时配置fallback和blockHandler

  ```java
  @RestController
  public class CircleBreakerController {
      @Value("${service-url.nacos-user-service}")
      public String SERVICE_URL;
  
      @Autowired
      private RestTemplate restTemplate;
  
      @RequestMapping("/consumer/fallback/{id}")
      @SentinelResource(value="fallback", blockHandler = "blockHandler", fallback = "handleFallback")
      public String fallback(@PathVariable Long id) {
          String result = restTemplate.getForObject(SERVICE_URL + "/paymentSQL/" + id, String.class, id);
  
          if (id == 4) {
              throw new RuntimeException("非法参数异常");
          } else if (result == null) {
              throw new RuntimeException("没有对应数据");
          }
  
          return result;
      }
  
      public String handleFallback(@PathVariable Long id, Throwable e) {
          Payment payment = new Payment(id, null);
          return "handlerFallback，exception：" + e.getMessage() + "data: " + payment;
      }
  
      public String blockHandler(@PathVariable Long id, BlockException blockException) {
          Payment payment = new Payment(id, "null");
          return "blockHandler-sentinel限流，blockException：" + blockException.getMessage() + "data: " + payment;
      }
  }
  ```

  * 单次点击

    ![image-20221010091837257](D:\WorkSpace\Note\Picture\image-20221010091837257.png)

  * 多次点击

    ![image-20221010091856618](D:\WorkSpace\Note\Picture\image-20221010091856618.png)

* @SentinelResource配置exceptionToIgnore

  ```java
  @RestController
  public class CircleBreakerController {
      @Value("${service-url.nacos-user-service}")
      public String SERVICE_URL;
  
      @Autowired
      private RestTemplate restTemplate;
  
      @RequestMapping("/consumer/fallback/{id}")
      @SentinelResource(value = "fallback", blockHandler="blockHandler", exceptionsToIgnore = {RuntimeException.class})
      public String fallback(@PathVariable Long id) {
          String result = restTemplate.getForObject(SERVICE_URL + "/paymentSQL/" + id, String.class, id);
  
          if (id == 4) {
              throw new RuntimeException("非法参数异常");
          } else if (result == null) {
              throw new RuntimeException("没有对应数据");
          }
  
          return result;
      }
  
      public String handleFallback(@PathVariable Long id, Throwable e) {
          Payment payment = new Payment(id, null);
          return "handlerFallback，exception：" + e.getMessage() + "data: " + payment;
      }
  
      public String blockHandler(@PathVariable Long id, BlockException blockException) {
          Payment payment = new Payment(id, "null");
          return "blockHandler-sentinel限流，blockException：" + blockException.getMessage() + "data: " + payment;
      }
  }
  ```

  * 单次点击

    ![image-20221010092153295](D:\WorkSpace\Note\Picture\image-20221010092153295.png)

  * 多次点击

    ![image-20221010092159398](D:\WorkSpace\Note\Picture\image-20221010092159398.png)

### Sentinel持久化

**将限流配置规则持久化进Nacos保存，只要刷新8401某个rest地址，sentinel控制台的留空规则就能看到，只要Nacos中的配置不删除**

> 见MnsxUtils

## 分布式事务——Seata

### ID+三组件模型

* ID

  屈居唯一事务ID

* Transaction Coordinator(TC)

  事务协调器，维护全局事务，驱动全局事务提交或回滚

* Transaction Manager(TM)

  控制全局事务的范围：开始全局事务，提交或回滚全局事务

* Resource Manager(RM)

  控制分支事务处理的资源，与TC交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚

![image-20221010105322797](D:\WorkSpace\Note\Picture\image-20221010105322797.png)

1. TM向TC申请开启一个全局事务，全局事务创建成功并且生成一个全局唯一的XID
2. XID在微服务调用的链路的上下文中传播
3. RM向TM注册分支事务，将其纳入XID对应全局事务的管辖
4. TM向TC发起针对XID的全局提交或回滚决议
5. TC调度XID下管辖的所有分支事务完后提交或回滚
