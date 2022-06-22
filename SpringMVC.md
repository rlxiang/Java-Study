## SpringMVC

在父工程中导入依赖

```xml
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.13.2</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>5.3.8</version>
    </dependency>
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>servlet-api</artifactId>
        <version>2.5</version>
    </dependency>
    <dependency>
        <groupId>javax.servlet.jsp</groupId>
        <artifactId>jsp-api</artifactId>
        <version>2.2</version>
    </dependency>
    <!--支持el表达式标签-->
    <dependency>
        <groupId>javax.com.xrl.servlet</groupId>
        <artifactId>jstl</artifactId>
        <version>1.2</version>
    </dependency>
</dependencies>
```

### SpringMVC的特点

1. 轻量级，简单易学
2. 高效，基于请求响应的MVC框架
3. 与Spring兼容性好，无缝结合
4. 约定优于配置
5. 功能强大：RESTful、数据验证、格式化、本地化、主题等
6. 简洁灵活

### Hello SpringMVC

1. 新建一个Module，添加web支持

2. 确定到如SpringMVC的依赖！

3. 配置web.xml，注册DispatchServlet

   ```xml
    <servlet>
           <servlet-name>springmvc</servlet-name>
           <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
   
           <!--必须关联一个springmvc的配置文件：【spring-name】-servlet.xml-->
           <init-param>
               <param-name>contextConfigLocation</param-name>
               <param-value>classpath:springmvc-servlet.xml</param-value>
           </init-param>
           <!--启动级别为1，服务器启动就创建-->
           <load-on-startup>1</load-on-startup>
       </servlet>
   <!--
           SpringMVC中 / 与 /*的区别
           http://localhost:8080/a.jsp
         /: 只匹配所有请求，不会匹配jsp页面
         /*: 匹配所有请求，也匹配jsp页面
         如果配置的是/*,那么在视图解析器去拼接视图名时，会一直拼接.jsp，导致无法访问
       -->
       <servlet-mapping>
           <servlet-name>springmvc</servlet-name>
           <url-pattern>/</url-pattern>
       </servlet-mapping>
   ```

4. 编写SpringMVC的配置文件！ 名称：springmvc-servlet.xml: [servletname]-servlet.xml,这里的文件名字无所谓

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd">
   
   
   </beans>
   ```

5. 添加 处理器映射器

   ```xml
   // 处理器映射器有很多，这个是按照名字来找handler(controller)
   <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>
   ```

6. 添加 处理器适配器

   ```xml
   <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>
   ```

7. 添加视图解析器

   ```xml
   <bean id="InternalResourceViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
       <property name="prefix" value="/WEB-INF/jsp/"/>
       <property name="suffix" value=".jsp"/>
   </bean>
   ```

8. 编写操作业务的Controller，要么实现Controller接口，要么增加注解；需要返回一个ModelAndView，装数据，封视图

   ```java
   public class HelloController implements Controller {
       public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
           // 1. 模型与视图
           ModelAndView mv = new ModelAndView();
   
           //封装对象，放在ModelAndView的Model中
           mv.addObject("msg","Hello SpringMVC!");
           // 封装要跳转的视图，放在ModelAndView的View中
           mv.setViewName("hello"); // 因为配置了前缀和后缀，这里的全路径为/WEB-INF/jsp/hello.jsp
           return mv;
       }
   }
   ```

9. 将自己的类交给SpringIOC容器，注册bean

   ```java
   <bean id="/hello" class="com.xrl.controller.HelloController"/>
   ```

10. 最后需要跳转的页面，显示ModelAndView中的数据，同时配置tomcat

    ```jsp
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <html>
    <head>
        <title>Title</title>
    </head>
    <body>
    ${msg}
    </body>
    </html>
    ```

**可能会遇到的问题：访问出现404，排查步骤**

1. 查看控制台输出，看一下是不是缺少什么jar包
2. 如果jar包存在，实现无法输出，就在IDEA的项目发布中，添加lib依赖！这是因为在maven中的包没有上传到tomcat服务器中
3. ![image-20210709105500136](.\typora-user-images\image-20210709105500136.png)
4. 重启 Tomcat 即可解决！

![image-20210709105621484](.\typora-user-images\image-20210709105621484.png)



### SpringMVC的中心控制器

​	SpringMVC的web框架围绕DispatcherServlet设计的。DispatcherServlet的作用是将请求分发到不同的处理器(Controller)，**该框架以请求为驱动，围绕一个中心Servlet分派请求及提供其他功能。DispatcherServlet是一个实现的Servlet(它继承自HttpServlet基类)**

![image-20210709142613908](.\typora-user-images\image-20210709142613908.png)

Spring MVC的原理如下图所示:

​		当发起请求时被前置的控制器拦截到请求，根据请求参数生成代理请求，找到请求对应的实际控制器，控制器处理请求，创建数据模型，访问数据库，将模型响应给中心控制器，控制器使用模型与视图渲染视图结果，将结果返回给中心控制器，再将结果返回给请求者。

![image-20210709142724169](.\typora-user-images\image-20210709142724169.png)

### SpringMVC执行原理

![image-20210709142130139](.\typora-user-images\image-20210709142130139.png)

上图为SpringMVC的一个较完整的流程图，实线表示SpringMVC框架提供的技术，不需要开发者实现，虚线表示需要开发者实现。



**简要分析执行流程**

1. DispatcherServlet表示前置控制器，是整个SpringMVC的控制中心。用户发出请求，DispatcherServlet接收请求并拦截请求。
   - 我们假设请求的url为 : http://localhost:8080/SpringMVC/hello
   - 如上url拆分成三部分:
   -  http://localhost:8080服务器域名
   -  SpringMVC部署在服务器上的web站点
   -  hello表示控制器
   - 通过分析，如上url表示为:请求位于服务器localhost:8080上的SpringMVC站点的hello控制器。
2. HandlerMapping为处理器映射。由DispatcherServlet调用
3. HandlerMapping,HandlerMapping根据请求url查找Handler。
4. HandlerExecution表示具体的Handler,其主要作用是根据url查找控制器，如上url被查找控制器为: hello。
5. HandlerExecution将解析后的信息传递给DispatcherServlet,如解析控制器映射等。HandlerAdapter表示处理器适配器，其按照特定的规则去执行Handler。
6. Handler让具体的Controller执行。
7. Controller将具体的执行信息返回给HandlerAdapter,如ModelAndView。HandlerAdapter将视图逻辑名或模型传递给DispatcherServlet。
8. DispatcherServlet调用视图解析器(ViewResolver)来解析HandlerAdapter传递的逻辑视图名。
9. 视图解析器将解析的逻辑视图名传给DispatcherServlet。
10. DispatcherServlet根据视图解析器解析的视图结果，调用具体的视图。最终视图呈现给用户。



## SpringMVC注解开发

1. **建立项目**

2. **由于Maven可能存在资源过滤的问题，我们将其配置完善**

   ```xml
   <build>
       <resources>
           <resource>
               <directory>src/main/resources</directory>
               <includes>
                   <include>**/*.properties</include>
                   <include>**/*.xml</include>
               </includes>
               <filtering>true</filtering>
           </resource>
           <resource>
               <directory>src/main/java</directory>
               <includes>
                   <include>**/*.properties</include>
                   <include>**/*.xml</include>
               </includes>
               <filtering>true</filtering>
           </resource>
       </resources>
   </build>
   ```

3. **在pom.xml文件中引入相关的依赖**

   主要有Spring框架核心库、SpringMVC 、servlet、JSTL等，在父工程中已经引入了！

   ```xml
   <dependencies>
       <dependency>
           <groupId>junit</groupId>
           <artifactId>junit</artifactId>
           <version>4.13.2</version>
       </dependency>
       <dependency>
           <groupId>org.springframework</groupId>
           <artifactId>spring-webmvc</artifactId>
           <version>5.3.8</version>
       </dependency>
       <dependency>
           <groupId>javax.servlet</groupId>
           <artifactId>servlet-api</artifactId>
           <version>2.5</version>
       </dependency>
       <dependency>
           <groupId>javax.servlet.jsp</groupId>
           <artifactId>jsp-api</artifactId>
           <version>2.2</version>
       </dependency>
       <!--支持el表达式标签-->
       <dependency>
           <groupId>javax.servlet</groupId>
           <artifactId>jstl</artifactId>
           <version>1.2</version>
       </dependency>
   </dependencies>
   ```

4. **配置web.xml**

   注意点：

   - 注意web.xml版本问题，要最新版
   - 注册DispatcherServlet
   - 关联SpringMVC的配置文件
   - 配置DispatcherServlet的启动级别为1
   - 映射路径为/ 【不要用/*,否则会404】

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
            version="4.0">
       <servlet>
           <servlet-name>springmvc</servlet-name>
           <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
           <init-param>
               <param-name>contextConfigLocation</param-name>
               <param-value>classpath:springmvc-servlet.xml</param-value>
           </init-param>
           <load-on-startup>1</load-on-startup>
       </servlet>
       <servlet-mapping>
           <servlet-name>springmvc</servlet-name>
           <url-pattern>/</url-pattern>
       </servlet-mapping>
   </web-app>
   ```

