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

  ![Autowired_1](Picture\Spring\@Autowired_1.png)

* @Autowired标注在方法上后者方法参数上

  ![Autowired_2](Picture\Spring\@Autowired_2.png)

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

![Resource_1](Picture\Spring\@Resource_1.png)

* @Resource标注在方法上或者方法参数上

  ![Resource_2](Picture\Spring\@Resource_2.png)

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

### Bean信息定义4种方式

* API的方式
* Xml文件的方式
* properties文件的方式
* 注解的方式

### API的方式

其他几种方式最终都会采用这种方式来定义bean配置信息

Spring容器启动的过程中，会将Bean解析成Spring内部的BeanDefinition结构

最后，Bean工厂会根据这份Bean的定义信息，对Bean进行实例化、初始化等等操作

**BeanDefinition里面包含了Bean定义的各种信息，如：bean对应的class、scope、lazy等信息、dependOn信息、autowireCandidate、primary等信息**

**BeanDefinition接口：Bean定义信息接口**

标识Bean定义信息的接口，里面定义了一些获取Bean定义配置信息的各种方法

```java
public interface BeanDefinition extends AttributeAccessor, BeanMetadataElement {
 
    /**
     * 设置此bean的父bean名称（对应xml中bean元素的parent属性）
     */
    void setParentName(@Nullable String parentName);
 
    /**
     * 返回此bean定义时指定的父bean的名称
     */
    @Nullable
    String getParentName();
 
    /**
     * 指定此bean定义的bean类名(对应xml中bean元素的class属性)
     */
    void setBeanClassName(@Nullable String beanClassName);
 
    /**
     * 返回此bean定义的当前bean类名
     * 注意，如果子定义重写/继承其父类的类名，则这不一定是运行时使用的实际类名。此外，这可能只是调用工厂方法的类，或者在调用方法的工厂bean引用的情况下，它甚至可能是空的。因此，不要认为这是运行时的最终bean类型，而只将其用于单个bean定义级别的解析目的。
     */
    @Nullable
    String getBeanClassName();
 
    /**
     * 设置此bean的生命周期，如：singleton、prototype（对应xml中bean元素的scope属性）
     */
    void setScope(@Nullable String scope);
 
    /**
     * 返回此bean的生命周期，如：singleton、prototype
     */
    @Nullable
    String getScope();
 
    /**
     * 设置是否应延迟初始化此bean（对应xml中bean元素的lazy属性）
     */
    void setLazyInit(boolean lazyInit);
 
    /**
     * 返回是否应延迟初始化此bean，只对单例bean有效
     */
    boolean isLazyInit();
 
    /**
     * 设置此bean依赖于初始化的bean的名称,bean工厂将保证dependsOn指定的bean会在当前bean初始化之前先初始化好
     */
    void setDependsOn(@Nullable String... dependsOn);
 
    /**
     * 返回此bean所依赖的bean名称
     */
    @Nullable
    String[] getDependsOn();
 
    /**
     * 设置此bean是否作为其他bean自动注入时的候选者
     * autowireCandidate
     */
    void setAutowireCandidate(boolean autowireCandidate);
 
    /**
     * 返回此bean是否作为其他bean自动注入时的候选者
     */
    boolean isAutowireCandidate();
 
    /**
     * 设置此bean是否为自动注入的主要候选者
     * primary：是否为主要候选者
     */
    void setPrimary(boolean primary);
 
    /**
     * 返回此bean是否作为自动注入的主要候选者
     */
    boolean isPrimary();
 
    /**
     * 指定要使用的工厂bean（如果有）。这是要对其调用指定工厂方法的bean的名称。
     * factoryBeanName：工厂bean名称
     */
    void setFactoryBeanName(@Nullable String factoryBeanName);
 
    /**
     * 返回工厂bean名称（如果有）（对应xml中bean元素的factory-bean属性）
     */
    @Nullable
    String getFactoryBeanName();
 
    /**
     * 指定工厂方法（如果有）。此方法将使用构造函数参数调用，如果未指定任何参数，则不使用任何参数调用。该方法将在指定的工厂bean（如果有的话）上调用，或者作为本地bean类上的静态方法调用。
     * factoryMethodName：工厂方法名称
     */
    void setFactoryMethodName(@Nullable String factoryMethodName);
 
    /**
     * 返回工厂方法名称（对应xml中bean的factory-method属性）
     */
    @Nullable
    String getFactoryMethodName();
 
    /**
     * 返回此bean的构造函数参数值
     */
    ConstructorArgumentValues getConstructorArgumentValues();
 
    /**
     * 是否有构造器参数值设置信息（对应xml中bean元素的<constructor-arg />子元素）
     */
    default boolean hasConstructorArgumentValues() {
        return !getConstructorArgumentValues().isEmpty();
    }
 
    /**
     * 获取bean定义是配置的属性值设置信息
     */
    MutablePropertyValues getPropertyValues();
 
    /**
     * 这个bean定义中是否有属性设置信息（对应xml中bean元素的<property />子元素）
     */
    default boolean hasPropertyValues() {
        return !getPropertyValues().isEmpty();
    }
 
    /**
     * 设置bean初始化方法名称
     */
    void setInitMethodName(@Nullable String initMethodName);
 
    /**
     * bean初始化方法名称
     */
    @Nullable
    String getInitMethodName();
 
    /**
     * 设置bean销毁方法的名称
     */
    void setDestroyMethodName(@Nullable String destroyMethodName);
 
    /**
     * bean销毁的方法名称
     */
    @Nullable
    String getDestroyMethodName();
 
    /**
     * 设置bean的role信息
     */
    void setRole(int role);
 
    /**
     * bean定义的role信息
     */
    int getRole();
 
    /**
     * 设置bean描述信息
     */
    void setDescription(@Nullable String description);
 
    /**
     * bean描述信息
     */
    @Nullable
    String getDescription();
 
    /**
     * bean类型解析器
     */
    ResolvableType getResolvableType();
 
    /**
     * 是否是单例的bean
     */
    boolean isSingleton();
 
    /**
     * 是否是多列的bean
     */
    boolean isPrototype();
 
    /**
     * 对应xml中bean元素的abstract属性，用来指定是否是抽象的
     */
    boolean isAbstract();
 
    /**
     * 返回此bean定义来自的资源的描述（以便在出现错误时显示上下文）
     */
    @Nullable
    String getResourceDescription();
 
    @Nullable
    BeanDefinition getOriginatingBeanDefinition();
 
}
```

BeanDefinition接口还继承了两个接口——

* AttributeAccessor
* BeanMetadataElement

**AttributeAccessor接口：属性访问接口**

```java
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

BeanDefinition内部实际上使用了LinkedHashMap来实现这个接口中的方法，通常使用这些方法来保存BeanDefinition定义过程中的一些附加信息

**BeanMetadataElement接口**

```java
public interface BeanMetadataElement {
 
    @Nullable
    default Object getSource() {
        return null;
    }
 
}
```

getSource返回BeanDefinition定义的来源，如xml文件定义bean，getSource就返回表示定义bean的xml资源，这个功能可以用来排错

**RootBeanDefinition类：表示根bean定义信息**

通常bean中没有父bean的就是用这种表示

**ChildBeanDefinition类：表示子bean定义信息**

如果需要指定父bean，可以使用ChildBeanDefinition来定义子bean的配置信息，里面有个parentName属性，用来指定父bean的名称

**GenericBeanDefinition类：通用的bean定义信息**

既可以表示没有父bean的bean配置信息，也可以表示有父bean的子bean配置信息，这个类里面也有parentName，用来指定父bean的名称

**ConfigurationClassBeanDefinition类：表示通过配置类中@Bean方法定义bean信息**

可以通过配置类中使用@Bean来标注一些方法，通过这些方法来定义bean，这些方法配置的bean信息最后会转换为ConfigurationClassBeanDefinition类型的对象

**AnnotatedBeanDefinition接口：表示通过注解方式定义的Bean**

```java
AnnotationMetadata getMetadata();
```

用来获取定义这个bean的类上的所有注解信息

**BeanDefinitionBuilder：构建BeanDefinition的工具**

spring中为了方便BeanDefinition，提供工具类，内部提供了很多静态方法，通过这些方法可以非常方便的组装BeanDefinition对象

### 案例

> 案例1
>
> 组装一个简单的bean

* Car

  ```java
  public class Car {
      private String name;
  
      public String getName() {
          return name;
      }
  
      public void setName(String name) {
          this.name = name;
      }
  
      @Override
      public String toString() {
          return "Car{" +
                  "name='" + name + '\'' +
                  '}';
      }
  }
  ```

* Test

  ```java
  @Test
  public void test1() {
      BeanDefinitionBuilder builder = BeanDefinitionBuilder.rootBeanDefinition(Car.class.getName());
      AbstractBeanDefinition beanDefinition = builder.getBeanDefinition();
      System.out.println(beanDefinition);
  }
  ```

* 执行结果

  ```shell
  Root bean: class [top.mnsx.spring.study.domain.Car]; scope=; abstract=false; lazyInit=null; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=null; factoryMethodName=null; initMethodName=null; destroyMethodName=null
  ```

> 案例2
>
> 组装一个有属性的bean

* Test

  ```java
  @Test
  public void test2() {
      BeanDefinitionBuilder builder = BeanDefinitionBuilder.rootBeanDefinition(Car.class.getName());
      builder.addPropertyValue("name", "bmw");
      AbstractBeanDefinition beanDefinition = builder.getBeanDefinition();
      System.out.println(beanDefinition);
  
      DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
      beanFactory.registerBeanDefinition("car", beanDefinition);
      Car car = beanFactory.getBean("car", Car.class);
      System.out.println(car);
  }
  ```

* 执行结果

  ```shell
  Root bean: class [top.mnsx.spring.study.domain.Car]; scope=; abstract=false; lazyInit=null; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=null; factoryMethodName=null; initMethodName=null; destroyMethodName=null
  Car{name='bmw'}
  ```

> 案例3
>
> 组装一个有依赖关系的Bean

* User

  ```java
  public class User {
      private String name;
  
      private Car car;
  
      public String getName() {
          return name;
      }
  
      public void setName(String name) {
          this.name = name;
      }
  
      public Car getCar() {
          return car;
      }
  
      public void setCar(Car car) {
          this.car = car;
      }
  
      @Override
      public String toString() {
          return "User{" +
                  "name='" + name + '\'' +
                  ", car=" + car +
                  '}';
      }
  }
  ```

* Test

  ```java
  @Test
  public void test3() {
      BeanDefinitionBuilder carBuilder = BeanDefinitionBuilder.rootBeanDefinition(Car.class.getName());
      AbstractBeanDefinition carBeanDefinition = carBuilder.getBeanDefinition();
      AbstractBeanDefinition userBeanDefinition = BeanDefinitionBuilder.rootBeanDefinition(User.class.getName())
          .addPropertyValue("name", "Mnsx")
          .addPropertyReference("car", "car")
          .getBeanDefinition();
      DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
      beanFactory.registerBeanDefinition("car", carBeanDefinition);
      beanFactory.registerBeanDefinition("user", userBeanDefinition);
      System.out.println(beanFactory.getBean("car"));
      System.out.println(beanFactory.getBean("user"));
  }
  ```

* 执行结果

  ```shell
  Car{name='null'}
  User{name='Mnsx', car=Car{name='null'}}
  ```

> 案例4
>
> 含有父子关系的Bean

* Test

  ```java
  @Test
  public void test4() {
      BeanDefinition carBeanDefinition1 = BeanDefinitionBuilder.genericBeanDefinition(Car.class)
          .addPropertyValue("name", "bmw")
          .getBeanDefinition();
  
      BeanDefinition carBeanDefinition2 = BeanDefinitionBuilder.genericBeanDefinition()
          .setParentName("car1")
          .getBeanDefinition();
  
      DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
      factory.registerBeanDefinition("car1", carBeanDefinition1);
      factory.registerBeanDefinition("car2", carBeanDefinition2);
      System.out.println(factory.getBean("car1"));
      System.out.println(factory.getBean("car2"));
  }
  ```

* 执行结果

  ```shell
  Car{name='bmw'}
  Car{name='bmw'}
  ```

> 案例5
>
> 通过API设置属性

* CompositeObj

  ```java
  public class CompositeObj {
      private List<String> stringList;
      private Set<String> stringSet;
      private Map<String, String> stringMap;
  
      public List<String> getStringList() {
          return stringList;
      }
  
      public void setStringList(List<String> stringList) {
          this.stringList = stringList;
      }
  
      public Set<String> getStringSet() {
          return stringSet;
      }
  
      public void setStringSet(Set<String> stringSet) {
          this.stringSet = stringSet;
      }
  
      public Map<String, String> getStringMap() {
          return stringMap;
      }
  
      public void setStringMap(Map<String, String> stringMap) {
          this.stringMap = stringMap;
      }
  
      @Override
      public String toString() {
          return "CompositeObj{" +
                  "stringList=" + stringList +
                  ", stringSet=" + stringSet +
                  ", stringMap=" + stringMap +
                  '}';
      }
  }
  ```

* Test

  ```java
  @Test
  public void test5() {
      ManagedList<String> stringList = new ManagedList<>();
      stringList.addAll(Arrays.asList("Mnsx", "mnsx", "MNSX"));
  
      ManagedSet<String> stringSet = new ManagedSet<>();
      stringSet.addAll(Arrays.asList("Mnsx", "mnsx", "MNSX"));
  
      ManagedMap<String, String> stringMap = new ManagedMap<>();
      stringMap.put("1", "Mnsx");
      stringMap.put("2", "mnsx");
      stringMap.put("3", "MNSX");
  
      GenericBeanDefinition beanDefinition = new GenericBeanDefinition();
      beanDefinition.setBeanClassName(CompositeObj.class.getName());
      beanDefinition.getPropertyValues().add("stringList", stringList)
          .add("stringSet", stringSet)
          .add("stringMap", stringMap);
  
      DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
      factory.registerBeanDefinition("compositeObj", beanDefinition);
      System.out.println(factory.getBean("compositeObj"));
  }
  ```

* 执行结果

  ```shell
  CompositeObj{stringList=[Mnsx, mnsx, MNSX], stringSet=[Mnsx, mnsx, MNSX], stringMap={1=Mnsx, 2=mnsx, 3=MNSX}}
  ```

  * RuntimeBeanReference：用来表示bean引用类型
  * ManagedList：属性是List类型继承了ArrayList
  * ManagedSet：属性是Set类型继承了LinkedHashSet
  * ManagedMap：属性是Map类型继承了LinkedHashMap

### XML的方式

xml中的bean配置信息会被解析器解析为BeanDefinition对象

### Properties的方式

将bean定义信息放在properties文件中，然后通过解析器将配置信息解析为BeanDefinition对象

### 注解的方式

1. 类上标注@Component注解来定义一个bean
2. 配置类中使用@Bean注解来定义bean

## Bean元信息的解析阶段

Bean元信息的解析就是将各种方式定义的bean配置信息解析为BeanDefinition对象

### Bean元信息的解析主要有3中方式

1. xml文件定义bean的解析
2. properties文件定义的解析
3. 注解方式定义bean的解析

### XML方式解析：XmlBeanDefinitionReader

Spring中提供一个类XmlBeanDefinitionReader，将Xml中定义的bean解析为BeanDefinition对象

**主要步骤**

```java
DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(factory);
String location = "...";
int countBean = reader.loadBeanDefinitions(location);
```

### properties方式解析：PropertiesBeanDefinitionReader

与XMl类似，将XmlBeanDefinition换成PropertiesBeanDefinitionReader

## 注解方式解析：PropertiesBeanDefinitionReader

注解方式定义的bean，需要使用PropertiesBeanDefinitionReader这个类进行解析

* Service1

  ```java
  @Primary
  @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
  @Lazy
  public class Service1 {
  }
  ```

* Test

  ```java
  @Test
  public void test6() {
      DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
      AnnotatedBeanDefinitionReader reader = new AnnotatedBeanDefinitionReader(factory);
      reader.register(Service1.class);
      System.out.println(factory.getBean("service1"));
  }
  ```

## Spring Bean注册阶段

bean注册阶段需要用到BeanDefinitionRegistry

**Bean注册接口：BeanDefinitionRegistry**

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

**别名注册接口：AliasRegistry**

```java
public interface AliasRegistry {
 
    /**
     * 给name指定别名alias
     */
    void registerAlias(String name, String alias);
 
    /**
     * 从此注册表中删除指定的别名
     */
    void removeAlias(String alias);
 
    /**
     * 判断name是否作为别名已经被使用了
     */
    boolean isAlias(String name);
 
    /**
     * 返回name对应的所有别名
     */
    String[] getAliases(String name);
 
}
```

**BeanDefinitionRegistry唯一实现：DefaultListableBeanFactory**

```java
org.springframework.beans.factory.support.DefaultListableBeanFactory
```

## BeanDefinition合并阶段

```java
org.springframework.beans.factory.support.AbstractBeanFactory#getMergedBeanDefinition
```

bean定义可能存在多级父子关系，合并的时候进行递归合并，最终得到一个包含完整信息的RootBeanDefinition

* Test

  ```java
  @Test
  public void test7() {
      BeanDefinition carBeanDefinition1 = BeanDefinitionBuilder.genericBeanDefinition(Car.class)
          .addPropertyValue("name", "bmw")
          .getBeanDefinition();
  
      BeanDefinition carBeanDefinition2 = BeanDefinitionBuilder.genericBeanDefinition()
          .setParentName("car1")
          .getBeanDefinition();
  
      DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
      factory.registerBeanDefinition("car1", carBeanDefinition1);
      factory.registerBeanDefinition("car2", carBeanDefinition2);
  
      for (String beanName : factory.getBeanDefinitionNames()) {
          BeanDefinition beanDefinition = factory.getBeanDefinition(beanName);
          BeanDefinition mergedBeanDefinition = factory.getMergedBeanDefinition(beanName);
          System.out.println(beanName);
          System.out.println(beanDefinition);
          System.out.println(mergedBeanDefinition);
      }
  }
  ```

* 执行结果

  ```shell
  car1
  Generic bean: class [top.mnsx.spring.study.domain.Car]; scope=; abstract=false; lazyInit=null; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=null; factoryMethodName=null; initMethodName=null; destroyMethodName=null
  Root bean: class [top.mnsx.spring.study.domain.Car]; scope=singleton; abstract=false; lazyInit=null; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=null; factoryMethodName=null; initMethodName=null; destroyMethodName=null
  car2
  Generic bean with parent 'car1': class [null]; scope=; abstract=false; lazyInit=null; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=null; factoryMethodName=null; initMethodName=null; destroyMethodName=null
  Root bean: class [top.mnsx.spring.study.domain.Car]; scope=singleton; abstract=false; lazyInit=null; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=null; factoryMethodName=null; initMethodName=null; destroyMethodName=null
  ```

从执行结果中看出，合并之前，BeanDefinition是不完整的，car2是null，属性信息不完整，但是合并之后这些信息就完整了

**合并之前是GenericBeanDefinition类型的，合并之后是RootBeanDefinition类型的**

**后面阶段使用的是合并之后的RootBeanDefinition**

## Bean Class加载阶段

分为两个阶段

* Bean实例化前操作
* Bean实例化操作

### Bean实例化前操作

```java
private final List<BeanPostProcessor> beanPostProcessors = new CopyOnWriteArrayList<>();
```

**BeanPostProcessor是一个接口，还有很多子接口，spring在bean生命周期的不同阶段，会调用上面列表中的BeanPostProcessor中的一些方法，来对生命周期进行扩展**

Bean实例化之前会调用

```java
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

**这段代码可以自己在这个地方直接去创建一个对象作为Bean实例，而跳过Spring内部实例化bean的过程**

这段代码将轮询beanPostProcessors列表，如果类型是InstantiationAwareBeanPostProcessor，尝试调用InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation获取bean的实例对象，如果能够获取，那么将返回值作为当前bean实例，spring自带的实例化bean的过程就被跳过了

```java
default Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
    return null;
}
```

### Bean实例化操作

这个过程会通过反射调用Bean的构造器来创建Bean的实例

**具体使用什么构造器，Spring为开发者提供了一个接口，允许开发者自己来判断使用那个构造器**

```java
for (BeanPostProcessor bp : getBeanPostProcessors()) {
    if (bp instanceof SmartInstantiationAwareBeanPostProcessor) {
        SmartInstantiationAwareBeanPostProcessor ibp = (SmartInstantiationAwareBeanPostProcessor) bp;
        Constructor<?>[] ctors = ibp.determineCandidateConstructors(beanClass, beanName);
        if (ctors != null) {
            return ctors;
        }
    }
}
```

调用SmartInstantiationAwareBeanPostProcessor接口的determineCandidateConstructors方法，这个方法会返回候选的构造器列表，也可以返回空

```java
@Nullable
default Constructor<?>[] determineCandidateConstructors(Class<?> beanClass, String beanName)
throws BeansException {
    return null;
}
```

```java
org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor
```

上述实现类可以将@Autowired标注的方法作为候选构造器返回

**案例：自定义注解，构造器被注解注释，Spring自动选择这个构造器**

* MyAutowired

  ```java
  @Target(ElementType.CONSTRUCTOR)
  @Retention(RetentionPolicy.RUNTIME)
  @Documented
  public @interface MyAutowired {
  }
  ```

* Person

  ```java
  public class Person {
      private String name;
      private Integer age;
  
      public Person() {
          System.out.println("调用Person()");
      }
      
      public Person(String name, Integer age) {
          System.out.println("调用Person(String name, Integer age)");
          this.name = name;
          this.age = age;
      }
      
      @MyAutowired
      public Person(String name) {
          System.out.println("调用Person(String name)");
          this.name = name;
      }
  
      @Override
      public String toString() {
          return "Person{" +
                  "name='" + name + '\'' +
                  ", age=" + age +
                  '}';
      }
  }
  ```

* MySmartInstantiationAwareBeanPostProcessor

  ```java
  public class MySmartInstantiationAwareBeanPostProcessor implements SmartInstantiationAwareBeanPostProcessor {
      @Override
      public Constructor<?>[] determineCandidateConstructors(Class<?> beanClass, String beanName) throws BeansException {
          System.out.println(beanClass);
          Constructor<?>[] declaredConstructors = beanClass.getDeclaredConstructors();
          if (declaredConstructors != null) {
              List<Constructor<?>> collect = Arrays.stream(declaredConstructors)
                      .filter(constructor -> constructor.isAnnotationPresent(MyAutowired.class))
                      .collect(Collectors.toList());
              Constructor[] constructors = collect.toArray(new Constructor[collect.size()]);
              return constructors.length != 0 ? constructors : null;
          } else {
              return null;
          }
      }
  }
  ```

* Test

  ```java
  @Test
  public void test8() {
      DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
      factory.addBeanPostProcessor(new MySmartInstantiationAwareBeanPostProcessor());
      factory.registerBeanDefinition("name", BeanDefinitionBuilder
                                     .genericBeanDefinition(String.class)
                                     .addConstructorArgValue("mnsx")
                                     .getBeanDefinition());
      factory.registerBeanDefinition("age", BeanDefinitionBuilder
                                     .genericBeanDefinition(Integer.class)
                                     .addConstructorArgValue(20)
                                     .getBeanDefinition());
      factory.registerBeanDefinition("person",
                                     BeanDefinitionBuilder
                                     .genericBeanDefinition(Person.class)
                                     .getBeanDefinition());
      Person person = factory.getBean("person", Person.class);
      System.out.println(person);
  }
  ```

* 执行结果

  ```shell
  class top.mnsx.spring.study.domain.Person
  class java.lang.String
  调用Person(String name)
  Person{name='mnsx', age=null}
  ```

## 合并后的BeanDefinition处理

```java
protected void applyMergedBeanDefinitionPostProcessors(RootBeanDefinition mbd, Class<?> beanType, String beanName) {
    for (BeanPostProcessor bp : getBeanPostProcessors()) {
        if (bp instanceof MergedBeanDefinitionPostProcessor) {
            MergedBeanDefinitionPostProcessor bdp = (MergedBeanDefinitionPostProcessor) bp;
            bdp.postProcessMergedBeanDefinition(mbd, beanType, beanName);
        }
    }
}
```

会调用MergedBeanDefinitionPostProcessor接口的postProcessMergedBeanDefinition方法

```java
void postProcessMergedBeanDefinition(RootBeanDefinition beanDefinition, Class<?> beanType, String beanName);
```

第一个参数表示beanDefinition，表示合并之后的RootBeanDefinition，可以在这个方法内部对合并之后的BeanDefinition进行再次处理

```java
org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor
在 postProcessMergedBeanDefinition 方法中对 @Autowired、@Value 标注的方法、字段进行缓存
 
org.springframework.context.annotation.CommonAnnotationBeanPostProcessor
在 postProcessMergedBeanDefinition 方法中对 @Resource 标注的字段、@Resource 标注的方法、 @PostConstruct 标注的字段、 @PreDestroy标注的方法进行缓存
```

## Bean属性设置阶段

属性设置阶段分为3个小阶段

* 实例后阶段
* Bean属性赋值前阶段
* Bean属性赋值阶段

### 实例化后阶段

会调用InstantiationAwareBeanPostProcessor接口的postProcessAfterInstantiation这个方法

```java
for (BeanPostProcessor bp : getBeanPostProcessors()) {
    if (bp instanceof InstantiationAwareBeanPostProcessor) {
        InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
        if (!ibp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
            return;
        }
    }
}
```

> postProcessAfterInstantiation方法返回false的时候，后续的Bean属性赋值前处理，Bean属性赋值都会被跳过

```java
default boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
    return true;
}
```

### Bean属性赋值前阶段

这个阶段调用InstantiationAwareBeanPostProcessor接口的postProcessProperties方法

```java
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
```

> 如果InstantiationAwareBeanPostProcessor中的postProcessProperties和postProcessPropertyValues都返回空的时候，表示这个Bean不需要设置属性，直接返回，进入下一个阶段

```java
@Nullable
default PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName)
    throws BeansException {
 
    return null;
}
```

> PropertyValues中保存了bean实例对象中所有属性值的设置，所以可以在这个方法中对PropertyValues值进行修改

```java
org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor
在 postProcessMergedBeanDefinition 方法中对 @Autowired、@Value 标注的方法、字段进行缓存
 
org.springframework.context.annotation.CommonAnnotationBeanPostProcessor
在 postProcessMergedBeanDefinition 方法中对 @Resource 标注的字段、@Resource 标注的方法、 @PostConstruct 标注的字段、 @PreDestroy标注的方法进行缓存
```

**案例**

```java
@Test
    public void test9() {
        DefaultListableBeanFactory factory = new DefaultListableBeanFactory();

        factory.addBeanPostProcessor(new InstantiationAwareBeanPostProcessor() {
            @Override
            public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) throws BeansException {
                if ("user1".equals(beanName)) {
                    if (pvs == null) {
                        pvs = new MutablePropertyValues();
                    }
                    if (pvs instanceof MutablePropertyValues) {
                        MutablePropertyValues mpvs = (MutablePropertyValues) pvs;
                        mpvs.add("name", "mnsx");
                        mpvs.add("age", 18);
                    }
                }
                return null;
            }
        });

        factory.registerBeanDefinition("user1",
                BeanDefinitionBuilder
                        .genericBeanDefinition(User.class)
                        .getBeanDefinition());

        factory.registerBeanDefinition("user2",
                BeanDefinitionBuilder
                        .genericBeanDefinition(User.class)
                        .addPropertyValue("name", "Mnsx")
                        .addPropertyValue("age", 20)
                        .getBeanDefinition());

        for (String beanName : factory.getBeanDefinitionNames()) {
            System.out.println(beanName + ":" + factory.getBeanDefinition(beanName));
        }
    }
}
```

### Bean属性赋值阶段

循环处理PropertyValues中的属性值信息，通过反射调用set方法将属性的值设置到Bean中

## Bean初始化阶段

这个阶段分为5个阶段

* BeanAware接口回调
* Bean初始化前操作
* Bean初始化操作
* Bean初始化后操作
* Bean初始化完成操作

