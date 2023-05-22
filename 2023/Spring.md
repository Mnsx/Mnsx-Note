# 控制反转（IoC）与依赖注入（DI）

## 常规方法存在的问题

* 将依赖的对象写在构造器中，在需要创建不同的依赖对象，由于写死在构造器中，所以加大了修改的繁琐程度
* 如果有很多地方都需要这个对象，那么采用这种new对象的写法，代码量很大，不方便维护，代码耦合度高

## Spring容器

程序启动的时候会创建spring容器，会给spring容器一个清单，清单中列出了需要创建的对象以及对象依赖关系，spring容器会创建和组装好清单中的对象，然后将这些对象存放在spring容器中，当程序中需要使用的时候，可以到容器中查找获取，然后直接使用

## IoC控制反转

IOC是是面相对象编程中的一种设计原则，主要是为了降低系统代码的耦合度，让系统利于维护和扩展

## 依赖注入DI

​	依赖注入是Spring容器中创建对象时给其设置依赖对象的一种方式

# Spring容器基本使用及原理

## IoC容器

IoC容器是具有依赖注入功能的容器，负责对象的实例化、对象的初始化，对象和对象之间依赖关系配置、对象的销毁、对外提供对象的查找等操作

## IoC容器如何知道管理那些对象？

需要给IoC容器提供一个配置清单，这个配置支持XML格式和Java注解的方式，在配置文件中列出需要让IoC容器管理的对象，以及可以指定IoC容器如何构建这些对象

## Bean概念

Spring容器管理的对象统称为Bean对象，Bean就是普通的java对象，只是这些对象是由Spring创建和管理。

容器中对象的配置统称为Bean定义配置元数据信息，Spring容器通过读取bean的配置元数据信息来构建和组装Bean对象

## Spring容器对象

* **BeanFactory接口**

  ```java
  org.springframework.beans.factory.BeanFactory
  ```

  Spring容器中具有代表性的容器就是BeanFactory接口，这个是spring容器的顶层接口，提供了容器最基本的功能

  ```java
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

# XML中bean定义详解

> 由于现在的开发中，已经不常用XML定义bean了，所以需要时参考下面连接学习
>
> [XML中bean定义详解](http://itsoku.com/course/5/86)

# 容器创建bean实例的方法

## 通过反射调用构造方法创建bean对象

调用类的构造方法获取对应的bean实例，是使用最多的方式，spring容器内部会自动调用该类型的构造方法来创建bean对象，将其放在容器中以供使用

```java
@Component
public class Component1 {
    public String success() {
        return "success";
    }
}
```

```java
@SpringBootApplication
public class SpringStudyApplication {
    public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException {
        ConfigurableApplicationContext context = SpringApplication.run(SpringStudyApplication.class);

        Component1 component1 = context.getBean("component1", Component1.class);
        System.out.println(component1.success());
    }
}
```

```shell
success
```

## 通过静态工厂方法创建bean对象

创建静态工厂，内部提供一些静态方法生成所需要的对象，将这些静态方法创建的对象交给Spring以供使用

## 通过实例工厂方法创建bean对象

让Spring容器去调用某些对象的某些实例方法来生成bean对象放在容器中使用

## 通过FactoryBean来创建Bean对象

**FactoryBean接口**

```java
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

这个接口中除了default方法不需要我们实现之外其他都需要我们实现，getObejct方法实现对象的创建，然后将创建好的对象返回给Spring容器，getObjectType需要指定我们创建的bean的类型，isSingleton表示通过这个接口创建的对象是否是单例

**实例**

```java
@Component
public class ComponentFactoryBean implements FactoryBean<Component3> {

    @Override
    public Component3 getObject() throws Exception {
        Component3 component3 = new Component3();
        return component3;
    }

    @Override
    public Class<?> getObjectType() {
        return Component3.class;
    }

    @Override
    public boolean isSingleton() {
        return true;
    }
}
```

```java
@SpringBootApplication
public class SpringStudyApplication {
    public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException {
        ConfigurableApplicationContext context = SpringApplication.run(SpringStudyApplication.class);

        Component3 component3 = context.getBean(Component3.class);
        System.out.println(component3);
    }
}
```

```shell
top.mnsx.spring.study.component.Component3@134c370e
```

## 通过@Bean注解创建Bean对象

使用@Bean修饰方法，方法的返回值，将会被放在容器中，作为Bean对象实例进行使用

# Bean作用域scope详解

## Singleton

当scope的值设置为singleton时，整个Spring容器中只会存在一个bean实例，通过容器多次查找bean的时候，返回的都是一个bean对象，Singleton是scope的默认值（**当bean的lazy属性被设置为true，表示懒加载**）

* Singleton实例在容器启动过程中就创建 好了，放在容器中缓存着

**单例Bean使用注意**

单例Bean是整个应用共享的，所有需要考虑到线程安全问题

## Prototype

如果scope被设置为prototype类型，表示这个bean是多例的，通过容器没获取的bean都是不同的实例，每次获取都会重新创建一个bean实例对象

**多例Bean使用注意**

多例Bean每次获取的时候都会重新创建，如果这个Bean比较复杂，创建时间比较长，会影响系统的性能

## Request

当一个bean的作用域为request，表示在一次http请求中，一个bean对应一个实例

**request作用域用于在Spring容器的web环境，Spring中有个Web容器接口WebApplicationContext，这里面对Request作用域提供了支持**

## Session

这个和request相似，也是用在Web环境，session级别共享bean，每个会话对应一个bean实例

## Application

全局Web应用级别，也是在Web环境中使用，一个Web应用程序对应一个Bean实例

**通常情况与Singleton效果类似，但是一个应用程序中可以创建多个spring容器，不同的容器中可以存在同名的bean，但是scope-application不管有多少容器，同名的bean只有一个**

## 自定义scope

1. 实现Scope接口

   **接口定义**

   ```java
   package org.springframework.beans.factory.config;
   
   import org.springframework.beans.factory.ObjectFactory;
   import org.springframework.lang.Nullable;
   
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

2. 将自定义的scope注册到容器

   需要调用org.springframework.beans.factory.config.ConfigurableBeanFactory#registerScope方法

   ```java
   /**
    * 向容器中注册自定义的Scope
    * scopeName：作用域名称
    * scope：作用域对象
   **/
   void registerScope(String scopeName, Scope scope);
   ```

3. 使用自定义的作用域

**案例**

* **需求**

  实现线程级别的bean作用域

* **ThreadScope**

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
          System.out.println(name);
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
  
* **Component4**

  ```java
  @Scope("thread")
  @Component
  public class Component4 {
      public void success() {
          System.out.println("success");
      }
  }
  ```

* **ScopConfig**

  ```java
  @Configuration
  public class ScopeConfig {
      @Bean
      public static CustomScopeConfigurer customScopeConfigurer() {
          CustomScopeConfigurer customScopeConfigurer = new CustomScopeConfigurer();
  
          Map<String, Object> map = new HashMap<>();
          map.put(ThreadScope.THREAD_SCOPE, new ThreadScope());
  
          customScopeConfigurer.setScopes(map);
          return customScopeConfigurer;
      }
  }
  ```

* **SpringStudyApplication**

  ```java
  @SpringBootApplication
  public class SpringStudyApplication {
      public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException {
          ConfigurableApplicationContext context = SpringApplication.run(SpringStudyApplication.class);
  
          Component4 component4_1 = context.getBean(Component4.class);
          System.out.println(component4_1);
  
          Thread thread = new Thread(() -> {
              Component4 component4_2 = context.getBean(Component4.class);
              System.out.println(component4_2);
          });
          thread.start();
      }
  }
  ```

# depnd-on作用

## 无依赖bean创建和销毁的顺序

> 使用DisposableBean接口，接口中destory方法，来实现bean的销毁行为

* BeanN

  ```java
  @Component
  public class BeanN implements DisposableBean {
  
      public Bean1() {
          System.out.println("BeanN create");
      }
  
      @Override
      public void destroy() throws Exception {
          System.out.println("beanN destroy");
      }
  }
  ```

* SpringStudyApplication

  ```java
  @SpringBootApplication
  public class SpringStudyApplication {
      public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException {
          ConfigurableApplicationContext context = SpringApplication.run(SpringStudyApplication.class);
  
          context.close();
      }
  }
  ```

* 执行结果

  ```shell
  Bean1 create
  Bean2 create
  Bean3 create
  
  bean3 destroy
  bean2 destroy
  bean1 destroy
  ```

## 通过depend-on干预bean创建和销毁顺序

这个可以确保depend-on指定的bean在当前bean创建之前先创建好，销毁顺序刚好相反

使用注解@DependOn

```java
@DependOn("beanN-1")
@Component
public class BeanN implements DisposableBean {

    public Bean1() {
        System.out.println("BeanN create");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("beanN destroy");
    }
}
```

# @Primary的作用

## 使用前提

* BeanConfig

  ```java
  @Configuration
  public class BeanConfig {
      @Bean
      public Component1 component1() {
          return new Component1();
      }
  
      @Bean
      public Component1 component1_() {
          return new Component1();
      }
  }
  ```

* BeanService

  ```java
  @Component
  public class BeanService {
      @Autowired
      private Component1 component1;
  
      public void test() {
          System.out.println(component1);
      }
  }
  ```

* SpringStudyApplication

  ```java
  @SpringBootApplication
  public class SpringStudyApplication {
      public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException {
          ConfigurableApplicationContext context = SpringApplication.run(SpringStudyApplication.class);
          Component1 component1 = context.getBean("component1", Component1.class);
          Component1 component1_ = context.getBean("component1_", Component1.class);
          System.out.println(component1);
          System.out.println(component1_);
          BeanService beanService = context.getBean(BeanService.class);
          beanService.test();
      }
  }
  ```

* 执行结果

  ```shell
  top.mnsx.spring.study.component.Component1@39651a82
  top.mnsx.spring.study.component.Component1@6be7bf6d
  top.mnsx.spring.study.component.Component1@39651a82
  ```

## 使用@Primary

* BeanConfig

  ```java
  @Configuration
  public class BeanConfig {
      @Bean
      public Component1 component1() {
          return new Component1();
      }
  
      @Primary
      @Bean
      public Component1 component1_() {
          return new Component1();
      }
  }
  ```

* 执行结果

  ```shell
  top.mnsx.spring.study.component.Component1@4038cd3a
  top.mnsx.spring.study.component.Component1@14ac77b9
  top.mnsx.spring.study.component.Component1@14ac77b9
  ```

> **通过类型自动注入依赖时，使用@Primary修饰的类，能够优先被注入**

# Bean对象懒加载-@Lazy

一般情况下，Spring容器在启动时会创建所有的Bean对象，**使用@Lazy注解可以将Bean对象的创建延迟到第一次使用Bean的时候**

* LazyBean

  ```java
  @Lazy
  @Component
  public class LazyBean {
      public LazyBean() {
          System.out.println("LazyBean create");
      }
  }
  ```

* SpringStudyApplicationContext

  ```java
  @SpringBootApplication
  public class SpringStudyApplication {
      public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException {
          ConfigurableApplicationContext context = SpringApplication.run(SpringStudyApplication.class);
          context.getBean("lazyBean");
      }
  }
  ```

* 执行结果

  ```shell
  Bean1 create
  Bean2 create
  Bean3 create
  ...
  LazyBean create
  ```

# 单例Bean中使用多例Bean

## 案例

> 单例BeanA中依赖多例BeanB，但是调用单例BeanA中的BeanB总是同一个，如何在BeanA中每次调用的BeanB都是新的？

* BeanA

  ```java
  @Component
  public class BeanA {
      @Autowired
      private BeanB beanB;
  
      public void test() {
          System.out.println(beanB);
      }
  }
  ```

* BeanB

  ```java
  @Scope("prototype")
  @Component
  public class BeanB {
  }
  ```

* SpringStudyApplication

  ```java
  @SpringBootApplication
  public class SpringStudyApplication {
      public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException {
          ConfigurableApplicationContext context = SpringApplication.run(SpringStudyApplication.class);
          BeanA beanA = context.getBean("beanA", BeanA.class);
          beanA.test();
          beanA.test();
      }
  }
  ```

* 执行结果

  ```shell
  top.mnsx.spring.study.component.BeanB@2cca611f
  top.mnsx.spring.study.component.BeanB@2cca611f
  ```

## 解决思路

在BeanA中，主动从容器中获取BeanB，这样就能宝恒BeanB的Scope是Prototype的

实现接口ApplicationContextAware可以在Bean中后去容器

```java
org.springframework.context.ApplicationContextAware
 