5. **添加SpringMVC的配置文件**

   - 让IOC的注解生效

   - 静态资源过滤：HTML、JS、CSS、图片、视频……

   - MVC的注解驱动

   - 配置视图解析器

     在resource目录下添加springmvc-servlet.xml配置文件，配置的形式与Spring容器配置基本类似。为了支持基于注解的IOC，设置了自动扫描包的功能，具体配置信息如下;

     ```xml
     <?xml version="1.0" encoding="UTF-8"?>
     <beans xmlns="http://www.springframework.org/schema/beans"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:context="http://www.springframework.org/schema/context"
            xmlns:mvc="http://www.springframework.org/schema/mvc"
            xsi:schemaLocation="
             http://www.springframework.org/schema/beans
             http://www.springframework.org/schema/beans/spring-beans.xsd
             http://www.springframework.org/schema/context
             http://www.springframework.org/schema/context/spring-context.xsd
             http://www.springframework.org/schema/mvc
             http://www.springframework.org/schema/mvc/spring-mvc.xsd
     
     
     ">
         <!-- 自动包扫描，让指定包下的注解生效，由IOC容器统一管理-->
         <context:component-scan base-package="com.xrl.controller"/>
         <!-- 让SpringMVC不去处理静态资源-->
         <mvc:default-servlet-handler/>
         <!--
             支持mvc注解驱动
                 在spring中一版采用@RequestMapping注解来完成映射关系
                 要想使@RequestMapping注解生效
                 必须向上下文中注册DefaultAnnotationHandLerMapping
                 和一个AnnotationMethodHandLerAdopter实例
                 这两个实例分别在类级别和方法级别处理。
                 而annotation-driven配置帮助我们自动完成上述两个实例的注入。
         -->
         <mvc:annotation-driven/>
         <!-- 视图解析器 -->
         <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
             <property name="prefix" value="/WEB-INF/jsp/"/>
             <property name="suffix" value=".jsp"/>
         </bean>
     </beans>
     ```

     在视图解析器中，我们把所有的视图都存放在/WEB-INF/目录下，这样可以保证视图安全，因为这个目录下的文件，客户端不能直接访问

6. **创建Controller**

   编写一个java控制类： com.xrl.controller.HelloController.

   ```java
   import org.springframework.stereotype.Controller;
   import org.springframework.ui.Model;
   import org.springframework.web.bind.annotation.RequestMapping;
   
   @Controller
   @RequestMapping("/hello")
   public class HelloController {
   
       @RequestMapping("/h1")
       public String hello(Model model){
           // 封装数据
           model.addAttribute("msg","Hello SpringMVCAnnotation!");
           return "hello"; // 返回值会被视图解析器处理
       }
   }
   ```

   - @Controller是为了让Spring lOC容器初始化时自动扫描到;
   - @RequestMapping是为了映射请求路径，这里因为类与方法上都有映射所以访问时应该是/hello/h1;
   - 方法中声明Model类型的参数是为了把Action中的数据带到视图中;
   - 方法返回的结果是视图的名称hello，加上配置文得中的前后缀变成WEB-INF/jsp/hello.jsp。

7. **创建视图层**

   ```jsp
   <%@ page contentType="text/html;charset=UTF-8" language="java" %>
   <html>
   <head>
       <title>Title</title>
   </head>
   <body>
   ${msg}
   </body>
   </html>
   ```

8. **配置Tomcat运行**

   配置Tomcat，开启服务器，访问对应的请求路径！

![image-20210709163014817](.\typora-user-images\image-20210709163014817.png)



**总结**

使用springMVC必须配置的三大件:

​	**处理器映射器、处理器适配器、视图解析器**

通常，我们只需要**手动配置视图解析器**，而**处理器映射器**和**处理器适配器**只需要开启**注解驱动**即可，而省去了大真的xml配置





## Controller 及 RestFul风格

### 控制器Controller

- 控制器复杂提供访问应用程序的行为，通常通过接口定义或注解定义两种方法实现

- 控制器负责解析用户的请求并将其转换为一个模型。

- 在Spring MVC中一个控制器类可以包含多个方法

- 在Spring MVC中，对于Controller的配置方式有很多种

  - 实现Controller接口

  - Controller是一个接口，在org.springframework.web.servlet.mvc包下，接口中只有一个方法

    ```java
    // 实现该接口的类就可以获取控制器的功能
    public interface Controller {
       // 处理请求且返回一个模型与视图对象
       ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception;
    
    }
    ```

  - 编写一个Controller类，ControllerTest

    ```java
    // 只要实现了Controller接口的类，说明这就是一个控制器
    public class ControllerTest1 implements Controller {
        public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
            ModelAndView mv = new ModelAndView();
            mv.addObject("msg","ControllerTest1");
            mv.setViewName("test");
            return mv;
        }
    }
    ```

  - 编写完毕后，去Spring配置文件中注册请求的bean；name对应请求路径，class对应处理请求的类

    ```xml
    <bean id="/t1" class="com.xrl.controller.ControllerTest1"/>
    ```

  - 编写前端jsp页面，注意在WEB-INF/jsp目录下编写，对应我们的视图解析器

    ```jsp
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <html>
    <head>
        <title>Title</title>
    </head>
    <body>
    ${msg}
    </body>
    </html>
    ```

  - 测试

    ![image-20210709171247855](.\typora-user-images\image-20210709171247855.png)

    说明：

    - 实现接口Controller定义控制器是较老的办法
    - 缺点是:一个控制器中只有一个方法，如果要多个方法则需要定义多个Controller;定义的方式比较麻烦;





### 使用注解@Controller

- @Controller注解类型用于声明Spring类的实例是一个控制器（在讲IOC时还提到了另外3个注解);

- Spring可以使用扫描机制来找到应用程序中所有基于注解的控制器类，为了保证Spring能找到你的控制器，需要在配置文件中声明组件扫描。

  ```xml
  <mvc:annotation-driven/>
  ```

- 增加一个ControllerTest2类，使用注解实现

  ```java
  @Controller
  public class ControllerTest2 {
  
      @RequestMapping("/t2")
      public String test1(Model model){
          model.addAttribute("msg","ControllerTest2");
          return "test";
      }
  }
  ```

- 测试

  ![image-20210709172338246](.\typora-user-images\image-20210709172338246.png)

### RestFul风格

### 概念

Restful就是一个资源定位及资源操作的风格。不是标准也不是协议，只是一种风格。基于这个风格设计的软件可以更简洁,更有层次，更易于实现缓存等机制。



### 功能

- 资源:互联网所有的事物都可以被抽象为资源

- 资源操作:使用POST、DELETE、PUT、GET，使用不同方法对资源进行操作。

- 分别对应添加、删除、修改、查询。

  **传统方式操作资源**︰通过不同的参数来实现不同的效果!方法单一，post和get

- http://127.0.0.1/item/queryltem.action?id=1查询,GET

- http://127.0.0.1/item/saveltem.action新增,POST

- http://127.0.0.1/item/updateltem.action更新,POST

- http://127.0.0.1/item/deleteltem.action?id=1删除,GET或POST

  **使用RESTful操作资源︰**可以通过不同的请求方式来实现不同的效果!如下:请求地址一样，但是功能可以不同!

- http:/ /127.0.0.1/item/1查询,GET

- http:/ /127.0.0.1/item新增,POST

- http:/ /127.0.0.1/item更新,PUT

- http:/ /127.0.0.1/item/1删除,DELETE



**学习测试**

```java
@Controller
public class RestFulController {

    @RequestMapping("/add/{a}/{b}")
    public String test1(@PathVariable int a, @PathVariable int b, Model model){
        int res = a + b;
        model.addAttribute("msg","结果为"+res);
        return "test";
    }
}
```

**在Spring MVC中可以使用@PathVariable注解，让方法参数的值对应绑定到一个URI模板变量上。**



使用RestFul风格的好处