### BeanAware接口回调

```java
private void invokeAwareMethods(final String beanName, final Object bean) {
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

如果Bean实例实现了下面的接口，会按照下面顺序依次进行调用

```shell
BeanNameAware：将bean的名称注入进去
BeanClassLoaderAware：将BeanClassLoader注入进去
BeanFactoryAware：将BeanFactory注入进去
```

**案例**

* AwareBean

  ```java
  public class AwareBean implements BeanNameAware, BeanClassLoaderAware, BeanFactoryAware {
      @Override
      public void setBeanClassLoader(ClassLoader classLoader) {
          System.out.println("setBeanClassLoader: " + classLoader);
      }
  
      @Override
      public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
          System.out.println("setBeanFactory: " + beanFactory);
      }
  
      @Override
      public void setBeanName(String name) {
          System.out.println("setBeanName: " + name);
      }
  }
  ```

* Test

  ```java
  @Test
  public void test10() {
      DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
      factory.registerBeanDefinition("awareBean", BeanDefinitionBuilder.genericBeanDefinition(AwareBean.class).getBeanDefinition());
      factory.getBean("awareBean");
  }
  ```

### Bean初始化前操作

```java
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

会调用BeanPostProcessor的postProcessBeforeInitialization方法，若返回null，当前方法将结束

这个接口有两个实现类

```java
org.springframework.context.support.ApplicationContextAwareProcessor
org.springframework.context.annotation.CommonAnnotationBeanPostProcessor
```

ApplicationContextAwareProcessor注入6个Aware接口对象

如果Bean实现了下面的接口，在ApplicationContextAwareProcessor#postProcessBeforeInitialization中依次调用下面接口中的方法，将Aware前缀对应的对象注入到bean实例中

```java
EnvironmentAware：注入Environment对象
EmbeddedValueResolverAware：注入EmbeddedValueResolver对象
ResourceLoaderAware：注入ResourceLoader对象
ApplicationEventPublisherAware：注入ApplicationEventPublisher对象
MessageSourceAware：注入MessageSource对象
ApplicationContextAware：注入ApplicationContext对象
```

从名称上看出，这个类只能在ApplicationContext环境中使用

CommonAnnotationBeanPostProcessor#postProcessBeforeInitialization中会调用bean中所有标注@PostConstruct注解的方法

**案例**

* Bean1

  ```java
  public class Bean1 {
      @PostConstruct
      public void postConstruct1() {
          System.out.println("postConstruct1()");
      }
  }
  ```

* Test

  ```java
  @Test
  public void test11() {
      AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
      context.register(Bean1.class);
      context.refresh();
  }
  ```

### Bean初始化阶段

包含两个步骤

1. 调用InitializationBean接口的afterPropertiesSet方法
2. 调用定义bean的时候指定的初始化方法

```java
public interface InitializingBean {
 
    void afterPropertiesSet() throws Exception;
 
}
```

**只有当Bean实现了这个接口，才会在这个阶段被调用**

设置Bean初始化方法的三种方式

* XML

  ```xml
  <bean init-method="bean中方法名称"/>
  ```

* @Bean

  ```java
  @Bean(initMethod = "初始化的方法")
  ```

* API

  ```java
  this.beanDefinition.setInitMethodName(methodName);
  ```

最后初始化方法最终会赋值给这个字段`org.springframework.beans.factory.support.AbstractBeanDefinition#initMethodName`

### Bean初始化后阶段

```java
@Override
public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName)
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

调用BeanPostProcessor接口的postProcessAfterInitialization方法，返回null的时候，会中断上面的操作

## 所有单例bean初始化完成后阶段

所有单例bean实例化完成后，Spring会回调下面的接口

```java
public interface SmartInitializingSingleton {
    void afterSingletonsInstantiated();
}
```

调用逻辑

```java
/**
 * 确保所有非lazy的单例都被实例化，同时考虑到FactoryBeans。如果需要，通常在工厂设置结束时调用。
 */
org.springframework.beans.factory.support.DefaultListableBeanFactory#preInstantiateSingletons
```

## Bean实用阶段

...

## Bean销毁阶段

调用Bean销毁的几种方式

1. 调用`org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#destroyBean`
2. 调用`org.springframework.beans.factory.config.ConfigurableBeanFactory#destroySingletons`
3. 调用ApplicationContext中的close方法

Bean销毁阶段会依次执行

1. 轮询beanPostProcessor列表，如果是DestructionAwareBeanPostProcessor这种类型，会调用其内部的postProcessBeforeDestruction方法
2. 如果Bean实现了org.springframework.beans.factory.DisposableBean接口，会调用这个接口中的destory方法
3. 调用bean自定义的销毁方法

**DestructionAwareBeanPostProcessor接口**

```java
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

```java
org.springframework.context.annotation.CommonAnnotationBeanPostProcessor
```

自定义销毁方法的方式和自定义创建方式相同

### 案例

> 案例1
>
> 自定义DestructionAwareBeanPostProcessor

* ServiceA

  ```java
  public class ServiceA {
      public ServiceA() {
          System.out.println("create " + this.getClass());
      }
  }
  ```

* MyDestructionAwareBeanPostProcessor

  ```java
  public class MyDestructionAwareBeanPostProcessor implements DestructionAwareBeanPostProcessor {
      @Override
      public void postProcessBeforeDestruction(Object bean, String beanName) throws BeansException {
          System.out.println("destroy " + beanName);
      }
  }
  ```

* Test

  ```java
  @Test
  public void test12() {
      DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
      factory.addBeanPostProcessor(new MyDestructionAwareBeanPostProcessor());
      factory.registerBeanDefinition("serviceA", BeanDefinitionBuilder.genericBeanDefinition(ServiceA.class)
                                     .getBeanDefinition());
      // 触发所有单例Bean的初始化
      factory.preInstantiateSingletons();
      factory.destroySingletons();;
  }
  ```

> 案例2
>
> 触发@PreDestroy标注的方法被调用

* ServiceB

  ```java
  public class ServiceB {
      public ServiceB() {
          System.out.println("create " + this.getClass());
      }
      
      @PreDestroy
      public void preDestroy() {
          System.out.println("preDestroy()");
      }
  }
  ```

* Test

  ```java
  @Test
  public void test13() {
      DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
      factory.addBeanPostProcessor(new MyDestructionAwareBeanPostProcessor());
      factory.addBeanPostProcessor(new CommonAnnotationBeanPostProcessor());
      factory.registerBeanDefinition("serviceB", BeanDefinitionBuilder.genericBeanDefinition(ServiceB.class)
                                     .getBeanDefinition());
      factory.preInstantiateSingletons();
      factory.destroySingletons();
  }
  ```

> 案例3
>
> 销毁阶段执行顺序

**ApplicationContext内部已经将Spring内部一些常见的BeanPostProcessor自动装配到beanPostProcessor列表中**

```shell
1.org.springframework.context.annotation.CommonAnnotationBeanPostProcessor
  用来处理@Resource、@PostConstruct、@PreDestroy的
2.org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor
  用来处理@Autowired、@Value注解
3.org.springframework.context.support.ApplicationContextAwareProcessor
  用来回调Bean实现的各种Aware接口
```

**结论**

1. @PreDestroy标注的所有方法
2. DisposableBean接口中的destroy方法
3. 自定的销毁方法

### AbstractApplicationContext类

**BeanFactory接口**

Bean工厂的顶层接口

**DefaultListableBeanFactory类**

实现了BeanFactory接口，可以说是BeanFactory接口真正的唯一实现，内部真正实现了bean生命周期中的所有方法

其他一些类都是依赖于DefaultListableBeanFactory类

**其他类**

AnnotationConfigApplicationContext、ClassPathXmlApplicationContext、FileSystemXmlApplicationContext

主要内部功能是依赖父类AbstractApplicationContext实现

**AbstractApplicationContext类**

refresh方法会调用下列方法

```java
public abstract ConfigurableListableBeanFactory getBeanFactory() throws IllegalStateException;
protected void registerBeanPostProcessors(ConfigurableListableBeanFactory beanFactory)
```

* getBeanFactory()

  返回当前应用上下文的ConfigurationListableBeanFactory，这个接口有一个唯一实现类DefaultListableBeanFactory

* registerBeanPostProcessor()

  这个方法就是向ConfigurableListableBeanFactory中注册BeanPostProcessor，内容会从spring容器中获取所有类习惯的BeanPostProcessor，将其添加到DefaultListableBeanFactory#beanPostProcessors列表中

  ```java
  protected void registerBeanPostProcessors(ConfigurableListableBeanFactory beanFactory) {
      PostProcessorRegistrationDelegate.registerBeanPostProcessors(beanFactory, this);
  }
  ```

  会将请求转发给PostProcessorRegistrationDelegate#registerBeanPostProcessor

  ```java
  List<BeanPostProcessor> priorityOrderedPostProcessors = new ArrayList<>();
  List<BeanPostProcessor> orderedPostProcessors
  List<BeanPostProcessor> nonOrderedPostProcessors;
  List<BeanPostProcessor> internalPostProcessors = new ArrayList<>();
  ```

## Bean生命周期流程

![Bean生命周期](Picture\Spring\Bean生命周期.png)

# 父子容器详解

`childBeanFactory.setParentBeanFactory(parentBeanFactory)`

`childContext.setParent(parenContext)`

## 父子容器特点

1. 父容器和子容器是相互隔离的，内部可以存在同名的Bean
2. 子容器可以访问父容器中的Bean，而父容器不可以访问子容器中的Bean
3. 调用子容器的getBean方法获取Bean时，子容器会沿着当前容器向上面的容器查找，直到找到对应的Bean为止
4. 子容器中可以通过任何注入方式注入父容器中的Bean，而父容器中是无法注入子容器中的Bean

## 父容器使用注意点

BeanFactory支持按照容器嵌套结构查询，但是ListableBeanFactory不支持按照容器嵌套结构查询

getBeanNamesForType是ListableBeanFactory中定义的，所以不支持按照容器嵌套结构查询

**解决ListableBeanFactory不支持按照容器嵌套结构查询的问题**

使用`org.springframework.beans.factory.BeanFactoryUtils`工具类，其中名称包含了Ancestors都是支持嵌套层次查询的

## SpringMVC中父子容器相关

* SpringMVC只是用一个容器是可以启动的

* 使用父子容器的原因是：

  避免在Service层中注入Controller层的bean，导致结构混乱

  父容器和子容器的需求也是不一样的，相互隔离开，父子容器的加载速度也能提升

# @Value

## @Value的用法

将配置信息统一放在配置文件中，方便上线时修改

Spring中提供@Value注解，通过@Value("${配置文件中的key}")来引用指定的key对应的value

> 使用@PropertySource来指定外部的自定义Properties配置文件

## @Value数据来源

```java
org.springframework.core.env.PropertySource
```

可以理解为一个配置源，里面包含了key->value的配置信息，可以通过其中提供的方式获取key对应的value信息

```java
public abstract Object getProperty(String name);
```

配置源相关的一个重要接口

```java
org.springframework.core.env.Environment
```

其中的重要方法
```java
String resolvePlaceholders(String text);
MutablePropertySources getPropertySources();
```

resolvePlaceholders用来解析${text}的，@Value注解就是调用这个方法解析

getPropertySource返回MutablePropertySources对象

```java
public class MutablePropertySources implements PropertySources {
 
    private final List<PropertySource<?>> propertySourceList = new CopyOnWriteArrayList<>();
 
}
```

内部包含了一个propertySourceList列表

**想要改变@Value数据源，只需要将配置信息包装为PropertySource对象，放到Environment中MutablePropertySource内部**

**案例**

* MailConfig

  ```java
  @Component
  public class MailConfig {
      @Value("${mail.host}")
      public String host;
  
      @Value("${mail.username}")
      public String username;
  
      public String getHost() {
          return host;
      }
  
      public void setHost(String host) {
          this.host = host;
      }
  
      public String getUsername() {
          return username;
      }
  
      public void setUsername(String username) {
          this.username = username;
      }
  
      @Override
      public String toString() {
          return "MailConfig{" +
                  "host='" + host + '\'' +
                  ", username='" + username + '\'' +
                  '}';
      }
  }
  ```

* DbUtils

  ```java
  public class DbUtils {
      public static Map<String, Object> getMailInfoFromDb() {
          Map<String, Object> result = new HashMap<>();
          result.put("mail.host", "gmail.com");
          result.put("mail.username", "mnsx");
          return result;
      }
  }
  ```

* MainConfig1

  ```java
  @Configuration
  @ComponentScan
  public class MainConfig1 {
  }
  ```

* Test

  ```java
  @SpringBootTest
  public class SpringStudyTest {
      @Test
      public void test1() {
          AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
          Map<String, Object> map = DbUtils.getMailInfoFromDb();
          MapPropertySource source = new MapPropertySource("mail", map);
          context.getEnvironment().getPropertySources().addFirst(source);
  
          context.register(MainConfig1.class);
          context.refresh();
          MailConfig mailConfig = context.getBean(MailConfig.class);
          System.out.println(mailConfig);
      }
  }
  ```

* 执行结果

  ```shell
  MailConfig{host='gmail.com', username='mnsx'}
  ```

## @Value动态刷新

> @Scope中一个参数`ScopedProxyMode proxyMode() default ScopedProxyMode.DEFAULT;`
>
> ```java
> public enum ScopedProxyMode {
>  DEFAULT,
>  NO,
>  INTERFACES,
>  TARGET_CLASS;
> }
> ```
>
> 当@Scope中proxyMode为TARGET_CLASS时，会给当前创建的Bean通过CGLIB生成一个代理对象，通过这个代理对象访问bean
>
> **使用案例**
>
> * User
>
>   ```java
>   @Component
> @Scope(value = MyScopeBean.SCOPE_MY, proxyMode = ScopedProxyMode.TARGET_CLASS)
>   public class User {
>     private String name;
>   
>       public User() {
>           this.name = UUID.randomUUID().toString();
>           System.out.println("构造函数创建User对象: " + this.getName());
>       }
>
>       public String getName() {
>           return name;
>       }
>   
>     public void setName(String name) {
>           this.name = name;
>       }
> }
>   ```
>
> * MyScopeBean
>
>   ```java
>   public class MyScopeBean implements Scope {
>   
>       public static final String SCOPE_MY = "my";
>   
>       @Override
>       public Object get(String name, ObjectFactory<?> objectFactory) {
>           System.out.println("MyScopeBean get:" + name);
>           User user = (User) objectFactory.getObject();
>           user.setName("hhh");
>           return user;
>       }
>   
>       @Override
>       public Object remove(String name) {
>           return null;
>       }
>   
>       @Override
>       public void registerDestructionCallback(String name, Runnable callback) {
>   
>       }
>   
>       @Override
>       public Object resolveContextualObject(String key) {
>           return null;
>       }
>   
>       @Override
>       public String getConversationId() {
>           return null;
>       }
>   }
>   ```
>   
> * ScopeConfig
>
>   ```java
>   @Configuration
>   @ComponentScan("top.mnsx.spring.study")
>   public class ScopeConfig {
>       @Bean
>       public CustomScopeConfigurer customScopeConfigurer() {
>           CustomScopeConfigurer customScopeConfigurer = new CustomScopeConfigurer();
>           HashMap<String, Object> map = new HashMap<>();
>           map.put(MyScopeBean.SCOPE_MY, new MyScopeBean());
>           customScopeConfigurer.setScopes(map);
>           return customScopeConfigurer;
>       }
>   }
>   ```
>
> * Test
>
>   ```java
>   @Test
>   public void test2() {
>       AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
>       context.register(ScopeConfig.class);
>       context.refresh();
>   
>       User user = context.getBean(User.class);
>       System.out.println("创建Bean: " + user);
>   
>       for (int i = 0; i < 3; ++i) {
>           System.out.println(i + ">>>" + user.getName());
>       }
>   }
>   ```
>
> * 执行结果
>
>   ```shell
>   MyScopeBean get:scopedTarget.user
>   构造函数创建User对象: 02f4df20-178d-4149-82fe-9b549ad4de94
>   创建Bean: top.mnsx.spring.study.domain.User@49c8f6e8
>   MyScopeBean get:scopedTarget.user
>   构造函数创建User对象: 6fadde75-8226-4a18-8675-38f7d0e3cd15
>   0>>>hhh
>   MyScopeBean get:scopedTarget.user
>   构造函数创建User对象: adc87d0f-b95a-4cde-bc70-2fbfea23e30f
>   1>>>hhh
>   MyScopeBean get:scopedTarget.user
>   构造函数创建User对象: 88c4121a-383d-43dd-afe0-9b9fe9b54e42
>   2>>>hhh
>   ```
>
> * 结论
>
>   当自定义的Scope中proxyMode=ScopedProxyMode.TARGET_CLASS的时候，会给这个bean创建一个代理对象，调用代理对象的任何方法，都会调用这个自定的作用域实现类中的get方法来重新获取这个bean对象

动态刷新@Value，当配置修改的时候，调用这些bean的任何方法的时候，就会让Spring重启初始化这个Bean

* RefreshScope

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

* TestConfig

  ```java
  @Component
  @RefreshScope
  public class TestConfig {
      @Value("${test.name}")
      private String name;
  
      public String getName() {
          return name;
      }
  
      public void setName(String name) {
          this.name = name;
      }
  
      @Override
      public String toString() {
          return "TestConfig{" +
                  "name='" + name + '\'' +
                  '}';
      }
  }
  ```

* TestService

  ```java
  @Component
  public class TestService {
      @Autowired
      private TestConfig testConfig;
  
      @Override
      public String toString() {
          return "TestService{" +
                  "testConfig=" + testConfig +
                  '}';
      }
  }
  ```

* DbUtil

  ```java
  public class DbUtil {
      public static Map<String, Object> getInfoFromDb() {
          HashMap<String, Object> result = new HashMap<>();
          result.put("test.name", UUID.randomUUID());
          return result;
      }
  }
  ```

* RefreshConfigUtil

  ```java
   public class RefreshConfigUtil {
      public static void updateDbConfig(AbstractApplicationContext context) {
          refreshPropertySouce(context);
          BeanRefreshScope.clean();
      }
      
      public static void refreshPropertySouce(AbstractApplicationContext context) {
          Map<String, Object> infoFromDb = DbUtil.getInfoFromDb();
          MapPropertySource mapPropertySource = new MapPropertySource("test", infoFromDb);
          context.getEnvironment().getPropertySources().addFirst(mapPropertySource);
      }
  }
  ```

* Test

  ```java
  @Test
  public void test4() {
      AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(); context.getBeanFactory().registerScope(BeanRefreshScope.SCOPE_REFRESH, BeanRefreshScope.getInstance());
      context.register(MainConfig.class);
  
      RefreshConfigUtil.refreshPropertySouce(context);
      context.refresh();
  
      TestService service = context.getBean(TestService.class);
      System.out.println("配置更新前——");
      for (int i = 0; i < 3; i++) {
          System.out.println(service);
          try {
              TimeUnit.MILLISECONDS.sleep(200);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
      }
  
      System.out.println("配置更新后——");
      for (int i = 0; i < 3; i++) {
          RefreshConfigUtil.updateDbConfig(context);
          System.out.println(service);
          try {
              TimeUnit.MILLISECONDS.sleep(200);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
      }
  }
  ```

* 执行结果

  ```shell
  配置更新前——
  21:42:54.096 [main] DEBUG org.springframework.core.env.PropertySourcesPropertyResolver - Found key 'test.name' in PropertySource 'test' with value of type UUID
  TestService{testConfig=TestConfig{name='ecf83aff-0cd2-4aa7-9e25-1a9413010dbc'}}
  TestService{testConfig=TestConfig{name='ecf83aff-0cd2-4aa7-9e25-1a9413010dbc'}}
  TestService{testConfig=TestConfig{name='ecf83aff-0cd2-4aa7-9e25-1a9413010dbc'}}
  配置更新后——
  21:42:54.747 [main] DEBUG org.springframework.core.env.PropertySourcesPropertyResolver - Found key 'test.name' in PropertySource 'test' with value of type UUID
  TestService{testConfig=TestConfig{name='cbff7f7a-1bbb-4daf-8f34-11239a0a8b08'}}
  21:42:54.947 [main] DEBUG org.springframework.core.env.PropertySourcesPropertyResolver - Found key 'test.name' in PropertySource 'test' with value of type UUID
  TestService{testConfig=TestConfig{name='c20f8955-6b2e-4b28-850a-4fd2b7cc6740'}}
  21:42:55.150 [main] DEBUG org.springframework.core.env.PropertySourcesPropertyResolver - Found key 'test.name' in PropertySource 'test' with value of type UUID
  TestService{testConfig=TestConfig{name='9c76b421-f2fd-4d01-8858-0e6c520beeec'}}
  ```

# Spring国际化

Java中使用java.util.Locale来表示地区语言这个对象，内部包含了国家和语言的信息

```java
public Locale(String language, String country) {
    this(language, country, "");
}
```

**Locale类中已经创建好了很多常用的Locale对象，直接可以拿过来用**

## Spring国际化使用

Spring中国际化通过MessageSource这个接口来支持的

```java
org.springframework.context.MessageSource
```

```java
public interface MessageSource {
 
    /**
     * 获取国际化信息
     * @param code 表示国际化资源中的属性名；
     * @param args用于传递格式化串占位符所用的运行参数；
     * @param defaultMessage 当在资源找不到对应属性名时，返回defaultMessage参数所指定的默认信息；
     * @param locale 表示本地化对象
     */
    @Nullable
    String getMessage(String code, @Nullable Object[] args, @Nullable String defaultMessage, Locale locale);
 
    /**
     * 与上面的方法类似，只不过在找不到资源中对应的属性名时，直接抛出NoSuchMessageException异常
     */
    String getMessage(String code, @Nullable Object[] args, Locale locale) throws NoSuchMessageException;
 
    /**
     * @param MessageSourceResolvable 将属性名、参数数组以及默认信息封装起来，它的功能和第一个方法相同
     */
    String getMessage(MessageSourceResolvable resolvable, Locale locale) throws NoSuchMessageException;
 
}
```

**常见的实现类**

ResourceBundleMessageSource

这个是基于Java的ResourceBundle基础类实现，允许仅同故宫资源名加载国际化资源

ReloadableResourceBundleMessageSource

这个功能和第一个类的功能类似，多个定时刷新功能，允许在不重启系统的情况下，更新资源的信息

StaticMessageSource

它允许通过编程式的方法提供国际化信息

## Spring中使用国际化的步骤

**AbstractApplicationContext接口实现了国际化接口**

1. 创建国际化文件
2. 向容器中注册一个MessageResource类型的Bean，bean名称必须为messageSource
3. 调用AbstractApplicationContext的getMessage来获取国际化信息，其内部将交给容器中的messageSource处理

**案例**

* 创建国际化文件

  **国际化文件命名格式：名称_语言\_地区.properties**

  * message.properties

    ```properties
    name=名称
    personal_introduction=个人信息：{0},{1}
    ```

    这个文件名称没有指定Local信息，当系统找不到的时候会默认使用这个

  * message_zh_CN.properties

    ```properties
    name=名称
    personal_introduction=个人信息：{0},{1}
    ```

  * message_en_GB.properties

    ```properties
    name=Full Name
    personal_introduction=personal_introduction: {0},{1},{0}
    ```

* Spring中注册国际化的bean

  ```java
  @Configuration
  public class MainConfig1 {
      @Bean
      public ResourceBundleMessageSource messageSource() {
          ResourceBundleMessageSource result = new ResourceBundleMessageSource();
          result.setBasenames("message");
          return result;
      }
  }
  ```

* Test

  ```java
  @SpringBootApplication
  public class SpringStudyApplication {
      public static void main(String[] args) {
          ConfigurableApplicationContext context = SpringApplication.run(SpringStudyApplication.class, args);
          System.out.println(context.getMessage("name", null, null));
          System.out.println(context.getMessage("name", null, Locale.CHINA));
          System.out.println(context.getMessage("name", null, Locale.UK));
      }
  }
  ```

## 动态参数设置

注意配置文件中，{0}， {1}， {0}这样一部分内容，这个就是动态参数，调用getMessage的时候，通过第一个参数传递过去

```java
System.out.println(context.getMessage("personal_introduction", new String[]{"Mnsx", "通天带"}, Locale.CHINA)); 
    System.out.println(context.getMessage("personal_introduction", new String[]{"spring", "java"}, Locale.UK)); 
```

## 监控国际化文件的变化

用ReloadableResourceBundleMessageSource这个类，功能与案例相似，但是可以用来监控国际化资源文件变化的功能

```java
public void setCacheMillis(long cacheMillis) // 设置缓存时间
public void setCacheSeconds(long cacheSeconds)
```

> -1: 表示永远缓存
>
> 0: 表示每次获取国际化信息的时候，都会重新读取国际化文件
>
> 大于0：上次读取配置文件的时间距离当前时间超过了这个时间，重新读取国际化文件

**线上环境，缓存时间最好设置大一点，性能会好一些**

## 国际化信息存在db中

StaticMessageSource，这个类允许通过编程的方式提供国际化信息

```java
public void addMessage(String code, Locale locale, String msg);
public void addMessages(Map<String, String> messages, Locale locale);
```

* MessageSourceFromDb

  ```java
  public class MessageSourceFromDb extends StaticMessageSource implements InitializingBean {
      @Override
      public void afterPropertiesSet() throws Exception {
          this.addMessage("desc", Locale.CHINA, "从db中来");
          this.addMessage("desc", Locale.UK, "from db");
      }
  }
  ```

* MainConfig

  ```java
  @Configuration
  public class MainConfig2 {
      @Bean
      public MessageSource messageSource() {
          return new MessageSourceFromDb();
      }
  }
  ```

* Test

  ```java
  @SpringBootApplication
  public class SpringStudyApplication {
      public static void main(String[] args) {
          ConfigurableApplicationContext context = SpringApplication.run(SpringStudyApplication.class, args);
          System.out.println("---------------------");
          System.out.println(context.getMessage("desc", null, Locale.CHINA));
          System.out.println(context.getMessage("desc", null, Locale.UK));
      }
  }
  ```

  ## 为什么Bean要叫messageSource

  refresh方法内部会调用`org.springframework.context.support.AbstractApplicationContext#initMessageSource`

  这个方法用来初始化MessageSource，方法内部会查找当前容器中是否有messageSource名称的bean，如果没有找到，此时会注册一个名称为messageSource的MessageSource

# Spring事件机制

事件源：事件的触发者

事件：描述发生了生么事情的对象

事件监听器：监听到事件发生的时候做一些处理

## Spring事件基础使用案例

* 事件对象

  ```java
  public class AbstractEvent {
      protected Object source;
  
      public AbstractEvent(Object source) {
          this.source = source;
      }
  
      public Object getSource() {
          return source;
      }
  
      public void setSource(Object source) {
          this.source = source;
      }
  }
  ```

* 事件监听器

  > 使用一个接口来表示事件监听器，是个泛型接口，后面的类型E表示当前监听器要监听的事件类型，此接口中只有一个方法，用来实现处理事件的业务，其定义的监听器需要实现这个接口

  ```java
  public interface EventListener<E extends AbstractEvent> {
      void onEvent(E event);
  }
  ```

* 事件广播器

  > * 负责事件监听器的管理（注册监听器&移除监听器，将事件和监听器关联起来）
  > * 负责事件的广播（将事件广播给所有的监听器，对该事件感兴趣的监听器会处理该事件）

  ```java
  public interface EventMulticaster {
      /**
       * 广播事件给所有的监听器，对该事件感兴趣的监听器会处理该事件
       * @param event 事件
       */
      void multicastEvent(AbstractEvent event);
  
      /**
       * 添加一个事件监听器（监听器中包含了监听器能处理那些事件
       * @param listener 监听器
       */
      void addEventListener(EventListener<?> listener);
  
      /**
       * 删除一个事件监听器
       * @param listener 监听器
       */
      void removeEventListener(EventListener<?> listener);
  }
  ```

* 事件广播默认实现

  ```java
  public class SimpleEventMulticaster implements EventMulticaster {
  
      private Map<Class<?>, List<EventListener>> eventObjectEventListenerMap = new ConcurrentHashMap<>();
  
      @Override
      public void multicastEvent(AbstractEvent event) {
          List<EventListener> eventListeners = this.eventObjectEventListenerMap.get(event.getClass());
          if (eventListeners != null) {
              for (EventListener eventListener : eventListeners) {
                  eventListener.onEvent(event);
              }
          }
      }
  
      @Override
      public void addEventListener(EventListener<?> listener) {
          Class<?> eventType = this.getEventType(listener);
          List<EventListener> eventListeners = this.eventObjectEventListenerMap.get(eventType);
          if (eventListeners == null) {
              eventListeners = new ArrayList<>();
              this.eventObjectEventListenerMap.put(eventType, eventListeners);
          }
          eventListeners.add(listener);
      }
  
      @Override
      public void removeEventListener(EventListener<?> listener) {
          Class<?> eventType = this.getEventType(listener);
          List<EventListener> eventListeners = this.eventObjectEventListenerMap.get(eventType);
          if (eventListeners != null) {
              eventListeners.remove(listener);
          }
      }
  
      /**
       * 获取事件监听器监听的事件类型
       * @param listener 监听器
       * @return 事件类型
       */
      protected Class<?> getEventType(EventListener listener) {
          ParameterizedType parameterizedType = (ParameterizedType) listener.getClass().getGenericInterfaces()[0];
          Type eventType = parameterizedType.getActualTypeArguments()[0];
          return (Class<?>) eventType;
      }
  }
  ```

* 事件

  ```java
  public class UserRegisterSuccessEvent extends AbstractEvent {
      
      private String username;
      
      public UserRegisterSuccessEvent(Object source, String username) {
          super(source);
          this.username = username;
      }
  
      public String getUsername() {
          return username;
      }
  
      public void setUsername(String username) {
          this.username = username;
      }
  }
  ```

* 服务

  ```java
  public class UserRegisterService {
      
      private EventMulticaster eventMulticaster;
      
      public void registerUser(String username) {
          System.out.printf("用户注册[%s]成功%n", username);
          this.eventMulticaster.multicastEvent(new UserRegisterSuccessEvent(this, username));
      }
  
      public EventMulticaster getEventMulticaster() {
          return eventMulticaster;
      }
  
      public void setEventMulticaster(EventMulticaster eventMulticaster) {
          this.eventMulticaster = eventMulticaster;
      }
  }
  ```

* 配置类

  ```java
  @Configuration
  public class MainConfig1 {
      @Bean
      @Autowired(required = false)
      public EventMulticaster eventMulticaster(List<EventListener> eventListeners) {
          SimpleEventMulticaster eventPublisher = new SimpleEventMulticaster();
          if (eventListeners != null) {
              eventListeners.forEach(eventPublisher::addEventListener);
          }
          return eventPublisher;
      }
      
      @Bean
      public UserRegisterService userRegisterService(EventMulticaster eventMulticaster) {
          UserRegisterService userRegisterService = new UserRegisterService();
          userRegisterService.setEventMulticaster(eventMulticaster);
          return userRegisterService;
      }
  }
  ```

* 测试

  ```java
  @SpringBootApplication
  public class SpringStudyApplication {
      public static void main(String[] args) {
          ConfigurableApplicationContext context = SpringApplication.run(SpringStudyApplication.class, args);
          UserRegisterService userRegisterService = context.getBean(UserRegisterService.class);
          userRegisterService.registerUser("Mnsx");
      }
  }
  ```

* 添加功能

  ```java
  @Component
  public class SendEmailOnUserRegisterSuccessListener implements EventListener<UserRegisterSuccessEvent> {
      @Override
      public void onEvent(UserRegisterSuccessEvent event) {
          System.out.println("发送邮件-用户" + event.getUsername() + "创建成功");
      }
  }
  ```

## Spring中实现事件模式

### 硬编码的方式

1. 定义事件

   自定义事件，需要继承ApplicationEvent

2. 定义监听器

   自定义事件监听器，需要实现ApplicationListener接口

   ```java
   @FunctionalInterface
   public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {
    
       /**
        * Handle an application event.
        * @param event the event to respond to
        */
       void onApplicationEvent(E event);
    
   }
   ```

3. 创建事件广播器

   这是个接口，需要自己实现接口，也可以使用系统提供的SimpleApplicationEventMulticaster

   ```java
   ApplicationEventMulticaster applicationEventMulticaster = new SimpleApplicationEventMulticaster();
   ```

4. 向广播器中注册事件监听器

   将事件监听器注册到广播器中

   ```java
   ApplicationEventMulticaster applicationEventMulticaster = new SimpleApplicationEventMulticaster();
   applicationEventMulticaster.addApplicationListener(new SendEmailOnOrderCreateListener());
   ```

5. 通过广播器发布事件

   广播事件，调用ApplicationEventMulticaster#multicastEvent方法广播事件，此时广播器中队这个事件感兴趣的监听器会处理这个事件

   ```java
   applicationEventMulticaster.multicastEvent(new OrderCreateEvent(applicationEventMulticaster, 1L));
   ```

### ApplicationContext容器中事件的支持

> 1.AnnotationConfigApplicationContext和ClassPathXmlApplicationContext都继承了AbstractApplicationContext
> 2.AbstractApplicationContext实现了ApplicationEventPublisher接口
> 3.AbstractApplicationContext内部有个ApplicationEventMulticaster类型的字段

AbstractApplicationContext内部已经集成了事件广播器ApplicationEventMulticaster

### ApplicationEventPublisher接口

```java
@FunctionalInterface
public interface ApplicationEventPublisher {
 
    default void publishEvent(ApplicationEvent event) {
        publishEvent((Object) event);
    }
 
    void publishEvent(Object event);
 
}
```

这个接口用来发布事件

调用AbstractApplicationContext中的publishEvent方法也能实现广播事件的效果

### 获取ApplicationEventPublisher对象

如果想要在普通的bean中获取ApplicationEventPublisher对象，需要实现ApplicationEventPublisherAware接口

```java
public interface ApplicationEventPublisherAware extends Aware {
    void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher);
}
```

## 面向接口方式

监听器需要实现ApplicationListener接口，spring容器在创建bean的过程中，会判断bean是否为ApplicationListener类型，进而将其作为监听器添加到AbstractApplicationContext#applicationEventMulticaster中

```java
org.springframework.context.support.ApplicationListenerDetector#postProcessAfterInitialization
```

## 面向注解方式

直接将注解标注在一个bean的方法

```java
@Component
public class UserRegisterListener {
    @EventListener
    public void sendMail(UserRegisterEvent event) {
        System.out.println("发送邮件-用户" + event.getUsername() + "创建成功");
    }
}
```

**原理**

`org.springframework.context.event.EventListenerMethodProcessor#afterSingletonsInstantiated`

EventListenerMethodProcessor实现了SmarInitializingSingleton接口，其中的afterSingletonsInstantiated方法会在所有单例bean床啊金完成之后被spring容器调用

## 监听支持排序功能

* 通过接口实现监听器

  * 实现Orderd接口
  * 实现PriorityOrdered接口
  * 使用@Order注解

* 通过@EventListener实现监听器

  使用@Order注解

## 监听器异步模式

默认的SimpleApplicationEventMulticaster支持监听器异步调用的

`private Executor taskExecutor;`

```java
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

ivokeListener就会调用监听器，如果当前executor不为空，监听器就会被异步调用，所以如果需要异步只需要让executor不为空

```java
@Configuration
public class MainConfig {
    @Bean
    public ApplicationEventMulticaster applicationEventMulticaster() {
        SimpleApplicationEventMulticaster simpleEventMulticaster = new SimpleApplicationEventMulticaster();
        Executor executor = this.applicationEventMulticasterThreadPool().getObject();
        simpleEventMulticaster.setTaskExecutor(executor);
        return simpleEventMulticaster;
    }
    
    @Bean
    public ThreadPoolExecutorFactoryBean applicationEventMulticasterThreadPool() {
        ThreadPoolExecutorFactoryBean result = new ThreadPoolExecutorFactoryBean();
        result.setThreadNamePrefix("applicationEventMulticasterThreadPool-");
        result.setCorePoolSize(5);
        return result;
    }
}
```

# Bean循环依赖

> 主要针对“请讲一下Spring中的循环依赖”问题的解答方案
>
> 解答方向
>
> 1. 什么是循环依赖？
> 2. 什么情况下循环依赖可以被处理？
> 3. Spring是如何解决循环依赖？

## 什么是循环依赖？

![循环依赖](Picture\Spring\循环依赖.png)

## 什么情况下循环依赖可以被处理？

Spring解决循环依赖是有前置条件的

1. 出现循环依赖的Bean必须是单例的
2. 依赖注入的方式不能全是构造器注入的方式

| 依赖情况   | 依赖注入方式                                     | 循环依赖是否被解决 |
| ---------- | ------------------------------------------------ | ------------------ |
| AB相互依赖 | 均采用setter方式注入                             | 是                 |
| AB相互依赖 | 均采用构造器方式注入                             | 否                 |
| AB相互依赖 | A中注入B的方式为setter，B中注入A的方式是为构造器 | 是                 |
| AB相互依赖 | B中注入A的方式为setter，A中注入B的方式是为构造器 | 否                 |

## Spring是如何解决循环依赖？

### 简单的循环依赖（没有AOP）

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

首先，Spring在创建Bean的时候默认是按照自然排序来进行创建的，所以第一步Spring会先创建A

> Spring在创建Bean的过程中分为三步
>
> 1. 实例化，对应方法：AbstractAutowireCapableBeanFactory#createBeanInstance
> 2. 属性注入，对应方法：AbstractAutowireCapableBeanFactory#polulateBean方法
> 3. 初始化，对应方法：AbstractAutowireCapableBeanFactory#initializeBean

调用getBean获取A

```java
@Override
public Object getBean(String name) throws BeansException {
    return doGetBean(name, null, null, false);
}

@Override
public <T> T getBean(String name, Class<T> requiredType) throws BeansException {
    return doGetBean(name, requiredType, null, false);
}
```

getBean主要的逻辑处理是调用doGetBean方法

```java
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
    }

    else {
        // 不存在，但是这个Bean已经在创建过程中，那么报错循环依赖
        if (isPrototypeCurrentlyInCreation(beanName)) {
            throw new BeanCurrentlyInCreationException(beanName);
        }

       	// 判断父容器中是否存在这个Bean
        BeanFactory parentBeanFactory = getParentBeanFactory();
        if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
            String nameToLookup = originalBeanName(name);
            if (parentBeanFactory instanceof AbstractBeanFactory) {
                return ((AbstractBeanFactory) parentBeanFactory).doGetBean(
                    nameToLookup, requiredType, args, typeCheckOnly);
            }
            else if (args != null) {
                return (T) parentBeanFactory.getBean(nameToLookup, args);
            }
            else if (requiredType != null) {
                return parentBeanFactory.getBean(nameToLookup, requiredType);
            }
            else {
                return (T) parentBeanFactory.getBean(nameToLookup);
            }
        }

        if (!typeCheckOnly) {
            markBeanAsCreated(beanName);
        }

        try {
            RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
            checkMergedBeanDefinition(mbd, beanName, args);
			
            // 处理dependsOn
            String[] dependsOn = mbd.getDependsOn();
            if (dependsOn != null) {
                for (String dep : dependsOn) {
                    if (isDependent(beanName, dep)) {
                        throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                                                        "Circular depends-on relationship between '" + beanName + "' and '" + dep + "'");
                    }
                    registerDependentBean(dep, beanName);
                    try {
                        getBean(dep);
                    }
                    catch (NoSuchBeanDefinitionException ex) {
                        throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                                                        "'" + beanName + "' depends on missing bean '" + dep + "'", ex);
                    }
                }
            }

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

            // prototype类型bean处理
            else if (mbd.isPrototype()) {
                Object prototypeInstance = null;
                try {
                    beforePrototypeCreation(beanName);
                    prototypeInstance = createBean(beanName, mbd, args);
                }
                finally {
                    afterPrototypeCreation(beanName);
                }
                bean = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
            }

            else {
                String scopeName = mbd.getScope();
                if (!StringUtils.hasLength(scopeName)) {
                    throw new IllegalStateException("No scope name defined for bean ´" + beanName + "'");
                }
                Scope scope = this.scopes.get(scopeName);
                if (scope == null) {
                    throw new IllegalStateException("No Scope registered for scope name '" + scopeName + "'");
                }
                try {
                    Object scopedInstance = scope.get(beanName, () -> {
                        beforePrototypeCreation(beanName);
                        try {
                            return createBean(beanName, mbd, args);
                        }
                        finally {
                            afterPrototypeCreation(beanName);
                        }
                    });
                    bean = getObjectForBeanInstance(scopedInstance, name, beanName, mbd);
                }
                catch (IllegalStateException ex) {
                    throw new BeanCreationException(beanName,
                                                    "Scope '" + scopeName + "' is not active for the current thread; consider " +
                                                    "defining a scoped proxy for this bean if you intend to refer to it from a singleton",
                                                    ex);
                }
            }
        }
        catch (BeansException ex) {
            cleanupAfterBeanCreationFailure(beanName);
            throw ex;
        }
    }
    if (requiredType != null && !requiredType.isInstance(bean)) {
        try {
            T convertedBean = getTypeConverter().convertIfNecessary(bean, requiredType);
            if (convertedBean == null) {
                throw new BeanNotOfRequiredTypeException(name, requiredType, bean.getClass());
            }
            return convertedBean;
        }
        catch (TypeMismatchException ex) {
            if (logger.isTraceEnabled()) {
                logger.trace("Failed to convert bean '" + name + "' to required type '" +
                             ClassUtils.getQualifiedName(requiredType) + "'", ex);
            }
            throw new BeanNotOfRequiredTypeException(name, requiredType, bean.getClass());
        }
    }
    return (T) bean;
}
```

doGetBean中这一次主要关注getSingleton(beanName, true)方法

getSingleton(beanName, true)这个方法实际上就是到缓存中尝试获取Bean

1. singletonObjects，一级缓存，存储的是所有创建好的单例Bean
2. earlySingletonObjects，二级缓存，完成实例化，但是还未进行属性注入及初始化对象
3. singletonFactories，提前暴露的一个单例工厂，二级缓存中存储的就是从这个工厂中获取到的对象

```java
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
                        // 从三级缓存中获取这个bean
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

