# SpringMVC简介

## MVC简介

MVC是一种软件架构的思想，将软件按照模型、视图、控制器划分

M：Model，模型层，指工程中的JavaBean，作用是处理数据

JavaBean分为两类：

* 一类称为实体类Bean：专门存储业务数据的
* 一类称为业务处理Bean：指Service或Dao对象，专门用于处理业务逻辑和数据访问

V：View，视图层，指工程中的html或jsp等页面，作用是与用户进行交互，展示数据

C：Controller，控制层，指工程中的Servlet，作用是接受请求和响应服务器

MVC的工作流程：

用户通过视图层发送请求到服务器，在服务器中请求被Controller接受，Controller调用响应的Model层处理请求，处理完毕将结果返回到Controller，Controller再根据请求处理的结果找到响应的View视图，渲染数据后最终响应给浏览器

## SpringMVC简介

SpringMVC是Spring的一个后续产品，是Spring的一个子项目

SpringMVC是Spring为表述层开发提供的一套完整的解决方法。再表述层框架历经Strust、WebWork、Strust2等诸多产品的历代迭代之后，目前业界普遍选择SpringMVC作为JavaEE项目表述层开发的首选方法

> 三层架构分为表述层、业务逻辑层、数据访问层，表述层表示前台页面和后台servlet

## SpringMVC特点

* Spring的家族原生产品，与IOC容器等基础设施无缝对接
* 基于原生的Servlet，通过了功能强大的前端控制器DispathcerServlet，对请求和响应进行统一处理
* 表述层各细分领域需要解决的问题各方面覆盖，提供全方面解决方案
* 代码清新简洁，大幅度提升开发效率
* 内部组件化程度高，可插拔式组件即插即用
* 性能卓越，适合现代大型、超大型互联网项目要求

# SpringMVC快速入门

1. 创建maven工程

    1. 添加web模块
    2. 打包方式web
    3. 引入依赖

2. 配置web.xml

   注册SpringMVC的前端控制器DispatcherServlet

    1. 默认配置方式

       此配置作用下，SpringMVC的配置文件默认位于WEB-INF下，默认名称为\<servlet-name\>-servlet.xml，例如，以下配置所对应SpringMVC的配置文件位于WEB-INF下，文件名为SpringMVC-servlet.xml

   ```xml
   <servlet>
   	<servlet-name>SpringMVC</servlet-name>
       <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
   </servlet>
   <servlet-mapping>
   	<servlet-name>SpringMvc</servlet-name>
       <!--
   		设置SpringMvc核心控制器能够处理的请求的请求路径
   		/所匹配的请求可以是/login或.html或.js或.css方式的请求路径
   		但是/不能匹配jsp请求路径的请求
   	-->
       <url-pattern>/</url-pattern>
   </servlet-mapping>
   ```

    2. 扩展配置方式

       可通过init-param标签设置SpringMVC配置文件的位置和名称，通过load-on-startup标签设置SpringMVC前端控制器DispatcherServlet的初始化时间

   ```xml
   <servlet>
   	<servlet-name>SpringMVC</servlet-name>
       <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
       <init-param>
       	<param-name>contextConfigLocation</param-name>
           <param-value>classpath:springMVC.xml</param-value>
       </init-param>
       <!--将前端控制器DispacherServlet启动时间设置到服务器启动时-->
       <load-on-startup>1</load-on-startup>
   </servlet>
   <servlet-mapping>
   	<servlet-name>SpringMvc</servlet-name>
       <!--
   		设置SpringMvc核心控制器能够处理的请求的请求路径
   		/所匹配的请求可以是/login或.html或.js或.css方式的请求路径
   		但是/不能匹配jsp请求路径的请求
   	-->
       <url-pattern>/</url-pattern>
   </servlet-mapping>
   ```

   > <url-pattern>标签中使用/和/\*的区别：
   >
   > /所匹配的请求可以是/login或.html或.js或.css方式的请求路径，但是/不能匹配.jsp请求路径的请求
   >
   > 因此就可以避免再访问jsp页面时，该请求被DispatcherServlet处理，从而找不到相应的页面/\*则能够匹配所有请求，例如再使用过滤器时，若需要队所有请求进行过滤，就需要使用/\*的写

    3. 创建请求控制器

   ```java
   @Contoller
   public class HelloController {
       @RequestMapping("/")
       public String index(){
           //返回视图名称
           return index;
       }
   }
   ```

   ```xml
   <!--组件扫描-->
   <context:component-scan base-package="top.mnsx.springMvc"/>
   
   <!--配置Thymeleaf视图解析器-->
   <bean id="viewResolver" class="org.thymeleaf.spring5.view.ThymeleafViewResolver">
   	<property name="order" value="1"/>
       <property name="characterEncoding" value="UTF-8"/>
       <property name="templateEngine">
       	<bean class="org.thymeleaf.spring5.springTemplateEngine">
           	<property name="prefix" value="/WEB-INF/templates"/>
               <property name="suffix" value=".html"/>
               <property name="templateMode" value="HTML5"/>
               <property name="characterEncoding" value="UTF-8"/>
           </bean>
       </property>
   </bean>
   ```

   ```html
   <html lang="en" xmlns:th="http://www.thymeleaf.org">
       <!--...-->
   </html>
   ```

    4. 总结

       浏览器发送请求，若请求地址符合前端控制器的url-pattern，该请求就会被前端控制器DispatcherServlet处理。前端控制器会会读取SpringMVC的核心配置文件，通过扫描组件来找到控制器，将请求地址和控制器@RequestMapping注解的value属性值进行匹配，若匹配成功，该注解所表示的控制器方法就是处理请求的方法。处理请求的方法需要返回一个字符串类型的视图类型，该视图名称会被视图解析器解析，加上前缀和后缀组成视图路径，通过Thymeleaf对视图进行渲染，最终转发到视图所对应的页面

