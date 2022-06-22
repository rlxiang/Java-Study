## Thread

![image-20210729143444284](.\typora-user-images\image-20210729143444284.png)

### Process（进程）& Thread（线程）

在操作系统中运行的程序就是进程，比如我们的QQ，播放器，游戏，IDE等等……

一个进程可以有多个进程，如视频中同时听到声音，看图像，看弹幕，等等

- 说起进程，就不得不说下程序。程序是指令和数据的有序集合，其本身没有任何运行的含义，是一个静态的概念。

- 而进程则是执行程序的一次执行过程，它是一个动态的概念。是系统资源分配的单位

- 通常在一个进程中可以包含若干个线程，当然一个进程中至少有一个线程，不然没有存在的意义。线程是CPU调度和执行的的单位。

  **注意**:很多多线程是模拟出来的，真正的多线程是指有多个cpu，即多核，如服务器。如果是模拟出来的多线程，即在一个cpu的情况下，在同一个时间点，cpu只能执行一个代码，因为切换的很快，所以就有同时执行的错觉。

### 核心概念

- 线程就是独立的执行路径;
- 在程序运行时，即使没有自己创建线程，后台也会有多个线程，如主线程，gc线程（垃圾回收线程）;
- main()称之为主线程，为系统的入口，用于执行整个程序;
- 在一个进程中，如果开辟了多个线程，线程的运行由调度器安排调度，调度器是与操作系统紧密相关的，先后顺序是不能人为的干预的。
- 对同一份资源操作时，会存在资源抢夺的问题，需要加入并发控制;
- 线程会带来额外的开销，如cpu调度时间，并发控制开销。
- 每个线程在自己的工作内存交互，内存控制不当会造成数据不一致

### 线程创建

三种方式创建线程

- 继承Thread类(重点)

  - 自定义线程类继承Thread类

  - 重写**run()**方法，编写线程执行题

  - 创建线程对象，调用**start()**方法启动线程

    ```java
    package com.xrl.thread;
    
    // 创建线程方式一：继承Thread类，重写run()方法，调用start()方法
    
    /*
        总结：注意，线程开启不一定会执行，他是有cpu进行调度的
     */
    public class TestThread1 extends Thread {
        @Override
        public void run() {
            // run方法线程体
            for (int i = 0; i < 20; i++) {
                System.out.println("我在看代码---" + i);
            }
        }
    
        public static void main(String[] args) {
            // main线程，主线程
    
            // 创建一个线程对象
            TestThread1 testThread1 = new TestThread1();
    
            // 调用start方法开启线程
            testThread1.start();
    
    
            for (int i = 0; i < 1000; i++) {
                System.out.println("我在学习多线程===" + i);
            }
        }
    }// 程序输出结果可看到，两个输出是交替进行的
    ```

    ```java
    package com.xrl.thread;
    
    import org.apache.commons.io.FileUtils;
    
    import java.io.File;
    import java.io.IOException;
    import java.net.URL;
    
    // 练习Thread，实现多线程同步下载图片
    public class TestThread2 extends Thread {
    
        private String url; // 网络图片地址
        private String name; // 保存的文件名
    
        public TestThread2(String url, String name) {
            this.url = url;
            this.name = name;
        }
        @Override
        public void run() {
            WebDownLoader.downloader(url, name);
            System.out.println("下载了文件名为" + name);
        }
    
        public static void main(String[] args) {
            TestThread2 t1 = new TestThread2("https://img.iplaysoft.com/wp-content/uploads/2019/free-images/free_stock_photo.jpg", "1.jpg");
            TestThread2 t2 = new TestThread2("http://www.360zimeiti.com/uploads/allimg/170901/1-1FZ1212F0522.png", "2.jpg");
            TestThread2 t3 = new TestThread2("https://photo.16pic.com/00/66/32/16pic_6632560_b.jpg", "3.jpg");
            /*
                理想路径 先下t1，再下t2，再下t3
                但是真实情况是同时执行，输出结果为乱序的
             */
            t1.start();
            t2.start();
            t3.start();
        }
    }
    
    // 下载器
    class WebDownLoader {
        // 下载方法
        public static void downloader(String url, String name) {
            try {
                FileUtils.copyURLToFile(new URL(url), new File(name));
            } catch (IOException e) {
                e.printStackTrace();
                System.out.println("Io异常，downloader方法出现问题");
            }
        }
    }
    //下载了文件名为1.jpg
    //下载了文件名为3.jpg
    // 下载了文件名为2.jpg
    ```

