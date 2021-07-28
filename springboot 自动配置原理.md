## springboot 自动配置原理

**原理初探 **

自动配置：





**pom.xml**

- spring-boot-dependencies: 核心依赖在父工程中
- 我们在写或者引入一些springboot依赖的时候，不需要指定版本，就是因为有这些版本仓库



**启动器**

- ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
  </dependency>
  ```

- 启动器：说白了就是springboot的启动场景

- 比如spring-boot-starte-web，他就会帮我们自动导入web环境的所有依赖！

- springboot会将所有的功能场景变成一个个启动器

- 我们要使用什么功能只需要找到对应的启动器即可 `starter`





**主程序**

```java
//@SpringBootApplication: 标注这个类是一个springboot的应用
@SpringBootApplication
public class Springboot01HelloworldApplication {

    public static void main(String[] args) {
        // 将springboot应用启动
        SpringApplication.run(Springboot01HelloworldApplication.class, args);
    }

}
```

- **注解**

  ```java
  @SpringBootConfiguration ： springboot的配置
      	@Configuration： spring配置类
      	@Component： 说明这也是spring的一个组件
  
  @EnableAutoConfiguration： 自动配置
      @AutoConfigurationPackage： 自动配置包
      	@Import(AutoConfigurationPackages.Registrar.class)： 自动配置 `包注册`
      @Import(AutoConfigurationImportSelector.class): 自动配置导入选择
      	// 获取所有的配置
      	List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
      
      
  ```

获取候选的配置

```java
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
   List<String> configurations = SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),
         getBeanClassLoader());
   Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you "
         + "are using a custom packaging, make sure that file is correct.");
   return configurations;
}
```

META-INF/spring.factories： 自动配置的核心文件

![image-20210616132334310](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210616132334310.png)

```java
Properties properties = PropertiesLoaderUtils.loadProperties(resource);
所有资源加载到配置类中
```

结论：springboot所有的自动配置都是在启动的时候扫描并加载：`spring.factories`所有的自动配置类都在这个文件里面，但是不一定会生效，要判断条件是否成立，只要导入了对应的start，就有对应的启动器了，有了启动器，我们的自动装配就会生效，然后配置成功。



1. springboot在启动的时候，从类路径下/META-INF/spring.factories获取指定的值；
2. 将这些自动配置的类导入容器，自动配置就会生效，帮我们进行自动配置
3. 以前我们需要自己配置的东西，现在springboot帮我们做了
4. 整个javaEE，解决方案和自动配置的东西都在spring-boot-autoconfigure-2.5.1.jar这个包下
5. 它会把所有需要导入的组件，以类名的方式返回，这些组件就会被添加到容器；
6. 容器中也会存在非常多的xxxAutoConfiguration的文件(@Bean),就是这些类给容器中导入了这个场景需要的所有组件；并自动配置，@Configuration 
7. 有了自动配置类，免去了我们手动编写配置文件的工作！

### yaml配置注入

```java
// 普通的spring方式注入，使用@Value注解
@Component //注册bean
public class Dog {
    @Value("阿黄")
    private String name;
    @Value("18")
    private Integer age;
}
```

![image-20210617171247649](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210617171247649.png)

### JSR303数据校验

springboot中可以使用@Validated来校验数据，如果数据异常则会统一抛出异常，方便异常中心统一处理。我们这里的例子是让name属性只支持Email格式输入，不然就报错

```java
@Component //注册bean
@Data
@ConfigurationProperties(prefix = "person")
@Validated
public class Person {
    @Email
    private String name;
...
}
```

![image-20210617172255564](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210617172255564.png)



### springboot 自动配置

我们以**HttpEncodingAutoConfiguration（Http编码自动配置）**为例解释自动配置原理；

```java

//表示这是一个配置类，和以前编写的配置文件一样，也可以给容器中添加组件；
@Configuration 

//启动指定类的ConfigurationProperties功能；
  //进入这个ServerProperties查看，将配置文件中对应的值和ServerProperties绑定起来；
  //并把ServerProperties加入到ioc容器中
@EnableConfigurationProperties({ServerProperties.class}) 

//Spring底层@Conditional注解
  //根据不同的条件判断，如果满足指定的条件，整个配置类里面的配置就会生效；
  //这里的意思就是判断当前应用是否是web应用，如果是，当前配置类生效
@ConditionalOnWebApplication(
    type = Type.SERVLET
)

//判断当前项目有没有这个类CharacterEncodingFilter；SpringMVC中进行乱码解决的过滤器；
@ConditionalOnClass({CharacterEncodingFilter.class})

//判断配置文件中是否存在某个配置：server.servlet.encoding.enabled；
  //如果不存在，判断也是成立的
  //即使我们配置文件中不配置server.servlet.encoding.enabled=true，也是默认生效的；
@ConditionalOnProperty(
    prefix = "server.servlet.encoding",
    value = {"enabled"},
    matchIfMissing = true
)

public class HttpEncodingAutoConfiguration {
    //他已经和SpringBoot的配置文件映射了
    private final Encoding properties;
    //只有一个有参构造器的情况下，参数的值就会从容器中拿
    public HttpEncodingAutoConfiguration(ServerProperties properties) {
        this.properties = properties.getEncoding();
    }
    
    //给容器中添加一个组件，这个组件的某些值需要从properties中获取
    @Bean
    @ConditionalOnMissingBean //判断容器没有这个组件？
    public CharacterEncodingFilter characterEncodingFilter() {
        CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
        filter.setEncoding(this.properties.getCharset().name());
        filter.setForceRequestEncoding(this.properties.shouldForce(org.springframework.boot.autoconfigure.http.HttpProperties.Encoding.Type.REQUEST));
        filter.setForceResponseEncoding(this.properties.shouldForce(org.springframework.boot.autoconfigure.http.HttpProperties.Encoding.Type.RESPONSE));
        return filter;
    }
    //。。。。。。。
}
```

**一句话总结 ：根据当前不同的条件判断，决定这个配置类是否生效！**

- 一但这个配置类生效；这个配置类就会给容器中添加各种组件；
- 这些组件的属性是从对应的properties类中获取的，这些类里面的每一个属性又是和配置文件绑定的；
- 所有在配置文件中能配置的属性都是在xxxxProperties类中封装着；
- 配置文件能配置什么就可以参照某个功能对应的这个属性类

**精髓**

1、SpringBoot启动会加载大量的自动配置类

2、我们看我们需要的功能有没有在SpringBoot默认写好的自动配置类当中；

3、我们再来看这个自动配置类中到底配置了哪些组件；（只要我们要用的组件存在在其中，我们就不需要再手动配置了）

4、给容器中自动配置类添加组件的时候，会从properties类中获取某些属性。我们只需要在配置文件中指定这些属性的值即可；

**xxxxAutoConfigurartion：自动配置类；**给容器中添加组件

**xxxxProperties:封装配置文件中相关属性；**

我们怎么知道哪些自动配置类生效？

**我们可以通过启用 debug=true属性；来让控制台打印自动配置报告，这样我们就可以很方便的知道哪些自动配置类生效；**

```java
//在yaml中配置
#开启springboot的调试类
debug=true
```

**Positive matches:（自动配置类启用的：正匹配）**

**Negative matches:（没有启动，没有匹配成功的自动配置类：负匹配）**

**Unconditional classes: （没有条件的类）**

【演示：查看输出的日志】



### 静态资源

```java
	@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    if (!this.resourceProperties.isAddMappings()) {
        logger.debug("Default resource handling disabled");
        return;
    }
    addResourceHandler(registry, "/webjars/**", "classpath:/META-INF/resources/webjars/");
    addResourceHandler(registry, this.mvcProperties.getStaticPathPattern(), (registration) -> {
        registration.addResourceLocations(this.resourceProperties.getStaticLocations());
        if (this.servletContext != null) {
            ServletContextResource resource = new ServletContextResource(this.servletContext, SERVLET_LOCATION);
            registration.addResourceLocations(resource);
        }
    });
}
```

总结：

1. 在springboot中，我们可以使用一下方式处理静态资源
   - webjars  映射地址为 `localhost:8080/webjars/`
   - classpath:[/META-INF/resources/,/resources/, /static/, /public/]   映射地址为 `localhost:8080/`
2. 优先级： resources > static(默认) > public



### 首页如何定制(springboot-03-web)

放在静态资源的路径下即可，但是如果放在了类路径下的templates中，则需要使用controller层进行跳转才能访问

### 模板引擎

```java
@ConfigurationProperties(prefix = "spring.thymeleaf")
public class ThymeleafProperties {