# @RequestMapping注解

## @ReqeustMapping注解的功能

从注解名称上可以看出，@RequestMapping注解的作用就是将请求和处理请求的控制器方法关联起来，建立起映射关系

SpringMVC接收到指定的请求，就会来找到在映射关系中对应的控制器方法来处理这个请求

## @RequestMapping注解位置

@RequestMapping标识一个类：设置映射请求的请求路径的初始化信息

@RequestMapping标识一个方法：设置映射请求的请求路径的具体信息

```java
@Controller
@RequestMapping("/hello")
public class RequestMappingController {
    @RequestMapping("/testRequestMapping")
    public String success() {
        return "success";
    }
}
```

## @RequestMapping的value属性

@RequestMapping注解的value属性通过请求的请求地址匹配请求映射

@RequestMapping注解的value属性是一个字符串类型的数组，表示该请求映射能够匹配多个请求地址所对应的请求

@RequestMapping注解的value属性必须设置，至少通过请求地址匹配请求映射

```html
<a th:href="@{/testRequestMapping}">测试@RequestMapping的value属性</a><br/>
<a th:href="@{/test}">测试@RequestMapping的value属性</a>
```

```java
@RequestMapping(value = {"/testRequestMapping", "/test"})
public String testRequestMapping() {
    return "success";
}
```

## @RequestMapping的method属性

@RequestMapping注解的method属性通过请求的请求方式（get或post）匹配请求映射

@RequestMapping注解的method属性是一个RequestMethod类型的数组，表示该请求映射能够匹配多个请求方式的请求

若当前请求的请求地址满足请求映射的value属性，但是请求方式不满足method属性，则浏览器报错405：Request method ‘POST’ not supported

```html
<a th:href="@{/test}">测试@RequestMapping的value属性</a><br/>
<form th:action="@{/test}" method="post">
    <input type="submit" value="测试RequestMapping注解的method属性"/>
</form>
```

```java
@RequestMapping(value={"/testRequestMapping", "/test"},
         		method={RequestMathod.GET, RequestMethod.POST}
                )
public String testRequestMapping(){
    return "success";
}
```

> 1. 对于处理指定请求方式的控制器方法，SpringMVC中提供了@RequestMapping的派生注解
     >
     >    处理get请求的映射——》@GetMapping
     >
     >    处理post请求的映射——》@PostMapping
     >
     >    处理put请求的映射——》@PutMapping
     >
     >    处理delete请求的映射——》@DeleteMapping
>
> 2. 常用的请求方法有get、post、put、delete
     >
     >    但是目前浏览器只支持get和post，若在form表单提交时，method设置为其他请求方式的字符串，则按照默认的请求方式get来处理
     >
     >    若要发送put和delete请求，则需要通过spring提供的过滤器HiddenHttpMethodFilter

## @RequestMapping注解的params属性（了解）

@RequestMapping注解的params属性通过请求的请求参数匹配请求映射

@RequestMapping注解的params属性是一个字符串类型的数组，可以通过四种表达式设置请求参数和请求映射的匹配情况

"param": 要求请求映射所匹配的请求必须携带param请求参数

"!param": 要求请求映射所匹配的请求必须不携带param请求参数

"param=value": 要求请求映射所匹配的请求必须携带param请求参数且param=value

"param!=value": 要求请求映射所匹配的请求必须携带param请求参数但是param!=value

```html
<a th:href="@{/test(username='admin', password=123123)}">测试@RequestMapping的params属性</a>
```

```java
@RequestMapping(value={"/testRequestMapping", "/test"}, method={RequestMethod.GET, RequestMethod.POST}, params={"username", "password!=123123"})
public String testRequestMapping(){
    return "success";
}
```