- 实现Runnable接口(重点)(推荐使用Runnable对象，因为Java单继承的局限性)

  1. 定义MyRunnable类实现Runnable接口

  2. 实现run()方法，编写线程执行体

  3. 创建线程对象，调用start()方法启动线程

     ```java
     package com.xrl.thread;
     
     // 创建线程方式2：实现Runnable接口，实现run方法，执行线程需要丢入Runnable接口的实现类，然后调用start方法
     public class TestThread3 implements Runnable {
         @Override
         public void run() {
             // run方法线程体
             for (int i = 0; i < 20; i++) {
                 System.out.println("我在看代码---" + i);
             }
         }
     
         public static void main(String[] args) {
     
             // 创建Runnable接口实现类
             TestThread3 testThread3 = new TestThread3();
     
             // 创建线程对象，通过线程对象来开启我们的线程，代理
     //        Thread thread = new Thread(testThread3);
     //        thread.start();
             new Thread(testThread3).start();
     
             for (int i = 0; i < 1000; i++) {
                 System.out.println("我在学习多线程===" + i);
             }
         }
     }
     ```

     小结：

      	1. 继承Thread类：
      	 - 子类继承Thread类具备多线程能力启动线程:
      	 - 子类对象. start()
      	 - 不建议使用:避免OOP单继承局限性
      	2. 实现Runnable接口
      	- 实现接口Runnable具有多线程能力
      	- 启动线程:传入目标对象+Thread对象.start()
      	- 推荐使用:避免单继承局限性，灵活方便，方便同一个对象被多个线程使用

- 实现Callable接口(了解)

  1. 实现Callable接口，需要返回值类型重写call方法，需要抛出异常
  2. 创建目标对象
  3. 创建执行服务:ExecutorService ser = Executors.newFixedThreadPool(1);
  4. 提交执行:Future<Boolean> result1 = ser.submit(t1);
  5. 获取结果: boolean r1 = result1.get()
  6. 关闭服务:ser.shutdownNow();

  ```java
  package com.xrl.callablle;
  
  import com.xrl.thread.TestThread2;
  import org.apache.commons.io.FileUtils;
  
  import java.io.File;
  import java.io.IOException;
  import java.net.URL;
  import java.util.concurrent.*;
  
  // 线程创建方式三：实现callable接口
  /*
      callable的好处
      1、可以定义返回值
      2、可以抛出异常
   */
  public class TestCallable implements Callable<Boolean> {
      private String url; // 网络图片地址
      private String name; // 保存的文件名
  
      public TestCallable(String url, String name) {
          this.url = url;
          this.name = name;
      }
      @Override
      public Boolean call() {
          WebDownLoader.downloader(url, name);
          System.out.println("下载了文件名为" + name);
          return true;
      }
  
  
      public static void main(String[] args) throws ExecutionException, InterruptedException {
          TestCallable t1 = new TestCallable("https://img.iplaysoft.com/wp-content/uploads/2019/free-images/free_stock_photo.jpg", "1.jpg");
          TestCallable t2 = new TestCallable("http://www.360zimeiti.com/uploads/allimg/170901/1-1FZ1212F0522.png", "2.jpg");
          TestCallable t3 = new TestCallable("https://photo.16pic.com/00/66/32/16pic_6632560_b.jpg", "3.jpg");
          //创建执行服务
          ExecutorService ser = Executors.newFixedThreadPool(3);
          //提交执行
          Future<Boolean> r1 = ser.submit(t1);
          Future<Boolean> r2 = ser.submit(t2);
          Future<Boolean> r3 = ser.submit(t3);
  
          //获取结果:
          boolean rs1 = r1.get();
          boolean rs2 = r2.get();
          boolean rs3 = r3.get();
          //关闭服务
          ser.shutdownNow();
  
  
      }
  }
  
  
  // 下载器
  class WebDownLoader {
      // 下载方法
      public static void downloader(String url, String name) {
          try {
              FileUtils.copyURLToFile(new URL(url), new File(name));
          } catch (IOException e) {
              e.printStackTrace();
              System.out.println("Io异常，downloader方法出现问题");
          }
      }
  }
  ```