	private static final Charset DEFAULT_ENCODING = StandardCharsets.UTF_8;

	public static final String DEFAULT_PREFIX = "classpath:/templates/";

	public static final String DEFAULT_SUFFIX = ".html";
......
}
```

结论

只要需要使用thymeleaf，只需要导入对应的依赖就可以了！我们将我们的html页面放在templates目录下即可







### 扩展SpringMVC

If you want to keep those Spring Boot MVC customizations and make more [MVC customizations](https://docs.spring.io/spring-framework/docs/5.3.8/reference/html/web.html#mvc) (interceptors, formatters, view controllers, and other features), you can add your own `@Configuration` class of type `WebMvcConfigurer` but **without** `@EnableWebMvc`.

If you want to provide custom instances of `RequestMappingHandlerMapping`, `RequestMappingHandlerAdapter`, or `ExceptionHandlerExceptionResolver`, and still keep the Spring Boot MVC customizations, you can declare a bean of type `WebMvcRegistrations` and use it to provide custom instances of those components.

If you want to take complete control of Spring MVC, you can add your own `@Configuration` annotated with `@EnableWebMvc`, or alternatively add your own `@Configuration`-annotated `DelegatingWebMvcConfiguration` as described in the Javadoc of `@EnableWebMvc`.

```java
@Configuration
// 如果我们想diy一些定制化的功能，只要编写这个组件，然后加入到springboot中，springboot就会给我们自动装配，这是springboot官方推荐的方式
public class MyMvcConfig implements WebMvcConfigurer {

    // public interface ViewResolver(视图解析器接口)，只要实现了该接口的类，我们可以将该类认为是一个视图解析器
    @Bean
    public ViewResolver myViewResolver(){
        return new MyViewResolver();
    }


    // 自定义了一个自己的视图解析器MyViewResolver
    public static class MyViewResolver implements ViewResolver{

        @Override
        public View resolveViewName(String s, Locale locale) throws Exception {
            return null;
        }
    }

    // 实现视图跳转方法，则可以修改默认的视图跳转，访问http://localhost:8080/xrl   ---> 跳转到test.html页面
    @Override 
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("xrl").setViewName("test");   
    }
}
```

总结： 在springboot有非常多的XxxConfiguration它会帮助进行扩展配置，只要看到了他们，我们就要注意了

### 首页定制

1. 首页配置
   1. 注意点，所有的页面静态资源都需要使用thymeleaf接管
   2. url 使用 @{}，其中第一个“/”表示静态资源的默认路径`th:href="@{/index(l='zh_CN')}"`
2. 页面国际化
   1. 我们需要配置i18n文件
   2. 我们如果需要在项目中进行按钮自动切换，我们需要自定义一个组件去实现LocaleResolver接口
   3. 然后将自定义的组件放在容器中
   4. 前端使用#{}来获取   `th:text="#{login.remember}"`

```java
package com.xrl.config;

import org.springframework.web.servlet.LocaleResolver;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.Locale;

public class MyLocaleResolver implements LocaleResolver {
    // 解析请求
    @Override
    public Locale resolveLocale(HttpServletRequest request) {
        // 获取请求中的语言参数
        String language = request.getParameter("l");
        System.out.println("huaisd");
        Locale locale = Locale.getDefault();
        // 如果请求链接带了国际化参数
        if(!"".equals(language) && language != null){
            String[] split = language.split("_");
            locale = new Locale(split[0],split[1]);
        }
        return locale;
    }

    @Override
    public void setLocale(HttpServletRequest request, HttpServletResponse response, Locale locale) {

    }
}



@Configuration
public class MyMvcConfig implements WebMvcConfigurer {
    @Bean
    public LocaleResolver localeResolver(){
        return new MyLocaleResolver();
    }
}

```

3. 登录 + 拦截器

```java
@Configuration
public class MyMvcConfig implements WebMvcConfigurer {

    @Override
    // 登录处理
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("index");
        registry.addViewController("/index").setViewName("index");
        registry.addViewController("/main").setViewName("dashboard");
    }

    @Override
    // 拦截器处理
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginHandlerInterceptor()).addPathPatterns("/**")
                .excludePathPatterns("/index","/user/login","/","/css/*","/js/*","/img/*");
    }
}



import org.springframework.web.servlet.HandlerInterceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

public class LoginHandlerInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        HttpSession session = request.getSession();
        Object loginUser = session.getAttribute("loginUser");
        if(loginUser == null){
            request.setAttribute("msg","请先登录");
            request.getRequestDispatcher("/index").forward(request,response);
            return false;
        }
        return true;
    }
}

```

4. CRUD
    	1. 提取公共页面
         	1. `th:fragment="sidebar"`
         	2. `th:replace="~{commons/commons::topbar}`
         	3. 如果要传递参数，可以直接使用()传参，`th:replace="~{commons/commons::sidebar(active='main')}`接收判断即可
   
   2. 列表循环展示
   
5. 添加员工

    1. 按钮提交
    2. 跳转到添加页面
    3. 添加员工成功
    4. 返回首页

6. CRUD搞定

7. 404 

    只要将处理错误的页面放在templates目录下的error(自己建立)下，然后页面名称标记成错误状态码.html就可以了







### Data

### 认识 SpringSecurity(springboot-06-security)

Spring Security 是针对Spring项目的安全框架，也是Spring Boot底层安全模块默认的技术选型，他可以实现强大的Web安全控制，对于安全控制，我们仅需要引入 spring-boot-starter-security 模块，进行少量的配置，即可实现强大的安全管理！

记住几个类：

- WebSecurityConfigurerAdapter：自定义Security策略
- AuthenticationManagerBuilder：自定义认证策略
- @EnableWebSecurity：开启WebSecurity模式

Spring Security的两个主要目标是 “认证” 和 “授权”（访问控制）。