> 如果当前请求满足@RequestMapping注解的value和method属性，但是不满足params属性，此时页面报错400：Paramter conditions "username, password!=123123" not met for actual request parameters: username={admin}, password={123123}

## @RequestMapping注解的headers属性（了解）

@Request Mapping注解的headers属性通过请求的请求头信息匹配请求映射

@RequestMapping注解的headers属性是一个字符串类型的数组，可以通过四种表达式设置请求头信息和请求映射的匹配关系

"header": 要求请求映射所匹配的请求必须携带header请求头信息

"!header": 要求请求映射的请求必须不懈怠header请求头信息

"header=value": 要求请求映射所匹配的请求必须携带header请求头信息且header=value

"header!=value": 要求请求映射所匹配的请求必须携带header请求头信息且header!=value

> 若当前请求满足@RequestMapping注解的value和method属性，但是不满足headers属性，此时页面显示404错误，即资源未找到

## SpringMVC支持ant风格的路径

?——表示任意的单个字符

\*——表示零个或多个字符

\*\*——表示任意的一层或多层目录

**注意：使用\*\*时，只能使用/\*\*/xxx的方式**

## SpringMVC支持路径中的占位符

原始方式——/deleteUser?id=1

rest方式——/deleteUser/1

SpringMVC路径中的占位符常用于restful风格中，当请求路径中将默写数据通过路径的方式传输到服务器中，就可以再相应的@RequestMapping注解的value属性中通过占位符{xxx}表示传输的数据，再通过@PathVariable注解，将占位符所表示的数据赋值给控制器方式的形参

```java
@RequestMapping("/testPath/{id}")
public String testPath(@PathVariable("id") Integer id) {
    System.out.println("id: " + id);
    return "success";
}
```

# SpringMVC获取请求参数

## 通过servletAPI获取

将HttpServletRequest作为控制器方法的形参，此时HttpServeltRequest类型的参数表示封装了当前请求的请求报文的对象

```java
@RequestMapping("/testParam")
public String testParam(HttpServletRequest request){
    String username = request.getParameter("username");
    String password = request.getParameter("password");
    System.out.println("username: " + username + ", password: " + password);
    return "success";
}
```

## 通过控制器方法的形参获取请求参数

在控制器方法的形参位置，设置和请求参数同名的形参，当浏览器发送请求，匹配到请求映射时，在DispatcherServlet中就会将请求参数赋值给响应的形参

```html
<a th:href="@{/testParam(username='admin', password=123123)}">测试获取请求参数</a>
```

```java
@RequestMapping("/testParam")
public String testParam(String username, String password) {
    System.out.println("username: " + username + ", password: " + password);
    return "success";
}
```

## @RequestParam

@RequestParam是将请求参数和控制器方法的形参创建映射关系

@RequestParam注解一共有三个属性：

value——指定为形参赋值的请求参数的参数名

required——设置是否必须传输此请求参数，默认为true

若设置为true时，则当前请求必须传输value所指定的请求参数，若没有传输该请求参数，且没有设置defaultValue属性，则页面报错400：Required String Paramter ‘xxx’ is not present;
若设置为false，则当前请求不是必须传输value所指定的请求参数，若没有传输，则注解所表示的形参的值为null

defaultValue——不管required属性值为true或false，当value所指定的请求参数没有传输时，则使用默认值为形参赋值

## @RequestHeader

@RequestHeader是将请求头信息和控制器方法的形参创建映射关系

@RequestHeader注解一共有三个属性：value、required、defaultValue，用法同@RequestHeader

```java
@RequestMapping("/testParam")
public String testParam(@RequestHeader("Host") String host) {
    System.out.println("host: " + host);
}
```

## @CookieValue

@CookieValue是将Cookie数据和控制器方法的形参创建映射关系

@CookieValue注解一共有三个属性：value、required、defaultValue，用法等同@RequestParam

```java
@RequestMapping("/testParam")
public String testParam(@CookieValue("curId") String curId){
    System.out.println("curId: " + curId);
}
```

## 通过POJO获取请求参数

可以在控制器方法的形参位置设置一个实体类类型的形参，此时若浏览器传输的请求参数的参数名和实体类中的属性名一致，那么请求参数就会为此属性赋值

```html
<form th:action="@{/testPOJO}" method="post">
    用户名：<input type="text" name="username"><br/>
    密码：<input type="password" name="password"><br/>
    性别：<input type="redio" name="sex" value="男">男<input type="radio" name="sex" value="女">女<br/>
    年龄：<input type="text" name="age"><br/>
    邮箱：<input type="text" name="email"><br/>
    <input type="submit">
</form>
```

