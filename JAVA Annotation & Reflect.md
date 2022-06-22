## JAVA Annotation & Reflect

- **Annotation是从JDK5.0开始引入的新技术**
- **Annotation的作用：**
  1. 不是程序本身,可以对程序作出解释.(这一点和注释(comment)没什么区别)
  2. 可以被其他程序(比如:编译器等)读取.
- **Annotation的格式：**
  - 注解是以"@注释名"在代码中存在的,还可以添加一些参数值,例如:@SuppressWarnings(value="unchecked").
- **Annotation在哪里使用？**
  - 可以附加在package , class , method , field等上面,相当于给他们添加了额外的辅助信息,我们可以通过反射机制编程实现对这些元数据的访问

### 内置注解

- @Override:定义在java.lang.Override中,此注释只适用于修辞方法，表示一个方法声明打算重写超类中的另一个方法声明.
- @Deprecated:定义在java.lang.Deprecated中,此注释可以用于修辞方法,属性﹐类，表示不鼓励程序员使用这样的元素,通常是因为它很危险或者存在更好的选择．
- @suppressWarnings:定义在java.lang.SuppressWarnings中,用来抑制编译时的警告信息.与前两个注释有所不同,你需要添加一个参数才能正确使用,这些参数都是已经定义好了的,我们选择性的使用就好了
  - @SuppressWarnings("all")﹒
  - @SuppressWarnings("unchecked")
  - @SuppressWarnings(value={"unchecked","deprecation"})
  - 等等.

### 元注解

- 元注解的作用就是负责注解其他注解， Java定义了4个标准的meta-annotation类型,他们被用提供对其他annotation类型作说明．
- 这些类型和它们所支持的类在java.lang.annotation包中可以找到.**(@Target , @Retention ,@Documented , @Inherited )**
  - **@Target**:用于描述注解的使用范围(即:被描述的注解可以用在什么地方)
  - **@Retention**:表示需要在什么级别保存该注释信息﹐用于描述注解的生命周期
    - (SOURCE< CLASS < **RUNTIME**)
  - **@Document**:说明该注解将被包含在javadoc中
  - **@Inherited**:说明子类可以继承父类中的该注解

### 自定义注解

- 使用@interface自定义注解时﹐自动继承了java.lang.annotation.Annotation接口
- **分析：**
  - @interface用来声明一个注解,格式: public @interface注解名{定义内容}
  - 其中的每一个方法实际上是声明了一个配置参数。
  - 方法的名称就是参数的名称.
  - 返回值类型就是参数的类型(返回值只能是基本类型,Class , String , enum ).可以通过default来声明参数的默认值
  - 如果只有一个参数成员，一般参数名为value
  - 注解元素必须要有值,我们定义注解元素时,经常使用空字符串,0作为默认值.



## 反射机制

### Java Reflection

- Reflection(反射）是Java被视为动态语言的关键，反射机制允许程序在执行期借助于Reflection APl取得任何类的内部信息，并能直接操作任意对象的内部属性及方法。

  ```java
  Class c = Class.forName("java.lang.String")
  ```

- 加载完类之后，在堆内存的方法区中就产生了一个Class类型的对象（一个类只有一个Class对象)，这个对象就包含了完整的类的结构信息。我们可以通过这个对象看到类的结构。这个对象就像一面镜子，透过这个镜子看到类的结构，所以，我们形象的称之为:反射

![image-20210728152005783](.\typora-user-images\image-20210728152005783.png)

### java反射机制提供的功能

- 在运行时判断任意一个对象所属的类
- 在运行时构造任意一个类的对象
- 在运行时判断任意一个类所具有的成员变量和方法
- 在运行时获取泛型信息
- 在运行时调用任意一个对象的成员变量和方法
- 在运行时处理注解
- 生成动态代理
- ……

优点：

- 可以实现动态创建对象和编译，体现出很大的灵活性

缺点

- 对性能有影响。使用反射基本上是一种解释操作，我们可以告诉JVM，我们希望做什么并且它满足我们的要求。这类操作总是慢于直接执行相同的操作。

### 反射主要的API

- java.lang.Class:代表一个类

- java.lang.reflect.Method:代表类的方法

- java.lang.reflect.Field:代表类的成员变量

- java.lang.reflect.Constructor:代表类的构造器

- ……