**“认证”（Authentication）**

身份验证是关于验证您的凭据，如用户名/用户ID和密码，以验证您的身份。

身份验证通常通过用户名和密码完成，有时与身份验证因素结合使用。

 **“授权” （Authorization）**

授权发生在系统成功验证您的身份后，最终会授予您访问资源（如信息，文件，数据库，资金，位置，几乎任何内容）的完全权限。

这个概念是通用的，而不是只在Spring Security 中存在。

**认证授权**

目前，我们的测试环境，是谁都可以访问的，我们使用 Spring Security 增加上认证和授权的功能

1、引入 Spring Security 模块

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

2、编写 Spring Security 配置类

参考官网：https://spring.io/projects/spring-security 

查看我们自己项目中的版本，找到对应的帮助文档：

https://docs.spring.io/spring-security/site/docs/5.3.0.RELEASE/reference/html5  #servlet-applications 8.16.4

![image-20210628151940697](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210628151940697.png)

3、编写基础配置类

```java
package com.kuang.config;

import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@EnableWebSecurity // 开启WebSecurity模式
public class SecurityConfig extends WebSecurityConfigurerAdapter {

   @Override
   protected void configure(HttpSecurity http) throws Exception {
       
  }
}
```

4、定制请求的授权规则

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
   // 定制请求的授权规则
   // 首页所有人可以访问
   http.authorizeRequests().antMatchers("/").permitAll()
  .antMatchers("/level1/**").hasRole("vip1")
  .antMatchers("/level2/**").hasRole("vip2")
  .antMatchers("/level3/**").hasRole("vip3");
}
```

5、测试一下：发现除了首页都进不去了！因为我们目前没有登录的角色，因为请求需要登录的角色拥有对应的权限才可以！

6、在configure()方法中加入以下配置，开启自动配置的登录功能！

```java
// 开启自动配置的登录功能
// /login 请求来到登录页
// /login?error 重定向到这里表示登录失败
http.formLogin();
```

7、测试一下：发现，没有权限的时候，会跳转到登录的页面！



![Image](https://mmbiz.qpic.cn/mmbiz_png/uJDAUKrGC7JolV3xA4rEtxSCgbN76QbXMBrh9xGQDQjlW4EpV2DscJbUzAMwjqIGGgw8fqWET4swGIue9mbwwQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

8、查看刚才登录页的注释信息；

我们可以定义认证规则，重写configure(AuthenticationManagerBuilder auth)方法

```
//定义认证规则
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
   
   //在内存中定义，也可以在jdbc中去拿....
   auth.inMemoryAuthentication()
          .withUser("kuangshen").password("123456").roles("vip2","vip3")
          .and()
          .withUser("root").password("123456").roles("vip1","vip2","vip3")
          .and()
          .withUser("guest").password("123456").roles("vip1","vip2");
}
```

9、测试，我们可以使用这些账号登录进行测试！发现会报错！

There is no PasswordEncoder mapped for the id “null”

![Image](https://mmbiz.qpic.cn/mmbiz_png/uJDAUKrGC7JolV3xA4rEtxSCgbN76QbXSHnC5kd7W2DeryMuzoSOx3evDWoqlVIoGxBA3TEjAF54s4cRQsld0g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

10、原因，我们要将前端传过来的密码进行某种方式加密，否则就无法登录，修改代码

```java
//定义认证规则
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
   //在内存中定义，也可以在jdbc中去拿....
   //Spring security 5.0中新增了多种加密方式，也改变了密码的格式。
   //要想我们的项目还能够正常登陆，需要修改一下configure中的代码。我们要将前端传过来的密码进行某种方式加密
   //spring security 官方推荐的是使用bcrypt加密方式。
   
   auth.inMemoryAuthentication().passwordEncoder(new BCryptPasswordEncoder())
          .withUser("kuangshen").password(new BCryptPasswordEncoder().encode("123456")).roles("vip2","vip3")
          .and()
          .withUser("root").password(new BCryptPasswordEncoder().encode("123456")).roles("vip1","vip2","vip3")
          .and()
          .withUser("guest").password(new BCryptPasswordEncoder().encode("123456")).roles("vip1","vip2");
}
```

11、测试，发现，登录成功，并且每个角色只能访问自己认证下的规则！搞定



**权限控制与注销**

1、开启自动配置的注销的功能

```java
//定制请求的授权规则
@Override
protected void configure(HttpSecurity http) throws Exception {
   //....
   //开启自动配置的注销的功能
      // /logout 注销请求
   http.logout();
}
```

2、我们在前端，增加一个注销的按钮，index.html 导航栏中

```html
<a class="item" th:href="@{/logout}">
   <i class="address card icon"></i> 注销
</a>
```

3、我们可以去测试一下，登录成功后点击注销，发现注销完毕会跳转到登录页面！

4、但是，我们想让他注销成功后，依旧可以跳转到首页，该怎么处理呢？

```java
// .logoutSuccessUrl("/"); 注销成功来到首页
http.logout().logoutSuccessUrl("/");
```

5、测试，注销完毕后，发现跳转到首页OK

6、我们现在又来一个需求：用户没有登录的时候，导航栏上只显示登录按钮，用户登录之后，导航栏可以显示登录的用户信息及注销按钮！还有就是，比如kuangshen这个用户，它只有 vip2，vip3功能，那么登录则只显示这两个功能，而vip1的功能菜单不显示！这个就是真实的网站情况了！该如何做呢？

我们需要结合thymeleaf中的一些功能

sec：authorize="isAuthenticated()":是否认证登录！来显示不同的页面

Maven依赖：

```xml
<!-- https://mvnrepository.com/artifact/org.thymeleaf.extras/thymeleaf-extras-springsecurity5 -->
<dependency>
   <groupId>org.thymeleaf.extras</groupId>
   <artifactId>thymeleaf-extras-springsecurity5</artifactId>
   <version>3.0.4.RELEASE</version>
</dependency>
```

7、修改我们的 前端页面

1. 导入命名空间

2. ```html
   xmlns:sec="http://www.thymeleaf.org/spring-security"
   ```

3. 修改导航栏，增加认证判断

4. ```html
   <!--登录注销-->
   <div class="right menu">
   
      <!--如果未登录-->
      <div sec:authorize="!isAuthenticated()">
          <a class="item" th:href="@{/login}">
              <i class="address card icon"></i> 登录
          </a>
      </div>
   
      <!--如果已登录-->
      <div sec:authorize="isAuthenticated()">
          <a class="item">
              <i class="address card icon"></i>
             用户名：<span sec:authentication="principal.username"></span>
             角色：<span sec:authentication="principal.authorities"></span>
          </a>
      </div>
   
      <div sec:authorize="isAuthenticated()">
          <a class="item" th:href="@{/logout}">
              <i class="address card icon"></i> 注销
          </a>
      </div>
   </div>
   ```

8、重启测试，我们可以登录试试看，登录成功后确实，显示了我们想要的页面；

9、如果注销404了，就是因为它默认防止csrf跨站请求伪造，因为会产生安全问题，我们可以将请求改为post表单提交，或者在spring security中关闭csrf功能；我们试试：在 配置中增加 `http.csrf().disable();`

```java
http.csrf().disable();//关闭csrf功能:跨站请求伪造,默认只能通过post方式提交logout请求
http.logout().logoutSuccessUrl("/");
```

10、我们继续将下面的角色功能块认证完成！

```html
<!-- sec:authorize="hasRole('vip1')" -->
<div class="column" sec:authorize="hasRole('vip1')">
   <div class="ui raised segment">
       <div class="ui">
           <div class="content">
               <h5 class="content">Level 1</h5>
               <hr>
               <div><a th:href="@{/level1/1}"><i class="bullhorn icon"></i> Level-1-1</a></div>
               <div><a th:href="@{/level1/2}"><i class="bullhorn icon"></i> Level-1-2</a></div>
               <div><a th:href="@{/level1/3}"><i class="bullhorn icon"></i> Level-1-3</a></div>
           </div>
       </div>
   </div>
