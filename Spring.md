## 1、Spring

### 1.1、简介

- Spring：春天 ----> 给软件行业带来了春天！
- 2002，首次退出了Spring框架的雏形：interface21框架
- Spring框架以interface21框架为基础，经过重新设计，并不断丰富其内涵，于2004年3月24号发布了1.0证实版本
- **Rod Johnson**的学历，真的让好多人大吃一惊，他是悉尼大学的博士，然而他的专业不是计算机，而是音乐学
- Spring的理念：是现有的技术更加容易使用，本身是一个大杂烩，整合了现有的技术框架

```xml
<!-- https://mvnrepository.com/artifact/org.springframework/spring-webmvc -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.2.0.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.2.0.RELEASE</version>
</dependency>
```

### 1.2、优点

- Spring是一个开源的免费的框架(容器)！
- Spring是一个轻量级的，非入侵式的框架！
- ==控制反转(Ioc),面向切面编程(AOP)==
- 支持事务的处理，对框架整合的支持！



==总结一句话：Spring就是一个轻量级的控制反转（IOC）和面向切面编程（AOP）的框架==



### 1.3、组成

![image-20210705143232671](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210705143232671.png)

### 1.4、拓展

在Spring的官网有这个介绍：现代化的java开发！说白了就是基于Spring的开发！

![image-20210705143627139](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210705143627139.png)

- Spring Boot
  - 一个快速开发的脚手架
  - 基于SpringBoot可以快速的开发单个微服务
  - 约定大于配置！
- Spring Cloud
  - SpringCloud是基于SpringBoot实现的

现在大多数公司都在使用SpringBoot进行快速开发，学习SpringBoot的前提，需要完全掌握Spring及SpringMVC！



**弊端：发展了太久之后，违背了原来额历练！配置十分繁琐。人称：“配置地狱”**





## 2、IOC理论推导

1. UserDao 接口
2. UserDaoImpl 实现类
3. UserService 接口
4. UserServiceImpl 实现类





在以前的业务中，用户的需求可能会影响我们原来的代码，我们需要根据用户的需求去修改代码！如果程序代码量十分大，修改一次的成本代价十分昂过



**以前的代码**

```java
import com.xrl.dao.UserDao;
import com.xrl.dao.UserDaoImpl;
import com.xrl.dao.UserDaoMysqlImpl;
import com.xrl.dao.UserDaoOracleImpl;

public class UserServiceImpl implements UserService {
    private UserDao userDao = new UserDaoMysqlImpl();

    public void getUser() {
        userDao.getUser();
    }
}
如果这样写了之后，每增加一个Dao都要改service层的代码，这样十分麻烦
```

**对应的测试代码**

```java
public class MyTest {

    public static void main(String[] args) {

        // 用户实际调用的是业务层，dao层他们不需要接触！
        // 这里就对应了service的具体实现，如果UserServiceImpl里的是
        // private UserDao userDao = new UserDaoMysqlImpl();这样的写法，则增加一个dao我们就要修改UserServiceImpl里的代码。
        UserServiceImpl userService = new UserServiceImpl();
		
        userService.getUser();
    }
}
```

**改进**

```java
// 在UserServiceImpl里加入set方法
public class UserServiceImpl implements UserService {
    private UserDao userDao;

    // 利用set进行东条实现UserDao实现类的注入
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    public void getUser() {
        userDao.getUser();
    }
}
```

**改进后的测试**

```java
public class MyTest {

    public static void main(String[] args) {

        // 用户实际调用的是业务层，dao层他们不需要接触！
        UserServiceImpl userService = new UserServiceImpl();
        // 这样无论是哪个Dao的实现，我们都只用在这去new对象，不用改变UserServiceImpl里的代码
        userService.setUserDao(new UserDaoMysqlImpl());
        userService.getUser();
    }
}
```

- 之前，程序是主动创建对象！控制权在开发人员手中
- 使用set注入后，程序不在具有主动性，而是变成被动的接受对象！

这种思想，从本质上解决了问题，开发人员再也不用去管理对象的创建了。系统的耦合性大大降低~，可以更加专注于在业务上的实现上！这是IOC的原型！



### IOC本质

**控制反转loC(Inversion of Control)，是一种设计思想，DI(依赖注入)是实现loC的一种方法**，也有人认为DI只是loC的另一种说法。没有loC的程序中，我们使用面向对象编程，对象的创建与对象间的依赖关系完全硬编码在程序中，对象的创建由程序自己控制，控制反转后将对象的创建转移给第三方，个人认为所谓控制反转就是:获得依赖对象的方式反转了。

采用XML方式配置Bean的时候，Bean的定义信息是和实现分离的，而采用注解的方式可以把两者合为一体，Bean的定义信息直接以注解的形式定义在实现类中，从而达到了零配置的目的。