```java
@RequestMapping("/testPOJO")
public String testPOJO(User user){
    System.out.println(user);
    return "success";
}
```

## 解决获取请求参数的乱码问题

解决获取请求参数的乱码问题，可以使用SpringMVC提供的编码过滤器CharacterEncodingFilter，到那时必须在web.xml中进行注册

```xml
<filter>
	<filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
    	<param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
    <init-param>
    	<param-name>forceResponseEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
<filter-mapping>
	<filter-name>CharacterEncoding</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

# 域对象共享数据

## 使用ServletAPI向request域对象共享数据

```java
@RequestMapping("/testServletAPI")
public String testServletAPI(HttpServletRequest request){
    request.setAttibute("test", "hello");
    return "success";
}
```

## 使用ModelAndView向request域对象共享数据

```java
@RequestMapping("/testModelAndView")
public ModelAndView testModelAndView() {
    ModelAndView mav = new ModelAndView();
    mav.addObject("test", "hello");
    mav.setViewName("success");
    return mav;
}
```

## 使用Model向request域对象共享数据

```java
@RequestMapping("/testModel")
public String testModel(Model model) {
    model.addAttibute("test", "hello");
    return "success";
}
```

## 使用map向request域对象共享数据

```java
@RequestMapping("/testMap")
public String testMap(Map<String, Object> map){
    map.put("test", "hello");
    return "success";
}
```

## 使用ModelMap向request域对象共享数据

```java
@RequestMapping("/testModelMap")
public String testModelMap(ModelMap modelMap){
    maodelMap.addAttibute("test", "hello");
    return "success";
}
```

## Model、ModelMap、Map的关系

Model、ModelMap、Map类型的参数其实本质上都是BindingAwareModelMap类型的

```java
public interface Model{}
public class ModelMap extends LinkedHashMap<String, Object> {}
public class ExtendModelMap extends ModelMap implements Model {}
public class BindingAwareModelMap extends ExtendedModelMap {}
```

## 向session域共享数据

```JAVA
@RequestMapping("/testSession")
public String testSession(HttpSession session) {
    session.setAttribute("test", "hello");
    return "success";
}
```

## 向application域共享数据

```java
@RequestMapping("/testApplication")
public String testApplication(httpSession session) {
    ServletContext application = session.getServletContext();
    application.setAttribute("test", "hello");
    return "success";
}
```

# SpringMVC的视图

SpringMVC中的视图是View接口，视图的作用是渲染数据，将模型Model中的数据展示给用户

SpringMVC视图的种类很多，默认有转发视图InternalResourceView和重定向视图RedirectView

当工程中引入jstl的依赖，战法视图汇自动转换为stlView

若使用的视图技术为Thymeleaf，在SpringMVC的配置文件中配置了Thymeleaf的视图解析器，由此视图解析器解析之后所得到的是ThymeleafView

## ThymeleafView

当控制器方法中所设置的视图名称没有任何前缀时，此时的视图名称会被SpringMVC配置文件中所配置的视图解析器解析，视图名称拼接视图前缀和视图后缀所得到的最终路线，会通过转发的方式实现跳转

```java
@RequestMapping("/testHello")
public String testHello() {
    return "hello";
}
```

## 转发视图

SpringMVC中默认的转发视图是InternalResourceView

SpringMVC中创建转发视图的情况

当控制器方法中所设置的视图名称以“forward:”为前缀时，创建InternalResourceView视图，此时的视图名称不会被SpringMVC配置文件中所配置的视图解析器解析，而是会将前缀“forward:
“去掉，剩余的部分作为最终路径通过转发的方式进行跳转

```java
@RequestMapping("/testForward")
public String testForward() {
    return "forward:/testThymeleafView";
}
```

## 重定向视图

SpringMVC中默认的重定向视图时RedirectView

当控制器方法中设置的视图名称以”redirect“为前缀时，创建RedirectView视图，此时的视图名称不会被SpringMVC配置文件中所配置的视图解析器解析，而会将前缀”redirect“去掉，剩余部分作为最终路径通过重定向的方法实现跳转

```java
@RequestMapping("/testRedirect")
public String testRedirect() {
    return "redirect:/testThymeleafView";
}
```

## 视图控制器view-controller

当控制器方法中，仅仅用来实现页面跳转，即只需要设置视图名称时，可以将处理方法使用view-controller标签进行表示

```xml
<!--
	path:设置处理的请求地址
	view-name：设置请求地址所对应的视图名称