- ```java
  // 什么叫反射
  public class Test02 {
      public static void main(String[] args) throws ClassNotFoundException {
          // 通过反射获取类的Class对象
          Class c1 = Class.forName("com.xrl.reflection.User");
          Class c2 = Class.forName("com.xrl.reflection.User");
          Class c3 = Class.forName("com.xrl.reflection.User");
          Class c4 = Class.forName("com.xrl.reflection.User");
          // 一个类在内存中只有一个Class对象
          // 一个类被加载后，类的整个结构都会被封装在Class对象中
          System.out.println(c2.hashCode());
          System.out.println(c3.hashCode());
          System.out.println(c4.hashCode());
  
      }
  }
  ```

### Class类

在Object类中定义了以下方法，此方法将被所有子类继承

```java
public final Class getClass()
```

以上的方法返回值的类型是一个Class类，此类是Java反射的源头，实际上所谓反射从程序的运行结果来看也很好理解，即:可以通过对象反射求出类的名称。

![image-20210728153919925](.\typora-user-images\image-20210728153919925.png)

对象照镜子后可以得到的信息:某个类的属性、方法和构造器、某个类到底实现了哪些接口。对于每个类而言，JRE 都为其保留一个不变的Class类型的对象。一个Class对象包含了特定某个结构(classlinterfacelenumlannotation/primitive type/void/)的有关信息。

- Class本身也是一个类
- Class对象只能由系统建立对象
- 一个加载的类在JVM中只会有一个Class实例
- 一个Class对象对应的是一个加载到JVM中的一个.class文件
- 每个类的实例都会记得自己是由哪个Class实例所生成
- 通过Class可以完整地得到一个类中的所有被加载的结构
- Class类是Reflection的根源，针对任何你想动态加载、运行的类，唯有先获得相应的Class对象

### Class类中的常用方法

![image-20210728154428461](.\typora-user-images\image-20210728154428461.png)



### 获取Class类的实例

1. 若已知具体的类，通过类的class属性获取，该方法最为安全可靠，程序性能最高。

   ```java
   Class clazz = Person.class;
   ```

2. 已知某个类的实例，调用该实例的getClass()方法获取Class对象

   ```java
   Class class = person.getClass();
   ```

3. 已知一个类的全类名，且该类在类路径下，可通过Class类的静态方法forName()获取,可能抛出ClassNotFoundException

   ```java
   Class clazz = Class.forName("demo01.Student");
   ```

4. 内置基本数据类型可以直接用类名.Type

5. 还可以利用ClassLoader我们力后讲解

   ```java
   // 测试Class类的创建方式有哪些
   public class Test03 {
       public static void main(String[] args) throws ClassNotFoundException {
           Person person = new Student();
           System.out.println("这个人是："+person.name);
   
           // 方式一： 通过对象获得
           Class c1 = person.getClass();
           System.out.println(c1.hashCode());
   
           // 方式二：forName
           Class c2 = Class.forName("com.xrl.reflection.Student");
           System.out.println(c2.hashCode());
   
           // 方式三： 通过类名.class获取
           Class c3 = Student.class;
           System.out.println(c3.hashCode());
   
           // 方式四：基本内置类型的包装类都有一个Type属性
           Class c4 = Integer.TYPE;
           System.out.println(c4);
   
           // 获得父类类型
           Class c5 = c1.getSuperclass();
           System.out.println(c5);
   
       }
   }
   ```

### 那些类型可以有Class对象？

- class：外部类，成员(成员内部类，静态内部类)，局部内部类，匿名内部类。
- []: 数组
- interface：接口
- enum：枚举
- annotation：注解@interface
- primitive type：基本数据类型
- void

```java
// 所有类型的Class
public class Test04 {
    public static void main(String[] args) {
        Class c1 = Object.class; // 类
        Class c2 = Comparable.class; // 接口
        Class c3 = String[].class;// 一维数组
        Class c4 = int[][].class; // 二维数组
        Class c5 = Override.class; // 注解
        Class c6 = ElementType.class; // 枚举
        Class c7 = Integer.class; // 基本数据类型
        Class c8 = void.class; // void
        Class c9 = Class.class; // Class
        System.out.println(c1);
        System.out.println(c2);
        System.out.println(c3);
        System.out.println(c4);
        System.out.println(c5);
        System.out.println(c6);
        System.out.println(c7);
        System.out.println(c8);
        System.out.println(c9);
        int[] a = new int[10];
        int[] b = new int[100];
        System.out.println(a.getClass().hashCode());// 685325104
        System.out.println(b.getClass().hashCode()); // 685325104
    }
}
```