public interface ApplicationContextAware extends Aware {
    void setApplicationContext(ApplicationContext applicationContext) throws BeansException;
}
```

## 解决案例

```java
@Component
public class BeanA implements ApplicationContextAware {

    private ApplicationContext context;

    public void test() {
        System.out.println(context.getBean("beanB", BeanB.class));
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.context = applicationContext;
    }
}
```

执行结果——

```shell
top.mnsx.spring.study.component.BeanB@5e230fc6
top.mnsx.spring.study.component.BeanB@7a8341c1
```

# Spring代理详解

## Jdk动态代理

Jdk中实现代理主要依据两个类

```java
java.lang.reflect.Proxy
java.lang.reflect.InvocationHandler
```

Jdk自带的代理使用上有限制，只能为接口创建代理类，如果需要给具体的类创建代理类，需要使用CGLIB

* **getProxyClass方法**

  为指定的接口创建代理类，返回代理类的Class对象

  ```java
  public static Class<?> getProxyClass(ClassLoader loader,
                                           Class<?>... interfaces)
  ```

  loader：定义代理类的类加载器

  interfaces：指定需要实现的接口列表，创建的代理默认会按顺序实现interfaces指定的接口

* **newProxyInstance方法**

  创建代理的实例对象

  ```java
  public static Object newProxyInstance(ClassLoader loader,
                                            Class<?>[] interfaces,
                                            InvocationHandler h)
  ```

  当调用代理对象的任何方法时，会被InvocationHandler接口的invoke方法处理

* **isProxy方法**

  判断指定的类是否是一个代理类

  ```java
  public static boolean isProxyClass(Class<?> cl)
  ```

* **getInvocationHandler方法**

  获取代理对象的InvocationHandler对象

  ```java
  public static InvocationHandler getInvocationHandler(Object proxy)
          throws IllegalArgumentException
  ```

**创建代理** 

* ProxyInterface

  ```java
  public interface ProxyInterface {
      public void test();
  }
  ```

* ProxyComponent

  ```java
  public class ProxyComponent implements ProxyInterface {
      public void test() {
          System.out.println("hhh");
      }
  }
  ```

* TestProxy

  ```java
  public class TestProxy<T> {
      private T target;
  
      public TestProxy(T target) {
          this.target = target;
      }
  
      @SuppressWarnings("unchecked")
      public T getProxyObject() {
          return (T) Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(), (proxy, method, args) -> {
              // 前置方法
              Object result = method.invoke(target, args);
              // 后置方法
              return result;
          });
      }
  }
  ```

* StudySpringApplication

  ```java
  @SpringBootApplication
  public class SpringStudyApplication {
      public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException {
          ConfigurableApplicationContext context = SpringApplication.run(SpringStudyApplication.class);
  
          TestProxy<ProxyInterface> testProxy = new TestProxy<>(new ProxyComponent());
          ProxyInterface proxyObject = testProxy.getProxyObject();
          proxyObject.test();
      }
  }
  ```

## CGLIB动态代理

CGLIB是一个强大、高性能的字节码生成库，它用于运行时扩展Java类和实现接口，本质上它动态的生成一个子类去覆盖所要代理的类

Enhancer既能够代理普通的class，也能够代理接口

**Enhancer创建一个被代理对象的子类并且拦截所有的方法调用**

**Enhancer不能拦截final方法**

### 使用CGLIB动态代理的流程

1. 创建Enhancer对象
2. 通过setSuperclass设置父类型
3. 设置回调函数，需要实现`org.springframework.cglib.proxy.Callback`，MethodInterceptor实现了Callback接口，所有方法调用都会被intercept方法拦截
4. 获取代理对象，调用enhancer.create方法

> **案例1**
>
> 拦截所有方法（MethodInterceptor）

* ProxyService

  ```java
  public class ProxyService {
      public String test() {
          System.out.println("test1");
          return "test1";
      }
  }
  ```

* CglibTest

  ```java
  @Test
  public void test1() {
      Enhancer enhancer = new Enhancer();
      enhancer.setSuperclass(ProxyService.class);
      enhancer.setCallback(new MethodInterceptor() {
          @Override
          public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
              System.out.println(method);
              return methodProxy.invokeSuper(o, objects);
          }
      });
      ProxyService proxyService = (ProxyService) enhancer.create();
      proxyService.test();
  }
  ```

> **案例2**
>
> 拦截所有方法并返回固定值（Fixed Value）

* CglibTest

  ```java
  @Test
  public void test2() {
      Enhancer enhancer = new Enhancer();
      enhancer.setSuperclass(ProxyService.class);
      enhancer.setCallback((FixedValue) () -> {
          return "hhh";
      });
      ProxyService proxyService = (ProxyService) enhancer.create();
      System.out.println(proxyService.test());
  }
  ```

> **案例3**
>
> 直接放行，不做任何操作（NoOp.INSTANCE）

* CglibTest

  ```java
  @Test
  public void test3() {
      Enhancer enhancer = new Enhancer();
      enhancer.setSuperclass(ProxyService.class);
      enhancer.setCallback(NoOp.INSTANCE);
      ProxyService proxyService = (ProxyService) enhancer.create();
      System.out.println(proxyService.test());
  }
  ```


> **案例4**
>
> 不同的方法使用不同的拦截器（CallbackFilter）

* CglibTest

   ```java
   @Test
   public void test4() {
       Enhancer enhancer = new Enhancer();
       enhancer.setSuperclass(ProxyService.class);
       Callback[] callbacks = {
           (MethodInterceptor) (o, method, objects, methodProxy) -> {
               System.out.println(method);
               return methodProxy.invokeSuper(o, objects);
           },
           (FixedValue) () -> "Hello world"
       };
       enhancer.setCallbacks(callbacks);
       enhancer.setCallbackFilter((method) -> {
           String methodName = method.getName();
           return methodName.startsWith("insert") ? 0 : 1;
       });
       ProxyService proxyService = (ProxyService) enhancer.create();
       proxyService.insert();
       System.out.println(proxyService.get());
   }
   ```

* ProxyService

  ```java
  public String get() {
      System.out.println("get");
      return "get";
  }
  
  public String insert() {
      System.out.println("insert");
      return "insert";
  }
  ```

> **案例5**
>
> 对案例4进行优化

* CglibTest

  ```java
  @Test
  public void test5() {
      Enhancer enhancer = new Enhancer();
      enhancer.setSuperclass(ProxyService.class);
      Callback callback = (MethodInterceptor) (o, method, objects, methodProxy) ->{
          System.out.println(method);
          return methodProxy.invokeSuper(o, objects);
      };
  
      Callback fixedValueCallback = (FixedValue) () -> "Hello world";
  
      CallbackHelper callbackHelper = new CallbackHelper(ProxyService.class, null) {
          @Override
          protected Object getCallback(Method method) {
              return method.getName().startsWith("insert") ? callback : fixedValueCallback;
          }
      };
  
      enhancer.setCallbacks(callbackHelper.getCallbacks());
      enhancer.setCallbackFilter(callbackHelper);
      ProxyService proxyService = (ProxyService) enhancer.create();
      proxyService.insert();
      System.out.println(proxyService.get());
  }
  ```

# 深入理解Java注解

注解是对代码的一种增强，可以在代码编译或者程序运行期间获取注解的信息，然后根据这些信息操作

## 使用注解的步骤

* **定义注解语法**

  jdk中注解相关的接口和类都定义在java.lang.annotation中

  注解的定义和常见类、接口相似，只是注解使用@interface来定义

  ```java
  public @interface MyAnnotation {
  }
  ```

* **注解中定义参数**

  注解中有没有参数都可以

  注解中可以定义多个参数，特点如下

  * 访问修饰符必须为public，不写默认为public
  * 该元素的类型只能是基本数据类型，String、Class、枚举类型、注解类型、上述类型的数组
  * 该元素名称一般定义为名词，如果注解中只有一个元素，将名字起为value
  * 参数名称后面的()不是定义方法参数的地方，不能再括号中定义任何参数，仅仅是一个特殊的语法
  * default代表默认值
  * 没有默认值，代表后续使用注解时必须给该类型元素赋值

* **指定注解的使用范围**：@Target

  使用@Target注解定义注解的使用范围、

  ```java
  @Target(value = {ElementType.TYPE, ElementType.METHOD})
  public @interface MyAnnotation {
  }
  ```

  不主动声明注解的使用范围的话，默认标配是可以用在任何地方

  **@Targer源码**

  ```java
  @Documented
  @Retention(RetentionPolicy.RUNTIME)
  @Target(ElementType.ANNOTATION_TYPE)
  public @interface Target {
      ElementType[] value();
  }
  ```

  有一个参数value，是ElementType类型的数组

  ```java
  package java.lang.annotation;
  /*注解的使用范围*/
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

* **指定注解的保留策略**：@Retention

  ```java
  @Retention(RetentionPolicy.SOURCE)
  public @interface MyAnnotaion {
  }
  ```

  **@Retention源码**

  ```java
  @Documented
  @Retention(RetentionPolicy.RUNTIME)
  @Target(ElementType.ANNOTATION_TYPE)
  public @interface Retention {
      RetentionPolicy value();
  }
  ```

  有一个参数value，是RetentionPolicy枚举

  ```java
  public enum RetentionPolicy {
      /*注解只保留在源码中，编译为字节码之后就丢失了，也就是class文件中就不存在了*/
      SOURCE,
      /*注解只保留在源码和字节码中，运行阶段会丢失*/
      CLASS,
      /*源码、字节码、运行期间都存在*/
      RUNTIME
  }
  ```

> **@Target(ElementType.TYPE_PARAMETER)**
>
> 用来标注类型参数，类型参数一般用于类后面声明或者方法上声明
>
> ```java
> @Target(value = {
>         ElementType.TYPE_PARAMETER
> })
> @Retention(RetentionPolicy.RUNTIME)
> public @interface TypeParameterAnnotation {
>     String value();
> }
> ```
>
> ```java
> public class AnnotationService1<@TypeParameterAnnotation("声明在类泛型参数上") P1> {
>     public <@TypeParameterAnnotation("声明在方法泛型参数上") P2> void m1() {
>     }
> }
> ```
>
> **@Target(ElementType.TYPE_USER)**
>
> 能用在任何类型名称上
>
> ```java
> @Target({ElementType.TYPE_USE})
> @Retention(RetentionPolicy.RUNTIME)
> public @interface TypeUseAnnotation {
>     String value();
> }
> ```
>
> ```java
> @TypeUseAnnotation("声明在类上")
> public class AnnotationService2<@TypeUseAnnotation("声明在类泛型参数上") P1> {
>     private Map<@TypeUseAnnotation("声明在类的泛型参数类型上")String, Integer> map;
> 
>     public <@TypeParameterAnnotation("声明在方法泛型参数上") P2> void m1() {
>     }
> }
> ```

## 注解信息的获取

为了运行时准确获取注解相关信息，在java.lang.reflect反射包下新增了AnnotatedElement接口，它主要用于表示目前正在虚拟机中运行的程序中已使用注解的元素，通过该接口提供的方法可以利用反射获取注解信息

**AnnotatedElement常用方法**

