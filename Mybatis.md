## Mybatis

### 从 XML 中构建 SqlSessionFactory

每个基于 MyBatis 的应用都是以一个 **SqlSessionFactory** 的实例为核心的。SqlSessionFactory 的实例可以通过 **SqlSessionFactoryBuilder** 获得。而 SqlSessionFactoryBuilder 则可以从 XML 配置文件或一个预先配置的 **Configuration** 实例来构建出 SqlSessionFactory 实例。

- 编写核心配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;userUnicode=true&amp;characterEncoding=utf-8&amp;serverTimezone=UTC"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments>
</configuration>
```

- 从 XML 文件中构建 SqlSessionFactory 的实例

```java
// 官方代码，读取配置文件的操作进行封装
String resource = "org/mybatis/example/mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```

- 编写mybatis工具类

```java
package com.xrl.utils;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;

// 获取SqlSessionFactory的实例从而方便获取SqlSession
public class MybatisUtils {
    private static SqlSessionFactory sqlSessionFactory;

    static {
        try {
            //使用mybatis的第一步  获取SqlSessionFactory对象
            String resource = "mybatis-config.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    //既然有了 SqlSessionFactory，顾名思义，我们可以从中获得 SqlSession 的实例。
    // SqlSession 提供了在数据库执行 SQL 命令所需的所有方法。

    public static SqlSession getSqlSession() {
        return sqlSessionFactory.openSession();

    }
}
```

### 编写代码

- 实体类

```java
package com.xrl;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    private int id;
    private String name;
    private String pwd;
}

```

- Dao接口

```java
package com.xrl.dao;

import com.xrl.pojo.User;

import java.util.List;

public interface UserDao {

    List<User> getUserList();
}
```

- 接口实现类由原来的UseDaoImpl变成了Mapper配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--namespace=绑定一个dao或者mapper接口-->
<mapper namespace="com.xrl.dao.UserDao">
    <!--id就是原来实现接口中的方法名-->
    <select id="getUserList" resultType="com.xrl.pojo.User">
    select * from user where id = #{id}
  </select>
</mapper>
```

### 测试

注意点： org.apache.ibatis.binding.BindingException: Type interface com.xrl.dao.UserDao is not known to the MapperRegistry.

如果没有在mybatis的核心配置文件中注册每个Mapper.xml就会报**MapperRegistry**异常，更正后的mybatis核心配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;userUnicode=true&amp;characterEncoding=utf-8&amp;serverTimezone=UTC"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments>
    <!--每一个Mapper.xml都需要在mybatis核心配置文件中去注册！-->
    <mappers>
        <mapper resource="com/xrl/dao/UserMapper.xml"/>
    </mappers>
</configuration>
```

- junit测试

```java
package com.xrl.dao;

import com.xrl.utils.MybatisUtils;
import org.apache.ibatis.session.SqlSession;
import org.junit.Test;

public class UserDaoTest {

    @Test
    public void test(){

        // 获得SqlSession对象
        SqlSession sqlSession = MybatisUtils.getSqlSession();

        // 执行sql(方式一：getMapper)
        UserDao mapper = sqlSession.getMapper(UserDao.class);
        System.out.println(mapper.getUserList());


        // 关闭SqlSession
        sqlSession.close();
    }
}
```

解决maven导出资源问题，导入下面的依赖

```xml
<!--在build中配置resources，来防止我们资源导出失败的问题-->
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

### CRUD

**1、namespace**

namespace中的包要和Dao/Mapper接口中的包名一致

**2、select 标签**

选择，查询语句；

- id：表示对应的namespace中的方法名
- resultType：Sql语句执行的返回值
- parameterType：参数类型

1、编写接口中的方法

```java
// 根据id查询
User getUserById(int id);
```

2、编写mapper.xml中对应的sql语句

```xml
<select id="getUserById" resultType="com.xrl.pojo.User" parameterType="integer">
    select * from user where id=#{id}