### Java 内存分析

**Java内存**

- **堆**
  - 存放new的对象和数组
  - 可以被所有的线程共享，不会存放别的对象引用
- **栈**
  - 存放基本变量类型（会包含这个基本类型的具体数值）
  - 引用对象的变量（会存放这个引用在堆里面的具体地址）
- **方法区（是一个特殊的堆）**
  - 可以被所有的线程共享
  - 包含了所有的class和static变量

### 类加载过程

当程序主动使用某个类时，如果该类还未被加载到内存中，则系统会通过如下三个步骤来对该类进行初始化。

![image-20210728170521570](.\typora-user-images\image-20210728170521570.png)

### 类的加载与ClassLoader的理解

- 加载:将class文件字节码内容加载到内存中，并将这些静态数据转换成方法区的运行时数据结构,然后生成一个代表这个类的java.lang.Class对象.

- 链接:将Java类的二进制代码合并到JVM的运行状态之中的过程。

  - 验证:确保加载的类信息符合JVM规范，没有安全方面的问题
  - 准备:正式为类变量(static)分配内存并设置类变量默认初始值的阶段，这些内存都将在方法区中进行分配.
  - 解析:虚拟机常量池内的符号引用（常量名）替换为直接引用(地址)的过程。

- 初始化：

  - 执行类构造器<clinit> ()方法的过程。类构造器<clinit>()方法是由编译期自动收集类中所有类变量的赋值动作和静态代码块中的语句合并产生的。(类构造器是构造类信息的，不是构造该类对象的构造器)。
  - 当初始化一个类的时候，如果发现其父类还没有进行初始化，则需要先触发其父类的初始化。
  - 虚拟机会保证一个类的<clinit>()方法在多线程环境中被正确加锁和同步。

  ```java
  package com.xrl.reflection;
  
  
  public class Test05 {
      public static void main(String[] args) {
          // 创建对象后，先执行类里的静态代码块，然后执行无参构造
          A a = new A();
          System.out.println(A.m);
  
          /*
              1. 加载到内存，会产生一个类对应的Class对象
              2. 链接，链接结束后m=0
              3. 初始化
                  <clinit>(){
                       System.out.println("A类静态代码块初始化");
                       m = 300;
                       m = 100;
                  }
              输出结果：
                  A类静态代码块初始化
                  A类无参构造初始化
                  100
           */
      }
  }
  
  
  class A{
      static{
          System.out.println("A类静态代码块初始化");
          m = 300;
      }
      static int m = 100;
  
      public A() {
          System.out.println("A类无参构造初始化");
      }
  }
  
  ```

### 什么时候会发生类初始化

- 类的主动引用（一定会发生类的初始化）

  - 当虚拟机启动，先初始化main方法所在的类
  - new一个类的对象
  - 调用类的静态成员(除了final常量)和静态方法
  - 使用java.lang.reflect包的方法对类进行反射调用
  - 当初始化一个类，如果其父类没有被初始化，则先会初始化它的父类

- 类的被动调用（不会发生类的初始化）

  - 当访问一个静态域时，只有真正声明这个域的类才会被初始化。如:当通过子类引用父类的静态变量，不会导致子类初始化

  - 通过数组定义类引用，不会触发此类的初始化

  - 引用常量不会触发此类的初始化（常量在链接阶段就存入调用类的常量池中了)

  - ```java
    package com.xrl.reflection;
    
    // 测试类什么时候会初始化
    public class Test06 {
        static{
            System.out.println("Main类被加载");
        }
    
        public static void main(String[] args) throws ClassNotFoundException {
            // 1. 主动引用
    //        Son son = new Son();
            /*
                Son son = new Son();的结果为：
                Main类被加载
                父类被加载
                子类被加载
             */
    
            // 反射也会产生主动引用
           // Class.forName("com.xrl.reflection.Son");
            /*
                Class.forName("com.xrl.reflection.Son");的结果为：
                Main类被加载
                父类被加载
                子类被加载
             */
            // 不会产生类的引用的方法
    //        System.out.println(Son.b);
            /*
                Main类被加载
                父类被加载
                2
             */
            // Son[] array = new Son[5]; // Main类被加载
            System.out.println(Son.M);
            /*
                Main类被加载
                1
             */
        }
    }
    
    
    class Father {
        static int b = 2;
    
        static {
            System.out.println("父类被加载");
        }
    }
    
    class Son extends Father {
        static {
            System.out.println("子类被加载");
            m = 300;
        }
        static int m = 100;
        static final int M = 1;
    }
    ```

