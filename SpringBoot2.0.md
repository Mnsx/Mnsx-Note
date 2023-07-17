# 基础入门

## 依赖管理

```xml
<properties>
    <spring-boot.version>2.3.7.RELEASE</spring-boot.version>
</properties>
```

* 无需导入starter版本号
* 无需关注版本，有自动的版本仲裁
* 可以手动修改版本号

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>
```

## 自动配置

**引入starter就能够自动进行配置**

默认包结构——**能够自动进行包扫描获取组件**

所有配置都被设置了**默认值**，**最终都是映射到一个Properties类**

**按需加载**所有自动配置项，引入场景starter后才会开启

SpringBoot自动配置功能都放在**spring-boot-autoconfigure包**中

# 底层注解

## Configuration注解

* 配置类里面使用@Bean注解标注在方法上给容器注册组件，**默认是单例模式**

* 配置类也是组件

* @ProxyBeanMethods**代理Bean中的方法**，默认为true

  * true——由代理对象调用方法，**保持组件单实例**——FULL

    **配置类组件之间有依赖关系**

  * false——调用组件中的方法，**不再从容器中查找**——LITE

    **配置类组件之间没有依赖关系**

## Import注解

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Import {

	Class<?>[] value();

}
```

通过Import注解，**自动获取指定类型的组件并进行注入**

## Conditional注解

**条件装配**：满足Conditional指定条件，则进行组件注入

![image-20221018174540286](D:\WorkSpace\Note\Picture\image-20221018174540286.png)

## ImportResource注解

**导入XML文件作为组件，并存入容器中**

## ConfugurationProperties注解

```java
@Target({ ElementType.TYPE, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface ConfigurationProperties {

	@AliasFor("prefix")
	String value() default "";

	@AliasFor("value")
	String prefix() default "";

	boolean ignoreInvalidFields() default false;

	boolean ignoreUnknownFields() default true;

}
```

**只有在容器中才能够使用配置绑定**

```java
@Data
@Component
@ConfigurationProperties(prefix = "mycar")
public class Car {
    private String brand;
    private Integer price;
}
```

![image-20221019105931466](D:\WorkSpace\Note\Picture\image-20221019105931466.png)

```java
@Configuration
@EnableConfigurationProperties(Car.class)
public class MyConfig {
}
```

* 为配置绑定类添加@Component和@ConfigurationProperties
* 或者为绑定类条件@ConfigurationProperties方法，再在配置类中添加@EnableConfigurationProperties注解

# 自动配置

## 引导加载自动配置类

```java
//@SpringBootApplication
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @ComponentScan.Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
        @ComponentScan.Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public class SpringBootStudyApplication {
    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(SpringBootStudyApplication.class, args);
    }
}
```

* @SpringBootConfiguration

  ```java
  @Target(ElementType.TYPE)
  @Retention(RetentionPolicy.RUNTIME)
  @Documented
  @Configuration
  public @interface SpringBootConfiguration {
  
  	@AliasFor(annotation = Configuration.class)
  	boolean proxyBeanMethods() default true;
  
  }
  ```

  * **@Configuration代表当前是一个配置类**

* @ComponentScan指定扫描路径

