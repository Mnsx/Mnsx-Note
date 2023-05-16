# Swagger简介

**前后端分离**

Vue + SpringBoot

* 后端时代：前端只用管理静态页面 html --> 后端 模板引擎 JSP --> 后端主力
* 前后端分离时代
    * 后端：后端控制层，服务层，数据访问层
    * 前端：前端控制层，视图层
* 前后端通过接口交互
* 前后端相对独立，松耦合
* 前后端甚至可以部署在不同的服务器上
* 前后端分离：
    * 前端测试后端接口：postman
    * 后端提供接口，需要实时更新最新的额消息以及改动

# Swagger

* 号称世界上最流行的API框架
* RestFul Api文档在线自动生成工具—>Api文档与API自定义同步更新
* 直接运行，可以在线测试API接口
* 支持多种语言

在项目中使用Swagger需要SpringBox

* swagger2
* ui

# SpringBoot继承Swagger

**重要问题**——使用2.6.x的springboot配置3.0.0的swagger2时提示

> Exception encountered during context initialization - cancelling refresh attempt: org.springframework.context.ApplicationContextException: Failed to start bean 'documentationPluginsBootstrapper'; nested exception is java.lang.NullPointerException

```yaml
spring:
	mvc:
		pathmatch:
			matching-strategy: ant_path_matcher
```

1. 新建一个SpringBoot-web项目

2. 导入相关依赖

   ```xml
   		<dependency>
               <groupId>io.springfox</groupId>
               <artifactId>springfox-swagger2</artifactId>
               <version>3.0.0</version>
           </dependency>
           <dependency>
               <groupId>io.springfox</groupId>
               <artifactId>springfox-swagger-ui</artifactId>
               <version>3.0.0</version>
           </dependency>
   ```

3. 编写一个Hello工程

4. 配置Swagger-》config

   ```java
   @Configuration
   @EnableSwagger2 // 开启Swagger2
   public class SwaggerConfig {
   }
   ```

5. 测试运行

   ![Swagger-ui](..\Picture\Swagger\Swagger-ui.png)

# 配置Swagger

## Swagger的bean实例Docket

```java
@Configuration
@EnableSwagger2 // 开启Swagger2
public class SwaggerConfig {

    // 配置Swagger2的Bean实例
    @Bean
    public Docket docket() {

        return new Docket(DocumentationType.SWAGGER_2).apiInfo(apiInfo());
    }

    // 配置Swagger信息-apiInfo
    @Bean
    public ApiInfo apiInfo() {

        //作者信息
        Contact contact = new Contact("Mnsx_x", "https://mnsx.github.io", "xx1527030652@gmail.com");
        return new ApiInfo(
                "Mnsx_x Swagger",
                "xxx项目Api文档",
                "1.4.0",
                "https://github.com/xxx",
                contact,
                "Apache 2.0",
                "http://www.apache.org/licenses/LICENSE-2.0",
                new ArrayList());
    }
}
```

## Swagger配置扫描接口

```java
// 配置Swagger2的Bean实例
    @Bean
    public Docket docket() {

        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                // 配置扫描接口的方式
                // basePackage：指定扫描的包
                // any()：扫描全波
                // none()：都不扫描
                // withClassAnnotation：扫描类上的注解
                // withMethodAnnotation：扫描方法上的注解
                .apis(RequestHandlerSelectors.basePackage("top.mnsx.swagger.controller"))
                // paths()：过滤路径
                .paths(PathSelectors.any())
                .build();
    }
```

## 配置启动Swagger

```java
// enable是否启动Swagger，如果为false，则Swagger不能在浏览器中访问呢
    // 配置Swagger2的Bean实例
    @Bean
    public Docket docket() {

        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .enable(true)
                .select()
                // 配置扫描接口的方式
                // basePackage：指定扫描的包
                // any()：扫描全波
                // none()：都不扫描
                // withClassAnnotation：扫描类上的注解
                // withMethodAnnotation：扫描方法上的注解
                .apis(RequestHandlerSelectors.basePackage("top.mnsx.swagger.controller"))
                // paths()：过滤路径
                .paths(PathSelectors.any())
                .build();
    }
```

## 配置Swagger针对生产环境中使用

```java
// enable是否启动Swagger，如果为false，则Swagger不能在浏览器中访问呢
    // 配置Swagger2的Bean实例
    @Bean
    public Docket docket(Environment environment) {

        // 设置要显示Swagger的环境
        Profiles profiles = Profiles.of("dev", "test");
        // 获取项目环境
        boolean b = environment.acceptsProfiles(profiles);

        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .enable(b)
                .select()
                // 配置扫描接口的方式
                // basePackage：指定扫描的包
                // any()：扫描全波
                // none()：都不扫描
                // withClassAnnotation：扫描类上的注解
                // withMethodAnnotation：扫描方法上的注解
                .apis(RequestHandlerSelectors.basePackage("top.mnsx.swagger.controller"))
                // paths()：过滤路径
                .paths(PathSelectors.any())
                .build();
    }
```

## 配置Api文档分组

```java
.groupName("mnsx")
```

使用多个docket即可产生多个分组

## 配置实体类

```java
//@Api(注释）
@ApiModel("用户实体类")
public class User {
    @ApiModelProperty("用户名")
    public String username;
    @ApiModelProperty("密码")
    public String password;
}
```

# 总结

1. 我们可以通过Swagger给一些比较难理解的属性或者接口，增加注释信息
2. 接口文档实时更新
3. 可以在线测试

**在正式发布的时候，关闭swagger，处于安全考虑，而且节省运行的内存**

# 使用swagger-spring-boot-starter

1. 添加依赖

   ```xml
           <!-- swagger -->
           <dependency>
               <groupId>com.battcn</groupId>
               <artifactId>swagger-spring-boot-starter</artifactId>
               <version>2.1.5-RELEASE</version>
           </dependency>
   ```

2. 添加配置

   ```yaml
   # 开发环境S
   spring:
     # swagger 配置
     swagger:
       enabled: true
       title: '后台API接口开发文档'
       description: 'API开发文档'
       version: 1.4.1
       contact:
         name: 'Mnsx_x'
         url: 'https://github.com/Mnsx'
         email: 'xx1527030652@gmail.com'
         
   spring:
     mvc:
       pathmatch:
         matching-strategy: ant_path_matcher
   ```

   