| 返回值                  | 方法名称                                                     | 说明                                                         |
| ----------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| \<A extends Annotation> | getAnnotation(Class\<A> annotationClass)                     | 该元素如果存在指定类型的注解，则返回这些注解，否则返回null   |
| Annotation[]            | getAnnotations()                                             | 返回此元素上存在的所有注解，包括从父类继承的                 |
| boolean                 | isAnnotationPresent(Class<? extends Annotation> annotationClass) | 如果指定类型的注解存在于此元素上，则返回true，否则返回false  |
| Annotation[]            | getDeclaredAnnotations()                                     | 返回直接存在于此元素上的所有注解，注意，不包括父类的注解，调用者可以随意修改返回的数组，这不会对其他调用者返回的数组产生任何影响，没有则返回长度为0的数组 |

**@Inherit：实现类之间的注解继承**

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Inherited {
}
```

让子类可以继承父类中被@inherited修饰的注解，**注意是继承父类中的，如果接口中的注解也是用了@Inherited修饰了，那么接口的实现类无法继承这个注解**

**@Repeatable重复使用注解**

```java
@AnnTest
@AnnTest
public class test() {}
```

如果像上面代码将会报错，因为在test方法上重复使用了注解，默认情况下注解是不允许重复使用的，这个时候就应该使用@Repeatable注解

**@Repeatable使用步骤**

* 先定义容器注解

  ```java
  @Retention(RetentionPolicy.RUNTIME)
  @Target({ElementType.TYPE, ElementType.FIELD})
  @interface AnnTests {
      AnnTest value(); 
  }
  ```

  **容器注解中必须有个value类型的参数，参数类型为子注解类型的数组**

* 为注解指定容器

  ```java
  @Retention(RetentionPolicy.RUNTIME)
  @Target({ElementType.TYPE, ElementType.FIELD})
  @Repeatable(AnnTests.class)
  @interface AnnTest {
      String name();
  }
  ```

* 使用注解

  * 重复使用注解

    ```java
    @AnnTest(name = "hhh")
    @AnnTest(name = "www")
    public class AnnotationTest {}
    ```

  * 通过容器注解来使用更多个注解

    ```java
    @AnnTest2({
        @AnnTest(name = "hhh"),
        @AnnTest(name = "www")
    })
    public class AnnotationTest {}
    ```

## Spring @AliasFor：对注解进行增强

### 问题重现

- AnnotationA

  ```
  @Target(value = {ElementType.TYPE})
  @Retention(value = RetentionPolicy.RUNTIME)
  public @interface AnnotationA {
      String value() default "a";
  }
  ```

- AnnotationB

  ```
  @Target(value = {ElementType.TYPE})
  @Retention(value = RetentionPolicy.RUNTIME)
  @AnnotationA("hhh")
  public @interface AnnotationB {
      String value() default "b";
  }
  ```

- NormalAnnoService

  ```
  @AnnotationB
  @Service
  public class NormalAnnoService {
      public void test1() {
          System.out.println(NormalAnnoService.class.getAnnotation(AnnotationB.class));
          System.out.println(AnnotationB.class.getAnnotation(AnnotationA.class));
      }
  }
  ```

- 问题

  无法通过NormalAnnotationService对B中的A进行设值，注解定义是无法被继承的

### 使用@AliaFor解决

- AnnotationB

  ```
  @Target(value = {ElementType.TYPE})
  @Retention(value = RetentionPolicy.RUNTIME)
  @AnnotationA
  public @interface AnnotationB {
      String value() default "b";
  
      @AliasFor(annotation = AnnotationA.class, value = "value")
      String annotationAValue();
  }
  ```

- NormalAnnoService

  ```
  @AnnotationB(value = "b", annotationAValue = "hhh")
  @Service
  public class NormalAnnoService {
      public void test1() {
          System.out.println(NormalAnnoService.class.getAnnotation(AnnotationB.class));
          System.out.println(AnnotatedElementUtils.getMergedAnnotation(NormalAnnoService.class, AnnotationA.class));
      }
  }
  ```

  > **遇到的问题**：
  >
  > 最开始使用原生Java的`System.out.println(NormalAnnoService.class.getAnnotation(AnnotationA.class))`
  >
  > 最后打印的值为null
  >
  > **问题解析**：
  >
  > **因为@AliasFor是Spring的特殊注解，不是Java原生支持，因此要使用Spring的工具类取值**

## 特殊注意

* @AliasFor如果不指定annotation参数的值，那么annotation默认值就是当前注释
* AliasFor注解中value和attribute互为别名，随便设置一个，同时会给另外一个设置相同的值
* 如果@AliasFor中不指定value和attribute，那么自动将@AliasFor修饰的参数作为value和attribute的值

# @Configuration和@Bean注解

## @Configuration

@configuration这个注解可以在类上，让这个类的功能等同于一个bean.xml配置文件

## @Bean

这个注解类似于bean.xml配置文件中的bean元素，用来在spring容器中注册一个bean

@Bean注解用在方法上，表示通过方法来定义一个bean，默认将方法名作为bean名称，将方法返回值作为bean对象，注册到spring容器中

**@Bean注解源码**

```java
@Target({ElementType.METHOD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Bean {
 	// value和name是一样能够的，设置的时候，这两个参数只能选一个，原因是@AliasFor的value属性只能选择一个属性
    @AliasFor("name")
    String[] value() default {};
    @AliasFor("value")
    String[] name() default {};
 
    // autowire参数表示了@Deprecated，表示已经过期，不建议使用
    @Deprecated
    Autowire autowire() default Autowire.NO;
 
    /**
     * autowireCandidate表示是否作为其他对象注入时的候选bean
     * 如果Bean注入时按照类型注入，同时存在两个相同类型的Bean，这是就会报错
     * 我们可以将一个Bean的autowireCandidate属性改为false，那么这个Bean将不会作为候选Bean
     */
    boolean autowireCandidate() default true;
 
    // bean的初始化方法
    String initMethod() default "";
 
    // bean的销毁方法
    String destroyMethod() default AbstractBeanDefinition.INFER_METHOD;
}
```

## 如果去掉Configuration会怎么样？

* 去掉Configuration

  ```shell
  beanConfig:[]:top.mnsx.spring.study.config.BeanConfig@1372ed45
  component1:[]:top.mnsx.spring.study.component.Component1@6a79c292
  ```

* 不去掉Configuration

  ```shell
  beanConfig:[]:top.mnsx.spring.study.config.BeanConfig$$EnhancerBySpringCGLIB$$9f22a8c3@4ca8dbfa
  component1:[]:top.mnsx.spring.study.component.Component1@5f780a86
  ```

* 结论
  1. 有没有@Configuration注解，@Bean都会起效，只要将Config注册到容器中即可生效
  2. @Configuration修饰的bean最后输出的时候带有EnhancerBySpringCGLIB，说明这个bean被CGLIB处理过的，会变成一个代理对象

**@Configuration加不加区别**

* ServiceA

  ```jaava
  public class ServiceA {
  }
  ```

* ServiceB

  ```java
  public class ServiceB {
      private ServiceA serviceA;
  
      public ServiceB(ServiceA serviceA) {
          this.serviceA = serviceA;
      }
  
      @Override
      public String toString() {
          return "ServiceB{" + "serviceA=" + serviceA + '}';
      }
  }
  ```

* BeanConfig1

  ```java
  @Configuration
  public class BeanConfig1 {
      @Bean
      public ServiceA serviceA() {
          System.out.println("调用ServiceA方法");
          return new ServiceA();
      }
  
      @Bean
      public ServiceB serviceB1() {
          System.out.println("调用ServiceB1方法");
          ServiceA serviceA = this.serviceA();
          return new ServiceB(serviceA);
      }
  
      @Bean
      public ServiceB serviceB2() {
          System.out.println("调用ServiceB2方法");
          ServiceA serviceA = this.serviceA();
          return new ServiceB(serviceA);
      }
  }
  ```

* 执行结果

  ```shell
  调用ServiceA方法
  调用ServiceB1方法
  调用ServiceB2方法
  beanConfig1:[]:top.mnsx.spring.study.config.BeanConfig1$$EnhancerBySpringCGLIB$$431d57d6@4ea5b703
  serviceA:[]:top.mnsx.spring.study.service.ServiceA@2a7ed1f
  serviceB1:[]:ServiceB{serviceA=top.mnsx.spring.study.service.ServiceA@2a7ed1f}
  serviceB2:[]:ServiceB{serviceA=top.mnsx.spring.study.service.ServiceA@2a7ed1f}
  ```

  **ServiceA方法只是被调用了一次，但是ServiceB1和ServiceB2中的ServiceA都是同一个**

  这是因为被@Configuration修饰的类，Spring容器中通过CGLIB创建一个代理，代理会拦截所有被@Bean修饰的方法，默认情况下确保这些方法只被调用一次，从而确保bean都是同一个（单例）

* 不加Configuration的执行结果

  ```shell
  调用ServiceA方法
  调用ServiceB1方法
  调用ServiceA方法
  调用ServiceB2方法
  调用ServiceA方法
  beanConfig1:[]:top.mnsx.spring.study.config.BeanConfig1@4ea5b703
  serviceA:[]:top.mnsx.spring.study.service.ServiceA@2a7ed1f
  serviceB1:[]:ServiceB{serviceA=top.mnsx.spring.study.service.ServiceA@795cd85e}
  serviceB2:[]:ServiceB{serviceA=top.mnsx.spring.study.service.ServiceA@59fd97a8}
  ```

## Spring中Configuration源码

这个类主要处理@Configuration这个注解

```java
org.springframework.context.annotation.ConfigurationClassPostProcessor
```

主要的处理方法

```java
// 将配置类创建成Bean添加到容器中，并且将其依赖也添加到容器中
public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) 
public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory)
public void enhanceConfigurationClasses(ConfigurableListableBeanFactory beanFactory) 
```

```java
@Override
public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) {
    // identityHashCode会返回对象的hashCode，而不管对象是否重写了hashCode方法
    int registryId = System.identityHashCode(registry);
   	// 判断postProcessBeanDefinitionRegistry是否已经被执行过
    if (this.registriesPostProcessed.contains(registryId)) {
        throw new IllegalStateException(
            "postProcessBeanDefinitionRegistry already called on this post-processor against " + registry);
    }
    // 判断postProcessBeanFactory是否已经被执行过
    if (this.factoriesPostProcessed.contains(registryId)) {
        throw new IllegalStateException(
            "postProcessBeanFactory already called on this post-processor against " + registry);
    }
    // 将注册器ID添加到容器中，防止重复执行
    this.registriesPostProcessed.add(registryId);
	// 详情见下
    processConfigBeanDefinitions(registry);
}
```

```java
public void processConfigBeanDefinitions(BeanDefinitionRegistry registry) {
    // 候选配置容器
    List<BeanDefinitionHolder> configCandidates = new ArrayList<>();
    // 获取注册器中的BeanDefinition
    String[] candidateNames = registry.getBeanDefinitionNames();

    // 循环处理candidateNames
    for (String beanName : candidateNames) {
       	// 根据beanName获取对应的BeanDefinition
        BeanDefinition beanDef = registry.getBeanDefinition(beanName);
       	// 不懂跳过
        if (beanDef.getAttribute(ConfigurationClassUtils.CONFIGURATION_CLASS_ATTRIBUTE) != null) {
            if (logger.isDebugEnabled()) {
                logger.debug("Bean definition has already been processed as a configuration class: " + beanDef);
            }
        }
        // 判断指定的BeanDefinition是不是配置类
        else if (ConfigurationClassUtils.checkConfigurationClassCandidate(beanDef, this.metadataReaderFactory)) {
            // 将其添加到容器中
            configCandidates.add(new BeanDefinitionHolder(beanDef, beanName));
        }
    }

    // 如果容器是空直接跳出
    if (configCandidates.isEmpty()) {
        return;
    }

    // 根据@Order注解进行排序
    configCandidates.sort((bd1, bd2) -> {
        int i1 = ConfigurationClassUtils.getOrder(bd1.getBeanDefinition());
        int i2 = ConfigurationClassUtils.getOrder(bd2.getBeanDefinition());
        return Integer.compare(i1, i2);
    });

    // 判断Bean名称生成策略？
    SingletonBeanRegistry sbr = null;
    // 判断registry是否为单例
    if (registry instanceof SingletonBeanRegistry) {
        sbr = (SingletonBeanRegistry) registry;
        if (!this.localBeanNameGeneratorSet) {
            BeanNameGenerator generator = (BeanNameGenerator) sbr.getSingleton(
                AnnotationConfigUtils.CONFIGURATION_BEAN_NAME_GENERATOR);
            if (generator != null) {
                this.componentScanBeanNameGenerator = generator;
                this.importBeanNameGenerator = generator;
            }
        }
    }

    // 配置默认环境
    if (this.environment == null) {
        this.environment = new StandardEnvironment();
    }

    // 解析被@Configuration修饰的类
    ConfigurationClassParser parser = new ConfigurationClassParser(
        this.metadataReaderFactory, this.problemReporter, this.environment,
        this.resourceLoader, this.componentScanBeanNameGenerator, registry);

    // 去重
    Set<BeanDefinitionHolder> candidates = new LinkedHashSet<>(configCandidates);
    // 已经被解析的容器
    Set<ConfigurationClass> alreadyParsed = new HashSet<>(configCandidates.size());
    // 循环解析
    do {
        parser.parse(candidates);
        parser.validate();

        Set<ConfigurationClass> configClasses = new LinkedHashSet<>(parser.getConfigurationClasses());
        configClasses.removeAll(alreadyParsed);

        // 读取配置信息，创建配置Bean
        if (this.reader == null) {
            this.reader = new ConfigurationClassBeanDefinitionReader(
                registry, this.sourceExtractor, this.resourceLoader, this.environment,
                this.importBeanNameGenerator, parser.getImportRegistry());
        }
        this.reader.loadBeanDefinitions(configClasses);
        alreadyParsed.addAll(configClasses);

        candidates.clear();
        // 将配置类中所有被用到的bean对象添加到容器中，如果有配置对象将其加入容器，解析其内部依赖
        if (registry.getBeanDefinitionCount() > candidateNames.length) {
            String[] newCandidateNames = registry.getBeanDefinitionNames();
            Set<String> oldCandidateNames = new HashSet<>(Arrays.asList(candidateNames));
            Set<String> alreadyParsedClasses = new HashSet<>();
            for (ConfigurationClass configurationClass : alreadyParsed) {
                alreadyParsedClasses.add(configurationClass.getMetadata().getClassName());
            }
            for (String candidateName : newCandidateNames) {
                if (!oldCandidateNames.contains(candidateName)) {
                    BeanDefinition bd = registry.getBeanDefinition(candidateName);
                    if (ConfigurationClassUtils.checkConfigurationClassCandidate(bd, this.metadataReaderFactory) &&
                        !alreadyParsedClasses.contains(bd.getBeanClassName())) {
                        candidates.add(new BeanDefinitionHolder(bd, candidateName));
                    }
                }
            }
            candidateNames = newCandidateNames;
        }
    }
    while (!candidates.isEmpty());

    // 创建单例配置类
    if (sbr != null && !sbr.containsSingleton(IMPORT_REGISTRY_BEAN_NAME)) {
        sbr.registerSingleton(IMPORT_REGISTRY_BEAN_NAME, parser.getImportRegistry());
    }

    if (this.metadataReaderFactory instanceof CachingMetadataReaderFactory) {
        ((CachingMetadataReaderFactory) this.metadataReaderFactory).clearCache();
    }
}
```

```java
@Override
public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
  	// 获取beanFactory的hashcode
    int factoryId = System.identityHashCode(beanFactory);

    if (this.factoriesPostProcessed.contains(factoryId)) {
        throw new IllegalStateException(
            "postProcessBeanFactory already called on this post-processor against " + beanFactory);
    }
    // 添加到容器中，防止重复
    this.factoriesPostProcessed.add(factoryId);

    if (!this.registriesPostProcessed.contains(factoryId)) {
        // BeanDefinitionRegistryPostProcessor hook apparently not supported...
        // Simply call processConfigurationClasses lazily at this point then.
        processConfigBeanDefinitions((BeanDefinitionRegistry) beanFactory);
    }

    // 详情见下
    enhanceConfigurationClasses(beanFactory);
    // 添加Bean的前置执行，ImportAwareBeanPostProcessor作用见下
    beanFactory.addBeanPostProcessor(new ImportAwareBeanPostProcessor(beanFactory));
}
```

```java
public void enhanceConfigurationClasses(ConfigurableListableBeanFactory beanFactory) {
    // 存放配置Bean的信息的容器
    Map<String, AbstractBeanDefinition> configBeanDefs = new LinkedHashMap<>();
    // 获取beanFacotry中的所有bean，循环处理
    for (String beanName : beanFactory.getBeanDefinitionNames()) {
        // 通过beanName获取Bean的信息
        BeanDefinition beanDef = beanFactory.getBeanDefinition(beanName);
        // 获取一个参数，用来判断是否为配置对象？
        Object configClassAttr = beanDef.getAttribute(ConfigurationClassUtils.CONFIGURATION_CLASS_ATTRIBUTE);
        // 获取方法的元数据
        MethodMetadata methodMetadata = null;
       	// 判断Bean是否被注解修饰的类
        if (beanDef instanceof AnnotatedBeanDefinition) {
            // 获取工厂方法的元数据？
            methodMetadata = ((AnnotatedBeanDefinition) beanDef).getFactoryMethodMetadata();
        }
        // 判断是否为参数类或者是否能够获取元数据
        if ((configClassAttr != null || methodMetadata != null) && beanDef instanceof AbstractBeanDefinition) {
            AbstractBeanDefinition abd = (AbstractBeanDefinition) beanDef;
            // 查看BeanDefinition是否指定了对象的Bean类
            if (!abd.hasBeanClass()) {
                try {
                    // 通过类名和类加载器，解析Bean类
                    abd.resolveBeanClass(this.beanClassLoader);
                }
                catch (Throwable ex) {
                    throw new IllegalStateException(
                        "Cannot load configuration class: " + beanDef.getBeanClassName(), ex);
                }
            }
        }
        // 判断是否为配置类
        if (ConfigurationClassUtils.CONFIGURATION_CLASS_FULL.equals(configClassAttr)) {
            if (!(beanDef instanceof AbstractBeanDefinition)) {
                throw new BeanDefinitionStoreException("Cannot enhance @Configuration bean definition '" +
                                                       beanName + "' since it is not stored in an AbstractBeanDefinition subclass");
            }
            else if (logger.isInfoEnabled() && beanFactory.containsSingleton(beanName)) {
                logger.info("Cannot enhance @Configuration bean definition '" + beanName +
                            "' since its singleton instance has been created too early. The typical cause " +
                            "is a non-static @Bean method with a BeanDefinitionRegistryPostProcessor " +
                            "return type: Consider declaring such methods as 'static'.");
            }
            // 将配置类加到配置类容器中
            configBeanDefs.put(beanName, (AbstractBeanDefinition) beanDef);
        }
    }
    if (configBeanDefs.isEmpty()) {
        return;
    }

    // 创建enhancer
    ConfigurationClassEnhancer enhancer = new ConfigurationClassEnhancer();
    // 遍历所有的配置类，创建其代理对象
    for (Map.Entry<String, AbstractBeanDefinition> entry : configBeanDefs.entrySet()) {
        AbstractBeanDefinition beanDef = entry.getValue();
        
        beanDef.setAttribute(AutoProxyUtils.PRESERVE_TARGET_CLASS_ATTRIBUTE, Boolean.TRUE);
        
        Class<?> configClass = beanDef.getBeanClass();
        Class<?> enhancedClass = enhancer.enhance(configClass, this.beanClassLoader);
        if (configClass != enhancedClass) {
            if (logger.isTraceEnabled()) {
                logger.trace(String.format("Replacing bean definition '%s' existing class '%s' with " +
                                           "enhanced class '%s'", entry.getKey(), configClass.getName(), enhancedClass.getName()));
            }
            beanDef.setBeanClass(enhancedClass);
        }
    }
}
```

# ImportAware接口作用

@Import注解的使用和ImportAware接口相关

@Import注解注入Bean的方法有三种

1. 基于Configuration Class
2. 基于ImportSelector接口
3. 基于ImportBeanDefinitionRegistrar接口

基于ImportSelector接口和ImportBeanDefinitionRegistrar接口的方式都可以从方法上获取AnnotationMetadata（注解原信息），基于Configuration Class方式就只能通过实现ImportAware接口

**案例**

* ProxyMode

  ```java
  public class ProxyMode {
      private String mode;
  
      public String getMode() {
          return mode;
      }
  
      public void setMode(String mode) {
          this.mode = mode;
      }
  }
  ```

* EnableProxy

  ```java
  @Target(ElementType.TYPE)
  @Retention(RetentionPolicy.RUNTIME)
  @Import(ProxyConfiguration.class)
  public @interface EnableProxy {
      String mode() default "jdk";
  }
  ```

* ProxyConfiguration

  ```java
  @Configuration
  public class ProxyConfiguration implements ImportAware {
      
      private AnnotationAttributes info;
      
      @Bean
      public ProxyMode proxyMode() {
          ProxyMode proxyMode = new ProxyMode();
          proxyMode.setMode(info.getString("mode"));
          return proxyMode;
      }
  
      @Override
      public void setImportMetadata(AnnotationMetadata importMetadata) {
         this.info = AnnotationAttributes.fromMap(
                 importMetadata.getAnnotationAttributes(EnableProxy.class.getName(), false)
         );
      }
  }
  ```

**主要就是在ConfigurationClassPostProcessor中注入了一个ImportAwareBeanPostProcessor，在Bean的生命周期中将属性设置进去**

# @ComponentScan、@ComponentScans详解

## @ComponentScan

@ComponentScan用于批量注册bean

这个注解会让Spring去扫描某些包及其子包中的所有类，然后将满足一定条件的类作为Bean注册到容器中

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Repeatable(ComponentScans.class) //@1
public @interface ComponentScan {

    // 指定需要扫描的包
	@AliasFor("basePackages")
	String[] value() default {};
	@AliasFor("value")
	String[] basePackages() default {};

    // 指定类，Spring容器会扫描这些类所在的包及其子包中的类
	Class<?>[] basePackageClasses() default {};

    // 自定义名称生成器
	Class<? extends BeanNameGenerator> nameGenerator() default BeanNameGenerator.class;

	Class<? extends ScopeMetadataResolver> scopeResolver() default AnnotationScopeMetadataResolver.class;
	ScopedProxyMode scopedProxy() default ScopedProxyMode.DEFAULT;

    // 需要扫描包中的那些资源，默认是：**/*.class，即会扫描指定包中所有的class文件
	String resourcePattern() default "**/*.class";

    // 对扫描的包是否启用默认过滤器，默认为true
	boolean useDefaultFilters() default true;

    // 过滤器，用来配置被扫描出来的哪些类会作为组件注册到容器中
	Filter[] includeFilters() default {};

    // 过滤器，用来对扫描到的类进行排除，被排除的类不会被注册到容器中
	Filter[] excludeFilters() default {};

    // 是否延迟初始化被注册的bean
	boolean lazyInit() default false;
}
```

