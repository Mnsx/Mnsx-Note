# Spring IoC

## 常见Spring容器

* **BeanFactory接口**

  Spring容器中具有代表性的容器就是BeanFactory接口，这个是spring容器的**顶层接口**，提供了容器最基本的功能

  ```java
  org.springframework.beans.factory.BeanFactory
  
  //按bean的id或者别名查找容器中的bean
  Object getBean(String name) throws BeansException
  //这个是一个泛型方法，按照bean的id或者别名查找指定类型的bean，返回指定类型的bean对象
  <T> T getBean(String name, Class<T> requiredType) throws BeansException;
  //返回容器中指定类型的bean对象
  <T> T getBean(Class<T> requiredType) throws BeansException;
  //获取指定类型bean对象的获取器
  <T> ObjectProvider<T> getBeanProvider(Class<T> requiredType);
  ```

* **ApplicationContext接口**

  ```java
  org.springframework.context.ApplicationContext
  ```

  这个接口继承了BeanFactory接口，所以内部包含了BeanFactory所有的功能，并且在其上进行了扩展，增加了很多企业级的功能

* **ClassPathXmlApplicationContext类**

  ```java
  org.springframework.context.support.ClassPathXmlApplicationContext
  ```

  这个类实现了ApplicationContext接口，类名包含了ClassPath、Xml，说明这个类可以从classPath中加载bean的XML配置文件，然后创建bean对象

* **AnnotationConfigApplicationContext类**

  ```java
  org.springframework.context.annotation.AnnotationConfigApplicationContext
  ```

  这个类也实现了ApplicationContext接口，类名包含了Annotation和Config，可以看出这个类内部会解析注解来构建和管理需要的bean

## 容器创建bean实例的方法

* **通过反射调用构造方法创建bean对象**

  调用类的构造方法获取对应的bean实例，是使用最多的方式，**spring容器内部会自动调用该类型的构造方法来创建bean对象**，将其放在容器中以供使用

* **通过静态工厂方法创建bean对象**

  @Bean标注静态方法，Bean对象作为方法返回值，这个对象将被加入Spring容器中

* **通过实例工厂方法创建bean对象**

  @Bean标注一个普通方法，类似静态工厂

* **通过FactoryBean来创建Bean对象**

  **从Spring3.0开始，支持泛型FactoryBean**

  ```java
  rg.springframework.beans.factory.FactoryBean
  
  public interface FactoryBean<T> {
      /**
       * 返回创建好的对象
       */
      @Nullable
      T getObject() throws Exception;
      /**
       * 返回需要创建的对象的类型
       */
      @Nullable
      Class<?> getObjectType();
      /**
      * bean是否是单例的
      **/
      default boolean isSingleton() {
          return true;
      }
  }
  ```

## Bean作用域scope

```java
org.springframework.context.annotation.Scope

@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Scope {

    /**
     * 选择作用域类型
     */
	@AliasFor("scopeName")
	String value() default "";
	@AliasFor("value")
	String scopeName() default "";

	/**
	 * 指定否给Bean配置为代理作用域
	 * ScopedProxyMode有四个值，DEFAULT、NO、INTERFACES、TARGET_CLASS
	 * INTERFACES——>指定JDK代理
	 * TARGET_CLASS-->指定CGLIB代理
	 * 给Bean创建代理对象，调用代理对象的任何方法，都会通过Scope中的get方法获取bean对象
	 */
	ScopedProxyMode proxyMode() default ScopedProxyMode.DEFAULT;

}
```

* **Singleton**

  当scope的值设置为singleton时，**整个Spring容器中只会存在一个bean实例**，Singleton是scope的**默认值**

  Singleton实例在**容器启动过程中就创建完毕**

  单例Bean是整个应用共享的，所有**需要考虑到线程安全问题**

* **Prototype**

  如果scope被设置为prototype类型，表示这个bean是多例的

  多例Bean**每次获取的时候都会重新创建**，如果这个Bean比较复杂，创建时间比较长，会影响系统的性能

* **Request**

  如果scope被设置为request，表示在一次http请求中，一个bean对应一个实例

  request作用域用于在Spring容器的web环境，Spring中有个Web容器接口WebApplicationContext，这里面对Request作用域提供了支持

* **Session**

  和request相似，session级别共享bean，每个会话对应一个bean实例

* **Application**

  在Web环境中使用，一个Web应用程序对应一个Bean实例

  **通常情况与Singleton效果类似，但是一个应用程序中可以创建多个spring容器，不同的容器中可以存在同名的bean**

* **自定义scope**

  ```java
  org.springframework.beans.factory.config.Scope
  
  public interface Scope {
  
      /**
       * 返回当前作用域中name对应的bean对象
       * name：需要检索的bean的名称
       * objectFactory：如果name对应的bean在当前操作域中没有找到，那么可以调用这个objectFactory来创建这个对象
       */
  	Object get(String name, ObjectFactory<?> objectFactory);
  
  	/**
  	 * 将name对应的bean从当前作用域中删除
  	 */
  	@Nullable
  	Object remove(String name);
  
  	/**
  	 * 用于注册注销回调，如果想要销毁对应的对象，则由Spring容器注册相应的销毁回调，而由自定义作用域选择是不是要销毁对应的对象
  	 */
  	void registerDestructionCallback(String name, Runnable callback);
  
  	/**
  	 * 用于解析相应的上下文数据
  	 */
  	@Nullable
  	Object resolveContextualObject(String key);
  
  	/**
  	 * 作用域的会话标识
  	 */
  	@Nullable
  	String getConversationId();
  
  }
  ```

  将自定义的scope注册到容器，需要调用registerScope方法

  ```java
  org.springframework.beans.factory.config.ConfigurableBeanFactory#registerScope
      
  /**
   * 向容器中注册自定义的Scope
   * scopeName：作用域名称
   * scope：作用域对象
  **/
  void registerScope(String scopeName, Scope scope);
  ```

  **或者通过创建CustomScopeConfigurer类型的Bean，Bean对象调用setScope**

  ```java
  @Bean
  public CustomScopeConfigurer customScopeConfigurer() {
      CustomScopeConfigurer customScopeConfigurer = new CustomScopeConfigurer();
  
      Map<String, Object> map = new HashMap<>();
      map.put("custom", new CustomScope());
  
      customScopeConfigurer.setScopes(map);
      return customScopeConfigurer;
  }
  ```

  **案例**

  实现线程级别的作用域

  ```java
  @Component
  public class ThreadScope implements Scope {
  
      public static final String THREAD_SCOPE = "thread";
  
      private ThreadLocal<Map<String, Object>> beanMap = ThreadLocal.withInitial(HashMap::new);
  
      @Override
      public Object get(String name, ObjectFactory<?> objectFactory) {
          Object bean = beanMap.get().get(name);
          if (Objects.isNull(bean)) {
              bean = objectFactory.getObject();
              beanMap.get().put(name, bean);
          }
          return bean;
      }
  
      @Override
      public Object remove(String name) {
          return this.beanMap.get().remove(name);
      }
  
      @Override
      public void registerDestructionCallback(String name, Runnable callback) {
          // bean作用域范围结束的时候调用这个方法，用于bean清理
      }
  
      @Override
      public Object resolveContextualObject(String key) {
          return null;
      }
  
      @Override
      public String getConversationId() {
          return Thread.currentThread().getName();
      }
  }
  ```

## @DepndOn作用

**通过depend-on可以干预bean创建和销毁顺序**

创建顺序与销毁顺序刚好相反

## @Primary作用

**通过类型自动注入依赖时，使用@Primary修饰的Bean，能够优先被注入**

## @Lazy作用

**使用@Lazy注解可以将Bean对象的创建延迟到第一次使用Bean的时候**

## 单例Bean中使用多例Bean

> 单例BeanA中依赖多例BeanB，但是**调用单例BeanA中的BeanB总是同一个**，如何在BeanA中每次调用的BeanB都是新的？

在BeanA中，主动从容器中获取BeanB，**实现接口ApplicationContextAware**可以在Bean中获取容器

```java
org.springframework.context.ApplicationContextAware
 
public interface ApplicationContextAware extends Aware {
    void setApplicationContext(ApplicationContext applicationContext) throws BeansException;
}
```

## Java注解详解

### 注解参数注意事项

* 访问修饰符**必须为public**，不写默认为public
* 该元素的类型只能是**基本数据类型、String、Class、枚举类型、注解类型、上述类型的数组**
* 该元素名称一般定义为名词，**如果注解中只有一个元素，将名字起为value**
* 参数名称后面的**()**不是定义方法参数的地方，不能在括号中定义任何参数，**仅仅是一个特殊的语法**
* **default代表默认值**
* 没有默认值，代表后续使用注解时必须给该类型元素赋值

### @Targe指定注解使用范围

不主动声明注解的使用范围的话，**默认标配是可以用在任何地方**

```java
java.lang.annotation.Target

@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Target {
    ElementType[] value();
}
```

参数ElementType是一个枚举类

```java
java.lang.annotation.ElementType
    
public enum ElementType {
    /*类、接口、枚举、注解上面*/
    TYPE,
    /*字段上*/
    FIELD,
    /*方法上*/
    METHOD,
    /*方法的参数上*/
    PARAMETER,
    /*构造函数上*/
    CONSTRUCTOR,
    /*本地变量上*/
    LOCAL_VARIABLE,
    /*注解上*/
    ANNOTATION_TYPE,
    /*包上*/
    PACKAGE,
    /*类型参数上*/
    TYPE_PARAMETER,
    /*类型名称上*/
    TYPE_USE
}
```

> **TYPE_PARAMETER**
>
> 表示被修饰的注解，可以被使用在自定义类型参数的声明语句上
>
> **TYPE_USE**
>
> 表示被修饰的注解，可以被使用在使用类型的任何语句上

### @Retention指定注解保留策略

```java
java.lang.annotation.Retention

@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.ANNOTATION_TYPE})
public @interface Retention {
    RetentionPolicy value();
}
```

注解参数为RetentionPolicy枚举类

```java
java.lang.annotation.RetentionPolicy

    public enum RetentionPolicy {
    /*注解只保留在源码中，编译为字节码之后就丢失了，也就是class文件中就不存在了*/
    SOURCE,
    /*注解只保留在源码和字节码中，运行阶段会丢失*/
    CLASS,
    /*源码、字节码、运行期间都存在*/
    RUNTIME
        private RetentionPolicy() {
    }
}
```

### @Inherited实现类之间的注解继承

让子类可以继承父类被@Inherit修饰的注解，**接口和实现类无法使用**

```java
java.lang.annotation.Inherited

@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.ANNOTATION_TYPE})
public @interface Inherited {
}
```

### @Repeatable重复使用注解

```java
java.lang.annotation.Repeatable

@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.ANNOTATION_TYPE})
public @interface Repeatable {
    Class<? extends Annotation> value();
}
```

**重复使用注解两种方式**

1. 使用@Repeatable后直接修饰

2. 创建一个容器注解，来存放子注解

   **容器注解中必须有个value类型的参数，参数类型为子注解类型的数组**

### Spring注解@AliasFfor

@AliasFor可以为注解的属性指定别名属性

```java
org.springframework.core.annotation.AliasFor

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Documented
public @interface AliasFor {

	@AliasFor("attribute")
	String value() default "";

	@AliasFor("value")
	String attribute() default "";

	Class<? extends Annotation> annotation() default Annotation.class;

}
```

@Alias如果不指定annotation参数，那么**默认指定当前注解**

**value和attribute互为别名**，两者将被设置相同的值

如果@AliasFor不指定value和attribute，那么自动设置修饰的属性作为其值

> **问题记录**
>
> 想要通过反射获取通过@AliasFor修改的属性值失败
>
> **因为@AliasFor是Spring的特殊注解，不是Java原生支持，因此要使用Spring的工具类取值**
>
> ```java
> AnnotatedElementUtils.getMergedAnnotation(目标对象.class, 目标注解.class)
> ```

## @Configuration注解

使用@Configuration修饰类，这个类将作为bean.xml文件

```java
org.springframework.cglib.core.Configuration

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Configuration {

	@AliasFor(annotation = Component.class)
	String value() default "";

	boolean proxyBeanMethods() default true;

}
```

> 不添加@Configuration区别
>
> 1. 有没有@Configuration，@Bean都会生效，只需要将配置类放入Spring容器中即可
> 2. @Configuration修饰的bean会被CGLIB处理，名称带有EnhancerBySpringCGLIB，会变成一个代理对象
> 3. 被CGLIB代理的配置bean，会拦截@Bean修饰的方法，产生的都是同一个Bean

## @Bean注解

类似于bean.xml中的bean标签，用来注册bean

@Bean修饰在方法上，通过方法定义bean，**默认将方法名作为bean名称**，方法返回bean对象

```java
org.springframework.context.annotation.Bean

@Target({ElementType.METHOD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Bean {

    /**
     * Bean名称属性
     */
	@AliasFor("name")
	String[] value() default {};
	@AliasFor("value")
	String[] name() default {};

	@Deprecated
	Autowire autowire() default Autowire.NO;

    /**
     * 按照类型注入Bean时，如果存在两个就会报错
     * 将autowireCandidate设置为false，这个bean，将不会作为候选bean
     */
	boolean autowireCandidate() default true;
    
    /**
     * bean的初始化方法
     */
    String initMethod() default "";

    /**
     * bean的销毁方法
     */
	String destroyMethod() default AbstractBeanDefinition.INFER_METHOD;

}
```

## @Import注解

```java
org.springframework.context.annotation.Import

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Import {

	Class<?>[] value();

}
```

### value参数常见用法

* **value为普通类**

  将value指定的类加入到Spring容器中

* **value为@Configuration标注的类**

  将指定的配置类中的Bean加入到Spring容器中

* **value为@ComponentScan标注的类**

  将@ComponentScan能够扫描的类添加到Spring容器中
  
* **value为ImportBeanDefinitionRegistrar接口类型**

  这个接口提供向Spring容器添加Bean的API方式

  ```java
  org.springframework.context.annotation.ImportBeanDefinitionRegistrar
  
  public interface ImportBeanDefinitionRegistrar {
  
      /**
       * AnnotationMetadata 通过这个可以获取@Import修饰的类的所有注解的信息
       * BeanDefinitionRegistry 提供了注册bean的各种方法的接口
       * BeanNameGenerator Bean名称生成器
       */
  	default void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry,
  			BeanNameGenerator importBeanNameGenerator) {
  
  		registerBeanDefinitions(importingClassMetadata, registry);
  	}
  
  	default void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
  	}
  
  }
  ```

  * **AnnotationMetadata类**

    ```java
    org.springframework.core.type.AnnotationMetadata
    
    public interface AnnotationMetadata extends ClassMetadata, AnnotatedTypeMetadata {
    
    	/**
    	 * 获取基础类上存在的所有注释类型的完全限定类名
    	 */
    	default Set<String> getAnnotationTypes() {
    		return getAnnotations().stream()
    				.filter(MergedAnnotation::isDirectlyPresent)
    				.map(annotation -> annotation.getType().getName())
    				.collect(Collectors.toCollection(LinkedHashSet::new));
    	}
    
    	/**
    	 * 获取基础类上给定注释类型上存在的所有元注释类型的完全限定类名（参数不明白）
    	 */
    	default Set<String> getMetaAnnotationTypes(String annotationName) {
    		MergedAnnotation<?> annotation = getAnnotations().get(annotationName, MergedAnnotation::isDirectlyPresent);
    		if (!annotation.isPresent()) {
    			return Collections.emptySet();
    		}
    		return MergedAnnotations.from(annotation.getType(), SearchStrategy.INHERITED_ANNOTATIONS).stream()
    				.map(mergedAnnotation -> mergedAnnotation.getType().getName())
    				.collect(Collectors.toCollection(LinkedHashSet::new));
    	}
    
    	/**
    	 * 确定基础类上是否存在给定类型的注释
    	 */
    	default boolean hasAnnotation(String annotationName) {
    		return getAnnotations().isDirectlyPresent(annotationName);
    	}
    
    	/**
    	 * 确定基础类上是否存在给定类型的元注解（参数不明白）
    	 */
    	default boolean hasMetaAnnotation(String metaAnnotationName) {
    		return getAnnotations().get(metaAnnotationName,
    				MergedAnnotation::isMetaPresent).isPresent();
    	}
    
    	/**
    	 * 确定基础类是否具有使用给定注释类型进行注释（或元注释）的任何方法
    	 */
    	default boolean hasAnnotatedMethods(String annotationName) {
    		return !getAnnotatedMethods(annotationName).isEmpty();
    	}
    
    	/**
    	 * 检索使用给定注释类型进行注释（或元注释）的所有方法的方法元数据
    	 */
    	Set<MethodMetadata> getAnnotatedMethods(String annotationName);
    
    
    	/**
    	 * 使用标准反射为给定类创建新的注释元数据实例的工厂方法
    	 */
    	static AnnotationMetadata introspect(Class<?> type) {
    		return StandardAnnotationMetadata.from(type);
    	}
    
    }
    ```

  * **BeanDefinitionRegistry接口**

    ```java
    org.springframework.core.type.AnnotationMetadata
    
    public interface AnnotationMetadata extends ClassMetadata, AnnotatedTypeMetadata {
    
    	/**
    	 * 获取基础类上存在的所有注释类型的完全限定类名
    	 */
    	default Set<String> getAnnotationTypes() {
    		return getAnnotations().stream()
    				.filter(MergedAnnotation::isDirectlyPresent)
    				.map(annotation -> annotation.getType().getName())
    				.collect(Collectors.toCollection(LinkedHashSet::new));
    	}
    
    	/**
    	 * 获取基础类上给定注释类型上存在的所有元注释类型的完全限定类名（参数不明白）
    	 */
    	default Set<String> getMetaAnnotationTypes(String annotationName) {
    		MergedAnnotation<?> annotation = getAnnotations().get(annotationName, MergedAnnotation::isDirectlyPresent);
    		if (!annotation.isPresent()) {
    			return Collections.emptySet();
    		}
    		return MergedAnnotations.from(annotation.getType(), SearchStrategy.INHERITED_ANNOTATIONS).stream()
    				.map(mergedAnnotation -> mergedAnnotation.getType().getName())
    				.collect(Collectors.toCollection(LinkedHashSet::new));
    	}
    
    	/**
    	 * 确定基础类上是否存在给定类型的注释
    	 */
    	default boolean hasAnnotation(String annotationName) {
    		return getAnnotations().isDirectlyPresent(annotationName);
    	}
    
    	/**
    	 * 确定基础类上是否存在给定类型的元注解（参数不明白）
    	 */
    	default boolean hasMetaAnnotation(String metaAnnotationName) {
    		return getAnnotations().get(metaAnnotationName,
    				MergedAnnotation::isMetaPresent).isPresent();
    	}
    
    	/**
    	 * 确定基础类是否具有使用给定注释类型进行注释（或元注释）的任何方法
    	 */
    	default boolean hasAnnotatedMethods(String annotationName) {
    		return !getAnnotatedMethods(annotationName).isEmpty();
    	}
    
    	/**
    	 * 检索使用给定注释类型进行注释（或元注释）的所有方法的方法元数据
    	 */
    	Set<MethodMetadata> getAnnotatedMethods(String annotationName);
    
    
    	/**
    	 * 使用标准反射为给定类创建新的注释元数据实例的工厂方法
    	 */
    	static AnnotationMetadata introspect(Class<?> type) {
    		return StandardAnnotationMetadata.from(type);
    	}
    
    }
    ```

    **基本上所有Bean工厂都实现了这个接口**，让工厂拥有bean注册的各种功能

  * **BeanNameGenerator接口**

    ```java
    org.springframework.beans.factory.support.BeanNameGenerator
    
    public interface BeanNameGenerator {
    
    	/**
    	 * 给给定Bean生成beanName
    	 */
    	String generateBeanName(BeanDefinition definition, BeanDefinitionRegistry registry);
    
    }
    ```

    **DefaultBeanNameGenerator**

    默认Bean名称生成器，XML中Bean指定名称时，使用这个生成器，默认为**完整的类名#bean编号**

    **AnnotationBeanNameGenerator**

    注解方式指定Bean名称，如果不指定名称，默认会**将完整的类名作为bean名称**

    **FullyQualifiedAnnotationBeanNameGenerator**

    **将完整的类作为bean的名称**

* **value为ImportSelector接口类型**

  ```java
  org.springframework.context.annotation.ImportSelector
      
  public interface ImportSelector {
  
  	/**
  	 * 返回需要导入的类名数组
  	 */
  	String[] selectImports(AnnotationMetadata importingClassMetadata);
  
  	/**
  	 * 返回一个谓词，用于从导入候选项中，进行筛选
  	 * 如果返回true，则该类将不被视为Bean被导入
  	 */
  	@Nullable
  	default Predicate<String> getExclusionFilter() {
  		return null;
  	}
  
  }    
  ```

* **value为DeferredImportSelector接口类型**

  SpringBoot核心功能**@EnableAutoConfiguration**主要依靠DeferredImportSelector接口实现的

  DefferedImportSelector是ImportSelector的子接口

  **特点**

  * 延迟导入

  * 指定导入类的处理顺序

    当@Import中有多个DeferredImportSelector实现类，可以指定顺序

    **通过实现Ordered接口或者通过@Order修饰**，value值越小，优先级越高

### ImportAware接口作用

```java
org.springframework.context.annotation.ImportAware

public interface ImportAware extends Aware {

	/**
	 * 设置导入@Configuration类的注释元数据
	 */
	void setImportMetadata(AnnotationMetadata importMetadata);

}
```

@Import注入Bean的方式中，**基于ImportSelector接口和ImportBeanDefinitionRegistrar接口的都可以从方法获取AnnotationMetadata**，基于Configuration类只能通过实现ImportAware接口

```java
@Configuration
public class ProxyConfiguration implements ImportAware {
    
    private AnnotationAttributes info;

    @Override
    public void setImportMetadata(AnnotationMetadata importMetadata) {
       this.info = AnnotationAttributes.fromMap(
               importMetadata.getAnnotationAttributes(目标注解.class.getName(), false)
       );
    }
}
```

**主要是在ConfigurationClassPostProcessor中注入了一个ImportAwareBeanPostProcessor**