</div>

<div class="column" sec:authorize="hasRole('vip2')">
   <div class="ui raised segment">
       <div class="ui">
           <div class="content">
               <h5 class="content">Level 2</h5>
               <hr>
               <div><a th:href="@{/level2/1}"><i class="bullhorn icon"></i> Level-2-1</a></div>
               <div><a th:href="@{/level2/2}"><i class="bullhorn icon"></i> Level-2-2</a></div>
               <div><a th:href="@{/level2/3}"><i class="bullhorn icon"></i> Level-2-3</a></div>
           </div>
       </div>
   </div>
</div>

<div class="column" sec:authorize="hasRole('vip3')">
   <div class="ui raised segment">
       <div class="ui">
           <div class="content">
               <h5 class="content">Level 3</h5>
               <hr>
               <div><a th:href="@{/level3/1}"><i class="bullhorn icon"></i> Level-3-1</a></div>
               <div><a th:href="@{/level3/2}"><i class="bullhorn icon"></i> Level-3-2</a></div>
               <div><a th:href="@{/level3/3}"><i class="bullhorn icon"></i> Level-3-3</a></div>
           </div>
       </div>
   </div>
</div>
```

11、测试一下！

12、权限控制和注销搞定！



**记住我**

现在的情况，我们只要登录之后，关闭浏览器，再登录，就会让我们重新登录，但是很多网站的情况，就是有一个记住密码的功能，这个该如何实现呢？很简单

1、开启记住我功能

```java
//定制请求的授权规则
@Override
protected void configure(HttpSecurity http) throws Exception {
//。。。。。。。。。。。
   //记住我
   http.rememberMe();
}
```

2、我们再次启动项目测试一下，发现登录页多了一个记住我功能，我们登录之后关闭 浏览器，然后重新打开浏览器访问，发现用户依旧存在！

思考：如何实现的呢？其实非常简单

我们可以查看浏览器的cookie

![Image](https://mmbiz.qpic.cn/mmbiz_png/uJDAUKrGC7JolV3xA4rEtxSCgbN76QbXqDg8FodHRKUM8K5z79zEghbybrKD6WtqO0B9JBkD6FQNQ6dhARQsTA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

3、我们点击注销的时候，可以发现，spring security 帮我们自动删除了这个 cookie

![Image](https://mmbiz.qpic.cn/mmbiz_png/uJDAUKrGC7JolV3xA4rEtxSCgbN76QbXS8kt9d3jGhJPZGNa5V97ZKVBIUNdHmtPibNia7U59tbfQyzla4mHFtYg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

4、结论：登录成功后，将cookie发送给浏览器保存，以后登录带上这个cookie，只要通过检查就可以免登录了。如果点击注销，则会删除这个cookie。

**定制登录页**

现在这个登录页面都是spring security 默认的，怎么样可以使用我们自己写的Login界面呢？

1、在刚才的登录页配置后面指定 loginpage

```java
http.formLogin().loginPage("/toLogin");
```

2、然后前端也需要指向我们自己定义的 login请求

```html
<a class="item" th:href="@{/toLogin}">
   <i class="address card icon"></i> 登录
</a>
```

3、我们登录，需要将这些信息发送到哪里，我们也需要配置，login.html 配置提交请求及方式，方式必须为post:

在 loginPage()源码中的注释上有写明：

![Image](https://mmbiz.qpic.cn/mmbiz_png/uJDAUKrGC7JolV3xA4rEtxSCgbN76QbXCxA0YjyGXDmBHOMYfpwolJ5yZxvMAINJRvTx7HBwyTtO4azI2QuqdA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

```html
<form th:action="@{/login}" method="post">
   <div class="field">
       <label>Username</label>
       <div class="ui left icon input">
           <input type="text" placeholder="Username" name="username">
           <i class="user icon"></i>
       </div>
   </div>
   <div class="field">
       <label>Password</label>
       <div class="ui left icon input">
           <input type="password" name="password">
           <i class="lock icon"></i>
       </div>
   </div>
   <input type="submit" class="ui blue submit button"/>
</form>
```

4、这个请求提交上来，我们还需要验证处理，怎么做呢？我们可以查看formLogin()方法的源码！我们配置接收登录的用户名和密码的参数！

```java
http.formLogin()
  .usernameParameter("username")
  .passwordParameter("password")
  .loginPage("/toLogin")
  .loginProcessingUrl("/login"); // 登陆表单提交请求(如果前端提交的请求与loginPage()的参数不一致时，加上该方法则可改变，类似于参数接受)
								// 即如果我前端登录提交的是/xxx,然而loginPage中的参数是/AAA,此时用上loginProcessingUrl("/xxx")一样可以接收
```

5、在登录页增加记住我的多选框

```html
<input type="checkbox" name="remember"> 记住我
```

6、后端验证处理！

```java
//定制记住我的参数！
http.rememberMe().rememberMeParameter("remember");
```

7、测试，OK

8、完整配置代码

```java
package com.xrl.config;

import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {


    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // 首页所有人都可以访问，功能页只有有特定权限的人才能访问,写法是死的

        // 请求授权的规则
        http.authorizeRequests()
                .mvcMatchers("/").permitAll() // mvcMatchers("/")编写访问路径，permitAll()表示所有人都可以访问首页
                .mvcMatchers("/level1/**").hasRole("vip1")   // 表示level1下的所有资源只有vip1的人可以访问
                .mvcMatchers("/level2/**").hasRole("vip2")
                .mvcMatchers("/level3/**").hasRole("vip3");

        // 没有权限会跳转到登录页面,开启了登录页面的功能
        // 自定义登录页面  .loginPage("/toLogin"),并自定义接受前端参数
        http.formLogin().loginPage("/toLogin").usernameParameter("user").passwordParameter("pwd").loginProcessingUrl("/login");

        // 注销。开启了注销功能,注销成功会跳到首页
        // 防止网站攻击
        http.csrf().disable(); // 关闭csrf功能，然后注销会直接跳到首页
        http.logout().logoutSuccessUrl("/");

        // 开启记住我功能, cookie技术，默认保存两周 自定义接受前端参数
        http.rememberMe().rememberMeParameter("remember");

    }

    // 认证(认证用户的角色信息) springboot 2.1.x可以直接使用，不用进行加密设置
    // 现在会报错 PasswordEncoder 就是密码没有编码
    // 在spring security5.0+ 新增了很多加密方法
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        // 这里是从内存中去读
        // 认证多个用户时使用and()连接
        /*auth.inMemoryAuthentication() // 未加密会报错
                .withUser("rlxiang").password("123456").roles("vip2","vip3")
                .and()
                .withUser("root").password("123456").roles("vip1","vip2","vip3")
                .and()
                .withUser("guest").password("123456").roles("vip1");*/

        // 对密码加密的写法
        auth.inMemoryAuthentication().passwordEncoder(new BCryptPasswordEncoder())
                .withUser("rlxiang").password(new BCryptPasswordEncoder().encode("123456")).roles("vip2","vip3")
                .and()
                .withUser("root").password(new BCryptPasswordEncoder().encode("123456")).roles("vip1","vip2","vip3")
                .and()
                .withUser("guest").password(new BCryptPasswordEncoder().encode("123456")).roles("vip1");
    }
}