因为没有在三级缓存中获取到bean对象，那么将初始化这个bean

```java
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
```

其中调用getSingleton方法

```java
public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
    Assert.notNull(beanName, "Bean name must not be null");
    synchronized (this.singletonObjects) {
        Object singletonObject = this.singletonObjects.get(beanName);
        if (singletonObject == null) {
            if (this.singletonsCurrentlyInDestruction) {
                throw new BeanCreationNotAllowedException(beanName,
                                                          "Singleton bean creation not allowed while singletons of this factory are in destruction " +
                                                          "(Do not request a bean from a BeanFactory in a destroy method implementation!)");
            }
            beforeSingletonCreation(beanName);
            boolean newSingleton = false;
            boolean recordSuppressedExceptions = (this.suppressedExceptions == null);
            if (recordSuppressedExceptions) {
                this.suppressedExceptions = new LinkedHashSet<>();
            }
            try {
                singletonObject = singletonFactory.getObject();
                newSingleton = true;
            }
            catch (IllegalStateException ex) {
                // Has the singleton object implicitly appeared in the meantime ->
                // if yes, proceed with it since the exception indicates that state.
                singletonObject = this.singletonObjects.get(beanName);
                if (singletonObject == null) {
                    throw ex;
                }
            }
            catch (BeanCreationException ex) {
                if (recordSuppressedExceptions) {
                    for (Exception suppressedException : this.suppressedExceptions) {
                        ex.addRelatedCause(suppressedException);
                    }
                }
                throw ex;
            }
            finally {
                if (recordSuppressedExceptions) {
                    this.suppressedExceptions = null;
                }
                afterSingletonCreation(beanName);
            }
            if (newSingleton) {
                addSingleton(beanName, singletonObject);
            }
        }
        return singletonObject;
    }
}
```

在bean创建之前，会调用beforeSingletonCreation(beanName)这个方法检查是否出现循环依赖

```java
protected void beforeSingletonCreation(String beanName) {
    if (!this.inCreationCheckExclusions.contains(beanName) && !this.singletonsCurrentlyInCreation.add(beanName)) {
        throw new BeanCurrentlyInCreationException(beanName);
    }
}
```

这就是循环依赖会报错的原因

```java
try {
    singletonObject = singletonFactory.getObject();
    newSingleton = true;
}
```

调用通过参数传入的匿名函数

```java
try {
    return createBean(beanName, mbd, args);
}
catch (BeansException ex) {
    destroySingleton(beanName);
    throw ex;
}
```

调用createBean方法

```java
	@Override
	protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
			throws BeanCreationException {

		RootBeanDefinition mbdToUse = mbd;

		Class<?> resolvedClass = resolveBeanClass(mbd, beanName);
		if (resolvedClass != null && !mbd.hasBeanClass() && mbd.getBeanClassName() != null) {
			mbdToUse = new RootBeanDefinition(mbd);
			mbdToUse.setBeanClass(resolvedClass);
		}

		try {
			mbdToUse.prepareMethodOverrides();
		}
		catch (BeanDefinitionValidationException ex) {
			throw new BeanDefinitionStoreException(mbdToUse.getResourceDescription(),
					beanName, "Validation of method overrides failed", ex);
		}

		try {
			Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
			if (bean != null) {
				return bean;
			}
		}
		catch (Throwable ex) {
			throw new BeanCreationException(mbdToUse.getResourceDescription(), beanName,
					"BeanPostProcessor before instantiation of bean failed", ex);
		}

		try {
			Object beanInstance = doCreateBean(beanName, mbdToUse, args);
			if (logger.isTraceEnabled()) {
				logger.trace("Finished creating instance of bean '" + beanName + "'");
			}
			return beanInstance;
		}
		catch (BeanCreationException | ImplicitlyAppearedSingletonException ex) {
			throw ex;
		}
		catch (Throwable ex) {
			throw new BeanCreationException(
					mbdToUse.getResourceDescription(), beanName, "Unexpected exception during bean creation", ex);
		}
	}
```

通过createBean返回的Bean最终放到了一级缓存中

createBean主要的逻辑是调用doCreateBean方法

`Object beanInstance = doCreateBean(beanName, mbdToUse, args);`

```java
protected Object doCreateBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
    throws BeanCreationException {

	// 初始化这个Bean
    BeanWrapper instanceWrapper = null;
    if (mbd.isSingleton()) {
        instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
    }
    if (instanceWrapper == null) {
        //通过反射调用构造器实例化
        instanceWrapper = createBeanInstance(beanName, mbd, args);
    }
    // 表示刚才通过反射构造器实例化的Bean
    Object bean = instanceWrapper.getWrappedInstance();
    Class<?> beanType = instanceWrapper.getWrappedClass();
    if (beanType != NullBean.class) {
        mbd.resolvedTargetType = beanType;
    }

  	synchronized (mbd.postProcessingLock) {
        if (!mbd.postProcessed) {
            try {
                applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
            }
            catch (Throwable ex) {
                throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                                                "Post-processing of merged bean definition failed", ex);
            }
            mbd.postProcessed = true;
        }
    }

    //判断是否需要暴露早期的bean，条件为（是否是单例bean && 当前容器允许循环依赖 && bean名称存在于正在创建的bean名称清单中）
    boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
                                      isSingletonCurrentlyInCreation(beanName));
    if (earlySingletonExposure) {
        if (logger.isTraceEnabled()) {
            logger.trace("Eagerly caching bean '" + beanName +
                         "' to allow for resolving potential circular references");
        }
        // 将这个Bean暴露出去
        addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
    }
	
    // ...
}
```

刚刚实例化好的Bean就是早期Bean，此时Bean还未进行属性填充，主要是将其放入三级缓存中

```java
protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
    Assert.notNull(singletonFactory, "Singleton factory must not be null");
    synchronized (this.singletonObjects) {
        //第1级缓存中不存在bean
        if (!this.singletonObjects.containsKey(beanName)) {
            //将其丢到第3级缓存中
            this.singletonFactories.put(beanName, singletonFactory);
            //后面的2行代码不用关注
            this.earlySingletonObjects.remove(beanName);
            this.registeredSingletons.add(beanName);
        }
    }
}
```

这里只是添加了一个工厂，通过这个工厂的getObject方法可以得到一个对象，而这个对象实际上就是通过getEarlyBeanReference这个方法创建的

那么，是在什么时候又需要调用这个工厂的getObject方法？

当创建B时，会发现B的内部有属性A，那么会调用getBean(A)，但是这次不是为了创建A，而是从三级缓存中获取A

```java
addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
```

可以看出主要通过getEarlyBeanReference方法获取提前暴露的一个对象

```java
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

这段代码的主要逻辑是轮询Bean后置处理器中的SmartInstantiationAwareBeanPostProcessor，然后调用其中的getEarlyBeanReference方法

而真正实现了这个方法的只有一个，就是通过@EnableAspectAutoProxy注解导入的AnnotationAwareAspectJAutoProxyCreator，也就是说如果不考虑AOP，那么上面代码等价于

```java
protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
    Object exposedObject = bean;
    return exposedObject;
}
```

也就是说，如果不考虑AOP，那么三级缓存没有什么用

> B中提前注入了一个没有经过初始化的A类型对象不会有问题吗？
>
> 虽然在创建B时会提前给B注入了一个还未初始化的A对象，但是在创建A的流程中一直使用的是注入到B中的A对象的引用，之后会根据这个引用对A进行初始化，所以这是没有问题的

### 结合了AOP的情况

开启或者不开启AOP两者主要区别在于

```java
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

如果在开启AOP的情况下，那么就是调用到`AnnotationAwareAspectJAutoProxyCreator`的`getEarlyBeanReference`方法

```java
public Object getEarlyBeanReference(Object bean, String beanName) {
    Object cacheKey = getCacheKey(bean.getClass(), beanName);
    this.earlyProxyReferences.put(cacheKey, bean);
    // 如果需要代理，返回一个代理对象，不需要代理，直接返回当前传入的这个bean对象
    return wrapIfNecessary(bean, beanName, cacheKey);
}
```

因为开启了AOP，那么A中注入的B，不是实例化阶段创建的对象，这样就意味着B中注入的A将是一个代理对象而不是A的实例化阶段创建后的对象

```java
Object exposedObject = bean;
try {
    populateBean(beanName, mbd, instanceWrapper);
    exposedObject = initializeBean(beanName, exposedObject, mbd);
}
catch (Throwable ex) {
    if (ex instanceof BeanCreationException && beanName.equals(((BeanCreationException) ex).getBeanName())) {
        throw (BeanCreationException) ex;
    }
    else {
        throw new BeanCreationException(
            mbd.getResourceDescription(), beanName, "Initialization of bean failed", ex);
    }
}

if (earlySingletonExposure) {
    Object earlySingletonReference = getSingleton(beanName, false);
    if (earlySingletonReference != null) {
        if (exposedObject == bean) {
            exposedObject = earlySingletonReference;
        }
    }
}
```

**为什么要使用第三级缓存**

**这个工厂的目的在于延迟对实例化阶段生成的对象的代理，只有真正发生循环依赖的时候，才去提前生成代理对象，否则只会创建一个工厂并将其放入到三级缓存中，但是不会去通过这个工厂去真正创建对象**

**最后Spring的三级缓存并没有提升效率**

# BeanFactory扩展

## Spring容器中主要的4个阶段

* 阶段1：Bean注册阶段，此阶段会完成所有bean的注册
* 阶段2：BeanFactory后置处理阶段
* 阶段3：注册BeanPostProcessor
* 阶段4：bean创建阶段，此阶段完成所有单例bean的注册和装载操作

## Bean注册阶段

Spring所有bean的注册都会在此阶段完成，所有bean的注册都必须在此阶段进行，其他阶段不再进行bean的注册

这个阶段Spring提供一个接口：BeanDefinitionRegistryPostProcessor，Spring容器在这个阶段中会获取容器中所有类型为BeanDefinitionRegistryPostProcessor的bean，然后会调用postProcessBeanDefinitionRegistry方法，方法参数是BeanDefinitionRegistry即bean定义注册器，内部提供方法用来向容器中注册bean

```java
public interface BeanDefinitionRegistryPostProcessor extends BeanFactoryPostProcessor {
    void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException;
}
```

这个接口继承了BeanFactoryPostProcessor接口

当容器中有多个BeanDefinitionRegistryPostPorcessor，可以通过以下方式来指定顺序

* 实现PriorityOrdered接口
* 实现Ordered接口

### 案例1：简单使用

* 自定义一个BanDefinitionRegistryPostProcessor

  ```java
  @Component
  public class MyBeanDefinitionRegistryPostProcessor implements BeanDefinitionRegistryPostProcessor {
      @Override
      public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
          AbstractBeanDefinition nameBdf = BeanDefinitionBuilder
                  .genericBeanDefinition(String.class)
                  .addConstructorArgValue("Mnsx")
                  .getBeanDefinition();
  
          registry.registerBeanDefinition("name", nameBdf);
      }
  
      @Override
      public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
  
      }
  }
  ```

### 案例2：多个指定顺序

* 自定义两个BeanDefinitionRegistryPostProcessor，都实现Ordered接口，第一个order的值为2，第二个order的值为1

  ```java
  @Component
  public class BeanDefinitionRegistryPostProcessor1 implements BeanDefinitionRegistryPostProcessor, PriorityOrdered {
      @Override
      public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
          System.out.println("BeanDefinitionRegistryPostProcessor1");
          AbstractBeanDefinition nameBdf = BeanDefinitionBuilder
                  .genericBeanDefinition(String.class)
                  .addConstructorArgValue("Mnsx")
                  .getBeanDefinition();
  
          registry.registerBeanDefinition("name", nameBdf);
      }
  
      @Override
      public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
  
      }
  
      @Override
      public int getOrder() {
          return 2;
      }
  }
  ```

  ```java
  @Component
  public class BeanDefinitionRegistryPostProcessor2 implements BeanDefinitionRegistryPostProcessor, PriorityOrdered {
      @Override
      public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
          System.out.println("BeanDefinitionRegistryPostProcessor2");
          AbstractBeanDefinition nameBdf = BeanDefinitionBuilder
                  .genericBeanDefinition(String.class)
                  .addConstructorArgValue("BMW")
                  .getBeanDefinition();
  
          registry.registerBeanDefinition("car", nameBdf);
      }
  
      @Override
      public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
  
      }
  
      @Override
      public int getOrder() {
          return 1;
      }
  }
  ```

BeanDefinitionRegistryPostProcessor有个非常重要的实现类

```java
org.springframework.context.annotation.ConfigurationClassPostProcessor
```

这个实现类，通过注解实现bean的批量注册

```shell
@Configuration
@ComponentScan
@Import
@ImportResource
@PropertySource
```

## BeanFactory后置处理阶段

Spring容器已经完成了所有Bean的注册，这个阶段中可以队BeanFactory中的一些信息进行修改，此阶段Spring也提供了一个接口来进行扩展：BeanFactoryPostPostProcessor类型的bean然后调用他们的postProcessorBeanFactory

```java
@FunctionalInterface
public interface BeanFactoryPostProcessor {
 
    void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;
 
}
```

当容器中有多个BeanFactoryPostProcessor的时候，可以通过下面任意下面一种方式指定顺序

1. 实现PriorityOrdered
2. 实现Ordered

### 案例

在BeanFactoryPostProcessor来修改Bean中已经注册的bean定义的信息，给一个bean属性设置一个值

* 定义一个BeanFacotryPostProcessor实现类

  ```java
  @Component
  public class MyBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
      @Override
      public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
          System.out.println("准备修改lessonModel bean定义信息");
          BeanDefinition beanDefinition = beanFactory.getBeanDefinition("lessonModel");
          beanDefinition.getPropertyValues().add("name", "Mnsx低手课堂");
      }
  }
  ```