## @ComponentScan注解

这个注解会扫描某些包及其子包中的所有类，满足条件注册到容器中

```java
org.springframework.context.annotation.ComponentScan

    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.TYPE)
    @Documented
    @Repeatable(ComponentScans.class)
    public @interface ComponentScan {

    // 指定需要扫描的包
    @AliasFor("basePackages")
    String[] value() default {};
    @AliasFor("value")
    String[] basePackages() default {};

    // 指定Spring容器会扫描这些类所在的包及其子包中的类
    Class<?>[] basePackageClasses() default {};

    // 指定名称生成器
    Class<? extends BeanNameGenerator> nameGenerator() default BeanNameGenerator.class;

    Class<? extends ScopeMetadataResolver> scopeResolver() default AnnotationScopeMetadataResolver.class;
    ScopedProxyMode scopedProxy() default ScopedProxyMode.DEFAULT;

    // 指定需要扫描包中资源的匹配方式
    String resourcePattern() default "**/*.class";

    // 对扫描的包是否启用默认过滤器
    boolean useDefaultFilters() default true;

    // 用来指定哪些类会作为组件注册到容器中
    Filter[] includeFilters() default {};

    // 用来指定那些类不会被注册到容器中
    Filter[] excludeFilters() default {};

    // 是否延迟初始化被注册的bean
    boolean lazyInit() default false;

    @Retention(RetentionPolicy.RUNTIME)
    @Target({})
    @interface Filter {

        /**
     	 * 过滤器类型，是一个枚举类型
     	 * ANNOTATION：通过注解的方式来筛选候选者，即判断候选者是否有指定的注解
     	 * ASSIGNABLE_TYPE：通过指定的类型来筛选后选择，判断候选者是否为指定的类
     	 * ASPECTJ：ASPECTJ表达式，判断是否匹配ASPECTJ表达式
     	 * REGEX：正则表达式，判断是否匹配RegEx表达式
     	 * CUSTOM：用户自定义过滤器来筛选候选者
     	 */
         FilterType type() default FilterType.ANNOTATION;

        /**
	 	 * 当type为ANNOTATION时，class指定注解，用来判断被扫描的类上是否有classes参数指定的注解
		 * 当type为ASSIGNABLE_TYPE时，classes指定类型，用来判断被扫描的类是否是classes参数指定的类型
		 * 当type为CUSTOM时，表示过滤器是自定义，classes指定自定义过滤器
		 * 自定义过滤器需要实现org.springframework.core.type.filter.TypeFilter接口
		 */
         @AliasFor("classes")
         Class<?>[] value() default {};
         @AliasFor("value")
         Class<?>[] classes() default {};

        /**
		 * 当type=FilterType.ASPCETJ时，通过pattern来指定需要匹配ASPECTJ表达式的值
		 * 当type=FilterType.REGEX时，通过pattern来指定需要的正则表达式值
		 */
         String[] pattern() default {};

    }

}    
```

将Filter的type属性设置为FilterType.CUSTOM类型，class属性需要指定自定义过滤器

**自定义过滤器需要实现TypeFilter接口**

```java
org.springframework.core.type.filter.TypeFilter
    
@FunctionalInterface
public interface TypeFilter {

	/**
	 * 确定此筛选器是否与给定元数据描述的类匹配
	 */
	boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory)
			throws IOException;

}
```

* **MetadataReader接口**

  ```java
  org.springframework.core.type.classreading.MetadataReader
      
  public interface MetadataReader {
  
  	/**
  	 * 返回类文件的资源索引
  	 */
  	Resource getResource();
  
  	/**
  	 * 返回一个ClassMetadata对象，可以获取类的一些元素据信息
  	 */
  	ClassMetadata getClassMetadata();
  
  	/**
  	 * 获取类上所有的注解信息
  	 */
  	AnnotationMetadata getAnnotationMetadata();
  
  }
  ```

* **MetadataReaderFactory接口**

  通过类元数据读取器工厂，可以通过这个类**获取任意一个类MetadataReader对象**

  ```java
  org.springframework.core.type.classreading.MetadataReaderFactory
      
  public interface MetadataReaderFactory {
  
  	/**
  	 * 返回给定类名的MetadataReader对象
  	 */
  	MetadataReader getMetadataReader(String className) throws IOException;
  
  	/**
  	 * 返回指定资源的MetadataRreader对象
  	 */
  	MetadataReader getMetadataReader(Resource resource) throws IOException;
  
  }
  ```

## @ComponentScans注解

将多个@ComponentScan修饰在同一个类上有两种方式

**@ComponentScan注解被@Repetable注解修饰**

```java
org.springframework.context.annotation.ComponentScans
    
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
public @interface ComponentScans {

	ComponentScan[] value();

}
```

* 直接使用多个@ComponentScan修饰

* 使用@ComponentScans，包装多个@ComponentScan

## @Conditional注解

@Conditional可以配置条件判断，所有条件满足，目标才会被Spring容器处理

```java
org.springframework.context.annotation.Conditional

@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Conditional {

	/**
	 * 条件类集合
	 */
	Class<? extends Condition>[] value();

}
```

Condition是函数式接口，表示一个条件判断

```java
org.springframework.context.annotation.Condition

@FunctionalInterface
public interface Condition {

	/**
	 * 判断条件是否匹配
	 * ConditionContext
	 * AnnotatedTypeMetadata 用来获取被@conditional标注的对象上的所有注释信息
	 */
	boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata);

}
```

ConditionContext接口提供方法获取Spring容器中的各种信息

```java
org.springframework.context.annotation.ConditionContext

public interface ConditionContext {

	/**
	 * 获取Bean信息注册器，通过注册器获取bean定义的各种配置信息
	 */
	BeanDefinitionRegistry getRegistry();

	/**
	 * 获取BeanFactory
	 */
	@Nullable
	ConfigurableListableBeanFactory getBeanFactory();

	/**
	 * 返回当前Spring容器的环境配置信息对象
	 */
	Environment getEnvironment();

	/**
	 * 返回资源加载器
	 */
	ResourceLoader getResourceLoader();

	/**
	 * 返回类加载器
	 */
	@Nullable
	ClassLoader getClassLoader();

}
```

### ConfigurationCondition

条件判断主要在两个阶段进行

* **配置类解析阶段**

  获取配置类的信息和需要注册的Bean

* **Bean注册阶段**

  将配置类解析阶段获取的配置类和需要注册的Bean注册到Spring容器中

如果将Condition的实现类配置到@Conditional中，**此时无法精确的控制某个阶段**

ConfigurationCondition接口的实现类就可以控制某个阶段

```java
org.springframework.context.annotation.ConfigurationCondition

public interface ConfigurationCondition extends Condition {

	/**
	 * 返回条件针对的阶段
	 */
	ConfigurationPhase getConfigurationPhase();

	enum ConfigurationPhase {

		/**
		 * 配置类解析阶段，如果条件为false，配置类不会被解析
		 */
		PARSE_CONFIGURATION,

		/**
		 * Bean注册阶段，如果条件为false，Bean不会被注册
		 */
		REGISTER_BEAN
	}

}
```

## @Value注解

```java
org.springframework.beans.factory.annotation.Value

@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Value {

	String value();

}
```

使用@Value注解可以获取配置文件中的字段

**使用@PropertySource来指定外部的自定义Properties配置文件**

```java
org.springframework.context.annotation.PropertySource

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Repeatable(PropertySources.class)
public @interface PropertySource {

	/**
	 * 指定属性源名称
	 */
	String name() default "";

	/**
	 * 指定要加载的属性文件的资源位置
	 * 支持classpath:...和file:...
	 * 不允许使用资源通配符
	 * 每一个值，都按声明的顺序添加
	 */
	String[] value();

	/**
	 * 返回是否忽略找不到属性资源的问题，如果属性文件是可选的，那么应设置为true
	 */
	boolean ignoreResourceNotFound() default false;

	/**
	 * 设置字符编码
	 */
	String encoding() default "";

	/**
	 * 指定自定义属性源工厂，默认情况下，将使用标准资源文件的默认工厂
	 */
	Class<? extends PropertySourceFactory> factory() default PropertySourceFactory.class;

}
```

@Value数据来源可以理解为一个PropertySource类

```java
org.springframework.core.env.PropertySource
    
public abstract class PropertySource<T> {
    
    // 配置源关联的名称
	protected final String name;
	// 配置源源对象
	protected final T source;

	/**
	 * 返回此属性源是否包含给定的名称
	 */
	public boolean containsProperty(String name) {
		return (getProperty(name) != null);
	}

	/**
	 * 根据名称返回源
	 */
	@Nullable
	public abstract Object getProperty(String name);



	/**
	 * 根据名称生成一个PropertySource，用作在集合中进行比较
	 */
	public static PropertySource<?> named(String name) {
		return new ComparisonPropertySource(name);
	}
    
}
```

配置源相关有一个重要的接口

```java
org.springframework.core.env.Environment

public interface Environment extends PropertyResolver {

	/**
	 * 返回此环境显示激活的配置文件集合（如设置spring.profiles.active）
	 * 如果没有将任何配置文件显示指定为活动，那么自动激活默认配置文件
	 */
	String[] getActiveProfiles();

	/**
	 * 当未显式设置活动配置文件时，返回默认处于活动状态的配置文件集合
	 */
	String[] getDefaultProfiles();

	@Deprecated
	boolean acceptsProfiles(String... profiles);

	/**
	 * 返回活动配置文件是否于给定的 配置文件谓词匹配 
	 */
	boolean acceptsProfiles(Profiles profiles);

}
```

Environment中主要的方法来自接口PropertyResolver

```java
org.springframework.core.env.PropertyResolver
    
public interface PropertyResolver {

	/**
	 * 判断是否包含key对应的属性源
	 */
	boolean containsProperty(String key);

	/**
	 * 获取属性源，可以为空
	 */
	@Nullable
	String getProperty(String key);

	/**
	 * 获取属性源，如果不存在，返回指定默认值
	 */
	String getProperty(String key, String defaultValue);

	/**
	 * 获取指定类型的属性源，可以为空
	 */
	@Nullable
	<T> T getProperty(String key, Class<T> targetType);

	/**
	 * 获取指定类型的属性源，否则返回指定默认值
	 */
	<T> T getProperty(String key, Class<T> targetType, T defaultValue);

	/**
	 * 获取属性值，否则报错
	 */
	String getRequiredProperty(String key) throws IllegalStateException;

	/**
	 * 获取指定类型属性值，否则报错
	 */
	<T> T getRequiredProperty(String key, Class<T> targetType) throws IllegalStateException;

	/**
	 * 解析${...}字段
	 */
	String resolvePlaceholders(String text);

	/**
	 * 解析${...}字段，如果字段不存在，报错
	 */
	String resolveRequiredPlaceholders(String text) throws IllegalArgumentException;

}
```

Environment中属性源**存放在**Environment中MutablePropertySource内部，**通过getPropertySources()获取**

改变@Value数据源，**只需要将配置信息包装为PropertySource对象，放到Environment中MutablePropertySource中**

**案例**

```java
MapPropertySource source = new MapPropertySource("map", map);
context.getEnvironment().getPropertySources().addFirst(source);
```

### @Value动态刷新

**主要依赖于@Scope中的ScopedProxyMode属性**

```java
public class BeanRefreshScope implements Scope {

    public static final String SCOPE_REFRESH = "refresh";

    private static final BeanRefreshScope INSTANCE = new BeanRefreshScope();

    private ConcurrentHashMap<String, Object> beanMap = new ConcurrentHashMap<>();

    private BeanRefreshScope() {
    }

    public static BeanRefreshScope getInstance() {
        return INSTANCE;
    }
    
    public static void clean() {
        INSTANCE.beanMap.clear();
    }

    @Override
    public Object get(String name, ObjectFactory<?> objectFactory) {
        Object bean = beanMap.get(name);
        if (bean == null) {
            bean = objectFactory.getObject();
            beanMap.put(name, bean);
        }
        return bean;
    }

    @Override
    public Object remove(String name) {
        return beanMap.remove(name);
    }

    @Override
    public void registerDestructionCallback(String name, Runnable callback) {

    }

    @Override
    public Object resolveContextualObject(String key) {
        return null;
    }

    @Override
    public String getConversationId() {
        return SCOPE_REFRESH;
    }
}
```

将注解修饰在配置类中，每次调用方法，都会获取新的配置属性源

## Bean生命周期

<img src="Picture\Spring\Bean生命周期.png" alt="Bean生命周期流程图" style="zoom: 67%;" />

### Bean元信息配置阶段

* **API方式配置Bean元信息**

  其他几种方式底层都采用这种方式定义

  Spring容器启动过程中，会将Bean解析成BeanDefinition

  BeanFactory根据BeanDefinition，对Bean实例化、初始化...

  ```java
  org.springframework.beans.factory.config.BeanDefinition
  
  public interface BeanDefinition extends AttributeAccessor, BeanMetadataElement {
  
      // 作用域标识符
  	String SCOPE_SINGLETON = ConfigurableBeanFactory.SCOPE_SINGLETON;
  	String SCOPE_PROTOTYPE = ConfigurableBeanFactory.SCOPE_PROTOTYPE;
  
  	int ROLE_APPLICATION = 0;
  
  	int ROLE_SUPPORT = 1;
  
  	int ROLE_INFRASTRUCTURE = 2;
  
  	/**
  	 * 设置父定义名称
  	 */
  	void setParentName(@Nullable String parentName);
  	@Nullable
  	String getParentName();
  
  	/**
  	 * 设置Bean的类名
  	 */
  	void setBeanClassName(@Nullable String beanClassName);
  	@Nullable
  	String getBeanClassName();
  
  	/**
  	 * 设置Bean作用域
  	 */
  	void setScope(@Nullable String scope);
  	@Nullable
  	String getScope();
  
  	/**
  	 * 设置是否懒加载Bean
  	 */
  	void setLazyInit(boolean lazyInit);
  	boolean isLazyInit();
  
  	/**
  	 * 设置DependsOn
  	 */
  	void setDependsOn(@Nullable String... dependsOn);
  	@Nullable
  	String[] getDependsOn();
  
  	/**
  	 * 设置AutowireCandidate
  	 */
  	void setAutowireCandidate(boolean autowireCandidate);
  	boolean isAutowireCandidate();
  
  	/**
  	 * 设置Primary
  	 */
  	void setPrimary(boolean primary);
  	boolean isPrimary();
  
  	/**
  	 * 指定工厂Bean名称
  	 */
  	void setFactoryBeanName(@Nullable String factoryBeanName);
  	@Nullable
  	String getFactoryBeanName();
  
  	/**
  	 * 设置工厂方法名称
  	 */
  	void setFactoryMethodName(@Nullable String factoryMethodName);
  	@Nullable
  	String getFactoryMethodName();
  
  	/**
  	 * 获取构造器参数
  	 */
  	ConstructorArgumentValues getConstructorArgumentValues();
  
  	/**
  	 * 判断是有构造器参数
  	 */
  	default boolean hasConstructorArgumentValues() {
  		return !getConstructorArgumentValues().isEmpty();
  	}
  
  	/**
  	 * 返回Bean的属性值
  	 */
  	MutablePropertyValues getPropertyValues();
  	default boolean hasPropertyValues() {
  		return !getPropertyValues().isEmpty();
  	}
  
  	/**
  	 * 设置Bean初始化方法
  	 */
  	void setInitMethodName(@Nullable String initMethodName);
  	@Nullable
  	String getInitMethodName();
  
  	/**
  	 * 设置Bean销毁方法
  	 */
  	void setDestroyMethodName(@Nullable String destroyMethodName);
  	@Nullable
  	String getDestroyMethodName();
  
  	/**
  	 * 指定角色提示？？？
  	 */
  	void setRole(int role);
  	int getRole();
  
  	/**
  	 * 设置Bean介绍
  	 */
  	void setDescription(@Nullable String description);
  	@Nullable
  	String getDescription();
      
  	ResolvableType getResolvableType();
  
  	/**
  	 * 判断是否单例
  	 */
  	boolean isSingleton();
  
  	/**
  	 * 判断是否多例
  	 */
  	boolean isPrototype();
  
  	/**
  	 * 判断是否抽象Bean
  	 */
  	boolean isAbstract();
  
  	/**
  	 * 返回Bean定义来自的资源的描述（查错）
  	 */
  	@Nullable
  	String getResourceDescription();
  
  	/**
  	 * 返回原始BeanDefinition
  	 */
  	@Nullable
  	BeanDefinition getOriginatingBeanDefinition();
  
  }
  ```

  BeanDefinition接口继承了两个接口

  * **AttributeAccessor接口：属性访问接口**

    ```java
    org.springframework.core.AttibuteAccessor
    
    public interface AttributeAccessor {
     
        /**
         * 设置属性->值
         */
        void setAttribute(String name, @Nullable Object value);
     
        /**
         * 获取某个属性对应的值
         */
        @Nullable
        Object getAttribute(String name);
     
        /**
         * 移除某个属性
         */
        @Nullable
        Object removeAttribute(String name);
     
        /**
         * 是否包含某个属性
         */
        boolean hasAttribute(String name);
     
        /**
         * 返回所有的属性名称
         */
        String[] attributeNames();
     
    }
    ```

  * **BeanMetadataElement接口**

    ```java
    org.springframework.beans.BeanMetadataElement
    
    public interface BeanMetadataElement {
    
    	/**
    	 * 返回元数据的配置元对象
    	 */
    	@Nullable
    	default Object getSource() {
    		return null;
    	}
    
    }
    ```

  BeanDefinition常用的实现类

  * **RootBeanDefinition类：表示根Bean定义信息**

    通常Bean中没有父bean的使用这种BeanDefinition

  * **ChildBeanDefinition类：表示子Bean定义信息**

    如果需要指定父Bean，使用这个BeanDefinition定义子Bean的配置信息

  * **GenericBeanDefinition类：通用的Bean定义信息**

    有无父Bean都可以使用

  * **ConfigurationClassBeanDefinitionl类：表示通过配置类中@Bean创建的Bean定义信息**

  * **AnnotatedBeanDefinition接口：表示通过注解方式定义的Bean**

    其中`AnnotationMetadata getMetadata()`可以获取Bean上的注解信息

  * **BeanDefinitionBuilder：构建BeanDefinition工具**

* **读取XML配置文件的方式**

* **读取Properties配置文件的方式**

* **通过注解的方式**

### Bean元信息的解析阶段

将各种方式定义的Bean元信息转换成BeanDefinition

Bean元信息解析方式

* **XML配置Bean元信息解析**

  Spring提供XmlBeanDefinitionReader

  ```java
  DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
  XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(factory);
  String location = "...";
  int countBean = reader.loadBeanDefinitions(location);
  ```

* **Properties配置Bean元信息解析**

  Spring提供PropertiesBeanDefinitionReader，与XML流程类似

* **注解方式配置Bean元信息解析**

  Spring提供AnnotatedBeanDefinitionReader

### Bean注册阶段

Bean的注册需要使用BeanDefinitionRegistry接口

```java
org.springframework.beans.factory.support.BeanDefinitionRegistry

public interface BeanDefinitionRegistry extends AliasRegistry {

	/**
	 * 向注册表注册一个新的BeanDefinition
	 */
	void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
			throws BeanDefinitionStoreException;

	/**
	 * 删除注册表中的BeanDefinition
	 */
	void removeBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;

	/**
	 * 获取注册表中的BeanDefinition
	 */
	BeanDefinition getBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;

	/**
	 * 检查是否存在指定的BeanDefinition
	 */
	boolean containsBeanDefinition(String beanName);

	/**
	 * 返回该注册表中的所有BeanDefinition
	 */
	String[] getBeanDefinitionNames();

	/**
	 * 返回注册表中的BeanDefinition数量
	 */
	int getBeanDefinitionCount();

	/**
	 * 检测指定beanName是否在注册表中已经使用
	 */
	boolean isBeanNameInUse(String beanName);

}
```

BeanDefinitionRegistry实现了接口

```java
org.springframework.core.AliasRegistry
    
public interface AliasRegistry {

	/**
	 * 給指定名称注册别名
	 */
	void registerAlias(String name, String alias);

	/**
	 * 删除别名
	 */
	void removeAlias(String alias);

	/**
	 * 判断名称是不是别名
	 */
	boolean isAlias(String name);

	/**
	 * 返回给定名称的别名
	 */
	String[] getAliases(String name);

}    
```

**DefaultListableBeanFactory**是BeanDefinitionRegistry的唯一实现类

### BeanDefinition合并阶段

这个阶段主要对应方法

```java
org.springframework.beans.factory.support.AbstractBeanFactory#getMergedBeanDefinition

protected RootBeanDefinition getMergedBeanDefinition(
    String beanName, BeanDefinition bd, @Nullable BeanDefinition containingBd)
    throws BeanDefinitionStoreException {

    synchronized (this.mergedBeanDefinitions) {
        RootBeanDefinition mbd = null;
        RootBeanDefinition previous = null;

        // 从已经合并的BeanDefinition集合中获取
        if (containingBd == null) {
            mbd = this.mergedBeanDefinitions.get(beanName);
        }

        // 已合并的集合中不存在
        if (mbd == null || mbd.stale) {
            previous = mbd;
            // 该BeanDefinition没有父信息
            if (bd.getParentName() == null) {
                
                if (bd instanceof RootBeanDefinition) {
                    mbd = ((RootBeanDefinition) bd).cloneBeanDefinition();
                }
                else {
                    mbd = new RootBeanDefinition(bd);
                }
            }
            // 有父信息，合并父子信息定义
            else {
                BeanDefinition pbd;
                try {
                    String parentBeanName = transformedBeanName(bd.getParentName());
                    // 父子名称不同
                    if (!beanName.equals(parentBeanName)) {
                        pbd = getMergedBeanDefinition(parentBeanName);
                    }
                    // 父子名称相同，父类如果是BeanFactotry则通过名字获取工厂中的合并信息，否则报错
                    else {
                        BeanFactory parent = getParentBeanFactory();
                        if (parent instanceof ConfigurableBeanFactory) {
                            pbd = ((ConfigurableBeanFactory) parent).getMergedBeanDefinition(parentBeanName);
                        }
                        else {
                            throw new NoSuchBeanDefinitionException(parentBeanName,
                                                                    "Parent name '" + parentBeanName + "' is equal to bean name '" + beanName +
                                                                    "': cannot be resolved without a ConfigurableBeanFactory parent");
                        }
                    }
                }
                catch (NoSuchBeanDefinitionException ex) {
                    throw new BeanDefinitionStoreException(bd.getResourceDescription(), beanName,
                                                           "Could not resolve parent bean definition '" + bd.getParentName() + "'", ex);
                }
                // 深度将子类的信息重写给父类，合并完成
                mbd = new RootBeanDefinition(pbd);
                mbd.overrideFrom(bd);
            }

            if (!StringUtils.hasLength(mbd.getScope())) {
                mbd.setScope(SCOPE_SINGLETON);
            }
            
            if (containingBd != null && !containingBd.isSingleton() && mbd.isSingleton()) {
                mbd.setScope(containingBd.getScope());
            }

            // 将合并信息放入缓存中
            if (containingBd == null && isCacheBeanMetadata()) {
                this.mergedBeanDefinitions.put(beanName, mbd);
            }
        }
        if (previous != null) {
            copyRelevantMergedBeanDefinitionCaches(previous, mbd);
        }
        return mbd;
    }
}
```

