# Ribbon负载均衡

1. Ribbon负载均衡规则

   * 规则接口是IRule
   * 默认实现是ZoneAvoidanceRule，根据zone选择服务列表然后轮询

2. 负载均衡自定义方式

   * 代码方式：配置灵活，但修改时需要重新打包发布

     ```java
     @Bean
     public IRule randomRule() {
         return new RandomRule();
     }
     ```

   * 配置方式：直观，方便，无需重新打包发布，但是无法做全局配置

     ```yaml
     userService: # 指定微服务的名称
       ribbon:
         NFLoadBalancerRuleClassName: com.alibaba.cloud.nacos.ribbon.NacosRule
     ```

3. 饥饿加载

   * 开启饥饿加载

   * 指定饥饿加载的微服务名称

     ```yaml
     ribbon:
       eager-load:
         enabled: true
         clients: userService # 指定开启的微服务名称
     ```

# Nacos注册中心

1. Nacos服务搭建

   > docker运行Nacos

2. Nacos服务注册或发现

   1. 引入nacos.discotry依赖

   2. 配置nacos地址

      ```yaml
      spring:
      	cloud:
          	nacos:
            		server-addr: 192.168.32.130:8848
      ```

## Nacos服务分级模型

1. Nacos服务分级存储模型

   1. 一级是服务
   2. 二级是集群
   3. 三级是实例

2. 设置实例的集群属性

   设置application.yml文件

   ```yaml
   spring:
       cloud:
           nacos:
             discovery:
               cluster-name: CD
   ```

## 集群负载均衡策略

1. NacosRule负载均衡策略
   1. 优先选择同集群服务实例列表
   2. 本地集群找不到提供者，采取其他集群寻找，并且会报警告
   3. 确定了可用实力列表后，再采用孙吉负载均衡挑选实例

## 加权负载均衡

1. 实例的权重控制
   1. Nacos控制台可以设置实例的权重值，0~1之间
   2. 同集群内的多个实例，权重越高被访问的频率越高
   3. 权重设置为0则完全不会被访问

## Nacos负载均衡策略

1. Nacos环境隔离
   1. 每个namespace都有唯一的Id
   2. 服务设置namespace时写Id而不是名称
   3. 不同namespace下的服务互相不可见

# Nacos配置管理

将配置交给Nacos管理的步骤

1. 在Nacos中添加配置文件

2. 在微服务中引入Nacos的config依赖

3. 在微服务中添加bootstrap.yml，配置nacos地址、当前环境、服务名称、文件后缀

   ```yaml
   spring:
     application:
       name: userService
     profiles:
       active: dev
     cloud:
       nacos:
         server-addr: 192.168.32.130:8848
         config:
           file-extension: yaml
   ```

Nacos配置更改后，微服务可以实现热部署，方式：

1. 通过@Value注解注入，结合@RefreshScope来刷新
2. 通过@ConfigurationProperties注入，自动刷新

注意事项：

* 不是所有的配置都适合放到配置中心，维护起来比较麻烦
* 建议将一些关键参数，需要运行时调整的参数放到nacos配置中心，一般都是自定义配置

微服务会从nacos读取的配置文件

1. [服务名]-[spring.profile.active].yaml，环境变量
2. [服务名].yaml，默认配置，多环境共享

优先级：

1 > 2 > 本地配置

# http客户端Feign

Feign的使用步骤

1. 引入依赖
2. 添加@EnableFeignClients注解
3. 编写FeignClient接口
4. 使用FeignClient中定义的方法替代RestTemplate

Feign的日志配置

1. 方式一是配置文件

   ```yaml
   feign:
     client:
       config:
         default:
           loggerLevel: HEADERS
   ```

2. 方式二是java代码设置Logger.Level这个Bean

   ```java
   @Bean
   public Logger.Level loggerLevel() {
       return Logger.Level.BASIC;
   }
   ```

   * 全局开启

     ```java
     @EnableFeignClients(defaultConfiguration = FeignConfig.class)
     ```

   * 指定服务开启

     ```java
     @FeignClient(value = "userService", configuration = FeignConfig.class)
     ```