-->
<mvc:view-controller path="/testView" view-name="success"></mvc:view-controller>
```

> 当SpringMVC中设置一个view-controller后，其他控制器中的请求映射群不失败，此时需要在SpringMVC核心配置文件中设置开启mvc注解驱动标签
>
> ```xml
> <mvc:annotation-drive/>
> ```
>
> 当使用默认的Servlet来响应静态文件之后，为了防止动态文件的匹配失败，也需要使用这个配置
>
> ```xml
> <mvc:default-servlet-handler/>
> ```

# RESTFul

## RESTFul简介

REST：Representational State Transfer，表现层资源状态转移

**资源**

**资源的表述**

**状态转移**

## RESTFul的实现

具体说，就是HTTP协议里面，四个表示操作方式的动词：GET、POST、PUT、DELETE

它们分别对应四种基本操作：GET用来获取资源、POST用来新建资源、PUT用来更新资源、DELETE用来删除资源

RestFul风格提倡URL地址使用统一的风格设计，从前到后哥哥单词使用斜杠分开，不适用问好键值对方式携带请求参数，而是将要发送给服务器的数据作为URL地址的一部分，以保证风格的一致性

| 操作     | 传递方式         | REST风格               |
| -------- | ---------------- | ---------------------- |
| 查询操作 | getUserById?id=1 | user/1->get请求方式    |
| 保存操作 | saveUser         | user->post请求方式     |
| 删除操作 | deleteUser?id=1  | user/1->delete请求方式 |
| 更新操作 | updateUser       | user->put请求方式      |

## HiddenHttpMethodFilter

由于浏览器支支持发送get和post方式的请求，那么SpringMVC提供了HiddenHttpMethodFilter帮助我们将Post请求转换成Delete和Put请求

HiddenHttpMethodFilter使用条件——

1. 当前请求的请求方式必须为Post
2. 当前请求必须传输参\_method

```xml
<filter>
	<filter-name>HiddenHttpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>
<filter-mapping>
	<filter-name>HiddenHttpMethodFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

> CharacterEncodingFilter需要放到最前面，如果CharacterEncodingFilter之前，有属性被调用，则过滤器将会失效

# HttpMessageConverter

HttpMessageConverter，报文信息转换器，将请求报文转换成java对象，或者java对象转换位响应报文

HttpMessageConverter提供了两个注解和两个类型，@RequestBody，@ResponseBody，RequestEntity，ResponseEntity

## @RequestBody

@RequestBody可以获取请求体，需要再控制器中设置一个形参，使用@RequestBody进行表示，当前请求的请求体就会位当前注解所表示的形参赋值

```html
<form th:action="@{/testRequestBody}" method="post">
    用户名：<input type="text" name="username"><br/>
    密码：<input type="password" name="password"><br/>
    <input type="submit">
</form>
```

```java
@RequestMapping("/testRequestBody")
public String testRequestBody(@RequestBody String requestBody) {
    System.out.println("requestBody:" + requestBody);
    return "success";
}
```

## RequestEntity

RequestEntity封装请求报文的一种类型看，需要控制器方法的形参中设置该类型的形参，当前请求的请求报文就会赋值给该形参，可以通过getHeaders()获取请求头信息，通过getBody()获取请求体信息

```java
@RequestMapping("/testRequestEntity")
public String testRequestEntity(RequestEntity<String> requestEntity) {
    System.out.println("requestHeader:" + requestEntity.getHeader());
    System.out.println("requestBody:" + requestEntity.getBody());
    return "success";
}
```

## @ResponseBody

@ResponseBody用于表示一个控制器方法，可以将该方法的返回值直接作为响应报文的响应体响应道浏览器

```java
@RequestMapping("/testResponseBody")
@ResponseBody
public String testResponseBody(){
    return "success";
}
```

## SpringMVC处理ajax

1. 请求超链接

```html
<div id="app">
    <a th:href="@{/testAjax}" @click="testAjax">testAjax</a>
</div>
```

2. 通过vue和axios处理点击事件

```html
<script type="text/javascript" th:src="@{/static/js/vue.js}"></script>
<script type="text/javascript" th:src="@{/static/js/axios.min.js}"></Script>
<script type="text/javascript">
	var vue = new Vue({
        el:"#app",
        methods:{
        	testAjax: function(event){
                axios({
                    method:"post",
                    url:event.target.href,
                    params:{
                        username:"admin",
                        password:"123123"
                    }
                }).then(function(response) {
                    alert(response.data);
                });
                event.preventDeafault();
            }   
        }
    })
</script>
```

```java
@RequestMapping("/testAxois")
@ResponseBody
public String testAxios(String username, String password){
    System.out.println(username + ", " + password);
    return "hello";
}
```

## @RestController注解

@RestController注解是SpringMVC提供的一个复合注解，标识再控制器的类上，就相当于为类添加了@Controller注解，并且为其中的每个方法添加了@ResponseBody注解

## ResponseEntity

ResponseEntity用于控制器方法的返回值类型，该控制器方法的返回值就是响应到浏览器的响应报文