- 使路径变得更加简结；
- 获得参数更加方便，框架会自动进行类型转换
- 通过路径变量的类型可以约束访问参数，如果类型不一样，则访问不到对应的请求方法。

## ModelAndView

设置ModelAndView对象，根据view的名称，和视图解析器跳到指定的页面.

页面∶{视图解析器前缀}+ viewName +{视图解析器后缀}

```xml
<!-- 视图解析器 -->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/jsp/"/>
    <property name="suffix" value=".jsp"/>
</bean>
```

对应的controller类

```java
// 这种实现Controller是最老的方式
public class HelloController implements Controller {
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
        ModelAndView mv = new ModelAndView();
        mv.addObject("msg","Hello SpringMVC");
        mv.setViewName("test");
        return mv;
    }
}
```

### 通过ServletAPI进行模型与视图跳转

通过设置ServletAPI，不需要视图解析器

1. 通过HttpServletResponse进行输出

2. 通过HttpServletResponse实现重定向

3. 通过HttpServletRequest实现转发

   ```java
   @controller
   public class ResultGo {
       @RequestMapping( "/result/t1")
       public void test1(HttpServletRequest req，HttpServletResponse rsp) throws IOException{
         rsp.getwriter().println( "Hello, spring BY servlet API");  
       }
       @RequestMapping( "/result/t2")
       public void test2(HttpServletRequest req，HttpServletResponse rsp) throws IOException{
           rsp.sendRedirect( "/index.jsp" );
       }
        @RequestMapping( "/result/t3")
       public void test3(HttpServletRequest req，HttpservletResponse rsp) throws Exception{
         //转发 
          req.setAttribute( "msg " , "/result/t3");
          req.getRequestDispatcher("/WEB-INF/jsp/test.jsp" ).forward(req,rsp);
       }
   }
   ```



### SpringMVC实现转发和重定向 - 无需视图解析器

测试前，先将视图解析器注释掉

```java
@Controller
public class ModelTest1 {

    // 在没有视图解析器的情况下，无论是转发还是重定向都需要写下完整的路径名，其中转发时，forward前缀可以省略

    @RequestMapping("/m1/t1")
    public String test(Model model){
        // 转发
        model.addAttribute("msg","ModelTest1");
        return "forward:/WEB-INF/jsp/test.jsp";  // 只要没有加redirect前缀，都是转发

    }

    @RequestMapping("/m1/t2")
    public String test2(Model model){
        // 转发
        model.addAttribute("msg","ModelTest1,重定向");
        return "redirect:/index.jsp";  // 只要没有加redirect前缀，都是转发

    }
}
```

### SpringMVC实现转发和重定向 - 使用视图解析器

重定向，不需要视图解析器，本质就是重新请求一个新地方嘛，所以注意路径问题.

可以重定向到另外一个请求实现。

```java
@Controller
public class ModelTest2 {

// 有视图解析器
    @RequestMapping("/m1/t3")
    public String test(Model model){
        // 转发
        model.addAttribute("msg","ModelTest1");
        return "test";

    }

    @RequestMapping("/m1/t4")
    public String test2(Model model){
        // 重定向
        model.addAttribute("msg","ModelTest1,重定向");
        return "redirect:/index.jsp";
//        return "redirect:/m1/t3";// 再此请求/m1/t3
    }
}
```

注意：

==重定向是一次请求，和我们在地址栏上输入的请求是一个概念，所以在重定向是不可以直接重定向到客户端无法访问的页面，比如 WEB-INF下的页面，templates下的页面==



## 接受请求参数与数据回显

### 提交的域名称(请求地址的参数)和处理方法的参数名一致时

提交数据：http://localhost:8080/hello?name=xrl

处理方法

```java
// localhost:8080/user/t1?name=xxx
@GetMapping("/t1")
public String test1(String name, Model model){
    // 接受前端参数
    System.out.println("接受到前端的参数了");
    // 将返回的结果返回给前端
    model.addAttribute("msg",name);
    // 跳转视图
    return "test";
}
```

结果

![image-20210709222753242](.\typora-user-images\image-20210709222753242.png)

### 提交的域名称(请求地址的参数)和处理方法的参数名不一致时

处理方法

```java
// localhost:8080/user/t1?username=xxx
@GetMapping("/t2")
public String test2(@RequestParam("username") String name, Model model){
    // 接受前端参数
    System.out.println("接受到前端的参数了");
    // 将返回的结果返回给前端
    model.addAttribute("msg","接受到前端的参数"+name);
    // 跳转视图
    return "test";
}
```

结果：

![image-20210709223015971](.\typora-user-images\image-20210709223015971.png)



### 前端提交的是一个对象

处理方法

```java
// 前端提交的是一个对象，对象中有 id,name,age
@GetMapping("/t3")
public String test3(User user, Model model){
    // 接受前端参数
    System.out.println("接受到前端的参数了");
    // 将返回的结果返回给前端
    model.addAttribute("msg","接受到前端的参数"+user.toString());
    // 跳转视图
    return "test";
}
```

结果

![image-20210709223459780](.\typora-user-images\image-20210709223459780.png)

注意：

在传递参数的接收是一个对象的时候(user)，会自动匹配参数名字是否和对象的属性名字一致，一致则ok，否则404

### 数据显示到前端

1. 通过ModelAndView

2. 通过Model

3. 通过ModelMap

   ```java
   // 前端提交的是一个对象，对象中有 id,name,age
   @GetMapping("/t4")
   public String test4(User user, ModelMap model){
       // 接受前端参数
       System.out.println("接受到前端的参数了");
       // 将返回的结果返回给前端
       model.addAttribute("msg","接受到前端的参数"+user.toString());
       // 跳转视图
       return "test";
   }
   ```

   **对比:**

   - Model只有寥寥几个方法只适合用于储存数器，简化了新手对于Model对象的操作和理解:
   - ModelMap继承了LinkedMap ，除了实现了自身的一些方法，同样的继承LinkedMap的方法和特性;
   - ModelAndview可以在储存数据的同时，可以进行没置返回的逻辑视图，进行控制展示层的跳转，

## 乱码问题

测试步骤

1. 编写jsp页面

   ```jsp
   <form action="/e/t1" method="get">
       <input type="text" name="name">
       <input type="submit">
   </form>
   ```

2. 编写后台对应的处理类

   ```java
   @PostMapping("/e/t1")
   public String test1(String name, Model model){
       model.addAttribute("msg",name);
       return "test";
   }
   ```

3. 输入中文测试，发现乱码

   ![image-20210712214930700](.\typora-user-images\image-20210712214930700.png)

   以前乱码的问题我们通过过滤器解决

   ```java
   import javax.servlet.*;
   import java.io.IOException;
   
   public class EncodingFilter implements Filter {
       public void init(FilterConfig filterConfig) throws ServletException {
   
       }
   
       public void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain) throws IOException, ServletException {
           request.setCharacterEncoding("utf-8");
          response.setCharacterEncoding("utf-8");
          filterChain.doFilter(request,response);
       }
   
       public void destroy() {
   
       }
   }
   ```

   然后在web.xml配置过滤器

   ```xml
   <filter>
       <filter-name>encoding</filter-name>
       <filter-class>com.xrl.filter.EncodingFilter</filter-class>
   </filter>
   <filter-mapping>
       <filter-name>encoding</filter-name>
       <url-pattern>/*</url-pattern>
   </filter-mapping>
   ```

   

   而SpringMVC给我们提供了一个过滤器，可以在web.xml中配置

   ```xml
   <!--配置SpringMVC的乱码过滤-->
   <filter>
       <filter-name>encoding</filter-name>
       <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
       <init-param>
           <param-name>encoding</param-name>
           <param-value>utf-8</param-value>
       </init-param>
   </filter>
   <filter-mapping>
       <!-- /*表示可以过滤所有请求，包括在jsp里提交的请求 -->
       <filter-name>encoding</filter-name>
       <url-pattern>/*</url-pattern>
   </filter-mapping>
   ```

   然而，有时候这个过滤器对get方式提交的数据不太友好

   处理方法

   ```xml
   <Connector port="8080" protocol="HTTP/1.1"
              connectionTimeout="20000"
              redirectPort="8443" 
              URIEncoding="UTF-8"/>
   ```

## JSON

### 什么是JSON

- JSON(JavaScript Object Notation, JS对象标记)是一种轻量级的数据交换格式，目前使用特别广泛。
- 采用完全独立于编程语言的文本格式来存储和表示数据。
- 简洁和清晰的层次结构使得JSON成为理想的数据交换语言。
- 易于人阅读和编写，同时也易于机器解析和生成，并有效地提升网络传输效率。