从上述流程中可以看出，合并之前使用的是GenericBeanDefinition，**合并之后使用的是RootBeanDefinition**，后续过程也是使用这个合成后的RootBeanDefinition

### Bean的Class加载阶段

> BeanPostProcessor接口，主要用于**在Bean生命周期不同阶段提供扩展**
>
> ```java
> org.springframework.beans.factory.config.BeanPostProcessor
> 
> public interface BeanPostProcessor {
> 
> 	/**
> 	 * Bean初始化前回调
> 	 */
> 	@Nullable
> 	default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
> 		return bean;
> 	}
> 
> 	/**
> 	 * Bean初始化后回调
> 	 */
> 	@Nullable
> 	default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
> 		return bean;
> 	}
> 
> }
> ```

* **Bean实例化前操作**

  Bean实例化前会调用方法

  ```java
  org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#applyBeanProcessorsBeforeInstantiation
  
  @Nullable
  protected Object applyBeanPostProcessorsBeforeInstantiation(Class<?> beanClass, String beanName) {
      for (BeanPostProcessor bp : getBeanPostProcessors()) {
          if (bp instanceof InstantiationAwareBeanPostProcessor) {
              InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
              Object result = ibp.postProcessBeforeInstantiation(beanClass, beanName);
              if (result != null) {
                  return result;
              }
          }
      }
      return null;
  }
  ```

  这段代码可以直接手动创建Bean实例，**跳过Bean初始化操作**

  轮询beanDefinitionProcessor列表，如果类型是InstantiationAwareBeanPostProcessor，那么调用postProcessBeforeInstantiation获取bean的实例对象，**将返回值作为当前bean实例**

* **Bean实例化操作**

  通过反射调用Bean的**构造器**创建Bean实例

  Spring提供一个接口，用来判断使用哪个构造器

  ```java
  org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#determineConstructorsFromBeanPostProcessors
  
  @Nullable
  protected Constructor<?>[] determineConstructorsFromBeanPostProcessors(@Nullable Class<?> beanClass, String beanName)
      throws BeansException {
  
      if (beanClass != null && hasInstantiationAwareBeanPostProcessors()) {
          for (BeanPostProcessor bp : getBeanPostProcessors()) {
              if (bp instanceof SmartInstantiationAwareBeanPostProcessor) {
                  SmartInstantiationAwareBeanPostProcessor ibp = (SmartInstantiationAwareBeanPostProcessor) bp;
                  Constructor<?>[] ctors = ibp.determineCandidateConstructors(beanClass, beanName);
                  if (ctors != null) {
                      return ctors;
                  }
              }
          }
      }
      return null;
  }
  ```

  调用SmartInstantiationAwareBeanPostProcessor的determineCandidateConstructor方法，**方法返回指定构造器列表**

  `org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor`实现类**可以将@Autowired标注的构造器作为候选构造器返回**

### 合并后Bean元信息处理阶段

```java
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory
    
protected void applyMergedBeanDefinitionPostProcessors(RootBeanDefinition mbd, Class<?> beanType, String beanName) {
    for (BeanPostProcessor bp : getBeanPostProcessors()) {
        if (bp instanceof MergedBeanDefinitionPostProcessor) {
            MergedBeanDefinitionPostProcessor bdp = (MergedBeanDefinitionPostProcessor) bp;
            bdp.postProcessMergedBeanDefinition(mbd, beanType, beanName);
        }
    }
}
```

这个方法会调用MergedBeanDefinitionPostProcessor接口中的postProcessMergedBeanDefinition方法

**方法中RootBeanDefinition表示之前合并流程返回的元信息**

```java
org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor
在 postProcessMergedBeanDefinition 方法中对 @Autowired、@Value 标注的方法、字段进行缓存
 
org.springframework.context.annotation.CommonAnnotationBeanPostProcessor
在 postProcessMergedBeanDefinition 方法中对 @Resource 标注的字段、@Resource 标注的方法、 @PostConstruct 标注的字段、 @PreDestroy标注的方法进行缓存
```

### Bean属性设置阶段

* **实例后阶段**

  ```java
  org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#populateBean
  
  protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
      
      ...
          
      if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
          for (BeanPostProcessor bp : getBeanPostProcessors()) {
              if (bp instanceof InstantiationAwareBeanPostProcessor) {
                  InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
                  if (!ibp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
                      return;
                  }
              }
          }
      }
      
      ...
  }
  ```

  主要会调用InstantiationAwareBeanPostProcessor中的postProcessAfterInstantiation方法

  如果postProcessAfterInstantiation返回false，**后续赋值前操作和赋值操作都会跳过**

* **Bean属性赋值前操作**

  ```java
  org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#populateBean
  
  protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
      
      ...
          
          for (BeanPostProcessor bp : getBeanPostProcessors()) {
              if (bp instanceof InstantiationAwareBeanPostProcessor) {
                  InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
                  PropertyValues pvsToUse = ibp.postProcessProperties(pvs, bw.getWrappedInstance(), beanName);
                  if (pvsToUse == null) {
                      if (filteredPds == null) {
                          filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
                      }
                      pvsToUse = ibp.postProcessPropertyValues(pvs, filteredPds, bw.getWrappedInstance(), beanName);
                      if (pvsToUse == null) {
                          return;
                      }
                  }
                  pvs = pvsToUse;
              }
          }
      
      ...
  }
  ```

  如果InstantiationAwareBeanPostProcessor中的**postProcessProperties和postProcessValues都返回空**，表示Bean不需要设置属性，**直接跳过Bean属性赋值操作**

  PropertyValues中**保存了bean实例化中所有的属性的设置**，可以在该方法中修改属性

* **Bean属性赋值操作**

  **循环处理PropertyValues**中的属性值信息，通过反射调用set方法将属性的值设置到Bean中

### Bean初始化阶段

* **BeanAware接口回调操作**

  如果Bean实现了BeanNameAware、BeanClassLoaderAware、BeanFactoryAware接口，会在此阶段执行回调

  ```java
  org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#invokeAwareMethods
  
  private void invokeAwareMethods(String beanName, Object bean) {
      if (bean instanceof Aware) {
          if (bean instanceof BeanNameAware) {
              ((BeanNameAware) bean).setBeanName(beanName);
          }
          if (bean instanceof BeanClassLoaderAware) {
              ClassLoader bcl = getBeanClassLoader();
              if (bcl != null) {
                  ((BeanClassLoaderAware) bean).setBeanClassLoader(bcl);
              }
          }
          if (bean instanceof BeanFactoryAware) {
              ((BeanFactoryAware) bean).setBeanFactory(AbstractAutowireCapableBeanFactory.this);
          }
      }
  }
  ```

* **Bean初始化前操作**

  ```java
  org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#applyBeanPostProcessorsBeforeInitialization
      
  @Override
  public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName)
      throws BeansException {
  
      Object result = existingBean;
      for (BeanPostProcessor processor : getBeanPostProcessors()) {
          Object current = processor.postProcessBeforeInitialization(result, beanName);
          if (current == null) {
              return result;
          }
          result = current;
      }
      return result;
  }
  ```

  方法会调用BeanPostProcessor中的postProcessBeforeInitialization方法，**返回null，跳过Bean实例化**

  这个类两个常用实现类

  * **org.springframework.context.support.ApplicationContextAwareProcessor**

    ```shell
    EnvironmentAware：注入Environment对象
    EmbeddedValueResolverAware：注入EmbeddedValueResolver对象
    ResourceLoaderAware：注入ResourceLoader对象
    ApplicationEventPublisherAware：注入ApplicationEventPublisher对象
    MessageSourceAware：注入MessageSource对象
    ApplicationContextAware：注入ApplicationContext对象
    ```

    ApplicationContextAwareProcessor会自动将自定对象注入

  * **org.springframework.context.annotation.CommonAnnotationBeanPostProcessor**

    会调用Bean中所标注的@PostConstruct注解的方法

* **Bean初始化操作**

  Bean必须实现InitializingBean接口才会在这个阶段调用

  ```java
  org.springframework.beans.factory.InitializingBean
  
  public interface InitializingBean {
      
  	void afterPropertiesSet() throws Exception;
  
  }
  ```

  这个阶段首先会调用InitializingBean中的afterPropertiesSet方法，然后会调用Bean的初始化方法

  > **Bean初始化方法设置**
  >
  > * XML方式
  >
  >   ```xml
  >   <bean init-method="bean中方法名称"/>
  >   ```
  >
  > * @Bean配置方式
  >
  >   ```java
  >   @Bean(initMethod = "初始化的方法")
  >   ```
  >
  > * API方式
  >
  >   ```java
  >   beanDefinition.setInitMethodName(methodName);
  >   ```

* **Bean初始化后操作**

  ```java
  org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#applyBeanPostProcessorsAfterInitialization
      
  @Override
  public Object applyBeanPostProcessorsAfterInitialization
      (Object existingBean, String beanName)
      throws BeansException {
  
      Object result = existingBean;
      for (BeanPostProcessor processor : getBeanPostProcessors()) {
          Object current = processor.postProcessAfterInitialization(result, beanName);
          if (current == null) {
              return result;
          }
          result = current;
      }
      return result;
  }
  ```

  方法返回null会中断上面的操作

* **Bean初始化完成操作**

### 单例Bean初始化完成阶段

```java
org.springframework.beans.factory.support.DefaultListableBeanFactory#preInstantiateSingletons

@Override
public void preInstantiateSingletons() throws BeansException {
    
    ...
    
    for (String beanName : beanNames) {
        Object singletonInstance = getSingleton(beanName);
        if (singletonInstance instanceof SmartInitializingSingleton) {
            SmartInitializingSingleton smartSingleton = (SmartInitializingSingleton) singletonInstance;
            if (System.getSecurityManager() != null) {
                AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
                    smartSingleton.afterSingletonsInstantiated();
                    return null;
                }, getAccessControlContext());
            }
            else {
                smartSingleton.afterSingletonsInstantiated();
            }
        }
    }
}
```

**所有单例Bean（除LazyBean）实例化完成后回调**SmartInitializingSingletion接口的afterSingletonsInstantiated方法

### Bean使用阶段

**省略**

### Bean销毁阶段

> **调用Bean销毁方法方式**
>
> * 调用`org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#destroyBean`
> * 调用`org.springframework.beans.factory.config.ConfigurableBeanFactory#destroySingletons`
> * 调用ApplicationContext中的close方法

```java
org.springframework.beans.factory.config.DestructionAwareBeanPostProcessor

public interface DestructionAwareBeanPostProcessor extends BeanPostProcessor {
 
    /**
     * bean销毁前调用的方法
     */
    void postProcessBeforeDestruction(Object bean, String beanName) throws BeansException;
 
    /**
     * 用来判断bean是否需要触发postProcessBeforeDestruction方法
     */
    default boolean requiresDestruction(Object bean) {
        return true;
    }
 
}
```

Bean销毁阶段

首先轮询获取DestructionAwareBeanPostProcessor调用内部postProcessBeforeDestruction方法

之后如果实现了`org.springframework.beans.factory.DisposableBean`接口，会调用其中destory方法

```java
org.springframework.beans.factory.DisposableBean

public interface DisposableBean {

	void destroy() throws Exception;

}
```

最后调用Bean自定义的销毁方法

> **销毁方法执行顺序**
>
> 1. @PreDestroy标注的所有方法
> 2. DisposableBean接口中的destroy方法
> 3. 自定的销毁方法

## 注解实现依赖注入

### @Autowired依赖注入

```java
org.springframework.beans.factory.annotation.Autowired

@Target({ElementType.CONSTRUCTOR, ElementType.METHOD, ElementType.PARAMETER, ElementType.FIELD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Autowired {

	/**
	 * 表示这个依赖是否必须
	 */
	boolean required() default true;

}
```

**@Autowired查找候选者过程**

* **标注在字段上**

  <img src="Picture\Spring\@Autowired字段.png" alt="@Autowired字段依赖注入" style="zoom: 67%;" />

* **标注在方法上或者方法参数上**

  <img src="Picture\Spring\@Autowired方法.png" alt="@Autowired方法" style="zoom:67%;" />

@Autowired流程简化 -> **按类型找->通过限定符@Qualifier过滤->@Primary->@Priority->根据名称找（字段名称或者方法名称）**

### @Resource依赖注入

```java
javax.annotation.Resource
    
@Target({TYPE, FIELD, METHOD})
@Retention(RUNTIME)
@Repeatable(Resources.class)
public @interface Resource {
    String name() default "";
	
    ...
}
```

**@Resource注解修饰方法，该方法只能有一个参数**

**@Resource查找候选者的过程**

* **标注在字段上**

  <img src="Picture\Spring\@Resource字段.png" alt="@Resource字段" style="zoom:67%;" />

* **标注在方法上或者方法参数上**

  <img src="Picture\Spring\@Resource方法.png" alt="@Resource方法" style="zoom:67%;" />

@Resource流程简化 -> **先按name作为名称找->按名称找->按类型找->通过限定符@Qualifier过滤->@Primary->@Priority->根据名称找（字段名称或者方法参数名称）**

### @Qualifier限定符

可以在依赖注入查找候选者的过程中对候选者进行过滤

```java
org.springframework.beans.factory.annotation.Qualifier

@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Qualifier {

	String value() default "";

}
```

### @Primary设置优先候选

当存在多个候选者等待注入时，有@Primary修饰的作为优选候选

```java
org.springframework.context.annotation.Primary

@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Primary {

}
```

## Bean循环依赖

### Spring解决循环依赖情况

Spring解决循环依赖是有前置条件的

* **出现循环依赖的Bean必须是单例**
* **依赖注入的方式不能全是通过构造器注入**

| 依赖情况   | 依赖注入方式                                     | 循环依赖是否被解决 |
| ---------- | ------------------------------------------------ | ------------------ |
| AB相互依赖 | 均采用setter方式注入                             | 是                 |
| AB相互依赖 | 均采用构造器方式注入                             | 否                 |
| AB相互依赖 | A中注入B的方式为setter，B中注入A的方式是为构造器 | 是                 |
| AB相互依赖 | B中注入A的方式为setter，A中注入B的方式是为构造器 | 否                 |

### Spring解决循环依赖的方式

```java
@Component
public class A {
    // A中注入了B
    @Autowired
    private B b;
}

@Component
public class B {
    // B中也注入了A
    @Autowired
    private A a;
}
```

* **非AOP循环依赖**

  通过getBean获取A

  ```java
  org.springframework.beans.factory.support.AbstractBeanFactory#getBean
  
  @Override
  public Object getBean(String name) throws BeansException {
      return doGetBean(name, null, null, false);
  }
  
  @Override
  public <T> T getBean(String name, Class<T> requiredType) throws BeansException {
      return doGetBean(name, requiredType, null, false);
  }
  ```

  getBean内部会调用doGetBean方法

  ```java
  org.springframework.beans.factory.support.AbstractBeanFactory#doGetBean
  
  protected <T> T doGetBean(
      String name, @Nullable Class<T> requiredType, @Nullable Object[] args, boolean typeCheckOnly)
      throws BeansException {
  
      String beanName = transformedBeanName(name);
      Object bean;
  
      // 判断一级缓存中是否有这个bean
      Object sharedInstance = getSingleton(beanName);
      if (sharedInstance != null && args == null) {
          // 如果有直接获取这个bean
          bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
      } else {
  
          ...
  
              // 创建Bean实例
              if (mbd.isSingleton()) {
                  sharedInstance = getSingleton(beanName, () -> {
                      try {
                          return createBean(beanName, mbd, args);
                      }
                      catch (BeansException ex) {
                          destroySingleton(beanName);
                          throw ex;
                      }
                  });
                  bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
              }
  
          ...
      }
  }
  ```

  其中主要关注getSingleton和getSingleton中回调的createBean

  * **getSingleton方法**

    > 1. singletonObjects，一级缓存，存储的是所有创建好的单例Bean
    > 2. earlySingletonObjects，二级缓存，完成实例化，但是还未进行属性注入及初始化对象
    > 3. singletonFactories，提前暴露的一个单例工厂，二级缓存中存储的就是从这个工厂中获取到的对象

    ```java
    org.springframework.beans.factory.support.DefaultSingletonBeanRegistry#getSingletion
    
    @Nullable
    protected Object getSingleton(String beanName, boolean allowEarlyReference) {
        // 检查一级缓存中是否存在这个bean
        Object singletonObject = this.singletonObjects.get(beanName);
        if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
            // 检查二级缓存中是否存在这个bean
            singletonObject = this.earlySingletonObjects.get(beanName);
            if (singletonObject == null && allowEarlyReference) {
                synchronized (this.singletonObjects) {
                    上锁，并进行double-check
                        singletonObject = this.singletonObjects.get(beanName);
                    if (singletonObject == null) {
                        singletonObject = this.earlySingletonObjects.get(beanName);
                        if (singletonObject == null) {
                            // 如果allowEarlyReference为true，从三级缓存中获取这个bean
                            ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                            if (singletonFactory != null) {
                                singletonObject = singletonFactory.getObject();
                                this.earlySingletonObjects.put(beanName, singletonObject);
                                this.singletonFactories.remove(beanName);
                            }
                        }
                    }
                }
            }
        }
        return singletonObject;
    }
    ```

  * **getSingle回调的createBean方法**

    ```java
    org.springframework.beans.factory.support.DefaultSingletonBeanRegistry#getSingletion
    
    public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
        synchronized (this.singletonObjects) {
            Object singletonObject = this.singletonObjects.get(beanName);
            if (singletonObject == null) {
                
                ...
                    
                // 检测是否循环依赖
                beforeSingletonCreation(beanName);
                
                ...
                    
                // 匿名函数的回调
                try {
                    singletonObject = singletonFactory.getObject();
                    newSingleton = true;
                }
                catch (IllegalStateException ex) {
                   	...
                }
                catch (BeanCreationException ex) {
                    ...
                }
                finally {
                    ...
                }
                if (newSingleton) {
                    addSingleton(beanName, singletonObject);
                }
            }
            return singletonObject;
        }
    }
    ```

  * **createBean方法**

    ```java
    org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#createBean
    
    @Override
    protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
        throws BeanCreationException {
    
        ...
    
        try {
            Object beanInstance = doCreateBean(beanName, mbdToUse, args);
            
            return beanInstance;
        }
        catch (BeanCreationException | ImplicitlyAppearedSingletonException ex) {
            ...
        }
        catch (Throwable ex) {
            ...
        }
    }
    ```

    createBean方法其中调用doCreateBean

    ```java
    org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#doCreateBean
    
    protected Object doCreateBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
        throws BeanCreationException {
    
        ...
    
            //判断是否需要暴露早期的bean，条件为（是否是单例bean && 当前容器允许循环依赖 && bean名称存在于正在创建的bean名称清单中）
            boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
                                              isSingletonCurrentlyInCreation(beanName));
        if (earlySingletonExposure) {
            
            // 将这个Bean暴露出去
            addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
        }
    
        ...
    
    }
    ```

    doCreateBean调用addSingletonFactory方法

    ```java
    org.springframework.beans.factory.support.DefaultSingletonBeanRegistry#addSingletonFactory
    
    protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
        synchronized (this.singletonObjects) {
            //第1级缓存中不存在bean
            if (!this.singletonObjects.containsKey(beanName)) {
                //将其丢到第3级缓存中
                this.singletonFactories.put(beanName, singletonFactory);
                ...
            }
        }
    }
    ```

    这个方法只是将一个工厂添加到第三级缓存中，通过工厂的getObject方法可以得到一个对象，**这个方法通过getEarlyBeanReference创建**

    这时A需要一个B依赖，所以通过getBean获取B，又发现B中有A

    但是这次不是创建A，而是从第三级缓存中获取A，调用方法getEarlyBeanReference

    ```java
    org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#doCreateBean
    
    protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
        Object exposedObject = bean;
        if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
            for (BeanPostProcessor bp : getBeanPostProcessors()) {
                if (bp instanceof SmartInstantiationAwareBeanPostProcessor) {
                    SmartInstantiationAwareBeanPostProcessor ibp = (SmartInstantiationAwareBeanPostProcessor) bp;
                    exposedObject = ibp.getEarlyBeanReference(exposedObject, beanName);
                }
            }
        }
        return exposedObject;
    }
    ```

    方法主要逻辑轮询遍历获取SmartInstantiationAwareBeanPostProcessor，调用getEarlyBeanReference

    **如果没有导入AOP，那么会使用默认的实现**

    ```java
    org.springframework.beans.factory.config.SmartInstantiationAwareBeanPostProcessor#getEarlyBeanReference
    
    default Object getEarlyBeanReference(Object bean, String beanName) throws BeansException {
        return bean;
    }
    ```

    **不考虑AOP，那么第三级缓存没有作用**

    > **B中提前注入了一个没有经过初始化的A类型对象不会有问题吗？**
    >
    > 虽然在创建B时会提前给B注入了一个还未初始化的A对象，但是在创建A的流程中一直使用的是注入到B中的A对象的引用，之后会根据这个引用对A进行初始化，所以这是没有问题的