```



### Shiro

1、导入依赖

2、配置文件

3、HelloWorld



```java
// 获取当前用户对象 Subject
Subject currentUser = SecurityUtils.getSubject();
// 通过当前用户拿到session
Session session = currentUser.getSession();
 // 判断当前的用户是否被认证
currentUser.isAuthenticated()
 // 获取当前被认证的username
currentUser.getPrincipal()
 // 获取当前用户的角色信息		
currentUser.hasRole("schwartz")
 // 获取当前用户的权限
 currentUser.isPermitted("lightsaber:wield")
 // 注销
 currentUser.logout();
// 这些函数在spring-security中都有
```

shiro核心三大对象 `subject` (代表用户)，`SecurityManager`(管理所有用户)，`Realm` 连接数据

SpringBoot集成Shiro

```xml
<!-- https://mvnrepository.com/artifact/org.apache.shiro/shiro-spring -->
<!--shiro 整合spring的包-->
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring</artifactId>
    <version>1.7.1</version>
</dependency>
<!--shiro与Thymeleaf的整合包-->
<!-- https://mvnrepository.com/artifact/com.github.theborakompanioni/thymeleaf-extras-shiro -->
<dependency>
    <groupId>com.github.theborakompanioni</groupId>
    <artifactId>thymeleaf-extras-shiro</artifactId>
    <version>2.0.0</version>
</dependency>
```

```java
package com.xrl.config;

import com.xrl.pojo.User;
import com.xrl.service.UserService;
import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authc.*;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.authz.SimpleAuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.session.Session;
import org.apache.shiro.subject.PrincipalCollection;
import org.apache.shiro.subject.Subject;
import org.springframework.beans.factory.annotation.Autowired;


// 自定义的Realm
public class UserRealm extends AuthorizingRealm {

    @Autowired
    UserService userService;

    // 授权
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principal) {
        System.out.println("执行了=》授权doGetAuthorizationInfo");
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
        // 给用户授权
//        info.addStringPermission("user:add");

        // 拿到当前用户对象
        Subject subject = SecurityUtils.getSubject();
        User currentUser = (User) subject.getPrincipal();

        // 从数据库中取出权限并授权
        info.addStringPermission(currentUser.getPerms());


        return info;
    }


    // 认证
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        System.out.println("执行了=》认证doGetAuthenticationInfo");

        UsernamePasswordToken userToken = (UsernamePasswordToken) token;
        // 用户名 密码  数据库中取
        User user = userService.queryUserByName(userToken.getUsername());
        if (user == null) { // 没有该用户
            return null; // 返回null 就会抛出 UnknownAccountException
        }
        /*System.out.println(getName()); // com.xrl.config.UserRealm_0
        System.out.println(userToken.getUsername()); // rlxiang
        System.out.println(userToken.getCredentials()); // [C@42529dd1
        System.out.println(userToken.getPassword()); // 123456
        System.out.println(userToken.getPrincipal());*/// rlxiang

        Subject subject = SecurityUtils.getSubject();
        Session currentSession = subject.getSession();
        currentSession.setAttribute("loginUser",user); // 把用户存在shiro的session中
        // 密码的认证不需要我们去做，shiro会自动去(密码可以进行加密)
        return new SimpleAuthenticationInfo(user, user.getPwd(), "");
    }
}
```

```java
package com.xrl.config;

import at.pollux.thymeleaf.shiro.dialect.ShiroDialect;
import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.LinkedHashMap;
import java.util.Map;

@Configuration
public class ShiroConfig {

    // ShiroFilterFactoryBean 第三步
    @Bean
    public ShiroFilterFactoryBean getShiroFilterFactoryBean(@Qualifier("getDefaultWebSecurityManager") DefaultWebSecurityManager defaultWebSecurityManager){
        ShiroFilterFactoryBean bean = new ShiroFilterFactoryBean();
        // 设置安全管理器
        bean.setSecurityManager(defaultWebSecurityManager);

        //添加shiro的内置过滤器（shiro 中都是过滤器）
        /*
            anon: 无需认证就可以访问
            authc: 必须认证了才能访问
            user: 必须拥有 记住我 功能才能访问
            perms: 拥有对某个资源的权限才能访问、
            role: 拥有某个角色权限才能访问

//        filterMap.put("/user/add","authc");
//        filterMap.put("/user/update","authc");
         */

        // 登录拦截
        Map<String, String> filterMap = new LinkedHashMap<>();

        // 设置权限，正常情况下，如果用户没有授权，那么访问时会跳到未授权页面
        filterMap.put("/user/add","perms[user:add]"); // 必须带有user:add才能访问
        filterMap.put("/user/update","perms[user:update]"); // 必须带有user:add才能访问

        filterMap.put("/user/*","authc");
        bean.setFilterChainDefinitionMap(filterMap); // 设置过滤器链

        // 设置登录请求
        bean.setLoginUrl("/toLogin");

        // 设置未授权页面
        bean.setUnauthorizedUrl("/noauth");
        return bean;
    }

    // DefaultWebSecurityManager 第二步(死代码)

    // @Qualifier("userRealm") 该注解表示指定某个bean与参数绑定
    @Bean
    public DefaultWebSecurityManager getDefaultWebSecurityManager(@Qualifier("userRealm") UserRealm userRealm){
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();

        // 关联realm对象
        securityManager.setRealm(userRealm);
        return securityManager;
    }

    // 创建 realm对象，需要自定义类 第一步

    @Bean
    public UserRealm userRealm(){
        return new UserRealm();
    }