### 几个重要的实现类

**PropertySourcePlaceHolderConfigurer**

```xml
<bean class="xxxxx">
    <property name="userName" value="${userName}"/>
    <property name="address" value="${address}"/>
</bean>
```

Spring就是在PropertySourcePlaceholderConfigurer#postProcessoBeanFactory中来处理xml中属性中的${xxx}，会对这种格式的进行解析处理为真正的值

**CustomScopeConfigurer**

向荣日中注册自定义的Scope对象，即注册自定义的作用域实现类，关于自定义的作用域

**EventListenerMethodProcessor**

处理@EventListener注解的，即Spring中事件机制

### 使用注意

BeanFactoryPostProcessor接口的使用有一个需要注意的地方，在其postProcessorBeanFactory方法中，强烈禁止去通过容器获取其他bean，此时会导致bean的提前初始化，会出现意想不到的问题，此阶段中BeanPostProcessor还未准备好

BeanPostProcessor是在第三阶段中注册到Spring容器的，而BeanPostProcessor可以对bean的创建过程进行干预，此时如果去获取bean，此时bean不会被BeanPostProcessor处理，所以创建的bean可能是有问题的

## 后续源码学习

4个阶段的源码

```java
org.springframework.context.support.AbstractApplicationContext#refresh
```

重要代码如下

```java
// 对应阶段1和阶段2：调用上下文中注册为bean的工厂处理器，即调用本文介绍的2个接口中的方法
invokeBeanFactoryPostProcessors(beanFactory);
 
// 对应阶段3：注册拦截bean创建的bean处理器，即注册BeanPostProcessor
registerBeanPostProcessors(beanFactory);
 
// 对应阶段3：实例化所有剩余的（非延迟初始化）单例。
finishBeanFactoryInitialization(beanFactory);
```

阶段一和阶段而的源码位于下面的方法

```java
org.springframework.context.support.PostProcessorRegistrationDelegate#invokeBeanFactoryPostProcessors(org.springframework.beans.factory.config.ConfigurableListableBeanFactory, java.util.List<org.springframework.beans.factory.config.BeanFactoryPostProcessor>)
```

# Spring上下文生命周期

## 应用上下文的生命周期

1. 创建Spring应用上下文
2. 上下文启动准备阶段
3. BeanFactory创建阶段
4. BeanFactory准备阶段
5. BeanFactory后置处理阶段
6. BeanFactory注册BeanPostProcessor阶段
7. 初始化内建Bean：MessageSource
8. 初始化内建Bean：Spring事件广播器
9. Spring应用上下文刷新阶段
10. Spring事件监听器注册阶段
11. 单例Bean实例化阶段
12. BeanFactory初始化完成阶段
13. Spring应用上下文启动完成阶段
14. Spring应用上下文关闭阶段

## 创建Spring应用上下文

对应代码

`AnnotationConfigApplicationContext configApplicationContext = new AnnotationConfigApplicationContext();`

![AnnotationConfigApplicationContext类图](D:\WorkSpace\Mnsx-Note\2023\Picture\Spring\AnnotationConfigApplicationContext类图.png)

当调用父类的构造器时会先调用子类的构造器

```java
public GenericApplicationContext() {
    this.beanFactory = new DefaultListableBeanFactory();
}
```

在这个构造函数中，创建了BeanFactory

```java
public AnnotationConfigApplicationContext() {
    this.reader = new AnnotatedBeanDefinitionReader(this);
    this.scanner = new ClassPathBeanDefinitionScanner(this);
}
```

`new AnnotatedBeanDefinitionReader()`中有一个非常关键的方法，其中会创建5个关键Bean

```java
org.springframework.context.annotation.AnnotationConfigUtils#registerAnnotationConfigProcessors(org.springframework.beans.factory.support.BeanDefinitionRegistry, java.lang.Object)
 
public static Set<BeanDefinitionHolder> registerAnnotationConfigProcessors(
   BeanDefinitionRegistry registry, @Nullable Object source) {
 
    //1、注册ConfigurationClassPostProcessor，这是个非常关键的类，实现了BeanDefinitionRegistryPostProcessor接口
    // ConfigurationClassPostProcessor这个类主要做的事情：负责所有bean的注册
    if (!registry.containsBeanDefinition(CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(ConfigurationClassPostProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME));
    }
 
    //2、注册AutowiredAnnotationBeanPostProcessor：负责处理@Autowire注解
    if (!registry.containsBeanDefinition(AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(AutowiredAnnotationBeanPostProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME));
    }
 
    //3、注册CommonAnnotationBeanPostProcessor：负责处理@Resource注解
    if (jsr250Present && !registry.containsBeanDefinition(COMMON_ANNOTATION_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(CommonAnnotationBeanPostProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, COMMON_ANNOTATION_PROCESSOR_BEAN_NAME));
    }
 
 //4、注册EventListenerMethodProcessor：负责处理@EventListener标注的方法，即事件处理器
    if (!registry.containsBeanDefinition(EVENT_LISTENER_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(EventListenerMethodProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_PROCESSOR_BEAN_NAME));
    }
    
 //5、注册DefaultEventListenerFactory：负责将@EventListener标注的方法包装为ApplicationListener对象
    if (!registry.containsBeanDefinition(EVENT_LISTENER_FACTORY_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(DefaultEventListenerFactory.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_FACTORY_BEAN_NAME));
    }
 
    return beanDefs;
}
```

上面代码主要会注册5个Bean

* ConfigurationClassPostProcessor：这是个非常关键的Bean，基本上自定义的Bean都是通过这个类注册的

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

* AutowiredAnnotationBeanPostProcessor：负责处理@Autowire注解

* 注册CommonAnnotationBeanPostProcessor：负责处理@Resource注解

* 注解EventListenerMethodProcessor：负责处理@EventListener标注的方法，即事件处理器

* 注册DefaultEventListenerFactory：负责将@EventListener标注的方法包装为ApplicationListener对象

## 阶段2-阶段13

阶段2-阶段13，这中间的12个阶段都位于refresh方法中

```java
org.springframework.context.support.AbstractApplicationContext#refresh

    @Override
    public void refresh() throws BeansException, IllegalStateException {
    // 阶段2：Spring应用上下文启动准备阶段
    prepareRefresh();

    // 阶段3：BeanFactory创建阶段
    ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

    // 阶段4：BeanFactory准备阶段
    prepareBeanFactory(beanFactory);

    try {
        // 阶段5：BeanFactory后置处理阶段
        postProcessBeanFactory(beanFactory);

        // 阶段6：BeanFactory注册BeanPostProcessor阶段
        invokeBeanFactoryPostProcessors(beanFactory);

        // 阶段7：注册BeanPostProcessor
        registerBeanPostProcessors(beanFactory);

        // 阶段8：初始化内建Bean：MessageSource
        initMessageSource();

        // 阶段9：初始化内建Bean：Spring事件广播器
        initApplicationEventMulticaster();

        // 阶段10：Spring应用上下文刷新阶段，由子类实现
        onRefresh();

        // 阶段11：Spring事件监听器注册阶段
        registerListeners();

        // 阶段12：实例化所有剩余的（非lazy init）单例。
        finishBeanFactoryInitialization(beanFactory);

        // 阶段13：刷新完成阶段
        finishRefresh();
    }
}
```

### Spring上下文启动准备阶段

```java
protected void prepareRefresh() {
    // 标记启动时间
    this.startupDate = System.currentTimeMillis();
    // 是否关闭：false
    this.closed.set(false);
    // 是否启动：true
    this.active.set(true);
 
    // 初始化PropertySource，留个子类去实现的，可以在这个方法中扩展属性配置信息，丢到this.environment中
    initPropertySources();
 
    // 验证环境配置中是否包含必须的配置参数信息，比如我们希望上下文启动中，this.environment中必须要有某些属性的配置，才可以启动，那么可以在这个里面验证
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

> **什么是早期事件？什么是早期的事件监听器？**
>
> 当使用Spring发布事务
>
> ```java
> applicationContext.publishEvent(事件对象)
> ```
>
> 其内部最终会用到AbstractApplicationContext下的这个属性来发布事件，但是可能此时applicationEventMulticaster还没有创建好，是空对象，所以此时是无法广播事件的，那么此时发布的事件就是早期事件，就会被放在this.earlyApplicationEvents中暂时存放，当事件广播器applicationEventMulticaster创建好之后，才会将早期事件广播出去
>
> ```java
> applicationContext.addApplicationListener(事件监听器对象)
> ```
>
> 同理，当applicationEventMulticaster还未准备好时，调用Spring上下文准备添加监听器，这时放进去的监听器就是早期的事件监听器

### BeanFactory创建阶段

这个阶段负责将BeanFactory创建好，返回给Spring应用上下文

```java
ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
```

obtainFreshBeanFactory

```java
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
    //刷新BeanFactory，由子类实现
    refreshBeanFactory();
    //返回spring上下文中创建好的BeanFacotry
    return getBeanFactory();
}
```

### BeanFactory准备阶段

```java
prepareBeanFactory(beanFactory);
```

```java
protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
    // 设置类加载器
    beanFactory.setBeanClassLoader(getClassLoader());
    // 设置spel表达式解析器
    beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));
    // 设置属性编辑器注册器
    beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));
 
    // 添加一个BeanPostProcessor：ApplicationContextAwareProcessor，当你的bean实现了spring中xxxAware这样的接口，那么bean创建的过程中，将由ApplicationContextAwareProcessor来回调这些接口，将对象注入进去
    beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
    // 自动注入的时候，下面这些接口定义的方法，将会被跳过自动注入
    beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
    beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
    beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
    beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
    beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
    beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);
 
    // 注册依赖注入的时候查找的对象，比如你的bean中想用ResourceLoader
    beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
    beanFactory.registerResolvableDependency(ResourceLoader.class, this);
    beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
    beanFactory.registerResolvableDependency(ApplicationContext.class, this);
 
    // 注入一个beanpostprocessor：ApplicationListenerDetector
    beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));
 
    // 将Environment注册到spring容器，对应的bena名称是environment
    if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
        beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
    }
    // 将系统属性注册到spring容器，对应的bean名称是systemProperties
    if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {
        beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
    }
    // 将系统环境变量配置信息注入到spring容器，对应的bean名称是systemEnvironment
    if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) {
        beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
    }
}
```

其中关键代码`beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));`ApplicationContextAwareProcessor源码

```java
if (bean instanceof EnvironmentAware) {
    ((EnvironmentAware) bean).setEnvironment(this.applicationContext.getEnvironment());
}
if (bean instanceof EmbeddedValueResolverAware) {
    ((EmbeddedValueResolverAware) bean).setEmbeddedValueResolver(this.embeddedValueResolver);
}
if (bean instanceof ResourceLoaderAware) {
    ((ResourceLoaderAware) bean).setResourceLoader(this.applicationContext);
}
if (bean instanceof ApplicationEventPublisherAware) {
    ((ApplicationEventPublisherAware) bean).setApplicationEventPublisher(this.applicationContext);
}
if (bean instanceof MessageSourceAware) {
    ((MessageSourceAware) bean).setMessageSource(this.applicationContext);
}
if (bean instanceof ApplicationContextAware) {
    ((ApplicationContextAware) bean).setApplicationContext(this.applicationContext);
}
```

```java
beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
beanFactory.registerResolvableDependency(ResourceLoader.class, this);
beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
beanFactory.registerResolvableDependency(ApplicationContext.class, this);
```

这个用来向Spring容器中添加依赖在查找对象，第一个行添加BeanFactory，当需要使用这些属性时，BeanFactory将自动注入

```java
beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));
```

ApplicationListenerDetebtor也是一个BeanPostProcessor，处理自定义的监听器

当Bean实现了ApplicationListener接口，是一个事件监听器的时候，那么这个Bean的创建过程会被ApplicationListenerDetector，会将Bean添加到Spring上下文容器的事件监听器列表中

```java
if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
    beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
}
```

将Environment作为单例注册到Spring容器单例列表中，对应的bean名称是environment，当我们的bean中需要使用到Environment的时候，可以使用下面的写法，此时Spring容器创建bean的过程，就会从单例的bean列表中找到Environment，将其注入到下面的属性中

```java
@Autowire
private Environment environment
```

```java
getEnvironment().getSystemProperties() -- 对应 --> System.getProperties()
getEnvironment().getSystemEnvironment() -- 对应 --> System.getenv()
```

上面两个方法，通过`java -jar -D参数=值`命令配置的参数，会被放在System.getProperties()中

### BeanFactory后置处理阶段

```java
postProcessBeanFactory(beanFactory);
```

这个方法留给子类实现的，此时beanFactory已经创建好了，但是容器中的bean还没有被实例化，子类可以实现这个方法，可以对BeanFactory做一些特殊的配置，比如可以添加一些自定义BeanPostProcessor等等，主要是留给子类去扩展的

### BeanFactory注册BeanPostProcessor阶段

```java
invokeBeanFactoryPostProcessors(beanFactory);
```

这个阶段主要就是从Spring容器中找到BeanFactoryPostProcessor接口的所有实现类，然后调用，完成所有Bean注册的功能，注意是**Bean注册**，即将Bean的定义信息转换为BeanDefinition对象，然后注册到Spring容器中，此时Bean还未实例化

> 阶段一在Spring容器中注册了一个非常关键的Bean：ConfigurationClassPostProcessor
>
> ```java
> if (!registry.containsBeanDefinition(CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME)) {
>     RootBeanDefinition def = new RootBeanDefinition(ConfigurationClassPostProcessor.class);
>     def.setSource(source);
>     beanDefs.add(registerPostProcessor(registry, def, CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME));
> }
> ```
>
> ConfigurationClassPostProcessor就实现了BeanFactoryPostProcessor接口，所以ConfigurationClassPostProcessor会在阶段6中被调用，下面这些注解都是在这个类中处理的
>
> ```java
> @Configuration
> @Component
> @PropertySource
> @PropertySources
> @ComponentScan
> @ComponentScans
> @Import
> @ImportResource
> @Bean
> ```

**通常，阶段6执行完毕之后，我们所有自定义的bean都已经被注册到spring容器中了，被转换为BeanDefinition丢到BeanFactory中了，此时bean还未被实例化**

### 注册BeanPostProcessor

```java
registerBeanPostProcessors(beanFactory);
```

注册BeanPostProcessor，这个阶段会遍历Spring容器bean定义列表，把所有实现了BeanPostProcessor接口的Bean提出来，然后将他们添加到Spring容器的BeanPostProcessor列表中

BeanPostProcessor是bean后置处理器，其内部提供了很多方法，用来对Bean创建过程进行扩展

### 初始化内存Bean：MessageSource

```java
initMessageSource();
```

MessageSource是用来处理国际化的，这个阶段会将MessageSource创建好

### 初始化内建Bean：Spring事件广播器

```java
initApplicationEventMulticaster();
```

ApplicationEventMulticaster是事件广播器，用来广播事件，这个阶段会将ApplicationEventMulticaster创建好，如果想自定义事件广播器

### Spring应用上下文刷新阶段，由子类实现

```java
onRefresh();
```

用来初始化其他特殊的Bean，由子类去实现

### Spring事件监听器注册阶段

```java
registerListeners();
```

注册事件监听器到事件广播器中

```java
protected void registerListeners() {
    // 先注册静态指定的侦听器,即将spring上下文中添加的时间监听器，添加到时间广播器（ApplicationEventMulticaster）中
    for (ApplicationListener<?> listener : getApplicationListeners()) {
        getApplicationEventMulticaster().addApplicationListener(listener);
    }
 
    // 将spring容器中定义的事件监听器，添加到时间广播器（ApplicationEventMulticaster）中
    String[] listenerBeanNames = getBeanNamesForType(ApplicationListener.class, true, false);
    for (String listenerBeanName : listenerBeanNames) {
        getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);
    }
 
    // 此时事件广播器准备好了，所以此时发布早期的事件（早期的事件由于事件广播器还未被创建，所以先被放在了earlyApplicationEvents中，而此时广播器创建好了，所以将早期的时间发布一下）
    Set<ApplicationEvent> earlyEventsToProcess = this.earlyApplicationEvents;
    this.earlyApplicationEvents = null;
    if (earlyEventsToProcess != null) {
        for (ApplicationEvent earlyEvent : earlyEventsToProcess) {
            getApplicationEventMulticaster().multicastEvent(earlyEvent);
        }
    }
}
```

首先看，将上下文中的事件监听器添加到事件广播器中

```java
org.springframework.context.support.AbstractApplicationContext
 
Set<ApplicationListener<?>> applicationListeners = new LinkedHashSet<>();
 
public Collection<ApplicationListener<?>> getApplicationListeners() {
    return this.applicationListeners;
}
 
@Override
public void addApplicationListener(ApplicationListener<?> listener) {
    Assert.notNull(listener, "ApplicationListener must not be null");
    if (this.applicationEventMulticaster != null) {
        this.applicationEventMulticaster.addApplicationListener(listener);
    }
    this.applicationListeners.add(listener);
}
```

再看，广播早期的事件，由于事件广播器applicationEventMulticaster还是空的，所以事件被放在了this.earlyApplicationEvents这个结合中并没有广播，等到这个时候才开始广播，在阶段11之前调用publishEvent都不会有任何效果

### 实例化所有剩余的（非Lazy Init）单例

```java
finishBeanFactoryInitialization(beanFactory);
```

这个方法中将实例化所有单例Bean（不包含需要延迟实例化的Bean）

```java
protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
    // 添加${}表达式解析器
    if (!beanFactory.hasEmbeddedValueResolver()) {
        beanFactory.addEmbeddedValueResolver(strVal -> getEnvironment().resolvePlaceholders(strVal));
    }

    // 冻结所有Bean定义，表示已注册的Bean定义不会被进一步修改或处理，这允许工厂缓存Bean
    beanFactory.freezeConfiguration();

	// 实例化所有单例Bean，通常scope=singleton的bean
    beanFactory.preInstantiateSingletons();
}
```

这里会调用beanFactory的preInstantiateSingletons()方法，源码位于DefaultListableBeanFactory

```java
org.springframework.beans.factory.support.DefaultListableBeanFactory#preInstantiateSingletons
 
public void preInstantiateSingletons() throws BeansException {
    // beanDefinitionNames表示当前BeanFactory中定义的bean名称列表，
    List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);
 
    // 循环实例化bean
    for (String beanName : beanNames) {
        //这里根据beanName来来实例化bean，代码省略了。。。。
    }
 
    // 变量bean，若bean实现了SmartInitializingSingleton接口，将调用其afterSingletonsInstantiated()方法
    for (String beanName : beanNames) {
        Object singletonInstance = getSingleton(beanName);
        if (singletonInstance instanceof SmartInitializingSingleton) {
            final SmartInitializingSingleton smartSingleton = (SmartInitializingSingleton) singletonInstance;
            smartSingleton.afterSingletonsInstantiated();
        }
    }
}
```

1. 完成所有单例bean的实例化：循环遍历beanNames列表，完成所有单例bean的实例化工作，这个循环完成之后，所有单例Bean已经实例化完成了，被放在spring容器缓存
2. 回调SmartInitializingSingleton接口：此时所有的单例bean已经实例化好了，此时遍历所有单例bean，若bean实现了SmartInitializingSingleton接口，这个接口中有个afterSingletonsInstantiated方法，此时会回调这个方法，若想在所有单例bean创建完毕之后，可以实现这个接口

### 刷新完成阶段

```java
finishRefresh();
```

```java
protected void finishRefresh() {
    // 清理一些资源缓存
    clearResourceCaches();
 
    // 为此上下文初始化生命周期处理器
    initLifecycleProcessor();
 
    // 首先将刷新传播到生命周期处理器
    getLifecycleProcessor().onRefresh();
 
    // 发布ContextRefreshedEvent事件
    publishEvent(new ContextRefreshedEvent(this));
}
```

为上下文初始化生命周期处理器

```java
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

首先将刷新传播到生命周期处理器

```java
getLifecycleProcessor().onRefresh();
```

调用LifecycleProcessor的onRefresh方法，org.springframework.context.LifecycleProcessor这个接口，生命周期处理器，接口中定义了两个方法，而onClose方法会在上下文关闭中会被调用，稍后会看到

```java
public interface LifecycleProcessor extends Lifecycle {
 
 /**
  * 上下文刷新通知
  */
 void onRefresh();
 
 /**
  * 上下文关闭通知
  */
 void onClose();
 
}
```

先看onRefresh方法，这个接口有默认实现org.springframework.context.support.DefaultLifecycleProcessor，所以默认情况下，我们可以直接看DefaultLifecycleProcessor中的onRefresh源码

```java
public void onRefresh() {
    startBeans(true);
    this.running = true;
}
```

进入startBeans(true)内部看看，就是从容器上找到所有实现org.springframework.context.Lifecycle接口的bean，然后调用start方法

```java
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

发布ConetextRefreshedEvent事件

```java
publishEvent(new ContextRefreshedEvent(this));
```

发布ContectRefreshedEvent事件，想在这个阶段做事情，可以监听这个事件

**总结**

这个阶段主要干了3个事情：

1. 初始化当前上下文的生命周期处理器LifecycleProcessor
2. 调用LifecycleProcessor.onRefresh()方法，若没有自定义LifecycleProcessor，那么会走DefaultLifecycleProcessor，其内部会调用Spring容器中所实现Lifecycle接口的bean的start方法
3. 发布ContextRefreshedEvent事件

## Spring应用上下文关闭阶段

```java
applicationContext.close()
```

最终会进入到doClose()方法

```java
org.springframework.context.support.AbstractApplicationContext#doClose
 