* **AOP循环依赖**

  加入AOP依赖，通过@EnableAspectAutoProxy注解导入的AnnotationAwareAspectJAutoProxyCreator**其中实现了getEarlyBeanReference**

  ```java
  org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator
  
  @Override
  public Object getEarlyBeanReference(Object bean, String beanName) {
      Object cacheKey = getCacheKey(bean.getClass(), beanName);
      this.earlyProxyReferences.put(cacheKey, bean);
      // 如果需要代理，返回一个代理对象，不需要代理，直接返回当前传入的这个bean对象
      return wrapIfNecessary(bean, beanName, cacheKey);
  }
  ```

> **为什么要使用三级缓存**
>
> **三级缓存的目的在于延迟对实例化阶段生成的对象的代理，只有真正发生循环依赖的时候，才去提前生成代理对象，否则只会创建一个工厂并将其放入到三级缓存中，但是不会去通过这个工厂去真正创建对象**

## Spring上下文生命周期

### 创建Spring应用上下文

```java
org.springframework.context.annotation.AnnotationConfigApplicationContext
    
public class AnnotationConfigApplicationContext extends GenericApplicationContext implements AnnotationConfigRegistry {

	private final AnnotatedBeanDefinitionReader reader;

	private final ClassPathBeanDefinitionScanner scanner;
    
    public AnnotationConfigApplicationContext() {
		this.reader = new AnnotatedBeanDefinitionReader(this);
		this.scanner = new ClassPathBeanDefinitionScanner(this);
	}
    
    ...
}
```

其中创建AnnotatedBeanDefinitionReader类时，会调用构造器

```java
org.springframework.context.annotation.AnnotatedBeanDefinitionReader#AnnotatedBeanDefinitionReader

public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry, Environment environment) {
    
    this.registry = registry;
    this.conditionEvaluator = new ConditionEvaluator(registry, environment, null);
    AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
}
```

在构造器中调用方法registerAnnotationConfigProcessors

```java
org.springframework.context.annotation.AnnotationConfigUtils#registerAnnotationConfigProcessors

public static Set<BeanDefinitionHolder> registerAnnotationConfigProcessors(
    BeanDefinitionRegistry registry, @Nullable Object source) {

    ...

    // 注册ConfigurationClassPostProcessor，实现了BeanDefinitionRegistryPostProcessor接口: 负责所有bean的注册
    if (!registry.containsBeanDefinition(CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(ConfigurationClassPostProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME));
    }
	
   	// 注册AutowiredAnnotationBeanPostProcessor: 负责处理@Autowire注解
    if (!registry.containsBeanDefinition(AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(AutowiredAnnotationBeanPostProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME));
    }

    // 注册CommonAnnotationBeanPostProcessor: 负责处理@Resource注解
    if (jsr250Present && !registry.containsBeanDefinition(COMMON_ANNOTATION_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(CommonAnnotationBeanPostProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, COMMON_ANNOTATION_PROCESSOR_BEAN_NAME));
    }

    ...

    // 注册EventListenerMethodProcessor：负责处理@EventListener标注的方法，即事件处理器
    if (!registry.containsBeanDefinition(EVENT_LISTENER_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(EventListenerMethodProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_PROCESSOR_BEAN_NAME));
    }

    // 注册DefaultEventListenerFactory：负责将@EventListener标注的方法包装为ApplicationListener对象
    if (!registry.containsBeanDefinition(EVENT_LISTENER_FACTORY_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(DefaultEventListenerFactory.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_FACTORY_BEAN_NAME));
    }

    return beanDefs;
}
```

### Spring上下文启动准备阶段

```java
org.springframework.context.support.AbstractApplicationContext#refresh#prepareRefresh

protected void prepareRefresh() {
    // 标记启动时间
    this.startupDate = System.currentTimeMillis();
    this.closed.set(false);
    this.active.set(true);

    // 初始化PropertySource，留个子类去实现的，可以在这个方法中扩展属性配置信息，丢到this.environment中
    initPropertySources();

    // 验证环境配置中是否包含必须的配置参数信息
    getEnvironment().validateRequiredProperties();

    // earlyApplicationListeners用来存放早期的事件监听器
    if (this.earlyApplicationListeners == null) {
        this.earlyApplicationListeners = new LinkedHashSet<>(this.applicationListeners);
    }
    else {
        // 重置applicationListeners，并且更新集合
        this.applicationListeners.clear();
        this.applicationListeners.addAll(this.earlyApplicationListeners);
    }

    // earlyApplicationEvents用来存放早期的事件
    this.earlyApplicationEvents = new LinkedHashSet<>();
}
```

> 早期事件和早期事件监听器，针对相关组件没有准备就绪时，存放，待到组件初始化之后，发布添加存放的早期事件和监听器

### BeanFactory创建阶段

```java
org.springframework.context.support.AbstractApplicationContext#refresh#obtainFreshBeanFactory

protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
    // 刷新BeanFactory
    refreshBeanFactory();
    // 返回Spring上下文中创建好的BeanFactory
    return getBeanFactory();
}
```

### BeanFactory准备阶段

```java
org.springframework.context.support.AbstractApplicationContext#refresh#prepareBeanFactory

protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
	// 设置类加载器
    beanFactory.setBeanClassLoader(getClassLoader());
    // 设置spel表达式解析器
    beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));
    // 设置属性编辑器注册器
    beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));

    // 添加ApplicationContextAwareProcessor，当Bean继承了xxxAware接口，会自动回调，注入对象
    beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
    beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
    beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
    beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
    beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
    beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
    beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);

    // 当需要指定类型属性时，自动注入
    beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
    beanFactory.registerResolvableDependency(ResourceLoader.class, this);
    beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
    beanFactory.registerResolvableDependency(ApplicationContext.class, this);

    // 处理当Bean实现了ApplicationListener接口，会被放在容器监听列表
    beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));

    if (beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
        beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
        beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
    }

    // 注册默认环境到容器中，bean名称environment
    if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
        beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
    }
    // 将系统属性注册到容器中，bean名称systemProperties
    if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {
        beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
    }
    // 将系统环境变量配置信息注册到容器中，bean名称systemEnvironment
    if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) {
        beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
    }
}
```

### BeanFactor后置处理阶段

```java
org.springframework.context.support.AbstractApplicationContext#refresh#postProcessBeanFactory

protected void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
}
```

这个方法主要留给子类进行实现，此阶段Bean还未实例化，**其中添加自定义的BeanPostProcessor**

### 注册BeanFactoryPostProcessor阶段

```java
org.springframework.context.support.AbstractApplicationContext#refresh#invokeBeanFactoryPostProcessors

protected void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory) {
    PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory, getBeanFactoryPostProcessors());

    if (beanFactory.getTempClassLoader() == null && beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
        beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
        beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
    }
}
```

此阶段主要就是从Spring容器中找到BeanFactoryPostProcessor接口的所有实现类，完成所有Bean注册功能

**仅仅是完成Bean注册**，即Bean的定义信息转换为BeanDefinition对象，注册到Spring容器中，此时Bean还未实例化

阶段一中创建的ConfigurationClassPostProcessor实现了BeanFactoryPostProcessor接口

```shell
@Configuration
@Component
@PropertySource
@PropertySources
@ComponentScan
@ComponentScans
@Import
@ImportResource
@Bean
```

上述注解修饰的类都将被注册为Bean

BeanDefinitionRegistryPostProcessor是BeanFactoryPostProcessor的子类，启动提供postProcessorBeanDefinitionRegistry方法，参数是一个Bean定义注册器，**可以内部提供方法用来向容器中注册bean**

```java
org.springframework.beans.factory.support.BeanDefinitionRegistryPostProcessor

public interface BeanDefinitionRegistryPostProcessor extends BeanFactoryPostProcessor {

	void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException;

}
```

还可以通过BeanFactoryPostProcessor类型的Bean调用其postProcessorBeanFactory对BeanFactory中的一些信息进行修改

```java
@FunctionalInterface
public interface BeanFactoryPostProcessor {

	void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;

}
```

**重要的BeanFactoryPostProcessor实现类**

* **PropertySourcePlaceHolderConfigurer**

  Spring就是在这个类中处理xml属性中的${xxx}并进行解析

* **CustomScopeConfigurer**

  向Spring容器中注册自定义的Scope对象

* **EventListenerMethodProcessor**

  处理@EventListener注解

### 注册BeanPostProcessor阶段

```java
org.springframework.context.support.AbstractApplicationContext#refresh#registerBeanPostProcessors

protected void registerBeanPostProcessors(ConfigurableListableBeanFactory beanFactory) {
    PostProcessorRegistrationDelegate.registerBeanPostProcessors(beanFactory, this);
}
```

这个阶段遍历容器中的BeanDefinition列表，将所有实现了BeanPostProcessor接口的Bean添加到BeanPostProcessor列表中

### 初始化MessageSource阶段

```java
org.springframework.context.support.AbstractApplicationContext#refresh#initMessageSource

protected void initMessageSource() {
    ConfigurableListableBeanFactory beanFactory = getBeanFactory();
    if (beanFactory.containsLocalBean(MESSAGE_SOURCE_BEAN_NAME)) {
        this.messageSource = beanFactory.getBean(MESSAGE_SOURCE_BEAN_NAME, MessageSource.class);
        // 设置父MessageSource
        if (this.parent != null && this.messageSource instanceof HierarchicalMessageSource) {
            HierarchicalMessageSource hms = (HierarchicalMessageSource) this.messageSource;
            if (hms.getParentMessageSource() == null) {
                // 如果没有父MessageSource，设置上下文作为父
                hms.setParentMessageSource(getInternalParentMessageSource());
            }
        }
    }
    else {
        // 创建一个空MessageSource
        DelegatingMessageSource dms = new DelegatingMessageSource();
        dms.setParentMessageSource(getInternalParentMessageSource());
        this.messageSource = dms;
        beanFactory.registerSingleton(MESSAGE_SOURCE_BEAN_NAME, this.messageSource);
    }
}
```

### 初始化Spring广播器阶段

```java
org.springframework.context.support.AbstractApplicationContext#refresh#initApplicationEventMulticaster

protected void initApplicationEventMulticaster() {
    ConfigurableListableBeanFactory beanFactory = getBeanFactory();
    if (beanFactory.containsLocalBean(APPLICATION_EVENT_MULTICASTER_BEAN_NAME)) {
        // 获取容器中已经存在的广播器
        this.applicationEventMulticaster =
            beanFactory.getBean(APPLICATION_EVENT_MULTICASTER_BEAN_NAME, ApplicationEventMulticaster.class);
    }
    else {
        // 创建默认广播器
        this.applicationEventMulticaster = new SimpleApplicationEventMulticaster(beanFactory);
        beanFactory.registerSingleton(APPLICATION_EVENT_MULTICASTER_BEAN_NAME, this.applicationEventMulticaster);
    }
}
```

### Spring上下文刷新阶段

```java
org.springframework.context.support.AbstractApplicationContext#refresh#onRefresh

protected void onRefresh() throws BeansException {
    // 子类实现
}
```

用来**初始化其他特殊Bean**，由子类实现

### 事件监听器注册阶段

```java
org.springframework.context.support.AbstractApplicationContext#refresh#onRefresh
    
protected void registerListeners() {
    // 注册静态指定的监听器
    for (ApplicationListener<?> listener : getApplicationListeners()) {
        getApplicationEventMulticaster().addApplicationListener(listener);
    }

    // 从容器中获取监听器bean
    String[] listenerBeanNames = getBeanNamesForType(ApplicationListener.class, true, false);
    for (String listenerBeanName : listenerBeanNames) {
        // 添加监听器
        getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);
    }

    // 发布早期事件
    Set<ApplicationEvent> earlyEventsToProcess = this.earlyApplicationEvents;
    this.earlyApplicationEvents = null;
    if (!CollectionUtils.isEmpty(earlyEventsToProcess)) {
        for (ApplicationEvent earlyEvent : earlyEventsToProcess) {
            getApplicationEventMulticaster().multicastEvent(earlyEvent);
        }
    }
}
```

### 实例化所有单例（非Lazy）

```java
org.springframework.context.support.AbstractApplicationContext#refresh#finishBeanFactoryInitialization

protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
    ...
        
    // 添加${}表达式解析器
    if (!beanFactory.hasEmbeddedValueResolver()) {
        beanFactory.addEmbeddedValueResolver(strVal -> getEnvironment().resolvePlaceholders(strVal));
    }
    
    ...

    // 冻结所有Bean定义，表示已注册的Bean定义不会被进一步修改或处理
    beanFactory.freezeConfiguration();

    // 实例化所有单例Bean，通常scope=singleton的bean
    beanFactory.preInstantiateSingletons();
}
```

这里调用preInstantiationSingletons方法

```java
org.springframework.beans.factory.support.DefaultListableBeanFactory#preInstantiateSingletons

@Override
public void preInstantiateSingletons() throws BeansException {

    // beanDefinitionNames表示当前BeanFactory中定义的bean名称列表
    List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);

    // 循环实例化bean
    for (String beanName : beanNames) {
        ...
    }

    // 变量bean，若bean实现了SmartInitializingSingleton接口，将调用其afterSingletonsInstantiated()方法
    for (String beanName : beanNames) {
        ...
    }
}
```

### 刷新完成阶段

```java
org.springframework.beans.factory.support.DefaultListableBeanFactory#finishRefresh

protected void finishRefresh() {
    // 清理资源缓存
    clearResourceCaches();

    // 为此上下文初始化生命周期处理器
    initLifecycleProcessor();

    // 首先将刷新传播到生命周期处理器
    getLifecycleProcessor().onRefresh();

    // 发布ContextRefreshedEvent事件
    publishEvent(new ContextRefreshedEvent(this));

    ...
}
```

需要关注初始化生命周期处理器

```java
org.springframework.beans.factory.support.DefaultListableBeanFactory#initLifecycleProcessor

protected void initLifecycleProcessor() {
    
    ConfigurableListableBeanFactory beanFactory = getBeanFactory();
    //spring容器中是否有名称为"lifecycleProcessor"的bean
    if (beanFactory.containsLocalBean("lifecycleProcessor")) {
        //从spring容器中找到LifecycleProcessor，赋给this.lifecycleProcessor
        this.lifecycleProcessor =
            beanFactory.getBean("lifecycleProcessor", LifecycleProcessor.class);
    }
    else {
        //如果spring容器中没有自定义lifecycleProcessor，那么创建一个默认的DefaultLifecycleProcessor
        DefaultLifecycleProcessor defaultProcessor = new DefaultLifecycleProcessor();
        defaultProcessor.setBeanFactory(beanFactory);
        //设置当前spring应用上下文的生命周期处理器
        this.lifecycleProcessor = defaultProcessor;
        //将其注册到spring容器中
        beanFactory.registerSingleton("lifecycleProcessor", this.lifecycleProcessor);
    }
}
```

生命周期处理器

```java
org.springframework.context.LifecycleProcessor

public interface LifecycleProcessor extends Lifecycle {

    /**
	 * 上下文刷新通知
	 */
    void onRefresh();

    /**
	 * 上下文关闭阶段的通知
	 */
    void onClose();

}
```

该阶段调用生命周期处理器的onRefresh方法

```java
org.springframework.context.support.DefaultLifecycleProcessor#onRefresh

@Override
public void onRefresh() {
    startBeans(true);
    this.running = true;
}
```

startBeans主要从容器中找到**所有实现Lifecycle接口的bean**，调用start方法

```java
org.springframework.context.support.DefaultLifecycleProcessor#startBeans

private void startBeans(boolean autoStartupOnly) {
    Map<String, Lifecycle> lifecycleBeans = getLifecycleBeans();
    Map<Integer, LifecycleGroup> phases = new HashMap<>();
    lifecycleBeans.forEach((beanName, bean) -> {
        if (!autoStartupOnly || (bean instanceof SmartLifecycle && ((SmartLifecycle) bean).isAutoStartup())) {
            int phase = getPhase(bean);
            LifecycleGroup group = phases.get(phase);
            if (group == null) {
                group = new LifecycleGroup(phase, this.timeoutPerShutdownPhase, lifecycleBeans, autoStartupOnly);
                phases.put(phase, group);
            }
            group.add(beanName, bean);
        }
    });
    if (!phases.isEmpty()) {
        List<Integer> keys = new ArrayList<>(phases.keySet());
        Collections.sort(keys);
        for (Integer key : keys) {
            phases.get(key).start();
        }
    }
}
```

### Spring上下文关闭阶段

```java
org.springframework.context.support.AbstractApplicationContext#doClose

protected void doClose() {
    // 判断是否需要关闭容器，通过判断active和cas设置close值
    if (this.active.get() && this.closed.compareAndSet(false, true)) {

        LiveBeansView.unregisterApplicationContext(this);

        try {
            // 发布关闭事件ContextClosedEvent
            publishEvent(new ContextClosedEvent(this));
        }
        catch (Throwable ex) {
            logger.warn("Exception thrown from ApplicationListener handling ContextClosedEvent", ex);
        }

        // 调用生命周期处理器的onClose方法与刷新的onRefresh相似
        if (this.lifecycleProcessor != null) {
            try {
                this.lifecycleProcessor.onClose();
            }
            catch (Throwable ex) {
                logger.warn("Exception thrown from LifecycleProcessor on context close", ex);
            }
        }

        // 销毁上下文的BeanFactory中所有缓存的单例
        // 实现了org.springframework.beans.factory.DisposableBean接口，此时会被调用
        // 实现标注了@PreDestroy注解方法
        destroyBeans();

        // 关闭BeanFactory本身
        closeBeanFactory();

        // 就给子类去扩展的
        onClose();

        // 恢复事件监听器列表至刷新之前的状态，即将早期的事件监听器还原
        if (this.earlyApplicationListeners != null) {
            this.applicationListeners.clear();
            this.applicationListeners.addAll(this.earlyApplicationListeners);
        }

        // 标记活动状态为：false
        this.active.set(false);
    }
}
```

关闭BeanFactory本身，**主要是还原当前上下文中BeanFactory属性**

```java
org.springframework.context.support.AbstractRefreshableApplicationContext#closeBeanFactory

@Override
protected final void closeBeanFactory() {
    DefaultListableBeanFactory beanFactory = this.beanFactory;
    if (beanFactory != null) {
        beanFactory.setSerializationId(null);
        this.beanFactory = null;
    }
}
```

## 父子容器

* **父子容器特点**

  1. 父容器和子容器是**相互隔离的**，内部**可以存在同名的Bean**

  2. **子容器可以访问父容器中的Bean**，而父容器**不可以**访问子容器中的Bean

  3. 调用子容器的getBean方法获取Bean时，**子容器会沿着当前容器向上面的容器查找**，直到找到对应的Bean为止

  4. 子容器中可以**通过任何注入方式注入父容器中的Bean**，而父容器中是**无法**注入子容器中的Bean

     > BeanFactory支持容器嵌套结构查询，即沿着当前容器向上面的容器查找，但是**ListableBeanFactory不支持**
     >
     > getBeanNamesForType是ListableBeanFactory中定义的，所以不支持嵌套结构查询
     >
     > 使用`org.springframework.beans.factory.BeanFactoryUtils`中所有名字**包含Ancestors**，这些都是支持嵌套查询的

* **SpringMVC父子容器**

  SpringMVC**只使用一个容器也是可以启动的**，不一定需要嵌套容器

  使用嵌套容器启动的原因——

  * 避免Service层中注入Controller层的bean，**导致结构混乱**
  * 父子容器需求不一样，**相互隔离**，父子容器加载速度提升

## Spring国际化

AbstractApplicationContext接口实现了国际化接口

1. **创建国际化文件**

   **国际化文件命名格式：名称_语言\_地区.properties**

   ```properties
   # 案例 message_en_GB.properties
   name=Full Name
   ```

2. **注册国际化配置Bean**

   ```java
   @Configuration	
   public class MainConfig {
       @Bean
       public ResourceBundleMessageSource messageSource() {
           ResourceBundleMessageSource result = new ResourceBundleMessageSource();
           result.setBasenames("message");
           return result;
       }
   }
   ```

3. **调用AbstractApplicationContext的getMessage来获取国际化信息，其内部将交给容器中的messageSource处理**

**动态参数设置**

在国际化文件中**使用`{index}`**来获取getMessage中携带的动态参数

使用`ReloadableResourceBundleMessageSource`作为MessageSource实现类，可以**监控国际化文件变化**

```java
/**
 * -1: 表示永远缓存
 * 0: 表示每次获取国际化信息的时候，都会重新读取国际化文件
 * 大于0：上次读取配置文件的时间距离当前时间超过了这个时间，重新读取国际化文件
 */
public void setCacheMillis(long cacheMillis);
public void setCacheSeconds(long cacheSeconds)
```

**线上环境，缓存时间最好设置大一点，性能会好一些**

**从DB中获取国际化信息**

使用`StaticMessageSource`接口，可以**编程方式添加国际化信息**

```java
public void addMessage(String code, Locale locale, String msg);
public void addMessages(Map<String, String> messages, Locale locale);
```

```java
public class MessageSourceFromDb extends StaticMessageSource implements InitializingBean {
    @Override
    public void afterPropertiesSet() throws Exception {
        this.addMessage("desc", Locale.UK, "from db");
    }
}
```

## Spring事务机制