    // 整合shiroDialect: 用来整合 shiro 与 thymeleaf
    @Bean
    public ShiroDialect getShiroDialect(){
        return new ShiroDialect();
    }
}
```



## Swagger

学习目标：

- 了解Swagger的作用
- 了解前后端分离
- 在SpringBoot中集成Swaager





### Swagger 简介

前后端分离时代：

- 后端：后端控制层，服务层，数据访问层【后端团队的事】
- 前端：前端控制层，视图层【前端团队】
  - 未扫后端数据，json。已经存在了，需要后端，全段工程依旧能跑起来
- 前端后台如何交互？  ===》 API 访问
- 前后端相对独立，并且松耦合；
- 前后端甚至可以部署在不同的服务器上；



以上会产生一个问题：

- 前后端集成联调时，前端人员和后端人员无法做到及时协商，该问题需要尽早解决，不然问题就会集中爆发

解决方案：

- 首先指定schema[计划的提纲]，实时更新最新的API（即后端做出改变，前端立马能知道做了那些变动）,降低集成风险；
- 早些年：制定word计划文档；
- 前后端分离：
  - 前端测试后端接口：postman
  - 后端提供接口，需要实时更新的消息及改动！(推荐)

在以上情况下Swagger诞生，他就是针对后端提供实时更新的接口设计的



### Swagger

- 号称世界上最流行的Api框架；
- RestFul Api 文档在线自动生成工具=》==Api文档与API定义同步更新==
- 直接运行，可以在线测试API接口；
- 支持多种语言：（Java，Php……）



在项目中使用Swagger需要springfox

- swagger2（jar 包）
- ui（jar 包）



### SpringBoot集成Swagger

1. 新建一个SpringBoot web项目

2. 导入相关依赖

   ```xml
   <!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger2 -->
   <dependency>
       <groupId>io.springfox</groupId>
       <artifactId>springfox-swagger2</artifactId>
       <version>2.9.2</version>
   </dependency>
   <!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger-ui -->
   <dependency>
       <groupId>io.springfox</groupId>
       <artifactId>springfox-swagger-ui</artifactId>
       <version>2.9.2</version>
   </dependency>
   
   
   ```

3. 编写一个Hello工程

4. 配置Swagger  == 》 Config

   ```java
   @Configuration
   @EnableSwagger2    // 开启Swagger2
   public class SwaggerConfig {
   
   }
   ```

5. 测试运行 http://localhost:8080/swagger-ui.html

   ![image-20210725221935219](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210725221935219.png)



### 配置Swagger

配置 Swagger的bean实例 Docket

```java
package com.xrl.swagger.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

import java.util.ArrayList;

@Configuration
@EnableSwagger2    // 开启Swagger2
public class SwaggerConfig {

    // 配置了Swagger的Docket的bean实例
    @Bean
    public Docket getDocket(){
        return new Docket(DocumentationType.SWAGGER_2).apiInfo(getApiInfo());
    }

    // 配置Swagger信息 ==》ApiInfo
    private ApiInfo getApiInfo(){

        // 作者信息
        Contact contact = new Contact("向锐龙", "https://www.baidu.com", "594663327@qq.com");
        return new ApiInfo("rlxiang's的SwaggerAPI文档",
                "加油",
                "1.0",
                "https://www.baidu.com",
                contact,
                "Apache 2.0",
                "http://www.apache.org/licenses/LICENSE-2.0",
                new ArrayList()
        );
    }
}
// 以上配置为swagger的基本信息  认识一下
```

测试结果（观察与之前的页面的区别）

![image-20210725224155156](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210725224155156.png)

### Swagger配置扫描接口

Docket.select()

```java
package com.xrl.swagger.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import springfox.documentation.RequestHandler;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

import java.util.ArrayList;

@Configuration
@EnableSwagger2    // 开启Swagger2
public class SwaggerConfig {

    // 配置了Swagger的Docket的bean实例
    @Bean
    public Docket getDocket(){
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(getApiInfo())
                .select()
                // RequestHandlerSelectors 配置要扫描接口的方式
                // basePackage(): 指定要扫描的包
                // any(): 扫描所有
                // none(): 都不扫描
                // withClassAnnotation(RestController.class): 扫描带有指定注解的类,参数为能在类上标注的注解的class
                // withMethodAnnotation(RequestMapping.class): 扫描方法上带有指定注解的方法，参数为能在方法上标注的注解的class
                // 以上所有的扫描方式生成的文档都是在包含这些指定参数的类中
                .apis(RequestHandlerSelectors.basePackage("com.xrl.swagger"))
                // 过滤，只有请求中带有/xrl的才能通过，才会生成api
                .paths(PathSelectors.ant("/xrl/**"))
                .build();
    }

    // 配置Swagger信息 ==》ApiInfo
    private ApiInfo getApiInfo(){

        // 作者信息
        Contact contact = new Contact("向锐龙", "https://www.baidu.com", "594663327@qq.com");
        return new ApiInfo("rlxiang's的SwaggerAPI文档",
                "加油",
                "1.0",
                "https://www.baidu.com",
                contact,
                "Apache 2.0",
                "http://www.apache.org/licenses/LICENSE-2.0",
                new ArrayList()
        );
    }
}

```



配置是否启动Swagger

```java
@Bean
public Docket getDocket(){
    return new Docket(DocumentationType.SWAGGER_2)
            .apiInfo(getApiInfo())
            .enable(false)// 是否启用swagger，如果为false，则swagger不能在浏览器中使用
            .select()
            .apis(RequestHandlerSelectors.basePackage("com.xrl.swagger"))
           // .paths(PathSelectors.ant("/xrl/**"))
            .build();
}
```

我只希望在我的swagger中在生成环境中使用，在发布的环境中不使用？

- 判断是否为生产环境 flag = false

- 注入enable(false)

  ```java
  @Bean
  public Docket getDocket(Environment environment){
      // 设置在浏览器中使用Swagger的环境，这里设置为 dev 与 test 两个环境
      Profiles profiles = Profiles.of("dev","test");
      // 通过environment.acceptsProfiles(profiles)判断是否处在Profiles中设置的环境里
      boolean flag = environment.acceptsProfiles(profiles);
      return new Docket(DocumentationType.SWAGGER_2)
              .apiInfo(getApiInfo())
              .enable(flag)// 是否启用swagger，如果为false，则swagger不能在浏览器中使用
              .select()
              .apis(RequestHandlerSelectors.basePackage("com.xrl.swagger"))
             // .paths(PathSelectors.ant("/xrl/**"))
              .build();
  }
  ```



配置API文档分组

```java
@Bean
public Docket getDocket(Environment environment){
    // 设置在浏览器中使用Swagger的环境，这里设置为 dev 与 test 两个环境
    Profiles profiles = Profiles.of("dev","test");
    // 通过environment.acceptsProfiles(profiles)判断是否处在Profiles中设置的环境里
    boolean flag = environment.acceptsProfiles(profiles);
    return new Docket(DocumentationType.SWAGGER_2)
            .apiInfo(getApiInfo())
            .groupName("xrl")// 配置分组
            .enable(flag)// 是否启用swagger，如果为false，则swagger不能在浏览器中使用
            .select()
            .apis(RequestHandlerSelectors.basePackage("com.xrl.swagger"))
           // .paths(PathSelectors.ant("/xrl/**"))
            .build();
}
```

如何配置多个文档组；设置多个Docket实例即可

```java
@Bean
public Docket getDocket1(){
    return new Docket(DocumentationType.SWAGGER_2).groupName("A");
}
@Bean
public Docket getDocket2(){
    return new Docket(DocumentationType.SWAGGER_2).groupName("B");
}
@Bean
public Docket getDocket3(){
    return new Docket(DocumentationType.SWAGGER_2).groupName("C");
}
```

测试

![image-20210726141309509](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210726141309509.png)



实体类配置

```java
package com.xrl.swagger.pojo;

import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.Data;