### 类加载器的作用

- 类加载器的作用:将class文件字节码内容加载到内存中，并将这些静态数据转换成方法区的运行时数据结构，然后在堆中生成一个代表这个类的java.lang.Class对象，作为方法区中类数据的访问

- 类缓存:标准的JavaSE类加载器可以按要求查找类，但一旦某个类被加载到类加载器中，它将维持加载(缓存)一段时间。不过JVM垃圾回收机制可以回收这些Class对象

  ![image-20210728200223263](.\typora-user-images\image-20210728200223263.png)

类加载器作用是用来把类(class)装载进内存的。JVM规范定义了如下类型的类的加载器。

- 引导类加载器:用C++编写的，是JVM自带的类加载器,负责Java平台核心库，==用来装载核心类库（rt.jar）==。该加载器无法直接获取
- 扩展类加载器:负责jre/lib/ext目录下的jar包装入工作库
- 系统类加载器:负责java -classpath所指的目录下的类与jar包装入工作，是最常用的加载器

![image-20210728200358051](.\typora-user-images\image-20210728200358051.png)

```java
package com.xrl.reflection;

public class Test07 {
    public static void main(String[] args) throws ClassNotFoundException {
        // 获取系统类的加载器(常用的加载器)
        ClassLoader systemClassLoader = ClassLoader.getSystemClassLoader();
        System.out.println(systemClassLoader); // sun.misc.Launcher$AppClassLoader@14dad5dc

        // 获取系统类加载器的父类加载器 --> 扩展类加载器
        ClassLoader parent = systemClassLoader.getParent();
        System.out.println(parent); // sun.misc.Launcher$ExtClassLoader@28d93b30

        // 获取扩展类加载器的父类加载器--> 根加载器(C/C++编写，我们拿不到会返回null)
        ClassLoader parent1 = parent.getParent();
        System.out.println(parent1); // null

        // 测试当前类是由那个类加载器加载的
        ClassLoader classLoader = Class.forName("com.xrl.reflection.Test07").getClassLoader();
        System.out.println(classLoader); // sun.misc.Launcher$AppClassLoader@14dad5dc

        // 测试JDK内置的类是由那个类加载器加载的
        classLoader = Class.forName("java.lang.Object").getClassLoader();
        System.out.println(classLoader); // null

        // 如何获取系统类加载器可以加载的路径
        System.out.println(System.getProperty("java.class.path"));
        /*
            E:\Program Files\Java\jdk1.8.0_45\jre\lib\charsets.jar;
            E:\Program Files\Java\jdk1.8.0_45\jre\lib\deploy.jar;
            E:\Program Files\Java\jdk1.8.0_45\jre\lib\ext\access-bridge-64.jar;
            E:\Program Files\Java\jdk1.8.0_45\jre\lib\ext\cldrdata.jar;
            E:\Program Files\Java\jdk1.8.0_45\jre\lib\ext\dnsns.jar;
            E:\Program Files\Java\jdk1.8.0_45\jre\lib\ext\jaccess.jar;
            E:\Program Files\Java\jdk1.8.0_45\jre\lib\ext\jfxrt.jar;
            E:\Program Files\Java\jdk1.8.0_45\jre\lib\ext\localedata.jar;
            E:\Program Files\Java\jdk1.8.0_45\jre\lib\ext\nashorn.jar;
            E:\Program Files\Java\jdk1.8.0_45\jre\lib\ext\sunec.jar;
            E:\Program Files\Java\jdk1.8.0_45\jre\lib\ext\sunjce_provider.jar;
            E:\Program Files\Java\jdk1.8.0_45\jre\lib\ext\sunmscapi.jar;
            E:\Program Files\Java\jdk1.8.0_45\jre\lib\ext\sunpkcs11.jar;
            E:\Program Files\Java\jdk1.8.0_45\jre\lib\ext\zipfs.jar;
            E:\Program Files\Java\jdk1.8.0_45\jre\lib\javaws.jar;
            E:\Program Files\Java\jdk1.8.0_45\jre\lib\jce.jar;
            E:\Program Files\Java\jdk1.8.0_45\jre\lib\jfr.jar;
            E:\Program Files\Java\jdk1.8.0_45\jre\lib\jfxswt.jar;
            E:\Program Files\Java\jdk1.8.0_45\jre\lib\jsse.jar;
            E:\Program Files\Java\jdk1.8.0_45\jre\lib\management-agent.jar;
            E:\Program Files\Java\jdk1.8.0_45\jre\lib\plugin.jar;
            E:\Program Files\Java\jdk1.8.0_45\jre\lib\resources.jar;
            E:\Program Files\Java\jdk1.8.0_45\jre\lib\rt.jar;
            F:\ideaspace\anno_reflect\out\production\anno_reflect;
            E:\Program Files\JetBrains\IntelliJ IDEA 2019.1.4\lib\idea_rt.jar

         */
    }
}
```