* **硬编码方式实现**

  1. **定义事件**

     自定义事件，需要继承**ApplicationEvent**

     ```java
     org.springframework.context.ApplicationEvent
     
     public abstract class ApplicationEvent extends EventObject {
     
     	private static final long serialVersionUID = 7099057708183571937L;
     
         // 记录事件触发时间
     	private final long timestamp;
     
     	public ApplicationEvent(Object source) {
     		super(source);
     		this.timestamp = System.currentTimeMillis();
     	}
     	
         // 返回事件触发时间
     	public final long getTimestamp() {
     		return this.timestamp;
     	}
     
     }
     ```

  1. **定义监听器**

     自定义事件监听器，需要实现ApplicationListener接口

     ```java
     org.springframework.context.ApplicationListener
     
     @FunctionalInterface
     public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {
     
     	/**
     	 * 处理ApplicationEvent
     	 */
     	void onApplicationEvent(E event);
     
     }
     ```
  
  3. **创建事件广播器**
  
     可以实现ApplicationEventMulticaster接口，也可以使用系统提供的SimpleApplicationEventMulticaster
  
     ```java
     org.springframework.context.event.ApplicationEventMulticaster
     
     public interface ApplicationEventMulticaster {
     
     	/**
     	 * 添加事件监听器
     	 */
     	void addApplicationListener(ApplicationListener<?> listener);
     
     	/**
     	 * 添加事件监听器Bean
     	 */
     	void addApplicationListenerBean(String listenerBeanName);
     
     	/**
     	 * 删除事件监听器
     	 */
     	void removeApplicationListener(ApplicationListener<?> listener);
     
     	/**
     	 * 删除事件监听器Bean通过beanName
     	 */
     	void removeApplicationListenerBean(String listenerBeanName);
     
     	/**
     	 * 删除所有事件监听器
     	 */
     	void removeAllListeners();
     
     	/**
     	 * 发布事件给所有事件监听器
     	 */
     	void multicastEvent(ApplicationEvent event);
     
     	/**
     	 * 发布指定类型的事件给所有事件监听器
     	 */
     	void multicastEvent(ApplicationEvent event, @Nullable ResolvableType eventType);
     
     }
     ```
  
  4. **向广播器中注册事件监听器**
  
  
  5. **通过广播器发布事件**
  
  AbstractApplicationContext**集成了事件广播器**
  
  AbstractApplicationContext中的**publishEvent方法也能实现广播事件的效果**，因为实现了ApplicationEventPublisher
  
  ```java
  org.springframework.context.ApplicationEventPublisher
  
  @FunctionalInterface
  public interface ApplicationEventPublisher {
  
  	default void publishEvent(ApplicationEvent event) {
  		publishEvent((Object) event);
  	}
  
  	void publishEvent(Object event);
  
  }
  
  ```
  
  通过ApplicationEventPublisherAware接口，可以获取ApplicationEventPublisher对象
  
  ```java
  org.springframework.context.ApplicationEventPublisherAware
  
  public interface ApplicationEventPublisherAware extends Aware {
  
  	void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher);
  
  }
  ```
  
* **面向接口方式实现**

  监听器需要实现ApplicationListener接口，Spring容器创建Bean过程中，**会判断是否有ApplicationListener**，就会将其添**加到容器自带的applicationEventMulticaster中**

  ```java
  org.springframework.context.support.ApplicationListenerDetector#postProcessAfterInitialization
  
  @Override
  public Object postProcessAfterInitialization(Object bean, String beanName) {
      if (bean instanceof ApplicationListener) {
          Boolean flag = this.singletonNames.get(beanName);
          if (Boolean.TRUE.equals(flag)) {
              // 将监听器添加到自带的广播器中
              this.applicationContext.addApplicationListener((ApplicationListener<?>) bean);
          }
          else if (Boolean.FALSE.equals(flag)) {
             	// 将监听器Bean删除
              this.singletonNames.remove(beanName);
          }
      }
      return bean;
  }
  ```


* **面向注解方式**

  直接使用@EventListener标注一个Bean方法即可

  **原理**

  EventListenerMethodProcessor实现了SmarInitializingSingleton接口，其中的afterSingletonsInstantiated方法**会在所有单例bean创建完成之后调用**

  ```java
  org.springframework.context.event.EventListenerMethodProcessor#afterSingletonsInstantiated
  
  @Override
  public void afterSingletonsInstantiated() {
      ConfigurableListableBeanFactory beanFactory = this.beanFactory;
  
      String[] beanNames = beanFactory.getBeanNamesForType(Object.class);
      for (String beanName : beanNames) {
          if (!ScopedProxyUtils.isScopedTarget(beanName)) {
              Class<?> type = null;
              try {
                  type = AutoProxyUtils.determineTargetClass(beanFactory, beanName);
              }
              catch (Throwable ex) {
                  ...
              }
              if (type != null) {
                  if (ScopedObject.class.isAssignableFrom(type)) {
                      try {
                          Class<?> targetClass = AutoProxyUtils.determineTargetClass(
                              beanFactory, ScopedProxyUtils.getTargetBeanName(beanName));
                          if (targetClass != null) {
                              type = targetClass;
                          }
                      }
                      catch (Throwable ex) {
                          ...
                      }
                  }
                  try {
                      // 最后调用addApplicationListener添加到容器中
                      processBean(beanName, type);
                  }
                  catch (Throwable ex) {
                      ...
                  }
              }
          }
      }
  }
  ```

* **监听支持排序**

* **监听器异步模式**

  默认**SimpleApplicationEventMulticaster支持监听器异步调用的**

  主要关于其中的`Executor`属性

  ```java
  org.springframework.context.event.SimpleApplicationEventMulticaster#multicastEvent
  
  @Override
  public void multicastEvent(final ApplicationEvent event, @Nullable ResolvableType eventType) {
      ResolvableType type = (eventType != null ? eventType : resolveDefaultEventType(event));
      Executor executor = getTaskExecutor();
      for (ApplicationListener<?> listener : getApplicationListeners(event, type)) {
          if (executor != null) { //@1
              executor.execute(() -> invokeListener(listener, event));
          }
          else {
              invokeListener(listener, event);
          }
      }
  }
  ```

  **如果当前executor不为空**，将挺起就会被异步调用，通过ivokeListener方法调用监听器

  创建广播器时，**设置参数executor**，开启异步调用

## @Async实现方法异步操作

**实现方式**

1. 给需要异步处理的方法添加@Async注解
2. 给配置类添加@EnableAsync

**获取异步执行结果**

```java
java.util.concurrent.Future

public interface Future<V> {
    boolean cancel(boolean var1);

    boolean isCancelled();

    boolean isDone();

    V get() throws InterruptedException, ExecutionException;

    V get(long var1, TimeUnit var3) throws InterruptedException, ExecutionException, TimeoutException;
}
```

* **返回类型不为Future**

  执行结束后，立即返回，无法获取方法返回值

* **返回类型为Future**

  若需要获取异步执行结果，返回值必须为Future，**使用Spring提供静态方法返回`AsyncResult.forValue(String)`**

**自定义异步执行的线程池**

默认情况下，**@EnableAsync使用内置线程池来异步调用方法**

```java
// 方法1：Spring容器中定义一个线程池类型的bean，bean名称必须为taskExecutor
@Bean
public Executor taskExecutor() {
    ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
    executor.setCorePoolSize(10);
    executor.setMaxPoolSize(100);
    executor.setThreadNamePrefix("my-thread-");
    return executor;
}

// 方法2：定义实现AsyncConfigurer接口的Bean，getAsyncExecutor方法，返回自定义线程池
@Bean
public AsyncConfigurer asyncConfigurer(Executor executor) {
    return new AsyncConfigurer() {
        @Nullable
        @Override
        public Executor getAsyncExecutor() {
            return executor;
        }
    };
}
```

**自定义异常处理**

异步方法发生异常，**可以通过自定义异常处理**解决

* 当返回值为Future，方法内部有异常时，异常会向外抛出，可以**通过对Future.get采用try-catch捕获异常**
* 返回值不为Future，可以自定义Bean，**实现AsyncConfigurer接口中的getAsyncUncaughExceptionHandler方法**，返回自定义异常处理器

## @Scheduled定时器

**@Scheduled配置定时规则**

```java
org.springframework.scheduling.annotation.Scheduled

@Target({ElementType.METHOD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Repeatable(Schedules.class)
public @interface Scheduled {
 
 String cron() default "";
 
 String zone() default "";
 
 long fixedDelay() default -1;
 
 String fixedDelayString() default "";
 
 long fixedRate() default -1;
 
 String fixedRateString() default "";
 
 long initialDelay() default -1;
 
 String initialDelayString() default "";
 
}
```

* **cron**

  该参数接受一个cron表达式，**字符串以5或6个空格隔开**，分开共6或7个域，每一个域表达一个含义

  ```shell
  [秒] [分] [小时] [日] [月] [周] [年]
  ```
  
  > 年不是必需要的域，可以省略年

  | 说明 | 允许使用的通配符 |
  | ---- | ---------------- |
  | 秒   | , - * /          |
  | 分   | , - * /          |
  | 时   | , - * /          |
  | 日   | ,  - * ? / L W   |
  | 月   | , - * /          |
  | 周   | , - * ? / L #    |
  | 年   | , - * /          |
  
  **通配符说明**

  * \*表示所有值
  * ?表示不指定值，使用场景不需要关心当前设置这个字段的值
  * -表示区间
  * ,表示多个值
  * /表示递增触发，例：5/15，从5秒开始，每15秒执行依次
  * L表示最后的意思
  * W表示例指定日期最近的工作日
  * #表示每个月的第几个周几，例：6#3，第三个周6
  
* **zone**

  时区，**接受一个java.util.TimeZone#ID**，cron表达式会基于时区分析，默认是取服务器所在地的时区，**该字段一般留空**

* **fixedDelay**

  上一次执行完毕时间点之后的多长时间再执行（单位为毫秒）

* **fixedDelayString**

  与fixedDelay相同，只是使用字符串的形式，唯一不同的是支持占位符

* **fixedRate**

  上一次开始执行时间之后多长时间再执行

* **fixedRateString**

  与fixedRate相同，唯一不同支持占位符

* **initialDelay**

  第一次延迟多长时间再执行

* **initialDelayString**

  与initialDelay意思相同，只是使用字符串的形式，唯一不同支持占位符

@Schedules是@Schedule注解的合集

```java
org.springframework.scheduling.annotation.Schedules

@Target({ElementType.METHOD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Schedules {
 Scheduled[] value();
}
```

**为定时器自定义线程池**

默认情况下使用`new ScheduledThreadPoolExecutor(1)`线程池执行定时任务

只有一个线程，如果需要定时执行的任务太多，只能排队执行

**定义一个ScheduleExecutorService类型的Bean，名称必须为taskScheduler**

# Spring AOP

## 常用代理模式

### Jdk动态代理

**代理对象类**

```java
java.lang.reflect.Proxy
    
public class Proxy implements Serializable {
 
    /**
     * 创建代理的实例对象
     * 当调用代理对象的任何方法时，会被InvocationHandler接口的invoke方法处理
     */
    public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h) {
     	...   
    }
    
    /**
     * 获取代理对象的InvocationHandler对象
     */
    public static InvocationHandler getInvocationHandler(Object proxy) throws IllegalArgumentException {
    	...
    }
}
```

**案例** 

```java
public class TestProxy<T> {
    private T target;

    public TestProxy(T target) {
        this.target = target;
    }

    public T getProxyObject() {
        return (T) Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(), 
                                          (proxy, method, args) -> {
        
            Object result = method.invoke(target, args);

            return result;
        });
    }
}
```

### CGLIB动态代理

**本质上它动态的生成一个子类去覆盖所要代理的类**

Enhancer既能够代理普通的class，也能够代理接口

**Enhancer不能拦截final方法**

CGLIB会根据Callback和CallbackFilter生成二进制字节文件，通过ClassLoader生成对应类

#### 使用流程

1. 创建Enhancer对象
2. 通过setSuperclass设置父类型
3. 设置回调函数，需要实现`org.springframework.cglib.proxy.Callback`，MethodInterceptor实现了Callback接口，所有方法调用都会被intercept方法拦截
4. 获取代理对象，调用enhancer.create方法

#### Callback接口常用的实现类

* MethodInterceptor——动态获取返回值

  ```java
  org.springframework.cglib.proxy.MethodInterceptor
      
  public interface MethodInterceptor extends Callback {
      Object intercept(Object var1, Method var2, Object[] var3, MethodProxy var4) throws Throwable;
  }
  ```

* FixedValue——返回固定值

  ```java
  org.springframework.cglib.proxy.FixedValue
      
  public interface FixedValue extends Callback {
      Object loadObject() throws Exception;
  }
  ```

* NoOp.INSTANCE——不返回任何值

  ```java
  org.springframework.cglib.proxy.NoOp
  
  public interface NoOp extends Callback {
      NoOp INSTANCE = new NoOp() {
      };
  }
  ```

* LazyLoader——实现懒加载的Callback，初次调用时会触发返回对象，但是之后都会返回第一次调用生成的对象

  ```java
  org.springframework.cglib.proxy.LazyLoader
  
  public interface LazyLoader extends Callback {
      Object loadObject() throws Exception;
  }
  ```

* Dispatcher——与LazyLoader相同，懒加载，但是每次都会调用生成返回对象

  ```java
  org.springframework.cglib.proxy.Dispatcher
  
  public interface Dispatcher extends Callback {
      Object loadObject() throws Exception;
  }
  ```

#### 自定义代理对象类命名规则

可以通过自定义NamingPolicy接口实现类，修改生成代理对象类的命名规则

默认实现类是DefaultNamingPolicy

```java
org.springframework.cglib.core.DefaultNamingPolicy

public class DefaultNamingPolicy implements NamingPolicy {
    public static final DefaultNamingPolicy INSTANCE = new DefaultNamingPolicy();
    private static final boolean STRESS_HASH_CODE = Boolean.getBoolean("org.springframework.cglib.test.stressHashCodes");

    public DefaultNamingPolicy() {
    }

    public String getClassName(String prefix, String source, Object key, Predicate names) {
        if (prefix == null) {
            prefix = "org.springframework.cglib.empty.Object";
        } else if (prefix.startsWith("java")) {
            prefix = "$" + prefix;
        }

        String base = prefix + "$$" + source.substring(source.lastIndexOf(46) + 1) + this.getTag() + "$$" + Integer.toHexString(STRESS_HASH_CODE ? 0 : key.hashCode());
        String attempt = base;

        for(int var7 = 2; names.evaluate(attempt); attempt = base + "_" + var7++) {
        }

        return attempt;
    }

    protected String getTag() {
        return "ByCGLIB";
    }
	
    ...
}
```

自定义NamingPolicy，通常回继承DefalultNamingPolicy来实现，Spring中默认提供——

```java
public class SpringNamingPolicy extends DefaultNamingPolicy {

	public static final SpringNamingPolicy INSTANCE = new SpringNamingPolicy();

	@Override
	protected String getTag() {
		return "BySpringCGLIB";
	}

}
```

#### Objenesis：生成实例对象的方式

> 主要针对没有合适构造器的类

使用Objenesis**不会使用目标对象的构造器生成实例**

```java
org.springframework.objenesis.Objenesis

public interface Objenesis {
    <T> T newInstance(Class<T> var1);

    <T> ObjectInstantiator<T> getInstantiatorOf(Class<T> var1);
}
```

#### CallbackFilter

**可以使用CallbackFilter来判断使用不同的Callback实现类**

```java
org.springframework.cglib.proxy.CallbackFilter

public interface CallbackFilter {
    int accept(Method var1);

    boolean equals(Object var1);
}
```

使用CallbackHelper优化CGLIB动态代理

```java
org.springframework.cglib.proxy.CallbackHelper
    
public abstract class CallbackHelper implements CallbackFilter {
    
    /**
     * 返回Callback实例对象
     */
    protected abstract Object getCallback(Method var1);
   	
    ...
}
```

**可以通过CallbackHelper获取Callback，并且CallbackHelper实现了CallbackFilter**

## Spring中AOP对应实现

* **连接点（JoinPoint）**

  指定织入加强的位置，**最常见的JoinPoint就是方法调用**

  ```java
  org.aspectj.lang.Joinpoint
  
  public interface JoinPoint {
  	
      // 连接点所在位置的相关信息
      String toString();
  
      /**
       * 返回连接点所在位置简短的相关信息
       */
      String toShortString();
  
      /**
       * 返回连接点所在位置全部相关信息
       */
      String toLongString();
  
      /**
       * 返回当前的Aop代理对象
       */
      Object getThis();
  
      /**
       * 返回目标对象
       */
      Object getTarget();
  
      /**
       * 返回被加强的方法的参数列表
       */
      Object[] getArgs();
  
      /**
       * 获取连接点签名，这个对象可以获取目标方法的详细信息
       */
      Signature getSignature();
  
      /**
       * 返回连接点方法所在类文件中的位置
       */
      SourceLocation getSourceLocation();
  
      /** 
       * 返回连接点类型
       */
      String getKind();
  
      ...
  
  }
  ```

  proceedingJoinPoint是环绕通知连接点信息

  ```java
  org.aspectj.lang.ProceedingJoinPoint
      
  public interface ProceedingJoinPoint extends JoinPoint {
  
      void set$AroundClosure(AroundClosure arc);
  
       default void stack$AroundClosure(AroundClosure arc) {
      	 throw new UnsupportedOperationException();
       }
  
      /**
       * 继续执行下一个通知或者目标方法的调用
       */
  	Object proceed() throws Throwable;
  
      /**
       * 带参数的proceed
       */
  	Object proceed(Object[] args) throws Throwable;
  
  }
  ```

  Invocation接口继承了JoinPoin

  ```java
  org.aopalliance.intercept.Invocation
      
  public interface Invocation extends Joinpoint {
  
  	/**
  	 * 获取参数组成的数组
  	 */
  	Object[] getArguments();
  
  }
  ```

  用来表示连接点中方法的调用，可以获取调用过程中的目标方法

  ```java
  org.aopalliance.intercept.MethodInvocation
  
  public interface MethodInvocation extends Invocation {
  
  	/**
  	 * 获取被调用的方法，再方法调用时提供给拦截器
  	 */
  	Method getMethod();
  
  }
  ```

  表示代理方法的调用

  ```java
  org.springframework.aop.ProxyMethodInvocation
  
  public interface ProxyMethodInvocation extends MethodInvocation {
  
  	/**
  	 * 获取被调用的代理对象
  	 */
  	Object getProxy();
  
  	/**
  	 * 克隆一个MethodInvocation
  	 */
  	MethodInvocation invocableClone();
  
  	/**
  	 * 克隆一个MethodInvocation，并未Invocation指定参数
  	 */
  	MethodInvocation invocableClone(Object... arguments);
  
  	/**
  	 * 设置用于此链中通知的后续调用参数
  	 */
  	void setArguments(Object... arguments);
  
  	void setUserAttribute(String key, @Nullable Object value);
  
  	@Nullable
  	Object getUserAttribute(String key);
  
  }
  ```

  **具体实现类**

  * **ReflectiveMethodInvocation**

    ```java
    org.springframework.aop.framework.ReflectiveMethodInvocation
    ```

    **当代理对象是采用JDK动态代理创建**，通过代理对象来访问目标对象的方法的时，由这个类处理，**内部会通过递归调用方法拦截器**，最终会调用到目标方法

  * **CglibMethodInvocation**

    ```java
    org.springframework.aop.framework.CglibAopProxy.CglibMethodInvocation
    ```

    **当处理对象是采用CGLIB创建的**，通过代理对象来访问目标对象的方法，最终过程由这个类处理，**内部通过递归调用方法拦截器**，最终会调用到目标方法

* **通知（Advice）**

  **Advice定义了织入到连接点的逻辑**

  ```java
  org.aopalliance.aop.Advice
  
  public interface Advice {
  
  }
  ```

  前置通知标志性接口

  ```java
  org.springframework.aop.BeforeAdvice
      
  public interface BeforeAdvice extends Advice {
  
  }
  ```

  后置通知标志性接口

  ```java
  org.springframework.aop.AfterAdvice
  
  public interface AfterAdvice extends Advice {
  
  }
  ```

  此接口表示通用侦听器

  ```java
  org.aopalliance.intercept.Interceptor
  
  public interface Interceptor extends Advice {
  
  }
  ```

  MethodBeforeAdvice接口，**方法执行前通知**，可以执行方法前的逻辑

  ```java
  org.springframework.aop.MethodBeforeAdvice
  
  public interface MethodBeforeAdvice extends BeforeAdvice {
  
  	/**
  	 * 执行前置逻辑
  	 */
  	void before(Method method, Object[] args, @Nullable Object target) throws Throwable;
  
  }
  ```

  AfterReturningAdvice接口，方法执行后通知，**当目标方法有异常，那么通知会被跳过**

  ```java
  org.springframework.aop.AfterReturningAdvice
  
  public interface AfterReturningAdvice extends AfterAdvice {
  
  	/**
  	 * 执行后置逻辑，异常跳过
  	 */
  	void afterReturning(@Nullable Object returnValue, Method method, Object[] args, @Nullable Object target) throws Throwable;
  
  }
  ```

  异常触发接口，**方法执行过程中触发异常**，就会执行通知

  ```java
  org.springframework.aop.ThrowsAdvice
  
  public interface ThrowsAdvice extends AfterAdvice {
  
  }
  ```

  此接口没有任何方法，**因为方法由反射调用**，调用方法前三个参数是**可选**的，**最后一个参数是需要匹配的异常**

  ```java
  void afterThrowing([Method, args, target], ThrowableSubclass);
  ```

  方法拦截器，这个接口最强大，可以实现上面3种类型的通知，**上面三种通知，会通过适配器转换为Methodinterceptor**

  ```java
  org.aopalliance.intercept.MethodInterceptor
  
  @FunctionalInterface
  public interface MethodInterceptor extends Interceptor {
  
  	/**
  	 * Implement this method to perform extra treatments before and
  	 * after the invocation. Polite implementations would certainly
  	 * like to invoke {@link Joinpoint#proceed()}.
  	 * @param invocation the method invocation joinpoint
  	 * @return the result of the call to {@link Joinpoint#proceed()};
  	 * might be intercepted by the interceptor
  	 * @throws Throwable if the interceptors or the target object
  	 * throws an exception
  	 */
  	Object invoke(MethodInvocation invocation) throws Throwable;
  
  }org.aopalliance.intercept.MethodInterceptor
  ```

  **通知包装器**

  将各种Advice包装成MethodInterceptor类型，组成一个方法调用链

  以MethodBeforeAdviceInterceptor类演示，**关键方法就是invoke**

  ```java
  org.springframework.aop.framework.adapter.MethodBeforeAdviceInterceptor
  
  public class MethodBeforeAdviceInterceptor implements MethodInterceptor, BeforeAdvice, Serializable {
  
  	private final MethodBeforeAdvice advice;
  
  	public MethodBeforeAdviceInterceptor(MethodBeforeAdvice advice) {
  		Assert.notNull(advice, "Advice must not be null");
  		this.advice = advice;
  	}
  
  
  	@Override
  	public Object invoke(MethodInvocation mi) throws Throwable {
  		this.advice.before(mi.getMethod(), mi.getArguments(), mi.getThis());
  		return mi.proceed();
  	}
  
  }
  ```