Feign的优化：

1. 日志界别尽量用basic
2. 使用HttpClient或OKHttp代替URLConnection
   1. 引入feign-httpClient依赖
   2. 配置文件开启httpClient功能，设置连接池参数

Feign的最佳实践：

1. 让controller喝FeignClient继承同一个接口

2. 将FeignClient、POJO、Feign的默认配置都定义到一个项目中，供所有消费者使用

   不同包的FeignClient的导入方式：

   1. 在@EnableFeignClients注解中添加basePackages，指定FeignClient所在包
   2. 在@EnableFeignClients注解中添加clients，指定具体FeignClient的字节码

# 统一网关Gateway

网关的作用：

* 对用户请求做身份认证、权限校验
* 将用户请求路由到微服务，并实现负载均衡
* 对用户请求做限流

## 搭建网关服务

网关搭建步骤：

1. 创建项目，引入nacos服务发现和getway依赖

2. 配置application.yml，包括服务基本信息、nacos地址、路由

   ```yaml
   server:
     port: 10010
   spring:
     application:
       name: gateway
     cloud:
       nacos:
         discovery:
           server-addr: 192.168.32.130:8848
       gateway:
         routes:
           - id: user-service
             uri: lb://userservice
             predicates:
               - Path=/user/**
           - id: order-service
             uri: lb://orderservice
             predicates:
               - Path=/order/**
   ```

路由配置包括

1. 路由id：路由的唯一标识
2. 路由目标（uri）：路由的目标地址，http代表固定地址，lb代表根据服务名负载均衡
3. 路由断言（predicates）：判断路由的规则
4. 路由过滤器（filters）：对请求或响应的处理

## 路由断言工厂

* predicateFactory的作用

  读取用户定义的断言条件，对请求做出判断

* Path=/user/**的含义

  路径是以/user开头的就认为是符合的

## 过滤器工厂

* 过滤器的作用
  1. 对路由的请求或响应做加工处理
  2. 配置在路由下的过滤器只对当前路由的请求生效
* defaultFilters的作用
  1. 对所有路由都生效的过滤器

## 全局过滤器

* 全局过滤器的作用

  对所有路由都生效的过滤器，并且可以自定以处理逻辑

* 实现全局过滤器的步骤

  1. 实现GlobalFIlter接口

  2. 添加@Order注解或者实现Ordered接口

  3. 编写处理逻辑

     ```java
     @Order(-1)
     @Component
     public class AuthorizeFilter implements GlobalFilter {
         @Override
         public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
             // 获取请求参数
             ServerHttpRequest request = exchange.getRequest();
             MultiValueMap<String, String> queryParams = request.getQueryParams();
     
             // 读取参数中的authorization参数
             String authorization = queryParams.getFirst("authorization");
     
             // 判断数值是否等于admin
             if ("admin".equals(authorization)) {
                 // 是，放行
                 return chain.filter(exchange);
             }
             // 否，拦截
             exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
             return exchange.getResponse().setComplete();
         }
     }
     ```

## 过滤器执行顺序

* 路由过滤器，defaultFilter，全局过滤器的执行顺序
  1. order值越小，优先级越高
  2. 当order值一样，顺序是defaultFilter最先，然后局部的路由过滤器，最后是全局过滤器

## 跨域问题

```yaml
spring:
	cloud:
		gateway:
			globalcors:
				add-to-simple-url-handler-mapping: true
				corsConfigurations:
					'[/**]':
						allowedOrigins:
							- "http://localhost:8090"
                            - "http://www.legou.com"
                        allowedMethods:
                        	- "Get"
                        	- "POST"
                        	- "DELETE"
                        	- "PUT"
                        	- "OPTIONS"
                      	allowedHeaders: "*"
                      	allowCredentials: true
                      	maxAge: 360000
```