所谓双亲委派机制就是如果我们自定义的类与jdk的类重名了，我们自己的类是不会起作用的。因为，在类加载的过程中，一旦检测到jdk中的类和我们的写的类重名就不会起作用。（个人感觉是不会加载进内存）



### 获取运行时类的完整结构

**通过反射获取运行时类的完整结构**

**类的结构有**：Filed（字段）、Method、Constructor、SuperClass、Interface、Annotation

```java
package com.xrl.reflection;

import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.Method;

// 获取类的信息
public class Test08 {
    public static void main(String[] args) throws ClassNotFoundException, NoSuchFieldException, NoSuchMethodException {
        Class c1 = Class.forName("com.xrl.reflection.User");

        // 获取类的全类名
        System.out.println(c1.getName()); // com.xrl.reflection.User
        //获取类的简单名字
        System.out.println(c1.getSimpleName()); // User

        // 获取类的属性
        System.out.println("=================");
        Field[] fields = c1.getFields(); // 之类找到public的属性
        fields = c1.getDeclaredFields();
        for (Field field : fields) {
            System.out.println(field);
        }
        // 获得指定属性的值
        Field name = c1.getDeclaredField("name");
        System.out.println(name);

        // 获得类的方法
        System.out.println("====================");
        Method[] methods = c1.getMethods();
        for (Method method : methods) {
            System.out.println("正常的"+method); // 获得及其父类的所有public方法
        }
        methods = c1.getDeclaredMethods();
        for (Method method : methods) {
            System.out.println("getDeclaredMethods: "+method); // 获得本类的所有方法
        }

        // 获取指定的方法
        // 因为有重载，所以一定要有参数
        Method getName = c1.getMethod("getName", null);
        Method setName = c1.getMethod("setName", String.class);
        System.out.println(getName);
        System.out.println(setName);

        // 获取所有的构造器
        System.out.println("=================");
        Constructor[] constructors = c1.getConstructors(); // 获取本类的public构造器
        for (Constructor constructor : constructors) {
            System.out.println(constructor);
        }
        constructors = c1.getDeclaredConstructors(); // 获得本类的所有构造器
        for (Constructor constructor : constructors) {
            System.out.println("#"+constructor);
        }

        // 获取指定的构造器
        Constructor declaredConstructor = c1.getDeclaredConstructor(String.class, int.class, int.class);
        System.out.println(declaredConstructor);


    }
}
```

## 有了Class对象，能做什么？

### 创建对象

- 创建类的对象：调用Class对象的newInstance()方法，但是使用该方法有条件

  1. 类必须有一个无参构造器

  2. 类的构造器的访问权限需要足够

- 思考？难道没有无参构造器就不能创建对象了吗？只要在操作的时候明确调用类中的构造器，并将参数传递进去之后，才可以实例化操作

  - 步骤如下：
    1. 通过Class类的getDeclaredConstructor(Class ... parameterTypes)取得本类的指定形参类型的构造器
    2. 向构造器的形参中传递一个对象数组进去，里面包含了构造器中所需的各个参数。
    3. 通过Constructor实例化对象

  

### 调用指定的方法

通过反射，调用类中的方法，通过Method类完成。

1. 通过Class类的getMethod(String name,Class..parameterTypes)方法取得一个Method对象，并设置此方法操作时所需要的参数类型。

2. 之后使用Object invoke(Object obj, Object[] args)进行调用，并向方法中传递要设置的obj对象的参数信息。

   ![image-20210728212842854](.\typora-user-images\image-20210728212842854.png)