### Lamda表达式

- ![image-20210729214138707](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210729214138707.png)希腊字母表中排序第十一位的字母，英语名称为Lambda避免匿名内部类定义过多

- 避免匿名内部类定义过多

- 其实质属于函数式编程的概念

  ```java
  ( params) ->expression[表达式]
  (params) -> statement[语句]
  (params) -> { statements }
  ```

  ```java
  a->System.out.println("i like lambda-->"+a);
  
  new Thread(()->System.out.println("多线程学习")).start();
  ```

- 为什么要使用lambda表达式

  - 避免匿名内部类定义过多
  - 可以让你的代码看起来很简洁
  - 去掉了一堆没有意义的代码，只留下核心的逻辑。

- 也许你会说，我看了Lambda表达式，不但不觉得简洁，反而觉得更乱，看不懂了。那是因为我们还没有习惯，用的多了，看习惯了，就好了。

### Functional Interface (函数式接口)

- 理解Functional lnterface(函数式接口）是学习Java8 lambda表达式的关键所在。

- 函数式接口的定义：

  - 任何接口，如果只包含唯一一个抽象方法，那么它就是一个函数式接口。

    ```java
    public interface Runnable{
        public void run();
    }
    ```

    

  - 对于函数式接口，我们可以通过lambda表达式来创建该接口的对象。

    ```java
    package com.xrl.lambda;
    
    public class TestLambda2 {
    
        public static void main(String[] args) {
            // 1. Lambda表达式简化
            ILove love = (int a)-> {
                System.out.println("i love you-->" + a);
            };
            // 简化lambda表达式的参数类型
            love = (a)->{
                System.out.println("i love you-->" + a);
            };
    
            // 简化lambda表达式的()
            love = a->{
                System.out.println("i love you-->" + a);
            };
    
            // // 简化lambda表达式的{}
            love = a->System.out.println("i love you-->" + a);
    
            /*
                lambda表达式的前提是，接口必须是函数式接口
                lambda表达式只能在只有一行代码的情况下，才能去掉{}，如果是多行，则必须有{}
                多个参数也可以去掉参数类型，要去掉参数类型就都去掉,但是此时必须要加上(),如果只有一个参数，()和参数类型都可以省略掉
             */
    
            love.love(7);
        }
    
    
    
        }
    
    interface ILove{
        void love(int a);
    }
    ```

### 线程的五大状态

![image-20210730102156549](.\typora-user-images\image-20210730102156549.png)

![image-20210730102314789](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210730102314789.png)

#### 线程方法

![image-20210730102443811](.\typora-user-images\image-20210730102443811.png)

#### 停止线程

- 不推荐使用JDK提供的stop()、destory()方法 【已经废弃】

- 推荐线程自己停下来

- 建议使用一个标志位进行终止变量，当flag=false，则终止线程运行

  ```java
  package com.xrl.state;
  
  // 测试线程停止
  /*
      1.建议线程正常停止--->利用次数,不建议死循环。
      2.建议使用标志位--->设置一个标志位
      3.不要使用stop或者destroy等过时或者JDK不建议使用的方法
   */
  public class TestStop implements Runnable {
  
      // 设置一个标志位
      private boolean flag = true;
  
      @Override
      public void run() {
          int i = 0;
          while (flag) {
              System.out.println("run......Thread" + i++);
          }
      }
  
      public void stop(){
          this.flag = false;
      }
  
      public static void main(String[] args) {
  
          TestStop testStop = new TestStop();
          new Thread(testStop).start();
          for (int i = 0; i < 1000; i++) {
              System.out.println("main" + i);
              if(i == 900){
                  testStop.stop();
                  System.out.println("线程该停止了");
              }
          }
      }
  }
  ```

#### 线程修眠

- sleep(时间)指定当前线程阻塞的毫秒数;

- sleep存在异常InterruptedException;

- sleep时间达到后线程进入就绪状态;

- sleep可以模拟网络延时，倒计时等。

- 每一个对象都有一个锁，sleep不会释放锁;

  ```java
  package com.xrl.state;
  
  import java.text.SimpleDateFormat;
  import java.util.Date;
  
  // 模拟倒计时
  public class TestStop2{
  
      public static void main(String[] args) {
          Date startDate = new Date(System.currentTimeMillis());// 获取系统当前时间
          while(true){
              try {
                  System.out.println(new SimpleDateFormat("HH:mm:ss").format(startDate));
                  Thread.sleep(1000);
                  startDate = new Date(System.currentTimeMillis());
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
          }
      }
  
      public static void tenDown() throws InterruptedException {
          int num = 10;
          while(true){
  
              Thread.sleep(1000);
              System.out.println(num--);
              if(num <= 0) break;
          }
      }
  }
  ```

#### 线程礼让

- 礼让线程，让当前正在执行的线程暂停，但不阻塞

- 将线程从运行状态转为就绪状态

- 让cpu重新调度，礼让不一定成功!看CPU心情

  ```java
  package com.xrl.state;
  
  // 礼让不一定成功，看cpu的调度
  public class TestYield {
  
      public static void main(String[] args) {
          MyYield myYield = new MyYield();
          new Thread(myYield,"a").start();
          new Thread(myYield,"b").start();
      }
  }
  
  class MyYield implements Runnable{
  
      @Override
      public void run() {
          System.out.println(Thread.currentThread().getName()+"线程开始执行");
          Thread.yield(); // 礼让
          System.out.println(Thread.currentThread().getName()+"线程执行完毕");
      }
  }
  /*
  	a线程开始执行
      b线程开始执行
      b线程执行完毕
      a线程执行完毕
  */
  ```

#### Join

- Join合并线程，待此线程执行完成后，再执行其他线程，其他线程阻塞

- 可以想象成插队

  ```java
  package com.xrl.state;
  
  // 测试join
  public class TestJoin{
  
  //    @Override
  //    public void run() {
  //        for (int i = 1; i <= 100; i++) {
  //            System.out.println("星悦vip" + i + "上线");
  //        }
  //    }
  
      public static void main(String[] args) throws InterruptedException {
          Thread thread = new Thread(()->{
              for (int i = 1; i <= 1000; i++) {
                  System.out.println("星悦vip" + i + "上线");
              }
          });
  
          
          // 主线程
          for (int i = 0; i < 500; i++) {
              if(i == 200){
                  thread.start();
                  thread.join();
  
              }
              System.out.println("main-->" +i);
          }
      }
  }
  ```

### 线程状态观测

- Thread.State

  线程状态。线程可以处于以下状态之一。

  - NEW（尚未启动的线程处于此状态）
  - RUNNABLE（在Java虚拟机中执行的线程处于此状态）
  - BLOCKED（被阻塞等待监视器锁定的线程处于此状态）
  - WAITING（正在等待另一个线程执行特定动作的线程处于此状态）
  - TIMED WAITING（正在等待另一个线程执行动作达到指定等待时间的线程处于此状态）
  - TERMINATED（已退出的线程处于此状态）

  一个线程可以在给定时间点处于一个状态。这些状态是不反映任何操作系统线程状态的虚拟机状态。

  ```java
  package com.xrl.state;
  
  public class TestState {
      public static void main(String[] args) throws InterruptedException {
          Thread thread = new Thread(()->{
              for (int i = 0; i < 5; i++) {
                  try {
                      System.out.println(Thread.currentThread().getName());
                      Thread.sleep(1000);
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
                  System.out.println("//////////");
              }
          },"小红");
  
          // 观察状态
          Thread.State state = thread.getState();
          System.out.println(state); // NEW
  
          thread.start();
          state = thread.getState();
          System.out.println(state);//RUNNABLE
  
          while(state != Thread.State.TERMINATED){// 只要线程不是终止状态，就一直输出线程的状态
              Thread.sleep(100);
              state = thread.getState();
              System.out.println(state);
          }
      }
  }
  ```

### 线程优先级

- Java提供一个线程调度器来监控程序中启动后进入就绪状态的所有线程，线程调度器按照优先级决定应该调度哪个线程来执行。
- 线程的优先级用数字表示，范围从1~10.
  - Thread.MIN_PRIORITY= 1;
  - Thread.MAX_PRIORITY= 10;
  - Thread.NORM_PRIORITY = 5;
- 使用以下方式改变或获取优先级
  - getPriority() . setPriority(int xxx)

总结 

	1. 优先级的设定建议设置在start()调度前，值越大优先级越高
	2. 优先级低只是以为着获得调度的概率低，并不是优先级低就不会被调用了。这都是看CPU的调度

```java
package com.xrl.state;


// 测试线程优先级
public class TestPriority {
    public static void main(String[] args) {
        System.out.println(Thread.currentThread().getName()+"的优先级为"+Thread.currentThread().getPriority());

        MyPriority myPriority = new MyPriority();
        Thread t1 = new Thread(myPriority,"t1");
        Thread t2 = new Thread(myPriority,"t2");
        Thread t3 = new Thread(myPriority,"t3");
        Thread t4 = new Thread(myPriority,"t4");
        Thread t5 = new Thread(myPriority,"t5");
        Thread t6 = new Thread(myPriority,"t6");

        // 先设置优先级再启动
        t1.start();

        t2.setPriority(1);
        t2.start();

        t3.setPriority(4);
        t3.start();

        t4.setPriority(Thread.MAX_PRIORITY); // 10
        t4.start();

        t5.setPriority(7);
        t5.start();

        t6.setPriority(9);
        t6.start();
    }
}


class MyPriority implements Runnable{

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName()+"的优先级为"+Thread.currentThread().getPriority());
    }
}
```

### 守护(daemon)线程

- 线程分为用户线程和守护线程
- 虚拟机必须确保用户线程(比如我们写的main线程)执行完毕
- 虚拟机不用等待守护线程(比如gc线程)执行完毕
- 如,后台记录操作日志,监控内存,垃圾回收等待.(这些都得用到守护线程)

```java
package com.xrl.state;

// 测试守护线程
// 上帝守护你
public class TestDaemon {
    public static void main(String[] args) {
        God god = new God();
        You you = new You();

        Thread thread = new Thread(god);
        thread.setDaemon(true); // 设置线程为守护线程，默认值为false
        thread.start(); // 守护线程启动

        new Thread(you).start(); // 用户线程启动
    }
}


class God implements Runnable{

    @Override
    public void run() {
        while(true){
            System.out.println("我与你同在");
        }
    }
}


class You implements Runnable{

    @Override
    public void run() {
        for (int i = 0; i < 36500; i++) {
            System.out.println("你一生都开心地活着");
        }
        System.out.println("你没了，无了，唢呐响了");
    }
}
```

### 线程同步（多个线程操作同一个资源）

- 并发：同一个对象被多个线程同时操作
- 处理多线程问题时,多个线程访问同一个对象，并且某些线程还想修改这个对象.这时候我们就需要线程同步.线程同步其实就是一种等待机制,多个需要同时访问此对象的线程进入这个对象的等待池形成队列,等待前面线程使用完毕，下一个线程再使用
- 实现线程同步需要有**队列和锁（锁是每个对象都有的，保证安全）**
- 由于同一进程的多个线程共享同一块存储空间，在带来方便的同时,也带来了访问冲突问题,为了保证数据在方法中被访问时的正确性,在访问时加入**锁机制**，**synchronized(同步)**,当一个线程获得对象的排它锁，独占资源，其他线程必须等待，使用后释放锁即可.存在以下问题:
  - 一个线程持有锁会导致其他所有需要此锁的线程挂起;
  - 在多线程竞争下﹐加锁﹐释放锁会导致比较多的上下文切换和调度延时,引起性能问题;
  - 如果一个优先级高的线程等待一个优先级低的线程释放锁会导致优先级倒置，引起性能问题．

#### 同步方法

- 由于我们可以通过private关键字来保证数据对象只能被方法访问,所以我们只需要针对方法提出一套机制，这套机制就是synchronized关键字，它包括两种用法:synchronized方法和synchronized块．

  ```java
  同步方法：public synchronized void method(int args){}
  ```

- synchronized方法控制对“对象”的访问,每个对象对应一把锁，每个synchronized方法都必须获得调用该方法的对象的锁才能执行﹐否则线程会阻塞，方法一旦执行﹐就独占该锁，直到该方法返回才释放锁，后面被阻塞的线程才能获得这个锁,继续执行

  - 缺陷：若将一个大的方法申明为synchronized将会影响效率

    ![image-20210730165301944](.\typora-user-images\image-20210730165301944.png)

#### 同步代码块

- 同步块:synchronized (Obj ) {}，obj称之为同步监视器
  - Obj可以是任何对象﹐但是推荐使用共享资源作为同步监视器
  - 同步方法中无需指定同步监视器﹐因为同步方法的同步监视器就是this ,就是这个对象本身﹐或者是class 
- 同步监视器的执行过程
  1. 第一个线程访问,锁定同步监视器﹐执行其中代码.
  2. 第二个线程访问﹐发现同步监视器被锁定，无法访问．
  3. 第一个线程访问完毕﹐解锁同步监视器.
  4. 第二个线程访问,发现同步监视器没有锁﹐然后锁定并访问
- 总之synchronized锁的就是变化的量，需要增删改的对象

### 死锁

多个线程互相抱着对方需要的资源，然后形成僵持

```java
package com.xrl.deadLock;
// 一个类如果有静态成员，那么不管这个类new了多少个实例，这个静态成员都只有一
//个，如果这些实例中有操作了这个静态成员的值则其他类在访问改静态值时访问的是操作后的值
// 死锁：多个线程互相抱有对方想要的资源，然后形成互相僵持
public class DeadLock {
    public static void main(String[] args) {
        Makeup g1 = new Makeup(0, "小红");
        Makeup g2 = new Makeup(1, "小芳");
        g1.start();
        g2.start();
    }
}

// 口红
class Lipstick {
}

// 镜子
class Mirror {
}

class Makeup extends Thread {
    // 资源只有一份
    static Lipstick lipstick = new Lipstick();
    static Mirror mirror = new Mirror();

    int choice;//选择
    String name;// 使用化妆品的人

    Makeup(int choice, String name) {
        this.choice = choice;
        this.name = name;
    }

    @Override
    public void run() {
        // 化妆
        try {
            makeup();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    // 化妆
    private void makeup() throws InterruptedException {
        if(choice==0){
            synchronized (lipstick){ // 获得口红的锁
                System.out.println(this.getName()+"获得口红的锁");
                Thread.sleep(1000);
                synchronized (mirror){ // 一秒钟后想获取镜子的锁
                    System.out.println(this.getName()+"获得镜子的锁");
                }
            }
        }else{
            synchronized (mirror){ // 获得镜子的锁
                System.out.println(this.getName()+"获得镜子的锁");
                Thread.sleep(2000);
                synchronized (lipstick){ // 两秒钟后想获取口红的锁
                    System.out.println(this.getName()+"获得口红的锁");
                }
            }
        }
    }
}
```

产生死锁的四个必要条件：

- 互斥条件:一个资源每次只能被一个进程使用。
- 请求与保持条件:一个进程因请求资源而阻塞时，对已获得的资源保持不放。
- 不剥夺条件:进程已获得的资源，在末使用完之前，不能强行剥夺。
- 循环等待条件:若干进程之间形成一种头尾相接的循环等待资源关系。

上面列出了死锁的四个必要条件，我们只要想办法破其中的任意一个或多个条件就可以避免死锁发生



### Lock(锁)

- 从JDK 5.0开始，Java提供了更强大的线程同步机制——通过显式定义同步锁对象来实现同步。同步锁使用Lock对象充当
- java.util.concurrent.locks.Lock接口是控制多个线程对共享资源进行访问的工具。锁提供了对共享资源的独占访问，每次只能有一个线程对Lock对象加锁，线程开始访问共享资源之前应先获得Lock对象
- ReentrantLock类实现了Lock，它拥有与synchronized相同的并发性和内存语义，在实现线程安全的控制中，比较常用的是ReentrantLock，可以显式加锁、释放锁。

ReentrantLock的一般写法为

```java
private final ReentrantLock lock = new ReentrantLock();
public void m(){
    try{
        lock.lock(); // 加锁
        ……
    }finally{
        lock.unlock();// 解锁，如果同步代码有异常，要讲unlock写在finally里面
    }
}
```



```java
package com.xrl.gaoji;

import java.util.concurrent.locks.ReentrantLock;

public class TestLock {
    public static void main(String[] args) {
        TestLock2 testLock2 = new TestLock2();
        new Thread(testLock2).start();
        new Thread(testLock2).start();
        new Thread(testLock2).start();
    }
}

class TestLock2 implements Runnable {

    // 定义lock锁
    private final ReentrantLock lock = new ReentrantLock();

    int ticketNums = 10;

    @Override
    public void run() {
        while (true) {
            try {
                lock.lock(); // 加锁
                if (ticketNums > 0) {
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(ticketNums--);
                }else{
                    break;
                }
            } finally {
                lock.unlock();// 解锁
            }
        }
    }
}
```

### synchronized 与 Lock 的对比

- Lock是显式锁（手动开启和关闭锁，别忘记关闭锁) synchronized是隐式锁，出了作用域自动释放
- Lock只有代码块锁，synchronized有代码块锁和方法锁
- 使用Lock锁，JVM将花费较少的时间来调度线程，性能更好。并且具有更好的扩展性(提供更多的子类)
- 优先使用顺序:
  - Lock >同步代码块(已经进入了方法体，分配了相应资源)>同步方法（在方法体之外)