在JavaScript语言中，一切都是对象。因此，任何JavaScript支持的类型都可以通过JSON来表示。例如字符串、数字、对象、数组等。看看他的要求和语法格式:

- 对象表示为键值对,数据由逗号分隔
- 花括号保存对象
- 方括号保存数组

**JSON 键值对**是用来保存JavaScript对象的一种方式，和JavaScript对象的写法也大同小异,键/值对组合中的键名写在前面并用双引号""包裹，使用冒号∶分隔，然后紧接着值:

```json
{"name": "小红"}
{"age": "3"}
{"sex": "男"}
```

很多人搞不清楚JSON和JavaScript对象的关系，甚至连谁是谁都不清楚。其实，可以这么理解:

- **JSON**是**JavaScript**对象的字符串表示法，它使用文本表示一个JS对象的信息，本质是一个字符串。

  ```javascript
  var obj = {a: 'Hello ', b: 'world ' }; //这是一个对象，注意键名也是可以使用引号包裹的
  var json = ' { "a ": "Hello","b":"world" }';//这是一个Json字符串，本质是一个字符串
  ```

### **JSON与JavaScript 对象互转**

- 要实现从JSON字符串转换为JavaScript对象，使用JSON.parse()方法:

  ```javascript
  var obj = JSON.parse('{"a": “Hello","b": "world"}');
  //结果是{a:" Hello', b: "world"}
  ```

- 要实现从JavaScript对象转换为JSON字符串，使用JSON.stringify()方法:

  ```javascript
  var json = JSON.stringify({a: "hello", b:"World"});
  // 结果是 '{"a": “Hello","b": "world"}'
  ```

- 代码测试

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
      <script>
          var user={
              name: "张三",
              age: 18,
              sex: "男"
          };
  
          // 将js对象转换为json对象
          var json = JSON.stringify(user);
          console.log(json);
  
  
          console.log("=========");
  
          // 将JSON对象转换为js对象
          var obj = JSON.parse(json);
          console.log(obj);
      </script>
  </head>
  <body>
  
  </body>
  </html>
  ```

## Controller返回JSON数据

### Jackson

- Jackson应该是目前比较好的json解析工具了

- 当然工具不止这一个，比如还有阿里巴巴的 fastjson等等。

- 我们这里使用Jackson，使用它需要导入它的jar包:

  ```xml
  <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>2.12.3</version>
  </dependency>
  ```

- 配置SpringMVC需要的配置

  web.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
           version="4.0">
      <servlet>
          <servlet-name>springmvc</servlet-name>
          <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
          <init-param>
              <param-name>contextConfigLocation</param-name>
              <param-value>classpath:springmvc-servlet.xml</param-value>
          </init-param>
          <load-on-startup>1</load-on-startup>
      </servlet>
      <servlet-mapping>
          <servlet-name>springmvc</servlet-name>
          <url-pattern>/</url-pattern>
      </servlet-mapping>
  
      <filter>
          <filter-name>encoding</filter-name>
          <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
          <init-param>
              <param-name>encoding</param-name>
              <param-value>utf-8</param-value>
          </init-param>
      </filter>
      <filter-mapping>
          <filter-name>encoding</filter-name>
          <url-pattern>/*</url-pattern>
      </filter-mapping>
  </web-app>
  ```

- springmvc-servlet.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:context="http://www.springframework.org/schema/context"
         xmlns:mvc="http://www.springframework.org/schema/mvc"
         xsi:schemaLocation="
                             http://www.springframework.org/schema/beans
                             http://www.springframework.org/schema/beans/spring-beans.xsd
                             http://www.springframework.org/schema/context
                             http://www.springframework.org/schema/context/spring-context.xsd
                             http://www.springframework.org/schema/mvc
                             http://www.springframework.org/schema/mvc/spring-mvc.xsd
  
  
                             ">
      <!-- 自动包扫描，让指定包下的注解生效，由IOC容器统一管理-->
      <context:component-scan base-package="com.xrl.controller"/>
      <!-- 让SpringMVC不去处理静态资源-->
      <mvc:default-servlet-handler/>
      <!--
          支持mvc注解驱动
              在spring中一版采用@RequestMapping注解来完成映射关系
              要想使@RequestMapping注解生效
              必须向上下文中注册DefaultAnnotationHandLerMapping
              和一个AnnotationMethodHandLerAdopter实例
              这两个实例分别在类级别和方法级别处理。
              而annotation-driven配置帮助我们自动完成上述两个实例的注入。
      -->
      <mvc:annotation-driven/>
      <!-- 视图解析器 -->
      <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
          <property name="prefix" value="/WEB-INF/jsp/"/>
          <property name="suffix" value=".jsp"/>
      </bean>
  </beans>
  ```

- 随便写个User的实体类，然后测试Controller

  ```java
  package com.xrl.pojo;
  
  import lombok.AllArgsConstructor;
  import lombok.Data;
  import lombok.NoArgsConstructor;
  
  @Data
  @AllArgsConstructor
  @NoArgsConstructor
  public class User {
      private String name;
      private int age;
      private String sex;
  }
  ```

- 访问 http://localhost:8080/j1 

- 发现出现了乱码，我们需要设置一下编码格式为utf-8，以及它返回的类型；

- 通过@RequestMapping的produces属性来实现，修改代码

  ```java
   @RequestMapping(value = "/j1",produces = "application/json;charset=utf-8")
  ```

- 再次测试，访问 http://localhost:8080/j1 ，乱码问题解决

  ==【注意：使用json记得处理乱码问题】==

### **代码优化**

**乱码问题统一解决**

​	上面的解决乱码的方法比较麻烦，如果项目中有许多次请求，则每一个都要添加，可以通过Spring配置统一指定，这样就不用每次都去处理了！

​	我们可以在springmvc的配置文件上添加一段消息StringHttpMessageConverter转换配置！

```xml
<mvc:annotation-driven>
    <!--处理JSON乱码问题-->
    <mvc:message-converters register-defaults="true">
        <bean class="org.springframework.http.converter.StringHttpMessageConverter">
            <constructor-arg value="UTF-8"/>
        </bean>
        <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
            <property name="objectMapper">
                <bean class="org.springframework.http.converter.json.Jackson2ObjectMapperFactoryBean">
                    <property name="failOnEmptyBeans" value="false"/>
                </bean>
            </property>
        </bean>
    </mvc:message-converters>