* **切入点（Pointcut）**

  **代表一组JoinPoint中需要织入的逻辑**

  ```java
  org.springframework.aop.Pointcut
  
  public interface Pointcut {
  
  	/**
  	 * 返回类过滤器，可以指定需要拦截的类
  	 */
  	ClassFilter getClassFilter();
  
  	/**
  	 * 返回方法匹配器，可以指定需要拦截的方法
  	 */
  	MethodMatcher getMethodMatcher();
  
  	/**
  	 * 匹配所有对象的Poincut，内部两个过滤器默认都会返回true
  	 */
  	Pointcut TRUE = TruePointcut.INSTANCE;
  
  }
  ```

  ClassFilter接口，用来指定类过滤器

  ```java
  org.springframework.aop.ClassFilter
  
  @FunctionalInterface
  public interface ClassFilter {
  
  	/**
  	 * 用来判断目标类型是否匹配
  	 */
  	boolean matches(Class<?> clazz);
  
  
  	/**
  	 * 匹配满足所有类的实例
  	 */
  	ClassFilter TRUE = TrueClassFilter.INSTANCE;
  
  }
  ```

  MethodMatcher接口，用来过滤方法

  ```java
  org.springframework.aop.MethodMatcher
  
  public interface MethodMatcher {
  
  	/**
  	 * 执行静态检查过滤方法
  	 */
  	boolean matches(Method method, Class<?> targetClass);
  
  	/**
  	 * 是否动态匹配，即是否每次执行目标方法都验证一次
  	 */
  	boolean isRuntime();
  
  	/**
  	 * 动态匹配验证，可以传入调用目标方法的参数
  	 */
  	boolean matches(Method method, Class<?> targetClass, Object... args);
  
  
  	/**
  	 * 匹配所有方法的实例
  	 */
  	MethodMatcher TRUE = TrueMethodMatcher.INSTANCE;
  
  }
  ```

* **切面（Aspect）**

  Advisor是Pointcut和Advice的组合

  ```java
  org.springframework.aop
      
  public interface Advisor {
  
  	/**
  	 * 空的顾问实例
  	 */
  	Advice EMPTY_ADVICE = new Advice() {};
  
  
  	/**
  	 * 返回引用的通知
  	 */
  	Advice getAdvice();
  
  	boolean isPerInstance();
  
  }
  ```

  PointcutAdvisor，内部有方法获取Pointcut，**目标方法中实现各种增强功能基本上都通过PointcutAdvisor实现**

## SpringAOP代理创建原理

* **创建代理所需参数配置**

  创建代理所需参数配置主要通过AdvisedSupport实现

  ```java
  org.springframework.aop.framework.AdvisedSupport
      
  public class AdvisedSupport extends ProxyConfig implements Advised {
  
  	private static final long serialVersionUID = 2651364800145442165L;
  
  	public static final TargetSource EMPTY_TARGET_SOURCE = EmptyTargetSource.INSTANCE;
  
  	TargetSource targetSource = EMPTY_TARGET_SOURCE;
  
  	/** Whether the Advisors are already filtered for the specific target class. */
  	private boolean preFiltered = false;
  
  	/** 
       * 判断是否支持预过滤
       */
  	AdvisorChainFactory advisorChainFactory = new DefaultAdvisorChainFactory();
  
  	/** 
       * 调用链工厂，用来获取目标方法的调用链
       */
  	private transient Map<MethodCacheKey, List<Object>> methodCache;
  
  	/**
  	 * 代理对象需要实现的接口列表，保存在列表中以保持注册的顺序，以创建具有指定接口顺序的代理
  	 */
  	private List<Class<?>> interfaces = new ArrayList<>();
  
  	/**
  	 * 配置的切面列表，所有添加的Advise对象都会被包装成Advisor对象
  	 */
  	private List<Advisor> advisors = new ArrayList<>();
  
  	/**
  	 * 随advisors的修改而修改，为了更容易内部操作
  	 */
  	private Advisor[] advisorArray = new Advisor[0];
  
  	public AdvisedSupport() {
  		this.methodCache = new ConcurrentHashMap<>(32);
  	}
  
  	public AdvisedSupport(Class<?>... interfaces) {
  		this();
  		setInterfaces(interfaces);
  	}
  
      // 设置需要被代理的对象，目标对象会被包装为TargetSource格式的对象
  	public void setTarget(Object target) {
  		setTargetSource(new SingletonTargetSource(target));
  	}
  
      // 设置被代理的目标源
  	@Override
  	public void setTargetSource(@Nullable TargetSource targetSource) {
  		this.targetSource = (targetSource != null ? targetSource : EMPTY_TARGET_SOURCE);
  	}
      
  	@Override
  	public TargetSource getTargetSource() {
  		return this.targetSource;
  	}
  
  	/**
  	 * 设置被代理的目标类型
  	 */
  	public void setTargetClass(@Nullable Class<?> targetClass) {
  		this.targetSource = EmptyTargetSource.forClass(targetClass);
  	}
  	@Override
  	@Nullable
  	public Class<?> getTargetClass() {
  		return this.targetSource.getTargetClass();
  	}
  
      /**
       * 设置刺代理配置是否经过预筛选，通过目标对象调用代理的时候
       * 需要通过匹配的方式获取调用链列表，查找过程需要两个步骤
       * 1.类是否匹配
       * 2.方法是否匹配
       * 当这个属性被设置true时，会跳过第一步
       */
  	@Override
  	public void setPreFiltered(boolean preFiltered) {
  		this.preFiltered = preFiltered;
  	}
  	@Override
  	public boolean isPreFiltered() {
  		return this.preFiltered;
  	}
  
  	/**
  	 * 设置切片链工厂，当调用目标方法时，需要获取这个方法上匹配的切片列表
  	 * 获取目标方法上匹配的Advisor列表的功能就是AdvisorChainFactory负责
  	 */
  	public void setAdvisorChainFactory(AdvisorChainFactory advisorChainFactory) {
  		Assert.notNull(advisorChainFactory, "AdvisorChainFactory must not be null");
  		this.advisorChainFactory = advisorChainFactory;
  	}
  	public AdvisorChainFactory getAdvisorChainFactory() {
  		return this.advisorChainFactory;
  	}
  
  
  	/**
  	 * 设置需要被代理的接口
  	 */
  	public void setInterfaces(Class<?>... interfaces) {
  		Assert.notNull(interfaces, "Interfaces must not be null");
  		this.interfaces.clear();
  		for (Class<?> ifc : interfaces) {
  			addInterface(ifc);
  		}
  	}
  
  	/**
  	 * 为代理对象添加需要实现的接口
  	 */
  	public void addInterface(Class<?> intf) {
  		Assert.notNull(intf, "Interface must not be null");
  		if (!intf.isInterface()) {
  			throw new IllegalArgumentException("[" + intf.getName() + "] is not an interface");
  		}
  		if (!this.interfaces.contains(intf)) {
  			this.interfaces.add(intf);
  			adviceChanged();
  		}
  	}
  
  	/**
  	 * 删除需要被代理的接口
  	 */
  	public boolean removeInterface(Class<?> intf) {
  		return this.interfaces.remove(intf);
  	}
  	@Override
  	public Class<?>[] getProxiedInterfaces() {
  		return ClassUtils.toClassArray(this.interfaces);
  	}
  	@Override
  	public boolean isInterfaceProxied(Class<?> intf) {
  		for (Class<?> proxyIntf : this.interfaces) {
  			if (intf.isAssignableFrom(proxyIntf)) {
  				return true;
  			}
  		}
  		return false;
  	}
  
  	/**
  	 * 获取配置的所有切片列表
  	 */
  	@Override
  	public final Advisor[] getAdvisors() {
  		return this.advisorArray;
  	}
  	
      /**
       * 添加切片
       */
  	@Override
  	public void addAdvisor(Advisor advisor) {
  		int pos = this.advisors.size();
  		addAdvisor(pos, advisor);
  	}
  	@Override
  	public void addAdvisor(int pos, Advisor advisor) throws AopConfigException {
  		if (advisor instanceof IntroductionAdvisor) {
  			validateIntroductionAdvisor((IntroductionAdvisor) advisor);
  		}
  		addAdvisorInternal(pos, advisor);
  	}
  	@Override
  	public boolean removeAdvisor(Advisor advisor) {
  		int index = indexOf(advisor);
  		if (index == -1) {
  			return false;
  		}
  		else {
  			removeAdvisor(index);
  			return true;
  		}
  	}
  	@Override
  	public void removeAdvisor(int index) throws AopConfigException {
  		if (isFrozen()) {
  			throw new AopConfigException("Cannot remove Advisor: Configuration is frozen.");
  		}
  		if (index < 0 || index > this.advisors.size() - 1) {
  			throw new AopConfigException("Advisor index " + index + " is out of bounds: " +
  					"This configuration only has " + this.advisors.size() + " advisors.");
  		}
  
  		Advisor advisor = this.advisors.remove(index);
  		if (advisor instanceof IntroductionAdvisor) {
  			IntroductionAdvisor ia = (IntroductionAdvisor) advisor;
  			for (Class<?> ifc : ia.getInterfaces()) {
  				removeInterface(ifc);
  			}
  		}
  
  		updateAdvisorArray();
  		adviceChanged();
  	}
  	@Override
  	public int indexOf(Advisor advisor) {
  		Assert.notNull(advisor, "Advisor must not be null");
  		return this.advisors.indexOf(advisor);
  	}
  	@Override
  	public boolean replaceAdvisor(Advisor a, Advisor b) throws AopConfigException {
  		Assert.notNull(a, "Advisor a must not be null");
  		Assert.notNull(b, "Advisor b must not be null");
  		int index = indexOf(a);
  		if (index == -1) {
  			return false;
  		}
  		removeAdvisor(index);
  		addAdvisor(index, b);
  		return true;
  	}
  
  	public void addAdvisors(Advisor... advisors) {
  		addAdvisors(Arrays.asList(advisors));
  	}
  
  	public void addAdvisors(Collection<Advisor> advisors) {
  		if (isFrozen()) {
  			throw new AopConfigException("Cannot add advisor: Configuration is frozen.");
  		}
  		if (!CollectionUtils.isEmpty(advisors)) {
  			for (Advisor advisor : advisors) {
  				if (advisor instanceof IntroductionAdvisor) {
  					validateIntroductionAdvisor((IntroductionAdvisor) advisor);
  				}
  				Assert.notNull(advisor, "Advisor must not be null");
  				this.advisors.add(advisor);
  			}
  			updateAdvisorArray();
  			adviceChanged();
  		}
  	}
  
      // 省略
  	private void validateIntroductionAdvisor(IntroductionAdvisor advisor) {
  		advisor.validateInterfaces();
  		Class<?>[] ifcs = advisor.getInterfaces();
  		for (Class<?> ifc : ifcs) {
  			addInterface(ifc);
  		}
  	}
  
  	private void addAdvisorInternal(int pos, Advisor advisor) throws AopConfigException {
  		Assert.notNull(advisor, "Advisor must not be null");
  		if (isFrozen()) {
  			throw new AopConfigException("Cannot add advisor: Configuration is frozen.");
  		}
  		if (pos > this.advisors.size()) {
  			throw new IllegalArgumentException(
  					"Illegal position " + pos + " in advisor list with size " + this.advisors.size());
  		}
  		this.advisors.add(pos, advisor);
  		updateAdvisorArray();
  		adviceChanged();
  	}
  
  	/**
  	 * 保持advisorArray和advisors保持一致
  	 */
  	protected final void updateAdvisorArray() {
  		this.advisorArray = this.advisors.toArray(new Advisor[0]);
  	}
  
  	/**
  	 * 获取顾问列表
  	 */
  	protected final List<Advisor> getAdvisorsInternal() {
  		return this.advisors;
  	}
  
  	/**
  	 * 添加通知
  	 */
  	@Override
  	public void addAdvice(Advice advice) throws AopConfigException {
  		int pos = this.advisors.size();
  		addAdvice(pos, advice);
  	}
  
  	@Override
  	public void addAdvice(int pos, Advice advice) throws AopConfigException {
  		Assert.notNull(advice, "Advice must not be null");
  		if (advice instanceof IntroductionInfo) {
  
  			addAdvisor(pos, new DefaultIntroductionAdvisor(advice, (IntroductionInfo) advice));
  		}
  		else if (advice instanceof DynamicIntroductionAdvice) {
  			throw new AopConfigException("DynamicIntroductionAdvice may only be added as part of IntroductionAdvisor");
  		}
  		else {
  			addAdvisor(pos, new DefaultPointcutAdvisor(advice));
  		}
  	}
  
  	@Override
  	public boolean removeAdvice(Advice advice) throws AopConfigException {
  		int index = indexOf(advice);
  		if (index == -1) {
  			return false;
  		}
  		else {
  			removeAdvisor(index);
  			return true;
  		}
  	}
  
  	@Override
  	public int indexOf(Advice advice) {
  		Assert.notNull(advice, "Advice must not be null");
  		for (int i = 0; i < this.advisors.size(); i++) {
  			Advisor advisor = this.advisors.get(i);
  			if (advisor.getAdvice() == advice) {
  				return i;
  			}
  		}
  		return -1;
  	}
  
  	/**
  	 * 判断是否包含通知
  	 */
  	public boolean adviceIncluded(@Nullable Advice advice) {
  		if (advice != null) {
  			for (Advisor advisor : this.advisors) {
  				if (advisor.getAdvice() == advice) {
  					return true;
  				}
  			}
  		}
  		return false;
  	}
  
  	/**
  	 * 获取当前配置中某种类型通知的数量
  	 */
  	public int countAdvicesOfType(@Nullable Class<?> adviceClass) {
  		int count = 0;
  		if (adviceClass != null) {
  			for (Advisor advisor : this.advisors) {
  				if (adviceClass.isInstance(advisor.getAdvice())) {
  					count++;
  				}
  			}
  		}
  		return count;
  	}
  
  
  	/**
  	 * 基于当前配置，获取给定方法的方法调用链列表
  	 */
  	public List<Object> getInterceptorsAndDynamicInterceptionAdvice(Method method, @Nullable Class<?> targetClass) {
  		MethodCacheKey cacheKey = new MethodCacheKey(method);
  		List<Object> cached = this.methodCache.get(cacheKey);
  		if (cached == null) {
  			cached = this.advisorChainFactory.getInterceptorsAndDynamicInterceptionAdvice(
  					this, method, targetClass);
  			this.methodCache.put(cacheKey, cached);
  		}
  		return cached;
  	}
  
  	/**
  	 * 通知更改时，清空当前方法调用链缓存
  	 */
  	protected void adviceChanged() {
  		this.methodCache.clear();
  	}
  
  	/**
  	 * 复制配置信息到当前对象
  	 */
  	protected void copyConfigurationFrom(AdvisedSupport other) {
  		copyConfigurationFrom(other, other.targetSource, new ArrayList<>(other.advisors));
  	}
  
  	protected void copyConfigurationFrom(AdvisedSupport other, TargetSource targetSource, List<Advisor> advisors) {
  		copyFrom(other);
  		this.targetSource = targetSource;
  		this.advisorChainFactory = other.advisorChainFactory;
  		this.interfaces = new ArrayList<>(other.interfaces);
  		for (Advisor advisor : advisors) {
  			if (advisor instanceof IntroductionAdvisor) {
  				validateIntroductionAdvisor((IntroductionAdvisor) advisor);
  			}
  			Assert.notNull(advisor, "Advisor must not be null");
  			this.advisors.add(advisor);
  		}
  		updateAdvisorArray();
  		adviceChanged();
  	}
  
  	AdvisedSupport getConfigurationOnlyCopy() {
  		AdvisedSupport copy = new AdvisedSupport();
  		copy.copyFrom(this);
  		copy.targetSource = EmptyTargetSource.forClass(getTargetClass(), getTargetSource().isStatic());
  		copy.advisorChainFactory = this.advisorChainFactory;
  		copy.interfaces = this.interfaces;
  		copy.advisors = this.advisors;
  		copy.updateAdvisorArray();
  		return copy;
  	}
  
  	...
  }
  ```

  AdvisedSupport继承于ProxyConfig

  ```java
  org.springframework.aop.framework.ProxyConfig
  
  public class ProxyConfig implements Serializable {
  
  	private static final long serialVersionUID = -8409359707199703185L;
  
      // 是否直接对目标类进行代理，而不是通过接口生成代理（JDK/CGLIB）
  	private boolean proxyTargetClass = false;
  	// 是否对代理进行优化，设置为true，表示代理已经创建之后，通知更改将不会生效
      // 如果exposeProxy为true，那么optimize设置为true也会被忽略
  	private boolean optimize = false;
  	// 设置是否应阻止将配置创建的代理转换为advice
  	boolean opaque = false;
  	// 代理对象是否应该被AOP框架通过AopContext以ThreadLocal的形式暴露出去
  	boolean exposeProxy = false;
  	// 表示配置是否需要被冻结，冻结后无法修改增强的配置
  	private boolean frozen = false;
  
  	public void setProxyTargetClass(boolean proxyTargetClass) {
  		this.proxyTargetClass = proxyTargetClass;
  	}
  	public boolean isProxyTargetClass() {
  		return this.proxyTargetClass;
  	}
  
  	public void setOptimize(boolean optimize) {
  		this.optimize = optimize;
  	}
  	public boolean isOptimize() {
  		return this.optimize;
  	}
  
  	public void setOpaque(boolean opaque) {
  		this.opaque = opaque;
  	}
  	public boolean isOpaque() {
  		return this.opaque;
  	}
      
  	public void setExposeProxy(boolean exposeProxy) {
  		this.exposeProxy = exposeProxy;
  	}
  	public boolean isExposeProxy() {
  		return this.exposeProxy;
  	}
  
  	public void setFrozen(boolean frozen) {
  		this.frozen = frozen;
  	}
  	public boolean isFrozen() {
  		return this.frozen;
  	}
  
  	...
  
  }
  ```

  AdvicedSupport实现了Advised接口

  ```java
  org.springframework.aop.framework.Advised
      
  public interface Advised extends TargetClassAware {
  
  	/**
  	 * 返回配置是否已冻结，被冻结之后，无法修改Advisor
  	 */
  	boolean isFrozen();
  
  	/**
  	 * 判断是否为CGLIB创建代理
  	 */
  	boolean isProxyTargetClass();
  
  	/**
  	 * 获取配置类中需要代理的接口列表
  	 */
  	Class<?>[] getProxiedInterfaces();
  
  	/**
  	 * 判断接口是否被代理
  	 */
  	boolean isInterfaceProxied(Class<?> intf);
  
  	/**
  	 * 设置被代理的目标源，
  	 */
  	void setTargetSource(TargetSource targetSource);
      
  	TargetSource getTargetSource();
  
  	/**
  	 * 设置是否需要将代理暴露在ThreadLocal中
  	 */
  	void setExposeProxy(boolean exposeProxy);
  
  	boolean isExposeProxy();
  
  	/**
  	 * 设置是否需要预过滤
  	 */
  	void setPreFiltered(boolean preFiltered);
  
  	boolean isPreFiltered();
  
  	/**
  	 * 返回所有的切片列表
  	 */
  	Advisor[] getAdvisors();
  
  	void addAdvisor(Advisor advisor) throws AopConfigException;
  
  	/**
  	 * 添加切片
  	 */
  	void addAdvisor(int pos, Advisor advisor) throws AopConfigException;
  
  	boolean removeAdvisor(Advisor advisor);
  
  	void removeAdvisor(int index) throws AopConfigException;
  
  	int indexOf(Advisor advisor);
  
  	boolean replaceAdvisor(Advisor a, Advisor b) throws AopConfigException;
  
  	/**
  	 * 添加通知
  	 */
  	void addAdvice(Advice advice) throws AopConfigException;
  
  	void addAdvice(int pos, Advice advice) throws AopConfigException;
  
  	boolean removeAdvice(Advice advice);
  
  	int indexOf(Advice advice);
  
  	/**
  	 * toString
  	 */
  	String toProxyConfigString();
  
  }
  ```

  Adviced接口继承于TargetClassAware

  ```java
  org.springframework.aop.TargetClassAware
      
      
  public interface TargetClassAware {
  
  	/**
  	 * 返回目标对象类
  	 */
  	@Nullable
  	Class<?> getTargetClass();
  
  }
  ```

  **结论**

  1. 配置中添加的Advice**都会转换成DefaultPointcutAdvisor**，此时DefalutPointcutAdvisor**未指定pointcut**，**默认会匹配任意类的任意方法**
  2. 配置被冻结时，**Advisor列表不允许修改**
  3. `getInterceptorsAndDynamicInterceptionAdvice`，通过代理调用目标方法时，需要通过方法和目标类的类型，从配置中匹配方法拦截器列表，**获取方法拦截器列表是由AdvisorChainFactory负责**
  4. 目标方法和其关联的方法拦截器列表会缓存在methodCache中，**当切片列表发生变化时，methodCache缓存会被清除**