### 线程通信

- 应用场景:生产者和消费者问题
  - 假设仓库中只能存放一件产品，生产者将生产出来的产品放入仓库,消费者将仓库中产品取走消费.
  - 如果仓库中没有产品，则生产者将产品放入仓库﹐否则停止生产并等待，直到仓库中的产品被消费者取走为止.
  - 如果仓库中放有产品,则消费者可以将产品取走消费，否则停止消费并等待,直到仓库中再次放入产品为止.
  - ![image-20210730194041653](.\typora-user-images\image-20210730194041653.png)

**这是一个线程同步问题，生产者和消费者共享同一个资源，并且生产者和消费者之间相互依赖，互为条件.**

- 对于生产者，没有生产产品之前,要通知消费者等待.而生产了产品之后，又需要马上通知消费者消费

- 对于消费者,在消费之后，要通知生产者已经结束消费，需要生产新的产品以供消费.

- 在生产者消费者问题中,仅有synchronized是不够的

  - synchronized可阻止并发更新同一个共享资源，实现了同步
  - synchronized不能用来实现不同线程之间的消息传递(通信)

- java提供了几个方法解决线程之间的通信问题

  ![image-20210730194502346](.\typora-user-images\image-20210730194502346.png)

  注意：**均是Object类的方法，都只能在同步方法或者同步代码块中使用,否则会抛出异常lllegalMonitorStateException**