</mvc:annotation-driven>
```

```java
// 此时的测试代码
@ResponseBody
@RequestMapping("/j1")
public String json1() throws JsonProcessingException {

    // jackson包里有个ObjectMapper类
    ObjectMapper objectMapper = new ObjectMapper();

    // 创建一个对象
    User user = new User("小红",18,"女");
    String str = objectMapper.writeValueAsString(user);
    return str;
} // 前端输出 {"name":"小红","age":18,"sex":"女"}
```

**测试集合输出**

```java
@RequestMapping("/j2")
public String json2() throws JsonProcessingException {

    // jackson包里有个ObjectMapper类
    ObjectMapper objectMapper = new ObjectMapper();

    // 创建一个对象
    User user1 = new User("小红1号",18,"女");
    User user2 = new User("小红2号",18,"女");
    User user3 = new User("小红3号",18,"女");
    User user4 = new User("小红4号",18,"女");
    List<User> userList = new ArrayList<User>();
    userList.add(user1);
    userList.add(user2);
    userList.add(user3);
    userList.add(user4);
    return  objectMapper.writeValueAsString(userList);
}
```

**输出时间对象**

```java
@RequestMapping("/j3")
public String json3() throws JsonProcessingException {

    // jackson包里有个ObjectMapper类
    ObjectMapper objectMapper = new ObjectMapper();

    Date date = new Date();

    // ObjectMapper,时间解析后的默认格式为：Timestamp，时间戳(1970年到此刻的时间的毫秒数)
    return  objectMapper.writeValueAsString(date);
}
```

运行结果：

- 默认日期格式会变成一个数组，是1970年1月1日到当前日期的毫秒数

- Jackson默认是会把时间转成timestamps形式

  解决方案：取消timestamps形式，自定义时间格式

  ```java
  @RequestMapping("/j3")
  public String json3() throws JsonProcessingException {
  
      // jackson包里有个ObjectMapper类
      ObjectMapper objectMapper = new ObjectMapper();
  
      Date date = new Date();
      // 自定义日期格式
      
      SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
  
      return  objectMapper.writeValueAsString(sdf.format(date));
  }
  ```

  直接是使用ObjectMapper来解决

  ```java
  @RequestMapping("/j3")
  public String json3() throws JsonProcessingException {
  
      // jackson包里有个ObjectMapper类
      ObjectMapper objectMapper = new ObjectMapper();
  
      // 使用ObjectMapper来格式化输出时间
      // 不使用时间戳的方式
  objectMapper.configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS,false);
      Date date = new Date();
      // 自定义日期格式
      SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
      objectMapper.setDateFormat(sdf);
  
      return  objectMapper.writeValueAsString(date);
  }
  ```

  将ObjectMapper封装成工具类

  ```java
  package com.xrl.utils;
  
  import com.fasterxml.jackson.core.JsonProcessingException;
  import com.fasterxml.jackson.databind.ObjectMapper;
  
  import java.text.SimpleDateFormat;
  
  public class JsonUtils {
  
      public static String getJson(Object object){
          return getJson(object,"yyyy-MM-dd HH:mm:ss");
      }
  
      public static String getJson(Object object, String dateFormat){
          ObjectMapper mapper = new ObjectMapper();
          SimpleDateFormat sdf = new SimpleDateFormat(dateFormat);
          mapper.setDateFormat(sdf);
          String str = null;
          try {
              str = mapper.writeValueAsString(object);
          } catch (JsonProcessingException e) {
              e.printStackTrace();
          }
          return str;
      }
  }
  
  ```



### FastJson

​	fastjson.jar是阿里开发的一款专门用于Java开发的包，可以方便的实现json对象与JavaBean对象的转换，实现JavaBean对象与json字符串的转换，实现json对象与json字符串的转换。实现json的转换方法很多，最后的实现结果都是一样的。



fastjson 的pom依赖!

```xml
<!-- https://mvnrepository.com/artifact/com.alibaba/fastjson -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.76</version>
</dependency>
```

fastjson三个主要的类:

-  [JSONObject 代表json对象】
  - JSONObject实现了Map接口,猜想JSONObject底层操作是由Map实现的。
  - JSONObject对应json对象，通过各种形式的get()方法可以获取json对象中的数据，也可利用诸如size()， isEmpty()等方法获取"键:值"对的个数和判断是否为空。其本质是通过实现Map接口并调用接口中的方法完成的。
- 【JSONArray 代表json对象数组】
  - 内部是有List接口中的方法来完成操作的。
- 【JSON代表JSONObject和JSONArray的转化】
  - JSON类源码分析与使用
  - 仔细观察这些方法，主要是实现json对象，json对象数组，javabean对象，json字符串之间的相互转化。

```java
@RequestMapping("/j4")
public String json4(){

    User user1 = new User("小红1号",18,"女");
    User user2 = new User("小红2号",18,"女");
    User user3 = new User("小红3号",18,"女");
    User user4 = new User("小红4号",18,"女");
    List<User> userList = new ArrayList<User>();
    userList.add(user1);
    userList.add(user2);
    userList.add(user3);
    userList.add(user4);
    return JSON.toJSONString(userList);
}
```

## SSM整合(ssmbuild)

### MyBatis层	

```sql
CREATE DATABASE ssmbuild;

USE ssmbuild;

CREATE TABLE books(
	bookID INT(10) PRIMARY KEY NULL AUTO_INCREMENT COMMENT '书id',
	bookName VARCHAR(100) NOT NULL COMMENT '书名',
	bookCounts INT(11) NOT NULL COMMENT '数量',
	detail VARCHAR(200) NOT NULL COMMENT '描述'	
	
)ENGINE=INNODB DEFAULT CHARSET=utf8;

INSERT INTO books VALUES(1,'Java',1,'从入门到放弃'),(2,'MySQL',10,'从删库到跑路'),
(3,'Linux',5,'从进门到放弃');
```

- 基本环境搭建

  1. 新建一个Maven项目！ssmbuild，添加web支持

  2. 导入相关的pom依赖！

     ```xml
     <dependencies>
         <dependency>
             <groupId>junit</groupId>
             <artifactId>junit</artifactId>
             <version>4.13.2</version>
         </dependency>
         <dependency>
             <groupId>org.projectlombok</groupId>
             <artifactId>lombok</artifactId>
             <version>1.18.20</version>
         </dependency>
         <!--数据库驱动-->
         <dependency>
             <groupId>mysql</groupId>
             <artifactId>mysql-connector-java</artifactId>
             <version>8.0.25</version>
         </dependency>
         <!--连接池-->
          <dependency>
                 <groupId>com.mchange</groupId>
                 <artifactId>c3p0</artifactId>
                 <version>0.9.5.5</version>
             </dependency>
         <!--servlet jsp-->
         <dependency>
             <groupId>javax.servlet</groupId>
             <artifactId>servlet-api</artifactId>
             <version>2.5</version>
         </dependency>
         <dependency>
             <groupId>javax.servlet</groupId>
             <artifactId>jstl</artifactId>
             <version>1.2</version>
         </dependency>
         <dependency>
             <groupId>javax.servlet.jsp</groupId>
             <artifactId>jsp-api</artifactId>
             <version>2.2</version>
         </dependency>
     
         <dependency>
             <groupId>org.mybatis</groupId>
             <artifactId>mybatis-spring</artifactId>
             <version>2.0.6</version>
         </dependency>
         <dependency>
             <groupId>org.mybatis</groupId>
             <artifactId>mybatis</artifactId>
             <version>3.5.7</version>
         </dependency>
     
         <!--Spring的-->
         <dependency>
             <groupId>org.springframework</groupId>
             <artifactId>spring-webmvc</artifactId>
             <version>5.3.8</version>
         </dependency>
         <dependency>
             <groupId>org.springframework</groupId>
             <artifactId>spring-jdbc</artifactId>
             <version>5.3.8</version>
         </dependency>
     </dependencies>
     ```

  3. 静态资源导出问题

     ```xml
     <build>
         <resources>
             <resource>
                 <directory>src/main/resources</directory>
                 <includes>
                     <include>**/*.properties</include>
                     <include>**/*.xml</include>
                 </includes>
                 <filtering>true</filtering>
             </resource>
             <resource>
                 <directory>src/main/java</directory>
                 <includes>
                     <include>**/*.properties</include>
                     <include>**/*.xml</include>
                 </includes>
                 <filtering>true</filtering>
             </resource>
         </resources>
     </build>
     ```

  4. 配置相应的mybatis-config.xml与applicationContext.xml文件

     mybatis-config.xml

     ```xml
     <?xml version="1.0" encoding="UTF-8" ?>
     <!DOCTYPE configuration
             PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
             "http://mybatis.org/dtd/mybatis-3-config.dtd">
     <configuration>
     
         <!--配置数据源，交给数据源-->
  <typeAliases>
         <package name="com.xrl.pojo"/>
  </typeAliases>
     <mappers>
         <mapper class="com.xrl.dao.BookMapper"/>
     </mappers>
  </configuration>
     ```
     
     applicationContext.xml
     
     ```xml
     <?xml version="1.0" encoding="UTF-8"?>
     <beans xmlns="http://www.springframework.org/schema/beans"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://www.springframework.org/schema/beans
             https://www.springframework.org/schema/beans/spring-beans.xsd">
     
     
     </beans>
     ```
     
  5. 数据库连接文件

     ```properties
     jdbc.driver=com.mysql.cj.jdbc.Driver
     jdbc.url=jdbc:mysql://localhost:3306/ssmbuild?serverTimezone=UTC&useSSL=true&useUnicode=true&characterEncoding=utf8
     jdbc.username=root
     jdbc.password=root
     ```

     

  

### Spring层

- **Spring dao(mapper)层**

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:context="http://www.springframework.org/schema/context"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
          https://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/context
          https://www.springframework.org/schema/context/spring-context.xsd
  ">
  
      <!--1. 关联数据库文件-->
      <context:property-placeholder location="classpath:database.properties"/>
  
  
      <!--2.连接池-->
      <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
          <property name="driverClass" value="${jdbc.driver}"/>
          <property name="jdbcUrl" value="${jdbc.url}"/>
          <property name="user" value="${jdbc.username}"/>
          <property name="password" value="${jdbc.password}"/>
  
          <!--C3p0的私有属性-->
          <property name="maxPoolSize" value="30"/>
          <property name="minPoolSize" value="10"/>
          <!-- 关闭连接后不自动提交commit-->
          <property name="autoCommitOnClose" value="false"/>
          <!--获取连接超时时间-->
          <property name="checkoutTimeout" value="10000"/>
          <!--当获取连接失败 重新获取的次数-->
          <property name="acquireRetryAttempts" value="2"/>
      </bean>
  
      <!--3.sqlSessionFactory-->
      <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
          <!-- 绑定数据源-->
          <property name="dataSource" ref="dataSource"/>
          <!-- 绑定Mybatis配置文件-->
          <property name="configLocation" value="classpath:mybatis-config.xml"/>
      </bean>
  
      <!--配置dao(mapper)接口扫描包，动态的实现了Dao(Mapper)接口，可以直接注入到Spring容器中-->
      <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
          <!--注入SqlSessionFactory-->
          <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
          <!--要扫描dao(mapper)包-->
          <property name="basePackage" value="com.xrl.dao"/>
      </bean>
  </beans>
  ```