@Data
// @Api(注释信息)
@ApiModel("用户实体类")  // 给实体类加入一个文档注释
public class User {
    @ApiModelProperty("用户名") // 给实体类属性加注释
    private String username;
    @ApiModelProperty("密码")
    private String password;
}

```

Controller

```java
package com.xrl.swagger.controller;

import com.xrl.swagger.pojo.User;
import io.swagger.annotations.ApiOperation;
import io.swagger.annotations.ApiParam;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping(value = "/hello")
    public String hello(){

        return "hello";
    }

    // 只要我们的接口中的返回值存在实体类，就会被扫描到Swagger中
    @PostMapping(value = "/user")
    public User User(){

        return new User();
    }



    @ApiOperation("Hello控制类")
    @GetMapping("/hello2")
    public String hello2(@ApiParam("用户名") String username){
        // @ApiParam 给参数加注释
        return "hello" + username;
    }


    @ApiOperation("post测试类")
    @PostMapping("/postt")
    public User postt(@ApiParam("用户名") User user){
        // @ApiParam 给参数加注释
        return user;
    }

}

```

总结：

1. 我们可以通过Swagger给一些比较难理解的属性或接口，增加注释

2. 接口文档实时更新
3. 可以在线测试

Swagger是一个优秀的工具，几乎所有大公司都有使用

【注意点】 在正式发布的时候，关闭Swagger！！！！出于安全考虑。而且节省内存





## 任务

### 异步任务

实现异步任务步骤

1. 在调用异步任务的业务方法上加上@Async注解

   ```java
   @Service
   public class AsyncService {
   
       @Async
       public void hello(){
           try {
               Thread.sleep(3000);
           } catch (InterruptedException e) {
               e.printStackTrace();
           }
           System.out.println("数据正在处理");
       }
   }
   ```

2. 在主启动类上加上@EnableAsync注解

   ```java
   @EnableAsync // 开启异步注解功能
   @SpringBootApplication
   public class Springboot09TestApplication {
   
       public static void main(String[] args) {
           SpringApplication.run(Springboot09TestApplication.class, args);
       }
   
   }
   ```

3. 测试

   ```java
   package com.xrl.controller;
   
   import com.xrl.service.AsyncService;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;
   
   @RestController
   public class AsyncController {
       @Autowired
       AsyncService asyncService;
   
       @RequestMapping("/hello")
       public String hello(){
           asyncService.hello();
           return "ok";
       }
   }
   
   // 前端秒反应，后端等待3秒打印结果
   ```

定时任务

### 邮件发送

1. 导入依赖

   ```xml
   <!--邮件发送与接收依赖-->
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-mail</artifactId>
   </dependency>
   ```

2. 配置

   ```yml
   spring:
     mail:
       username: 594663327@qq.com
       password: rhynvekstscabfjd
       host: smtp.qq.com
       #    在qq邮箱中要开启加密授权验证
       properties:
         mail:
           smtp:
             ssl:
               enable: true
   ```

3. 测试方法

   ```java
   @Autowired
   JavaMailSenderImpl mailSender;
   
   // 一个简单的邮件发送
   @Test
   void contextLoads() {
       SimpleMailMessage mailMessage = new SimpleMailMessage();
       // 设置主题
       mailMessage.setSubject("小狂神你好呀");
       // 文本内容
       mailMessage.setText("谢谢你的java系列");
       // 发送到哪
       mailMessage.setTo("594663327@qq.com");
       // 邮件接收，邮件从哪来
       mailMessage.setFrom("594663327@qq.com");
       mailSender.send(mailMessage);
   
   }
   
   
   // 一个复杂的邮件发送
   @Test
   void contextLoads2() throws MessagingException {
       MimeMessage mimeMessage = mailSender.createMimeMessage();
       // 组装
       MimeMessageHelper helper = new MimeMessageHelper(mimeMessage,true);
       // 主题
       helper.setSubject("小狂神你好呀~plus");
       // 正文
       helper.setText("<p style='color:red'>谢谢你的java系列</p>",true);
       // 附件
       helper.addAttachment("bg.png",new File("C:\\Users\\dell\\Desktop\\bg.png"));
       helper.addAttachment("bg2.png",new File("C:\\Users\\dell\\Desktop\\bg.png"));
       // 发送到哪
       helper.setTo("594663327@qq.com");
       // 邮件接收，邮件从哪来
       helper.setFrom("594663327@qq.com");
       mailSender.send(mimeMessage);
   
   }
   
   // 封装邮件发送接收的方法
   
   /**
    *
    * @param html 邮件是否以html的样式显示
    * @param subject 主题
    * @param text 正文
    * @throws MessagingException
    */
   public void sendMail(Boolean html,String subject,String text) throws MessagingException {
       MimeMessage mimeMessage = mailSender.createMimeMessage();
       // 组装
       MimeMessageHelper helper = new MimeMessageHelper(mimeMessage,html);
       // 主题
       helper.setSubject(subject);
       // 正文
       helper.setText(text,true);
       // 附件
       helper.addAttachment("bg.png",new File("C:\\Users\\dell\\Desktop\\bg.png"));
       helper.addAttachment("bg2.png",new File("C:\\Users\\dell\\Desktop\\bg.png"));
       // 发送到哪
       helper.setTo("594663327@qq.com");
       // 邮件接收，邮件从哪来
       helper.setFrom("594663327@qq.com");
       mailSender.send(mimeMessage);
   }
   ```

### 定时任务

```java
// 两个核心接口
TaskExecutor 任务执行接口
TaskScheduler 任务调度接口

// 核心注解
@EnableScheduling // 开启定时功能的注解
@Scheduled   // 什么时候执行

Cron表达式
```

配置

```java
@SpringBootApplication
@EnableScheduling // 开启定时功能的注解
public class Springboot09TestApplication {

    public static void main(String[] args) {
        SpringApplication.run(Springboot09TestApplication.class, args);
    }

}
```

service

```java
package com.xrl.service;

import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Service;

@Service
public class ScheduledService {