protected void doClose() {
    // 判断是不是需要关闭（active为tue的时候，才能关闭，并用cas确保并发情况下只能有一个执行成功）
    if (this.active.get() && this.closed.compareAndSet(false, true)) {
        
        // 发布关闭事件ContextClosedEvent
        publishEvent(new ContextClosedEvent(this));
 
        // 调用生命周期处理器的onClose方法
        if (this.lifecycleProcessor != null) {
            this.lifecycleProcessor.onClose();
        }
 
        // 销毁上下文的BeanFactory中所有缓存的单例
        destroyBeans();
 
        // 关闭BeanFactory本身
        closeBeanFactory();
 
        // 留给子类去扩展的
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

发布ContextClosedEvent事件

```java
publishEvent(new ContextClosedEvent(this));
```

调用生命周期处理器的onClose方法，默认使用DefaultLifecycleProcessor的onClose()

```java
public void onClose() {
    stopBeans();
    //将running设置为false
    this.running = false;
}
```

stopBeans()就是从容器中找到所有实现了org.springframework.context.Lifecycle接口的bean，然后调用他们的close()方法

```java
private void stopBeans() {
    Map<String, Lifecycle> lifecycleBeans = getLifecycleBeans();
    Map<Integer, LifecycleGroup> phases = new HashMap<>();
    lifecycleBeans.forEach((beanName, bean) -> {
        int shutdownPhase = getPhase(bean);
        LifecycleGroup group = phases.get(shutdownPhase);
        if (group == null) {
            group = new LifecycleGroup(shutdownPhase, this.timeoutPerShutdownPhase, lifecycleBeans, false);
            phases.put(shutdownPhase, group);
        }
        group.add(beanName, bean);
    });
    if (!phases.isEmpty()) {
        List<Integer> keys = new ArrayList<>(phases.keySet());
        keys.sort(Collections.reverseOrder());
        for (Integer key : keys) {
            phases.get(key).stop();
        }
    }
}
```

销毁上下文的BeanFactory中所有缓存的单例bean

```java
destroyBeans();
```

1. 调用bean的销毁方法

   bean实现了org.springframework.beans.factory.DisposableBean接口，此时会被调用，还有bean中有方法标记了@PreDestroy注解的，此时会被调用

2. 将Bean信息从Beanfactory中清除

```java
closeBeanFactory();
```

主要就是将Spring应用上下文中的beanFactory属性还原

```java
org.springframework.context.support.AbstractRefreshableApplicationContext#closeBeanFactory
 
@Override
protected final void closeBeanFactory() {
    synchronized (this.beanFactoryMonitor) {
        if (this.beanFactory != null) {
            this.beanFactory.setSerializationId(null);
            this.beanFactory = null;
        }
    }
}
```

# JDK动态代理和CGLIB代理

## JDK动态代理

### 特征

* 只能为接口创建代理对象
* 创建出来的代理都是Proxy的子类

## CGLIB代理

### 特征

* CGLIB弥补了JDK动态代理的不足，JDK动态代理只能为接口创建代理，而CGLIB非常强大，不管是接口还是类，都可以使用CGLIB来创建代理
* CGLIB创建代理的过程，相当于创建了一个新的类，可以通过CGLIB来配置这个新的类需要实现接口以及需要继承的父类
* CGLIB可以为类创建代理，但是这个类不是final类型的，CGLIB为类创建代理的过程，实际上通过继承实现的，相当于给需要被代理的类创建了一个子类，然后会重写父类中的方法，来进行增强，final修饰的类是不能被继承的，final修饰的方法是不能被子类重写的，被重写的这些方法可以通过CGLIB进行拦截增强

### 过程

1. CGLIB根据父类，Callback，Filter以及一些相关的信息生成key
2. 然后根据key生成对应的子类的二进制表示形式
3. 使用ClassLoader装载对应的二进制，生成Class对象，并缓存
4. 最后实例化Class对象，并缓存

### 案例

**LazyLoader的使用**

LazyLoader是CGLIB用于实现懒加载的callback，当被增强bean的方法初次被调用时，会触发回调，之后每次再进行方法调用都会对LazyLoader第一次返回的bean调用

```java
public class LazyLoaderTest1 {
    public static class UserModel {
        private String name;

        public UserModel() {
        }

        public UserModel(String name) {
            this.name = name;
        }

        public void say() {
            System.out.println("hello " + name);
        }
    }

    public static void main(String[] args) {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(UserModel.class);
        LazyLoader lazyLoader = new LazyLoader() {

            @Override
            public Object loadObject() throws Exception {
                System.out.println("调用LazyLoader。loadObject()方法");
                return new UserModel("Mnsx");
            }
        };
        enhancer.setCallback(lazyLoader);
        UserModel proxy = (UserModel) enhancer.create();
        System.out.println("第一次调用");
        proxy.say();
        System.out.println("第二次调用");
        proxy.say();
    }
}
```

**Dispatcher的使用**

Dispatcher和LazyLoader作用相似，区别是Dispatcher的话每次对增强bean进行方法调用都会触发回调函数

```java
public class DispatcherTest1 {
    public static class UserModel {
        private String name;

        public UserModel() {
        }

        public UserModel(String name) {
            this.name = name;
        }

        public void say() {
            System.out.println("hello " + name);
        }
    }

    public static void main(String[] args) {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(LazyLoaderTest1.UserModel.class);
        Dispatcher dispatch = new Dispatcher() {

            @Override
            public Object loadObject() throws Exception {
                System.out.println("调用Dispatcher.loadObject()方法");
                return new LazyLoaderTest1.UserModel("Mnsx, " + UUID.randomUUID());
            }
        };
        enhancer.setCallback(dispatch);
        LazyLoaderTest1.UserModel proxy = (LazyLoaderTest1.UserModel) enhancer.create();
        System.out.println("第一次调用say方法");
        proxy.say();
        System.out.println("第二次调用say方法");
        proxy.say();
    }
}
```

**NamingPolicy接口**

接口NamingPolicy表示生成代理类的名称策略，通过Enhancer.setNamingPolicy方法设置名称策略

默认实现类DefaultNamingPolicy，具体CGLIB动态类的命名控制

DefaultNamingPolicy生成代理类的类名命名规则

```shell
被代理class name + "$$" + 使用cglib处理的class name + "ByCGLIB" + "$$" + key的hashcode
```

自定义NamingPolicy，通常会继承DefaultNamingPolicy来实现，spring中默认就提供了一个

```java
public class SpringNamingPolicy extends DefaultNamingPolicy {
 
    public static final SpringNamingPolicy INSTANCE = new SpringNamingPolicy();
 
    @Override
    protected String getTag() {
        return "BySpringCGLIB";
    }
 
}
```

## Objenesis：实例化对象的一种方式

如果不适用构造函数的情况下如何创建对象

cglib中提供了一个接口：Objenesis，通过这个接口可以解决上面这种问题，它专门用来创建对象，即使没有空的构造函数，它不使用构造方法创建Java对象，所以即使有空的构造方法，也是不会执行的

**使用案例**

```java
@Test
public void test1() {
    Objenesis objenesis = new ObjenesisStd();
    User user = objenesis.newInstance(User.class);
    System.out.println(user);
}
```

# Aop概念详解

## Spring中AOP一些概念

> Joinpoint（连接点）：在系统运行之前，AOP 的功能模块都需要织入到具体的功能模块中。要进行这种 织入过程，我们需要知道 在系统的哪些执行点上(即具体的功能模块) 进行 织入过程，这些将要 在其之上 进行织入操作的 系统执行点就称之为 **Joinpoint，最常见的 Joinpoint 就是方法调用（即具体的功能模块）**。
>
> Pointcut（切点）：用于指定一组 Joinpoint，代表要在这一组 Joinpoint 中织入我们的逻辑，它定义了相应 Advice 将要发生的地方。**通常使用正则表达式来表示**。对于上面的例子，Pointcut 就是表示 “所有要加入日志记录的接口” 的一个 “表达式”。例如：“execution(* com.joonwhee.open.demo.service..*.*(..))”。
>
> Advice（通知/增强）：**Advice 定义了将会织入到 Joinpoint 的具体逻辑**，通过 @Before、@After、@Around 来区别在 JointPoint 之前、之后还是环绕执行的代码。
>
> Aspect（切面）：Aspect 是对系统中的横切关注点逻辑进行模块化**封装的 AOP 概念实体**。类似于 Java 中的类声明，在 Aspect 中可以包含多个 Pointcut 以及相关的 Advice 定义。
>
> Weaving（织入）：织入指的是将 Advice 连接到 Pointcut 指定的 Joinpoint 处的过程，也称为：将 Advice 织入到 Pointcut 指定的 Joinpoint 处。
>
> Target（目标对象）：符合 Pointcut 所指定的条件，被织入 Advice 的对象。

**目标对象（target）**

目标对象指将要被增强的对象

**连接点（JoinPoinit）**

程序执行过程中明确的点

连接点由两个信息确定：

* 方法（表示程序执行点）
* 相对点（表示方位）

简单来说连接点就是**被拦截到的程序执行点**，因为Spring只支持方法类型的连接点，所以再Spring中连接点就是**被拦截的方法**

**代理对象（Proxy）**

AOP中会通过代理的方式，对目标对象生成一个代理对象，代理对象中需要加入增强功能，通过代理对象来间接的方式起到增强目标对象的效果

**通知（Advice）**

需要再目标对象中增强的方法

**切入点（Pointcut）**

**用来指定需要将通知使用到那些地方**，切入点就是做这个配置的

**切面（Aspect）**

通知和切入点的组合，切面来定义再那些地方执行什么方法

**顾问（Advisor）**

Advisor其实是Pointcut和Advice的组合，Advice是要增强的逻辑，而增强的逻辑要再什么地方执行是通过Pointcut来指定的，所以Advice必须与Pointcut组合在一起

## SpringAOP中对应的实现

### 连接点（JoinPoint）

**JoinPoint接口**

```java
package org.aopalliance.intercept;
 
public interface Joinpoint {
 
    /**
     * 转到拦截器链中的下一个拦截器
     */
    Object proceed() throws Throwable;
 
    /**
     * 返回保存当前连接点静态部分【的对象】，这里一般指被代理的目标对象
     */
    Object getThis();
 
    /**
     * 返回此静态连接点  一般就为当前的Method(至少目前的唯一实现是MethodInvocation,所以连接点得静态部分肯定就是本方法)
     */
    AccessibleObject getStaticPart();
 
}
```

![JoinPoint重要实现类](Picture\Spring\JoinPoint重要实现类.png)

**Invocation接口**

此接口表示程序中的调用，调用是一个连接点，可以被拦截器拦截

```java
package org.aopalliance.intercept;
 
/**
 * 此接口表示程序中的调用
 * 调用是一个连接点，可以被拦截器拦截。
 */
public interface Invocation extends Joinpoint {
 
    /**
     * 将参数作为数组对象获取。可以更改此数组中的元素值以更改参数。
     * 通常用来获取调用目标方法的参数
     */
    Object[] getArguments();
 
}
```

**MethodInvocation接口**

用来表示连接点中方法的调用，可以获取调用过程中的目标方法

```java
package org.aopalliance.intercept;
 
import java.lang.reflect.Method;
 
/**
 * 方法调用的描述，在方法调用时提供给拦截器。
 * 方法调用是一个连接点，可以被方法拦截器拦截。
 */
public interface MethodInvocation extends Invocation {
 
    /**
     * 返回正在被调用得方法~~~  返回的是当前Method对象。
     * 此时，效果同父类的AccessibleObject getStaticPart() 这个方法
     */
    Method getMethod();
 
}
```

**ProxyMethodInvocation接口**

表示代理方法的调用

```java
public interface ProxyMethodInvocation extends MethodInvocation {
 
    /**
     * 获取被调用的代理对象
     */
    Object getProxy();
 
    /**
     * 克隆一个方法调用器MethodInvocation
     */
    MethodInvocation invocableClone();
 
    /**
     * 克隆一个方法调用器MethodInvocation，并为方法调用器指定参数
     */
    MethodInvocation invocableClone(Object... arguments);
 
    /**
     * 设置要用于此链中任何通知的后续调用的参数。
     */
    void setArguments(Object... arguments);
 
    /**
     * 添加一些扩展用户属性，这些属性不在AOP框架内使用。它们只是作为调用对象的一部分保留，用于特殊的拦截器。
     */
    void setUserAttribute(String key, @Nullable Object value);
 
    /**
     * 根据key获取对应的用户属性
     */
    @Nullable
    Object getUserAttribute(String key);
 
}
```

* **ReflectiveMethodInvocation**

  当代理对象是采用JDK动态代理创建，通过代理对象来访问目标对象的方法的时候，最终过程是由ReflectiveMethodInvocation来处理的，内部会通过递归调用方法拦截器，最终会调用到目标方法

* **CglibMethodInvocation**

  功能和上面的类似，当处理对象是采用CGLIB创建的，通过代理对象来访问目标对象的方法，最终过程是由CglibMethodInvocation来处理的，内部会通过递归调用方法拦截器，最终会调用到目标方法

### 通知（Advice）

通知的顶层接口，这个接口内部没有定义任何方法

```java
package org.aopalliance.aop;
 
public interface Advice {
}
```

![Advice重要实现类](Picture\Spring\Advice重要实现类.png)

**Advice接口**

```java
package org.aopalliance.aop;
public interface Advice {
}
```

**BeforeAdvice接口**

```java
package org.springframework.aop;
 
public interface BeforeAdvice extends Advice {
}
```

**Interceptor接口**

```java
package org.aopalliance.intercept;
 
public interface Interceptor extends Advice {
}
```

**MethodBeforeAdvice接口**

方法执行前通知，需要在目标方法执行前一些逻辑的，可以通过这个实现

通俗点：需要在执行方法执行之前增强一些逻辑，可以通过这个接口来实现，before方法：在调用给定方法之前回调

```java
package org.springframework.aop;
 
public interface MethodBeforeAdvice extends BeforeAdvice {
 
    /**
     * 调用目标方法之前会先调用这个before方法
     * method：需要执行的目标方法
     * args：目标方法的参数
     * target：目标对象
     */
    void before(Method method, Object[] args, @Nullable Object target) throws Throwable;
}
```

**AfterReturningAdvice接口**

方法执行通知，需要在目标方法执行之后增强一些逻辑的，可以通过这个实现

**注意：目标方法正常执行后，才会回调这个接口，当目标方法有异常，那么这通知会跳过**

```java
package org.springframework.aop;
 
public interface AfterReturningAdvice extends AfterAdvice {
 
    /**
     * 目标方法执行之后会回调这个方法
     * method：需要执行的目标方法
     * args：目标方法的参数
     * target：目标对象
     */
    void afterReturning(@Nullable Object returnValue, Method method, Object[] args, @Nullable Object target) throws Throwable;
 
}
```

**ThrowsAdvice接口**

```java
package org.springframework.aop;
 
public interface ThrowsAdvice extends AfterAdvice {
 
}
```

此接口上没有任何方法，因为方法由反射调用，实现类必须实现以下形式的方法，前3个参数是可选的，最后一个参数为需要匹配的异常类型

```java
void afterThrowing([Method, args, target], ThrowableSubclass);
```

**MethodInterceptor接口**

方法拦截器，这个接口最强大，可以实现上面3中类型的通知，上面3种通知最终都通过适配模式将其转换为MethodInterceptor方式去执行

```java
package org.aopalliance.intercept;
 
@FunctionalInterface
public interface MethodInterceptor extends Interceptor {
 
    /**
     * 拦截目标方法的执行，可以在这个方法内部实现需要增强的逻辑，以及主动调用目标方法
     */
    Object invoke(MethodInvocation invocation) throws Throwable;
 
}
```

> 拦截器链
>
> 一个目标方法中可以添加很多Advice，这些Advice最终都会被转换为MethodInterceptor类型的方法拦截器，最终会有多个MethodInterceptor，这些MethodInterceptor会组成一个方法调用链
>
> Aop内部会给目标对象创建一个代理，代理对象中会放入这些MethodInterceptor会组成一个方法调用链，当调用代理对象的方法时，会按顺序执行这些方法调用链，最后会通过反射再去调用目标方法，进而对目标方法进行增强

**通知包装器**

**负责将各种非MethodInterceptor类型的通知（Advice）包装为MethodInterceptor类型**

**Aop中所有Advice最终都会转换为MethodInterceptor类型的，组成一个方法调用链**

3个包装器类

* MethodBeforeAdviceInterceptor
* AfterReturningAdviceInterceptor
* ThrowsAdviceInterceptor

**MethodBeforeAdviceInterceptor类**

这个类实现了MethodInterceptor接口，负责将MethodBeforeAdvice方法前置通知包装成MethodInterceptor类型，创建这个类型的对象的时候需要传递一个MethodBeforeAdvice类型的参数，**重点是invoke方法**

```java
package org.springframework.aop.framework.adapter;
 
@SuppressWarnings("serial")
public class MethodBeforeAdviceInterceptor implements MethodInterceptor, BeforeAdvice, Serializable {
 
    private final MethodBeforeAdvice advice;
 
    public MethodBeforeAdviceInterceptor(MethodBeforeAdvice advice) {
        Assert.notNull(advice, "Advice must not be null");
        this.advice = advice;
    }
 
 
    @Override
    public Object invoke(MethodInvocation mi) throws Throwable {
        //负责调用前置通知的方法
        this.advice.before(mi.getMethod(), mi.getArguments(), mi.getThis());
        //继续执行方法调用链
        return mi.proceed();
    }
 
}
```



### 切入点（PointCut）

通知（Advice）用来指定需要增强的逻辑，切入点用来配置在哪里增强

**PointCut接口**

```java
package org.springframework.aop;
 
public interface Pointcut {
 
    /**
     * 类过滤器, 可以知道哪些类需要拦截
     */
    ClassFilter getClassFilter();
 
    /**
     * 方法匹配器, 可以知道哪些方法需要拦截
     */
    MethodMatcher getMethodMatcher();
 
    /**
     * 匹配所有对象的 Pointcut，内部的2个过滤器默认都会返回true
     */
    Pointcut TRUE = TruePointcut.INSTANCE;
 
}
```

**ClassFilter接口**

用来过滤类

```java
@FunctionalInterface
public interface ClassFilter {
 
    /**
     * 用来判断目标类型是否匹配
     */
    boolean matches(Class<?> clazz);
 
}
```

**MethodMatcher接口**

用来过滤方法的，第三个方法可以对方法的参数进行校验

```java
public interface MethodMatcher {
 
    /**
     * 执行静态检查给定方法是否匹配
     * @param method 目标方法
     * @param targetClass 目标对象类型
     */
    boolean matches(Method method, Class<?> targetClass);
 
    /**
     * 是否是动态匹配，即是否每次执行目标方法的时候都去验证一下
     */
    boolean isRuntime();
 
    /**
     * 动态匹配验证的方法，比第一个matches方法多了一个参数args，这个参数是调用目标方法传入的参数
     */
    boolean matches(Method method, Class<?> targetClass, Object... args);
 
 
    /**
     * 匹配所有方法，这个内部的2个matches方法任何时候都返回true
     */
    MethodMatcher TRUE = TrueMethodMatcher.INSTANCE;
 
}
```

### 顾问（Advisor）

通知定义了需要做什么，切入点定义了那些类的哪些方法中执行通知

**Advisor接口**

```java
package org.springframework.aop;
 
import org.aopalliance.aop.Advice;
 
/**
 * 包含AOP通知（在joinpoint处执行的操作）和确定通知适用性的过滤器（如切入点[PointCut]）的基本接口。
 * 这个接口不是供Spring用户使用的，而是为了支持不同类型的建议的通用性。
 */
public interface Advisor {
    /**
     * 返回引用的通知
     */
    Advice getAdvice();
 
}
```

![Advisor重要实现类](Picture\Spring\Advisor重要实现类.png)

**PointcutAdvisor接口**

这个和Pointcut有关，内部有个方法来获取Pointcut，AOP大部分都用这个类型

在目标方法中实现各种增强功能基本上都通过PointcutAdvisor来实现

```java
package org.springframework.aop;
 
/**
 * 切入点类型的Advisor
 */
public interface PointcutAdvisor extends Advisor {
 
    /**
     * 获取顾问中使用的切入点
     */
    Pointcut getPointcut();
 
}
```

**IntroductionAdvisor接口**

一个Java类，没有实现A接口，在不修改Java类的情况下，使其具备A接口的共能，可以通过IntroductionAdvisor给目标类引入更多接口功能

## 案例

```java
public class Test1 {
    @Test
    public void test1() {
        UserService target = new UserService();

        Pointcut pointcut = new Pointcut() {

            @Override
            public ClassFilter getClassFilter() {
                return UserService.class::isAssignableFrom;
            }

            @Override
            public MethodMatcher getMethodMatcher() {
                return new MethodMatcher() {
                    @Override
                    public boolean matches(Method method, Class<?> targetClass) {
                        return "work".equals(method.getName());
                    }

                    @Override
                    public boolean isRuntime() {
                        return true;
                    }

                    @Override
                    public boolean matches(Method method, Class<?> targetClass, Object... args) {
                        if (args.length == 1) {
                            String username = (String) args[0];
                            return username.contains("gf");
                        }
                        return false;
                    }
                };
            }
        };

        MethodBeforeAdvice advice = ((method, args, target1) -> System.out.println("你好" + args[0]));

        DefaultPointcutAdvisor advisor = new DefaultPointcutAdvisor(pointcut, advice);

        ProxyFactory proxyFactory = new ProxyFactory();
        proxyFactory.setTarget(target);
        proxyFactory.addAdvisor(advisor);
        UserService userServiceProxy = (UserService) proxyFactory.getProxy();

        userServiceProxy.work("ysy");
    }
}
```

# AOP原理详解

## 创建代理3大步骤

1. 创建代理所需参数配置
2. 根据代理参数获取AopProxy对象
3. 通过AopProxy获取代理对象

## 创建代理所需参数配置

创建代理所需参数配置主要通过AdvisedSupport这个类实现的

![AdvisedSupport相关接口和类](Picture\Spring\AdvisedSupport.png)

**TargetClassAware接口**

定义了一个方法，用来获取目标对象类型

所谓目标对象，就是被代理对象

```java
package org.springframework.aop;
 
public interface TargetClassAware {
    @Nullable
    Class<?> getTargetClass();
}
```

**Advised接口**

这个接口定义了操作Aop代理配置的各种方法

所以由SpringAop创建的代理对象默认都实现这个接口

```java
public interface Advised extends TargetClassAware {
 
    /**
     * 返回配置是否已冻结，被冻结之后，无法修改Advisor
     */
    boolean isFrozen();
 
    /**
     * 如果是通过cglib创建代理，此方法返回true，否则返回false
     */
    boolean isProxyTargetClass();
 
    /**
     * 获取配置中需要代理的接口列表
     */
    Class<?>[] getProxiedInterfaces();
 
    /**
     * 判断某个接口是否被代理
     */
    boolean isInterfaceProxied(Class<?> intf);
 
    /**
     * 设置被代理的目标源，创建代理的时候，通常需要传入被代理的对象，最终被代理的对象会被包装为TargetSource类型的
     */
    void setTargetSource(TargetSource targetSource);
 
    /**
     * 返回被代理的目标源
     */
    TargetSource getTargetSource();
 
    /**
     * 设置是否需要将代理暴露在ThreadLocal中，这样可以在线程中获取到被代理对象
     */
    void setExposeProxy(boolean exposeProxy);
 
    /**
     * 返回代理是否被暴露再ThreadLocal中
     */
    boolean isExposeProxy();
 
    /**
     * 设置此代理配置是否经过预筛选
     * 默认设置是“假”。如果已经对advisor进行了预先筛选，则将其设置为“true”
     * 这意味着在为代理调用构建实际的advisor链时可以跳过ClassFilter检查。
     */
    void setPreFiltered(boolean preFiltered);
 
    /**
     * 返回preFiltered
     */
    boolean isPreFiltered();
 
    /**
     * 返回代理配置中所有Advisor的列表
     */
    Advisor[] getAdvisors();
 
    /**
     * 添加一个Advisor
     */
    void addAdvisor(Advisor advisor) throws AopConfigException;
 
    /**
     * 指定的位置添加一个Advisor
     */
    void addAdvisor(int pos, Advisor advisor) throws AopConfigException;
 
    /**
     * 移除一个Advisor
     */
    boolean removeAdvisor(Advisor advisor);
 
    /**
     * 移除指定位置的Advisor
     */
    void removeAdvisor(int index) throws AopConfigException;
 
    /**
     * 查找某个Advisor的位置
     */
    int indexOf(Advisor advisor);
 
    /**
     * 对advisor列表中的a替换为b
     */
    boolean replaceAdvisor(Advisor a, Advisor b) throws AopConfigException;
 
    /**
     * 添加一个通知
     */
    void addAdvice(Advice advice) throws AopConfigException;
 
    /**
     * 向指定的位置添加一个通知
     */
    void addAdvice(int pos, Advice advice) throws AopConfigException;
 
    /**
     * 移除一个通知
     */
    boolean removeAdvice(Advice advice);
 
    /**
     * 获取通知的位置
     */
    int indexOf(Advice advice);
 
    /**
     * 将代理配置转换为字符串，这个方便排错和调试使用的
     */
    String toProxyConfigString();
 
}
```

**ProxyConfig类**

```java
package org.springframework.aop.framework;
 
/**
 * 对外提供统一的代理参数配置类，以确保所有代理创建程序具有一致的属性
 */
public class ProxyConfig implements Serializable {
 
    // 标记是否直接对目标类进行代理，而不是通过接口产生代理
    private boolean proxyTargetClass = false;
 
    // 标记是否对代理进行优化。启动优化通常意味着在代理对象被创建后，增强的修改将不会生效，因此默认值为false。
    // 如果exposeProxy设置为true，即使optimize为true也会被忽略。
    private boolean optimize = false;
 
    // 标记是否需要阻止通过该配置创建的代理对象转换为Advised类型，默认值为false，表示代理对象可以被转换为Advised类型
    boolean opaque = false;
 
    // 标记代理对象是否应该被aop框架通过AopContext以ThreadLocal的形式暴露出去。
    // 当一个代理对象需要调用它自己的另外一个代理方法时，这个属性将非常有用。默认是是false，以避免不必要的拦截。
    boolean exposeProxy = false;
 
    // 标记该配置是否需要被冻结，如果被冻结，将不可以修改增强的配置。
    // 当我们不希望调用方修改转换成Advised对象之后的代理对象时，这个配置将非常有用。
    private boolean frozen = false;
 
 
    //省略了属性的get set方法
}
```

**AdvisedSupport类**

这个类是重点，AOP代理配置管理器的基类，继承了ProxyConfig并且实现了Adviced接口，创建aop代理之前，所有需要配置的信息都是通过这个类来操作的

```java
package org.springframework.aop.framework;
 
public class AdvisedSupport extends ProxyConfig implements Advised {
 
    public static final TargetSource EMPTY_TARGET_SOURCE = EmptyTargetSource.INSTANCE;
 
    TargetSource targetSource = EMPTY_TARGET_SOURCE;
 
    /** 不支持预过滤 */
    private boolean preFiltered = false;
 
    /** 调用链工厂，用来获取目标方法的调用链 */
    AdvisorChainFactory advisorChainFactory = new DefaultAdvisorChainFactory();
 
    /** 方法调用链缓存：以方法为键，以顾问链表为值的缓存。 */
    private transient Map<MethodCacheKey, List<Object>> methodCache;
 
    //代理对象需要实现的接口列表。保存在列表中以保持注册的顺序，以创建具有指定接口顺序的JDK代理。
    private List<Class<?>> interfaces = new ArrayList<>();
 
    //配置的顾问列表。所有添加的Advise对象都会被包装为Advisor对象
    private List<Advisor> advisors = new ArrayList<>();
 
    //数组更新了对advisor列表的更改，这更容易在内部操作。
    private Advisor[] advisorArray = new Advisor[0];
 
 
    //无参构造方法
    public AdvisedSupport() {
        this.methodCache = new ConcurrentHashMap<>(32);
    }
 
    //有参构造方法，参数为：代理需要实现的接口列表
    public AdvisedSupport(Class<?>... interfaces) {
        this();
        setInterfaces(interfaces);
    }
 
    //设置需要被代理的目标对象，目标对象会被包装为TargetSource格式的对象
    public void setTarget(Object target) {
        setTargetSource(new SingletonTargetSource(target));
    }
 
    //设置被代理的目标源
    @Override
    public void setTargetSource(@Nullable TargetSource targetSource) {
        this.targetSource = (targetSource != null ? targetSource : EMPTY_TARGET_SOURCE);
    }
 
    //获取被代理的目标源
    @Override
    public TargetSource getTargetSource() {
        return this.targetSource;
    }
 
    //设置被代理的目标类
    public void setTargetClass(@Nullable Class<?> targetClass) {
        this.targetSource = EmptyTargetSource.forClass(targetClass);
    }
 
    //获取被代理的目标类型
    @Override
    @Nullable
    public Class<?> getTargetClass() {
        return this.targetSource.getTargetClass();
    }
 
    /** 
     * 设置此代理配置是否经过预筛选，通过目标方法调用代理的时候，
     * 需要通过匹配的方式获取这个方法上的调用链列表，查找过程需要2个步骤：
     * 第一步：类是否匹配，第二步：方法是否匹配，当这个属性为true的时候，会直接跳过第一步
     */
    @Override
    public void setPreFiltered(boolean preFiltered) {
        this.preFiltered = preFiltered;
    }
 
    // 返回preFiltered
    @Override
    public boolean isPreFiltered() {
        return this.preFiltered;
    }
 
    /**
     * 设置顾问链工厂，当调用目标方法的时候，需要获取这个方法上匹配的Advisor列表，
     * 获取目标方法上匹配的Advisor列表的功能就是AdvisorChainFactory来负责的
     */
    public void setAdvisorChainFactory(AdvisorChainFactory advisorChainFactory) {
        Assert.notNull(advisorChainFactory, "AdvisorChainFactory must not be null");
        this.advisorChainFactory = advisorChainFactory;
    }
 
    // 返回顾问链工厂对象
    public AdvisorChainFactory getAdvisorChainFactory() {
        return this.advisorChainFactory;
    }
 
 
    //设置代理对象需要实现的接口
    public void setInterfaces(Class<?>... interfaces) {
        Assert.notNull(interfaces, "Interfaces must not be null");
        this.interfaces.clear();
        for (Class<?> ifc : interfaces) {
            addInterface(ifc);
        }
    }
 
    //为代理对象添加需要实现的接口
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
 
    //移除代理对象需要实现的接口
    public boolean removeInterface(Class<?> intf) {
        return this.interfaces.remove(intf);
    }
 
    //获取代理对象需要实现的接口列表
    @Override
    public Class<?>[] getProxiedInterfaces() {
        return ClassUtils.toClassArray(this.interfaces);
    }
 
    //判断代理对象是否需要实现某个接口
    @Override
    public boolean isInterfaceProxied(Class<?> intf) {
        for (Class<?> proxyIntf : this.interfaces) {
            if (intf.isAssignableFrom(proxyIntf)) {
                return true;
            }
        }
        return false;
    }
 
    //获取配置的所有顾问列表
    @Override
    public final Advisor[] getAdvisors() {
        return this.advisorArray;
    }
 
    //添加顾问
    @Override
    public void addAdvisor(Advisor advisor) {
        int pos = this.advisors.size();
        addAdvisor(pos, advisor);
    }
 
    //指定的位置添加顾问
    @Override
    public void addAdvisor(int pos, Advisor advisor) throws AopConfigException {
        if (advisor instanceof IntroductionAdvisor) {
            validateIntroductionAdvisor((IntroductionAdvisor) advisor);
        }
        addAdvisorInternal(pos, advisor);
    }
 
    //移除指定的顾问
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
 
    //移除指定位置的顾问
    @Override
    public void removeAdvisor(int index) throws AopConfigException {
        //当配置如果是冻结状态，是不允许对顾问进行修改的，否则会抛出异常
        if (isFrozen()) {
            throw new AopConfigException("Cannot remove Advisor: Configuration is frozen.");
        }
        if (index < 0 || index > this.advisors.size() - 1) {
            throw new AopConfigException("Advisor index " + index + " is out of bounds: " +
                    "This configuration only has " + this.advisors.size() + " advisors.");
        }
        //移除advisors中的顾问
        Advisor advisor = this.advisors.remove(index);
        if (advisor instanceof IntroductionAdvisor) {
            IntroductionAdvisor ia = (IntroductionAdvisor) advisor;
            // We need to remove introduction interfaces.
            for (Class<?> ifc : ia.getInterfaces()) {
                removeInterface(ifc);
            }
        }
        //更新advisorArray
        updateAdvisorArray();
        //通知已改变，内部会清除方法调用链缓存信息。
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
 
    //批量添加顾问
    public void addAdvisors(Advisor... advisors) {
        addAdvisors(Arrays.asList(advisors));
    }
 
    //批量添加顾问
    public void addAdvisors(Collection<Advisor> advisors) {
        //配置如果是冻结状态，会抛出异常
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
 
    //此方法先忽略，用来为目标类引入接口的
    private void validateIntroductionAdvisor(IntroductionAdvisor advisor) {
        advisor.validateInterfaces();
        // If the advisor passed validation, we can make the change.
        Class<?>[] ifcs = advisor.getInterfaces();
        for (Class<?> ifc : ifcs) {
            addInterface(ifc);
        }
    }
 
    //指定的位置添加顾问
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
 
    //将advisorArray和advisors保持一致
    protected final void updateAdvisorArray() {
        this.advisorArray = this.advisors.toArray(new Advisor[0]);
    }
 
    //获取顾问列表
    protected final List<Advisor> getAdvisorsInternal() {
        return this.advisors;
    }
 
    //添加通知
    @Override
    public void addAdvice(Advice advice) throws AopConfigException {
        int pos = this.advisors.size();
        addAdvice(pos, advice);
    }
 
    //指定的位置添加通知
    @Override
    public void addAdvice(int pos, Advice advice) throws AopConfigException {
        //此处会将advice通知包装为DefaultPointcutAdvisor类型的Advisor
        addAdvisor(pos, new DefaultPointcutAdvisor(advice));
    }
 
    //移除通知
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
 
    //获取通知的位置
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
 
    //是否包含某个通知
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
 
    //获取当前配置中某种类型通知的数量
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
 
 
    //基于当前配置，获取给定方法的方法调用链列表
    public List<Object> getInterceptorsAndDynamicInterceptionAdvice(Method method, @Nullable Class<?> targetClass) {
        MethodCacheKey cacheKey = new MethodCacheKey(method);
        //先从缓存中获取
        List<Object> cached = this.methodCache.get(cacheKey);
        //缓存中没有时，从advisorChainFactory中获取
        if (cached == null) {
            cached = this.advisorChainFactory.getInterceptorsAndDynamicInterceptionAdvice(
                    this, method, targetClass);
            this.methodCache.put(cacheKey, cached);
        }
        return cached;
    }
 
    //通知更改时调用，会清空当前方法调用链缓存
    protected void adviceChanged() {
        this.methodCache.clear();
    }
 
    //将other中的配置信息复制到当前对象中
    protected void copyConfigurationFrom(AdvisedSupport other) {
        copyConfigurationFrom(other, other.targetSource, new ArrayList<>(other.advisors));
    }
 
    //将other中的配置信息复制到当前对象中
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
 
    //构建此AdvisedSupport的仅配置副本，替换TargetSource。
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
}
```

**结论**

1. 配置中添加的Advice对象最终都会被转换为DefaultPointcutAdvisor对象，此时DefaultPointcutAdvisor未指定pointcut，默认会匹配任意类的任意方法
2. 当配置被冻结的时候，此时配置中的Advisor列表是不允许修改的
3. 上面的getInterceptorAndSynamicInterceptionAdvice方法，通过代理调用目标方法的时候，最后需要通过方法和目标类的类型，从当前配置中会后去匹配的方法拦截器列表，获取方法拦截器列表是由AdvisorChainFactory负责的
4. 目标方法和其关联的方法拦截器列表会缓存在methodCache中，当顾问列表有变化的时候methodCache缓存会被清除

## 根据配置获取AopProxy

此阶段对应代码

```java
// 创建AopProxy使用了简单工厂模式
AopProxyFactory aopProxyFactory = new DefaultAopProxyFactory();
//通过AopProxy工厂获取AopProxy对象
AopProxy aopProxy = aopProxyFactory.createAopProxy(advisedSupport);
```

此阶段会根据AdvisedSupport中配置信息，判断具体是采用CGLIB的方式还是采用jdk动态代理的方式获取代理对象

![AopProxy相关类](Picture\Spring\根据配置获取AopProxy.png)

**AopProxy接口**

这个接口定义了一个方法，用来创建最终的代理对象

* CglibAopProxy：采用cglib的方式创建代理对象
* JdkDynamicAopProxy：采用jdk动态代理的方式创建代理对象

```java
package org.springframework.aop.framework;
 
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

**AopProxyFactory接口**

通过名称就可以看出来，是一个工厂，负责创建AopProxy，使用的是简单工厂模式

接口中定义了一个方法，会根据Aop的配置信息AdvisedSupport来获取AopProxy对象，主要是判断采用cglib的方式还是采用jdk动态代理的方式

```java
package org.springframework.aop.framework;
 
public interface AopProxyFactory {
    /**
     * 根据aop配置信息获取AopProxy对象
     */
    AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException;
}
```

**DefaultAopProxyFactory类**

```java
package org.springframework.aop.framework;
 
/**
 * 默认AopProxyFactory实现，创建CGLIB代理或JDK动态代理。
 * 对于给定的AdvisedSupport实例，以下条件为真，则创建一个CGLIB代理:
 * optimize = true
 * proxyTargetClass = true
 * 未指定代理接口
 * 通常，指定proxyTargetClass来强制执行CGLIB代理，或者指定一个或多个接口来使用JDK动态代理。
 */
public class DefaultAopProxyFactory implements AopProxyFactory, Serializable {
 
    @Override
    public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
        // optimize==true || proxyTargetClass 为true || 配置中没有需要代理的接口
        if (config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config)) {
            //获取需要被代理的类
            Class<?> targetClass = config.getTargetClass();
            if (targetClass == null) {
                throw new AopConfigException("TargetSource cannot determine target class: " +
                        "Either an interface or a target is required for proxy creation.");
            }
            //如果被代理的类为接口 或者 被代理的类是jdk动态代理创建代理类，则采用JdkDynamicAopProxy的方式，否则采用cglib代理的方式
            if (targetClass.isInterface() || Proxy.isProxyClass(targetClass)) {
                //采用jdk动态代理的方式
                return new JdkDynamicAopProxy(config);
            }
            //采用cglib代理的方式
            return new ObjenesisCglibAopProxy(config);
        }
        else {
            //采用jdk动态代理的方式
            return new JdkDynamicAopProxy(config);
        }
    }
 
    /**
     * 确定所提供的AdvisedSupport是否只指定了SpringProxy接口(或者根本没有指定代理接口)
     */
    private boolean hasNoUserSuppliedProxyInterfaces(AdvisedSupport config) {
        Class<?>[] ifcs = config.getProxiedInterfaces();
        return (ifcs.length == 0 || (ifcs.length == 1 && SpringProxy.class.isAssignableFrom(ifcs[0])));
    }
 
}
```

## 通过AopProxy获取代理对象

AopProxy.createAopProxy方法返回的结果

* JdkDynamicAopProxy：以jdk动态代理的方式创建代理
* ObjenesisCglibAopProxy：以cglib的方式创建动态代理

**JdkDynamicAopProxy类**

作用：采用jdk动态代理的方式创建代理对象，并处理代理对象的所有方法调用

```java
final class JdkDynamicAopProxy implements AopProxy, InvocationHandler, Serializable {
 