**@ComponentScan的工作过程**

1. Spring会扫描指定的包，且会递归下面的子包，得到一批类的数组
2. 然后这些类会经过上面的各种过滤器，最后剩下的类会被注册到容器中

**默认情况下，任何参数都不设置的情况下，此时，会将@ComponentScan修饰的类所在的包作为扫描包：默认情况下useDefaultFilters为true，Spring容器内部会使用默认的过滤器，规则是：类上有@Repository、@Service、@Controller、@Component这几个注解其中一个的，那么这个类就会被作为bean注册到Spring容器中**

### includeFilter的使用

```java
Filter[] includeFilters() default {};
```

是一个Filter类型的数组，多个Filter之间为或者关系，即满足任意一个就可以了

```java
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
     * /
	FilterType type() default FilterType.ANNOTATION;

	/**
	 * 当type=FilterType.ANNOTATION时，通过classes参数可以指定一些注解，用来判断被扫描的类上是否有classes参数指定的注解
	 * 当type=FilterType.ASSIGNABLE_TYPE时，通过classes参数可以指定一些类型，用来判断被扫描的类是否是classes参数指定的类型
	 * 当type=FilterType.CUSTOM时，表示这个过滤器时用户自定义的，classes参数就是用来指定用户自定的过滤器，自定义过滤器需要实现org.springframework.core.type.filter.TypeFilter接口
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
```

