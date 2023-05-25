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

![Bean生命周期](D:\WorkSpace\Mnsx-Note\2023\Picture\Spring\Bean生命周期.png)

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

```java
protected void beforeSingletonCreation(String beanName) {
    if (!this.inCreationCheckExclusions.contains(beanName) && !this.singletonsCurrentlyInCreation.add(beanName)) {
        throw new BeanCurrentlyInCreationException(beanName);
    }
}
```

singletonsCurrentlyInCreation是用来记录目前正在创建中的bean名称列表，this.singletonsCurrentlyInCreation.add(beanName)返回false，说明beanName已经在列表中，此时会抛出循环依赖异常BeanCurrentlyInCreationException

```java
public BeanCurrentlyInCreationException(String beanName) {
    super(beanName,
          "Requested bean is currently in creation: Is there an unresolvable circular reference?");
}
```

prototype类型，`org.springframework.beans.factory.support.AbstractBeanFactory#doGetBean`

```java
//检查正在创建的bean列表中是否存在beanName，如果存在，说明存在循环依赖，抛出循环依赖的异常
if (isPrototypeCurrentlyInCreation(beanName)) {
    throw new BeanCurrentlyInCreationException(beanName);
}
 
//判断scope是否是prototype
if (mbd.isPrototype()) {
    Object prototypeInstance = null;
    try {
        //将beanName放入正在创建的列表中
        beforePrototypeCreation(beanName);
        prototypeInstance = createBean(beanName, mbd, args);
    }
    finally {
        //将beanName从正在创建的列表中移除
        afterPrototypeCreation(beanName);
    }
}
```

## 如何解决循环依赖问题

* 通过构造器方式注入依赖

  通过构造器方式导致的循环依赖是无法解决的

* 通过非构造器方式注入依赖

  由于单例bean在Spring容器中肯定只会存在一个，所以Spring中肯定有一个缓存用来存放已经创建好的bean，在获取单例bean之前，先从缓存中查找，如果找到直接返回，如果没有找到再创建，然后放入单例缓存中

  ```java
  Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);
  ```

  > 1.spring轮询准备创建2个bean：serviceA和serviceB
  > 2.spring容器发现singletonObjects中没有serviceA
  > 3.调用serviceA的构造器创建serviceA实例
  > 4.serviceA准备注入依赖的对象，发现需要通过setServiceB注入serviceB
  > 5.serviceA向spring容器查找serviceB
  > 6.spring容器发现singletonObjects中没有serviceB
  > 7.调用serviceB的构造器创建serviceB实例
  > 8.serviceB准备注入依赖的对象，发现需要通过setServiceA注入serviceA
  > 9.serviceB向spring容器查找serviceA
  > 10.此时又进入步骤2了

  出现循环了，需要在第三步完成之后，将Service放入singletonObjects中，这样就可以解决循环了

  **Spring中解决方式与上面差不多，Spring内部使用三级缓存来解决**

  ```java
  /** 第一级缓存：单例bean的缓存 */
  private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);
   
  /** 第二级缓存：早期暴露的bean的缓存 */
  private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);
   
  /** 第三级缓存：单例bean工厂的缓存 */
  private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);
  ```

  首先serviceA会调用

  ```java
  org.springframework.beans.factory.support.AbstractBeanFactory#doGetBean
   
  protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
                                @Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {
      //1.查看缓存中是否已经有这个bean了
      Object sharedInstance = getSingleton(beanName);
      if (sharedInstance != null && args == null) {
          bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
      }else {
          //若缓存中不存在，准备创建这个bean
          if (mbd.isSingleton()) {
              //2.下面进入单例bean的创建过程
              sharedInstance = getSingleton(beanName, () -> {
                  try {
                      return createBean(beanName, mbd, args);
                  }
                  catch (BeansException ex) {
                      throw ex;
                  }
              });
              bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
          }
      }
      return (T) bean;
  }
  ```

  调用getSingleton方法查看缓存中是否有这个bean

  ```java
  public Object getSingleton(String beanName) {
      return getSingleton(beanName, true);
  }
  ```

  然后会进入以下方法尝试从三级缓存中找到这个bean，**只有当第二个参数为true时，才会从三级中找，否则只会找一二级**

  ```java
  //allowEarlyReference:是否允许从三级缓存singletonFactories中通过getObject拿到bean
  protected Object getSingleton(String beanName, boolean allowEarlyReference) {
      //1.先从一级缓存中找
      Object singletonObject = this.singletonObjects.get(beanName);
      if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
          synchronized (this.singletonObjects) {
              //2.从二级缓存中找
              singletonObject = this.earlySingletonObjects.get(beanName);
              if (singletonObject == null && allowEarlyReference) {
                  //3.二级缓存中没找到 && allowEarlyReference为true的情况下,从三级缓存中找
                  ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                  if (singletonFactory != null) {
                      //三级缓存返回的是一个工厂，通过工厂来获取创建bean
                      singletonObject = singletonFactory.getObject();
                      //将创建好的bean丢到二级缓存中
                      this.earlySingletonObjects.put(beanName, singletonObject);
                      //从三级缓存移除
                      this.singletonFactories.remove(beanName);
                  }
              }
          }
      }
      return singletonObject;
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