    //代理的配置信息
    private final AdvisedSupport advised;
 
    //需要被代理的接口中是否定义了equals方法
    private boolean equalsDefined;
 
    //需要被代理的接口中是否定义了hashCode方法
    private boolean hashCodeDefined;
 
    //通过AdvisedSupport创建实例
    public JdkDynamicAopProxy(AdvisedSupport config) throws AopConfigException {
        Assert.notNull(config, "AdvisedSupport must not be null");
        if (config.getAdvisors().length == 0 && config.getTargetSource() == AdvisedSupport.EMPTY_TARGET_SOURCE) {
            throw new AopConfigException("No advisors and no TargetSource specified");
        }
        this.advised = config;
    }
 
    //生成一个代理对象
    @Override
    public Object getProxy() {
        return getProxy(ClassUtils.getDefaultClassLoader());
    }
 
    //生成一个代理对象
    @Override
    public Object getProxy(@Nullable ClassLoader classLoader) {
        if (logger.isTraceEnabled()) {
            logger.trace("Creating JDK dynamic proxy: " + this.advised.getTargetSource());
        }
        //根据advised的信息获取代理需要被代理的所有接口列表
        Class<?>[] proxiedInterfaces = AopProxyUtils.completeProxiedInterfaces(this.advised, true); 
        //查找被代理的接口中是否定义了equals、hashCode方法
        findDefinedEqualsAndHashCodeMethods(proxiedInterfaces);
        /**
         * 这个大家应该很熟悉吧，通过jdk动态代理创建代理对象，注意最后一个参数是this
         * 表示当前类，当前类是InvocationHandler类型的，当调用代理对象的任何方法的时候
         * 都会被被当前类的 invoke 方法处理
         */
        return Proxy.newProxyInstance(classLoader, proxiedInterfaces, this);
    }
 
    //判断需要代理的接口中是否定义了这几个方法（equals、hashCode）
    private void findDefinedEqualsAndHashCodeMethods(Class<?>[] proxiedInterfaces) {
        for (Class<?> proxiedInterface : proxiedInterfaces) {
            //获取接口中定义的方法
            Method[] methods = proxiedInterface.getDeclaredMethods();
            for (Method method : methods) {
                //是否是equals方法
                if (AopUtils.isEqualsMethod(method)) {
                    this.equalsDefined = true;
                }
                //是否是hashCode方法
                if (AopUtils.isHashCodeMethod(method)) {
                    this.hashCodeDefined = true;
                }
                //如果发现这2个方法都定义了，结束循环查找
                if (this.equalsDefined && this.hashCodeDefined) {
                    return;
                }
            }
        }
    }
 
 
    // 这个方法比较关键了，当在程序中调用代理对象的任何方法，最终都会被下面这个invoke方法处理
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //旧的代理对象
        Object oldProxy = null;
        //用来标记是否需要将代理对象暴露在ThreadLocal中
        boolean setProxyContext = false;
        //获取目标源
        TargetSource targetSource = this.advised.targetSource;
        //目标对象
        Object target = null;
 
        //下面进入代理方法的处理阶段
        try {
            // 处理equals方法：被代理的接口中没有定义equals方法 && 当前调用是equals方法
            if (!this.equalsDefined && AopUtils.isEqualsMethod(method)) {
                // 直接调用当前类中的equals方法
                return equals(args[0]);
            }
            // 处理hashCode方法：被代理的接口中没有定义hashCode方法 && 当前调用是hashCode方法
            else if (!this.hashCodeDefined && AopUtils.isHashCodeMethod(method)) {
                // 直接调用当前类中的hashCode方法
                return hashCode();
            }
            /**
             * 方法来源于 DecoratingProxy 接口，这个接口中定义了一个方法
             * 用来获取原始的被代理的目标类，主要是用在嵌套代理的情况下（所谓嵌套代理：代理对象又被作为目标对象进行了代理）
             */
            else if (method.getDeclaringClass() == DecoratingProxy.class) {
                // 调用AopProxyUtils工具类的方法，内部通过循环遍历的方式，找到最原始的被代理的目标类
                return AopProxyUtils.ultimateTargetClass(this.advised);
            }
            // 方法来源于 Advised 接口，代理对象默认情况下会实现 Advised 接口，可以通过代理对象来动态向代理对象中添加通知等
            else if (!this.advised.opaque && method.getDeclaringClass().isInterface() &&
                    method.getDeclaringClass().isAssignableFrom(Advised.class)) {
                // this.advised是AdvisedSupport类型的，AdvisedSupport实现了Advised接口中的所有方法
                // 所以最终通过通过反射方式交给this.advised来响应当前调用
                return AopUtils.invokeJoinpointUsingReflection(this.advised, method, args);
            }
            // 用来记录方法返回值
            Object retVal;
 
            //是否需要在threadLocal中暴露代理对象
            if (this.advised.exposeProxy) {
                // 将代理对象暴露在上线文中，即暴露在threadLocal中，那么在当前线程中可以通过静态方法
                // AopContext#currentProxy获取当前被暴露的代理对象，这个是非常有用的，稍后用案例来讲解，瞬间就会明白
                oldProxy = AopContext.setCurrentProxy(proxy);
                // 将setProxyContext标记为true
                setProxyContext = true;
            }
 
            // 通过目标源获取目标对象
            target = targetSource.getTarget();
            // 获取目标对象类型
            Class<?> targetClass = (target != null ? target.getClass() : null);
 
            // 获取当前方法的拦截器链
            List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass); 
 
            // 拦截器链为空的情况下，表示这个方法上面没有找到任何增强的通知，那么会直接通过反射直接调用目标对象
            if (chain.isEmpty()) {
                // 获取方法请求的参数（有时候方法中有可变参数，所谓可变参数就是带有省略号(...)这种格式的参数，传入的参数类型和这种类型不一样的时候，会通过下面的adaptArgumentsIfNecessary方法进行转换）
                Object[] argsToUse = AopProxyUtils.adaptArgumentsIfNecessary(method, args);
                //通过反射直接调用目标方法
                retVal = AopUtils.invokeJoinpointUsingReflection(target, method, argsToUse);
            }
            else {
                // 创建一个方法调用器（包含了代理对象、目标对象、调用的方法、参数、目标类型、方法拦截器链）
                MethodInvocation invocation =
                        new ReflectiveMethodInvocation(proxy, target, method, args, targetClass, chain);
                // 通过拦截器链一个个调用最终到目标方法的调用
                retVal = invocation.proceed();
            }
 
            // 下面会根据方法返回值的类型，做一些处理，比如方法返回的类型为自己，则最后需要将返回值置为代理对象
            Class<?> returnType = method.getReturnType();
            if (retVal != null && retVal == target &&
                    returnType != Object.class && returnType.isInstance(proxy) &&
                    !RawTargetAccess.class.isAssignableFrom(method.getDeclaringClass())) {
                // 将返回值设置为代理对象
                retVal = proxy;
            }
            // 方法的返回值类型returnType为原始类型（即int、byte、double等这种类型的） && retVal为null，
            // 此时如果将null转换为原始类型会报错，所以此处直接抛出异常
            else if (retVal == null && returnType != Void.TYPE && returnType.isPrimitive()) {
                throw new AopInvocationException(
                        "Null return value from advice does not match primitive return type for: " + method);
            }
            // 返回方法调用结果
            return retVal;
        }
        finally {
            // 目标对象不为null && 目标源不是静态的
            //所谓静态的，你可以理解为是否是单例的
            // isStatic为true，表示目标对象是单例的，同一个代理对象中所有方法共享一个目标对象
            // isStatic为false的时候，通常每次调用代理的方法，target对象是不一样的，所以方法调用万之后需要进行释放，可能有些资源清理，连接的关闭等操作
            if (target != null && !targetSource.isStatic()) {
                // 必须释放来自TargetSource中的目标对象
                targetSource.releaseTarget(target);
            }
            // setProxyContext为ture
            if (setProxyContext) {
                // 需要将旧的代理再放回到上线文中
                AopContext.setCurrentProxy(oldProxy);
            }
        }
    }
 
}
```

**AopProxyUtils.completeProxiedInterfaces方法**

```java
static Class<?>[] completeProxiedInterfaces(AdvisedSupport advised, boolean decoratingProxy) {
    //获取代理配置中需要被代理的接口
    Class<?>[] specifiedInterfaces = advised.getProxiedInterfaces();
    // 需要被代理的接口数量为0
    if (specifiedInterfaces.length == 0) {
        // 获取需要被代理的目标类型
        Class<?> targetClass = advised.getTargetClass();
        //目标类型不为空
        if (targetClass != null) {
            //目标类型为接口
            if (targetClass.isInterface()) {
                //将其添加到需要代理的接口中
                advised.setInterfaces(targetClass);
            }
            // 目标类型为jdk动态代理创建的代理对象
            else if (Proxy.isProxyClass(targetClass)) {
                // 获取目标类型上的所有接口，将其添加到需要被代理的接口中
                advised.setInterfaces(targetClass.getInterfaces());
            }
            //再次获取代理配置中需要被代理的接口
            specifiedInterfaces = advised.getProxiedInterfaces();
        }
    }
    //判断SpringProxy接口是否已经在被代理的接口中
    boolean addSpringProxy = !advised.isInterfaceProxied(SpringProxy.class);
    //判断Advised接口是否已经在被代理的接口中
    boolean addAdvised = !advised.isOpaque() && !advised.isInterfaceProxied(Advised.class);
    //判断DecoratingProxy接口是否已经在被代理的接口中
    boolean addDecoratingProxy = (decoratingProxy && !advised.isInterfaceProxied(DecoratingProxy.class));
    //一个计数器，会根据上面三个boolean值做递增
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
   // 下面就是构建所有需要被代理的接口
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

最终创建出来的代理对象，默认会实现上面列出的所有接口，后面三个接口都是aop中自动添加的

**getInterceptorsAndDynamicInterceptionAdvice**

这个方法位于AdvisedSupport中，根据方法和目标类型获取方法上匹配的拦截器链

```java
public List<Object> getInterceptorsAndDynamicInterceptionAdvice(Method method, @Nullable Class<?> targetClass) {
    //会先尝试从还中获取，如果获取不到，会从advisorChainFactory中获取，然后将其丢到缓存中
    MethodCacheKey cacheKey = new MethodCacheKey(method);
    List<Object> cached = this.methodCache.get(cacheKey);
    if (cached == null) {
        cached = this.advisorChainFactory.getInterceptorsAndDynamicInterceptionAdvice(
            this, method, targetClass);
        this.methodCache.put(cacheKey, cached);
    }
    return cached;
}
```

**ReflectiveMethodInvocation.proceed**

这会调用拦截器链，最终会调用到目标方法，获得目标方法的返回值

**JdkDynamicAopProxy小结**

1. 被创建的代理对象默认会实现SpringProxy，Advised，DecoratingProxy3个接口
2. SpringProxy这个接口中没有任何方法，只是起一个标记作用，用来标记代理对象是使用SpringAop创建的
3. 代理对象默认都会实现Advised接口，所以可以通过这个接口动态变更代理对象中的通知
4. DecoratingProxy接口中定义了一个方法getDecoratedClass，用来获取被代理的原始目标对象的类型

**CglibAopProxy类**

作用：采用cglib代理的方式创建代理对象，并处理代理对象的所有方法调用

以getProxy方法为入口

**getProxy方法**

```java
public Object getProxy(@Nullable ClassLoader classLoader) {
    // 获取被代理的类
    Class<?> rootClass = this.advised.getTargetClass();
 
    // 代理对象的父类（cglib是采用继承的方式是创建代理对象的，所以将被代理的类作为代理对象的父类）
    Class<?> proxySuperClass = rootClass;
    // 判断被代理的类是不是cglib创建的类，如果是cblib创建的类，会将其父类作为被代理的类
    if (rootClass.getName().contains(ClassUtils.CGLIB_CLASS_SEPARATOR)) {
        proxySuperClass = rootClass.getSuperclass();
        //添加需要被代理的接口
        Class<?>[] additionalInterfaces = rootClass.getInterfaces();
        for (Class<?> additionalInterface : additionalInterfaces) {
            this.advised.addInterface(additionalInterface);
        }
    }
 
    // 开始cglib创建代理，这个大家对cglib比较熟悉的一看就懂
    Enhancer enhancer = createEnhancer();
    // 设置被代理的父类
    enhancer.setSuperclass(proxySuperClass);
    // 设置被代理的接口[开发者硬编码指定的需要被代理的接口列表,SpringProxy,Advised]，这个比jdk动态代理的方式少了一个DecoratingProxy接口
    enhancer.setInterfaces(AopProxyUtils.completeProxiedInterfaces(this.advised));
    // 设置代理类类名生成策略
    enhancer.setNamingPolicy(SpringNamingPolicy.INSTANCE);
    // 设置字节码的生成策略
    enhancer.setStrategy(new ClassLoaderAwareGeneratorStrategy(classLoader));
 
    // 获取Callback列表
    Callback[] callbacks = getCallbacks(rootClass);
    Class<?>[] types = new Class<?>[callbacks.length];
    for (int x = 0; x < types.length; x++) {
        types[x] = callbacks[x].getClass();
    }
    // 设置CallbackFilter，CallbackFilter内部会判断被代理对象中的方法最终会被callbacks列表中的那个Callback来处理
    enhancer.setCallbackFilter(new ProxyCallbackFilter(
        this.advised.getConfigurationOnlyCopy(), this.fixedInterceptorMap, this.fixedInterceptorOffset));
    enhancer.setCallbackTypes(types);
 
    // 获取代理对象（内部会先创建代理类，然后会根据代理类生成一个代理对象）
    return createProxyClassAndInstance(enhancer, callbacks);
}
```

**getCallbacks方法**

通过被代理的类来获取Callback列表，Callback是用来处理代理对象的方法调用的，代理对象中可能有很多方法，没饿过方法可能采用不同的处理方式，所以会有多个Callback

```java
private Callback[] getCallbacks(Class<?> rootClass) throws Exception {
    // 是否需要将代理暴露在threadLocal中
    boolean exposeProxy = this.advised.isExposeProxy();
    // 配置是否是冻结的
    boolean isFrozen = this.advised.isFrozen();
    // 被代理的目标对象是否是动态的（是否是单例的）
    boolean isStatic = this.advised.getTargetSource().isStatic();
 
    // 当方法上有需要执行的拦截器的时候，会用这个来处理
    Callback aopInterceptor = new DynamicAdvisedInterceptor(this.advised);
 
    // 当方法上没有需要执行的拦截器的时候，会使用targetInterceptor来处理，内部会通过反射直接调用目标对象的方法
    Callback targetInterceptor;
    /**
     * 这块根据是否需要暴露代理到threadLocal中以及目标对象是否是动态的，会创建不同的Callback
     * isStatic为true的时候，同一个代理的不同方法可能都是新的目标对象，所以当代理方法执行完毕之后，需要对目标对象进行释放
     */
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
 
    // targetDispatcher会直接调用目标方法
    Callback targetDispatcher = (isStatic ?
                                 new StaticDispatcher(this.advised.getTargetSource().getTarget()) : new SerializableNoOp());
 
    Callback[] mainCallbacks = new Callback[] {
        aopInterceptor,  // 处理匹配到拦截器的方法
        targetInterceptor,  // 处理未匹配到拦截器的方法
        new SerializableNoOp(), 
        targetDispatcher,  // 处理未匹配到拦截器的方法，和targetInterceptor有何不同呢？目标方法如果返回值的结果是目标对象类型的，会使用 targetInterceptor 处理，内部会返回代理对象
        this.advisedDispatcher, // 处理Advised接口中定义的方法
        new EqualsInterceptor(this.advised), // 处理equals方法
        new HashCodeInterceptor(this.advised) // 处理hashCode方法
    };
 
    Callback[] callbacks;
 
    // 如果被代理的对象是单例的 && 配置是冻结的，此时会进行优化，怎么优化呢？
    // 配置冻结的情况下，生成好的代理中通知是无法修改的，所以可以提前将每个方法对应的拦截器链找到给缓存起来
    // 调用方法的时候，就直接从缓存中可以拿到方法对应的缓存信息，效率会高一些
    if (isStatic && isFrozen) {
        Method[] methods = rootClass.getMethods();
        Callback[] fixedCallbacks = new Callback[methods.length];
        this.fixedInterceptorMap = new HashMap<>(methods.length);
 
        // 获取每个方法的调用链，然后给缓存在fixedInterceptorMap中
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

getCallbacks中主要设计5个类

* DynamicAdvisedInterceptor
* StaticUnadvisedExposedInterceptor
* StaticUnadvisedInterceptor
* DynamicUnadvisedInterceptor
* StaticDispatcher

**DynamicAdvisedInterceptor类**

```java
private static class DynamicAdvisedInterceptor implements MethodInterceptor, Serializable {
    //代理配置信息
    private final AdvisedSupport advised;
    //构造器，需要一个AdvisedSupport
    public DynamicAdvisedInterceptor(AdvisedSupport advised) {
        this.advised = advised;
    }
 
    //这个方法是关键，用来处理代理对象中方法的调用
    public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        //被暴露在threadLocal中旧的代理对象
        Object oldProxy = null;
        //用来标记代理对象是否被暴露在threadLocal中
        boolean setProxyContext = false;
        //目标对象
        Object target = null;
        //目标源
        TargetSource targetSource = this.advised.getTargetSource();
        try {
            //代理配置中是否需要将代理暴露在threadLocal中
            if (this.advised.exposeProxy) {
                //将代理对象暴露出去
                oldProxy = AopContext.setCurrentProxy(proxy);
                //将setProxyContext置为true
                setProxyContext = true;
            }
            //获取目标对象（即被代理的对象）
            target = targetSource.getTarget();
            Class<?> targetClass = (target != null ? target.getClass() : null);
            //@1：获取当前方法的拦截器链
            List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
            //记录方法返回值
            Object retVal;
            //拦截器链不为空 && 方法是public类型的
            if (chain.isEmpty() && Modifier.isPublic(method.getModifiers())) {
                //获取方法调用参数
                Object[] argsToUse = AopProxyUtils.adaptArgumentsIfNecessary(method, args);
                // 直接调用目标对象的方法
                retVal = methodProxy.invoke(target, argsToUse);
            }
            else {
                // 创建一个方法调用器（包含了代理对象、目标对象、调用的方法、参数、目标类型、方法拦截器链）
                // 并执行方法调用器的processd()方法，此方法会一次执行方法调用链，最终会调用目标方法，获取返回结果
                retVal = new CglibMethodInvocation(proxy, target, method, args, targetClass, chain, methodProxy).proceed();
            }
            // 处理方法返回结果：会根据方法返回值的类型，做一些处理，比如方法返回的类型为自己，则最后需要将返回值置为代理对象
            retVal = processReturnType(proxy, target, method, retVal);
            return retVal;
        }
        finally {
            // 目标对象不为null && 目标源不是静态的
            //所谓静态的，你可以理解为是否是单例的
            // isStatic为true，表示目标对象是单例的，同一个代理对象中所有方法共享一个目标对象
            // isStatic为false的时候，通常每次调用代理的方法，target对象是不一样的，所以方法调用万之后需要进行释放，可能有些资源清理，连接的关闭等操作
            if (target != null && !targetSource.isStatic()) {
                targetSource.releaseTarget(target);
            }
            // setProxyContext为ture
            if (setProxyContext) {
                // 需要将旧的代理再放回到上线文中
                AopContext.setCurrentProxy(oldProxy);
            }
        }
    }
}
```

**方法拦截器链的获取**

```java
org.springframework.aop.framework.AdvisedSupport#getInterceptorsAndDynamicInterceptionAdvice
 
AdvisorChainFactory advisorChainFactory = new DefaultAdvisorChainFactory();
 
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
```

会调用DefaultAdvisorChainFactory#getInterceptorsAndDynamicInterceptionAdvice方法获取方法上匹配的拦截器链

![获取方法连接器链](Picture\Spring\获取方法连接器链.png)

**AdvisorChainFactory接口**

```java
package org.springframework.aop.framework;
 
public interface AdvisorChainFactory {
 
    /**
     * 获取方法匹配的拦截器链列表
     * @param config：代理配置信息，里面包含了创建代理的所有信息，如：Advisor列表，此方法会从Advisor列表中找到和mehod匹配的
     * @param targetClass：目标类
     */
    List<Object> getInterceptorsAndDynamicInterceptionAdvice(Advised config, Method method, @Nullable Class<?> targetClass);
}
```

**DefaultAdvisorChainFactory类**

```java
public class DefaultAdvisorChainFactory implements AdvisorChainFactory, Serializable {
 