### 自定义Filter

1. 设置@Filter中type的类型：FilterType.CUSTOM
2. 自定义过滤器类，需要实现接口：org.springframework.core.type.filter.TypeFilter
3. 设置@Filter中的classes为自定义的过滤器类型

```java
@FunctionalInterface
public interface TypeFilter {
	boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory)
			throws IOException;
}
```

TypeFilter是一个函数式接口，包含一个match方法，方法返回boolean类型，有两个参数，都是接口类型的

* **MetadataReader接口**

  类元数据读取器，可以读取一个类上的任意信息，如类上面的注解信息，类的磁盘路径信息，类的class对象的各种信息

  ```java
  public interface MetadataReader {
  
  	/**
  	 * 返回类文件的资源引用
  	 */
  	Resource getResource();
  
  	/**
  	 * 返回一个ClassMetadata对象，可以通过这个读想获取类的一些元数据信息，如类的class对象、是否是接口、是否有注解、是否是抽象类、父类名称、接口名称、内部包含的之类列表等等
  	 */
  	ClassMetadata getClassMetadata();
  
  	/**
  	 * 获取类上所有的注解信息
  	 */
  	AnnotationMetadata getAnnotationMetadata();
  
  }
  ```

* **MetadataReaderFactory接口**

  类元数据读取器工厂，可以通过这个类获取任意一个类的MetadataReader对象

  ```java
  public interface MetadataReaderFactory {
  
  	/**
  	 * 返回给定类名的MetadataReader对象
  	 */
  	MetadataReader getMetadataReader(String className) throws IOException;
  
  	/**
  	 * 返回指定资源的MetadataReader对象
  	 */
  	MetadataReader getMetadataReader(Resource resource) throws IOException;
  
  }
  ```

> 案例
>
> 自定义Filter

* MyFilter

  ```java
  public class MyFilter implements TypeFilter {
      @Override
      public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException {
          Class<?> curClass = null;
          try {
              curClass = Class.forName(metadataReader.getClassMetadata().getClassName());
          } catch (ClassNotFoundException e) {
              e.printStackTrace();
          }
  
          // 判断当前curClass是否时IService类型的
          return IService.class.isAssignableFrom(curClass);
      }
  }
  ```

* SpringStudyApplication

  ```java
  @ComponentScan(
          basePackages = {"top.mnsx.spring.study"},
          useDefaultFilters = false,
          includeFilters = {
                  @ComponentScan.Filter(type = FilterType.CUSTOM, classes = MyFilter.class)
          }
  )
  @SpringBootApplication
  public class SpringStudyApplication {
      public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException {
          ConfigurableApplicationContext context = SpringApplication.run(SpringStudyApplication.class);
          for (String beanName : context.getBeanDefinitionNames()) {
              if ("serviceA".equals(beanName)) {
                  System.out.println(beanName + "->" + context.getBean(beanName));
              }
          }
      }
  }
  ```

## @ComponentScan重复使用

* @Repeatable注解表示这个注解可以多次使用

  ```java
  @ComponentScan(basePackageClasses = ScanClass.class)
  @ComponentScan(
          useDefaultFilters = false, 
          includeFilters = {
                  @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = IService.class)
          })
  public class BeanConfig {
  }
  ```

* 同样可以使用@ComponentScans来包装

  ```java
  @ComponentScans({
          @ComponentScan(basePackageClasses = ScanClass.class),
          @ComponentScan(
                  useDefaultFilters = false,
                  includeFilters = {
                          @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = IService.class)
                  })})
  public class BeanConfig {
  }
  ```

# @Import批量注册bean

## 目前已知批量注册Bean方法的漏洞

* 如果需要注册的类在第三方的jar中
  1. 通过@Bean标注方法的方式，一个个来注册
  2. @ComponentScan的方式：默认的@ComponentScan是没办法执行的，此时只能自定义过滤器来实现

* 通常项目中有很多子模块，每个模块都是独立开发，最后通过jar包引入，每个模块都有自己的bean配置类，此时，我们只需要其中几个模块的配置类

## 使用@Import

@Import可以用来批量导入需要注册的各种类，作用就是和xml配置的\<import/>标签作用一样，允许通过它的引入@Configuration标注的类，引入ImportSelector接口和ImportBeanDefinitionRegistrar接口的实现，包括@Component注解的普通类

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Import {
	Class<?>[] value();
}
```

**@Import的value常见的5种用法**

1. value为普通类
2. value为@Configuration标注的类
3. value为@ComponentScan标注的类
4. value为@ImportBeanDefinitionRegistrar接口类型
5. value为ImportSelector接口类型
6. value为DeferredImportSelector接口类型

## value案例

### value为普通的类

* Service1

  ```java
  public class Service1 {
  }
  ```

* Service2

  ```java
  public class Service2 {
  }
  ```

* MainConfig1

  ```java
  @Import({Service1.class, Service2.class})
  public class MainConfig1 {
  }
  ```

* ImportTest

  ```java
  @SpringBootTest
  public class ImportTest {
      @Test
      public void test1() {
          AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MainConfig1.class);
          for (String beanName : context.getBeanDefinitionNames()) {
              System.out.println(beanName + "->" + context.getBean(beanName));
          }
      }
  }
  ```

* 运行结果

  ```shell
  mainConfig1->top.mnsx.spring.study.config.MainConfig1@16736040
  top.mnsx.spring.study.service.Service1->top.mnsx.spring.study.service.Service1@7c5d1d25
  top.mnsx.spring.study.service.Service2->top.mnsx.spring.study.service.Service2@550e9be6
  ```

### value为@Configuration标注的配置类

* ConfigModule1

  ```java
  @Configuration
  public class ConfigModule1 {
      @Bean
      public String module1() {
          return "module1";
      }
  }
  ```

* ConfigModule2

  ```java
  @Configuration
  public class ConfigModule2 {
      @Bean
      public String module2() {
          return "module2";
      }
  }
  ```

* MainConfig2

  ```java
  @Import({ConfigModule1.class, ConfigModule2.class})
  public class MainConfig2 {   
  }
  ```

* ImportTest

  ```java
  @Test
  public void test2() {
      AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MainConfig2.class);
      for (String beanName : context.getBeanDefinitionNames()) {
          System.out.println(beanName + "->" + context.getBean(beanName));
      }
  }
  ```

* 运行结果

  ```shell
  mainConfig2->top.mnsx.spring.study.config.MainConfig2@16736040
  top.mnsx.spring.study.module1.config.ConfigModule1->top.mnsx.spring.study.module1.config.ConfigModule1
  top.mnsx.spring.study.module2.config.ConfigModule2->top.mnsx.spring.study.module2.config.ConfigModule2
  ```

### value为@ComponentScan标注的类

* Module1Service1

  ```java
  @Component
  public class Module1Service1 {
  }
  ```

* Module1Service2

  ```java
  @Component
  public class Module1Service2 {
  }
  ```

* ComponentScanModule1

  ```java
  @ComponentScan
  public class ComponentScanModule1 {
  }
  ```

* Module2Service1

  ```java
  @Component
  public class Module2Service1 {
  }
  ```

* Module2Service2

  ```java
  @Component
  public class Module2Service2 {
  }
  ```

* ComponentScanModule2

  ```java
  @ComponentScan
  public class ComponentScanModule2 {
  }
  ```

* MainConfig3

  ```java
  @Import({ComponentScanModule1.class, ComponentScanModule2.class})
  public class MainConfig3 {
  }
  ```

* ImportTest

  ```java
  @Test
  public void test3() {
      AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MainConfig3.class);
      for (String beanName : context.getBeanDefinitionNames()) {
          System.out.println(beanName + "->" + context.getBean(beanName));
      }
  }
  ```

* 执行结果

  ```shell
  module1Service1->top.mnsx.spring.study.module1.service.Module1Service1@5b239d7d
  module1Service2->top.mnsx.spring.study.module1.service.Module1Service2@6572421
  module2Service1->top.mnsx.spring.study.module2.service.Module2Service1@6b81ce95
  module2Service2->top.mnsx.spring.study.module2.service.Module2Service2@2a798d51
  ```

### ImportBeanDefinitionRegistrar接口

这个接口提供了通过Spring容器api的方式向容器种注册bean

```java
public interface ImportBeanDefinitionRegistrar {

    
	default void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry,
			BeanNameGenerator importBeanNameGenerator) {

		registerBeanDefinitions(importingClassMetadata, registry);
	}

	default void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
	}

}
```

* ImportingClassMetadata

  AnnotationMetadata类型，通过这个可以获取被@Import注解标识的类的所有注解的信息

* registry

  BeanDefinitionRegistry类型，是一个接口，内部提供了注册bean的各种方法

* ImportBeanNameGenerator

  BeanNameGenerator类型，是一个接口，内部有一个方法，用来生成bean的名称

**BeanDefinitionRegistry接口：bean定义注册器**

bean定义注册器，提供了bean注册的各种方法

```java
public interface BeanDefinitionRegistry extends AliasRegistry {

	/**
	 * 注册一个新的bean定义
	 * beanName：bean的名称
	 * beanDefinition：bean定义信息
	 */
	void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
			throws BeanDefinitionStoreException;

	/**
	 * 通过bean名称移除已注册的bean
	 * beanName：bean名称
	 */
	void removeBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;

	/**
	 * 通过名称获取bean的定义信息
	 * beanName：bean名称
	 */
	BeanDefinition getBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;

	/**
	 * 查看beanName是否注册过
	 */
	boolean containsBeanDefinition(String beanName);

	/**
	 * 获取已经定义（注册）的bean名称列表
	 */
	String[] getBeanDefinitionNames();

	/**
	 * 返回注册器中已注册的bean数量
	 */
	int getBeanDefinitionCount();