* **根据配置获取AopProxy**

  `aopProxyFactory.createAopProxy(advisedSupport)`

  此阶段根据AdvisedSupport中配置信息，**判断具体采用CGLIB还是JDK获取代理对象**

  ```java
  org.springframework.aop.framework.AopProxyFactory
  
  public interface AopProxyFactory {
  
  	/**
  	 * 创建Aop代理
  	 */
  	AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException;
  
  }
  ```

  DefaultAopProxyFactory时AopProxyFactory的实现类

  ```java
  org.springframework.aop.framework.DefaultAopProxyFactory
  
  /**
   * 默认AopProxyFactory实现，创建CGLIB和JDK
   * 当optimize = true || proxyTargetClass = true || 未指定代理接口则创建CGLIB代理
   */
  public class DefaultAopProxyFactory implements AopProxyFactory, Serializable {
  
  	@Override
  	public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
  		if (config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config)) {
  			Class<?> targetClass = config.getTargetClass();
  			if (targetClass == null) {
  				throw new AopConfigException("TargetSource cannot determine target class: " +
  						"Either an interface or a target is required for proxy creation.");
  			}
  			if (targetClass.isInterface() || Proxy.isProxyClass(targetClass)) {
  				return new JdkDynamicAopProxy(config);
  			}
  			return new ObjenesisCglibAopProxy(config);
  		}
  		else {
  			return new JdkDynamicAopProxy(config);
  		}
  	}
  
  	/**
  	 * 确定提供的AdvisedSupport是否只指定了Spring Proxy接口
  	 */
  	private boolean hasNoUserSuppliedProxyInterfaces(AdvisedSupport config) {
  		Class<?>[] ifcs = config.getProxiedInterfaces();
  		return (ifcs.length == 0 || (ifcs.length == 1 && SpringProxy.class.isAssignableFrom(ifcs[0])));
  	}
  
  }
  ```

  AopProxy接口定义了创建最终代理对象的方法

  ```java
  org.springframework.aop.framework.AopProxy
   
  public interface AopProxy {
   
      /**
       * 创建一个新的代理对象
       */
      Object getProxy();
   
      /**
       * 创建一个新的代理对象
       */
      Object getProxy(@Nullable ClassLoader classLoader);
   
  }
  ```

* **通过AopProxy获取代理对象**

  通过调用AopProxy的**createAopProxy方法**返回结果

  * **JdkDynamicAopProxy**

    ```java
    org.springframework.aop.framework.JdkDynamicAopProxy
    
    final class JdkDynamicAopProxy implements AopProxy, InvocationHandler, Serializable {
    
            private static final long serialVersionUID = 5531744639992436476L;
    
            private static final Log logger = LogFactory.getLog(JdkDynamicAopProxy.class);
    
            /** 
         * 代理配置信息 
     	 */
            private final AdvisedSupport advised;
    
            /**
    	 * 代理对象是否定义了equals方法
    	 */
            private boolean equalsDefined;
    
            /**
    	 * 代理对象是否定义了hashCode方法
    	 */
            private boolean hashCodeDefined;
    
            public JdkDynamicAopProxy(AdvisedSupport config) throws AopConfigException {
                Assert.notNull(config, "AdvisedSupport must not be null");
                if (config.getAdvisors().length == 0 && config.getTargetSource() == AdvisedSupport.EMPTY_TARGET_SOURCE) {
                    throw new AopConfigException("No advisors and no TargetSource specified");
                }
                this.advised = config;
            }
    
            /**
    	 * 获取一个代理对象
    	 */
            @Override
            public Object getProxy() {
                return getProxy(ClassUtils.getDefaultClassLoader());
            }
            @Override
            public Object getProxy(@Nullable ClassLoader classLoader) {
                if (logger.isTraceEnabled()) {
                    logger.trace("Creating JDK dynamic proxy: " + this.advised.getTargetSource());
                }
                // 根据advised的信息获取需要被代理的所有接口
                Class<?>[] proxiedInterfaces = AopProxyUtils.completeProxiedInterfaces(this.advised, true);
                // 处理equals和hashCode方法
                findDefinedEqualsAndHashCodeMethods(proxiedInterfaces);
                // JDK代理模式
                return Proxy.newProxyInstance(classLoader, proxiedInterfaces, this);
            }
    
            /**
    	 * 判断需要判断的接口中是否定义了equals和hashCode
    	 */
            private void findDefinedEqualsAndHashCodeMethods(Class<?>[] proxiedInterfaces) {
                for (Class<?> proxiedInterface : proxiedInterfaces) {
                    Method[] methods = proxiedInterface.getDeclaredMethods();
                    for (Method method : methods) {
                        if (AopUtils.isEqualsMethod(method)) {
                            this.equalsDefined = true;
                        }
                        if (AopUtils.isHashCodeMethod(method)) {
                            this.hashCodeDefined = true;
                        }
                        if (this.equalsDefined && this.hashCodeDefined) {
                            return;
                        }
                    }
                }
            }
    
    
            /**
    	 * InvokeHandler主要方法，代理对象任何方法都会被这个方法拦截
    	 */
            @Override
            @Nullable
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                Object oldProxy = null;
                // 判断是否需要将代理对象存放到ThreadLcoal中
                boolean setProxyContext = false;
    
                // 目标对象封装TargetSource
                TargetSource targetSource = this.advised.targetSource;
                // 目标对象
                Object target = null;
    
                try {
                    // 排除equals和hashCode方法
                    if (!this.equalsDefined && AopUtils.isEqualsMethod(method)) {
                        return equals(args[0]);
                    }
                    else if (!this.hashCodeDefined && AopUtils.isHashCodeMethod(method)) {
                        return hashCode();
                    }
                    // DecoratingProxy方法中，可以获取原始的被处理的目标对象
                    // 主要用在嵌套代理的情况下
                    else if (method.getDeclaringClass() == DecoratingProxy.class) {
                        // 内部通过循环遍历，找到最原始的被代理的目标类
                        return AopProxyUtils.ultimateTargetClass(this.advised);
                    }
                    // 代理对象默认会实现Adviced接口，可以同故宫代理对象动态添加通知
                    else if (!this.advised.opaque && method.getDeclaringClass().isInterface() &&
                             method.getDeclaringClass().isAssignableFrom(Advised.class)) {
                        // 通过反射将方法交给this.advised处理
                        return AopUtils.invokeJoinpointUsingReflection(this.advised, method, args);
                    }
    
                    Object retVal;
    
                    // 判断是否需要在ThreadLocal中暴露代理对象
                    if (this.advised.exposeProxy) {
                        // 设置当前代理对象到threadLocal中
                        oldProxy = AopContext.setCurrentProxy(proxy);
                        setProxyContext = true;
                    }
    
                    // 通过TargetSource获取目标对象
                    target = targetSource.getTarget();
                    Class<?> targetClass = (target != null ? target.getClass() : null);
    
                    // 根据方法类型和对象类型返回调用链
                    List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
    
                    // 检测调用链是否为空
                    if (chain.isEmpty()) {
                        // 获取方法请求的参数，这个方法主要将可变参数适配成正常数组
                        Object[] argsToUse = AopProxyUtils.adaptArgumentsIfNecessary(method, args);
                        // 通过反射直接调用目标方法
                        retVal = AopUtils.invokeJoinpointUsingReflection(target, method, argsToUse);
                    }
                    else {
                        // 创建方法调用器（包含了代理对象、目标对象、调用的方法、参数、目标类型、方法拦截器链）
                        MethodInvocation invocation =
                            new ReflectiveMethodInvocation(proxy, target, method, args, targetClass, chain);
                        // 调用器执行目标方法调用
                        retVal = invocation.proceed();
                    }
    
                    // 获取方法返回类型
                    Class<?> returnType = method.getReturnType();
                    if (retVal != null && retVal == target &&
                        returnType != Object.class && returnType.isInstance(proxy) &&
                        !RawTargetAccess.class.isAssignableFrom(method.getDeclaringClass())) {
                        // 返回代理对象
                        retVal = proxy;
                    }
                    // 返回值未null或原始类型报错
                    else if (retVal == null && returnType != Void.TYPE && returnType.isPrimitive()) {
                        throw new AopInvocationException(
                            "Null return value from advice does not match primitive return type for: " + method);
                    }
                    return retVal;
                }
                finally {
                    if (target != null && !targetSource.isStatic()) {
                        // 必须释放TargetSource中的目标对象
                        targetSource.releaseTarget(target);
                    }
                    if (setProxyContext) {
                        // 将旧的代理放入ThreadLocal
                        AopContext.setCurrentProxy(oldProxy);
                    }
                }
            }
    
            ...
                
        }
    
    }
    ```
    
    `AopProxyUtils.completeProxiedInterfaces方法`从配置中获取代理类中所有需要实现的接口
    
    ```java
    org.springframework.aop.framework.AopProxyUtils#completeProxiedInterfaces
    
    static Class<?>[] completeProxiedInterfaces(AdvisedSupport advised, boolean decoratingProxy) {
        // 获取需要实现的所有接口
        Class<?>[] specifiedInterfaces = advised.getProxiedInterfaces();
        if (specifiedInterfaces.length == 0) {
            // 没有实现接口，获取目标对象类型
            Class<?> targetClass = advised.getTargetClass();
            if (targetClass != null) {
                // 目标对象类型为接口
                if (targetClass.isInterface()) {
                    advised.setInterfaces(targetClass);
                }
                // 目标对象类型是代理类型，获取代理对象中的所有需要实现的接口
                else if (Proxy.isProxyClass(targetClass)) {
                    advised.setInterfaces(targetClass.getInterfaces());
                }
                specifiedInterfaces = advised.getProxiedInterfaces();
            }
        }
        // 判断SpringProxy接口是否已经在接口集合中
        boolean addSpringProxy = !advised.isInterfaceProxied(SpringProxy.class);
        // 判断Advised接口是否已经在接口集合中
        boolean addAdvised = !advised.isOpaque() && !advised.isInterfaceProxied(Advised.class);
        // 判断DecoratingProxy接口是否已经在接口集合中
        boolean addDecoratingProxy = (decoratingProxy && !advised.isInterfaceProxied(DecoratingProxy.class));
        int nonUserIfcCount = 0;
        if (addSpringProxy) {
            nonUserIfcCount++;
        }
        if (addAdvised) {
            nonUserIfcCount++;
        }
        if (addDecoratingProxy) {
            nonUserIfcCount++;
        }
        // 添加三个接口
        Class<?>[] proxiedInterfaces = new Class<?>[specifiedInterfaces.length + nonUserIfcCount];
        System.arraycopy(specifiedInterfaces, 0, proxiedInterfaces, 0, specifiedInterfaces.length);
        int index = specifiedInterfaces.length;
        if (addSpringProxy) {
            proxiedInterfaces[index] = SpringProxy.class;
            index++;
        }
        if (addAdvised) {
            proxiedInterfaces[index] = Advised.class;
            index++;
        }
        if (addDecoratingProxy) {
            proxiedInterfaces[index] = DecoratingProxy.class;
        }
        return proxiedInterfaces;
    }
    ```
    
    **JdkDynamicAopProxy总结**
    
    1. 被创建的代理对象默认实现SpringProxy，Adviced，DecoratingProxy三个接口
    2. SpringProxy只是一个标记接口，**用来标记代理对象是SpringAop创建的**
    3. 通过Advised接口，**可以动态给代理对象中添加通知**
    4. DecoratingProxy接口中定义getDecoratedClass方法，**用来获取被代理的原始目标对象类型**
    
  * **CglibAopProxy**
  
    ```java
    org.springframework.aop.framework.CglibAopProxy#getProxy
    
    @Override
    public Object getProxy(@Nullable ClassLoader classLoader) {
    
        try {
            // 获取目标对象类型
            Class<?> rootClass = this.advised.getTargetClass();
    		// cglib是采用继承的方式是创建代理对象的，所以将被代理的类作为代理对象的父类
            Class<?> proxySuperClass = rootClass;
            // 判断目标对象是不是代理对象
            if (rootClass.getName().contains(ClassUtils.CGLIB_CLASS_SEPARATOR)) {
                // 将原始父类对象设置为此对象的父类
                proxySuperClass = rootClass.getSuperclass();
                // 获取方法上需要实现的接口
                Class<?>[] additionalInterfaces = rootClass.getInterfaces();
                for (Class<?> additionalInterface : additionalInterfaces) {
                    this.advised.addInterface(additionalInterface);
                }
            }
    
    		...
    
            // 配置CGLIB Enhancer
            Enhancer enhancer = createEnhancer();
            // 设置classLoader干嘛？生成代理对象？
            if (classLoader != null) {
                enhancer.setClassLoader(classLoader);
                if (classLoader instanceof SmartClassLoader &&
                    ((SmartClassLoader) classLoader).isClassReloadable(proxySuperClass)) {
                    enhancer.setUseCache(false);
                }
            }
            // 设置父类
            enhancer.setSuperclass(proxySuperClass);
           	// 设置需要实现的接口	     	
            enhancer.setInterfaces(AopProxyUtils.completeProxiedInterfaces(this.advised));
            // 设置命名策略
            enhancer.setNamingPolicy(SpringNamingPolicy.INSTANCE);
            // 设置字节码生成策略
            enhancer.setStrategy(new ClassLoaderAwareGeneratorStrategy(classLoader));
    
            // 获取callbacks
            Callback[] callbacks = getCallbacks(rootClass);
            Class<?>[] types = new Class<?>[callbacks.length];
            for (int x = 0; x < types.length; x++) {
                types[x] = callbacks[x].getClass();
            }
            
            // 设置callBackFilter
            enhancer.setCallbackFilter(new ProxyCallbackFilter(
                this.advised.getConfigurationOnlyCopy(), this.fixedInterceptorMap, this.fixedInterceptorOffset));
            // 设置callback类型
            enhancer.setCallbackTypes(types);
    
            // 获取代理对象
            return createProxyClassAndInstance(enhancer, callbacks);
        }
        catch (CodeGenerationException | IllegalArgumentException ex) {
            throw new AopConfigException("Could not generate CGLIB subclass of " + this.advised.getTargetClass() +
                                         ": Common causes of this problem include using a final class or a non-visible class",
                                         ex);
        }
        catch (Throwable ex) {
            throw new AopConfigException("Unexpected AOP exception", ex);
        }
    }
    ```
  
    通过getCallbacks获取所有Callback列表
  
    ```java
    org.springframework.aop.framework.CglibAopProxy#getCallbacks
    
    private Callback[] getCallbacks(Class<?> rootClass) throws Exception {
    		// 是否需要将代理暴露到ThreadLocal中
    		boolean exposeProxy = this.advised.isExposeProxy();
        	// 是否冻结
    		boolean isFrozen = this.advised.isFrozen();
        	// 被代理的目标对象是否是动态的（是否是单例的）
    		boolean isStatic = this.advised.getTargetSource().isStatic();
    
    		// 使用DynamicAdvisedInterceptor
    		Callback aopInterceptor = new DynamicAdvisedInterceptor(this.advised);
    
    		// 当方法上没有需要执行的拦截器时，会使用targetInterceptor来处理，内部通过反射调用目标对象中的方法
    		Callback targetInterceptor;
        	// 是否暴露，是否单例进行特别处理，不同方法有不同的回调方式
    		if (exposeProxy) {
    			targetInterceptor = (isStatic ?
    					new StaticUnadvisedExposedInterceptor(this.advised.getTargetSource().getTarget()) :
    					new DynamicUnadvisedExposedInterceptor(this.advised.getTargetSource()));
    		}
    		else {
    			targetInterceptor = (isStatic ?
    					new StaticUnadvisedInterceptor(this.advised.getTargetSource().getTarget()) :
    					new DynamicUnadvisedInterceptor(this.advised.getTargetSource()));
    		}
    		Callback targetDispatcher = (isStatic ?
    				new StaticDispatcher(this.advised.getTargetSource().getTarget()) : new SerializableNoOp());
    
        	
    		Callback[] mainCallbacks = new Callback[] {
    				aopInterceptor,  // 处理普通的通知
    				targetInterceptor,  // 如果经过优化，调用目标时不考虑通知
    				new SerializableNoOp(),  // 实现代理对象序列化？
    				targetDispatcher, this.advisedDispatcher, //？
    				new EqualsInterceptor(this.advised),
    				new HashCodeInterceptor(this.advised)
    		};
    
    		Callback[] callbacks;
    
    		// 如果被代理对象时单例&&配置冻结的，此时优化
        	// 配置冻结，生成哈的代理中通知是无法修改的，所以可以提前将方法对应的缓存器链找到缓存
        	// 调用方法时，直接从缓存中拿到方法对应的缓存
    		if (isStatic && isFrozen) {
    			Method[] methods = rootClass.getMethods();
    			Callback[] fixedCallbacks = new Callback[methods.length];
    			this.fixedInterceptorMap = new HashMap<>(methods.length);
    
    			// 获取每个方法的调用链，缓存到fixedInterceptorMap
    			for (int x = 0; x < methods.length; x++) {
    				Method method = methods[x];
    				List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, rootClass);
    				fixedCallbacks[x] = new FixedChainStaticTargetInterceptor(
    						chain, this.advised.getTargetSource().getTarget(), this.advised.getTargetClass());
    				this.fixedInterceptorMap.put(method, x);
    			}
    
    			callbacks = new Callback[mainCallbacks.length + fixedCallbacks.length];
    			System.arraycopy(mainCallbacks, 0, callbacks, 0, mainCallbacks.length);
    			System.arraycopy(fixedCallbacks, 0, callbacks, mainCallbacks.length, fixedCallbacks.length);
    			this.fixedInterceptorOffset = mainCallbacks.length;
    		}
    		else {
    			callbacks = mainCallbacks;
    		}
    		return callbacks;
    	}
    ```
  
    通过`getInterceptorsAndDynamicInterceptionAdvice`方法可以获取调用链，如果缓存中没有调用链，**那么通过AdvisorChainFactory获取调用链**
  
    ```java
    org.springframework.aop.framework.AdvisorChainFactory
    
    public interface AdvisorChainFactory {
    
    	/**
    	 * 获取方法匹配的拦截器列表
    	 */
    	List<Object> getInterceptorsAndDynamicInterceptionAdvice(Advised config, Method method, @Nullable Class<?> targetClass);
    
    }
    ```
  
    AdvisorChainFactory接口的默认实现类
  
    ```java
    org.springframework.aop.framework.DefaultAdvisorChainFactory
    
    public class DefaultAdvisorChainFactory implements AdvisorChainFactory, Serializable {
    
    	@Override
    	public List<Object> getInterceptorsAndDynamicInterceptionAdvice(
    			Advised config, Method method, @Nullable Class<?> targetClass) {
    
    		// 获取Advisor适配器注册器
    		AdvisorAdapterRegistry registry = GlobalAdvisorAdapterRegistry.getInstance();
            // 获取Advisor列表
    		Advisor[] advisors = config.getAdvisors();
    		List<Object> interceptorList = new ArrayList<>(advisors.length);
            // 被调用方法所在类实际的类型
    		Class<?> actualClass = (targetClass != null ? targetClass : method.getDeclaringClass());
    		Boolean hasIntroductions = null;
    
            // 遍历Advisor列表，找到和actualClass和方法匹配的所有方法拦截器链列表
    		for (Advisor advisor : advisors) {
                // 判断是否时PointcutAdvisor，先看类是否匹配，然后再看方法是否匹配
    			if (advisor instanceof PointcutAdvisor) {
    				
    				PointcutAdvisor pointcutAdvisor = (PointcutAdvisor) advisor;
                    // 前置过滤||类匹配陈公公，进行方法匹配
    				if (config.isPreFiltered() || pointcutAdvisor.getPointcut().getClassFilter().matches(actualClass)) {
    					MethodMatcher mm = pointcutAdvisor.getPointcut().getMethodMatcher();
    					boolean match;
    					if (mm instanceof IntroductionAwareMethodMatcher) {
    						if (hasIntroductions == null) {
    							hasIntroductions = hasMatchingIntroductions(advisors, actualClass);
    						}
    						match = ((IntroductionAwareMethodMatcher) mm).matches(method, actualClass, hasIntroductions);
    					}
    					else {
    						match = mm.matches(method, actualClass);
    					}
    					if (match) {
                            // 将advisor转换成MethodInterceptor列表
    						MethodInterceptor[] interceptors = registry.getInterceptors(advisor);
                            
    						if (mm.isRuntime()) {
    							// 动态匹配
    							for (MethodInterceptor interceptor : interceptors) {
    								interceptorList.add(new InterceptorAndDynamicMethodMatcher(interceptor, mm));
    							}
    						}
    						else {
    							interceptorList.addAll(Arrays.asList(interceptors));
    						}
    					}
    				}
    			}
    			else if (advisor instanceof IntroductionAdvisor) {
    				IntroductionAdvisor ia = (IntroductionAdvisor) advisor;
    				if (config.isPreFiltered() || ia.getClassFilter().matches(actualClass)) {
    					Interceptor[] interceptors = registry.getInterceptors(advisor);
    					interceptorList.addAll(Arrays.asList(interceptors));
    				}
    			}
    			else {
    				Interceptor[] interceptors = registry.getInterceptors(advisor);
    				interceptorList.addAll(Arrays.asList(interceptors));
    			}
    		}
    
    		return interceptorList;
    	}
    
    	/**
    	 * ？ 
    	 */
    	private static boolean hasMatchingIntroductions(Advisor[] advisors, Class<?> actualClass) {
    		for (Advisor advisor : advisors) {
    			if (advisor instanceof IntroductionAdvisor) {
    				IntroductionAdvisor ia = (IntroductionAdvisor) advisor;
    				if (ia.getClassFilter().matches(actualClass)) {
    					return true;
    				}
    			}
    		}
    		return false;
    	}
    
    }
    ```
  
    AdvisorAdapterRegistry，切面适配器注册器
  
    ```java
    org.springframework.aop.framework.AdvisorAdapterRegistry
    
    public interface AdvisorAdapterRegistry {
    
    	/**
    	 * 将一个Advice包装为Advisor
    	 */
    	Advisor wrap(Object advice) throws UnknownAdviceTypeException;
    
    	/**
    	 * 根据Advisor获取MethodInterceptor
    	 */
    	MethodInterceptor[] getInterceptors(Advisor advisor) throws UnknownAdviceTypeException;
    
    	/**
    	 * 注册AdvisorAdapter
    	 */
    	void registerAdvisorAdapter(AdvisorAdapter adapter);
    
    }
    ```
  
    AdvisorAdapterRegistry的默认实现类
  
    ```java
    org.springframework.aop.framework.adapter.DefaultAdvisorAdapterRegistry
    
    public class DefaultAdvisorAdapterRegistry implements AdvisorAdapterRegistry, Serializable {
    
        //AdvisorAdapter转换器列表，AdvisorAdapter负责将Advisor中的Advice转换为MethodInterceptor类型的
    	private final List<AdvisorAdapter> adapters = new ArrayList<>(3);
    
    
    	//默认会注册3个AdvisorAdapter，这3个负责将前置通知，异常通知，后置通知转换为MethodInterceptor类型的
    	public DefaultAdvisorAdapterRegistry() {
    		registerAdvisorAdapter(new MethodBeforeAdviceAdapter());
    		registerAdvisorAdapter(new AfterReturningAdviceAdapter());
    		registerAdvisorAdapter(new ThrowsAdviceAdapter());
    	}
    
    
    	@Override
    	public Advisor wrap(Object adviceObject) throws UnknownAdviceTypeException {
            // advisor直接返回
    		if (adviceObject instanceof Advisor) {
    			return (Advisor) adviceObject;
    		}
            // 没有实现Advice，直接报错
    		if (!(adviceObject instanceof Advice)) {
    			throw new UnknownAdviceTypeException(adviceObject);
    		}
    		Advice advice = (Advice) adviceObject;
            // 方法拦截器
    		if (advice instanceof MethodInterceptor) {
    			// 封装直接返回
    			return new DefaultPointcutAdvisor(advice);
    		}
    		for (AdvisorAdapter adapter : this.adapters) {
    			// 检查适配器是否能够处理
    			if (adapter.supportsAdvice(advice)) {
    				return new DefaultPointcutAdvisor(advice);
    			}
    		}
    		throw new UnknownAdviceTypeException(advice);
    	}
    
    	@Override
    	public MethodInterceptor[] getInterceptors(Advisor advisor) throws UnknownAdviceTypeException {
    		List<MethodInterceptor> interceptors = new ArrayList<>(3);
    		Advice advice = advisor.getAdvice();
            // MethodInterceptor直接添加
    		if (advice instanceof MethodInterceptor) {
    			interceptors.add((MethodInterceptor) advice);
    		}
            // 轮询适配器，检查是否能够处理
    		for (AdvisorAdapter adapter : this.adapters) {
    			if (adapter.supportsAdvice(advice)) {
    				interceptors.add(adapter.getInterceptor(advisor));
    			}
    		}
    		if (interceptors.isEmpty()) {
    			throw new UnknownAdviceTypeException(advisor.getAdvice());
    		}
    		return interceptors.toArray(new MethodInterceptor[0]);
    	}
    
    	@Override
    	public void registerAdvisorAdapter(AdvisorAdapter adapter) {
    		this.adapters.add(adapter);
    	}
    }
    ```
  
    AdvisorAdapter接口是适配器接口
  
    ```java
    org.springframework.aop.framework.adapter.AdvisorAdapter
    
    public interface AdvisorAdapter {
    
    	/**
    	 * 判断通知类型是否支持
    	 */
    	boolean supportsAdvice(Advice advice);
    
    	/**
    	 * 获取Advisor对应的MethodInterceptor
    	 */
    	MethodInterceptor getInterceptor(Advisor advisor);
    
    }
    ```