    @Override
    public List<Object> getInterceptorsAndDynamicInterceptionAdvice(
            Advised config, Method method, @Nullable Class<?> targetClass) {
 
        // 获取Advisor适配器注册器，前面我们有提到过一个知识点：所有的Advisor最终都会转换为MethodInterceptor类型的，
        // 然后注册方法调用链去执行，AdvisorAdapterRegistry就是搞这个事情的,
        // 其内部会将非MethodInterceptor类型通知通过适配器转换为MethodInterceptor类型
        AdvisorAdapterRegistry registry = GlobalAdvisorAdapterRegistry.getInstance();
        //获取配置中的Advisor列表
        Advisor[] advisors = config.getAdvisors();
        List<Object> interceptorList = new ArrayList<>(advisors.length);
        //获取被调用方法所在类实际的类型
        Class<?> actualClass = (targetClass != null ? targetClass : method.getDeclaringClass());
        Boolean hasIntroductions = null;
        //遍历Advisor列表，找到和actualClass和方法匹配的所有方法拦截器（MethodInterceptor）链列表
        for (Advisor advisor : advisors) {
            //判断是否是PointcutAdvisor类型的，这种类型的匹配分为2个阶段，先看类是否匹配，然后再看方法是否匹配
            if (advisor instanceof PointcutAdvisor) {
                PointcutAdvisor pointcutAdvisor = (PointcutAdvisor) advisor;
                // 如果isPreFiltered为ture，表示类以及匹配过，不需要看类是否匹配了
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
                        //方法是否匹配
                        match = mm.matches(method, actualClass);
                    }
                    //方法匹配
                    if (match) {
                        // 通过AdvisorAdapterRegistry的getInterceptors将advisor转换为MethodInterceptor列表
                        MethodInterceptor[] interceptors = registry.getInterceptors(advisor);
                        //方法是否动态匹配
                        if (mm.isRuntime()) {
                            //轮询连接器，将其包装为InterceptorAndDynamicMethodMatcher对象，后续方法调用的时候可以做动态匹配
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
}
```

**AdvisorAdapterRegistry接口**

Advisor Adapter注册器，AdvisorAdapterk可以将Advisor中的Advice适配为MethodInterceptor

```java
package org.springframework.aop.framework.adapter;
 
public interface AdvisorAdapterRegistry {
 
    //将一个通知（Advice）包装为Advisor对象
    Advisor wrap(Object advice) throws UnknownAdviceTypeException;
 
    //根据Advisor获取方法MethodInterceptor列表
    MethodInterceptor[] getInterceptors(Advisor advisor) throws UnknownAdviceTypeException;
 
    //注册AdvisorAdapter，AdvisorAdapter可以将Advisor中的Advice适配为MethodInterceptor
    void registerAdvisorAdapter(AdvisorAdapter adapter);
 
}
```

**DefaultAdvisorAdapterRegistry类**

```java
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
        if (adviceObject instanceof Advisor) {
            return (Advisor) adviceObject;
        }
        if (!(adviceObject instanceof Advice)) {
            throw new UnknownAdviceTypeException(adviceObject);
        }
        Advice advice = (Advice) adviceObject;
        if (advice instanceof MethodInterceptor) {
            // So well-known it doesn't even need an adapter.
            return new DefaultPointcutAdvisor(advice);
        }
        //轮询adapters
        for (AdvisorAdapter adapter : this.adapters) {
            //adapter是否支持适配advice这个通知
            if (adapter.supportsAdvice(advice)) {
                return new DefaultPointcutAdvisor(advice);
            }
        }
        throw new UnknownAdviceTypeException(advice);
    }
 
    //将Advisor对象转换为MethodInterceptor列表，不过通常情况下一个advisor会返回一个MethodInterceptor
    @Override
    public MethodInterceptor[] getInterceptors(Advisor advisor) throws UnknownAdviceTypeException {
        List<MethodInterceptor> interceptors = new ArrayList<>(3);
        Advice advice = advisor.getAdvice();
        if (advice instanceof MethodInterceptor) {
            interceptors.add((MethodInterceptor) advice);
        }
        //轮询adapters
        for (AdvisorAdapter adapter : this.adapters) {
            //先看一下adapter是否支持适配advice这个通知
            if (adapter.supportsAdvice(advice)) {
                //如果匹配，这调用适配器的getInterceptor方法将advisor转换为MethodInterceptor
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

**AdvisorAdapter接口**

```java
package org.springframework.aop.framework.adapter;
 
public interface AdvisorAdapter {
    //判断这个适配器支持advice这个通知么
    boolean supportsAdvice(Advice advice);
 
    //获取advisor对应的MethodInterceptor
    MethodInterceptor getInterceptor(Advisor advisor);
}
```

**MethodBeforeAdviceAdapter类**

适配MethodBeforeAdvice前置通知，负责将MethodBeforeAdvice类型通知转换成MethodBeforeAdviceInterceptor

```java
class MethodBeforeAdviceAdapter implements AdvisorAdapter, Serializable {
 
    @Override
    public boolean supportsAdvice(Advice advice) {
        return (advice instanceof MethodBeforeAdvice);
    }
 
    @Override
    public MethodInterceptor getInterceptor(Advisor advisor) {
        MethodBeforeAdvice advice = (MethodBeforeAdvice) advisor.getAdvice();
        return new MethodBeforeAdviceInterceptor(advice);
    }
 
}
```

**MethodBeforeAdviceInterceptor类**

将MethodBeforeAdvice通知适配为MethodInterceptor类型

```java
package org.springframework.aop.framework.adapter;
 
public class MethodBeforeAdviceInterceptor implements MethodInterceptor, BeforeAdvice, Serializable {
 
    private final MethodBeforeAdvice advice;
 
    public MethodBeforeAdviceInterceptor(MethodBeforeAdvice advice) {
        Assert.notNull(advice, "Advice must not be null");
        this.advice = advice;
    }
 
 
    @Override
    public Object invoke(MethodInvocation mi) throws Throwable {
        //先调用前置通知
        this.advice.before(mi.getMethod(), mi.getArguments(), mi.getThis());
        //然后继续处理连接器连，内部会调用目标方法
        return mi.proceed();
    }
 
}
```

**AfterReturningAdviceAdapter类**

**AfterReturningAdviceInterceptor类**

**ThrowsAdviceAdapter类**

**ThrowsAdviceInterceptor类**

**创建ProxyCallbackFilter对象**

```java
enhancer.setCallbackFilter(new ProxyCallbackFilter(
        this.advised.getConfigurationOnlyCopy(), this.fixedInterceptorMap, this.fixedInterceptorOffset));
```

这一块重点在于ProxyCallbackFilter中的accept方法，这个方法会根据目标，获取目标对象最后让callbacks列表中那个Callback处理

## ProxyFactory简化代理创建

上面代理的整个创建过程和使用过程负载挺大，spring在AdvisedSupport类的基础上添加了两个子类

* ProxyCreatorSupport
* ProxyFactory

# ProxyFactoryBean创建Aop代理

## AOP创建代理的方式

* 手动方式

  通过硬编码一个个创建代理

* 自动方式

  批量的方式，批量的方式用在Spring环境中，通过bean后置处理器来对符合条件的bean创建代理

  AOP创建代理相关的类

  ![AOP](Picture\Spring\AOP.png)

## 手动三种方式

![手动创建AOP](Picture\Spring\手动创建AOP.png)

* ProxyFactory方式

  这种是硬编码的方式，可以脱离Spring直接使用，自动化方式创建代理中都是依靠ProxyFactory来实现的

* AspectJProxyFactory方式

  AspectJ是一个面向切面的框架，最方便的AOP框架

* ProxyFactoryBean方式

  Spring环境中给指定的bean创建代理的一种方式

## ProxyFactoryBean

这个类实现了一个接口FactoryBean

ProxyFactoryBean就是通过FactoryBean的方式来给指定的bean创建一个代理对象

创建代理，有3个信息比较关键

1. 需要增强的功能，这个放在通知中实现
2. 目标对象，表示需要给那个对象进行增强
3. 代理对象，将增强功能和目标组合在一起，然后形成的一个代理对象，通过代理对象来访问目标对象，起到对目标对象增强的效果

使用ProxyFactoryBean也是围绕这三个部分形成的

```java
1.创建ProxyFactoryBean对象
2.通过ProxyFactoryBean.setTargetName设置目标对象的bean名称，目标对象是spring容器中的一个bean
3.通过ProxyFactoryBean。setInterceptorNames添加需要增强的通知
4.将ProxyFactoryBean注册到Spring容器，假设名称为proxyBean
5.从Spring查找名称为proxyBean的bean，这个bean就是生成好的代理对象
```

**案例**

* Service1

  ```java
  public class Service1 {
  
      public void m1() {
          System.out.println("我是 m1 方法");
      }
  
      public void m2() {
          System.out.println("我是 m2 方法");
      }
  }
  ```

* MainConfig1

  ```java
  @Configuration
  public class MainConfig1 {
      @Bean
      public Service1 service1() {
          return new Service1();
      }
  
      @Bean
      public MethodBeforeAdvice beforeAdvice() {
          return (method, args, target) -> System.out.println("准备调用" + method);
      }
  
      @Bean
      public MethodInterceptor costTimeInterceptor() {
          return (invocation -> {
              long startTime = System.nanoTime();
              Object result = invocation.proceed();
              long endTime = System.nanoTime();
              System.out.println(invocation.getMethod() + "耗时：" + (endTime - startTime));
              return result;
          });
      }
  
      @Bean
      public ProxyFactoryBean serviceProxy() {
          ProxyFactoryBean proxyFactoryBean = new ProxyFactoryBean();
          proxyFactoryBean.setTargetName("service1");
          proxyFactoryBean.setInterceptorNames("beforeAdvice", "costTimeInterceptor");
          return proxyFactoryBean;
      }
  }
  ```

* Test1

  ```java
  @Test
  public void test1() {
      AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MainConfig1.class);
      Service1 serviceProxy = context.getBean("serviceProxy", Service1.class);
      System.out.println("----------------");
      serviceProxy.m1();
      System.out.println("----------------");
      serviceProxy.m2();
  }
  ```

### ProxyFactoryBean中的interceptorNames

interceptorNames用来指定拦截器的bean名称列表

* 批量的方式

  ```java
  proxyFactoryBean.setInterceptorNames([所需匹配的bean名称])
  ```

  需要匹配的bean名称后面跟上一个\*，可以用来批量的匹配，此时Spring会从容器中找到下面2种类型的所有bean，bean名称以interceptor开头的将作为增强器

  ```shell
  org.springframework.aop.Advisor
  org.aopalliance.intercept.Interceptor
  ```

  **案例**

  ```java
  public class MainConfig2 {
      @Bean
      public Service1 service1() {
          return new Service1();
      }
  
      @Bean
      public Advisor interceptor1() {
          MethodBeforeAdvice advice = (method, args, target) -> {
              System.out.println("准备调用" + method);
          };
          DefaultPointcutAdvisor advisor = new DefaultPointcutAdvisor();
          advisor.setAdvice(advice);
          return advisor;
      }
  
      @Bean
      public MethodInterceptor interceptor2() {
          return (MethodInterceptor) (invocation) -> {
              long startTime = System.nanoTime();
              Object result = invocation.proceed();
              long endTime = System.nanoTime();
              System.out.println(invocation.getMethod() + "耗时：" + (endTime - startTime));
              return result;
          };
      }
  
      @Bean
      public ProxyFactoryBean serviceProxy() {
          ProxyFactoryBean proxyFactoryBean = new ProxyFactoryBean();
          proxyFactoryBean.setTargetName("service1");
          proxyFactoryBean.setInterceptorNames("interceptor*");
          return proxyFactoryBean;
      }
  }
  ```

* 非批量的方式

  非批量的方式，需要注册多个增强器，需明确的指定多个增强器的bean名称，多个增强器按照参数种指定的顺序执行

  Advice对应的bean类型必须为下面列表种指定的类型

  ```java
  MethodBeforeAdvice（方法前置通知）
  AfterReturningAdvice（方法后置通知）
  ThrowsAdvice（异常通知）
  org.aopalliance.intercept.MethodInterceptor（环绕通知）
  org.springframework.aop.Advisor（顾问）
  ```

## 源码解析

`org.springframework.aop.framework.ProxyFactoryBean#getObject`

```java
public Object getObject() throws BeansException {
    //初始化advisor(拦截器)链
    initializeAdvisorChain();
    //是否是单例
    if (isSingleton()) {
        //创建单例代理对象
        return getSingletonInstance();
    }
    else {
        //创建多例代理对象
        return newPrototypeInstance();
    }
}
```

initializeAdvisorChain方法，用来初始化advisor（拦截器）链，是根据interceptorNames配置，找到Spring容器中符合条件的拦截器，将其放入创建AOP代理的配置中

```java
private synchronized void initializeAdvisorChain() throws AopConfigException, BeansException {
    if (!ObjectUtils.isEmpty(this.interceptorNames)) {
        // 轮询 interceptorNames
        for (String name : this.interceptorNames) {
            //批量注册的方式：判断name是否以*结尾
            if (name.endsWith(GLOBAL_SUFFIX)) {
                //从容器中匹配查找匹配的增强器，将其添加到aop配置中
                addGlobalAdvisor((ListableBeanFactory) this.beanFactory,
                                 name.substring(0, name.length() - GLOBAL_SUFFIX.length()));
            }
            else {
                //非匹配的方式：按照name查找bean，将其包装为Advisor丢到aop配置中
                Object advice;
                //从容器中查找bean
                advice = this.beanFactory.getBean(name);
                //将advice添加到拦截器列表中
                addAdvisorOnChainCreation(advice, name);
            }
        }
    }
}
```

addGlobalAdvisor批量的方式添加Advisor

```java
/**
 * 添加所有全局拦截器和切入点，
 * 容器中所有类型为Advisor/Interceptor的bean，bean名称prefix开头的都会将其添加到拦截器链中
 */
private void addGlobalAdvisor(ListableBeanFactory beanFactory, String prefix) {
    //获取容器中所有类型为Advisor的bean
    String[] globalAdvisorNames =
        BeanFactoryUtils.beanNamesForTypeIncludingAncestors(beanFactory, Advisor.class);
    //获取容器中所有类型为Interceptor的bean
    String[] globalInterceptorNames =
        BeanFactoryUtils.beanNamesForTypeIncludingAncestors(beanFactory, Interceptor.class);
    List<Object> beans = new ArrayList<>(globalAdvisorNames.length + globalInterceptorNames.length);
    Map<Object, String> names = new HashMap<>(beans.size());
    for (String name : globalAdvisorNames) {
        Object bean = beanFactory.getBean(name);
        beans.add(bean);
        names.put(bean, name);
    }
    for (String name : globalInterceptorNames) {
        Object bean = beanFactory.getBean(name);
        beans.add(bean);
        names.put(bean, name);
    }
    //对beans进行排序，可以实现Ordered接口，排序规则：order asc
    AnnotationAwareOrderComparator.sort(beans);
    for (Object bean : beans) {
        String name = names.get(bean);
        //判断bean是否已prefix开头
        if (name.startsWith(prefix)) {
            //将其添加到拦截器链中
            addAdvisorOnChainCreation(bean, name);
        }
    }
}
```

addAdvisorOnChainCreation

```java
private void addAdvisorOnChainCreation(Object next, String name) {
    //namedBeanToAdvisor用来将bean转换为advisor
    Advisor advisor = namedBeanToAdvisor(next);
    //将advisor添加到拦截器链中
    addAdvisor(advisor);
}
```

namedBeanToAdvisor

```java
private AdvisorAdapterRegistry advisorAdapterRegistry = new DefaultAdvisorAdapterRegistry();
 
private Advisor namedBeanToAdvisor(Object next) {
    //将对象包装为Advisor对象
    return this.advisorAdapterRegistry.wrap(next);
}
```

advisorAdapterRegistry#wrap方法将adviceObject包装为Advisor对象

```java
public Advisor wrap(Object adviceObject) throws UnknownAdviceTypeException {
    if (adviceObject instanceof Advisor) {
        return (Advisor) adviceObject;
    }
    if (!(adviceObject instanceof Advice)) {
        throw new UnknownAdviceTypeException(adviceObject);
    }
    Advice advice = (Advice) adviceObject;
    if (advice instanceof MethodInterceptor) {
        return new DefaultPointcutAdvisor(advice);
    }
    for (AdvisorAdapter adapter : this.adapters) {
        if (adapter.supportsAdvice(advice)) {
            return new DefaultPointcutAdvisor(advice);
        }
    }
    throw new UnknownAdviceTypeException(advice);
}
```

# @Aspect中@Pointcut12种用法

手动创建Aop其中一个AspectJProxyFactory，通过@Aspect这个注解，在Spring环境中实现Aop

而AspectJProxyFactory这个类可以通过解析@Aspect标注的类来生成代理aop代理对象

## AspectJ

### AspectJ是什么？

AspectJ是一个面向切面的框架，是目前最好用，最方便的AOP框架，和Spring中的AOP可以集成在一起使用，通过AspectJ提供的一些功能实现Aop代理变得非常方便

### AspectJ使用步骤

```shell
1.创建一个类，使用@Aspect标注
2.@Aspect标注的类中，通过@Pointcut定义切入点
3.@Aspect标注的类中，通过AspectJ提供的一些通知相关的注解定义通知
4.使用AspectJProxyFactory结合@Ascpect标注的类，来生成代理对象
```

**案例**

* Aspect

  ```java
  @Aspect
  public class Aspect1 {
  
      @Pointcut("execution(* top.mnsx.spring.study.test1.Service1.*(..))")
      public void pointcut1() {
      }
  
      @Before(value = "pointcut1()")
      public void before(JoinPoint joinPoint) {
          System.out.println("前置通知：" + joinPoint);
      }
  
      @AfterThrowing(value = "pointcut1()", throwing = "e")
      public void afterThrowing(JoinPoint joinPoint, Exception e) {
          System.out.println(joinPoint + "发生异常：" + e.getMessage());
      }
  }
  ```

### AspectJProxyFactory原理

@Aspect注解的类上，这个类中，可以通过@Pointcut来定义切入点，可以通过@Before、@Around、@After、@AfterRetuning、@AfterThrowing标注在方法上来定义通知，定义之后，将@Aspect标注的这个类交给AspectJProxyFactory来解析生成Advisor链，进而结合目标对象一起来生成代理对象

@Aspect有两个关键点

* @Pointcut：标注在方法上，用来定义切入点，有11中用法
* Aspect类中定义通知：可以通过@Before、@Around、@AfterRuning、@AfterThrowing标注在方法上来定义通知

## @Pointcut的12种用法

**作用**

用来标注在方法中来定义切入点

**定义**

@注解(value="表达标签(表达式格式)")

**表达式标签（10种）**

* execution：用于匹配方法执行的连接点
* within：用于匹配指定类型内的方法执行
* this：用于匹配当前AOP代理对象类型的执行方法（注意是Aop代理对象的类型匹配，这样就可能包括引入接口）
* target：用于匹配当前目标对象类型的执行方法（注意是目标对象的类型匹配，这样就不包括引入接口）
* args：用于匹配当前执行的方法传入的参数为指定类型的执行方法
* @within：用于匹配所持有指定注解类型内的执行方法
* @target：用于匹配当前目标对象类型的执行方法，其中目标对象持有指定的注解
* @args：用于匹配当前执行的方法传入的参数持有指定注解的执行
* @annotation：用于匹配当前执行方法持有指定注解的方法
* bean：SpringAOP扩展的，AspectJ没有对于指示符，用于匹配特定名称的bean对象的执行方法

**10种标签组成了12种用法**

### execution

使用execution（方法表达式）匹配方法执行

```java
execution(modifiers-pattern? ret-type-pattern declaring-type-pattern? name-pattern(param-pattern) throws-pattern?)
```

* 其中带?的modifiers-pattern?，declaring-type-pattern?，throws-pattern?是可选项

* ret-type-pattern，name-pattern，parameters-pattern是必选项
* modifier-pattern?修饰符匹配，如public
* ret-type-pattern返回匹配，*表示任何返回值，全路径的类名等
* declaring-type-pattern?类路径匹配
* name-pattern方法名匹配，\*代表所有，如set\*，代表以set开头的方法
* (param-pattern)参数匹配，指定方法参数（声明的类型），(..)代表所有参数，(*, String)代表第一个参数为任何值，第二个为String类型，(.., Stirng)代表最后一个参数是String类型
* throws-pattern?，异常类型匹配

**类型匹配语法**

* \*：匹配任何数量字符
* ..：匹配任何数量字符的重复
* \*：匹配指定类型及其子类型，仅能作为后缀放在类型模式后边

### within

within（类型表达式）：目标对象target的类型是否和within种指定的类型匹配

`within(top.mnsx.spring.study..*)`：包及其包下任何方法执行

匹配原则

```java
target.getClass().equals(within表达式中指定的类型)
```

### this

this（类型全限定名）：通过Aop创建的代理对象的类型是否和this种指定的类型匹配，注意判断的是目标的代理对象，this中使用的表达式必须是类型全限定名，不支持通配符

匹配原则

```java
x.getClass().isAssignableFrom(proxy.getClass());
```

### target

target（类型全限定名）：判断目标对象的类型是否和指定的类型匹配：注意判断的目标对象的类型；表达式必须式类型全限定名，不支持通配符

```java
x.getClass().isAssignableFrom(target.getClass());
```

### args

args（参数类型列表）匹配当前执行的方法传入的参数是否为args中指定的类型，注意是匹配传入的参数类型，不是匹配方法签名的参数类型，参数类型列表中的参数必须是类型全限定名，不支持通配符，args属于动态切入点，也就是执行方法的时候进行判断，这种切入点开销非常大，非特殊情最好不要使用

### @within

@within（注释类型）：匹配指定的注解内定义的方法

匹配规则

调用目标方法的时候，通过java中Method.getDeclaringClass()获取当前的方法是那个类型中定义的，然后会看这个类上是否有指定的注解

```java
被调用的目标方法Method对象.getDeclaringClass().getAnnotation(within中指定的注解类型) != null
```

### @target

@target（注解类型）：判断目标对象target类型上是否有指定的注解，@target中注解类型也必须是全限定类型名

匹配规则

```java
target.class.getAnnotation(指定的注解类型) != null
```

* 注解直接标注在目标类上
* 注解标注在父类上，但是注解必须是可以继承的

### @args

@args（注解类型）：方法参数所属的类上有指定的注解，注意不是参数上有指定的注解，而是参数的类上有指定的注解

### @annotation

@annotation（注解类型）：匹配被调用的方法上有指定的注解

### bean

bean（bean名称）：这个用在spring环境中，匹配容器中指定名称的bean

### reference pointcut

表示引用其他名称切入点

我们可以将切入专门放在一个类中集中定义

其他地方可以通过引用的方式引入其他类中集中定义

```java
@Pointcut("完整包名类名.方法名称()")
```

**案例**

```java

public class AspectPcDefine {
    @Pointcut("bean(bean1)")
    public void pc1() {
    }
 
    @Pointcut("bean(bean2)")
    public void pc2() {
    }
}

@Aspect
public class Aspect14 {
 
    @Pointcut("com.javacode2018.aop.demo9.test14.AspectPcDefine.pc1()")
    public void pointcut1() {
    }
 
    @Pointcut("com.javacode2018.aop.demo9.test14.AspectPcDefine.pc1() || com.javacode2018.aop.demo9.test14.AspectPcDefine.pc2()")
    public void pointcut2() {
    }
 
}
```

### 组合型的pointcut

pointcut定义时，还可以使用&&、||、!运算符

# @Aspect中5种通知详解

## @Before：前置通知

1. 类上需要使用@Aspect标注
2. 任意方法上使用@Before标注，将这个方法作为前置通知，目标方法被调用之前，会自动回调这个方法
3. 被@Before标注的方法参数可以为空，或为JoinPoint类型，当为JoinPoint类型时，必须为第一个参数
4. 被@Before标注的方法名称可以随意命名，符合java规范即可

@Before通知最后会被解析为`org.springframework.aop.aspectj.AspectJMethodBeforeAdvice`

## 通知中获取被调用方法信息

通知中如果想要获取被调用方法的信息，分为两种情况

1. 非环绕通知，可以将JoinPoint作为通知方法的第一个参数，通过这个参数获取被调用方法的信息
2. 如果是环绕通知，可以将ProceedingJoingPoint作为方法的第一个参数，通过这个参数获取被调用方法的信息

**JoinPoint：连接点信息**

提供访问当前被通知方法的目标对象、代理对象、方法参数等数据

```java
package org.aspectj.lang;  
import org.aspectj.lang.reflect.SourceLocation;
 
public interface JoinPoint {  
    String toString();         //连接点所在位置的相关信息  
    String toShortString();     //连接点所在位置的简短相关信息  
    String toLongString();     //连接点所在位置的全部相关信息  
    Object getThis();         //返回AOP代理对象
    Object getTarget();       //返回目标对象  
    Object[] getArgs();       //返回被通知方法参数列表，也就是目前调用目标方法传入的参数  
    Signature getSignature();  //返回当前连接点签名，这个可以用来获取目标方法的详细信息，如方法Method对象等
    SourceLocation getSourceLocation();//返回连接点方法所在类文件中的位置  
    String getKind();        //连接点类型  
    StaticPart getStaticPart(); //返回连接点静态部分  
} 
```

**ProceedingJoinPoint：环绕通知连接点信息**

用于环绕通知，内部主要关注两个方法，一个有参的，一个无参的，用来继续执行拦截器链上的下一个通知

```java
package org.aspectj.lang;
import org.aspectj.runtime.internal.AroundClosure;
 
public interface ProceedingJoinPoint extends JoinPoint {
 
    /**
     * 继续执行下一个通知或者目标方法的调用
     */
    public Object proceed() throws Throwable;
 
    /**
     * 继续执行下一个通知或者目标方法的调用
     */
    public Object proceed(Object[] args) throws Throwable;
 
}
```

**Signature：连接点签名信息**

注意JoinPoint#getSignature()这个方法，用来获取连接点的签名信息

通常情况，Spring中的AOP都是用来对方法进行拦截，所以通常情况下连接点都是一个具体的方法

`org.aspectj.lang.reflect.MethodSignature`

JoinPoint#getSignature()都可以转换为MethodSignature类型，然后可以通过这个接口提供一些方法来看获取被调用的方法的详细信息

## @Around：环绕通知

环绕通知会包裹目标方法的执行，可以在通知内部调用ProceedingJoinPoint.process方法继续执行下一个拦截器

1. 若需要获取目标对象信息，需要将ProceedingJoinPoint作为第一个参数
2. 通常使用Object类型作为方法的返回值，返回值也可以为void

**对应的通知类**

@Around通知最后会被解析为

```java
org.springframework.aop.aspectj.AspectJAroundAdvice
```

## @After：后置通知

后置通知，在方法执行之后执行，用法和前置通知类似

* 不管目标方法是否有一场，后置通知都会执行
* 这种通知无法获取方法返回值
* 可以使用Join Point作为方法的第一个参数，用来获取连接点的信息

@After通知最后会被解析为下面这个通知类

`org.springframework.aop.aspectj.AspectJAfterAdvice`

这个类中有invoke方法，这个方法内部会调用被通知的方法，其内部采用try-finally的方式实现的

```java
@Override
public Object invoke(MethodInvocation mi) throws Throwable {
    try {
        //继续执行下一个拦截器
        return mi.proceed();
    }
    finally {
        //内部通过反射调用被@After标注的方法
        invokeAdviceMethod(getJoinPointMatch(), null, null);
    }
}
```

## @AfterReturning：返回通知

返回通知，在方法返回结果之后执行

* 可以获取到方法的返回值
* 当目标方法返回异常的时候，这个通知不会被调用，这点和@After通知时有区别的

> @AfterReturning注解中有个关键参数
>
> * returning：用来指定返回值对应方法的参数名称，返回值对应方法的第二个参数，名称为retVal

@AfterReturning通知最后会被解析为`org.springframework.aop.aspectj.AspectJAfterReturningAdvice`

## @AfterThrowing：异常通知

在方法抛出异常之后回调@AfterThrowing标注的方法

@AfterThrowing标注的方法可以指定异常参数，当被调用的方法触发该异常及其子类型的异常之后，会触发异常方法的回调，也可以不指定异常类型，此时匹配所有的异常

**不管异常是否被异常通知捕获，异常还会继续向外抛出**

**对应的通知类**

@AfterThrowing通知最后会被解析为`org.springframework.aop.aspectj.AspectJAfterThrowingAdvice`

```java
@Override
public Object invoke(MethodInvocation mi) throws Throwable {
    try {
        //继续调用下一个拦截器链
        return mi.proceed();
    }
    catch (Throwable ex) {
        //判断ex和需要不糊的异常是否匹配
        if (shouldInvokeOnThrowing(ex)) {
            //通过反射调用@AfterThrowing标注的方法
            invokeAdviceMethod(getJoinPointMatch(), null, ex);
        }
        //继续向外抛出异常
        throw ex;
    }
}
```

## 几种通知对比

| 通知类型        | 执行时间点     | 可获取返回值 | 目标方法异常时是否会执行 |
| :-------------- | :------------- | :----------- | :----------------------- |
| @Before         | 方法执行之前   | 否           | 是                       |
| @Around         | 环绕方法执行   | 是           | 自己控制                 |
| @After          | 方法执行后     | 否           | 是                       |
| @AfterReturning | 方法执行后     | 是           | 否                       |
| @AfterThrowing  | 方法发生异常后 | 否           | 是                       |

![Aop通知类型](Picture\Spring\Aop通知类型.png)

# @EnableAspectJAutoProxy、@Aspect中通知顺序详解

## @EnableAspectJAutoProxy自动为bean创建代理对象

@EnableAspectJAutoProxy可以自动为Spring容器中符合条件的bean创建代理对象

@EnableAspectJAutoProxy需要结合@Aspect注解一起使用，用法简单

@EnableAspectJAutoProxy这个注解比较关键，用来启用自动代理创建，会找到容器中所有标注有@Aspect注解的bean以及Advisor类型的bean，会将他们转换为Advisor集合，Spring会通过Advisor集合对容器中满足切入点表达式的bean生成代理对象，整个都是Spring容器启动的过程中自动完成

## 通知执行顺序

@EnableAspectJAutoProxy允许Spring容器中通过Advisor、@Aspect来定义通知，当Spring容器中存在多个Advisor、@Aspect时，执行顺序如下

 SpringAOP中4中通知

```java
org.aopalliance.intercept.MethodInterceptor
org.springframework.aop.MethodBeforeAdvice
org.springframework.aop.AfterReturningAdvice
org.springframework.aop.ThrowsAdvice
```

所有的通知最终都需要转换为MethodInterceptor类型的通知，然后组成一个methodInterceptor列表

```java
org.springframework.aop.MethodBeforeAdvice -> org.springframework.aop.framework.adapter.MethodBeforeAdviceInterceptor
org.springframework.aop.AfterReturningAdvice -> org.springframework.aop.framework.adapter.AfterReturningAdviceInterceptor
org.springframework.aop.ThrowsAdvice -> org.springframework.aop.framework.adapter.ThrowsAdviceInterceptor
```

**org.aopalliance.intercept.MethodInterceptor：方法拦截器**

方法拦截器，可以在方法执行前后执行一些增强操作，其他类型的通知最终都会被包装为MethodInterceptor来执行

```java
class MyMethodInterceptor implements MethodInterceptor {
    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        System.out.println("我是MethodInterceptor start");
        //调用invocation.proceed()执行下一个拦截器
        Object result = invocation.proceed();
        System.out.println("我是MethodInterceptor end");
        //返回结果
        return result;
    }
}
```

**org.springframework.aop.MethodBeforeAdvice：方法前置通知**

方法前置通知，可以在方法之前定义增强操作

```java
class MyMethodBeforeAdvice implements MethodBeforeAdvice {
 
    @Override
    public void before(Method method, Object[] args, @Nullable Object target) throws Throwable {
        System.out.println("我是MethodBeforeAdvice");
    }
}
```

MethodBeforeAdvice最终会被包装为MethodBeforeAdviceInterceptor类型，然后放到拦截器链中去执行，通过MethodBeforeAdviceInterceptor代理可以理解为MethodBeforeAdvice的执行过程

```java
public class MethodBeforeAdviceInterceptor implements MethodInterceptor, BeforeAdvice, Serializable {
 
    private final MethodBeforeAdvice advice;
 
    public MethodBeforeAdviceInterceptor(MethodBeforeAdvice advice) {
        this.advice = advice;
    }
 
 
    @Override
    public Object invoke(MethodInvocation mi) throws Throwable {
        //调用MethodBeforeAdvice的before方法，执行前置通知
        this.advice.before(mi.getMethod(), mi.getArguments(), mi.getThis());
        //执行下一个拦截器
        return mi.proceed();
    }
 
}
```

**org.springframework.aop.AfterReturningAdvice：方法返回通知**

方法返回通知，用来在方法执行完毕之后执行一些增强操作

```java
class MyAfterReturningAdvice implements AfterReturningAdvice {
 
    @Override
    public void afterReturning(@Nullable Object returnValue, Method method, Object[] args, @Nullable Object target) throws Throwable {
        System.out.println("我是AfterReturningAdvice");
    }
}
```

AfterReturningAdvice最终会被包装为AfterReturningAdviceInterceptor类型，然后放到拦截器链中去执行，通过AfterReturningAdviceInterceptor可以理解为AfterReturningAdvice的执行过程

```java
public class AfterReturningAdviceInterceptor implements MethodInterceptor, AfterAdvice, Serializable {
 
    private final AfterReturningAdvice advice;
 
    public AfterReturningAdviceInterceptor(AfterReturningAdvice advice) {
        this.advice = advice;
    }
 
 
    @Override
    public Object invoke(MethodInvocation mi) throws Throwable {
        //执行下一个拦截器，可以获取目标方法的返回结果
        Object retVal = mi.proceed();
        //调用方法返回通知的afterReturning方法，会传入目标方法的返回值等信息
        this.advice.afterReturning(retVal, mi.getMethod(), mi.getArguments(), mi.getThis());
        return retVal;
    }
 
}
```

**org.springframework.aop.ThrowsAdvice：异常通知**

当目标方法发生异常时，可以通过ThrowsAdvice来指定需要回调的方法

```java
/**
 * 用来定义异常通知
 * 方法名必须是afterThrowing，格式参考下面2种定义
 * 1. public void afterThrowing(Exception ex)
 * 2. public void afterThrowing(Method method, Object[] args, Object target, Exception ex)
 */
class MyThrowsAdvice implements ThrowsAdvice {
    public void afterThrowing(Method method, Object[] args, Object target, Exception ex) {
        System.out.println("我是ThrowsAdvice");
    }
}
```

ThrowsAdvice最终会被包装为ThrowsAdviceInterceptor类型，然后放到拦截器链中去执行，通过ThrowsAdviceInterceptor可以理解ThrowsAdvice的执行过程，ThrowsAdviceInterceptor消息参数传入一个自定义个的ThrowsAdvice对象

```java
public class ThrowsAdviceInterceptor implements MethodInterceptor, AfterAdvice {
 
    private final Object throwsAdvice;
 
    public ThrowsAdviceInterceptor(Object throwsAdvice) {
        this.throwsAdvice = throwsAdvice;
    }
 
    @Override
    public Object invoke(MethodInvocation mi) throws Throwable {
        try {
            return mi.proceed();
        } catch (Throwable ex) {
            //调用 ThrowsAdvice 中的 afterThrowing 方法来处理异常
            this.throwsAdvice.afterThrowing(。。。。);
            //将异常继续往外抛
            throw ex;
        }
    }
}
```

**单个Aspect种多个通知的顺序**

```java
public class Aspect4 {
    @Pointcut("execution(* com.javacode2018.aop.demo11.test4.Service4.*(..))")
    public void pc() {
    }
 
    @Before("pc()")
    public void before() {
        System.out.println("@Before通知!");
    }
 
    @Around("pc()")
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("@Around通知start");
        Object result = joinPoint.proceed();
        System.out.println("@Around绕通知end");
        return result;
    }
 
    @After("pc()")
    public void after() throws Throwable {
        System.out.println("@After通知!");
    }
 
    @AfterReturning("pc()")
    public void afterReturning() throws Throwable {
        System.out.println("@AfterReturning通知!");
    }
 
    @AfterThrowing("pc()")
    public void afterThrowing() {
        System.out.println("@AfterThrowing通知!");
    }
 
}
```

先执行第一个通知@AfterThrowing

```java
try {
    //执行下一个拦截器
    return mi.proceed();
} catch (Throwable ex) {
    System.out.println("@AfterThrowing通知!");
    //继续抛出异常
    throw ex;
}
```

mi.processed()执行第二个通知@AfterReturning

```java
try {
    //执行下一个拦截器
    Object retVal = mi.proceed();
    System.out.println("@AfterReturning通知!");
    return retVal;
} catch (Throwable ex) {
    System.out.println("@AfterThrowing通知!");
    //继续抛出异常
    throw ex;
}
```

继续mi.proceed()执行第三个通知@After

```java
try {
    Object result = null;
    try {
        //执行下一个拦截器
        result = mi.proceed();
    } finally {
        System.out.println("@After通知!");
    }
    System.out.println("@AfterReturning通知!");
    return retVal;
} catch (Throwable ex) {
    System.out.println("@AfterThrowing通知!");
    //继续抛出异常
    throw ex;
}
```

继续mi.proceed()执行第四个通知@Around

```java
try {
    Object result = null;
    try {
        System.out.println("@Around通知start");
        result = joinPoint.proceed();
        System.out.println("@Around绕通知end");
        return result;
    } finally {
        System.out.println("@After通知!");
    }
    System.out.println("@AfterReturning通知!");
    return retVal;
} catch (Throwable ex) {
    System.out.println("@AfterThrowing通知!");
    //继续抛出异常
    throw ex;
}
```

继续joinPoint.proceed()执行第五个通知@Before

```java
try {
    Object result = null;
    try {
        System.out.println("@Around通知start");
        System.out.println("@Before通知!");
        result = mi.proceed();
        System.out.println("@Around绕通知end");
        return result;
    } finally {
        System.out.println("@After通知!");
    }
    System.out.println("@AfterReturning通知!");
    return retVal;
} catch (Throwable ex) {
    System.out.println("@AfterThrowing通知!");
    //继续抛出异常
    throw ex;
}
```

继续joinPoint.proceed()会调用目标方法

```java
try {
    Object result = null;
    try {
        System.out.println("@Around通知start");
        System.out.println("@Before通知!");
        result = // 通过反射调用目标方法; //@1
        System.out.println("@Around绕通知end");
        return result;
    } finally {
        System.out.println("@After通知!");
    }
    System.out.println("@AfterReturning通知!");
    return retVal;
} catch (Throwable ex) {
    System.out.println("@AfterThrowing通知!");
    //继续抛出异常
    throw ex;
}
```

**最终通知的执行顺序**

```java
@Around通知start
@Before通知!
@Around绕通知end
@After通知!
@AfterReturning通知!
方法调用
```

## @EnableAspectJAutoProxy中为通知指定顺序

* 为@Aspect指定顺序：用@Order注解
* 为Advisor指定顺序：实现Ordered接口

## 多个@Aspect、Advisor排序规则

排序规则

1. 在Spring容器中获取@Aspect、Advisor类型的所有bean，得到一个列表list

2. 对list按照order的值顺序排序，得到新的list_new

3. 然后再对list_new中@Aspect类型的bean内部的通知进行排序

   ```java
   @AfterThrowing
   @AfterReturning
   @After
   @Around
   @Before
   ```

4. 最后运行的时候会得到上面排序产生的方法调用链列表去执行

## @EnableAspectJAutoProxy另外两个功能

这个注解还有两个参数

```java
public @interface EnableAspectJAutoProxy {
 
 /**
  * 是否基于类来创建代理，而不是基于接口来创建代理
  * 当为true的时候会使用cglib来直接对目标类创建代理对象
  * 默认为 false：即目标bean如果有接口的会采用jdk动态代理来创建代理对象，没有接口的目标bean，会采用cglib来创建代理对象
  */
 boolean proxyTargetClass() default false;
 
 /**
  * 是否需要将代理对象暴露在ThreadLocal中，当为true的时候
  * 可以通过org.springframework.aop.framework.AopContext#currentProxy获取当前代理对象
  */
 boolean exposeProxy() default false;
 
}
```

## @EnableAspectJAutoProxy原理

@EnableAspectJAutoProxy会再Spring容器中注册一个Bean

```java
org.springframework.aop.aspectj.annotation.AnnotationAwareAspectJAutoProxyCreator
```

AnnotationAwareAspectProxyCreator是BeanPostProcessor类型的，BeanPostProcessor后置处理器，可以再bean生命期间对bean进行操作

# @EnableAsync&@Async实现方法异步操作

## 作用

Spring容器中实现bean方法的异步调用

## 用法

1. 需要异步执行的方法上面使用@Async注解标注，若bean中所有的方法都需要异步执行，可以直接将@Async加载再类上
2. 将@EnableAsync添加到Spring配置类上，此时@Async注解才会生效

## 获取异步执行结果

### 无返回值

方法返回值不是Furure类型的，被执行时，会立即返回，并且无法获取方法返回值

### 有返回值

若需获取异步执行结果，方法返回值必须为Future类型，使用Spring提供的静态方法

`AsyncResult.forValue(string)`

## 自定义异步执行的线程池

默认情况下，@EnableAsync使用内置的线程池来异步调用方法

**有两种方法来定义异步处理的线程池**

* 在Spring容器中定义一个线程池类型的bean，bean名称必须是taskExecutor

  ```java
  @Bean
  public Executor taskExecutor() {
      ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
      executor.setCorePoolSize(10);
      executor.setMaxPoolSize(100);
      executor.setThreadNamePrefix("my-thread-");
      return executor;
  }
  ```

* 定义一个bean，实现AsyncConfigurer接口中的getAsyncExecutor方法，这个方法需要返回自定义线程池

  ```java
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

## 自定义异常处理

异步方法若发生了异常，可以通过自定义异常处理来解决

异常处理分为两种情况

* 当返回值是Future的时候，方法内部有异常的时候，异常会向外抛出，可以对Future.get采用try-catch来捕获异常
* 当返回值不是Future的时候，可以自定义一个bean，实现AsyncConfigurer接口中的getAsyncUncaughtExceptionHandler方法，返回自定义的异常处理器

**案例：无返回值异常处理**

```java
@Bean
public AsyncConfigurer asyncConfigurer() {
    return new AsyncConfigurer() {
        @Nullable
        @Override
        public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
            return new AsyncUncaughtExceptionHandler() {
                @Override
                public void handleUncaughtException(Throwable ex, Method method, Object... params) {
                    //当目标方法执行过程中抛出异常的时候，此时会自动回调这个方法，可以在这个方法中处理异常
                }
            };
        }
    };
}
```

## 线程池隔离

一个系统中可能有很多业务，比如充值服务、提现服务或者其他服务，这些服务中都有一些方法需要异步执行，默认情况下他们会使用同一个线程池去执行，如果有一个业务量比较大，占用了线程池中的大量线程，此时会导致其他业务的方法无法执行，那么我们可以采用线程隔离的方式，对不同的业务使用不同的线程池，相互隔离，互不影响
@Async注解有一个value参数，用来指定线程池的bean名称，方法运行时，就会采用指定的线程池来执行目标方法

## 源码学习

内部使用AOP实现的，@EnableAsync会引入一个bean后置处理器

AsyncAnnotationBeanPostProcessor，将其注册到Spring容器，这个bean后置处理器在所有bean创建过程中，判断bean的类上是否有@Async注解或者类中是否有@Async标注方法，如果有，会通过AOP给这个bean生成代理对象`org.springframework.scheduling.annotation.AsyncAnnotationAdvisor`，这个切面中会引入一个拦截器`AnnotationAsyncExecutionInterceptor`，方法异步调用的关键代码就是在这个拦截器的invoke方法实现

# @Scheduled@EnableScheduling定时器详解

## @Scheduled配置定时规则

@Scheduled可以用来配置定时器的执行规则

```java
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

### cron

该参数接受一个cron表达式，cron表达式是一个字符串，字符串以5或6个空格隔开，分开共6或7个域，每一个域表达一个含义

cron表达式语法

```shell
[秒] [分] [小时] [日] [月] [周] [年]
```

> 年不是不需要的域，可以省略年

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

**cron属性接受的cron表达式支持占位符**

### zone

时区，接受一个java.util.TimeZone#ID，cron表达式会基于时区分析，默认是一个空字符串，即取服务器所在地的时区，该字段一般留空

### fixedDelay

上一次执行完毕时间点之后的多长事件再执行（单位为毫秒）

### fixedDelayString

与fixedDelay相同，只是使用字符串的形式，唯一不同的是支持占位符

### fixedRate

上一次开始执行时间之后多长时间再执行

### fixedRateString

与fixedRate相同，支持占位符

### initialDelay

第一次延迟多长时间再执行

### initialDelayString

与initialDelay意思相同，只是使用字符串的形式，唯一不同支持占位符

## @Schedules注解

当一个方法上需要同时指定多个定时规则的时候，可以指定这个配置

```java]
@Target({ElementType.METHOD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Schedules {
 Scheduled[] value();
}
```

## 为定时器定义线程池

定时器默认情况下使用下面的线程池执行定时任务

```java
new ScheduledThreadPoolExecutor(1)
```

只是一个线程，如果需要定时执行的任务太多，任务只能排队执行

可以通过自定义定时器中的线程池来解决这个问题，定义一个ScheduleExecutorService类型的bean，名称为taskScheduler

```java
@Bean
public ScheduledExecutorService taskScheduler() {
    //设置需要并行执行的任务数量
    int corePoolSize = 20;
    return new ScheduledThreadPoolExecutor(corePoolSize);
}
```

## 源码学习

从EnableScheduling注解开始，这个注解会导入SchedulingConfiguration类

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Import(SchedulingConfiguration.class)
@Documented
public @interface EnableScheduling {
    
}
```

ScheduleAnnotationBeanPostProcessor是一个bean后置处理器，内部有个postProcessAfterInitialization方法，Spring中任何bean的初始化完毕之后，会自动调用这个方法，而ScheduledAnnotationBeanPostProcessor再这个方法中会解析bena中标注有@Scheduled注解的方法

ScheduledAnnotationBeanPostProcessor还实现了一个接口：SmartInitializingSingleton，SmartinitializingSingleton中有个方法afterSingletonsInstantiated会在Spring容器中所有单例bean初始化完毕之后调用，定时器的装配及启动都是再这个方法中

```java
org.springframework.scheduling.annotation.ScheduledAnnotationBeanPostProcessor#afterSingletonsInstantiated
```

# 详解Spring编程式事务

通过硬编码的方式使用Spring中提供的事务相关类来控制事务

## 编程式事务主要有两种用法

* 方式1：通过PaleformTransactionManager控制事务
* 方式2：通过TransactionTemplate控制事务

## 通过PlatformTransactionManager

这是最原始的方式，代码量较大，其他方式都是对这种方式封装

* SQL

  ```sql
  create database spring_study;
  
  use spring_study;
  
  create table t_user(
  	id int primary key AUTO_INCREMENT,
  	name varchar(256) NOT NULL DEFAULT '' COMMENT '姓名'
  );
  ```

* Test

  ```java
  @Test
  public void test1() {
      PlatformTransactionManager platformTransactionManager = new DataSourceTransactionManager(datasource);
      DefaultTransactionDefinition transactionDefinition = new DefaultTransactionDefinition();
      TransactionStatus transactionStatus = platformTransactionManager.getTransaction(transactionDefinition);
  
      try {
          System.out.println(userMapper.selectList(null));
          User user1 = new User();
          user1.setId(0);
          user1.setName("mnsx");
          userMapper.insert(user1);
          User user2 = new User();
          user2.setName("xkq");
          userMapper.insert(user2);
  
          platformTransactionManager.commit(transactionStatus);
      } catch (Exception e) {
          platformTransactionManager.rollback(transactionStatus);
      }
  
      System.out.println(userMapper.selectList(null));
  }
  ```

**代码分析**

### 定义事务管理器

事务管理器相当于一个管理员，这个管理员就是用来帮你控制十五的

Spring中使用PlatformTransactionManager这个接口来表示事务管理器

```java
public interface PlatformTransactionManager {
 
 //获取一个事务（开启事务）
 TransactionStatus getTransaction(@Nullable TransactionDefinition definition)
   throws TransactionException;
 
 //提交事务
 void commit(TransactionStatus status) throws TransactionException;
 
 
 //回滚事务
 void rollback(TransactionStatus status) throws TransactionException;
 
}
```

PlatformTransactionManger多个实现类，用来应对不同的环境

![PlatformTransactionManager多个实现类](Picture\Spring\PlatformTransactionManager多个实现类.png)

* JpaTransactionManager：如果你用Jpa来操作DB，那么需要这个管理器来控制事务
* DataSourceTransactionManager：如果指定数据源的方式，JdbcTemplate、Mybatis、Ibatis，那么需要使用这个管理器来帮你控制事务
* HibernateTransactionManager：如果用Hibernate来操作DB，那么需要用这个管理器来控制事务
* JtaTransactionManager：如果使用的JTA来操作DB，这种通常是分布式事务，此时需要用这种管理器来控制事务

```java
PlatformTransactionManager platformTransactionManager = new DataSourceTransactionManager(dataSource);
```

### 定义事务属性TransactionDefinition

定义事务属性

Spring中使用TransactionDefinition接口来表示事务的定义信息

### 开启事务

调用事务管理器的getTransaction方法，即开启一个事务

```java
TransactionStatus transactionStatus = platformTransactionManager.getTransaction(transactionDefinition);
```

这个方法会返回一个TransactionStatus表示事务状态的一个对象，通过TransactionStatus提供一些方法可以用来控制事务的一些状态

执行getTransaction后，Spring内部会执行一些操作

```java
//有一个全局共享的threadLocal对象 resources
static final ThreadLocal<Map<Object, Object>> resources = new NamedThreadLocal<>("Transactional resources");
//获取一个db的连接
DataSource datasource = platformTransactionManager.getDataSource();
Connection connection = datasource.getConnection();
//设置手动提交事务
connection.setAutoCommit(false);
Map<Object, Object> map = new HashMap<>();
map.put(datasource,connection);
resources.set(map);
```

## 通过TransactionTemplate

方式一中的部分代码可以重用，所以spring对其进行优化，采用模板方法模式就其进行封装，主要省去了提交或回滚事务的代码

```java
@Test
public void test2() {
    PlatformTransactionManager  platformTransactionManager = new DataSourceTransactionManager(dataSource);
    DefaultTransactionDefinition transactionDefinition = new DefaultTransactionDefinition();
    transactionDefinition.setTimeout(10);
    TransactionTemplate transactionTemplate = new TransactionTemplate(platformTransactionManager, transactionDefinition);
    transactionTemplate.executeWithoutResult((transactionStatus -> {
        System.out.println(userMapper.selectList(null));
        User user1 = new User();
        user1.setId(0);
        user1.setName("mnsx");
        userMapper.insert(user1);
        User user2 = new User();
        user2.setName("xkq");
        userMapper.insert(user2);
    }));
    System.out.println(userMapper.selectList(null));
}
```

**TransactionTemplate提供的方法执行业务操作**

1. executeWithoutResult(Consumer<TransactionStatus\> action)：没有返回值，传递一个Consumer对象，在accept方法中执行业务操作
2. <T\> T execute(TransactionCallback<T\> action)，有返回值，需要传递一个TransactionCallback对象，在doInTransaction方法中执行业务操作

执行回滚的情况

1. transactionStatus.setRollbackOnly()：将事务状态标注为回滚状态
2. 方法内部抛出异常

当上述情况不满足时，则提交事务

# 详解Spring声明式事务

## 声明式事务的概念

通过配置的方式，通知Spring，那些方法需要被Spring管理事务

注解方式，只需要在方法上面加上一个@Transaction注解，那么方法执行之前Spring会自动开启事务，方法执行完毕之后，会自动提交或回滚事务，而方法内部没有任何事务相关代码

## 声明式事务注释方式步骤

### 启用Spring的注解驱动事务管理功能

在Spring配置类上加上@Enable Transaction Management注释

> 当spring容器启动的时候，发现有@EnableTransactionMangement注解，此时会拦截所有bean的创建，扫描是否有@Transaction注释，如果有之恶个，Spring会通过Aop方式给bean生成代理对象，代理对象中会增加一个拦截，拦截器会拦截bean中public方法的执行，会在方法执行之前启动事务，方法执行完毕之后提交或者回滚

```java
org.springframework.transaction.interceptor.TransactionInterceptor#invoke
```

@EnableTransactionManagement

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(TransactionManagementConfigurationSelector.class)
public @interface EnableTransactionManagement {
 
 /**
  * spring是通过aop的方式对bean创建代理对象来实现事务管理的
  * 创建代理对象有2种方式，jdk动态代理和cglib代理
  * proxyTargetClass：为true的时候，就是强制使用cglib来创建代理
  */
 boolean proxyTargetClass() default false;
 
 /**
  * 用来指定事务拦截器的顺序
  * 我们知道一个方法上可以添加很多拦截器，拦截器是可以指定顺序的
  * 比如你可以自定义一些拦截器，放在事务拦截器之前或者之后执行，就可以通过order来控制
  */
 int order() default Ordered.LOWEST_PRECEDENCE;
}
```

### 定义事务管理器

事务交给Spring管理，那么需要创建一个或者多个事务管理者，有这些管理者来管理具体的事务，这些都是管理者来负责

Spring中使用PlatformTransactionManager这个接口来表示事务管理者

## 需使用事务的目标上加@Transaction注解

* @Transaction放在接口上，那么接口的实现类中所有public都被Spring自动加上事务
* @Transaction放在类上，那么当前类以及其下无限极子类中所有public方法将被Spring自动加上事务
* @Transaction方法在public方法上，那么该方法将被Spring自动加上事务

@Transactional

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Transactional {
 
    /**
     * 指定事务管理器的bean名称，如果容器中有多事务管理器PlatformTransactionManager，
     * 那么你得告诉spring，当前配置需要使用哪个事务管理器
     */
    @AliasFor("transactionManager")
    String value() default "";
 
    /**
     * 同value，value和transactionManager选配一个就行，也可以为空，如果为空，默认会从容器中按照类型查找一个事务管理器bean
     */
    @AliasFor("value")
    String transactionManager() default "";
 
    /**
     * 事务的传播属性
     */
    Propagation propagation() default Propagation.REQUIRED;
 
    /**
     * 事务的隔离级别，就是制定数据库的隔离级别，数据库隔离级别大家知道么？不知道的可以去补一下
     */
    Isolation isolation() default Isolation.DEFAULT;
 
    /**
     * 事务执行的超时时间（秒），执行一个方法，比如有问题，那我不可能等你一天吧，可能最多我只能等你10秒
     * 10秒后，还没有执行完毕，就弹出一个超时异常吧
     */
    int timeout() default TransactionDefinition.TIMEOUT_DEFAULT;
 
    /**
     * 是否是只读事务，比如某个方法中只有查询操作，我们可以指定事务是只读的
     * 设置了这个参数，可能数据库会做一些性能优化，提升查询速度
     */
    boolean readOnly() default false;
 
    /**
     * 定义零(0)个或更多异常类，这些异常类必须是Throwable的子类，当方法抛出这些异常及其子类异常的时候，spring会让事务回滚
     * 如果不配做，那么默认会在 RuntimeException 或者 Error 情况下，事务才会回滚 
     */
    Class<? extends Throwable>[] rollbackFor() default {};
 
    /**
     * 和 rollbackFor 作用一样，只是这个地方使用的是类名
     */
    String[] rollbackForClassName() default {};
 
    /**
     * 定义零(0)个或更多异常类，这些异常类必须是Throwable的子类，当方法抛出这些异常的时候，事务不会回滚
     */
    Class<? extends Throwable>[] noRollbackFor() default {};
 
    /**
     * 和 noRollbackFor 作用一样，只是这个地方使用的是类名
     */
    String[] noRollbackForClassName() default {};
 
}
```

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

# Spring事务失效常见情况

## 未启动Spring事务管理功能

@EnableTransactionManagement注解用来启用Spring事务自动管理事务的共鞥你

## 方法不是public类型

@Transactional可以用在类上、接口上、public方法上，如果将@Transactional用在非public方法上，事务将失效

## 数据源未配置事务管理器

spring是通过事务管理器来管理事务的，一定不能忘记配置事务管理器，要注意为每一个数据源配置一个事务管理器

## 自身调用问题

Spring是通过aop的方式，对需要Spring管理事务的bean生成代理对象，然后通过代理对象拦截了目标方法的执行，在方法前后添加事务功能，所以必须通过代理对象调用目标方法的时候，事务才会生效

**案例**

```java
@Component
public class UserService {
    public void m1(){
        this.m2();
    }
    
    @Transactional
    public void m2(){
        //执行db操作
    }
}
```

m1中通过this的方式调用m2方法，而this并不是代理对象，所以事务无效

## 异常类型错误

Spring事务回滚机制：对业务方法进行try-catch，当捕获到有指定的异常时，spring自动对事务进行回滚

如果并不是指定的异常（默认：RuntimeException、Error）外的异常不会进行回滚，事务失效

自定义异常处理器参数

```java
@Transactional(rollbackFor = {异常类型列表})
```

## 异常被吞

当业务方法抛出异常了，Spring感知到异常的时候，才会做事务回滚的操作，若方法内部将异常吞了，那么事务无法感知到异常了，事务就不会回滚

在方法中，异常被catch处理了

## 业务和Spring事务代码必须在一个线程中

Spring事务实现中使用ThreadLocal，如果不是同一个线程，那么数据将不能共享