	/**
	 * 确定给定的bean名称或者别名是否已在此注册表中使用
	 * beanName：可以是bean名称或者bean的别名
	 */
	boolean isBeanNameInUse(String beanName);

}
```

基本上所有bean工厂都实现了这个接口，让bean工厂拥有bean注册的各种能力

**BeanNameGenerator接口：bean名称生成器**

bean名称生成器，这个接口只有一个方法，用来生成bean的名称

```java
public interface BeanNameGenerator {
	String generateBeanName(BeanDefinition definition, BeanDefinitionRegistry registry);
}
```

* DefaultBeanNameGenerator

  默认bean名称生成器，xml中bean为指定名称的时候，默认就会使用这个生成器，默认为：完整的类名#bean编号

* AnnotationBeanNameGenerator

  注解方式的bean名称生成器，通过注解方式指定bean名称，如果没有通过注解方式指定名称，默认会将完整的类名作为bean名称

* FullyQualifiedAnnotationBeanNameGenerator

  将完整的类作为bean的名称

**BeanDefinition接口：bean定义信息**

用来表示bean定义信息的接口，我们向容器中注册bean之前，bean的所有配置信息都会被转换为一个BeanDefinition对象，然后通过容器中的BeanDefinitionRegistry接口中的方法，将BeanDefinition注册到Spring容器中

###  value为ImportBeanDefinitionRegistrar接口类型

> 使用步骤：
>
> 1. 定义ImportBeanDefinitionRegistrar接口实现类，在registerBeanDefinitions方法中使用registry来注册bean
> 2. 使用@Import来导入步骤1中定义的类
> 3. 使用步骤2 中Import标注的类作为AnnotationConfigApplicationContext构建参数创建spring容器
> 4. 使用AnnotationConfigApplicationContext操作bean

* Service1

  ```java
  public class Service1 {
  }
  ```

* Service2

  ```java
  public class Service2 {
      private Service1 service1;
      
      public Service1 getService1() {
          return service1;
      }
      
      public void setService1(Service1 Service1) {
          this.service1 = service1;
      }
      
      @Override
      public String toString() {
          return "Service2{" + 
              "service1=" + service1 + 
              '}';
      }
  }
  ```

* MyImportBeanDefinitionRegistrar

  ```java
  public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
      @Override
      public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
          AbstractBeanDefinition beanDefinition = BeanDefinitionBuilder.genericBeanDefinition(Service1.class).getBeanDefinition();
          registry.registerBeanDefinition("service1", beanDefinition);
          AbstractBeanDefinition beanDefinition1 = BeanDefinitionBuilder.genericBeanDefinition(Service2.class)
                  .addPropertyReference("service1", "service1")
                  .getBeanDefinition();
          registry.registerBeanDefinition("service2", beanDefinition1);
      }
  }
  ```

* MainConfig4

  ```java
  @Import(MyImportBeanDefinitionRegistrar.class)
  public class MainConfig4 {
  }
  ```

* ImportTest

  ```java
  @Test
  public void test4() {
      AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MainConfig4.class);
      for (String beanName : context.getBeanDefinitionNames()) {
          System.out.println(beanName + "->" + context.getBean(beanName));
      }
  }
  ```

### value为ImportSelector接口类型

```java
public interface ImportSelector {

	/**
	 * 返回需要导入的类名的数组，可以是任何普通类，配置类（@Configuration、@Bean、@CompontentScan等标注的类）
	 * @importingClassMetadata：用来获取被@Import标注的类上面所有的注解信息
	 */
	String[] selectImports(AnnotationMetadata importingClassMetadata);

}
```

> 步骤
>
> 1. 定义ImportSelector接口实现类，在selectImports返回需要导入的类的名称数组
> 2. 使用@Import来导入步骤1中定义的类
> 3. 使用步骤2中@Import标注的类为AnnotationConfigApplicationContext构造参数创建Spring容器
> 4. 使用AnnotationConfigApplicationContext操作bean

* Service1

  ```java
  public class Service1 {
  }
  ```

* Module1Config

  ```java
  @Configuration
  public class ModuleConfig {
      @Bean
      public String name() {
          return "Mnsx";
      }
      
      @Bean
      public String address() {
          return "cdc";
      }
  }
  ```

* MyImportSelector

  ```java
  public class MyImportSelector implements ImportSelector {
  
      @Override
      public String[] selectImports(AnnotationMetadata importingClassMetadata) {
          return new String[] {
                  Service1.class.getName(), 
                  ModuleConfig.class.getName()
          };
      }
  }
  ```

* MainConfig5

  ```java
  @Import({MyImportSelector.class})
  public class MainConfig5 {
  }
  ```

* ImportTest

  ```java
  @Test
  public void test5() {
      AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MainConfig5.class);
      for (String beanName : context.getBeanDefinitionNames()) {
          System.out.println(beanName + "->" + context.getBean(beanName));
      }
  }
  ```

## 常用注解添加配置类案例

* Service1

  ```java
  public class Service1 {
      public void m1() {
          System.out.println("m1执行成功");
      }
  }
  ```

* CostTimeProxy

  ```java
  public class CostTimeProxy implements MethodInterceptor {
      private Object target;
  
      public CostTimeProxy(Object target) {
          this.target = target;
      }
  
      @Override
      public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
          long startTime = System.nanoTime();
          Object result = method.invoke(target, objects);
          long endTime = System.nanoTime();
          System.out.println(method + "，耗时（纳秒）：" + (endTime - startTime));
          return result;
      }
  
      public static <T> T createProxy(T target) {
          CostTimeProxy costTimeProxy = new CostTimeProxy(target);
          Enhancer enhancer = new Enhancer();
          enhancer.setSuperclass(target.getClass());
          enhancer.setCallback(costTimeProxy);
          return (T) enhancer.create();
      }
  }
  ```

* MethodCostTimeProxyBeanPostProcessor

  BeanPostProcessor接口的源码

  ```java
  org.springframework.beans.factory.config.BeanPostProcessor
  
  public interface BeanPostProcessor {
  
  	/**
  	 * bean初始化之后会调用的方法
  	 */
  	@Nullable
  	default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
  		return bean;
  	}
  
  	/**
  	 * bean初始化之后会调用的方法
  	 */
  	@Nullable
  	default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
  		return bean;
  	}
  
  }
  ```

  ```java
  public class MethodCostTimeProxyBeanPostProcessor implements BeanPostProcessor {
      @Override
      public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
          if (bean.getClass().getName().toLowerCase().contains("service")) {
              return CostTimeProxy.createProxy(bean);
          }
          return bean;
      }
  }
  ```

* MethodCostTimeImportSelector

  ```java
  public class MethodCostTimeImportSelector implements ImportSelector {
      @Override
      public String[] selectImports(AnnotationMetadata importingClassMetadata) {
          return new String[] {
                  MethodCostTimeProxyBeanPostProcessor.class.getName()
          };
      }
  }
  ```

* MainConfig6

  ```java
  @ComponentScan(basePackageClasses = Service1.class, includeFilters = {
      @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = {Service1.class})
  })
  @EnableMethodCostTime
  public class MainConfig6 {
  }
  ```

* ImportTest

  ```java
  @Test
  public void test6() {
      AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MainConfig6.class);
      Service1 service = context.getBean(Service1.class);
      service.m1();
  }
  ```

## DeferredImportSelector接口

SpringBoot中的核心功能@EnableAutoConfiguration主要依靠DeferredImportSelector接口实现的

DeferredImportSelector是ImportSelector的子接口，所以也可以通过@Import进行导入，这个接口和ImportSelector不同的地方有两个

* 延迟导入
* 指定导入的类的处理顺序

### 延迟导入

如果@Import的value包含需要导入的bean，会将DeferredImportSelector类型的放在最后处理，会先处理其他被导入的类，其他类会按照value所在的前后顺序进行处理

### 指定导入的类的处理顺序

当@Import中有多个DeferredImportSelector接口的实现类时，可以指定他们的顺序，指定顺序常见的两种方式

1. **实现Ordered接口的方式**

   ```java
   org.springframework.core.Ordered
   
   public interface Ordered {
   
   	int HIGHEST_PRECEDENCE = Integer.MIN_VALUE;
   
   	int LOWEST_PRECEDENCE = Integer.MAX_VALUE;
   
   	int getOrder();
   
   }
   ```

   **value的值越小，优先级越高**

2. 实现Order注解的方式

   ```java
   org.springframework.core.annotation.Order
   
   @Retention(RetentionPolicy.RUNTIME)
   @Target({ElementType.TYPE, ElementType.METHOD, ElementType.FIELD})
   @Documented
   public @interface Order {
   
   	int value() default Ordered.LOWEST_PRECEDENCE;
   
   }
   ```

   **value的值越小，优先级越高**

# @Conditional通过条件来控制bean的注册

@Condition注解可以用在任何类型或者方法上，通过@Confitional注解可以配置一些条件判断，当所有条件满足，被@Confitional标注的目标才会被Spring容器处理

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Conditional {
    Class<? extends Condition>[] value();
}
```

这个注解只有一个value参数，Condition类型的数组，Condition是一个接口，表示一个条件判断，内部有个方法返回true或false，当所有Conditon都成功，@Conditional才会成立

```java
@FunctionalInterface
public interface Condition {
 
    /**
     * 判断条件是否匹配
     * context:条件判断上下文
     */
    boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata);
 
}
```

是一个函数式接口，内部只有一个matches方法，用来判断条件是否成立

* context：条件上下文，ConditionContext接口类型，可以用来获取容器中的个人信息
* metadata：用来获取被@Conditional标注的对象上的所有注解信息

**ConditionContext接口**

这个接口中提供了一些常用的方法，可以用来获取Spring容器中的各种信息

```java
public interface ConditionContext {
 
    /**
     * 返回bean定义注册器，可以通过注册器获取bean定义的各种配置信息
     */
    BeanDefinitionRegistry getRegistry();
 
    /**
     * 返回ConfigurableListableBeanFactory类型的bean工厂，相当于一个ioc容器对象
     */
    @Nullable
    ConfigurableListableBeanFactory getBeanFactory();
 
    /**
     * 返回当前spring容器的环境配置信息对象
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

## 条件判断在什么时候执行？

* 配置类解析阶段

  会得到一批配置类的信息，和一些需要注册的Bean

* bean注册阶段

  将配置类解析阶段得到的配置类和需要注册的bean注册到Spring容器中

### Spring对配置类处理过程

```java
org.springframework.context.annotation.ConfigurationClassPostProcessor#processConfigBeanDefinitions
```

一个配置类被Spring处理有2个阶段：配置类解析阶段、bean注册阶段（将配置类作为bean注册到spring容器中）

如果将Condition接口的实现类作为配置类上@Condition中，那么这个条件会对两个阶段都有效，此时通过Condition是无法精细的控制某个阶段，如果想要控制某个阶段，此时就需要用到另外的接口ConfigurationCondition

**ConfigurationCondition接口**

```java
public interface ConfigurationCondition extends Condition {
 
    /**
     * 条件判断的阶段，是在解析配置类的时候过滤还是在创建bean的时候过滤
     */
    ConfigurationPhase getConfigurationPhase();
 
 
    /**
     * 表示阶段的枚举：2个值
     */
    enum ConfigurationPhase {
 
        /**
         * 配置类解析阶段，如果条件为false，配置类将不会被解析
         */
        PARSE_CONFIGURATION,
 