# 文件上传和下载

## 文件下载

使用ResponseEntity实现下载文件的功能

```java
@RequestMapping("/testDown")
public ResponseEntity<byte[]> testResponseEntity(HttpSession session) throws IOException {
    ServletContext servletContext = session.getServletContext();
    String realPath = servletContext.getRealPath("/static/img/1.jpg");
    InputStream is = new FileInputStream(realPath);
    byte[] bytes = new byte[is.available()];
    is.read(bytes);
    MultivalueMap<String, String> headers = new HttpHeaders();
    headers.add("Content-Disposition", "attachment;filename=1.jpg");
    HttpStatus statusCode = HttpStatus.OK;
    ResponseEntity<byte[]> responseEntity = new ResponseEntity<>(bytes, headers, statusCode);
    io.close();
    return responseEntity;
}
```

## 文件上传

文件上传要求表单的请求方式必须为post，并且添加属性enctype=“multipart/form-data”

SpringMVC中将上传的文件封装到MultipartFile对象中，通过此对象获取文件相关信息

```html
<form th:action="@{/testUp}" method="pose" enctype="multipart/form-data">
    头像：<input type="file" name="photo"><br/>
    <input type="submit" value="上传">
</form>
```

引入commons-fileupload依赖

```java
	@ReqeustMapping("/testUp")
public String testUp(MultipartFile photo, HttpSession session) {
    String fileName = photo.getOriginalFilename();
    ServletContext servletContext = session.getServletContext();
    String photoPath = servletContext.getRealPath("photo");
    File file = new File(photoPath);
    if(!file.exists()) {
        file.mkdir();
    }
    String finalPath = photoPath + File.separator + fileName;
    photo.transferTo(new File(finalPath));
    return "success"
}
```

```xml
<!--文件上传解析器-->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver"
```

## 文件上传重名问题

```java
String suffixName = fileName.substring(fileName.lastIndexOf("."));
String uuid = UUID.randomUUID().toString();
fileName = uuid + suffixName;
```

# 拦截器

## 拦截器的配置

SpringMVC中的拦截器用于拦截控制器方法的执行

SpringMVC中的拦截器需要实现HandlerInterceptor或者继承HandlerInterceptorAdapter类

SpringMVC拦截器必须再SpringMVC的配置文件中进行配置

```xml
<mvc:interceptors>
	<!--<bean class="top.mnsx.interceptor.FirstInterceptor"></bean>-->
    <!--<ref bean="firstInterceptor"></ref>-->
    <mvc:interceptor>
	<mvc:mapping path="/**"/>
    <mvc:exclude-mapping path="/testRequestEntity"/> 
    </mvc:interceptor>
    <ref bean="firstInterceptor"/>
</mvc:interceptors>
```

## 拦截器的三种抽象方法

SpringMVC中的三个抽象方法：

preHandle：控制器方法执行之前执行preHandle()，其boolean类型的返回值表示是否拦截或者放行，分会true为放行，即调用控制器方法；返回false表示拦截，即不调用控制器方法

postHandler：控制器方法执行之后执行postHandle()

afterComplation：处理完视图和模型数据，渲染视图完毕后执行afterComplation()

## 多个拦截器的执行顺序

若每个拦截器的preHandler()都返回true

此时多个拦截器的执行顺序和拦截器在SpringMVC的配置顺序有关：

preHandle()会按照配置的顺序执行，而postHandler()和afterComplation()会按照配置的反序执行

若某个拦截器的preHandle()返回了false

preHandle()返回false和它之前的拦截器的preHandler()都会执行，postHandler()都不执行，返回false的拦截器之前的拦截器的afterComplation()会执行

# 异常处理器

## 基于配置的异常处理

SpringMVC提供了一个处理控制器方法执行过程中所出现的异常的接口：HandlerException

HandlerExceptionResolver接口的实现类有：DefaultHandlerExceptionResolver和SimpleMappingExceptionResolver

SpringMVC提供了自定义的异常处理器SimpleMappingExceptionResovler，使用方式——

```xml
<bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
	<property name="exceptionMappings">
    	<props>
            <prop key="java.lang.ArithmeticException">error`</prop>
        </props>
    </property>
    <!-- 设置键异常信息设置一个属性名，将出现的异常信息在请求与中进行共享给 -->
    <property name="exceptionAttribute" value="ex"></property>