3. **Object invoke(Object obj, Object ... args)**

   1. object对应原方法的返回值，若原方法无返回值;此时返回null若原方法若为静态方法，此时形参Object obj可为null
   2. 若原方法形参列表为空，则Object[] args为null
   3. 若原方法声明为private,则需要在调用此invoke()方法前，显式调用方法对象的setAccessible(true)方法，将可访问private的方法。



### setAccessible

- Method和Field、Constructor对象都有setAccessible()方法。setAccessible作用是启动和禁用访问安全检查的开关。
- 参数值为true则指示反射的对象在使用时应该取消Java语言访问检查。
  - 提高反射的效率。如果代码中必须用反射，而该句代码需要频繁的被调用，那么请设置为true。
  - 使得原本无法访问的私有成员也可以访问
- 参数值为false则指示反射的对象应该实施Java语言访问检查

```java
package com.xrl.reflection;

import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

// 动态创建对象，通过反射
public class Test09 {
    public static void main(String[] args) throws ClassNotFoundException, IllegalAccessException, InstantiationException, NoSuchMethodException, InvocationTargetException, NoSuchFieldException {
        Class c1 = Class.forName("com.xrl.reflection.User");

        //User user = (User) c1.newInstance(); // 类里面必须有无参构造器，且访问权限足够才可以
//        System.out.println(user);

        // 通过构造器创建对象
        //Constructor constructor = c1.getDeclaredConstructor(String.class, int.class, int.class);
        //User user2 = (User) constructor.newInstance("rlxiang", 1, 18);
        //System.out.println(user2);

        /*
            通过反射调用普通方法
            1. 获取一个类的实例
            2. 获取该类中的方法
            3. 执行该方法  invoke(类实例，参数)
         */
        User user3 = (User) c1.newInstance();
        // 通过反射获取一个方法
        Method setName = c1.getDeclaredMethod("setName", String.class);
        setName.invoke(user3, "rlxiang");
        System.out.println(user3.getName()); // rlxiang

        /*
            通过反射操作属性
            1. 获取一个类的实例
            2. 获取该类中额属性
            3. 在操作私有属性时，要关闭权限安全检测 setAccessible(true)
            4. set(类实例, 值)
         */
        System.out.println("00000000000000000000");
        User user4 = (User) c1.newInstance();
        Field name = c1.getDeclaredField("name");
        name.setAccessible(true);
        name.set(user4, "rlxiang2");
        System.out.println(user4.getName()); // rlxiang2

    }
}
```





### 性能对比

```java
package com.xrl.reflection;

import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

// 分析性能问题
public class Test10 {
    // 普通方式
    public static void test01(){
        User user = new User();
        long startTime = System.currentTimeMillis();
        for (int i = 0; i < 1000000000; i++) {
            user.getName();
        }
        long endTime = System.currentTimeMillis();
        System.out.println("普通方式执行10亿次花费："+(endTime-startTime)+"ms");
    }

    // 反射方式调用
    public static void test02() throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
        User user = new User();
        Class c1 = user.getClass();

        Method getName = c1.getDeclaredMethod("getName", null);

        long startTime = System.currentTimeMillis();
        for (int i = 0; i < 1000000000; i++) {
            getName.invoke(user, null);
        }
        long endTime = System.currentTimeMillis();
        System.out.println("反射方式执行10亿次花费："+(endTime-startTime)+"ms");
    }

    // 反射方式调用，关闭权限检测
    public static void test03() throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
        User user = new User();
        Class c1 = user.getClass();

        Method getName = c1.getDeclaredMethod("getName", null);
        getName.setAccessible(true);
        long startTime = System.currentTimeMillis();
        for (int i = 0; i < 1000000000; i++) {
            getName.invoke(user, null);
        }
        long endTime = System.currentTimeMillis();
        System.out.println("关闭检测，反射方式执行10亿次花费："+(endTime-startTime)+"ms");
    }

    public static void main(String[] args) throws NoSuchMethodException, IllegalAccessException, InvocationTargetException {
        test01(); // 普通方式执行10亿次花费：3ms
        test02(); // 反射方式执行10亿次花费：2715ms
        test03(); // 关闭检测，反射方式执行10亿次花费：1341ms
    }

}
```





### 反射操作泛型