- 解决方式一：

  - 并发协作模型“生产者/消费者模式”--->管程法

    - 生产者:负责生产数据的模块(可能是方法﹐对象，线程,进程);

    - 消费者︰负责处理数据的模块(可能是方法﹐对象，线程﹐进程);

    - 缓冲区:消费者不能直接使用生产者的数据﹐他们之间有个“缓冲区"

      **生产者将生产好的数据放入缓冲区,消费者从缓冲区拿出数据**

- 解决方式二：

  - 并发协作模型“生产者/消费者模式”--->信号灯法
    - 设置一个标志位，并判断其真假
    - 如果为真就等待
    - 如果为假就通知另外一个线程

管程法

```java
package com.xrl.gaoji;

// 测试生产者消费者模型-->利用缓冲区解决：管程法
public class TestPC {
    public static void main(String[] args) {
        SynContainer synContainer = new SynContainer();
        new Productor(synContainer).start();
        new Consumer(synContainer).start();
    }
}


// 生产者
class Productor extends Thread {
    SynContainer synContainer;

    public Productor(SynContainer synContainer) {
        this.synContainer = synContainer;
    }

    @Override
    public void run() {
        for (int i = 1; i <= 100; i++) {
            synContainer.push(new Chicken(i));
            System.out.println("生产了" + i + "只鸡");
        }
    }
}

// 消费者
class Consumer extends Thread {
    SynContainer synContainer;

    public Consumer(SynContainer synContainer) {
        this.synContainer = synContainer;
    }
    // 消费
    @Override
    public void run() {
        for (int i = 1; i <= 100; i++) {
            System.out.println("消费了-->"+synContainer.pop().id+"只鸡");
        }
    }
}
// 产品
class Chicken {
    int id;

    public Chicken(int id) {
        this.id = id;
    }
}
// 容器
class SynContainer {
    // 需要一个容器大小
    Chicken[] chickens = new Chicken[10];
    //容器计数器
    int count = 0;
    // 生产这放入产品
    public synchronized void push(Chicken chicken) {
        // 如果容器满了就等待消费者消费
        if (count == chickens.length -1) {
            // 通知消费者进行消费，生产者等待
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        // 如果没有满，就需要加入产品
        chickens[count++] = chicken;

        // 可以通知消费者消费了
        this.notifyAll();
    }
    // 消费者消费产品
    public synchronized Chicken pop() {
        // 判断能否消费
        if (count == 0) {
            // 等待生产者生产，消费者等待
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        // 如果可以消费
        Chicken chicken = chickens[--count];
        // 通知生产者进行生产
        this.notifyAll();
        return chicken;
    }
}
```