</select>
```

3、测试

```java
@Test
public void testGetUserById(){
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    System.out.println(mapper.getUserById(1));
    sqlSession.close();
}
```

**3、insert标签**

```xml
<insert id="addUser" parameterType="com.xrl.pojo.User">
    insert into user(id,name,pwd) values (#{id},#{name},#{pwd})
</insert>
```

**4、update标签**

```xml
<update id="updateUser" parameterType="com.xrl.pojo.User">
    update user set name=#{name},pwd=#{pwd} where id= #{id}
</update>
```

**5、delete标签**

```xml
<update id="updateUser" parameterType="com.xrl.pojo.User">
    update user set name=#{name},pwd=#{pwd} where id= #{id}
</update>
```

注意点：

- 增删改需要提交事务！

**6、万能的Map**

假设，我们的实体类，或者数据库中的表，字段或参数过多，我们应该考虑使用Map来进行传参！

```java
 // 插入一个用户
    int addUser2(Map<String,Object> map);
```

```xml
<!--使用map传key-->
<insert id="addUser2" parameterType="map">
    insert into user(id,pwd) values (#{userId},#{password})
</insert>
```

```java
    // 用map来增加用户
@Test
public void testAddUser2(){
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    Map<String,Object> map = new HashMap<String, Object>();
    map.put("userId",4);
    map.put("password","sdkjgfjkas");
    mapper.addUser2(map);
    sqlSession.commit();
    sqlSession.close();
}
```

Map传递参数，直接在sql中取出key即可！【parameterType="map"】

对象传递参数，直接在sql中取对象的属性即可！【parameterType="Object"】

只有一个基本类型参数的情况下，可以直接在sql中取到(就是可以不用写)

多个参数用Map，或者注解。



模糊查询怎么写？

1、java代码执行的时候，传递通配符 % %

```java
@Test
public void testGetUserLike(){
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    List<User> users = mapper.getUserLike("%小%");
    System.out.println(users);
    sqlSession.close();
}
<select id="getUserLike" resultType="com.xrl.pojo.User">
    select * from user where name like #{value};
</select>
```

2、在sql拼接时使用统配符

```xml
<select id="getUserLike" resultType="com.xrl.pojo.User">
    select * from user where name like "%" #{value} "%";
</select>
```



### 配置解析

**1、核心配置文件**

- mybatis-config.xml
- MyBatis 的配置文件包含了会深深影响 MyBatis 行为的设置和属性信息。 配置文档的顶层结构如下：

```xml
configuration（配置）
properties（属性）
settings（设置）
typeAliases（类型别名）
typeHandlers（类型处理器）
objectFactory（对象工厂）
plugins（插件）
environments（环境配置）
environment（环境变量）
transactionManager（事务管理器）
dataSource（数据源）
databaseIdProvider（数据库厂商标识）
mappers（映射器）
```

**2、环境配置（environments）**

MyBatis 可以配置成适应多种环境

**不过要记住：尽管可以配置多个环境，但每个 SqlSessionFactory 实例只能选择一种环境。**

学会使用配置多套运行环境

```xml
<environments default="test"> (只要改变这个参数即可)
    <environment id="development">
        <transactionManager type="JDBC"/>
        <dataSource type="POOLED">
            <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
            <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;userUnicode=true&amp;characterEncoding=utf-8&amp;serverTimezone=UTC"/>
            <property name="username" value="root"/>
            <property name="password" value="root"/>
        </dataSource>
    </environment>

    <environment id="test">
        <transactionManager type="JDBC"/>
        <dataSource type="POOLED">
            <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
            <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;userUnicode=true&amp;characterEncoding=utf-8&amp;serverTimezone=UTC"/>
            <property name="username" value="root"/>
            <property name="password" value="root"/>
        </dataSource>
    </environment>
</environments
```



Mybatis默认的事务管理器就是JDBC(MANAGED一般不会使用)，连接池是：POOLED（UNPOOLED，JNDI 这两种也不会使用）

**3、属性**

我们可以通过properties属性来实现引用配置文件

这些属性可以在外部进行配置，并可以进行动态替换。你既可以在典型的 Java 属性文件中配置这些属性，也可以在 properties 元素的子元素中设置。【db.properties】

![image-20210701221220261](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210701221220261.png)

编写一个配置文件

db.properties

```properties
driver=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://localhost:3306/mybatis?useSSL=true&userUnicode=true&characterEncoding=utf-8&serverTimezone=UTC
username=root
password=root
```

在核心配置文件中引入db.properties

```xml
<!--引入外部配置文件-->
<properties resource="db.properties"> //(本质上来说只要这一句话就可以了，但是里面也可以配置其他的属性)
    <property name="username" value="root"/>
    <property name="pwd" value="root"/>
</properties>
```

- 可以直接引入外部文件
- 可以在其中增加一些属性配置
- 如果两个文件有同一字段，优先使用外部配置的文件！



**4、类型别名**

- 类型别名可为 Java 类型设置一个缩写名字。
- 意在降低冗余的全限定类名书写。

```xml
<typeAliases>
    <typeAlias type="com.xrl.pojo.User" alias="User"/>
</typeAliases>
```

也可以指定一个包名，Mybatis会在包名下面搜索需要的java Bean，比如：扫描实体类的包，它的默认别名就为这个类的类名 其中首字母小写

```xml

<typeAliases>   
    <package name="com.xrl.pojo"/>
</typeAliases>
```

在实体类较少的时候，使用第一种方式

如果实体类较多，建议使用第二种

第一种可以DIY别名，第二种则不行，如果非要改，需要在实体类上增加注解`@Alias("别名")`

```java
@Alias("hello")
public class User {
    private int id;
    private String name;
    private String pwd;
}
```

**5、设置**

这是 MyBatis 中极为重要的调整设置，它们会改变 MyBatis 的运行时行为。 下表描述了设置中各项设置的含义、默认值等。

![image-20210702101022904](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210702101022904.png)

![image-20210702101058639](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210702101058639.png)

**6、其他配置**

- [typeHandlers（类型处理器）](https://mybatis.org/mybatis-3/zh/configuration.html#typeHandlers)
- [objectFactory（对象工厂）](https://mybatis.org/mybatis-3/zh/configuration.html#objectFactory)
- plugins（插件）
  - mybatis-generator-core
  - mybatis-plus
  - 通用mapper

**7、映射器（mappers）**

MapperRegistry：注册绑定我们的Mapper文件

方式一：【推荐使用】

```xml
<!--每一个Mapper.xml都需要在mybatis核心配置文件中去注册！-->
<mappers>
    <mapper resource="com/xrl/dao/UserMapper.xml"/>
</mappers>
```

方式二：使用class文件绑定注册

```xml
<!--每一个Mapper.xml都需要在mybatis核心配置文件中去注册！-->
<mappers>
    <mapper class="com.xrl.dao.UserMapper"/>
</mappers>
```

方式二的注意点：

- 接口和他的Mapper配置文件必须同名！
- 接口和他的Mapper配置文件必须在同一个包下



方式三：使用扫描包进行注入绑定

```xml
<!--每一个Mapper.xml都需要在mybatis核心配置文件中去注册！-->
<mappers>
    <package name="com.xrl.dao"/>
</mappers>
```

方式三的注意点：

- 接口和他的Mapper配置文件必须同名！
- 接口和他的Mapper配置文件必须在同一个包下

**8、生命周期和作用域**

作用域和生命周期是至关重要的，因为错误的使用会导致非常严重的**并发问题**。

### 注意

#### SqlSessionFactoryBuilder

- 这个类可以被实例化、使用和丢弃，一旦创建了 SqlSessionFactory，就不再需要它了。
-  因此 SqlSessionFactoryBuilder 实例的最佳作用域是方法作用域（也就是局部方法变量）。

#### SqlSessionFactory

- SqlSessionFactory 可以想象为：数据库连接池。
- 一旦被创建就应该在应用的运行期间一直存在，没有任何理由丢弃它或重新创建另一个实例。 
- 最简单的就是使用**单例模式**或者**静态单例模式**。

#### SqlSession

- 连接到连接池的一个请求
- 每个线程都应该有它自己的 SqlSession 实例。SqlSession 的实例不是线程安全的，因此是不能被共享的，所以它的最佳的作用域是请求或方法作用域。
- 用完之后需要赶紧关闭。否则资源被占用

![image-20210702104628466](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210702104628466.png)

这里面的每一个Mapper，就代表一个具体的业务！



### 解决属性名和字段名不一致的问题

数据库中的字段

![image-20210702105233394](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210702105233394.png)

实体类中的属性名

```java
public class User {
    private int id;
    private String name;
    private String password;
    private String perms;
}
```

测试结果

![image-20210702105932042](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210702105932042.png)



解决方式

- 起别名(对sql中的字段起别名)

  ```xml
  <select id="getUserById" resultType="User" parameterType="integer">
      select id,name,pwd as password from user where id=#{id}
  </select>
  ```

  

2、resultMap(结果集映射)

```
id name pwd -- 数据库字段名
id name password -- 实体类属性名
```

```xml
<mapper namespace="com.xrl.dao.UserMapper">
    <!--结果集映射，这里的type就是映射的返回结果-->
    <resultMap id="UserMap" type="User">
        <!--column表示数据库中的字段，property表示实体类中的属性-->
        <result column="id" property="id"/>
        <result column="name" property="name"/>
        <result column="pwd" property="password"/>
    </resultMap>

    <select id="getUserById" resultMap="UserMap">
        select * from user where id=#{id}
    </select>

</mapper>
```

![image-20210702111230333](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210702111230333.png)

- `resultMap` 元素是 MyBatis 中最重要最强大的元素。

- ResultMap 的设计思想是，对简单的语句做到零配置，对于复杂一点的语句，只需要描述语句之间的关系就行了。

- `ResultMap` 的优秀之处——你完全可以不用显式地配置它们。(只去处理那些不匹配的属性，其他的可以不用管)

  ```xml
  <resultMap id="UserMap" type="User">
      <!--column表示数据库中的字段，property表示实体类中的属性-->
      <result column="pwd" property="password"/>
  </resultMap>
  ```

  

### 日志

#### 日志工厂

如果一个数据库操作，出现了异常，我们需要排错。日志就是最好的助手！

曾经：sout、 debug的方式排错

现在：使用日志工厂

![image-20210702152605915](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210702152605915.png)

- SLF4J 
-  LOG4J 【掌握】
- LOG4J2 
- JDK_LOGGING
-  COMMONS_LOGGING 
- STDOUT_LOGGING 【掌握】（控制台输出日志）
- NO_LOGGING

在mybatis中具体使用那个日志实现，在设置中设定！

**STDOUT_LOGGING标准日志输出**

指定 MyBatis 应该使用哪个日志记录实现。如果此设置不存在，则会自动发现日志记录实现。

```xml
<settings>
    <setting name="logImpl" value="STDOUT_LOGGING"/>
</settings>
```

测试，可以看到控制台有大量的输出！我们可以通过这些输出来判断程序到底哪里出了Bug

![image-20210703125150757](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210703125150757.png)

#### Log4j

**简介：**

- Log4j是Apache的一个开源项目
- 通过使用Log4j，我们可以控制日志信息输送的目的地：控制台，文本，GUI组件....
- 我们也可以控制每一条日志的输出格式；
- 通过定义每一条日志信息的级别，我们能够更加细致地控制日志的生成过程。最令人感兴趣的就是，这些可以通过一个配置文件来灵活地进行配置，而不需要修改应用的代码。

使用步骤

1. 导入log4j的包

   ```xml
   <dependency>
       <groupId>log4j</groupId>
       <artifactId>log4j</artifactId>
       <version>1.2.17</version>
   </dependency>
   ```

2. 配置文件编写

   ```properties
   #将等级为DEBUG的日志信息输出到console和file这两个目的地，console和file的定义在下面的代码
   log4j.rootLogger=DEBUG,console,file
   #控制台输出的相关设置
   log4j.appender.console = org.apache.log4j.ConsoleAppender
   log4j.appender.console.Target = System.out
   log4j.appender.console.Threshold=DEBUG
   log4j.appender.console.layout = org.apache.log4j.PatternLayout
   log4j.appender.console.layout.ConversionPattern=[%c]-%m%n
   #文件输出的相关设置
   log4j.appender.file = org.apache.log4j.RollingFileAppender
   log4j.appender.file.File=./log/kuang.log
   log4j.appender.file.MaxFileSize=10mb
   log4j.appender.file.Threshold=DEBUG
   log4j.appender.file.layout=org.apache.log4j.PatternLayout
   log4j.appender.file.layout.ConversionPattern=[%p][%d{yy-MM-dd}][%c]%m%n
   #日志输出级别
   log4j.logger.org.mybatis=DEBUG
   log4j.logger.java.sql=DEBUG
   log4j.logger.java.sql.Statement=DEBUG
   log4j.logger.java.sql.ResultSet=DEBUG
   log4j.logger.java.sql.PreparedStatement=DEBUG
   ```
   
3. setting设置日志实现

4. 在程序中使用Log4j进行输出！

   ```java
   //注意导包：org.apache.log4j.Logger
   static Logger logger = Logger.getLogger(MyTest.class);
   @Test
   public void selectUser() {
   logger.info("info：进入selectUser方法");
   logger.debug("debug：进入selectUser方法");
   logger.error("error: 进入selectUser方法");
   SqlSession session = MybatisUtils.getSession();
       UserMapper mapper = session.getMapper(UserMapper.class);
   List<User> users = mapper.selectUser();
   for (User user: users){
   System.out.println(user);
   }
   session.close();
   }
   ```

5. 测试，看控制台输出！

   - 使用Log4j 输出日志
   - 可以看到还生成了一个日志的文件 【需要修改file的日志级别】

   ![image-20210703125630338](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210703125630338.png)

#### limit实现分页

**思考：为什么需要分页？**

在学习mybatis等持久层框架的时候，会经常对数据进行增删改查操作，使用最多的是对数据库进行查询操作，如果查询大量数据的时候，我们往往使用分页进行查询，也就是每次处理小部分数据，这样对数据库压力就在可控范围内。

**使用Limit实现分页**

```sql
语法：select * from user limit startIndex,pageSize

#如果只给定一个参数，它表示返回最大的记录行数目：
SELECT * FROM table LIMIT 5; //检索前 5 个记录行
#换句话说，LIMIT n 等价于 LIMIT 0,n。
```

**步骤：**

1. 修改Mapper文件

   ```xml
   <select id="selectUser" parameterType="map" resultType="user">
   select * from user limit #{startIndex},#{pageSize}
   </select>
   ```

2. Mapper接口，参数为map

   ```java
   //选择全部用户实现分页
   List<User> selectUser(Map<String,Integer> map);
   ```

3. 在测试类中传入参数测试

   - 推断：起始位置 = （当前页面 - 1 ） * 页面大小

     ```java
     //分页查询 , 两个参数startIndex , pageSize
     @Test
     public void testSelectUser() {
     SqlSession session = MybatisUtils.getSession();
     UserMapper mapper = session.getMapper(UserMapper.class);
     int currentPage = 1; //第几页
     int pageSize = 2; //每页显示几个
     Map<String,Integer> map = new HashMap<String,Integer>();
     map.put("startIndex",(currentPage-1)*pageSize);
     map.put("pageSize",pageSize);
     List<User> users = mapper.selectUser(map);
     for (User user: users){
     System.out.println(user);
     }
     session.close();
     }
     ```

#### RowBounds分页

不在使用SQL实现分页(了解即可)

1. 接口

   ```java
   // 分页查询2
   List<User> getUserByRowBounds();
   ```

2. mapper.xml

   ```xml
   <select id="getUserByRowBounds" resultMap="UserMap">
       select * from user
   </select>
   ```

3. 测试

   ```java
   @Test
   public void getUserByRowBounds(){
   
       SqlSession sqlSession = MybatisUtils.getSqlSession();
   
       // RowBounds实现
       RowBounds rowBounds = new RowBounds(1,2);
   
   
       // 通过java代码层面实现分页
       List<User> userList = sqlSession.selectList("com.xrl.dao.UserMapper.getUserByRowBounds",null,rowBounds);
       for (User user : userList) {
           System.out.println(user);
       }
       sqlSession.close();
   }
   ```

   #### 分页插件

   ![image-20210703132406439](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210703132406439.png)

了解即可，会用就行。

官方文档：https://pagehelper.github.io/



### 使用注解开发

**面向接口编程**

- 大家之前都学过面向对象编程，也学习过接口，但在真正的开发中，很多时候我们会选择面向接口编程
- 根本原因 : 解耦 , 可拓展 , 提高复用 , 分层开发中 , 上层不用管具体的实现 , 大家都遵守共同的标准, 使得开发变得容易 , 规范性更好
- 在一个面向对象的系统中，系统的各种功能是由许许多多的不同对象协作完成的。在这种情况下，各个对象内部是如何实现自己的,对系统设计人员来讲就不那么重要了
- 而各个对象之间的协作关系则成为系统设计的关键。小到不同类之间的通信，大到各模块之间的交互，在系统设计之初都是要着重考虑的，这也是系统设计的主要工作内容。面向接口编程就是指按照这种思想来编程。



**关于接口的理解**

- 接口从更深层次的理解，应是定义（规范，约束）与实现（名实分离的原则）的分离。
- 接口的本身反映了系统设计人员对系统的抽象理解。
- 接口应有两类：
  - 第一类是对一个个体的抽象，它可对应为一个抽象体(abstract class)；
  - 第二类是对一个个体某一方面的抽象，即形成一个抽象面（interface）；
- 一个体有可能有多个抽象面。抽象体与抽象面是有区别的。



**三个面向区别**

- 面向对象是指，我们考虑问题时，以对象为单位，考虑它的属性及方法 .
- 面向过程是指，我们考虑问题时，以一个具体的流程（事务过程）为单位，考虑它的实现 .
- 接口设计与非接口设计是针对复用技术而言的，与面向对象（过程）不是一个问题.更多的体现就是对系统整体的架构

**使用注解开发**

1. 注解在接口上实现

   ```java
   @Select("select * from user")
   List<User> getUser();
   ```

2. 需要在Mybatis核心配置文件中进行绑定

   ```xml
   <!--绑定接口-->
   <mappers>
       <mapper class="com.xrl.dao.UserMapper"/>
   </mappers>
   ```

3. 测试

   ```java
   @Test
   public void test(){
       SqlSession sqlSession = MybatisUtils.getSqlSession();
       UserMapper mapper = sqlSession.getMapper(UserMapper.class);
       List<User> userList = mapper.getUser();
       for (User user : userList) {
           System.out.println(user);
       }
       sqlSession.close();
   }
   ```

   本质：反射机制实现

   底层：动态代理！

### CRUD

我们可以在工具类创建的时候实现自动提交事务！

```java
public static SqlSession getSqlSession() {
    // 自动提交事务
    return sqlSessionFactory.openSession(true);

}
```

编写接口

```java
package com.xrl.dao;


import com.xrl.pojo.User;
import org.apache.ibatis.annotations.*;

import java.util.List;

public interface UserMapper {

    @Select("select * from user")
    List<User> getUser();

    // 当方法上存在多个参数时，所有参数必须加上@Param注解，如果是引用类型则不需要,并且以注解中的参数为主
    @Select("select * from user where id = #{id}")
    User getUserById(@Param("id") int id);

    @Insert("insert into user values(#{id},#{name},#{password},#{perms})")
    int addUser(User user);

    @Update("update user set name=#{name},pwd=#{password} where id=#{id}")
    int updateUser(User user);

    @Delete("delete from user where id=#{uid}")
    int delete(@Param("uid")int id);

}
```

测试类

```java
package com.xrl.test;

import com.xrl.dao.UserMapper;
import com.xrl.pojo.User;
import com.xrl.utils.MybatisUtils;
import org.apache.ibatis.session.SqlSession;
import org.junit.Test;

import java.util.List;

public class UserMapperTest {

    @Test
    public void test(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
       mapper.delete(4);
  /*      List<User> userList = mapper.getUser();
        for (User user : userList) {
            System.out.println(user);
        }

        System.out.println(mapper.getUserById(1));
         mapper.addUser(new User(4,"bjt","123456",null));
          mapper.updateUser(new User(4,"小张","qifei",null));
        */

        sqlSession.close();
    }

}
```

【注意：我们必须要讲接口注册绑定到我们的核心配置文件中！】

```xml
<!--绑定接口-->
<mappers>
    <mapper class="com.xrl.dao.UserMapper"/>
</mappers>
```

**关于@Param()注解**

- 基本类型的参数或者String类型，需要加上
- 引用类型不需要加
- 如果只有一个基本类型的话，可以忽略，但是建议都加上
- 我们在sql中引用的就是我们这里的@Param()中设置的属性名！



### 多对一

![image-20210703165328162](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210703165328162.png)

```sql
CREATE TABLE `teacher` (
`id` INT(10) NOT NULL,
`name` VARCHAR(30) DEFAULT NULL,
PRIMARY KEY (`id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8

INSERT INTO teacher(`id`, `name`) VALUES (1, '秦老师');

CREATE TABLE `student` (
`id` INT(10) NOT NULL,
`name` VARCHAR(30) DEFAULT NULL,
`tid` INT(10) DEFAULT NULL,
PRIMARY KEY (`id`),
KEY `fktid` (`tid`),
CONSTRAINT `fktid` FOREIGN KEY (`tid`) REFERENCES `teacher` (`id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8

INSERT INTO `student` (`id`, `name`, `tid`) VALUES ('1', '小明', '1');
INSERT INTO `student` (`id`, `name`, `tid`) VALUES ('2', '小红', '1');
INSERT INTO `student` (`id`, `name`, `tid`) VALUES ('3', '小张', '1');
INSERT INTO `student` (`id`, `name`, `tid`) VALUES ('4', '小李', '1');
INSERT INTO `student` (`id`, `name`, `tid`) VALUES ('5', '小王', '1');
```

#### 按照查询嵌套处理

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.xrl.dao.StudentMapper">
    <!--
      // 查询所有学生与对应老师的信息
        思路：
            1. 查询所有学生的信息
            2. 根据查询出来的学生的tid，寻找对应的老师
    -->
    <select id="getStudent" resultMap="StudentTeacher">
        select * from student
    </select>

    <resultMap id="StudentTeacher" type="Student">
        <result property="id" column="id"/>
        <result property="name" column="name"/>
        <!--复杂的属性，我们需要单独去处理 对象：association 集合：collection-->
        <association property="teacher" column="tid" javaType="Teacher" select="getTeacher"/>
    </resultMap>

    <select id="getTeacher" resultType="Teacher">
        select * from teacher where id=#{id}
    </select>
</mapper>
```

#### 按照结果嵌套处理

```xml
<!--按照结果嵌套处理-->
<select id="getStudent2" resultMap="StudentTeacher2">
    select s.id sid,s.name sname,t.name tname
    from student s,teacher t
    where s.tid=t.id
</select>

// 取完别名之后，一一对应即可
<resultMap id="StudentTeacher2" type="Student">
    <result property="id" column="sid"/>
    <result property="name" column="sname"/>
    <association property="teacher" javaType="Teacher">
        <result property="name" column="sname"/>
    </association>
</resultMap>
```

### 一对多

比如： 一个老师拥有多个学生！

对于老师而言，就是一对多的关系

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Student {

    private int id;
    private String name;

    private int  tid;
}
```

```java
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.util.List;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class Teacher {
    private int id;
    private String name;

    List<Student> studentList;
}
```

#### 按照结果嵌套处理

```xml
<!--安结果嵌套查询-->
<select id="getTeacher" resultMap="TeacherStudent">
    select s.id sid,s.name sname,t.name tname,t.id tid
    from student s,teacher t
     where s.tid=t.id and t.id=#{tid}
</select>

<resultMap id="TeacherStudent" type="Teacher">
    <result property="id" column="tid"/>
    <result property="name" column="tname"/>
    <!--复杂的属性，我们需要单独去处理 对象：property 集合：collection
        javaTYpe="" 指定属性的类型！
        集合中的泛型信息，我们使用ofType获取

    -->
    <collection property="studentList" ofType="Student">
        <result property="id" column="sid"/>
        <result property="name" column="sname"/>
        <result property="tid" column="tid"/>
    </collection>
</resultMap>
```

#### 按照查询嵌套处理

```xml
<select id="getTeacher2" resultMap="TeacherStudent2">
    select * from teacher where id=#{tid}
</select>

<resultMap id="TeacherStudent2" type="Teacher">
    <collection property="studentList" javaType="ArrayList" ofType="Student" select="getStudentByTeacherId" column="id"/>
</resultMap>

<select id="getStudentByTeacherId" resultType="Student">
    select * from student where tid=#{tid}
</select>
```

### 小结

1. 关联 - association【多对一】
2. 集合 - collection 【-对多】
3. JavaType & ofType
   1. JavaType 用来指定实体类中属性的类型
   2. ofType  用来指定映射到集合中的pojo类型，泛型中的约束类型！

注意点：

- 保证SQL的可读性，尽量保证通俗易懂
- 注意一对多和多对一中，属性名和字段的问题
- 如果问题不好排查，可以使用日志，建议使用Log4j

### Mysql面试高频

- Mysql引擎
- InnoDB底层原理
- 索引
- 索引优化



### 动态sql

==什么是动态sql: 动态sql就是指根据不同的条件生成不同的sql语句==

利用动态 SQL，可以彻底摆脱这种痛苦。

```xml
动态SQL元素和JSTL 或任何基于类 XML 语言的文本处理器相似。在 MyBatis 之前的版本中，需要花时间了解大量的元素。借助功能强大的基于 OGNL 的表达式，MyBatis 3 替换了之前的大部分元素，大大精简了元素种类，现在要学习的元素种类比原来的一半还要少。

if
choose (when, otherwise)
trim (where, set)
foreach
```

#### 搭建环境

```sql
CREATE TABLE blog(
	id VARCHAR(50) NOT NULL COMMENT '博客id',
	title VARCHAR(100) NOT NULL COMMENT '博客标题',
	author VARCHAR(30) NOT NULL COMMENT '博客作者',
	create_time DATETIME NOT NULL COMMENT '创建时间',
	views INT(30) NOT NULL COMMENT '浏览量'
)ENGINE=INNODB DEFAULT CHARSET=utf8
```



创建一个基础工程

1. 导包
2. 编写配置文件
3. 编写实体类
4. 编写实体类对应的Mapper接口和Mapper.xml文件



#### IF

```xml
<select id="queryBlogIf" parameterType="map" resultType="Blog">
    select * from blog where 1=1
    <if test="title != null">
        and title=#{title}
    </if>
    <if test="author != null">
        and author=#{author}
    </if>
</select>
```

#### choose (when, otherwise)

```xml
// 用choose包裹之后就想java中的switch语句一样，只能选择一个满足条件的去执行
<select id="queryBlogChoose" resultType="Blog" parameterType="map">
    select * from blog
    <where>
        <choose>
            <when test="title != null">
                title = #{title}
            </when>
            <when test="author != null">
                and author = #{author}
            </when>
            <otherwise>
                and views=#{views}
            </otherwise>
        </choose>
    </where>
</select>
```





 

#### trim（where， set）

```xml
// where标签
<select id="queryBlogIf" parameterType="map" resultType="Blog">
    select * from blog
    <where>
        <if test="title != null">
            and title=#{title}
        </if>
        <if test="author != null">
            and author=#{author}
        </if>
    </where>

</select>
```

```xml
// set 标签
<update id="updateBolg" parameterType="map">
    update blog
    <set>
        <if test="title != null">
            title = #{title},
        </if>
        <if test="author != null">
            author = #{author}
        </if>
    </set>
    where id=#{id}
</update>
```

==所谓的动态sql，本质就是sql语句，只是我们可以在sql层面去执行逻辑代码==

#### sql片段

有的时候，我们可能会将一些功能的部分抽取出来，方便复用！

1. 使用sql标签抽取公共部分

   ```xml
   <sql id="if-title-author">
       <if test="title != null">
           title=#{title}
       </if>
       <if test="author != null">
           and author=#{author}
       </if>
   </sql>
   ```

   

2. 在需要使用的地方使用include标签引用即可

   ```xml
   <select id="queryBlogIf" parameterType="map" resultType="Blog">
       select * from blog
       <where>
           <include refid="if-title-author"></include>
       </where>
   
   </select>
   ```

   注意事项：

   - 最好基于单表来定义sql片段！
   - 不要存在where标签

#### foreach

```sql
select * from user where 1=1 and
<foreach item="id" collection="ids"
      open="(" separator="or" close=")">
        #{id}
</foreach>

(id=1 or id=2 or id=3)（上面的foreach语句表示的就是这个意思）

collection 表示集合的名字，item表示集合中的元素 
```

```xml
<!--查询1,2,3号记录
        select * from blog where 1=1 and (id=1 or id=2 or id=3)

        我们现在传递一个map，这个map中可以存在一个集合
    -->
<select id="queryBlogForEach" parameterType="map" resultType="Blog">
    select * from blog
    <where>
        <foreach collection="ids" item="id"
                 open="and (" separator="or" close=")" >
            id = #{id}
        </foreach>
    </where>
</select>
```

==动态SQL就是在拼接sql语句，我们只要保证sql的正确性，按照sql的格式，去排列组合就可以了==

### Mybatis缓存

#### 13.1、简介

```
每次查询都需要连接数据库，这样耗资源！（怎么优化）
		一次查询的结果，给它暂存在一个可以直接去到的地方！--》内存：缓存
我们在此查询相同的数据的时候，直接走缓存，就不用走数据库了
```

1. 什么是缓存[ Cache ]?
   - 存在内存中的临时数据。
   - 将用户经常查询的数据放在缓存(内存)中，用户去查询数据就不用从磁盘上(关系型数据库数据文件)查询，从缓存中查询，从而提高查询效率，解决了高并发系统的性能问题。
2. 为什么使用缓存?
   - 减少和数据库的交互次数。减少系统开销,提高系统效率。
   - 经常查询并且不经常改变的数据。
3. 什么样的数据能使用缓存? 【可以使用缓存】

#### 13.2、Mybatis缓存

- MyBatis包含一个非常强大的查询缓存特性，它可以非常方便地定制和配置缓存。缓存可以极大的提升查询效率。
- MyBatis系统中默认定义了两级缓存:一级缓存和二级缓存
- 默认情况下，只有一级缓存开启。(SqlSession级别的缓存，也称为本地缓存)。二级缓存需要手动开启和配置，他是基于namespace级别的缓存。
- 为了提高扩展性，MyBatis定义了缓存接口Cache。我们可以通过实现Cache接口来自定义二级缓存

#### 13.3、一级缓存

- —级缓存也叫本地缓存: SqlSession
  - 与数据库同一次会话期间查询到的数据会放在本地缓存中。
  - 以后如果需要获取相同的数据，直接从缓存中拿，没必须再去查询数据库;

测试步骤：

1. 开启日志

2. 测试在一次sqlSession中查询两次相同的记录

   ```java
    @Test
       public void test(){
           SqlSession sqlSession = MybatisUtils.getSqlSession();
           UserMapper mapper = sqlSession.getMapper(UserMapper.class);
           User user = mapper.queryUserById(1);
           System.out.println(user);
           System.out.println("==================");
           User user2 = mapper.queryUserById(1);
           System.out.println(user2);
   
           System.out.println(user==user2);
           sqlSession.close();
       }
   ```

   

3. 查看日志输出

4. ![image-20210704220755356](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210704220755356.png)

缓存失效的情况：

1. 查询不同的东西

2. 增删改操作，可能会改变原来的数据，所以缓存必定会刷新

   ![image-20210704221608341](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210704221608341.png)

3. 查询不同的Mapper.xml

4. 手动清理缓存！

   ```java
    @Test
       public void test(){
           SqlSession sqlSession = MybatisUtils.getSqlSession();
           UserMapper mapper = sqlSession.getMapper(UserMapper.class);
           User user = mapper.queryUserById(1);
           System.out.println(user);
   
           sqlSession.clearCache(); // 手动清理缓存
   
           System.out.println("==================");
           User user2 = mapper.queryUserById(1);
           System.out.println(user2);
   
           System.out.println(user==user2);
           sqlSession.close();
       }
   ```

   小结：一级缓存默认是开启的（也无法关闭）。只有在一次SqlSession中有效，也就是拿到连接到关闭连接这个区间段！

   一级缓存就是一个map

#### 13.4、二级缓存

- 二级缓存也叫全局缓存，一级缓存作用域太低了，所以诞生了二级缓存
- 基于namespace级别的缓存，一个名称空间，对应一个二级缓存;
- 工作机制
  - 一个会话查询一条数据，这个数据就会被放在当前会话的一级缓存中;
  - 如果当前会话关闭了，这个会话对应的一级缓存就没了;但是我们想要的是，会话关闭了—级缓存中的数据被保存到二级缓存中;
  - 新的会话查询信息，就可以从二级缓存中获取内容；
  - 不同的mapper查出的数据会放在自己对应的缓存(map)中。

步骤：

1. 开启全局缓存

   ```xml
   <!--显示开启全局缓存-->(在核心配置文件中开启)
           <setting name="cacheEnabled" value="true"/>
   ```

2. 在要使用二级缓存的Mapper中开启

   ```xml
       <!--在当前Mapper.xml中开启二级缓存-->
       <cache />
   ```

   也可以自定义参数

   ```xml
   <!--在当前Mapper.xml中开启二级缓存-->
   <cache eviction="FIFO"
          flushInterval="60000"
          size="512"
          readOnly="true"/>
   ```

   这个更高级的配置创建了一个 FIFO 缓存，每隔 60 秒刷新，最多可以存储结果对象或列表的 512 个引用，而且返回的对象被认为是只读的，因此对它们进行修改可能会在不同线程中的调用者产生冲突。

   可用的清除策略有：

   - `LRU` – 最近最少使用：移除最长时间不被使用的对象。
   - `FIFO` – 先进先出：按对象进入缓存的顺序来移除它们。
   - `SOFT` – 软引用：基于垃圾回收器状态和软引用规则移除对象。
   - `WEAK` – 弱引用：更积极地基于垃圾收集器状态和弱引用规则移除对象。

   默认的清除策略是 LRU。

3. 测试

   ```java
   User user = mapper.queryUserById(1);
   System.out.println(user);
   sqlSession.close(); // 关闭一个SqlSession
   
   UserMapper mapper2 = sqlSession2.getMapper(UserMapper.class);
   User user2 = mapper2.queryUserById(1);
   System.out.println(user2);
   
   System.out.println(user==user2);
   
   sqlSession2.close();// 关闭另一个SqlSession
   ```

   ![image-20210705094405620](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210705094405620.png)

注意

​	当开启二级缓存时，要将实体类进行序列化，不然会报错` java.io.NotSerializableException: com.xrl.pojo.User`



小结

- 只要开启了二级缓存，在同一个Mapper下就有效
- 所有的数据都会先放在一级缓存中
- 只有当会话提交，或者关闭的时候，可会提交到二级缓存中

#### 13.5、缓存原理

![image-20210705101040558](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210705101040558.png)

缓存顺序

- 当用户执行查询操作时，会先走二级缓存，如果二级缓存中有就直接返回结果，没有就去找一级缓存
- 如果在一级缓存中存在待查找的数据，就直接返回结果，没有就去数据库中去查询
- 数据库中查询结果
- 数据库中查询完结果，会把结果首先缓存在一级缓存中
- 当关闭SqlSession后，会把缓存提交到二级缓存中，这样就变成了一个循环



#### 13.6、自定义缓存-encache

```xml
EhCache 是一个广泛使用的开源java分布式缓存。主要面向通用缓存
```





要在程序中使用EhCache，先导包

```xml
<dependency>
    <groupId>org.mybatis.caches</groupId>
    <artifactId>mybatis-ehcache</artifactId>
    <version>1.2.1</version>
</dependency>
```



在mapper选择encache缓存

```xml
<cache type="org.mybatis.caches.ehcache.EhcacheCache"/>
```

配置ehcache.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="http://ehcache.org/ehcache.xsd"
         updateCheck="false">
    <!--
    diskStore：为缓存路径，ehcache分为内存和磁盘两级，此属性定义磁盘的缓存位
    置。参数解释如下：
    user.home – 用户主目录
    user.dir – 用户当前工作目录
    java.io.tmpdir – 默认临时文件路径
    -->
    <diskStore path="./tmpdir/Tmp_EhCache"/>

    <defaultCache
            eternal="false"
            maxElementsInMemory="10000"
            overflowToDisk="false"
            diskPersistent="false"
            timeToIdleSeconds="1800"
            timeToLiveSeconds="259200"
            memoryStoreEvictionPolicy="LRU"/>
    <cache
            name="cloud_user"
            eternal="false"
            maxElementsInMemory="5000"
            overflowToDisk="false"
            diskPersistent="false"
            timeToIdleSeconds="1800"
            timeToLiveSeconds="1800"
            memoryStoreEvictionPolicy="LRU"/>
    <!--
    defaultCache：默认缓存策略，当ehcache找不到定义的缓存时，则使用这个缓存策
    略。只能定义一个。
    -->
    <!--
    name:缓存名称。
    maxElementsInMemory:缓存最大数目
    maxElementsOnDisk：硬盘最大缓存个数。
    eternal:对象是否永久有效，一但设置了，timeout将不起作用。
    overflowToDisk:是否保存到磁盘，当系统宕机时
    timeToIdleSeconds:设置对象在失效前的允许闲置时间（单位：秒）。仅当
    eternal=false对象不是永久有效时使用，可选属性，默认值是0，也就是可闲置时间无穷大。
    timeToLiveSeconds:设置对象在失效前允许存活时间（单位：秒）。最大时间介于创建
    时间和失效时间之间。仅当eternal=false对象不是永久有效时使用，默认是0.，也就是对象存
    活时间无穷大。
    diskPersistent：是否缓存虚拟机重启期数据 Whether the disk store
    persists between restarts of the Virtual Machine. The default value is
    false.
    diskSpoolBufferSizeMB：这个参数设置DiskStore（磁盘缓存）的缓存区大小。默
    认是30MB。每个Cache都应该有自己的一个缓冲区。
    diskExpiryThreadIntervalSeconds：磁盘失效线程运行时间间隔，默认是120秒。
    memoryStoreEvictionPolicy：当达到maxElementsInMemory限制时，Ehcache将
    会根据指定的策略去清理内存。默认策略是LRU（最近最少使用）。你可以设置为FIFO（先进先
    出）或是LFU（较少使用）。
    clearOnFlush：内存数量最大时是否清除。
    memoryStoreEvictionPolicy:可选策略有：LRU（最近最少使用，默认策略）、
    FIFO（先进先出）、LFU（最少访问次数）。
    FIFO，first in first out，这个是大家最熟的，先进先出。
    LFU， Less Frequently Used，就是上面例子中使用的策略，直白一点就是讲一直以
    来最少被使用的。如上面所讲，缓存的元素有一个hit属性，hit值最小的将会被清出缓存。
    LRU，Least Recently Used，最近最少使用的，缓存的元素有一个时间戳，当缓存容
    量满了，而又需要腾出地方来缓存新的元素的时候，那么现有缓存元素中时间戳离当前时间最远的
    元素将被清出缓存。
    -->
</ehcache>
```

一般都不会用这个，层面低。