## ProxyFactoryBean创建Aop代理

**ProxyFactoryBean使用步骤**

```shell
1.创建ProxyFactoryBean对象
2.通过ProxyFactoryBean.setTargetName设置目标对象的bean名称
3.通过ProxyFactoryBean。setInterceptorNames添加需要增强的通知
4.将ProxyFactoryBean注册到Spring容器，名称为proxyBean
5.从Spring查找名称为proxyBean的bean，这个bean就是生成好的代理对象
```

需要匹配的bean名称后面跟上一个\*，**此时Spring会从容器中找到bean名称以interceptor开头的将作为增强器**

```java
org.springframework.aop.framework.ProxyFactoryBean#getObject

@Override
@Nullable
public Object getObject() throws BeansException {
    // 初始化调用链
    initializeAdvisorChain();
    if (isSingleton()) {
        // 创建单例代理对象
        return getSingletonInstance();
    }
    else {
        if (this.targetName == null) {
            ...
        }
        // 创建多例代理对象
        return newPrototypeInstance();
    }
}
```

`initializeAdvisorChain`方法，用来初始化调用链，根据interceptorNames配置获取

```java
org.springframework.aop.framework.ProxyFactoryBean#initializeAdvisorChain

private synchronized void initializeAdvisorChain() throws AopConfigException, BeansException {
    // 已经初始化直接返回
    if (this.advisorChainInitialized) {
        return;
    }

    if (!ObjectUtils.isEmpty(this.interceptorNames)) {
        if (this.beanFactory == null) {
            throw new IllegalStateException("No BeanFactory available anymore (probably due to serialization) " +
                                            "- cannot resolve interceptor names " + Arrays.asList(this.interceptorNames));
        }

        if (this.interceptorNames[this.interceptorNames.length - 1].endsWith(GLOBAL_SUFFIX) &&
            this.targetName == null && this.targetSource == EMPTY_TARGET_SOURCE) {
            throw new AopConfigException("Target required after globals");
        }

        // 遍历拦截器名称集合，添加到调用链
        for (String name : this.interceptorNames) {
            // 批量注册的方式：判断name是否以*结尾
            if (name.endsWith(GLOBAL_SUFFIX)) {
                // 从容器中匹配查找匹配的增强器，将其添加到aop配置中
                if (!(this.beanFactory instanceof ListableBeanFactory)) {
                    throw new AopConfigException(
                        "Can only use global advisors or interceptors with a ListableBeanFactory");
                }
                addGlobalAdvisors((ListableBeanFactory) this.beanFactory,
                                  name.substring(0, name.length() - GLOBAL_SUFFIX.length()));
            }

            else {
                // 添加单例拦截器
                Object advice;
                if (this.singleton || this.beanFactory.isSingleton(name)) {

                    advice = this.beanFactory.getBean(name);
                }
                else {
                   	// 添加多例拦截器
                    advice = new PrototypePlaceholderAdvisor(name);
                }
                addAdvisorOnChainCreation(advice);
            }
        }
    }

    this.advisorChainInitialized = true;
}
```

批量添加全局拦截器

```java
org.springframework.aop.framework.ProxyFactoryBean#addGlobalAdvisors

private void addGlobalAdvisors(ListableBeanFactory beanFactory, String prefix) {
    // 获取所有类型为Advisor的bean
    String[] globalAdvisorNames =
        BeanFactoryUtils.beanNamesForTypeIncludingAncestors(beanFactory, Advisor.class);
    
    // 获取所有类型为Interceptor类型的bean
    String[] globalInterceptorNames =
        BeanFactoryUtils.beanNamesForTypeIncludingAncestors(beanFactory, Interceptor.class);
    if (globalAdvisorNames.length > 0 || globalInterceptorNames.length > 0) {
        List<Object> beans = new ArrayList<>(globalAdvisorNames.length + globalInterceptorNames.length);
        for (String name : globalAdvisorNames) {
            if (name.startsWith(prefix)) {
                beans.add(beanFactory.getBean(name));
            }
        }
        for (String name : globalInterceptorNames) {
            if (name.startsWith(prefix)) {
                beans.add(beanFactory.getBean(name));
            }
        }
        AnnotationAwareOrderComparator.sort(beans);
        for (Object bean : beans) {
            addAdvisorOnChainCreation(bean);
        }
    }
}
```

```java
org.springframework.aop.framework.ProxyFactoryBean#addAdvisorOnChainCreation

private void addAdvisorOnChainCreation(Object next) {
    // 添加切面
    addAdvisor(namedBeanToAdvisor(next));
}
```

使用namedBeanToAdvisor将bean转换为advisor

```java
org.springframework.aop.framework.ProxyFactoryBean#namedBeanToAdvisor

private Advisor namedBeanToAdvisor(Object next) {
    try {
        return this.advisorAdapterRegistry.wrap(next);
    }
    catch (UnknownAdviceTypeException ex) {
        ...
    }
}
```

## AspectJ创建Aop代理

**AspectJ创建步骤**

```shell
1.创建一个类，使用@Aspect标注
2.@Aspect标注的类中，通过@Pointcut定义切入点
3.@Aspect标注的类中，通过AspectJ提供的一些通知相关的注解定义通知
4.使用AspectJProxyFactory结合@Ascpect标注的类，来生成代理对象
```

### @Pointcut的12种用法

* **execution**

  用于匹配方法执行的连接点

  ```shell
  execution(modifiers-pattern? ret-type-pattern declaring-type-pattern? name-pattern(param-pattern) throws-pattern?)
  ```

  * 其中带?的modifiers-pattern?，declaring-type-pattern?，throws-pattern?是**可选项**
  * ret-type-pattern，name-pattern，parameters-pattern是**必选项**
  * modifier-pattern?修饰符匹配
  * ret-type-pattern返回匹配，*表示任何返回值
  * declaring-type-pattern?类路径匹配
  * name-pattern方法名匹配，\*代表所有
  * (param-pattern)参数匹配，指定方法参数（声明的类型），(..)代表所有参数，(*, xxx)代表第一个参数为任何值，(.., xxx)代表最后一个参数是xxx类型
  * throws-pattern?，异常类型匹配

* **within**

  **目标对象target的类型**是否和within种指定的类型匹配

* **this**

  **通过Aop创建的代理对象的类型**是否和this种指定的类型匹配，**必须是类型全限定名，不支持通配符**

  **匹配原则**

* **target**

  判断**目标对象的类型**是否和指定的类型匹配，**表达式必须式类型全限定名，不支持通配符**

* **args**

  匹配当前执行的方法**传入的参数**是否为args中指定的类型，注意是匹配传入的参数类型，**参数类型列表中的参数必须是类型全限定名，不支持通配符**，**这种切入点开销非常大**，

* **@within**

  **匹配指定的注解**是否和@within中指定值相同

* **@target**

  判断**目标对象target类型**上是否有指定的注解，**必须是全限定类型名**

* **@args**

  **方法参数**所属的类上有指定的注解

* **@annotation**

  匹配**被调用的方法**上有指定的注解

* **bean**

  这个用在spring环境中，匹配容器中**指定名称的bean**

* **reference pointcut**

  表示引用其他名称切入点

  ```java
  @Pointcut("完整包名类名.方法名称()")
  ```

* **组合型的pointcut**

  pointcut定义时，还可以使用&&、||、!运算符

### AspectJ五种通知

* **@Before：前置通知**

  ```java
  org.aspectj.lang.annotation.Before
  
  @Retention(RetentionPolicy.RUNTIME)
  @Target(ElementType.METHOD)
  public @interface Before {
  
      /**
       * pointcut表达式
       */
      String value();
  
      String argNames() default "";
  
  }
  ```

  Before修饰的方法参数添加JoinPoint类型，**但是必须放在第一位**，可以从JoinPoint获取signature

  ```java
  org.aspectj.lang.Signature
      
  public interface Signature {
      String toString();
  
      String toShortString();
  
      String toLongString();
  
  
      /**
       * 获取方法名
       */
      String getName();
  
      /**
       * 获取方法权限修饰符
       */
      int    getModifiers();
  
      /**
       * 返回声明此方法的类
       */
      Class  getDeclaringType();
  
      String getDeclaringTypeName();
  }
  ```

  通常情况下，Spring中AOP**都是对方法进行拦截**，所以连接点签名信息，可以转换为MethodSignature

  ```java
  org.aspectj.lang.reflect.MethodSignature
  
  public interface MethodSignature extends CodeSignature {
      /**
       * 获取返回值类型
       */
      Class getReturnType();  
      
      /**
       * 获取方法对象
       */
  	Method getMethod();
  }
  ```

* **@Around：环绕通知**

  环绕通知会包裹目标方法的执行

  ```java
  org.aspectj.lang.annotation.Around
  
  @Retention(RetentionPolicy.RUNTIME)
  @Target(ElementType.METHOD)
  public @interface Around {
  
      /**
       * 表达式
       */
      String value();
  
      String argNames() default "";
  
  }
  ```

* **@After：后置通知**

  **有异常也会触发**

  也可以通过将JoinPoint作为方法的第一个参数，获取连接点信息

  ```java
  org.aspectj.lang.annotation.After
  
  @Retention(RetentionPolicy.RUNTIME)
  @Target(ElementType.METHOD)
  public @interface After {
  
      String value();
  
      String argNames() default "";
  }
  ```

* **@AfterReturning：返回通知**

  **抛出异常，不会执行**

  @AfterReturning的第二个参数是retVal，表示返回值

  ```java
  org.aspectj.lang.annotation.AfterReturning
  
  @Retention(RetentionPolicy.RUNTIME)
  @Target(ElementType.METHOD)
  public @interface AfterReturning {
  
      String value() default "";
  
      /**
       * 会覆盖value
       */
      String pointcut() default "";
  
      /**
       * 设置返回值参数的名字
       */
      String returning() default "";
  
      String argNames() default "";
  
  }
  ```

* **@AfterThrowing：异常通知**

  @AfterThrowing方法可以指定异常参数，触发异常后回调，**不指定异常类型，匹配所有异常**

  ```java
  org.aspectj.lang.annotation.AfterThrowing
  
  @Retention(RetentionPolicy.RUNTIME)
  @Target(ElementType.METHOD)
  public @interface AfterThrowing {
  
      String value() default "";
  
      String pointcut() default "";
  
      /**
       * 指定异常参数的名称
       */
      String throwing() default "";
  
      String argNames() default "";
  
  }
  ```

**通知执行顺序**

```shell
@Around通知start
@Before通知
@Around绕通知end
@After通知
@AfterReturning通知
方法调用
```

### @EnableAspectJAutoProxy

```java
org.springframework.context.annotation.EnableAspectJAutoProxy

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(AspectJAutoProxyRegistrar.class)
public @interface EnableAspectJAutoProxy {

	/**
	 * 是否基于类创建代理，为true使用CGLIB，默认false，没有接口时使用CGLIB
	 */
	boolean proxyTargetClass() default false;

	/**
	 * 是否需要将代理对象暴露在ThreadLocal中
	 */
	boolean exposeProxy() default false;

}
```

# Spring Transaction

## Spring编程式事务

* **通过PlatformTransactionManager**

  最原始的方式，代理较大

  * **定义事务管理器**

    事务管理器就是用来管理事务

    Spring中使用PlatformTransactionManager表示事务管理器

    ```java
    org.springframework.transaction.PlatformTransactionManger
    
    public interface PlatformTransactionManager extends TransactionManager {
        
        // 获取一个事务（开启事务）
        TransactionStatus getTransaction(@Nullable TransactionDefinition var1) throws TransactionException;
    
        // 提交事务
        void commit(TransactionStatus var1) throws TransactionException;
    
        // 回滚事务
        void rollback(TransactionStatus var1) throws TransactionException;
    }
    ```

    PlatformTransactionManager有多个实现类，针对不同的环境使用

    * JpaTransactionManager：**用Jpa来操作DB**，需要这个管理器控制事务
    * DataSourceTransactionManager：指定数据源的方式，**JdbcTemplate、Mybatis、Ibatis**，使用这个管理器来控制事务
    * HibernateTransactionManager：**用Hibernate来操作DB**，需要用这个管理器控制事务
    * JtaTransactionManager：**使用JTA来操作DB**，这种通常是**分布式事务**，需要用这种管理器控制事务

  * **定义事务属性TransactionDefinition**

    Spring中使用TransactionDefinition接口来表示事务的定义信息

    ```java
    org.springframework.transaction.TransadctionDefinition
    
    public interface TransactionDefinition {
        // 传播方式
        int PROPAGATION_REQUIRED = 0;
        int PROPAGATION_SUPPORTS = 1;
        int PROPAGATION_MANDATORY = 2;
        int PROPAGATION_REQUIRES_NEW = 3;
        int PROPAGATION_NOT_SUPPORTED = 4;
        int PROPAGATION_NEVER = 5;
        int PROPAGATION_NESTED = 6;
        // 隔离级别
        int ISOLATION_DEFAULT = -1;
        int ISOLATION_READ_UNCOMMITTED = 1;
        int ISOLATION_READ_COMMITTED = 2;
        int ISOLATION_REPEATABLE_READ = 4;
        int ISOLATION_SERIALIZABLE = 8;
        int TIMEOUT_DEFAULT = -1;
    
        // 获取传播方式
        default int getPropagationBehavior() {
            return 0;
        }
    
        // 获取隔离级别
        default int getIsolationLevel() {
            return -1;
        }
    
        // 获取超时事件属性
        default int getTimeout() {
            return -1;
        }
    
        // 设置可读属性
        default boolean isReadOnly() {
            return false;
        }
    
        // 获取事务名称
        @Nullable
        default String getName() {
            return null;
        }
    
        // 获取默认事务定义
        static TransactionDefinition withDefaults() {
            return StaticTransactionDefinition.INSTANCE;
        }
    }
    ```

  * **开启事务**

    调用事务管理器的getTransaction开启一个事务

    这个方法会返回TransactionStatus表示事务状态，**通过TransactionStatus提供方法用来控制事务状态**

* **通过TransactionTemplate**

  Spring对方式1进行优化，**采用模板方法模式**进行封装，主要省区提交或回滚事务的代码

  TransactionTemplate提供方法执行业务

  * `executeWithoutResult(Consumer<TransactionStatus\> action)`没有返回值，传递Consumer对象**在accept方法中执行业务**
  * `<T\> T execute(TransactionCallback<T\> action)`有返回值，需要传递一个TransactionCallback对象，**在doInTransaction中执行业务**

  **导致回滚的情况**

  1. `transactionStatus.setRollbackOnly()`，将事务状态标注为回滚状态
  2. 方法内部抛出异常

## Spring声明式事务

* **启动Spring的事务管理驱动注解**

  当spring容器启动的时候，发现有@EnableTransactionMangement注解，此时会拦截所有bean的创建，扫描是否有@Transaction注释，如果有，**会通过AOP方式给bean生成代理对象**，代理对象中有拦截器，**拦截器会拦截bean中public方法**

  ```java
  org.springframework.transaction.annotation.EnableTransactionManagement
  
  @Target({ElementType.TYPE})
  @Retention(RetentionPolicy.RUNTIME)
  @Documented
  @Import({TransactionManagementConfigurationSelector.class})
  public @interface EnableTransactionManagement {
      /**
       * true，交给CGLIB处理
       */
      boolean proxyTargetClass() default false;
  
      /**
       * 使用jdk还是aspectJ
       */
      AdviceMode mode() default AdviceMode.PROXY;
  
      int order() default 2147483647;
  }
  ```

* **需要事务管理的目标上加@Transaction注解**

  * @Transaction放在接口上，接口的实现类中所有public都被事务管理
  * @Transaction放在类上，当前类下所有子类public方法被事务管理
  * @Transaction在public方法上，该方法被事务管理

  ```java
  org.springframework.transaction.annotation.Transactional
  
  @Target({ElementType.TYPE, ElementType.METHOD})
  @Retention(RetentionPolicy.RUNTIME)
  @Inherited
  @Documented
  public @interface Transactional {
      /**
       * 指定事务管理器bean
       */
      @AliasFor("transactionManager")
      String value() default "";
      @AliasFor("value")
      String transactionManager() default "";
  
      /**
       * 传播方式，默认REQUIRED
       */
      Propagation propagation() default Propagation.REQUIRED;
  
      /**
       * 隔离级别，默认看数据源
       */
      Isolation isolation() default Isolation.DEFAULT;
  
      /**
       * 超时时间
       */
      int timeout() default -1;
  
      /**
       * 是否只读事务
       */
      boolean readOnly() default false;
  
      /**
       * 指定回滚类型，必须式Trowable子类，抛出这些异常才会回滚
       */
      Class<? extends Throwable>[] rollbackFor() default {};
      String[] rollbackForClassName() default {};
  
      /**
       * 不回滚的异常
       */
      Class<? extends Throwable>[] noRollbackFor() default {};
      String[] noRollbackForClassName() default {};
  }
  ```

## Spring事务传播行为

```java
org.springframework.transaction.annotation.Propagation

public enum Propagation {
	// 如果事务管理器中有事务，那么加入这个事务中，如果没有那么就新建一个事务
	REQUIRED(TransactionDefinition.PROPAGATION_REQUIRED),
	// 如果事务管理器中有事务，那么加入这个事务中，如果没有那么以非事务执行
	SUPPORTS(TransactionDefinition.PROPAGATION_SUPPORTS),
	// 如果事务管理器中有事务，那么加入这个事务中，如果没有那么直接抛出异常
	MANDATORY(TransactionDefinition.PROPAGATION_MANDATORY),
	// 不管事务管理器中有没有事务，都会新建一个新的事务使用
	REQUIRES_NEW(TransactionDefinition.PROPAGATION_REQUIRES_NEW),
	// 不管事务管理器中有没有事务，都以非事务方式执行
	NOT_SUPPORTED(TransactionDefinition.PROPAGATION_NOT_SUPPORTED),
	// 事务管理器中没有事务，那么以非事务方式执行，有事务则抛出异常
	NEVER(TransactionDefinition.PROPAGATION_NEVER),
	// 如果事务管理器中有事务，那么以该事务的子事务执行，如果没有则新建一个事务
	NESTED(TransactionDefinition.PROPAGATION_NESTED);

	private final int value;

	Propagation(int value) {
		this.value = value;
	}

	public int value() {
		return this.value;
	}
}
```

* **REQUIRED**：外部开启事务，内部加入外部事务，外部未开启事务，内部新建事务
* **REQUIRED_NEW**：无论外部是否开启事务，内部都会新建事务
* **NESTED**：外部未开启事务，内部新建事务，外部开启事务，内部成为外部子事务（**外部回滚内部必须回滚**，**内部回滚外部不受影响**

## Spring事务失效常见情况

* **未启动Spring事务管理功能**

  没有使用@EnableTransactionManagement注解来启动Spring事务自动管理功能

* **方法不是public类型**

  @Transactional对非public方法无效

* **数据源未配置事务管理器**

  没有为每一个数据源配置事务管理器

* **自身调用问题**

  事务是通过AOP代理实现的，必须通过代理对象调用目标方法，事务才会生效

* **异常类型错误**

  捕获异常，但是并没有回滚或者处理异常，那么就会失效

* **异常被吞**

  方法内部将需要被捕获的异常处理了，Spring无法感知异常，事务失效

* **业务和事务必须在同一线程中**

  事务实现中使用ThreadLocal，如果不在一个线程，那么数据不能共享