* @EnableAutoConfiguration

  ```java
  @Target(ElementType.TYPE)
  @Retention(RetentionPolicy.RUNTIME)
  @Documented
  @Inherited
  @AutoConfigurationPackage
  @Import(AutoConfigurationImportSelector.class)
  public @interface EnableAutoConfiguration {
  
  	String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";
  
  	Class<?>[] exclude() default {};
  
  	String[] excludeName() default {};
  
  }
  ```

  * @AutoConfigurationPackage

    自动配置包

    ```java
    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @Inherited
    @Import(AutoConfigurationPackages.Registrar.class)
    public @interface AutoConfigurationPackage {
    
    	String[] basePackages() default {};
    
    	Class<?>[] basePackageClasses() default {};
    
    }
    ```

    * @Import(AutoConfigurationPackage.Registrar.class)

      ```java
      static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {
      
          @Override
          public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
              register(registry, new PackageImports(metadata).getPackageNames().toArray(new String[0]));
          }
      
          @Override
          public Set<Object> determineImports(AnnotationMetadata metadata) {
              return Collections.singleton(new PackageImports(metadata));
          }
      
      }
      ```

      **利用Registrar给容器批量添加组件**

      ![image-20221019111600303](D:\WorkSpace\Note\Picture\image-20221019111600303.png)

      **将指定一个包下的所有组件导入进入容器中**即main方法所在的包下

  * @Import(AutoConfigurationImportSelector.class)

    ```java
    @Override
    public String[] selectImports(AnnotationMetadata annotationMetadata) {
        if (!isEnabled(annotationMetadata)) {
            return NO_IMPORTS;
        }
        AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(annotationMetadata);
        return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
    }
    ```

    * getAutoConfigurationEntry()

      ```java
      protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
          if (!isEnabled(annotationMetadata)) {
              return EMPTY_ENTRY;
          }
          AnnotationAttributes attributes = getAttributes(annotationMetadata);
          // 获取备选的Confuguration组件
          List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
          configurations = removeDuplicates(configurations);
          Set<String> exclusions = getExclusions(annotationMetadata, attributes);
          checkExcludedClasses(configurations, exclusions);
          configurations.removeAll(exclusions);
          configurations = getConfigurationClassFilter().filter(configurations);
          fireAutoConfigurationImportEvents(configurations, exclusions);
          return new AutoConfigurationEntry(configurations, exclusions);
      }
      ```

      ![image-20221019112105457](D:\WorkSpace\Note\Picture\image-20221019112105457.png)

      **利用getCandidateConfigurations获取所有备选的Configuration组件**

      ```java
      protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
          List<String> configurations = SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),
                                                                               getBeanClassLoader());
          Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you "
                          + "are using a custom packaging, make sure that file is correct.");
          return configurations;
      }
      ```

      **利用工厂加载器得到所有组件**

      ```java
      public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
          String factoryTypeName = factoryType.getName();
          return loadSpringFactories(classLoader).getOrDefault(factoryTypeName, Collections.emptyList());
      }
      ```

      ```java
      private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
          MultiValueMap<String, String> result = cache.get(classLoader);
          if (result != null) {
              return result;
          }
      
          try {
              Enumeration<URL> urls = (classLoader != null ?
                                       classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
                                       ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
              result = new LinkedMultiValueMap<>();
              while (urls.hasMoreElements()) {
                  URL url = urls.nextElement();
                  UrlResource resource = new UrlResource(url);
                  Properties properties = PropertiesLoaderUtils.loadProperties(resource);
                  for (Map.Entry<?, ?> entry : properties.entrySet()) {
                      String factoryTypeName = ((String) entry.getKey()).trim();
                      for (String factoryImplementationName : StringUtils.commaDelimitedListToStringArray((String) entry.getValue())) {
                          result.add(factoryTypeName, factoryImplementationName.trim());
                      }
                  }
              }
              cache.put(classLoader, result);
              return result;
          }
          catch (IOException ex) {
              throw new IllegalArgumentException("Unable to load factories from location [" +
                                                 FACTORIES_RESOURCE_LOCATION + "]", ex);
          }
      }
      ```

      ![image-20221019112622970](D:\WorkSpace\Note\Picture\image-20221019112622970.png)

      从**META-INF/spring.factories**位置来加载一个文件，**默认扫描当前系统中所有的spring.factories**

      **spring-boot-autoconfigure-2.3.7.RELEASE.jar**

      ![image-20221019112816706](D:\WorkSpace\Note\Picture\image-20221019112816706.png)

      ![image-20221019112901776](D:\WorkSpace\Note\Picture\image-20221019112901776.png)

      **spring.factories中写死了**，Springboot启动后需要加载的组件

## 按需开启自动配置项

127个场景的所有自动配置，启动时都会默认全部加载，**但是最终会按需配置**

![image-20221019113207948](D:\WorkSpace\Note\Picture\image-20221019113207948.png)

**按照条件装配规则（@Conditional），最终按需配置**

> ```java
> @Bean
> @ConditionalOnBean(MultipartResolver.class) // 容器中有这个类型组件
> @ConditionalOnMissingBean(name = DispatcherServlet.MULTIPART_RESOLVER_BEAN_NAME) // 容器中没有这个名字的组件
> public MultipartResolver multipartResolver(MultipartResolver resolver) {
>    // Detect if the user has created a MultipartResolver but named it incorrectly
>    return resolver;
> }
> ```
>
> 防止用户配置的文件上传解析器不符合规范

SpringBoot默认会在底层配置所有组件，**如果用户配置了，以用户的优先**

![image-20221019114452372](D:\WorkSpace\Note\Picture\image-20221019114452372.png)

**自动配置类会和xxxproperties绑定**