</bean>
```

## 基于注解的异常处理

```java
@ControllerAdvice
public class ExceptionController {
    @ExceptionHandler(ArithmeticException.class)
    public String handlerArtithmeticException(Exception ex, Model model) {
        model.addAttribute("ex", ex);
        return "error";
    }
}
```

# 注解配置SpringMVC

## 常规XML配置

### 配置web.xml

> 注意web.xml的版本要为最新版

* 需要绑定一个SpringMVC配置文件
* 设置启动级别为1
* 设置映射路径为/

```xml
<!--1.注册DispatcherServlet-->
<servlet>
	<servlet-name>Springmvc</servlet-name>
    <servlet-class>org.springframeword.web.servlet.DispatcherServlet</servlet-class>
    <!--关联一个springmvc的配置文件：[servlet-name]-serlet.xml-->
    <init-param>
    	<param-name>contextConfigLocation</param-name>
        <param-value>classpath:springmvc-servlet.xml</param-value>
    </init-param>
    <!--启动级别-1-->
    <load-on-startup>1</load-on-startup>
</servlet>

<!--/ 匹配所有请求; （不包括.jsp）-->
<!--/* 匹配所有请求; （包括.jsp）-->
<servlet-mapping>
	<servlet-name>springmvc</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

### 编写SpringMVC配置文件

> 上述DispatcherServlet绑定该配置文件，主要配置一下几个部分

1. 自动扫描包

   让指定包下的注解生效，由IOC容器统一管理

   ```xml
   <context:component-scanb base-package="controller"/>
   ```

2. 过滤静态资源

   它会像一个检查员，对进入DispatcherServlet的URL进行筛查，如果发现是静态资源的请求，就将该请求转由Web应用服务器默认的Servlet处理，如果不是静态资源的请求，才由DispatcherServlet继续处理

   ```xml
   <mvc:default-servlet-handler/>
   ```

   | 静态资源                                                     | 动态资源                                                     |
      | ------------------------------------------------------------ | ------------------------------------------------------------ |
   | 可以理解为前端的固定页面，这里面包含Html、css、js、图片等，不需要查询数据库也不需要程序处理，直接就能够显示的页面，如果想要修改内容则必须修改页面，但是访问效率相当高 | 需要程序处理或者从数据库中读取数据，能够根据不同的条件在页面显示不同的数据，内容更新不需要修改页面但是访问速度不及静态页面 |

3. 支持mvc注解驱动

   > 在spring中一般使用@RequestMaping注解来完成映射关系

   为了使其生效，必须向上下文中注册两个实例：

* DefaultAnnotationHadnlerMapping（处理器映射器）
* AnnotationMethodHandlerAdapter（处理器适配器）

```xml
<mvc:annotation-driven/>
```

 annotation-drien配置帮助我们自动完成上述两个实例的注入

4. 视图解析器

   ```xml
   <!--视图解析器-->
   <bean class="org.springframeword.web.servlet.view.InternalResourceViewResolver">
   	<property name="prefix" value="/WEB-INF/jsp/"/>
       <property name="suffix" value=".jsp"/>
   </bean>
   ```

**SpringMVC必须配置的三大件**

* 处理器映射器
* 处理器适配器
* 视图解析器

当我们使用注解实现时，只需要手动配置视图解析器，另外两个只需要开启注解驱动即可

### 创建controller

* @Controller是为了让SpringIOC容器初始化时自动扫描
* @Request Mapping是为了映射请求路径，这里因为类与方法上都有映射所以访问时应该是/HelloController/hello
* 方法中声明Model类型的参数是为了把Action中的数据带到视图中

* 方法返回的结果是视图的名称Hello，加上配置文件中的前后缀变成WEB-INF/jsp/hello.jsp

## 创建初始化类，代替web.xml

在Servlet3.0环境中，容器会在类路径中查找实现javax.servlet.ServletContatinerInitializer接口的类，如果找到的话就用它来配置Servlet容器

Spring提供了这个接口的实现，名称SpringServletContainInitializer，这类反过来又会查找实现WebApplicationInitlalizer的类并且将配置的任务交给他们来完成。Spring3.2引入了一个便利的WebApplicationInitializer基础实现，名为AbstractAnnotationConfigDispatcherServletInitializer，当我们的类扩展了AbstractAnnotationConfigDispatcherServletInitlalizer并将其部署到Servlet3.0容器的时候，容器会自动发现它，并用它来配置Servlet上下文

```java
public class WebInit extends AbstractAnnotationConfigDispatcherServletInitializer {
    /**
     * 指定Spring配置类
     * @return
     */
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[]{SpringConfig.class};
    }

    /**
     * 指定SpringMVC配置类
     * @return
     */
    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{WebConfig.class};
    }

    /**
     * 指定DispatcherServlet的映射规则——urlPattern
     * @return
     */
    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }

    @Override
    protected Filter[] getServletFilters() {
        CharacterEncodingFilter characterEncodingFilter = new CharacterEncodingFilter();
        characterEncodingFilter.setEncoding("UTF-8");
        characterEncodingFilter.setForceResponseEncoding(true);
        HiddenHttpMethodFilter hiddenHttpMethodFilter = new HiddenHttpMethodFilter();
        return new Filter[]{characterEncodingFilter, hiddenHttpMethodFilter};
    }
}
```

## 创建SpringConfig配置类，代替Spring的配置文件

```java
@Configuration
public class SpringConfig {
    //ssm整合后，spring的配置信息写在此类中
}
```

## 创建WebConfig配置类，代替SpringMVC的配置文件

```java
@Configuration
@EnableWebMvc
@ComponentScan
public class WebConfig implements WebMvcConfigurer {
    // default-servlet-handler
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