信号灯法

```java
package com.xrl.gaoji;

// 生产者消费者：信号灯法
public class TestPc2 {
    public static void main(String[] args) {
        TV tv = new TV();
        new Player(tv).start();
        new Watcher(tv).start();
    }
}


// 生产者:演员
class Player extends Thread {
    TV tv;

    public Player(TV tv) {
        this.tv = tv;
    }

    @Override
    public void run() {
        for (int i = 0; i < 20; i++) {
            if (i % 2 == 0){
                this.tv.play("java多线程");
            }else{
                this.tv.play("动漫");
            }
        }
    }
}

// 消费者： 观众
class Watcher extends Thread {
    TV tv;

    public Watcher(TV tv) {
        this.tv = tv;
    }

    @Override
    public void run() {
        for (int i = 0; i < 20; i++) {
            tv.watch();
        }
    }
}

// 产品 --》 节目
class TV {
    // 演员表演，观众等待 T
    // 观众观看，演员等待 F
    String voice; // 表演的节目
    boolean flag = true;

    // 表演
    public synchronized void play(String voice) {
        if (!flag) {
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println("演员表演了：" + voice);
        // 通知观众观看
        this.notifyAll();
        this.voice = voice;
        flag = !this.flag;
    }

    // 观看
    public synchronized void watch() {
        if (flag) {
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println("观看了" + voice);
        // 通知演员表演
        this.notifyAll();
        flag = !this.flag;
    }
}
```

### 线程池

- 背景:经常创建和销毁、使用量特别大的资源，比如并发情况下的线程，对性能影响很大。

- 思路:提前创建好多个线程，放入线程池中，使用时直接获取，使用完放回池中。可以避免频繁创建销毁、实现重复利用。类似生活中的公共交通工具。

- 好处：

  - 提高响应速度（减少了创建新线程的时间)

  - 降低资源消耗（重复利用线程池中线程，不需要每次都创建)

  - 便于线程管理(....)

    - corePoolSize:核心池的大小
    - maximumPoolSize:最大线程数
    - keepAliveTime:线程没有任务时最多保持多长时间后会终止

    ![image-20210731111738614](.\typora-user-images\image-20210731111738614.png)