- Java采用泛型擦除的机制来引入泛型，Java中的泛型仅仅是给编译器javac使用的,确保数据的安全性和免去强制类型转换问题，但是，一旦编译完成,所有和泛型有关的类型全部擦除
- 为了通过反射操作这些类型，Java新增了**ParameterizedType , GenericArrayType ,TypeVariable和 WildcardType**几种类型来代表不能被归一到Class类中的类型但是又和原始类型齐名的类型.
- ParameterizedType:表示一种参数化类型,比如Collection<String>
- GenericArrayType:表示一种元素类型是参数化类型或者类型变量的数组类型
- TypeVariable :是各种类型变量的公共父接口
- WildcardType :代表一种通配符类型表达式

```java
package com.xrl.reflection;

import java.lang.reflect.Method;
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;
import java.util.List;
import java.util.Map;

// 通过反射获取泛型
public class Test11 {

    public void test01(Map<String,User> map, List<User> list){
        System.out.println("test01");
    }

    public Map<String,User> test02(){
        System.out.println("test02");
        return null;
    }

    public static void main(String[] args) throws NoSuchMethodException {
        Method method = Test11.class.getDeclaredMethod("test01", Map.class, List.class);
        Type[] genericParameterTypes = method.getGenericParameterTypes(); // 获得泛型的参数类型
        for (Type genericParameterType : genericParameterTypes) {
            System.out.println(genericParameterType);
            // ParameterizedType 参数化类型
            if(genericParameterType instanceof ParameterizedType){
                Type[] actualTypeArguments = ((ParameterizedType) genericParameterType).getActualTypeArguments(); // 先强转，然后获取其真实类型
                for (Type actualTypeArgument : actualTypeArguments) {
                    System.out.println(actualTypeArgument);
                }

            }
        }

        method = Test11.class.getDeclaredMethod("test02", null);
        Type genericReturnType = method.getGenericReturnType(); // 获取泛型的返回值类型
        if(genericReturnType instanceof ParameterizedType){
            Type[] actualTypeArguments = ((ParameterizedType) genericReturnType).getActualTypeArguments();
            for (Type actualTypeArgument : actualTypeArguments) {
                System.out.println(actualTypeArgument);
            }
        }
    }
}
```





### 反射操作注解

- getAnnotations
- getAnnotation

练习ORM

- 了解什么是ORM？

  - Object relationship Mapping -->对象关系映射

    ![image-20210728220246265](.\typora-user-images\image-20210728220246265.png)

  - 类和表结构对应

  - 属性和字段对应

  - 对象和记录对应

- 要求:利用注解和反射完成类和表结构的映射关系

```java
package com.xrl.reflection;

import java.lang.annotation.*;
import java.lang.reflect.Field;

// 练习反射操作注解
public class Test12 {
    public static void main(String[] args) throws ClassNotFoundException, NoSuchFieldException {
        Class c1 = Class.forName("com.xrl.reflection.Student2");


        // 通过反射获取注解
        Annotation[] annotations = c1.getAnnotations(); // 获取类上的注解
        for (Annotation annotation : annotations) {
            System.out.println(annotation); // @com.xrl.reflection.TableX(value=db_student)
        }

        // 获取注解的value的值
        TableX annotation = (TableX) c1.getAnnotation(TableX.class);
        String value = annotation.value(); // db_student
        System.out.println(value);

        // 获得指定的注解
        Field name = c1.getDeclaredField("name");
        FieldX annotation1 = name.getAnnotation(FieldX.class);
        System.out.println(annotation1.columnName());
        System.out.println(annotation1.type());
        System.out.println(annotation1.length());
    }
}


@TableX("db_student")
class Student2{
    @FieldX(columnName="db_id",type="int",length = 10)
    private int id;
    @FieldX(columnName="db_age",type="int",length = 10)
    private int age;
    @FieldX(columnName="db_name",type="varchar",length = 3)
    private String name;

    public Student2() {
    }

    public Student2(int id, int age, String name) {
        this.id = id;
        this.age = age;
        this.name = name;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Student2{" +
                "id=" + id +
                ", age=" + age +
                ", name='" + name + '\'' +
                '}';
    }
}


// 类名的注解
@Target({ElementType.TYPE,ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@interface TableX{
    String value();
}


// 属性的注解
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@interface FieldX{
    String columnName();
    String type();
    int length();
}
```