- **Spring service 层**

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:context="http://www.springframework.org/schema/context"
         xmlns:aop="http://www.springframework.org/schema/aop"
         xmlns:tx="http://www.springframework.org/schema/tx"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
          https://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/context
          https://www.springframework.org/schema/context/spring-context.xsd
          http://www.springframework.org/schema/aop
          https://www.springframework.org/schema/aop/spring-aop.xsd
          http://www.springframework.org/schema/tx
          http://www.springframework.org/schema/tx/spring-tx.xsd">
  
      <!--1.扫描service下的包-->
      <context:component-scan base-package="com.xrl.service"/>
  
      <!--2. 将所有的业务类，注入到Spring中，可以通过配置，或者注解实现-->
      <bean id="bookServiceImpl" class="com.xrl.service.BookServiceImpl">
          <property name="mapper" ref="bookMapper"/>
      </bean>
  
      <!--3. 声明式事务 -->
      <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
          <!--注入数据源-->
          <property name="dataSource" ref="dataSource"/>
      </bean>
  
      <!--结合AOP实现事务的织入-->
      <!--配置事务通知-->
      <tx:advice id="txAdvice" transaction-manager="transactionManager">
          <!--给那些方法配置事务-->
          <!--配置事务的传播特性 propagation-->
          <tx:attributes>
              <tx:method name="*" propagation="REQUIRED"/>
          </tx:attributes>
      </tx:advice>
  
      <!--配置事务的切入-->
      <aop:config>
          <aop:pointcut id="txPointCut" expression="execution(* com.xrl.dao.*.*(..))"/>
          <aop:advisor advice-ref="txAdvice" pointcut-ref="txPointCut"/>
      </aop:config>
  </beans>
  ```

  ### Spring -MVC层

  **spring-mvc.xml**

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:mvc="http://www.springframework.org/schema/mvc"
         xmlns:context="http://www.springframework.org/schema/context"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
          https://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/mvc
           http://www.springframework.org/schema/mvc/spring-mvc.xsd
           http://www.springframework.org/schema/context
            https://www.springframework.org/schema/context/spring-context.xsd
  ">
  
      <!--1. 注解驱动 -->
      <mvc:annotation-driven/>
      <!--2. 静态资源过滤 -->
      <mvc:default-servlet-handler/>
      <!--3. 包扫描 -->
      <context:component-scan base-package="com.xrl.controller"/>
      <!--4. 视图解析器-->
      <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
          <property name="suffix" value=".jsp"/>
          <property name="prefix" value="/WEB-INF/jsp/"/>
      </bean>
  </beans>
  ```

  **web.xml**

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
           version="4.0">
      
      <!--DispatcherServlet-->
      <servlet>
          <servlet-name>springmvc</servlet-name>
          <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
          <init-param>
              <param-name>contextConfigLocation</param-name>
              <param-value>classpath:applicationContext.xml</param-value>
          </init-param>
          <load-on-startup>1</load-on-startup>
      </servlet>
      <servlet-mapping>
          <servlet-name>springmvc</servlet-name>
          <url-pattern>/</url-pattern>
      </servlet-mapping>
      // 乱码过滤
      <filter>
          <filter-name>encoding</filter-name>
          <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
          <init-param>
              <param-name>encoding</param-name>
              <param-value>utf-8</param-value>
          </init-param>
      </filter>
      <filter-mapping>
          <filter-name>encoding</filter-name>
          <url-pattern>/*</url-pattern>
      </filter-mapping>
  
      <!--session过期时间-->
      <session-config>
          <session-timeout>15</session-timeout>
      </session-config>
  </web-app>
  ```

- 最后的applicationContext.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
          https://www.springframework.org/schema/beans/spring-beans.xsd">
  
      <import resource="classpath:spring-dao.xml"/>
      <import resource="classpath:spring-service.xml"/>
      <import resource="classpath:spring-mvc.xml"/>
  
  </beans>
  ```

### Spring MVC: Ajax技术

### 简介

- AJAX = Asynchronous JavaScript and XML(异步的JavaScript和XML) .

- AJAX是一种在无需重新加载整个网页的情况下，能够更新部分网页的技术。

- Ajax 不是一种新的编程语言，而是一种用于创建更好更快以及交互性更强的Web应用程序的技术。

- 在2005年，Google通过其Google Suggest使AJAX变得流行起来。Google Suggest能够自动帮你完成搜索单词。

- Google Suggest使用AJAX创造出动态性极强的web界面:当您在谷歌的搜索框输入关键字时，JavaScript会把这些字符发送到服务器，然后服务器会返回一个搜索建议的列表。

- 就和国内百度的搜索框一样

  ![image-20210720205838118](.\typora-user-images\image-20210720205838118.png)

- 传统的网页(即不用ajax技术的网页)，想要更新内容或者提交一个表单，都需要重新加载整个网页。

- 使用ajax技术的网页，通过在后台服务器进行少量的数据交换，就可以实现异步局部更新。

- 使用Ajax，用户可以创建接近本地桌面应用的直接、高可用、更丰富、更动态的Web用户界面。

### 伪造Ajax

​	我们可以使用前端的一个标签来伪造一个ajax的样教。iframe标签

1. 新建一个module : sspringmvc-06-ajax ,导入web支持!

2. 编写一个ajax-frame.html使用iframe测试，感受下效果

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>iframe测试体验页面无刷新</title>

    <script>
        function go(){
            var url = document.getElementById("url").value;
            document.getElementById("iframe1").src=url;
        }
    </script>
</head>

<body>
<div>
    <p>请输入地址：</p>
    <p>
        <input type="text" id="url" value="https://www.bilibili.com/">
        <input type="submit" value="提交" onclick="go()">
    </p>

</div>

<div>
    <iframe id="iframe1" style="width: 100%;height: 500px"></iframe>
</div>
</body>
</html>
```

3. 使用idea测试一下

**利用Ajax可以做**

- 注册时，输入用户名自动检测用户是否已经存在。
- 登陆时，提示用户名密码错误
- 删除数据行时，将行ID发送到后台，后台在数据库中删除，数据库删除成功后，在页面DOM中将数据行也删除。
- ……等等

### jQuery.ajax

- Ajax的核心是XMLHttpRequest对象(XHR)。XHR为向服务器发送请求和解析服务器响应提供了接口。能够以异步方式从服务器获取新数据。
- jQuery提供多个与AJAX有关的方法。
- 通过 jQuery AJAX方法，您能够使用HTTP Get和 HTTP Post 从远程服务器上请求文本、HTML、XML或JSON –同时您能够把这些外部数据直接载入网页的被选元素中。
- jQuery不是生产者，而是大自然搬运工。
- jQuery Ajax本质就是XMLHttpRequest，对他进行了封装，方便调用!

使用最原始的HttpServletResponse处理，最简单，最通用

1. 配置web.xml和springmvc的配置文件，记得导入静态资源过滤

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xmlns:mvc="http://www.springframework.org/schema/mvc"
          xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context.xsd
           http://www.springframework.org/schema/mvc
           http://www.springframework.org/schema/mvc/spring-mvc.xsd
   
   
   ">
       <!-- 自动包扫描，让指定包下的注解生效，由IOC容器统一管理-->
       <context:component-scan base-package="com.xrl.controller"/>
       <!-- 让SpringMVC不去处理静态资源-->
       <mvc:default-servlet-handler/>
       <mvc:annotation-driven/>
       <!-- 视图解析器 -->
       <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
           <property name="prefix" value="/WEB-INF/jsp/"/>
           <property name="suffix" value=".jsp"/>
       </bean>
   </beans>
   ```

2. 编写一个AjaxController

   ```java
   @RequestMapping("/a1")
   public void a1(String name, HttpServletResponse response) throws IOException {
       System.out.println("a1:param=>" + name);
       if("xrl".equals(name)){
           response.getWriter().print("true");
       }else{
           response.getWriter().print("false");
       }
   }
   ```

3. 导入jquery，可以使用在线的cdn，也可以下载导入

   ```script
    <script src="${pageContext.request.contextPath}/statics/js/jquery-3.6.0.js"></script>
   ```

4. index.jsp测试

   ```jsp
   
   <%@ page contentType="text/html;charset=UTF-8" language="java" %>
   <html>
       <head>
           <title>$Title$</title>
           <script src="${pageContext.request.contextPath}/statics/js/jquery-3.6.0.js"></script>
           <script>
               function a() {
                   $.post({
                       url:"${pageContext.request.contextPath}/a1",
                       data:{"name":$("#username").val()}, // 这里才是传递的参数
                       success:function (data,status) {
                           console.log("data"+data); // 弹出的是后台 response.getWriter().print("数据");即这里的数据
                           console.log("status"+status);
                       }
                   });
               }
           </script>
       </head>
       <body>
           <%--失去焦点时，发起一个请求(携带信息)到后台--%>
           用户名：<input type="text" id="username" onblur="a()">
       </body>
   </html>
   
   ```

5. 启动tomcat测试!打开浏览器的控制台，当我们鼠标离开输入框的时候，可以看到发出了一个ajax的请求!是后台返回给我们的结果!测试成功!

### SpringMVC实现Ajax请求

1. 实体类

   ```java
   package com.xrl.pojo;
   
   import lombok.AllArgsConstructor;
   import lombok.Data;
   import lombok.NoArgsConstructor;
   
   @Data
   @AllArgsConstructor
   @NoArgsConstructor
   public class User {
       private String name;
       private int age;
       private String sex;
   }
   
   ```

2. 获取一个集合对象，展示到前端页面

   ```java
   @RequestMapping("/a2")
   public List<User> a2(){
       List<User> userList = new ArrayList<User>();
       userList.add(new User("狂神说java",1,"男"));
       userList.add(new User("狂神说前端",1,"女"));
       userList.add(new User("狂神说运维",1,"男"));
       return userList;
   }
   ```

3. 前端页面

   ```jsp
   <%@ page contentType="text/html;charset=UTF-8" language="java" %>
   <html>
   <head>
       <title>Title</title>
       <script src="${pageContext.request.contextPath}/statics/js/jquery-3.6.0.js"></script>
       <script>
           $(function () {
               $("#btn").click(function () {
                   $.post("${pageContext.request.contextPath}/a2", function (data) {
                       var html = "";
   
                       for (let i = 0; i < data.length; i++) {
                           html += "<tr>" +
                               "<td>" + data[i].name + "</td>" +
                               "<td>" + data[i].age + "</td>" +
                               "<td>" + data[i].sex + "</td>" +
                               "</tr>"
                       }
                       $("#content").html(html);
                   });
               })
           });
       </script>
   </head>
   <body>
   <input type="button" value="加载数据" id="btn">
   <table>
       <tr>
           <th>姓名</th>
           <th>年龄</th>
           <th>性别</th>
       </tr>
       <tbody id="content">
       </tbody>
   </table>
   
   </body>
   </html>
   
   ```

4. 成功实现了数据回显！

   ![image-20210721211153713](.\typora-user-images\image-20210721211153713.png)

### 注册提示效果

​	我们再测试一个小Demo，思考一下我们平时注册时候，输入框后面的实时提示怎么做到的;

**请求**

```java
@RequestMapping("/a3")
public String a3(String name, String pwd) {
    String msg = "";
    if (name != null) {
        // admin 的所有信息应该从数据库中查询出来
        if ("admin".equals(name)) {
            msg = "ok";
        } else {
            msg = "用户名有误";
        }
    }
    if (pwd != null) {
        // admin 的所有信息应该从数据库中查询出来
        if ("123456".equals(pwd)) {
            msg = "ok";
        } else {
            msg = "密码有误";
        }
    }
    return msg;
}
```

**前端页面**

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
    <head>
        <title>Title</title>
        <script src="${pageContext.request.contextPath}/statics/js/jquery-3.6.0.js"></script>
        <script>
            function a1() {
                $.post({
                    url: "${pageContext.request.contextPath}/a3",
                    data: {"name": $("#name").val()},
                    success: function (data) {
                        if (data.toString() === "ok") {
                            $("#userInfo").css("color", "green");
                        }else{
                            $("#userInfo").css("color", "red");
                        }
                        $("#userInfo").html(data.toString())
                    }
                });
            }

            function a2() {
                $.post({
                    url: "${pageContext.request.contextPath}/a3",
                    data: {"pwd": $("#pwd").val()},
                    success: function (data) {
                        if (data.toString() === "ok") {
                            $("#pwdInfo").css("color", "green");
                        }else{
                            $("#pwdInfo").css("color", "red");
                        }
                        $("#pwdInfo").html(data)

                    }
                });
            }
        </script>
    </head>
    <body>
        <p>
            用户名：<input type="text" id="name" onblur="a1()">
            <span id="userInfo"></span>
        </p>


        <p>
            密码：<input type="text" id="pwd" onblur="a2()">
            <span id="pwdInfo"></span>
        </p>

    </body>
</html>

```

**数据展示**（注意地址栏的地址没有改变）

![image-20210721221935301](.\typora-user-images\image-20210721221935301.png)





## Spring MVC 拦截器

### 概述

​		SpringMVC的处理器拦截器类似于Servlet开发中的过滤器Filter,用于对处理器进行预处理和后处理。开发者可以自己定效一些拦截器来实现特定的功能。

**过滤器与拦截器的区别**: 拦截器是AOP思想的具体应用。

**过滤器**

- servlet规范中的一部分，任何java web工程都可以使用
- 在url-pattern中配置了/*之后，可以对所有要访问的资源进行过滤

**拦截器**

- 拦截器是SpringMVC框架自己的，只有使用了SpringMVC框架的工程才能使用
- 拦截器只会拦截访问的控制器方法，如果访问的是jsp/html/css/image/js是不会进行拦截的

### 自定义拦截器

想要自定义拦截器，必须实现HandleIntercept

1. 新建一个Module，springmvc-07-Interceptor,添加web支持

2. 配置web.xml和applicationContext.xml文件

   **web.xml**

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
            version="4.0">
   
       <servlet>
           <servlet-name>springmvc</servlet-name>
           <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
           <init-param>
               <param-name>contextConfigLocation</param-name>
               <param-value>classpath:applicationContext.xml</param-value>
           </init-param>
           <load-on-startup>1</load-on-startup>
       </servlet>
       <servlet-mapping>
           <servlet-name>springmvc</servlet-name>
           <url-pattern>/</url-pattern>
       </servlet-mapping>
   
       <filter>
           <filter-name>encoding</filter-name>
           <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
           <init-param>
               <param-name>encoding</param-name>
               <param-value>utf-8</param-value>
           </init-param>
       </filter>
       <filter-mapping>
           <filter-name>encoding</filter-name>
           <url-pattern>/*</url-pattern>
       </filter-mapping>
   </web-app>
   ```

   **applicationContext.xml**

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xmlns:mvc="http://www.springframework.org/schema/mvc"
          xsi:schemaLocation="
                              http://www.springframework.org/schema/beans
                              http://www.springframework.org/schema/beans/spring-beans.xsd
                              http://www.springframework.org/schema/context
                              http://www.springframework.org/schema/context/spring-context.xsd
                              http://www.springframework.org/schema/mvc
                              http://www.springframework.org/schema/mvc/spring-mvc.xsd
   
   
                              ">
       <!-- 自动包扫描，让指定包下的注解生效，由IOC容器统一管理-->
       <context:component-scan base-package="com.xrl.controller"/>
       <!-- 让SpringMVC不去处理静态资源-->
       <mvc:default-servlet-handler/>
       <mvc:annotation-driven>
           <!--处理JSON乱码问题-->
           <mvc:message-converters register-defaults="true">
               <bean class="org.springframework.http.converter.StringHttpMessageConverter">
                   <constructor-arg value="UTF-8"/>
               </bean>
               <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
                   <property name="objectMapper">
                       <bean class="org.springframework.http.converter.json.Jackson2ObjectMapperFactoryBean">
                           <property name="failOnEmptyBeans" value="false"/>
                       </bean>
                   </property>
               </bean>
           </mvc:message-converters>
       </mvc:annotation-driven>
       <!-- 视图解析器 -->
       <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
           <property name="prefix" value="/WEB-INF/jsp/"/>
           <property name="suffix" value=".jsp"/>
       </bean>
   
       <!--拦截器配置-->
       <mvc:interceptors>
           <mvc:interceptor>
               <!--表示包括 / 请求下的所有请求
                   比如 / /admin /t1 ...
               -->
               <mvc:mapping path="/**"/>
               <bean class="com.xrl.interceptor.MyInterceptor"/>
           </mvc:interceptor>
   
           <mvc:interceptor>
               <mvc:mapping path="/user/**"/>
               <bean class="com.xrl.interceptor.LoginInterceptor"/>
           </mvc:interceptor>
       </mvc:interceptors>
   
   </beans>
   ```

3. 编写一个拦截器

```java
package com.xrl.interceptor;

import org.springframework.web.servlet.HandlerInterceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

public class LoginInterceptor implements HandlerInterceptor {
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        HttpSession session = request.getSession();
        if(request.getRequestURI().contains("goLogin")||request.getRequestURI().contains("login"))
            return true;
        if(session.getAttribute("userLoginInfo") != null)
            return true;
        request.getRequestDispatcher("/WEB-INF/jsp/login.jsp").forward(request,response);
        return false;
    }
}
```

## SpringMVC: 文件上传与下载

### 准备工作

​		文析上传是项目开发中最常见的功能之一,springMVC可以很好的支持文件上传，但是SpringMVC上下文中默认没有装配MultipartResolver，因此默认情况下其不能处理文件上传工如果想使用Spring的文件上传功能，则需要在上下文中配置MultipartResolver。

​		前端表单要求:为了能上传文件，必须将表单的method设置为POST，并将enctype设置为multipart/form-data。只有在这样的情况下，浏览器才会把用户选择的文件以二进制数据发送给服务器;

​		**对表单中enctype属性做个详细的说明：**

- application/x-www=form-urlencoded:默认方式，只处理表单域中的value 属性值，采用这种编码方式的表单会将表单域中的值处理成URL编码方式。

- multipart/form-data:这种编码方式会以二进制流的方式来处理表单数据，这种编码方式会把文件域指定文件的内容也封装到请求参数中，不会对字符编码。

- text/plain:除了把空格转换为“+"号外，其他字符都不做编码处理，这种方式适用直接通过表单发送邮件。

  ```html
  <form action="" enctype="multipart/form-data" method="post">
      <input type="file- name="file/>
      <input type=""submit">
   </form>
  ```

  ​		一旦设置了enctype为mutipart/form-data，浏览器即会采用二进制流的方式来处理表单数据，而对于文件上传的处理则涉及在服务器端解析原始的HTTP响应。在2003年，Apache SoftwareFoundation发布了开源的Commons FileUpload组件，其很快成为Servlet/JSP程序员上传文件的最佳选择。

  - Servlet3.0规范已经提供方法来处理文件上传，但这种上传需要在Servlet中完成。而Spring MVC则提供了更简单的封装。
  - Spring MVC为文件上传提供了直接的支持，这种支持是用即插即用的MultipartResolver实现的。
  - Spring MVC使用Apache Commons FileUpload技术实现了一个MultipartResolver实现类:CommonsMultipartResolver。因此，==SpringMVC的文件上传还需要依赖Apache Commons FileUpload的组件==。

### 文件上传

1. 导入文件上传的jar包,commons-fileupload, Maven会自动帮我们导入他的依赖包commons-io包;

   ```xml
   <!--文件上传-->
   <dependency>
       <groupId>commons-fileupload</groupId>
       <artifactId>commons-fileupload</artifactId>
       <version>1.4</version>
   </dependency>
   <!--servlet api-->
   <dependency>
       <groupId>javax.servlet</groupId>
       <artifactId>javax.servlet-api</artifactId>
       <version>4.0.1</version>
   </dependency>
   ```

2. 配置bean：multipartResolver

   【注意!!!这个bena的id必须为: multipartResolver，否则上传文件会报400的错误!在这里载过坑,教训!】

   ```xml
   <!-- 文件上传配置(死的)-->
   <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
       <!--请求的编码格式，必须和jsp的pageEncoding属性一致，以便正确读取表单的内容，默认为ISO-8859-1 -->
       <property name="defaultEncoding" value="utf-8"/>
       <!--上传文件大小上限，单位为字节(10485760=10M)-->
       <property name="maxUploadSize" value="10485760"/>
       <property name="maxInMemorySize" value="40960"/>
   </bean>
   ```

   ​	CommonsMultipartFile的常用方法:

   - String getOriginalFilename():获取上传文件的原名
   - lnputStream getlnputStream():获取文件流
   - void transferTo(File dest):将上传文件保存到一个目录文件中

3. 文件上传Controller

   ```java
   //@RequestParam("file")将name=file控件得到的文件封装成CommonsMultipartFile 对象
   // 批量上传CommonsMultipartFiLe则为数组即可
   @RequestMapping("/upload")
   public String fileupload(@RequestParam("file") CommonsMultipartFile file , HttpServletRequest request) throws IOException {
       // 获取文件名： file.getOriginalFileName();
       String uploadFileName = file.getOriginalFilename();
   
       // 如果文件文件名为空，直接返回首页
       if("".equals(uploadFileName)){
           return "redirect:/index";
       }
       System.out.println("上传文件名："+uploadFileName);
   
       // 上传路径保存设置
       String path = request.getServletContext().getRealPath("/upload");
   
       // 如果路径不存在，创建一个
       File realPath = new File(path);
       if(!realPath.exists()){
           realPath.mkdir();
       }
       System.out.println("上传文件保存地址："+realPath);
   
       InputStream is = file.getInputStream();// 文件输入流
       OutputStream os = new FileOutputStream(new File(realPath, uploadFileName));//文件输出流
   
       // 读取写出
       int len = 0;
       byte[] buff = new byte[1024];
       while((len=is.read(buff))!=-1){
           os.write(buff,0,len);
           os.flush();
       }
       os.close();
       is.close();
       return "redirect:/index";
   
   }
   ```

### 采用file.transferTo来保存上传的文件

1. 编写controller

   ```java
   @RequestMapping("/upload2")
   public String fileUpload2(@RequestParam("file")CommonsMultipartFile file , HttpServletRequest request) throws IOException{
   
       // 上传路径保存设置
       String path = request.getServletContext().getRealPath("/upload");
       File realPath = new File(path);
       if(!realPath.exists()){
           realPath.mkdir();
       }
       //上传文件地址
       System.out.println("上传文件保存地址："+realPath);
       //通过CommonsMultipartFile的方法直接写文件（注意这个时候)
       file.transferTo(new File(realPath +"/" +file.getOriginalFilename()));
       return  "redirect:/index";
   }
   ```

2. 前端表单提交

   ```jsp
   <%@ page contentType="text/html;charset=UTF-8" language="java" %>
   <html>
   <head>
       <title>$Title$</title>
   </head>
   <body>
   <form action="${pageContext.request.contextPath}/upload" enctype="multipart/form-data" method="post">
       <input type="file" name="file"/>
       <input type="submit" value="upload">
   </form>
   </body>
   </html>
   ```

3. 测试成功

   ![image-20210722170645723](.\typora-user-images\image-20210722170645723.png)

![image-20210722170702587](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210722170702587.png)





### 文件下载

1. 设置 response 响应头

2. 读取文件 -- InputStream

3. 写出文件 -- OutPutStream

4. 执行操作

5. 关闭流 （先开后关）

   代码实现(前端给个a连接就可以了)

   ```java
   // 文件下载
   @RequestMapping("/download")
   public String downloads(HttpServletResponse response, HttpServletRequest request) throws IOException {
   
       // 要下载的图片地址
       String path = request.getServletContext().getRealPath("/upload");
       String fileName = "bg.png";
   
       // 1. 设置 response 响应头
       response.reset(); // 设置页面不缓存，清空buffer
       response.setCharacterEncoding("UTF-8"); // 字符编码
       response.setContentType("multipart/form-data"); // 二进制传输数据
   
       // 设置响应头
       response.setHeader("content-Disposition",
               "attachment;fileName=" + URLEncoder.encode(fileName, "UTF-8"));
       File file = new File(path, fileName);
       // 2. 读取文件 -- 输入流
       InputStream input = new FileInputStream(file);
       // 3. 写出文件 -- 输出流
       ServletOutputStream out = response.getOutputStream();
       byte[] buff = new byte[1024];
       int index = 0;
       // 4. 执行操作
       while ((index = input.read(buff)) != -1) {
           out.write(buff, 0, index);
           out.flush();
       }
       out.close();
       input.close();
       return "ok";
   }
   ```