    // 设置在一个特定的时间执行该方法
    // cron表达式，
    // 秒 分 时 日 月 周几
    /*
        cron = "30 54 19 * * ?" 表示每天的19:54:30执行该方法一次
        cron = "30 0/5 10,19 * * ?" 每天的10点或者19点每隔五分钟的到30s时执行一次

        0 15 10 ? * 1-6 每个月的周一到周六的10:15执行一次
     */
    @Scheduled(cron = "0/2 * * * * ?") // 没两秒执行该方法一次
    public void hello(){
        System.out.println("hello, 你被执行了");
    }
}
```

测试 只要启动程序即可测试

![image-20210726200858383](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210726200858383.png)

## 分布式 Dubbo + Zookeeper +SpringBoot

### 分布式系统理论

在《分布式系统原理与范型》一书中有如下定义:“分布式系统是若干独立计算机的集合，这些计算机对于用户来说就像单个相关系统”;



分布式系统是由一组通过网络进行通信、为了完成共同的任务而协调工作的计算机节点组成的系统。分布式系统的出现是为了用廉价的、普通的机器完成单个计算机无法完成的计算、存储任务。其目的是**利用更多的机器，处理更多的数据**。

分布式系统（distributed system）是建立在网络之上的软件系统。

首先需要明确的是，只有当单个节点的处理能力无法满足日益增长的计算、存储任务的时候，且硬件的提升（加内存、加磁盘、使用更好的CPU)高昂到得不偿失的时候，应用程序也不能进一步优化的时候，我们才需要考虑分布式系统。因为，分布式系统要解决的问题本身就是和单机系统一样的，而由于分布式系统参节点、通过网络通信的拓扑结构，会引入很多单机系统没有的问题，为了解决这些问题又会引入更多的机制、协议，带来更多的问题。。。



**Dubbo文档**

随着互联网的发展，网站应用的规模不断扩大，常规的垂直应用架构已无法应对，分布式服务架构以及流动计算架构势在必行，急需一个治理系统确保架构有条不紊的演进。

在Dubbo的官网文档有这样一张图

![image-20210726202418448](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210726202418448.png)



**什么是dubbo?**

Apache Dubbo 是一款高性能、轻量级的开源Java RPC框架，它提供了三大核心能力:面向接口的远程方法调用,智能容错和负载均衡,以及服务自动注册和发现。



### dubbo 基本概念

![image-20210726204634471](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210726204634471.png)

**服务提供者（Provider)**:暴露服务的服务提供方，服务提供者在启动时，向注册中心注册自己提供的服务。

**服务消费者(Consumer):**调用远程服务的服务消费方，服务消费者在启动时，向注册中心订阅自己所需的服务，服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。

**注册中心(Registry)**:注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者

**监控中心(Monitor）**:服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心



**调用关系说明**

- 服务容器负责启动，加载，运行服务提供者。
- 服务提供者在启动时，向注册中心注册自己提供的服务。I服务消费者在启动时，向注册中心订阅自己所需的服务。
- 注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。
- 服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
- 服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心



步骤

前提： Zookeeper的服务已开启

1. 提供者提供服务

   1. 导入依赖

      ```xml
      <!--导入Dubbo+ Zookeeper的包-->
      <!-- https://mvnrepository.com/artifact/org.apache.dubbo/dubbo-spring-boot-starter -->
      <dependency>
          <groupId>org.apache.dubbo</groupId>
          <artifactId>dubbo-spring-boot-starter</artifactId>
          <version>2.7.8</version>
      </dependency>
      
      <!--zkclient(zookeeper客户端包)-->
      <!-- https://mvnrepository.com/artifact/com.github.sgroschupf/zkclient -->
      <dependency>
          <groupId>com.github.sgroschupf</groupId>
          <artifactId>zkclient</artifactId>
          <version>0.1</version>
      </dependency>
      
      <!-- 引入Zookeeper服务端，要将slf4j排除，不然会和springboot的日志冲突-->
      <dependency>
          <groupId>org.apache.curator</groupId>
          <artifactId>curator-recipes</artifactId>
          <version>2.12.0</version>
      </dependency>
      
      <dependency>
          <groupId>org.apache.curator</groupId>
          <artifactId>curator-framework</artifactId>
          <version>2.12.0</version>
      </dependency>
      
      <!-- https://mvnrepository.com/artifact/org.apache.zookeeper/zookeeper -->
      <dependency>
          <groupId>org.apache.zookeeper</groupId>
          <artifactId>zookeeper</artifactId>
          <version>3.4.14</version>
          <exclusions>
              <exclusion>
                  <groupId>org.slf4j</groupId>
                  <artifactId>slf4j-log4j12</artifactId>
              </exclusion>
          </exclusions>
      </dependency>
      ```

   2. 配置注册中心的地址，以及服务发现名，和要扫描的包~

      ```yml
      # 服务应用名字
      dubbo:
        application:
          name: provider-service
      # 注册中心地址
        registry:
          address: zookeeper://127.0.0.1:2181
      # 注册那些服务
        scan:
          base-packages: com.xrl.service
      ```

   3. 在想要被注册的服务上面增加注解 @DubboService  @Component

      ```java
      import org.apache.dubbo.config.annotation.DubboService;
      import org.springframework.stereotype.Component;
      
      //zookeeper: 服务注册与发现
      @DubboService // 可以被Dubbo扫描到，当项目一启动就自动注册到注册中心
      @Component // 使用了Dubbo后，尽量不要使用Service注解
      public class TicketServiceImpl implements TicketService {
          @Override
          public String getTicket() {
              return "中国队加油";
          }
      }
      ```

2. 消费者如何消费

   1. 导入依赖

      ```xml
      <!--导入Dubbo+ Zookeeper的包-->
      <!-- https://mvnrepository.com/artifact/org.apache.dubbo/dubbo-spring-boot-starter -->
      <dependency>
          <groupId>org.apache.dubbo</groupId>
          <artifactId>dubbo-spring-boot-starter</artifactId>
          <version>2.7.8</version>
      </dependency>
      
      <!--zkclient(zookeeper客户端包)-->
      <!-- https://mvnrepository.com/artifact/com.github.sgroschupf/zkclient -->
      <dependency>
          <groupId>com.github.sgroschupf</groupId>
          <artifactId>zkclient</artifactId>
          <version>0.1</version>
      </dependency>
      
      <!-- 引入Zookeeper服务端，要将slf4j排除，不然会和springboot的日志冲突-->
      <dependency>
          <groupId>org.apache.curator</groupId>
          <artifactId>curator-recipes</artifactId>
          <version>2.12.0</version>
      </dependency>
      
      <dependency>
          <groupId>org.apache.curator</groupId>
          <artifactId>curator-framework</artifactId>
          <version>2.12.0</version>
      </dependency>
      
      <!-- https://mvnrepository.com/artifact/org.apache.zookeeper/zookeeper -->
      <dependency>
          <groupId>org.apache.zookeeper</groupId>
          <artifactId>zookeeper</artifactId>
          <version>3.4.14</version>
          <exclusions>
              <exclusion>
                  <groupId>org.slf4j</groupId>
                  <artifactId>slf4j-log4j12</artifactId>
              </exclusion>
          </exclusions>
      </dependency>
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-test</artifactId>
          <scope>test</scope>
      </dependency>
      ```

   2. 配置注册中心的地址，配置自己的服务名

      ```yml
      server:
        port: 8082
      
      # 消费者去那里拿服务，此时需要暴露自己的名字
      dubbo:
        application:
          name: conmuser-server
        # 注册中心的地址
        registry:
          address: zookeeper://127.0.0.1:2181
      ```

   3. 从远程注入服务

      ```java
      package com.xrl.service;
      
      import org.apache.dubbo.config.annotation.DubboReference;
      import org.springframework.stereotype.Service;
      
      @Service // 放到Service容器中
      public class Uservice {
      
          // 想拿到provider-service提供的票(服务),要去注册中心去拿
          @DubboReference // 引用，类似用Autowired，但是Autowired只能针对本地的服务
                          // 要想去拿到注册中心的服务，一般是要导入pom坐标，也可以在这建一个与注册中心一样的接口(包名类名都相同)
          TicketService ticketService;
      
          public void buyTicket(){
              String ticket = ticketService.getTicket();
              System.out.println("在注册中心拿到"+ticket);
          }
      }
      ```