**控制反转是一种通过描述(XML或注解）并通过第三方去生产或获取特定对象的方式。在Spring中实现控制反转的是loC容器，其实现方法是依赖注入(Dependency Injection,Dl)。**



## 3、HelloSpring

**beans.xml(applicationContext.xml)**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--使用Spring创建对象，在Spring中都称这些为bean
        正常java代码创建对象
           类型 变量名 = new 类型();
           Hello hello = new Hello();

           这里的就是
           id = 变量名
           class = new 的对象；
           property 相当于给对象中的属性设置一个值！
    -->
    <bean id="hello" class="com.xrl.pojo.Hello">
        <property name="str" value="Spring"/>
    </bean>
</beans>
```

```java
package com.xrl.pojo;

public class Hello {

    private String str;

    public String getStr() {
        return str;
    }

    public void setStr(String str) {
        this.str = str;
    }

    @Override
    public String toString() {
        return "Hello{" +
                "str='" + str + '\'' +
                '}';
    }
}
```

```java
public class MyTest {
    public static void main(String[] args) {
        // 获取Spring的上下文对象！
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");

        // 我们的对象现在都在Spring中管理了，我们要使用，直接去里面取出来就可以了

        Hello hello = (Hello) context.getBean("hello");
        System.out.println(hello.toString());


    }
}
```

**思考问题?**

- Hello 对象是谁创建的?

  hello 对象是由Spring创建的

- Hello 对象的属性是怎么设置的?

  hello对象的属性是由Spring容器设置的,

这个过程就叫控制反转:

控制:谁来控制对象的创建，传统应用程序的对象是由程序本身控制创建的，使用Spring后，对象是由Spring来创建的．

反转:程序本身不创建对象，而变成被动的接收对象.依赖注入:就是利用set方法来进行注入的.

IOC是一种编程思想，由主动的编程变成被动的接收．

可以通过newClassPathXmlApplicationContext去浏览一下底层源码.

**OK，到了现在，我们彻底不用再程序中去改动了，要实现不同的操作，只需要在xml配置文件中进行修改，所谓的loC,一句话搞定:对象由Spring来创建，管理，装配!**



## 4、IOC创建对象的方式

1. 使用无参构造创建对象，这也是默认方式

   ```xml
   <!--按有参构造的参数的索引位置创建对象-->
   <bean id="user" class="com.xrl.pojo.User"/>
   ```

2. 使用有参构造创建对象

   1. 下标索引赋值

      ```xml
      <!--按有参构造的参数的索引位置创建对象-->
      <bean id="user" class="com.xrl.pojo.User">
          <constructor-arg index="0" value="李四"/>
      </bean>
      ```

   2. 参数类型赋值

      ```xml
      <!--按照有参构造的参数类型创建对象，不建议使用-->
      <bean id="user" class="com.xrl.pojo.User">
          <constructor-arg type="java.lang.String" value="王五"/>
      </bean>
      ```

   3. 有参构造的参数名来构造对象

      ```xml
      <!--直接通过有参构造的参数名来创建对象-->
      <bean id="user" class="com.xrl.pojo.User">
          <constructor-arg name="name" value="王五"/>
      </bean>
      ```

   

   **Spring 在我们注册bean时(applicationContext里的bean),就将对象创建好了，不管我们是否调用他们都在容器中了。**

   ```xml
   // 创建bean
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
           https://www.springframework.org/schema/beans/spring-beans.xsd">
       <!--直接通过有参构造的参数名来创建对象-->
       <bean id="user" class="com.xrl.pojo.User">
           <constructor-arg name="name" value="王五"/>
       </bean>
   
       <bean id="usert" class="com.xrl.pojo.Usert"/>
   
       <alias name="user" alias="userNew"/>
   </beans>
   ```

   ```java
   public class User {
   
       private String name;
   
       public User(String name) {
           this.name = name;
       }
   }
   public class Usert {
   
       private String name;
   
       public Usert() {
           System.out.println("Usert被创建了");
       }
   }
   // 测试
      public static void main(String[] args) {
           ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
   
           User user = (User) context.getBean("user");
   
           user.show();
       }
   // 控制台输出
   Usert被创建了
   name=王五
   ```

   

   

   总结：在配置文件加载的时候，容器中管理的对象就已经初始化了！



## 5、Spring配置



### 5.1、别名

```xml
<!--用了别名，我们在获取bean时可以使用原来的id也可以使用这里的别名
        给user起个别名userNew
    -->
<alias name="user" alias="userNew"/>
```

### 5.2、Bean的配置

```xml
<!--

    id: bean的唯一标识符，也就相当于new对象时的对象名
    class：bean 对象所对应的全限定名
    name：也是别名,而且name 可以同时去多个别名
-->

<bean id="usert" class="com.xrl.pojo.Usert" name="user2,u2">
    <property name="name" value="张某"/>
</bean>
```

### 5.3、import

这个import，一般用于团队开发使用，可以将多个配置文件导入合并成一个

假设，现在项目中有多个人开发，这三个人负责不同的类开发，不同的类需要注册在不同的bean中，我么可以利用iport将其所有人的beans.xml文件合并为一个总的

- 张三

- 李四

- 王五

- applicationContext.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
          https://www.springframework.org/schema/beans/spring-beans.xsd">
  
      <import resource="beans.xml"/>
      <import resource="beans2.xml"/>
      <import resource="beans3.xml"/>
  
  </beans>
  ```





## 6、依赖注入



### 6.1、构造器注入

在IOC创建对象的方式那已经说明了



### 6.2、Set方式注入【重点】

- 依赖注入：利用set方法去注入(Set注入)！
  - 依赖：bean对象的创建依赖于容器
  - 注入：bean对象中的所有属性，由容器来注入！

【环境搭建】

1. 复杂类型

   ```java
   public class Address {
   
       private String address;
   
       public String getAddress() {
           return address;
       }
   
       public void setAddress(String address) {
           this.address = address;
       }
   }
   ```

   

2. 真实测试对象

   ```java
   public class Student {
   
       private String name;
       private Address address;
       private String[] books;
       private List<String> hobbys;
       private Map<String,String> card;
       private Set<String> games;
       private String wife;
       private Properties info;
   }
   ```

3. beans.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
           https://www.springframework.org/schema/beans/spring-beans.xsd">
   
       <bean id="student" class="com.xrl.pojo.Student">
           <!--第一种，普通值注入，value
               name为变量名
           -->
           <property name="name" value="rlxiang"/>
       </bean>
   </beans>
   ```

4. 测试类

   ```java
   public class MyTest {
       public static void main(String[] args) {
           ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
           Student student = (Student) context.getBean("student");
           System.out.println(student.getName());
       }
   }
   ```



完善注入信息

```xml
<bean id="address" class="com.xrl.pojo.Address">
    <property name="address" value="武汉"/>
</bean>
<bean id="student" class="com.xrl.pojo.Student">
    <!--第一种，普通值注入，value
        name为变量名
    -->
    <property name="name" value="rlxiang"/>

    <!--第二种，Bean注入 ref-->
    <property name="address" ref="address"/>

    <!--第三种注入，数组注入 array-->
    <property name="books">
        <array>
            <value>红楼梦</value>
            <value>西游记</value>
            <value>水浒传</value>
            <value>三国演义</value>
        </array>
    </property>

    <!--第四种 list注入-->
    <property name="hobbys">
        <list>
            <value>篮球</value>
            <value>游泳</value>
            <value>coding</value>
            <value>看电影</value>
        </list>
    </property>

    <!--第五种 map注入-->
    <property name="card">
        <map>
            <entry key="身份证" value="12434563452342345"/>
            <entry key="银行卡" value="324354356234523"/>
        </map>
    </property>

    <!--第六种 set注入-->
    <property name="games">
        <set>
            <value>王者荣耀</value>
            <value>英雄联盟</value>
        </set>
    </property>

    <!--null值注入-->
    <property name="wife">
        <null/>
    </property>

    <!--
        properties注入
    -->
    <property name="info">
        <props>
            <prop key="driver">2019113152</prop>
            <prop key="url">男</prop>
            <prop key="username">root</prop>
            <prop key="password">root</prop>
        </props>
    </property>
 </bean>
```

测试输出

```java
Student{
                 name='rlxiang',
                 address=Address{address='武汉'},
                 books=[红楼梦, 西游记, 水浒传, 三国演义],
                 hobbys=[篮球, 游泳, coding, 看电影],
                 card=
                    {
                        身份证=12434563452342345, 
                        银行卡=324354356234523
                        },
                 games=[王者荣耀, 英雄联盟],
                 wife='null',
                 info={password=root, 
                        url=男, 
                        driver=2019113152, 
                        username=root}
           }
```





### 6.3、拓展方式注入

我们可以使用p命名空间和c命名空间进行注入

[官方解释：](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-p-namespace)

![image-20210706152746523](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210706152746523.png)

使用

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:c="http://www.springframework.org/schema/c"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--p命名空间注入，可以直接注入属性的值：相当于property-->
    <bean id="user" class="com.xrl.pojo.User" p:name="小明" p:age="18"/>

    <!--c命名空间注入，可以通过有参构造器注入：相当于construct-arg-->
    <bean id="user2" class="com.xrl.pojo.User" c:name="小宏" c:age="18"/>

</beans>
```

测试

```java
public class MyTest {
    @Test
    public void test2(){
        ApplicationContext context = new ClassPathXmlApplicationContext("userBeans.xml");
        User user =  context.getBean("user2",User.class);
        System.out.println(user);
    }
}
```



注意点：p命名空间和c命名空间不能直接使用，需要导入xml约束！

```xml
xmlns:p="http://www.springframework.org/schema/p"
xmlns:c="http://www.springframework.org/schema/c"
```



### 6.4、bean的作用域

![image-20210706153939017](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210706153939017.png)

1. 单例模式(Spring默认机制)

   ```xml
   <bean id="user2" class="com.xrl.pojo.User" c:name="小宏" c:age="18" scope="singleton"/>
   ```

2. 原型模式：每次从容器中getBean时，都会产生一个新对象！

   ```xml
   <bean id="user2" class="com.xrl.pojo.User" c:name="小宏" c:age="18" scope="prototype"/>
   ```

3. 其余的 request、session、application、这些个只能在web开发中使用



## 7、Bean的自动装配

- 自动装配是Spring满足bean依赖的一种方式！
- Spring会在上下文中自动虚招，并自动给bean装配属性！



在Spring中有三种装配方式

1. 在xml中显示的配置
2. 在java中显示的配置(javaConfig)
3. 隐士的自动装配bean【重要】



### 7.1、测试

环境搭建：一个人有两个宠物



### 7.2、ByName自动装配

```xml
<!--
        byName: 会自动在容器上下文中去查找与自己对象set方法后面的属性对应的 beanid
    -->
<bean id="people" class="com.xrl.pojo.People" autowire="byName">
    <property name="name" value="小红"/>
</bean>
```

### 7.3、ByType自动装配

```xml
<bean id="cat" class="com.xrl.pojo.Cat"/>
<bean  class="com.xrl.pojo.Dog"/>

<!--
        byName: 会自动在容器上下文中去查找与自己对象set方法后面的属性对应的 beanid
        byType: 会自动在容器上下文中去查找与自己对象属性类型相同的bean
    -->
<bean id="people" class="com.xrl.pojo.People" autowire="byType">
    <property name="name" value="小红"/>
</bean>
```

小结：

- byName时，需要保证所有bean的id唯一，并且这个bean需要和自动注入的属性的set方法后面的属性一致
- byType时，需要保证所有bean的class唯一，并且这个bean需要和自动注入的属性类型一致

### 7.4、使用注解实现自动装配

Spring2.5开始支持注解

要使用注解须知：

1. 导入约束
2. ==配置注解的支持: <context:annotation-config/>==

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

</beans>
```

**@Autowired**

直接在属性上使用即可！也可以在set方法上使用！

使用Autowired，我们可以不用编写Set方法了，前提是你这个自动装配的属性在IOC容器中存在。 

科普：

```java
@Nullable  字段如果标注了这个注解，表示这个字段可以为null，但是如果传值了就是传入的值

public void setName(@Nullable String name) {
        this.name = name;
    }
```

```java
public @interface Autowired {

	boolean required() default true;

}
```

测试

```java
public class People {

    // 如果显示定义了Autowired的required属性为false，说明这个对象可以为null，否则不允许为空
    @Autowired(required = false)
    private Cat cat;

    @Autowired
    private Dog dog;

    private String name;
}
```



如果@Autowired自动装配环境比较复杂，自动装配无法通过一个注解【@Autowired】完成的时候，我们可以使用@Qualifier(value="xxx")来指定唯一的bean对象注入！

```java
public class People {

    @Autowired
    @Qualifier(value="cat2")
    private Cat cat;

    @Autowired
    @Qualifier(value="dog2") // 显示的指定注入的对象
    private Dog dog;

    private String name;
}


beans.xml
    <bean id="cat1" class="com.xrl.pojo.Cat"/>
    <bean id="cat2" class="com.xrl.pojo.Cat"/>
    <bean id="dog1" class="com.xrl.pojo.Dog"/>
    <bean id="dog2" class="com.xrl.pojo.Dog"/>
```

**@Resource注解**

```java
import javax.annotation.Resource;

public class People {

    @Resource(name="cat1") // 指定自动装配的bean，参数为对应的bean的id
    private Cat cat;

    @Resource(name="dog1")
    private Dog dog;
    private String name;
```

小结

@Resource 和 @Autowired的区别：

- 都是用来自动装配的，都可以放在属性字段上
- @Autowired 默认通过byType的方式实现，而且必须要求这个对象存在！如果Autowired 不能进行唯一的自动装配时，则需要通过@Qualifier(value="指定装配的bean的id")来指定唯一自动装配的bean！【常用】
- @Resource 默认通过byName的方式实现，如果找不到名字，则通过byName实现！如果两个都找不到就报错！【常用】
- 执行顺序不同：@Autowired 通过byType的方式实现。@Resource 默认通过byName的方式实现



### 8、使用注解开发



在Spring4之后，要使用注解开发，必须保证aop的包已经导入

![image-20210706194024365](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210706194024365.png)

​	使用注解需要导入context约束，增加注解的支持！

1. bean

2. 属性如何注入

   ```java
   @Component
   public class User {
       public String name;
        // 相当于 给bean里的属性赋值<property name="name" value="小红"
       @Value("小红") // @Value注解也可以写在属性上
       public void setName(String name) {
           this.name = name;
       }
   ```

3. 衍生的注解

   @Component有几个衍生注解，我们在web开发中，按照mvc三层架构分层！

   - dao 【@Repository】

   - service 【@Service】

   - controller 【@Controller】

     ​	这四个注解功能都是一样的，都是代表将某个类注册到Spring中，装配bean

4. 自动装配

   - @Autowired ：自动装配通过类型，名字

     ​		如果Autowired 不能唯一自动装配上属性，则需要通过@Qualifier(value="指定的装配的bean的id")

   - @Nullable ： 字段标记了这个注解，说明该字段可以为null

   - @Resources：   自动装配通过名字，类型。如果找不到名字，则通过byName实现！如果两个都找不到就报错

5. 作用域(@Scope)

   ```java
   @Component
   @Scope("singleton")
   public class User {
   
       // 相当于 给bean里的属性赋值<property name="name" value="小红"
   
       public String name;
   
       @Value("小红")
       public void setName(String name) {
           this.name = name;
       }
   }
   ```

6. 小结

   xml与注解：

   - xml更加万能，适合于任何场合！维护简单方便
   - 注解 不是自己的类使用不了，维护相对复杂！

   xml与注解最佳实践

   - xml用来管理bean；
   - 注解只负责完成属性的注入
   - 我们在使用的过程中，只需要注意一个问题：必须让注解生效，就需要开启注解的支持

   ```xml
   <!--包扫描，指定要扫描的包，这个包下的注解才会生效-->
   <context:component-scan base-package="com.xrl"/>
   <context:annotation-config/>
   ```



## 9、使用Java的方式配置Spring

我们要完全不使用Spring的xml配置了，全权交给java来做！

JavaConfig是Spring的一个子项目，在Spring4之后，他就成为了一个核心功能！

![image-20210706202851305](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210706202851305.png)



**实体类**

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class User {


    private String name;

    public String getName() {
        return name;
    }

    @Value("小明")
    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                '}';
    }
}

```

**配置类（可以做配置文件所做的所有的事）**

```java
//使用@Configuration 标注的类表示这是一个配置类，也会被Spring托管，注册到容器中，它本身也是被@Component标注了的
// @Configuration 标注的类就和applicationContext.xml一样，可以做applicationContext.xml中做的所有的事
@Configuration
@ComponentScan("com.xrl.pojo")
@Import(MyConfig2.class) // 相当于applicationContext.xml中的导入其他的beans.xml
public class MyConfig {

    // 注册一个bean，就相当于我们之前写的一个bean标签
    // 这个方法的名字就相当于bean标签中的id属性
    // 这个方法的返回值，就相当于bean中的class属性
    @Bean
    public User user(){
        return new User(); // 就是返回注册到bean中的对象！
    }
}
```

**测试类**

```java

public class MyTest {
    public static void main(String[] args) {
        // 如果完全使用配置类的方式去做，我们就只能通过AnnotationConfig 上下文来获取容器，通过配置类的class对象来加载！
        ApplicationContext context = new AnnotationConfigApplicationContext(MyConfig.class);
        User user = (User) context.getBean("user");
        System.out.println(user.getName());
    }
}
```





## 10、代理模式

代理模式的分类：

- 静态代理
- 动态代理

![image-20210706213532259](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210706213532259.png)

### 10.1、静态代理

角色分析：

- 抽象角色：一般会使用接口或者抽象类来解决
- 真实角色：被代理的角色
- 代理角色：代理真实角色，代理真实角色后，我们一般会做一些附属操作
- 客户：访问代理对象的人！



代码步骤：

1. 接口

   ```java
   // 出租房
   public interface Rent {
   
       void rent();
   }
   ```

2. 真实角色

   ```java
   // 房东
   public class Host implements Rent{
       public void rent() {
           System.out.println("房东要出租房子");
       }
   }
   ```

3. 代理角色

   ```java
   package com.xrl.demo01;
   
   public class Proxy implements Rent{
   
       private Host host;
   
       public Proxy() {
       }
   
       public Proxy(Host host) {
           this.host = host;
       }
   
       public void rent() {
           seeHouse();
           host.rent();
           heTong();
           fare();
       }
   
       // 看房
       public void seeHouse(){
           System.out.println("中介带你看房");
       }
   
       // 收中介费
       public void fare(){
           System.out.println("收中介费");
       }
   
       // 签合同
       public void heTong(){
           System.out.println("签租赁合同");
       }
   
   }
   ```

4. 客户端访问代理角色

   ```java
   public class Client {
       public static void main(String[] args) {
           // 房东要出租房子
           Host host = new Host();
   
           // 代理， 中介帮房东出租房子， 但是代理角色一般会有一些附属操作
           Proxy proxy = new Proxy(host);
   
           // 直接找中介租房
           proxy.rent();
       }
   }
   ```

   



代理模式的好处：

- 可以使真实角色的操作更加纯粹！不用去关注一些公共的业务
- 公共业务就交给了代理角色！实现了业务的分工！
- 公共业务发生扩展的时候，方便集中管理

缺点：

- 一个真实角色就会产生一个代理角色；代码量会翻倍 开发效率会变低



### 10.2、加深静态代理的理解

spring-study

​			- spring-08-proxy

​				 - demo02

在实际的开发中，一般需要增加业务时(即附加操作)，是不会改变原有代码的，防止系统崩盘，于是就要进行横向操作代码完成附加的操作，即使用代理模式，这就是AOP的思想

![image-20210706221911989](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210706221911989.png)



10.3、动态代理

- 动态代理和静态代理 角色是一样的
- 动态代理的代理类是动态生成的，不是我们直接写好的！
- 动态代理分为两大类：基于接口的动态代理，基于类的动态代理
  - 基于接口 -- JDK动态代理
  - 基于类：cglib
  - java字节码实现： javasist  【使用的JBoss服务器上】





需要了解两个类：Proxy：代理，InvocationHandler：调用处理程序



动态代理的好处：

- 可以使真实角色的操作更加纯粹！不用去关注一些公共的业务
- 公共业务就交给了代理角色！实现了业务的分工！
- 公共业务发生扩展的时候，方便集中管理
- 一个动态代理类代理的是一个接口，一般就是对应的一类业务
- 一个动态代理可以代理多个类，因为代理对象变成动态的，所以再讲代理对象实现与真实角色相同的接口时，也可不用再去写静态代理那的将代理对象实现接口的代码了，只需要传入真实角色对象的信息即可生成实现了与真实角色相同接口的代理对象。

```java
// 真实角色与代理角色都要实现的接口
// 出租房
public interface Rent {

    void rent();
}
```

```java
// 房东(真实角色)
public class Host implements Rent {
    public void rent() {
        System.out.println("房东要出租房子");


    }
}
```

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

// 生成代理对象的类
public class ProxyInvocationHandler implements InvocationHandler {

    // 被代理的接口
    private Object target;

    public void setHost(Object target) {
        this.target = target;
    }


    // 生成代理类
    public Object getProxy(){
        return Proxy.newProxyInstance(this.getClass().getClassLoader(),target.getClass().getInterfaces(),this);
    }

    // 处理代理实例，并返回结果
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        log(method.getName());
        // 动态代理的本质就是利用反射
        Object result = method.invoke(target, args);
        return result;
    }

    public void log(String msg){
        System.out.println("执行了"+msg+"方法");
    }
}
```

```java
public class Client {
    public static void main(String[] args) {

        // 真实角色
        Host host = new Host();

        // 代理角色，现在没有
        ProxyInvocationHandler pih = new ProxyInvocationHandler();
        // 通过代理角色的处理程序来将代理角色实现与真实角色同样的接口
        pih.setHost(host);
        // 生成代理角色
        Rent proxy = (Rent) pih.getProxy();

        proxy.rent();
    }
}
```

```java
// 客户端（动态代理的第二种写法，不去显示创建InvocationHandler的实现类）
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class Client2 {
    public static void main(String[] args) {
        final Marry userService = new MouMarry();
        // 体现了动态性，将代理对象实现接口的过程变成利用参数传递
        Marry proxy = (Marry) Proxy.newProxyInstance(userService.getClass().getClassLoader(), userService.getClass().getInterfaces(), new InvocationHandler() {
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                System.out.println("执行了" + method.getName() + "方法");
                Object result = method.invoke(userService, args);
                return result;
            }
        });

        proxy.expression();
    }

}
```



## 11、AOP

### 11.1、什么是AOP

AOP (Aspect Oriented Programming)意为:面向切面编程，通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。IAOP是OOP的延续，是软件开发中的一个热点，也是Spring框架中的一个重要内容，是函数式编程的一种衍生范型。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。

![image-20210707150429224](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210707150429224.png)

### 11.2、Aop在Spring中的作用

- 横切关注点:跨越应用程序多个模块的方法或功能。即是，与我们业务逻辑无关的，但是我们需要关注的部分，就是横切关注点。如日志,安全，缓存，事务等等....

- 切面(ASPECT)∶横切关注点被模块化的特殊对象。即，它是一个类。

- 通知(Advice):切面必须要完成的工作。即，它是类中的一个方法。

- 目标(Target):被通知对象。

- 代理(Proxy):向目标对象应用通知之后创建的对象。

- 切入点(PointCut) :切面通知执行的“"地点""的定义。

- 连接点(JointPoint):与切入点匹配的执行点。

  ​					![image-20210707150743260](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210707150743260.png)

SpringAOP中，通过Advice定义横切逻辑，Spring中支持5中类型的Advice；

![image-20210707151212543](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210707151212543.png)

即 Aop 在不改变原有代码的情况下，去增加新的功能。



### 11.3 使用Spring实现Aop

【重点】 使用AOP织入，需要导入一个依赖包！

```xml
<!-- https://mvnrepository.com/artifact/org.aspectj/aspectjweaver -->
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.4</version>
</dependency>
```







#### 方式一：使用Spring的API接口【主要是SpringAPI接口实现】

1. 实现SpringAPI中的接口

   ```java
   import org.springframework.aop.AfterReturningAdvice;
   
   import java.lang.reflect.Method;
   
   public class AfterLog implements AfterReturningAdvice {
       /**
        *
        * @param returnValue 返回值
        * @param method 被执行的方法对象
        * @param args 参数
        * @param target 目标对象
        * @throws Throwable
        */
       public void afterReturning(Object returnValue, Method method, Object[] args, Object target) throws Throwable {
           System.out.println("执行了"+method.getName()+"方法，返回结果为"+returnValue);
   
       }
   }
   ```

   ```java
   import org.springframework.aop.MethodBeforeAdvice;
   
   import java.lang.reflect.Method;
   
   public class Log implements MethodBeforeAdvice {
       /**
        *
        * @param method 要执行的目标方法
        * @param args 参数
        * @param target 目标对象
        * @throws Throwable
        */
       public void before(Method method, Object[] args, Object target) throws Throwable {
           System.out.println(target.getClass().getName()+"的"+method.getName()+"被执行了");
       }
   }
   ```

2. applicationContext.xml中配置aop

   ```xml
   <!--方式一：使用原生Spring的API-->
   <!--配置aop,需要导入aop约束-->
   <aop:config>
       <!--切入点 expression:表达式，execution(要执行的位置)-->
       <aop:pointcut id="pointCut" expression="execution(* com.xrl.service.UserServiceImpl.*(..))"/>
       <!--执行环绕增强-->
       <aop:advisor advice-ref="log" pointcut-ref="pointCut"/>
       <aop:advisor advice-ref="afterLog" pointcut-ref="pointCut"/>
   </aop:config>
   ```

3. 测试

   ```java
   public class MyTest {
       public static void main(String[] args) {
   
           // 动态代理代理的是接口
           ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
           UserService bean = context.getBean("userService", UserService.class);
           bean.add();
       }
   }
   ```

   

#### 方式二：自定义类实现AOP【主要是切面定义】

1. 自定义类

   ```java
   public class DiyPointCut {
       public void before(){
           System.out.println("========方法执行前=========");
       }
   
       public void after(){
           System.out.println("========方法执行后==========");
       }
   }
   
   ```

2. applicationContext.xml中配置

   ```xml
   <bean id="diy" class="com.xrl.diy.DiyPointCut"/>
   <aop:config>
       <!--自定义切面，ref 表示要引用的类-->
       <aop:aspect ref="diy">
           <!--切点-->
           <aop:pointcut id="point" expression="execution(* com.xrl.service.UserServiceImpl.*(..))"/>
           <!--通知
                   method: 值为我们自定义的方法名
               -->
           <aop:before method="before" pointcut-ref="point"/>
           <aop:after method="after" pointcut-ref="point"/>
       </aop:aspect>
   </aop:config>
   ```

3. 测试

   ```java
   public class MyTest {
       public static void main(String[] args) {
   
           // 动态代理代理的是接口
           ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
           UserService bean = context.getBean("userService", UserService.class);
           bean.add();
       }
   }
   ```

#### 方式三：使用注解方式实现Aop

1. 自定义一个切面(就是一个类，然后标记@Aspect注解)

   ```java
   import org.aspectj.lang.ProceedingJoinPoint;
   import org.aspectj.lang.Signature;
   import org.aspectj.lang.annotation.*;
   
   @Aspect // 标注该类为一个切面
   public class AnnotationPointcut {
   
       @Before("execution(* com.xrl.service.UserServiceImpl.*(..))")
       public void before(){
           System.out.println("========方法执行前=========");
       }
   
       @After("expression()")
       public void after(){
           System.out.println("========方法执行后==========");
       }
   
       // 在环绕增强中。我们可以定义一个参数，代表我们要获取处理切入的点
       @Around("expression()")
       public void around(ProceedingJoinPoint jp) throws Throwable {
   
           Signature signature = jp.getSignature(); // 获取签名
           System.out.println("signature=" + signature); // void com.xrl.service.UserService.add()
   
           System.out.println(signature.getDeclaringTypeName()); // 获取方法所在的类 com.xrl.service.UserService
           System.out.println(signature.getName()); // 方法名 add
           System.out.println("环绕前");
           // 方法执行
           Object proceed = jp.proceed();
           System.out.println("环绕后");
       }
   
       // 抽取切点表达式
       @Pointcut("execution(* com.xrl.service.UserServiceImpl.*(..))")
       public void expression(){
   
       }
   }
   ```

2. 在applicationContext.xml文件中注册

   ```xml
   <bean id="annotationPointCut" class="com.xrl.diy.AnnotationPointcut"/>
   <!--开启aop注解支持 JDK(默认实现 proxy-target-class="false") cglib（proxy-target-class="true"）-->
   <aop:aspectj-autoproxy/>
   ```

   



## 12、整合Mybatis

步骤：

 	1. 导入相关jar包
 	 - junit
 	 - mybatis
 	 - mysql数据库
 	 - spring相关的
 	 - aop织入
 	 - mybatis-spring【新知识点】
 	2. 编写配置文件
 	3. 测试



### 12.1、回忆mybatis

1. 编写实体类
2. 编写核心配置文件
3. 编写借口
4. 编写Mapper.xml
5. 测试





### 12.2、Mybatis-spring

1. 编写数据源配置

2. SqlSessionFactory

3. SqlSessionTemplate

   ​	前三步都是在配置文件中完成

   ```xml
   // mybatis核心配置
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE configuration
           PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-config.dtd">
   <configuration>
       <typeAliases>
           <package name="com.xrl.pojo"/>
       </typeAliases>
   
   </configuration>
   ```

   ```xml
   //spring接管mybatis的配置
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:aop="http://www.springframework.org/schema/aop"
          xsi:schemaLocation="
          http://www.springframework.org/schema/beans
           https://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/aop
           https://www.springframework.org/schema/aop/spring-aop.xsd
   ">
   
       <!--
           DataSource: 使用spring的数据源替换Mybatis的配置  c3p0 dbcp druid
           我们这里使用Spring提供的JDBC：org.springframework.jdbc.datasource
       -->
       <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
           <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
           <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;userUnicode=true&amp;characterEncoding=utf-8&amp;serverTimezone=UTC"/>
           <property name="username" value="root"/>
           <property name="password" value="root"/>
       </bean>
   
       <!--配置sqlSessionFactory-->
       <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
           <property name="dataSource" ref="dataSource" />
           <!--绑定mybatis配置文件-->
           <property name="configLocation" value="classpath:mybatis-config.xml"/>
           <property name="mapperLocations" value="classpath:com/xrl/mapper/*.xml"/>
       </bean>
   
       <!--SqlSessionTemplate:就是我们使用的sqlSession-->
       <bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
           <!--只能使用有参构造注入，因为SqlSessionTemplate没有set方法-->
           <constructor-arg name="sqlSessionFactory" ref="sqlSessionFactory"/>
       </bean>
   
   </beans>
   ```

   ```xml
   // 整合前面两个配置
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="
          http://www.springframework.org/schema/beans
           https://www.springframework.org/schema/beans/spring-beans.xsd
   ">
       <import resource="spring-mapper.xml"/>
   
       <bean id="userMapper" class="com.xrl.mapper.UserMapperImpl">
           <property name="sqlSession" ref="sqlSession"/>
       </bean>
   
       <bean id="userMapper2" class="com.xrl.mapper.UserMapperImpl2">
           <property name="sqlSessionFactory" ref="sqlSessionFactory"/>
       </bean>
   </beans>
   ```

   

4. 给mapper接口加实现类

   ```java
   package com.xrl.mapper;
   
   import com.xrl.pojo.User;
   import org.mybatis.spring.SqlSessionTemplate;
   
   import java.util.List;
   
   public class UserMapperImpl implements UserMapper {
   
       // 以前我们读数据库的所有操作都是使用SqlSession，但是被Spring接管后，使用的是SqlSessionTemplate
       private SqlSessionTemplate sqlSession;
   
       public void setSqlSession(SqlSessionTemplate sqlSession) {
           this.sqlSession = sqlSession;
       }
   
       public List<User> selectUser() {
           UserMapper mapper = sqlSession.getMapper(UserMapper.class);
           return mapper.selectUser();
       }
   
       public int updateUser(User user) {
           return sqlSession.getMapper(UserMapper.class).updateUser(user);
       }
   }
   ```

5. 将自己写的实现类，注入到Spring中

   ```xml
   <bean id="userMapper" class="com.xrl.mapper.UserMapperImpl">
       <property name="sqlSession" ref="sqlSession"/>
   </bean>
   ```

6. 测试使用

   ```java
      @Test
       public void test() throws IOException {
   
          ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
           UserMapper userMapper = context.getBean("userMapper1", UserMapper.class);
           List<User> userList = userMapper.selectUser();
           for (User user : userList) {
               System.out.println(user);
           }
       }
   ```







## 13、声明式事务

### 1、回顾事务

- 把一组业务当成一个业务来做，要么操作都成功，要么都失败！
- 事务在项目开发中，十分的重要，涉及到数据的一致性问题，不能马虎！
- 确保完整性和一致性；



事务ACID原则：

- 原子性
- 一致性
- 隔离性
  - 多个业务可能操作同一个资源，业务之间不会相互影响，防止数据损坏
- 持久性
  - 事务一旦提交，无论系统发生什么问题，结果都不会在被影响，被持久化地写在存储器中



### 2、spring中的事务管理

- 声明式事务：AOP

  - 在spring的配置文件中去配置

    ```xml
    <bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
        <constructor-arg name="sqlSessionFactory" ref="sqlSessionFactory"/>
    </bean>
    
    <!--配置声明式事务-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <constructor-arg ref="dataSource" />
    </bean>
    
    <!--结合AOP实现事务的织入-->
    <!--配置事务通知-->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <!--给那些方法配置事务-->
            <!--配置事务的传播特性 propagation-->
        <tx:attributes>
            <tx:method name="add" propagation="REQUIRED"/>
            <tx:method name="update" propagation="REQUIRED"/>
            <tx:method name="delete" propagation="REQUIRED"/>
            <tx:method name="query" read-only="true"/>
            <tx:method name="*" propagation="REQUIRED"/>
        </tx:attributes>
    </tx:advice>
    
    <!--配置事务的切入-->
    <aop:config>
        <aop:pointcut id="txPointCut" expression="execution(* com.xrl.mapper.*.*(..))"/>
        <aop:advisor advice-ref="txAdvice" pointcut-ref="txPointCut"/>
    </aop:config> 
    ```

- 编程式事务：需要在代码中进行事务的管理(即改变原有的代码)









思考：

为什么需要事务？

- 如果不配置事务，可能存在数据提交不一致的情况
- 如果我们不在Spring中去配置声明式事务，我们就需要在代码中手动去配置事务
- 事务在项目的开发中十分重要，设计到数据的一致性和完整性，不能马虎