    // 配置拦截器
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        TestInterceptor testInterceptor = new TestInterceptor();
        registry.addInterceptor(testInterceptor).addPathPatterns("/**").excludePathPatterns("/login");
    }

    // view-Controller
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/hello").setViewName("hello");
    }

    // 文件上传解析器
    @Bean
    public MultipartResolver multipartResolver(){
        CommonsMultipartResolver commonsMultipartResolver = new CommonsMultipartResolver();
        return commonsMultipartResolver;
    }

    // 异常处理
    @Override
    public void configureHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
        SimpleMappingExceptionResolver exceptionResolver = new SimpleMappingExceptionResolver();
        Properties prop = new Properties();
        prop.setProperty("java.lang.ArithmeticException", "error");
        exceptionResolver.setExceptionMappings(prop);
        exceptionResolver.setExceptionAttribute("exception");
        resolvers.add(exceptionResolver);
    }
}
```

拦截器类

```java
public class TestInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        HandlerInterceptor.super.postHandle(request, response, handler, modelAndView);
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        HandlerInterceptor.super.afterCompletion(request, response, handler, ex);
    }
}
```

# SpringMVC执行流程

## SpringMVC常用组件

* DispatcherServlet：前端控制器，不需要攻城狮开发，有框架提供

  作用：统一处理请求和响应，整个流程控制的中心，有它调用其他组件处理用户的请求

* HandlerMapping：处理器映射器，不需要攻城狮开发，有框架提供

  作用：根据url、method等信息查找Handler，控制器方法

* Handler：处理器，需要攻城狮开发

  作用：在DispatcherServlet‘的控制下Handler对具体的用户请求进行处理

* HandlerAdapter：处理器配置器，不需要攻城狮开发，有框架提供

  作用：通过HandlerAdapter对处理器（控制器方法）进行执行

* ViewResolver：视图解析器，不需要攻城狮开发，有框架提供

  作用：进行视图解析，得到响应的视图

* View：视图，不需要攻城狮开发，有框架或试图技术提供

  作用：将模型数据通过页面展示给用户

## DispathcerServlet初始化过程

DispatcherServlet本质上是一个Servlet，所以天然的遵循Servlet的生命周期。所以宏观上是Servlet生命周期来进行调度

**参考源码**

## DispatcherServlet调用组件处理请求

**参考源码**

## SpringMVC的执行流程

用户向服务器发送请求，请求被SpringMVC前端控制器DispatcherServlet捕获

Dispatcherl对请求URL进行解析，得到穹丘资源标识符（URI），判断请求URI对应的映射

**参考源码**

* 不存在
    * 在判断是否配置了mvc:default-servlet-handler
    * 如果没有配置，则控制台报映射查找不到，客户端展示404错误
    * 如果有配置，则访问目标资源（一般为静态资源），找不到客户端也会展示404错误

* 存在

    * 根据该URI，调用HandlerMapping获得该Handler配置的所有相关的对象（包括Handler对象以及Handler对象对应的拦截器），最后以HandlerExecutionChain执行链对象的形式返回
    * DispatcherServlet根据获取的Handler，选择一个合适的HandlerAdapter
    * 如果成功获取HandlerAdapter，此时将开始执行拦截器的preHandlerAdapter
    * 提取Request中的模型数据，填充Handler入参，开始执行Handler（Controller）方法，处理请求。在填充Handler的入参过程中，根据你的配置，Spring将帮你做一些额外的工作
        * HttpMessageConverter：将请求消息转换成一个对象，将对象转换成指定的相应信息
        * 数据转换：对请求消息进行数据转换
        * 数据格式化：对请求消息进行数据格式化
        * 数据验证：验证数据的有效性，验证结果存储到BindingResult或Error中
    * Handler执行完成后，向DispatcherServlet返回一个ModelAndView对象

    * 此时将开始执行拦截器的postHandle方法
    * 根据返回的ModelAndView选择一个合适的ViewResolver进行视图解析，根据Model和View来渲染视图
    * 渲染视图完毕执行后拦截器的afterCompletion’方法
    * 将渲染结果返回客户端







































