        /**
         * bean注册阶段，如果为false，bean将不会被注册
         */
        REGISTER_BEAN
    }
 
}
```

ConfigurationCondition接口相对于Condition接口多了一个getConfigurationPhase方法，用来指定条件判断的阶段，是在解析配置类的时候过滤还是在创建Bean的时候过滤

### @Conditional使用的3个步骤

1. 自定义一个类，实现Conditoin或者ConfiguarionCondition接口，实现matches方法
2. 在目标对象上使用@Conditional注解，并指定value为自定义的Condition
3. 启动Spring容器加载资源，此时@Condition就起作用了

## 相关案例

> 案例1
>
> 阻止配置类的处理

在 配置类上面使用@Conditional，这个注解的value指定的Condition当中有一个false的时候，Spring就会跳过处理这个配置类

* MyCondition

  ```java
  public class MyCondition1 implements Condition {
      @Override
      public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
          return false;
      }
  }
  ```

* MainConfig1

  ```java
  @Conditional(MyCondition1.class)
  @Configuration
  public class MainConfig1 {
      @Bean
      public String name() {
          return "Mnsx";
      }
  }
  ```

* ConditionTest

  ```java
  @SpringBootTest
  public class ConditionTest {
      @Test
      public void test1() {
          AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MainConfig1.class);
          context.getBeansOfType(String.class).forEach((beanName, bean) -> {
              System.out.println(beanName + ":" + bean);
          });
      }
  }
  ```

> 案例2
>
> 阻止bean的注册

* MainConfig2

  ```java
  @Configuration
  public class MainConfig2 {
      @Conditional(MyCondition1.class)
      @Bean
      public String name() {
          return "mnsx";
      }
  
      @Bean
      public String address() {
          return "cdc";
      }
  }
  ```

* ConditionTest

  ```java
  @Test
  public void test2() {
      AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MainConfig2.class);
      context.getBeansOfType(String.class).forEach((beanName, bean) -> {
          System.out.println(beanName + ":" + bean);
      });
  }
  ```

> 案例3
>
> bean不存在的时候才注册

* OnMissingBeanCondition

  ```java
  public class OnMissingBeanCondition implements Condition {
      @Override
      public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
          ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
          Map<String, IService> serviceMap = beanFactory.getBeansOfType(IService.class);
          return serviceMap.isEmpty();
      }
  }
  ```

> 案例4
>
> 根据环境选择配置类

* EnvConditional

  ```java
  @Conditional(EnvCondition.class)
  @Target(ElementType.TYPE)
  @Retention(RetentionPolicy.RUNTIME)
  public @interface EnvConditional {
      enum Env {
          TEST, DEV, PROD
      }
  
      Env value() default Env.DEV;
  }
  ```

* EnvCondition

  ```java
  public class EnvCondition implements Condition {
      @Override
      public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
          EnvConditional.Env curEnv = EnvConditional.Env.DEV;
          EnvConditional.Env env = (EnvConditional.Env) metadata.getAllAnnotationAttributes(EnvConditional.class.getName()).get("value").get(0);
          return env.equals(curEnv);
      }
  }
  ```

> 案例5
>
> Condition指定优先级

**默认情况下，Condition执行顺序，按照@Conditional中value值的顺序是一样的**

**自定义的Condition可以实现PriorityOrdered接口或者继承Ordered接口，或者使用@Order注解，来指定优先级**

> 案例6
>
> ConfigurationCondition使用

* Service

  ```java
  public class Service {
  }
  ```

* BeanConfig3

  ```java
  @Configuration
  public class BeanConfig3 {
      @Bean
      public Service service() {
          return new Service();
      }
  }
  ```

* BeanConfig4

  ```java
  @Conditional(MyCondition2.class)
  @Configuration
  public class BeanConfig4 {
      @Bean
      public String name() {
          return "mnsx";
      }
  }
  ```

* MainConfig5

  ```java
  @Configuration
  @Import({BeanConfig3.class, BeanConfig4.class})
  public class MainConfig5 {
  }
  ```

* MyCondition2

  ```java
  public class MyCondition2 implements Condition {
      @Override
      public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
          ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
          return !beanFactory.getBeansOfType(Service.class).isEmpty();
      }
  }
  ```

配置类的处理会依次经过两个阶段：配置类解析阶段和bean注册阶段，Condition接口类型的条件会对这两个阶段都有效，解析阶段的时候，容器中还没有Service这个bean的，配置类中通过@Bean注解定义的bean在bean注册阶段才会被注册到容器中，所以BeanConfig2在解析阶段去容器中看不到Service这个Bean的，所以被拒绝

* MyConfigurationCondition

  ```java
  public class MyConfigurationCondition implements ConfigurationCondition {
      @Override
      public ConfigurationPhase getConfigurationPhase() {
          return ConfigurationPhase.REGISTER_BEAN;
      }
  
      @Override
      public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
          ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
          return !beanFactory.getBeansOfType(Service.class).isEmpty();
      }
  }
  ```

# 注解实现依赖注入

## @Autowired依赖注入

**实现依赖注入，Spring容器会对bean中所有字段、方法进行遍历，标注@Autowored注解的，都会进行注入**

```java
@Target({ElementType.CONSTRUCTOR, ElementType.METHOD, ElementType.PARAMETER, ElementType.FIELD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Autowired {
 
    /**
     * Declares whether the annotated dependency is required.
     * <p>Defaults to {@code true}.
     */
    boolean required() default true;
 
}
```

### @Autowired查找候选者的过程

* @Autowired标注在字段上面

  ![Autowired_1](D:\WorkSpace\Mnsx-Note\2023\Picture\Spring\@Autowired_1.png)

* @Autowired标注在方法上后者方法参数上

  ![Autowired_2](D:\WorkSpace\Mnsx-Note\2023\Picture\Spring\@Autowired_2.png)

**将指定类型的所有Bean注入到Collection中**

如果被注入的对象是Collection类型的，可以指定泛型的类型，然后回按照上面的方式查找所有满足泛型类型所有的bean

**将指定类型的所有Bean注入到Map中**

如果被注入的对象是Map类型的，可以指定泛型的类型，key通常为String类型，value为需要查找的bean的类型，然后会按照上面方式查找所有注入value类型的bean，将bean的那么作为key，bean对象作为value，放在HashMap中，然后注入

> @Autowired简化
>
> **按类型找->通过限定符@Qualifier过滤->@Primary->@Priority->根据名称找（字段名称或者方法名称）**

### @Autowired源码

```java
org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor
```

## @Resource依赖注入

**与@Autowired类似，也是用来注入依赖对象的，spring容器会对bean中所有字段、方法进行遍历，标注有@Resource的，都会进行注入**

```java
javax.annotation.Resource
 
@Target({TYPE, FIELD, METHOD})
@Retention(RUNTIME)
public @interface Resource {
    String name() default "";
    // ...其他不常用的参数省略
}
```

> 这个注解是Javax中定义的，并不是Spring定义的注解
>
> 可以用在任何类型、字段、方法上
>
> **使用在方法上，只能有一个参数**

### @Resource查找候选者的过程

* @Resource标注在字段上

![Resource_1](D:\WorkSpace\Mnsx-Note\2023\Picture\Spring\@Resource_1.png)

* @Resource标注在方法上或者方法参数上

  ![Resource_2](D:\WorkSpace\Mnsx-Note\2023\Picture\Spring\@Resource_2.png)

**将指定类型的所有Bean注入到Collection中**

如果被注入的对象是Collection类型的，可以指定泛型的类型，然后回按照上面的方式查找所有满足泛型类型所有的bean

**将指定类型的所有Bean注入到Map中**

如果被注入的对象是Map类型的，可以指定泛型的类型，key通常为String类型，value为需要查找的bean的类型，然后会按照上面方式查找所有注入value类型的bean，将bean的那么作为key，bean对象作为value，放在HashMap中，然后注入

> @Resource简化
>
> **先按Resource的name值作为bean名称找->按名称（字段名称、方法名称、set属性名称）找->按类型找->通过限定符@Qualifier过滤->@Primary->@Priority->根据名称找（字段名称或者方法参数名称）**

### @Resource源码

```java
org.springframework.context.annotation.CommonAnnotationBeanPostProcessor
```

## @Qualifier限定符

**可以在依赖注入查找候选者的过程中对候选者进行过滤**

```java
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Qualifier {
 
    String value() default "";
 
}
```

## @Primary设置优先候选者

**注入依赖的过程中，当有多个候选者的时候，可以指定哪个候选者为主要的候选者**

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Primary {
 
}
```

## 定义Bean时依赖注入的方式

1. 硬编码
2. @Autowired、@Resource的方式
3. @Bean标识的方法参数方式

# Bean生命周期详解

## Spring中Bean生命周期13个环节

1. Bean元信息配置阶段
2. Bean元信息解析阶段
3. 将Bean注册到容器中
4. BeanDefinition合并阶段
5. Bean Class加载阶段
6. Bean实例化阶段
   * Bean实例化前阶段
   * Bean实例化阶段
7. 合并后的BeanDefinition处理
8. 属性赋值阶段
   * Bean实例化后阶段
   * Bean属性赋值前阶段
   * Bean属性赋值阶段
9. Bean初始化阶段
   * Bean Aware接口回调阶段
   * Bean初始化前阶段
   * Bean初始化阶段
   * Bean初始化后阶段
10. 所有单例Bean初始化完成后阶段
11. Bean的使用阶段
12. Bean的销毁前阶段
13. Bean的销毁阶段

## Bean元信息配置阶段



# Spring事务中7种传播行为

## 什么是事务传播行为？

> 事务的传播行为（Transaction Propagation Behavior）是指在一个事务方法调用另一个事务方法时，被调用方法如何参与到当前事务或启动一个新的事务			——ChatGPT

## 如何配置事务传播行为？

通过@Transactional注解中的propagation属性来指定事务的传播行为

```java
Propagation propagation() default Propagation.REQUIRED;
```

## 事务的7种传播行为

Propagation是一个枚举类

```java
public enum Propagation {
	// 如果事务管理器中有事务，那么加入这个事务中，如果没有那么就新建一个事务
	REQUIRED(TransactionDefinition.PROPAGATION_REQUIRED),
	// 如果事务管理器中有事务，那么加入这个事务中，如果没有那么以非实物执行
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

## 其他问题

1. Spring声明式事务处理事务的过程

   通过事务拦截器TransactionInterceptor拦截目标方法，来实现事务管理的拦截功能

2. 何时事务会回滚

   默认情况下，目标方法抛出RuntimeException或者Error的时候，事务会被回滚

3. Spring事务管理中的Connection和操作DB中使用的Connection怎样使用的是同一个

   当事务管理器开启事务的时候，通过dataSource.getConnection()方法获取一个连接，然后将这个链接放入一个Map中，将Map放入ThreadLocal中

   当操作DB时，会通过dataSource去ThreadLocal中查找，是否存在可用链接

   所以事务管理中的Connection和操作DB使用的是同一个

## 代码验证

### 验证准备

* 数据库

  ```sql
  DROP DATABASE IF EXISTS mnsx_spring;
  CREATE DATABASE if NOT EXISTS mnsx_spring;
  
  USE mnsx_spring;
  DROP TABLE IF EXISTS temp_1;
  CREATE TABLE temp_user_1(
                        id int PRIMARY KEY AUTO_INCREMENT,
                        name varchar(64) NOT NULL DEFAULT '' COMMENT '姓名'
  );
  
  DROP TABLE IF EXISTS temp_user_2;
  CREATE TABLE temp_user_2(
                        id int PRIMARY KEY AUTO_INCREMENT,
                        name varchar(64) NOT NULL DEFAULT '' COMMENT '姓名'
  );
  ```

* pom.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <project xmlns="http://maven.apache.org/POM/4.0.0"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
      <modelVersion>4.0.0</modelVersion>
  
      <groupId>top.mnsx</groupId>
      <artifactId>mnsx_spring_study</artifactId>
      <version>1.0-SNAPSHOT</version>
      <build>
          <plugins>
              <plugin>
                  <groupId>org.apache.maven.plugins</groupId>
                  <artifactId>maven-compiler-plugin</artifactId>
                  <configuration>
                      <source>8</source>
                      <target>8</target>
                  </configuration>
              </plugin>
          </plugins>
      </build>
  
      <dependencies>
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-web</artifactId>
              <version>2.3.7.RELEASE</version>
          </dependency>
          <dependency>
              <groupId>com.baomidou</groupId>
              <artifactId>mybatis-plus-boot-starter</artifactId>
              <version>3.4.1</version>
          </dependency>
          <dependency>
              <groupId>mysql</groupId>
              <artifactId>mysql-connector-java</artifactId>
              <version>8.0.30</version>
          </dependency>
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-configuration-processor</artifactId>
              <version>2.3.7.RELEASE</version>
          </dependency>
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-test</artifactId>
              <version>2.3.7.RELEASE</version>
          </dependency>
      </dependencies>
  
  </project>
  ```

* application.yml

  ```yaml
  spring:
    datasource:
      url: jdbc:mysql://192.168.92.172:3306/mnsx_spring?characterEncoding=utf8&useSSL=true
      username: root
      password: xx1527030652
      driver-class-name: com.mysql.cj.jdbc.Driver
  ```

* domain、mapper

  使用MybatisX自动生成

* service

  * User1Service

    ```java
    @Service
    public class User1Service {
        @Resource
        private User1Mapper user1Mapper;
    }
    ```

  * User2Service

    ```java
    @Service
    public class User2Service {
        @Resource
        private User2Mapper user2Mapper;
    }
    ```

  * TxService

    ```java
    @Service
    public class TxService {
        @Autowired
        private User1Service user1Service;
        @Autowired
        private User2Service user2Service;
    }
    ```

* config

  ```java
  @Configuration
  @MapperScan("top.mnsx.spring.study.mapper")
  public class MyBatisPlusConfig {
  }
  ```

* test

  ```java
  @SpringBootTest
  public class DemoTest {
      @Autowired
      private TxService txService;
      @Resource
      private User1Mapper user1Mapper;
      @Resource
      private User2Mapper user2Mapper;
  
      @BeforeEach
      public void before() {
          user1Mapper.delete(null);
          user2Mapper.delete(null);
      }
  
      @AfterEach
      public void after() {
          user1Mapper.selectList(null).forEach(System.out::println);
          user2Mapper.selectList(null).forEach(System.out::println);
      }
  }
  ```

### REQUIRED

* User1Service

  ```java
  @Transactional(propagation = Propagation.REQUIRED)
  public void required(String name) {
      User1 user1 = new User1();
      user1.setName(name);
      user1Mapper.insert(user1);
  }
  ```

* User2Service

  ```java
  @Transactional(propagation = Propagation.REQUIRED)
  public void required(String name) {
      User2 user2 = new User2();
      user2.setName(name);
      user2Mapper.insert(user2);
  }
  
  @Transactional(propagation = Propagation.REQUIRED)
  public void requiredException(String name) {
      User2 user2 = new User2();
      user2.setName(name);
      user2Mapper.insert(user2);
      throw new RuntimeException();
  }
  ```

> 情景1
>
> 外围方法没有事务，内部调用两个REQUIRED级别的事务方法

* TxService

  ```java
  public void noTransactionExceptionRequiredRequired() {
      user1Service.required("zs");
      user2Service.required("ls");
      throw new RuntimeException();
  }
  ```

* DemoTest

  ```java
  @Test
  public void noTransactionExceptionRequiredRequired() {
      txService.noTransactionExceptionRequiredRequired();
  }
  ```

* 执行结果

  ```shell
  User1 [Hash = 5044, id=6, name=zs, serialVersionUID=1]
  User2 [Hash = 4610, id=6, name=ls, serialVersionUID=1]
  RuntimeException
  ```

* TxService

  ```java
  public void noTransactionRequiredRequiredException() {
      user1Service.required("zs");
      user2Service.requiredException("ls");
  }
  ```

* DemoTest

  ```java
  @Test
  public void noTransactionRequiredRequiredException() {
      txService.noTransactionRequiredRequiredException();
  }
  ```

* 执行结果

  ```shell
  User1 [Hash = 5075, id=7, name=zs, serialVersionUID=1]
  RuntimeException
  ```

> **结论：通过两个方法增明在外围方法未开启事务的情况下Propagation.REQUIRED修饰的颞部方法会开启自己的事务，且开启的事务相互独立，互不干扰**

> 情景2
>
> 外部方法也开启事务

* TxService

  ```java
  @Transactional(propagation = Propagation.REQUIRED)
  public void transactionRequiredRequired() {
      user1Service.required("zs");
      user2Service.required("ls");
  }
  ```

* DemoTest

  ```java
  @Test
  public void transactionRequiredRequired() {
      txService.transactionRequiredRequired();
  }
  ```

* 执行结果

  ```shell
  User1 [Hash = 5106, id=8, name=zs, serialVersionUID=1]
  User2 [Hash = 4672, id=8, name=ls, serialVersionUID=1]
  ```

* TxService

  ```java
  @Transactional(propagation = Propagation.REQUIRED)
  public void transactionRequiredRequiredException() {
      user1Service.required("zs");
      user2Service.requiredException("ls");
  }
  ```

* DemoTest

  ```java
  @Test
  public void transactionRequiredRequiredException() {
      txService.transactionRequiredRequiredException();
  }
  ```

* 执行结果

  ```shell
  RuntimeException
  ```

* TxService

  ```java
  @Transactional(propagation = Propagation.REQUIRED)
  public void transactionExceptionRequiredRequired() {
      user1Service.required("zs");
      user2Service.required("ls");
      throw new RuntimeException();
  }
  ```

* DemoTest

  ```java
  @Test
  public void transactionExceptionRequiredRequired() {
      txService.transactionExceptionRequiredRequired();
  }
  ```

* 执行结果

  ```shell
  RuntimeException
  ```

> **结论：如果外部方法开启事务的情况下，那么内部方法开启事务后，会加入到外部方法的事务中，其中要给方法抛出异常，那么所有方法都会回滚**

### REQUIRES_NEW

* User1Service

  ```java
  @Transactional(propagation = Propagation.REQUIRES_NEW)
  public void requiresNew(String name) {
      User1 user1 = new User1();
      user1.setName(name);
      user1Mapper.insert(user1);
  }
  ```

* User2Service

  ```java
  @Transactional(propagation = Propagation.REQUIRED)
  public void requiredException(String name) {
      User2 user2 = new User2();
      user2.setName(name);
      user2Mapper.insert(user2);
      throw new RuntimeException();
  }
  
  @Transactional(propagation = Propagation.REQUIRES_NEW)
  public void requiresNew(String name) {
      User2 user2 = new User2();
      user2.setName(name);
      user2Mapper.insert(user2);
  }
  ```

> 情景1
>
> 外部方法没有开启事务

* TxService

  ```java
  public void noTransactionRequiresNewRequiresNew() {
      user1Service.requiresNew("zs");
      user2Service.requiresNew("ls");
  }
  ```

* DemoTest

  ```java
  @Test
  public void noTransactionRequiresNewRequiresNew() {
      txService.noTransactionRequiresNewRequiresNew();
  }
  ```

* 执行结果

  ```shell
  User1 [Hash = 5199, id=11, name=zs, serialVersionUID=1]
  User2 [Hash = 4765, id=11, name=ls, serialVersionUID=1]
  ```

* TxService

  ```java
  public void noTransactionRequiresNewRequiresNewException() {
      user1Service.requiresNew("zs");
      user2Service.requiresNewException("ls");
  }
  ```

* DemoTest

  ```java
  @Test
  public void noTransactionRequiresNewRequiresNewException() {
      txService.noTransactionRequiresNewRequiresNewException();
  }
  ```

* 执行结果

  ```shell
  User1 [Hash = 5230, id=12, name=zs, serialVersionUID=1]
  RuntimeException
  ```

> **结论：在外部方法未开启事务的情况下，requires_new修饰的内部方法会开启两个独立事务，互不干扰**

> 情况2
>
> 外部方法开启事务

* TxServive

  ```java
  @Transactional(propagation = Propagation.REQUIRED)
  public void transactionRequiredRequiresNewRequiresNewException() {
      user1Service.required("zs");
      user1Service.requiresNew("ls");
      user2Service.requiresNew("ww");
      throw new RuntimeException();
  }
  ```

* DemoTest

  ```java
  @Test
  public void transactionRequiredRequiresNewRequiresNewException() {
      txService.transactionRequiredRequiresNewRequiresNewException();
  }
  ```

* 执行结果

  ```shell
  User1 [Hash = 4889, id=15, name=ls, serialVersionUID=1]
  User2 [Hash = 5203, id=14, name=ww, serialVersionUID=1]
  RuntimeException
  ```

* TxService

  ```java
  @Transactional(propagation = Propagation.REQUIRED)
  public void transactionRequiredRequiresNewRequiresNewException() {
      user1Service.required("zs");
      user1Service.requiresNew("ls");
      user2Service.requiresNewException("ww");
  }
  ```

* DemoTest

  ```java
  @Test
  public void transactionRequiredRequiresNewRequiresNewExcpetion() {
      txService.transactionRequiredRequiresNewRequiresNewException();
  }
  ```

* 执行结果

  ```shell
  User1 [Hash = 5013, id=19, name=ls, serialVersionUID=1]
  RuntimeException
  ```

* TxService

  ```java
  @Transactional(propagation = Propagation.REQUIRED)
  public void transactionRequiredRequiresNewRequiresNewExceptionCatch() {
      user1Service.required("zs");
      user1Service.requiresNew("ls");
      try {
          user2Service.requiresNewException("ww");
      } catch (Exception e) {
          e.printStackTrace();
      }
  }
  ```

* DemoTest

  ```java
  @Test
  public void transactionRequiredRequiresNewRequiresNewExceptionCatch() {
      txService.transactionRequiredRequiresNewRequiresNewExceptionCatch();
  }
  ```

* 执行结果

  ```shell
  User1 [Hash = 5478, id=20, name=zs, serialVersionUID=1]
  User1 [Hash = 5075, id=21, name=ls, serialVersionUID=1]
  ```

> **结论：在外部方法开启Requires_New事务的情况下，内部方法和外部依旧是独立的，互不干扰，内部方法和内部方法之间也是相互独立的**

### NESTED

* User1Service

  ```java
  @Transactional(propagation = Propagation.NESTED)
  public void nested(String name) {
      User1 user1 = new User1();
      user1.setName(name);
      user1Mapper.insert(user1);
  }
  ```

* User2Service

  ```java
  @Transactional(propagation = Propagation.NESTED)
  public void nested(String name) {
      User2 user2 = new User2();
      user2.setName(name);
      user2Mapper.insert(user2);
  }
  
  @Transactional(propagation = Propagation.NESTED)
  public void nestedException(String name) {
      User2 user2 = new User2();
      user2.setName(name);
      user2Mapper.insert(user2);
      throw new RuntimeException();
  }
  ```

> 情景1
>
> 外部方法没有事务

* TxService

  ```java
  public void noTransactionExceptionNestedNested() {
      user1Service.nested("zs");
      user2Service.nested("ls");
      throw new RuntimeException();
  }
  ```

* DemoTest

  ```java
  @Test
  public void noTransactionExceptionNestedNested() {
      txService.noTransactionExceptionNestedNested();
  }
  ```

* 执行结果

  ```shell
  User1 [Hash = 5540, id=22, name=zs, serialVersionUID=1]
  User2 [Hash = 4455, id=1, name=ls, serialVersionUID=1]
  ```

* TxService

  ```java
  public void noTransactionNestedNestedException() {
      user1Service.nested("zs");
      user2Service.nestedException("ls");
  }
  ```

* DemoTest

  ```java
  @Test
  public void noTransactionNestedNestedException() {
      txService.noTransactionNestedNestedException();
  }
  ```

* 执行结果

  ```shell
  User1 [Hash = 5571, id=23, name=zs, serialVersionUID=1]
  RuntimeException
  ```

> **结论：在外部方法未开启事务的情况下NESTED和REQUIRED作用相同，修饰的内部方法都会开启自己的事务，相互独立互不干扰**

> 场景2
>
> 外部方法开启事务

* TxServicer

  ```java
  @Transactional(propagation = Propagation.NESTED)
  public void transactionExceptionNestedNested() {
      user1Service.nested("zs");
      user2Service.nested("ls");
      throw new RuntimeException();
  }
  ```

* DemoTest

  ```java
  @Test
  public void transactionExceptionNestedNested() {
      txService.transactionExceptionNestedNested();
  }
  ```

* 执行结果

  ```shell
  RuntimeException
  ```

* TxService

  ```java
  @Transactional(propagation = Propagation.NESTED)
  public void transactionNestedNestedException() {
      user1Service.nested("zs");
      user2Service.nestedException("ls");
  }
  ```

* DemoTest

  ```java
  @Test
  public void transactionNestedNestedException() {
      txService.transactionNestedNestedException();
  }
  ```

* 执行结果

  ```shell
  RuntimeException
  ```

* TxService

  ```java
  @Transactional(propagation = Propagation.NESTED)
  public void transactionNestedNestedExceptionCatch() {
      user1Service.nested("zs");
      try {
          user2Service.nestedException("ls");
      } catch (Exception e) {
          e.printStackTrace();
      }
  }
  ```

* DemoTest

  ```java
  @Test
  public void transactionNestedNestedExceptionCatch() {
      txService.transactionNestedNestedExceptionCatch();
  }
  ```

* 执行结果

  ```shell
  User1 [Hash = 5757, id=29, name=zs, serialVersionUID=1]
  RuntimeException
  ```

> **结论：在外部开启事务后，内部杯NESTED修饰后，会开启一个外部事务的子事务，外部回滚，子事务必须回滚，子事务回滚外部事务不受影响**
