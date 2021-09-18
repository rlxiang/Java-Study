多线程进阶==》JUC并发编程

## 1、什么是JUC

![image-20210807221206448](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210807221206448.png)

**有些真实的业务如果只用普通的线程代码(Thread)去执行效率不高**

**Runnable** 没有返回值、效率相比callable 相对低下

![image-20210807221533392](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210807221533392.png)

![image-20210807221725344](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210807221725344.png)



## 2、线程与进程

进程：一个程序，QQ.exe Music.exe 程序的集合

一个进程往往可以包含多个线程，至少包含一个！

Java默认有几个线程？2个 main GC

线程：开了一个进程Typora，则在这里打字，自动保存都是线程负责的

**Java真的可以开启线程吗？**开不了

```java
    public synchronized void start() {
        /**
         * This method is not invoked for the main method thread or "system"
         * group threads created/set up by the VM. Any new functionality added
         * to this method in the future may have to also be added to the VM.
         *
         * A zero status value corresponds to state "NEW".
         */
        if (threadStatus != 0)
            throw new IllegalThreadStateException();

        /* Notify the group that this thread is about to be started
         * so that it can be added to the group's list of threads
         * and the group's unstarted count can be decremented. */
        group.add(this);

        boolean started = false;
        try {
            start0();
            started = true;
        } finally {
            try {
                if (!started) {
                    group.threadStartFailed(this);
                }
            } catch (Throwable ignore) {
                /* do nothing. If start0 threw a Throwable then
                  it will be passed up the call stack */
            }
        }
    }

// 本地方法，底层是c++，java无法操作硬件
    private native void start0();
```

>并发、并行

并发(多线程操作同一个资源)

- CPU一核，模拟出来多条线程，快速交替

并行(多个人一起行走)

- CPU多核，多个线程可以同时执行；如果此时要提高效率，我们需要使用线程池操作

```java
public class Test1 {
    public static void main(String[] args) {
        // 获取CPU的核数
        System.out.println(Runtime.getRuntime().availableProcessors());
    }
}
```

并发编程的本质：**充分利用CPU的资源**



>线程有几个状态

```java
 public enum State {
     
        // 新生
        NEW,

        // 运行
        RUNNABLE,

        // 阻塞
        BLOCKED,

   		// 等待，死死的等
        WAITING,

        // 超时等待，过时不候
        TIMED_WAITING,

       // 终止
        TERMINATED;
    }
```



>wait与sleep的区别

**1、来自不同的类**

wait ==》Object类中的方法

sleep==> Thread类中的方法

**2、关于锁的释放**

wait会释放锁，sleep是抱着锁睡觉的，不会释放

**3、使用范围不同**

wait必须在同步代码块中使用

sleep在任何地方都可以使用

**4、是否需要捕获异常**（当然只要是线程都会有中断异常）

wait不需要捕获异常

sleep必须捕获异常



## 3、Lock锁(重点)

>传统的Synchronized锁

```java
package com.xiang.demo01;

/**
 * 真正的多线程开发，公司中的开发要求 降低耦合性
 * 线程就是一个单独的资源类，没有任何附属的操作
 * 1、属性、方法
 */
public class SaleTicketDemo01 {
    public static void main(String[] args) {
        // 并发，多线程操作同一个资源类,即将资源类丢入线程
        Ticket ticket = new Ticket();
        new Thread(()->{
            for (int i = 0; i < 60; i++) {
                ticket.sale();
            }
        },"A").start();
        new Thread(()->{
            for (int i = 0; i < 60; i++) {
                ticket.sale();
            }
        },"B").start();
        new Thread(()->{
            for (int i = 0; i < 60; i++) {
                ticket.sale();
            }
        },"C").start();
    }
}

class Ticket {
    // 属性、方法
    private int number = 30;

    // 买票的方式
    /*
            synchronized 本质就是 队列，锁
            比如 食堂打饭，学生要排成队列，然后刷卡(就是拿锁)，取到饭完成之后(就是释放锁)，下一个在等待的人继续之前的操作
     */
    public synchronized void sale() {
        if (number > 0)
            System.out.println(Thread.currentThread().getName()+"卖出了第"+(number--)+"票,剩余"+number);
    }
}
```



>Lock 接口

![image-20210808105017257](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210808105017257.png)

![image-20210808105419197](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210808105419197.png)

![image-20210808105716680](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210808105716680.png)

公平锁：十分公平，讲究先来后到

- 比如有个线程3h执行完，有个线程3s执行完，然后3h的线程排在了3s的前面，那必须等到3h结束完，3s的才能执行

**非公平锁：十分不公平，可以插队（默认方式），根据cpu调度来**



```java
package com.xiang.demo01;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 真正的多线程开发，公司中的开发要求 降低耦合性
 * 线程就是一个单独的资源类，没有任何附属的操作
 * 1、属性、方法
 */
public class SaleTicketDemo02 {
    public static void main(String[] args) {
        // 并发，多线程操作同一个资源类,即将资源类丢入线程
        Ticket ticket2 = new Ticket();
        new Thread(()->{for (int i = 0; i < 40; i++) ticket2.sale();},"A").start();
        new Thread(()->{for (int i = 0; i < 40; i++) ticket2.sale();},"B").start();
        new Thread(()->{for (int i = 0; i < 40; i++) ticket2.sale();},"C").start();
    }
}

// Lock三部曲
/*
    1、Lock lock = new ReentrantLock();
    2、在需要锁的业务代码块上加锁 lock.lock(); 然后业务代码写在try中，该怎么写就怎么写
    3、finally中解锁 lock.unlock(); // 解锁
 */
class Ticket2 {
    // 属性、方法
    private int number = 30;
    Lock lock = new ReentrantLock();

    public void sale() {
        lock.lock();// 加锁
        try {
            // 业务代码
            if (number > 0)
                System.out.println(Thread.currentThread().getName() + "卖出了第" + (number--) + "票,剩余" + number);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock(); // 解锁
        }
    }
}
```



>Synchronized 和 Lock 区别

1、Synchronized 内置的Java关键字， Lock是一个Java接口

2、Synchronized  无法判断获取锁的状态，Lock 可以判断是否获取到了锁

3、Synchronized  会自动释放锁，lock 必须要手动释放锁！如果不释放就会死锁

4、Synchronized  会出现如果线程1获得了锁，然后此时阻塞了，那么线程2就是一直等待下去，然而lock不会

5、Synchronized  可重入锁，不可以中断的，非公平；Lock，可重入锁，可以判断锁，非公平，但是可以设置成公平锁

6、Synchronized  适合锁少量的代码同步问题，Lock适合锁大量的同步代码！



>锁是什么，如何判断锁的是谁





## 4、生产者消费者问题

面试：单例模式、排序算法、生产者消费者、死锁

>生产者消费者问题 Synchronized  版

```java
package com.xiang.pc;

/**
 * 线程之间的通信问题：生产者消费者问题！ 等待与通知唤醒
 * 线程交替执行 A B 操作同一个变量  num = 0
 * A num + 1
 * B num - 1
 */
public class A {
    public static void main(String[] args) {
        Data data = new Data();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "A").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "B").start();
    }
}

// 判断等待，业务，通知
class Data { // 数字 资源类
    private int number = 0;

    // + 1
    public synchronized void increment() throws InterruptedException {
        if (number != 0) {
            // 等待
            this.wait();
        }
        number++; // 业务
        System.out.println(Thread.currentThread().getName() + "==>" + number);
        // 通知其他线程，我的+1操作完毕了
        this.notifyAll();
    }

    // - 1
    public synchronized void decrement() throws InterruptedException {
        if (number == 0) {
            // 等待
            this.wait();
        }
        number--; // 业务
        System.out.println(Thread.currentThread().getName() + "==>" + number);
        // 通知其他线程，我的-1操作完毕了
        this.notifyAll();
    }
}
```

上面的这个版本，如果再加几个线程就会产生问题,虚假唤醒

![image-20210808135510694](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210808135510694.png)

**将 if 判断改为 while 判断就可以了**

```java
package com.xiang.pc;

/**
 * 线程之间的通信问题：生产者消费者问题！ 等待与通知唤醒
 * 线程交替执行 A B 操作同一个变量  num = 0
 * A num + 1
 * B num - 1
 */
public class A {
    public static void main(String[] args) {
        Data data = new Data();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "A").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "B").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "C").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "D").start();
    }
}

// 判断等待，业务，通知
class Data { // 数字 资源类
    private int number = 0;

    // + 1
    public synchronized void increment() throws InterruptedException {
        while (number != 0) {
            // 等待
            this.wait();
        }
        number++; // 业务
        System.out.println(Thread.currentThread().getName() + "==>" + number);
        // 通知其他线程，我的+1操作完毕了
        this.notifyAll(); // 此时当前的线程没有释放锁，而只是唤醒其他线程，唤醒的线程如果要执行也是要等cpu来进行调度才可以
    }

    // - 1
    public synchronized void decrement() throws InterruptedException {
        while (number == 0) {
            // 等待
            this.wait();
        }
        number--; // 业务
        System.out.println(Thread.currentThread().getName() + "==>" + number);
        // 通知其他线程，我的-1操作完毕了
        this.notifyAll();
    }
}
```



>JUC版生产者消费者模式

通过Lock找到Condition

![image-20210808142939467](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210808142939467.png)

![image-20210808143118190](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210808143118190.png)

代码实现：

```java
package com.xiang.pc;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class B {
    public static void main(String[] args) {
        Data2 data = new Data2();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "A").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "B").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "C").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "D").start();
    }
}

// 判断等待，业务，通知
class Data2 { // 数字 资源类
    private int number = 0;
    Lock lock = new ReentrantLock();
    Condition condition = lock.newCondition();

    // + 1
    public void increment() throws InterruptedException {
        lock.lock();
        try {
            while (number != 0) {
                condition.await(); // 等待
            }
            number++;
            System.out.println(Thread.currentThread().getName() + "==>" + number);
            condition.signalAll(); // 唤醒
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }

    }

    // - 1
    public synchronized void decrement() throws InterruptedException {
        lock.lock();
        try {
            while (number == 0) {
                condition.await();
            }
            number--; // 业务
            System.out.println(Thread.currentThread().getName() + "==>" + number);
            condition.signalAll();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

上面的代码执行只能随机地通知唤醒线程，如何可以做到精准的通知唤醒线程。



>Condition 精准通知唤醒线程

代码测试：

```java
package com.xiang.pc;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 要求 A执行完执行B，B执行完执行C，C执行完执行A
 */
public class C {
    public static void main(String[] args) {
        Data3 data = new Data3();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                data.printA();
            }
        }, "A").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                data.printB();
            }
        }, "B").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                data.printC();
            }
        }, "C").start();
    }
}

class Data3 { // 资源类
    private Lock lock = new ReentrantLock();
    private Condition condition1 = lock.newCondition(); // 同步监视器可以一个监视器只监视一个
    private Condition condition2 = lock.newCondition();
    private Condition condition3 = lock.newCondition();
    private int number = 1; // 1A 执行 2B 执行 3C 执行

    public void printA() {
        lock.lock();
        try {
            // 业务 判断 -> 执行-> 通知
            while (number != 1) {
                condition1.await();
            }
            System.out.println(Thread.currentThread().getName()+"AAAAAAAAAAAAAA");
            number = 2;
            condition2.signal(); // 唤醒condition2
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void printB() {
        lock.lock();
        try {
            // 业务 判断 -> 执行-> 通知
            while (number != 2) {
                condition2.await();
            }
            System.out.println(Thread.currentThread().getName()+"BBBBBBBBBBBBBBBBBBBBBB");
            number = 3;
            condition3.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void printC() {
        lock.lock();
        try {
            // 业务 判断 -> 执行-> 通知
            while (number != 3) {
                condition3.await();
            }
            System.out.println(Thread.currentThread().getName()+"CCCCCCCCCCCCCCCCCCCC");
            number = 1;
            condition1.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```



## 5、8锁现象

如何判断锁的是谁？

```java
package com.xiang.lock8;

import java.util.concurrent.TimeUnit;

/**
 *  8锁就是关于锁的8个问题
 *  1、标准情况下，两个线程是先打印 发短信还是 打电话？ 结果为：先发短信，再打电话
 *  2、sendSms延迟4s，两个线程是先打印 发短信还是 打电话？ 结果为：先发短信，再打电话
 */
public class Test1 {
    public static void main(String[] args) {
        Phone phone = new Phone();
        new Thread(()->{
            phone.sendSms();
        },"A").start();

        try {
            // 停止1秒
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        new Thread(()->{
            phone.call();
        },"B").start();
    }
}


class Phone{

    // synchronized 用在方法上，锁的是对象是方法的调用者
    // 这里两个方法用的是同一把锁，所以谁先拿到谁先执行
    public synchronized void sendSms(){
        try {
            // 停止1秒
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("发短信");
    }

    public synchronized void call(){
        System.out.println("打电话");
    }
}
```



```java
package com.xiang.lock8;
/*
    3、增加一个普通方法后，是先执行发短信还是先执行hello，结果为hello先执行
    4、两个对象，都调用同步方法，先打电话还是先发短信？ 先打电话

 */
import java.util.concurrent.TimeUnit;

public class Test2 {
    public static void main(String[] args) {
        // 两个对象，两个调用者，两把锁
        Phone2 phone1 = new Phone2();
        Phone2 phone2 = new Phone2();
        new Thread(() -> {
            phone1.sendSms();
        }, "A").start();

        try {
            // 停止1秒
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        new Thread(() -> {
            phone2.call();
        }, "B").start();
    }
}


class Phone2 {

    // synchronized 用在方法上，锁的是对象是方法的调用者
    // 这里两个方法用的是同一把锁，所以谁先拿到谁先执行
    public synchronized void sendSms() {
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("发短信");
    }

    public synchronized void call() {
        System.out.println("打电话");
    }

    // 没有锁，不是同步方法，不受锁的影响
    public void hello(){
        System.out.println("hello");
    }
}


```



```java
package com.xiang.lock8;

import java.util.concurrent.TimeUnit;

/**
 * 5、增加两个静态的同步方法，只有一个对象，先打印 发短信，还是打电话？  发短信
 * 6、同样两个静态的同步方法，有两个个对象，先打印 发短信，还是打电话 ？ 发短信
 */
public class Test3 {
    public static void main(String[] args) {
        Phone3 phone1 = new Phone3();
        Phone3 phone2 = new Phone3();
        new Thread(() -> {
            phone1.sendSms();
        }, "A").start();

        try {
            // 停止1秒
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        new Thread(() -> {
            phone2.call();
        }, "B").start();
    }
}


class Phone3 {

    // synchronized 用在方法上，锁的是对象是方法的调用者
    // synchronized在静态方法上或者静态属性上，锁的是Class对象
    public synchronized static void sendSms() {
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("发短信");
    }

    public synchronized static void call() {
        System.out.println("打电话");
    }

}

```



```java
package com.xiang.lock8;

import java.util.concurrent.TimeUnit;

/**
 * 1、一个静态同步方法，一个普通同步方法，，一个对象， 先打印发短信还是先打印打电话？ 先打电话
 * 1、一个静态同步方法，一个普通同步方法，，两个对象， 先打印发短信还是先打印打电话？ 先打电话
 */
public class Test4{
    public static void main(String[] args) {
        Phone4 phone1 = new Phone4();
        Phone4 phone2 = new Phone4();
        new Thread(() -> {
            phone1.sendSms();
        }, "A").start();

        try {
            // 停止1秒
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        new Thread(() -> {
            phone2.call();
        }, "B").start();
    }
}


class Phone4 {

    // 静态同步方法，锁的是Class 类模板
    public synchronized static void sendSms() {
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("发短信");
    }

    // 普通同步方法，锁的是调用者
    public synchronized  void call() {
        System.out.println("打电话");
    }

}
```

>小结

锁只会锁两个东西  一个是调用者，一个是class类模板，锁的东西是一样的，就是顺序执行的。



## 6、不安全的集合类

>List 不安全

```java
package com.xiang.unsafe;

import java.util.*;
import java.util.concurrent.CopyOnWriteArrayList;

// java.util.ConcurrentModificationException  并发修改异常
public class ListTest {
    public static void main(String[] args) {
        // 并发下ArrayList 是不安全的
        /**
         * 解决方案：
         * 1、List<String> list = new Vector<>();，但是最好不要这样回答，因为Vector在ArrayList前面出来的，
         * 2、List<String> list = Collections.synchronizedList(new ArrayList<>()); 将集合转换成安全的
         * 以上两种都不是JUC的解决方法
         * List<String> list = new CopyOnWriteArrayList<>(); // 这才是JUC下的解决方式
         *
         */

        // CopyOnWrite 写入时复制， COW思想，是计算机程序设计领域的一种优化策略
        // 多个线程调用的时候，list是固定的，读取的时候没有什么影响，但是在写入时可能会产生覆盖的现象
        // 为了避免这种现象，使用CopyOnWrite
        List<String> list = new CopyOnWriteArrayList<>();
        for (int i = 1; i <= 100; i++) {
            new Thread(() -> {
                list.add(UUID.randomUUID().toString().substring(0, 5));
                System.out.println(list);
            }, String.valueOf(i)).start();
        }
    }
}

```



>Set不安全

```java
package com.xiang.unsafe;

import java.util.Collections;
import java.util.HashSet;
import java.util.Set;
import java.util.UUID;
import java.util.concurrent.ConcurrentSkipListSet;

/**
 *  同理可证： ConcurrentModificationException
 *  1、  Set<String>set = Collections.synchronizedSet(new HashSet<>());
 *  2、Set<String>set = new ConcurrentSkipListSet<>();
 */
public class SetTest {
    public static void main(String[] args) {
//        Set<String>set = new HashSet<>();
//        Set<String>set = Collections.synchronizedSet(new HashSet<>());
        Set<String>set = new ConcurrentSkipListSet<>();
        for (int i = 1; i <= 100 ; i++) {
            new Thread(()->{
                set.add(UUID.randomUUID().toString().substring(0,5));
                System.out.println(set);
            },String.valueOf(i)).start();
        }
    }
}

```

HashSet的底层

```java
public HashSet() {
    map = new HashMap<>();
}
// 他的add方法就是把map中的key给put进去了
 public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
// PRESENT
private static final Object PRESENT = new Object(); // 常量，不变的
```



>Map不安全

```java
package com.xiang.unsafe;

import java.util.Collections;
import java.util.HashMap;
import java.util.Map;
import java.util.UUID;
import java.util.concurrent.ConcurrentHashMap;

// ConcurrentModificationException
public class MapTest {
    public static void main(String[] args) {
//        Map<String, String> map = new HashMap<>();
//        Map<String, String> map = Collections.synchronizedMap(new HashMap<>());
        Map<String, String> map = new ConcurrentHashMap<>();
        for (int i = 1; i <= 100; i++) {
           new Thread(()->{
               map.put(Thread.currentThread().getName(), UUID.randomUUID().toString().substring(0,5));
               System.out.println(map);
           },String.valueOf(i)).start();
        }

    }
}

```



## 7、Callable

![image-20210812150323441](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210812150323441.png)

1、可以有返回值

2、可以抛出异常

3、方法不同 call()

>代码测试



![image-20210812151206915](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210812151206915.png)



![image-20210812151123626](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210812151123626.png)

![image-20210812151346525](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210812151346525.png)

![image-20210812152552883](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210812152552883.png)



```java
package com.xiang.callable;


import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

public class CallableTest {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        // new Thread(new Runnable()).start();
        // new Thread(new FutureTask<V>()).start();
        // new Thread(new FutureTask<V>(Callable)).start();
        // FutureTask 实现了 Runnable
//        new Thread().start();   // 线程启动的唯一方式
        MyThread thread = new MyThread();
        FutureTask futureTask = new FutureTask(thread); // 适配类
        new Thread(futureTask,"A").start();
        new Thread(futureTask,"B").start(); // 结果会被缓存，效率高，这里只会打印一个call
        Object o = futureTask.get(); // 获取值，这个方法可能会产生阻塞,把它放在最后或者使用异步通信就可以了
        System.out.println(o);
    }
}

class MyThread implements Callable<Integer>{

    @Override
    public Integer call() throws Exception {
        System.out.println("call");
        // 如果这里有耗时操作，那么获取返回值就会等待
        return 1024;
    }
}
```



细节：

1、有缓存

2、结果可能需要等待，会阻塞



## 8、常用的辅助类

### 8.1、CountDownLatch（线程减法计数器）

![image-20210812153725030](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210812153725030.png)

```java
package com.xiang.add;

import java.util.concurrent.CountDownLatch;

// 计数器
public class CountDownLatchDemo {
    public static void main(String[] args) throws InterruptedException {
        // 总数为6，该类经常在必须等任务执行完后，在进行后续的操作时使用
        CountDownLatch countDownLatch = new CountDownLatch(6);
        for (int i = 1; i <= 6 ; i++) {
            new Thread(()->{
                System.out.println(Thread.currentThread().getName()+"Go Out");
                countDownLatch.countDown();// -1 操作
            },String.valueOf(i)).start();
        }
        countDownLatch.await(); // 等待计数器归零才会执行后续代码
        System.out.println("Close Door");

    }
}

```

原理：

 ==countDownLatch.countDown();== // 数量减一

 ==countDownLatch.await();== // 等待计数器归零才会执行后续代码

每次执行countDown()，数量就会-1，假设计数器为0， countDownLatch.await()就会被唤醒，继续执行后面的代码

### 8.2、CyclicBarrier（线程加法计数器）

![image-20210812155341637](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210812155341637.png)

加法计数器

```java
package com.xiang.add;

import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

public class CyclicBarrierDemo {
    public static void main(String[] args) {
        /**
         *  集齐7颗龙珠 召唤神龙
         */
        // 召唤龙珠的线程
        CyclicBarrier cyclicBarrier = new CyclicBarrier(7,()->{
            System.out.println("神龙召唤成功");
        });
        for (int i = 1; i <= 7; i++) {
            final int temp = i;
            new Thread(()->{
                System.out.println(Thread.currentThread().getName()+"收集了"+temp+"颗龙珠");
                try {
                    cyclicBarrier.await(); // 等待，直到计数器变成7才会执行召唤神龙
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            },String.valueOf(i)).start();
        }
    }
}

```



### 8.3、Semaphore

Semaphore：信号量

![image-20210812160318493](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210812160318493.png)



比如有6个车，只有3个车位，限制了流量

```java
package com.xiang.add;

import java.util.concurrent.Semaphore;
import java.util.concurrent.TimeUnit;

// 抢车位
public class SemaphoreDemo {
    public static void main(String[] args) {
        // 线程数量：停车位,限流的时候使用
        Semaphore semaphore = new Semaphore(3);
        for (int i = 1; i <= 6; i++) {
            new Thread(()->{
                // acquire() 得到
                try {
                    semaphore.acquire(); // 抢到一个车位
                    System.out.println(Thread.currentThread().getName()+"抢到车位");
                    TimeUnit.SECONDS.sleep(2);
                    System.out.println(Thread.currentThread().getName()+"离开车位");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    semaphore.release(); // 释放
                }
            },String.valueOf(i)).start();
        }
    }
}

```

**原理**

==semaphore.acquire();== 获得，如果已经满了，就会等待释放

==semaphore.release();==  释放，会将当前信号释放1，然后唤醒等待的线程

作用：多个共享的资源互斥使用！并发限流，控制线程的最大运行数量！



## 9、读写锁

**ReadWriteLock**

![image-20210812163907042](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210812163907042.png)



```java
package com.xiang.rw;

import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

/**
 *  独占锁(写锁) 一次只能被一个线程持有
 *  共享锁(读锁) 多个线程同时持有
 *  ReadWriteLock
 *  读-读    可以共存
 *  读-写    不能共存
 *  写-写    不能共存
 */
public class ReadWriteLockDemo {
    public static void main(String[] args) {
        MyCacheLock myCache = new MyCacheLock();
        // 写
        for (int i = 1; i <= 5; i++) {
            final int temp = i;
            new Thread(() -> {
                myCache.put(temp + "", temp + "");
            }, String.valueOf(i)).start();
        }

        // 写
        for (int i = 1; i <= 5; i++) {
            final int temp = i;
            new Thread(() -> {
                myCache.get(temp + "");
            }, String.valueOf(i)).start();
        }
    }
}


// 加锁的
class MyCacheLock {
    private volatile Map<String, Object> map = new HashMap<>();
    private ReadWriteLock readWriteLock = new ReentrantReadWriteLock(); // 读写锁，更加细粒度的控制

    // 存，写的时候，希望只有一个线程可以写入
    public void put(String key, Object value) {
        readWriteLock.writeLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "写入" + key);
            map.put(key, value);
            System.out.println(Thread.currentThread().getName() + "写入OK");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            readWriteLock.writeLock().unlock();
        }
    }

    // 取，读的时候，所有人都可以读
    public void get(String key) {
        readWriteLock.readLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "读取" + key);
            Object o = map.get(key);
            System.out.println(Thread.currentThread().getName() + "读取OK");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            readWriteLock.readLock().unlock();
        }
    }
}

/**
 * 自定义缓存
 */
class MyCache {
    private volatile Map<String, Object> map = new HashMap<>();

    // 存，写
    public void put(String key, Object value) {
        System.out.println(Thread.currentThread().getName() + "写入" + key);
        map.put(key, value);
        System.out.println(Thread.currentThread().getName() + "写入OK");
    }

    // 取，读
    public void get(String key) {
        System.out.println(Thread.currentThread().getName() + "读取" + key);
        Object o = map.get(key);

        System.out.println(Thread.currentThread().getName() + "读取OK");
    }
}
```



## 10、阻塞队列

![image-20210812170841068](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210812170841068.png)



阻塞队列

![image-20210812171009608](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210812171009608.png)



**BlockingQueue**  阻塞队列

![image-20210812173526294](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210812173526294.png)

![image-20210812173709847](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210812173709847.png)

什么时候用阻塞队列：多线程并发处理，线程池

**四组API**

| 方式     | 抛出异常  | 有返回值,不抛出异常 | 阻塞等待 | 超时等待  |
| -------- | --------- | ------------------- | -------- | --------- |
| 添加     | add()     | offer()             | put()    | offer(,,) |
| 移除     | remove()  | poll()              | take()   | poll(,)   |
| 判断队首 | element() | peek()              | -        | -         |



```java
/*
	抛出异常
*/
public static void test1() {
        // 设置队列初始大小
        ArrayBlockingQueue blockingQueue = new ArrayBlockingQueue<>(3);
        System.out.println(blockingQueue.add("a"));
        System.out.println(blockingQueue.add("b"));
        System.out.println(blockingQueue.add("c"));

    	System.out.println(blockingQueue.element()); // 返回当前队首元素
        // java.lang.IllegalStateException: Queue full，队列如果满了，还入队就报异常
//        System.out.println(blockingQueue.add("d"));
        System.out.println(blockingQueue.remove());
        System.out.println(blockingQueue.remove());
        System.out.println(blockingQueue.remove());

        // java.util.NoSuchElementException(队列如果空了，还继续出队的话 就会抛出这个异常)
        System.out.println(blockingQueue.remove());


    }
```



```java
   /*
        有返回值，没有异常
     */
    public static void test2(){
        ArrayBlockingQueue<Object> blockingQueue = new ArrayBlockingQueue<>(3);

        System.out.println(blockingQueue.offer("a"));
        System.out.println(blockingQueue.offer("b"));
        System.out.println(blockingQueue.offer("c"));
        System.out.println(blockingQueue.offer("d")); // 如果队列满了继续插入会返回false

        System.out.println(blockingQueue.peek()); // 返回队首元素

        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll()); // 如果队列空了继续弹出会返回null
    }
```



```java
    /*
        等待，阻塞（一直等待）
     */
    public static void test3() throws InterruptedException {
        ArrayBlockingQueue<Object> blockingQueue = new ArrayBlockingQueue<>(3);
        // 没有返回值，如果队中元素满了，继续添加会一直阻塞等待
        blockingQueue.put("a");
        blockingQueue.put("b");
        blockingQueue.put("c");
//        blockingQueue.put("d");

        // 如果队列为空时，继续弹出元素，会阻塞，一直等待
        System.out.println(blockingQueue.take());
        System.out.println(blockingQueue.take());
        System.out.println(blockingQueue.take());
        System.out.println(blockingQueue.take());
    }
```



```java
 /*
        等待，阻塞（超时等待，如果时间到了就不等待了）
     */
     public static void test4() throws InterruptedException {
         ArrayBlockingQueue<Object> blockingQueue = new ArrayBlockingQueue<>(3);
         System.out.println(blockingQueue.offer("a"));
         System.out.println(blockingQueue.offer("b"));
         System.out.println(blockingQueue.offer("c"));
         //blockingQueue.offer("d",2, TimeUnit.SECONDS); // 如果队列满了，继续添加值，会等待相应的时间，超时后会停止
         System.out.println("==============================");
         System.out.println(blockingQueue.poll());
         System.out.println(blockingQueue.poll());
         System.out.println(blockingQueue.poll());
         blockingQueue.poll(2,TimeUnit.SECONDS);// 如果队空，继续弹出操作，会等待相应的时间，超时后会停止
     }
```



>SynchronousQueue  同步队列

该队列没有容量，进去一个元素之后，必须等待其被取出，才能继续存放一个元素，队列里最多只能有一个元素！

put() 存     take() 取



```java
package com.xiang.bq;

import java.util.concurrent.BlockingQueue;
import java.util.concurrent.SynchronousQueue;
import java.util.concurrent.TimeUnit;

/**
 * 同步队列
 * 和其他的BlockingQueue不一样，SynchronousQueue 不存储元素
 * put了一个元素，必须等待里面的元素先被take取出来才能再次put值
 */
public class SynchronousQueueDemo {
    public static void main(String[] args) {
        BlockingQueue<String> blockingQueue = new SynchronousQueue<>();
        new Thread(()->{
            try {
                System.out.println(Thread.currentThread().getName()+"put 1");
                blockingQueue.put("1");
                System.out.println(Thread.currentThread().getName()+"put 2");
                blockingQueue.put("2");
                System.out.println(Thread.currentThread().getName()+"put 3");
                blockingQueue.put("3");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"T1").start();
        new Thread(()->{
            try {
                TimeUnit.SECONDS.sleep(3);
                System.out.println(Thread.currentThread().getName()+"=>"+blockingQueue.take());
                TimeUnit.SECONDS.sleep(3);
                System.out.println(Thread.currentThread().getName()+"=>"+blockingQueue.take());
                TimeUnit.SECONDS.sleep(3);
                System.out.println(Thread.currentThread().getName()+"=>"+blockingQueue.take());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"T2").start();
    }
}

```



## 11、线程池

线程池：三大方法、7大参数、4中拒绝策略

>池化技术

程序的运行，本质：占用系统的资源！优化资源的使用！=>池化技术

线程池，连接池、内存池、对象池……

池化技术：事先准备好一些资源，有人要用，就直接来拿，用完后归还即可

创建线程和销毁都是很需要资源的



**线程池的好处**

1、降低资源的消耗

2、提高响应的速度

3、方便管理

==线程复用，可以控制最大并发数、管理线程==



>线程池：三大方法

![image-20210812204031141](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210812204031141.png)



```java
package com.xiang.pool;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

// Executors 工具类，三大方法
public class Demo01 {
    public static void main(String[] args) {
//        ExecutorService threadPool = Executors.newSingleThreadExecutor();// 创建一个只有一个线程的线程池
//        ExecutorService threadPool = Executors.newFixedThreadPool(5); // 创建指定大小的线程池
         ExecutorService threadPool =Executors.newCachedThreadPool(); // 创建任意大小的线程池
        try {
            for (int i = 0; i < 100; i++) {
                // 使用线程池创建线程
                threadPool.execute(()->{
                    System.out.println(Thread.currentThread().getName()+" OK");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 线程池使用完，要关闭
            threadPool.shutdown();
        }
    }
}

```



>7大参数

源码分析

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}

public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }

public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }

// 本质：ThreadPoolExecutor()
public ThreadPoolExecutor(int corePoolSize, // 核心线程池大小
                              int maximumPoolSize, // 最大核心线程池大小
                              long keepAliveTime, // 超时了没有人调用就会释放
                              TimeUnit unit, // 超时单位
                              BlockingQueue<Runnable> workQueue, // 阻塞队列
                              ThreadFactory threadFactory, // 线程工厂，创建线程的，一般不用动
                              RejectedExecutionHandler handler// 拒绝策略) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```



![image-20210812210045284](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210812210045284.png)

比如在银行办理业务，虽然有5个窗口，但是最开始这五个窗口不是一直开的，只有1,2号窗口常开(这个就是核心线程池的大小)，当1,2号窗口有人时，此时又有人来办理业务，这些人就会去候客区等待（就是阻塞队列），普通情况下当1,2号窗口的人办理完业务后，候客区的人再过去1,2号窗口办理业务，但是有时候会出现1,2号窗口满了，候客区满了，银行还在进人，此时剩下的三个窗口就会陆续开放知道线程池达到最大数量，如果此时还有人要进来，则这些人要么等待要么离开（这就是拒绝策略），如果当银行里的所有人办理完业务了，过了一段时间还是没有人来，那么3,4,5号窗口就会再次关闭（超时等待，过了一定时间就停止）



>手动创建一个线程池



```java
package com.xiang.pool;


import java.util.concurrent.*;

// Executors 工具类，三大方法

/**
 * new ThreadPoolExecutor.AbortPolicy() // 银行满了，还有人进来，不处理这个人，然后抛出异常
 * new ThreadPoolExecutor.CallerRunsPolicy() // 从哪个线程进来的，就让那个线程去处理
 * new ThreadPoolExecutor.DiscardPolicy() // 队列满了，不会抛出异常，会丢掉线程
  new ThreadPoolExecutor.DiscardOldestPolicy() // 队列满了，不会抛出异常，会先尝试和最早的线程竞争，如果竞争失败就丢到线程
 */
public class Demo01 {
    public static void main(String[] args) {
         ExecutorService threadPool = new ThreadPoolExecutor(
                 2,
                 Runtime.getRuntime().availableProcessors(),
                 3,
                 TimeUnit.SECONDS,
                 new LinkedBlockingDeque<>(3),
                 Executors.defaultThreadFactory(),
                 new ThreadPoolExecutor.DiscardOldestPolicy() // 队列满了，不会抛出异常，会先尝试和最早的线程竞争，如果竞争失败就丢到线程
         );
        try {
            // 线程池的最大承载= max + Deque
            for (int i = 1; i <= 9; i++) {
                // 使用线程池创建线程
                threadPool.execute(()->{
                    System.out.println(Thread.currentThread().getName()+" OK");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 线程池使用完，要关闭
            threadPool.shutdown();
        }
    }
}

```











>4种拒绝策略

![image-20210812211001517](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210812211001517.png)



```java
/**
 * new ThreadPoolExecutor.AbortPolicy() // 银行满了，还有人进来，不处理这个人，然后抛出异常
 * new ThreadPoolExecutor.CallerRunsPolicy() // 从哪个线程进来的，就让那个线程去处理
 * new ThreadPoolExecutor.DiscardPolicy() // 队列满了，不会抛出异常，会丢掉线程
  new ThreadPoolExecutor.DiscardOldestPolicy() // 队列满了，不会抛出异常，会先尝试和最早的线程竞争，如果竞争失败就丢到线程
 */
```



>小结与扩展

线程池的最大大小如何去设置

1、CPU密集型，根据cpu的核数去设置大小，cpu是几何就设置为几，这样可以保证cpu的最高效率，获取cpu核数

2、IO密集型：当我们的程序中有十分耗IO的线程时，有多少个这样的线程就设置成比这些线程多的值，一般是两倍，比如有15个十分耗						IO的线程，那就将线程池最大大小设置为30







## 12、四大函数式接口（必须掌握）

新时代的程序员：lambda表达式、链式编程（函数式编程）、函数式接口、Stream流失计算

>函数式接口：只有一个方法的接口，并标记了@FunctionalInterface

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
// 超级多的 @FunctionalInterface
// 可以简化编程模型，在新版本的框架中大量使用
// foreach(消费者类的函数式接口)
```



![image-20210812215402440](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210812215402440.png)



>Function 函数式接口



![image-20210812220245202](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210812220245202.png)



```java
package com.xiang.function;

import java.util.function.Function;

/**
 *  Function 函数式接口，有一个输入参数，有一个输出
 *  只要是 函数式接口，都可以使用lambda表达式简化
 */
public class Demo01 {
    public static void main(String[] args) {
//        Function<String,String> function = new Function<String, String>() {
//            @Override
//            public String apply(String str) {
//                return str;
//            }
//        };
        Function<String,String> function=(str)->{return str;};
        System.out.println(function.apply("asd"));
    }
}

```





>断定型接口

![image-20210812220838835](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210812220838835.png)



```java
package com.xiang.function;

import java.util.function.Predicate;

/**
 *  断定型接口，有一个参数值，返回值只能是 布尔值
 */
public class Demo02 {
    public static void main(String[] args) {
//        Predicate<String> predicate = new Predicate<String>() {
//            // 判断字符串是否为空
//            @Override
//            public boolean test(String str) {
//                return str.isEmpty();
//            }
//        };
        Predicate<String> predicate = (String str)->{
            return str.isEmpty();
        };
        System.out.println(predicate.test("asd "));
    }
}

```





>Consumer 消费型接口

![image-20210812221445877](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210812221445877.png)



```java
package com.xiang.function;

import java.util.function.Consumer;

/**
 * Consumer 消费型接口，只有一个输出参数，没有返回值
 */
public class Demo03 {
    public static void main(String[] args) {
//        Consumer<String> consumer = new Consumer<String>() {
//            @Override
//            public void accept(String str) {
//                System.out.println(str);
//            }
//        };
        Consumer<String> consumer = (String str)->{
            System.out.println(str);
        };
        consumer.accept("asd");
    }
}

```



>Supplier  供给型接口

![image-20210812221928985](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210812221928985.png)

```java
/**
 * Supplier 供给型接口，没有参数，只有返回类型
 */
public class Demo04 {
    public static void main(String[] args) {
//        Supplier<Integer> supplier = new Supplier<Integer>() {
//            @Override
//            public Integer get() {
//                return 1024;
//            }
//        };
        Supplier<Integer> supplier = ()->{return 1024;};
        System.out.println(supplier.get());
    }
}
```



## 13、Stream流式计算

>什么是Stream流式计算

大数据：存储+计算

集合、MySQL本质就是存储东西的；

计算都应该交给流来操作！



![image-20210813100725737](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210813100725737.png)



```java
package com.xiang.stream;

import java.util.Arrays;
import java.util.List;

/**
 * 题目要求:一分钟内完成此题，只能用一行代码实现!
 * 现在有5个用户!筛选:
 * 1、ID必须是偶数
 * 2、年龄必须大于23岁
 * 3、用户名转为大写字母
 * 4、用户名字母倒着排序
 * 5、只输出一个用户!
 */
public class Test {
    public static void main(String[] args) {
        User user1 = new User(1, "a", 21);
        User user2 = new User(2, "b", 22);
        User user3 = new User(3, "c", 23);
        User user4 = new User(4, "d", 24);
        User user5 = new User(6, "e", 25);
        // 集合用来存储数据
        List<User> list = Arrays.asList(user1, user2, user3, user4, user5);
        // 计算交给流
        // lambda表达式，链式编程，函数式接口、Stream流式计算
        list.stream().filter((u) -> {return u.getId() % 2 == 0;})
                .filter((u)->{return u.getAge()>23;})
                .map((u)->{return u.getName().toUpperCase();})
                .sorted((u1,u2)->{return u2.compareTo(u1);})
                .limit(1)
                .forEach(System.out::println);
    }
}
```



## 14、ForkJoin

>什么是ForkJoin

ForkJoin 在JDK 1.7，并行执行任务！提高效率，只有在大数据量时才能显现出来他的作用

![image-20210813102603551](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210813102603551.png)



>ForkJoin特点：工作窃取

当多条线程在同时执行时，如果有的线程先执行完成了，这时候该线程会去执行其他线程的人物，这就是工作窃取，可以提高效率。

这里面维护的都是双端队列

![image-20210813102826032](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210813102826032.png)



>ForkJoin

![image-20210813105302598](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210813105302598.png)

![image-20210813105214357](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210813105214357.png)



```java
package com.xiang.forkjoin;

import java.util.concurrent.RecursiveTask;

/**
 * 求和计算的任务！
 * 如何使用forkjoin
 * 1、通过forkjoinPool 来执行
 * 2、计算任务 forkjoinPool。execute(ForkJoinTask task)
 * 3、计算类继承RecursiveTask
 */
public class ForkJoinDemo extends RecursiveTask<Long> {
    private Long start;
    private Long end;

    // 临界值
    private Long temp = 1000L;

    public ForkJoinDemo(Long start, Long end) {
        this.start = start;
        this.end = end;
    }

    @Override
    protected Long compute() {
        if (end - start < temp) {
            Long sum = 0L;
            for (Long i = start; i <= end; i++) {
                sum += i;
            }
            return sum;
        }else{ // forkjoin
            long mid = (start + end) / 2;   // 中间值
            ForkJoinDemo task1 = new ForkJoinDemo(start,mid);
            task1.fork(); // 拆分任务，把任务压入线程队列
            ForkJoinDemo task2 = new ForkJoinDemo(mid+1,end);
            task2.fork(); // 拆分任务，把任务压入线程队列
            return task1.join() + task2.join(); // 合并
        }
    }
}

```

测试

```java
package com.xiang.forkjoin;

import java.util.concurrent.ExecutionException;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.ForkJoinTask;
import java.util.stream.LongStream;

public class Test {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
//        test1(); // 7642
//        test2(); // 6954
        test3(); // 174
    }

    public static void test1() {
        Long sum = 0L;
        long start = System.currentTimeMillis();
        for (Long i = 1L; i <= 10_0000_0000L; i++) {
            sum += i;
        }
        long end = System.currentTimeMillis();
        System.out.println("sum=" + sum + "时间：" + (end - start));
    }

    public static void test2() throws ExecutionException, InterruptedException {
        long start = System.currentTimeMillis();
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        ForkJoinTask<Long> task = new ForkJoinDemo(0L, 10_0000_0000L);
        ForkJoinTask<Long> submit = forkJoinPool.submit(task);
        Long sum = submit.get();
        long end = System.currentTimeMillis();
        System.out.println("sum=" + sum + "时间：" + (end - start));
    }

    public static void test3()  {
        long start = System.currentTimeMillis();
        // Stream 并行流 range表示的是开区间(,),rangeClosed(,]
        long sum = LongStream.rangeClosed(0L, 10_0000_0000L).parallel().reduce(0, Long::sum);
        long end = System.currentTimeMillis();
        System.out.println("sum=" + sum + "时间：" + (end - start));
    }
}
```



## 15、异步回调

>Future 设计初衷：对将来的某个事件的结果进行建模

![image-20210813140922108](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210813140922108.png)



```java
package com.xiang.future;

import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.TimeUnit;
import java.util.function.BiConsumer;

/**
 * 异步调用 ：CompletableFuture
 * 1、异步执行
 * 2、成功回调
 * 3、实拍回调
 */
public class Demo01 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        // 没有返回值的 runAsync 异步回调
//        CompletableFuture<Void> completableFuture = CompletableFuture.runAsync(()->{
//            try {
//                TimeUnit.SECONDS.sleep(2);
//            } catch (InterruptedException e) {
//                e.printStackTrace();
//            }
//            System.out.println(Thread.currentThread().getName()+" runAsync==>Void");
//        });
//        System.out.println("111111111");
//        completableFuture.get();   // 获取返回结果

        // 有返回值的 supplyAsync 异步回调
        // 成功或者失败的回调
        // 失败就返回错误信息
        CompletableFuture<Integer> completableFuture = CompletableFuture.supplyAsync(() -> {
            System.out.println(Thread.currentThread().getName() + " supplyAsync==>Integer");
            int i = 1 / 0;
            return 1024;
        });
        System.out.println(completableFuture.whenComplete((t, u) -> { // 编译时会出现两种状态，成功或者失败
            System.out.println("t= " + t); // 如果成功就正常的返回结果，如果有错误会返回null
            System.out.println("u=" + u); // 有错误就会返回错误信息，没有错误就会返回null
        }).exceptionally((e) -> {
            System.out.println(e.getMessage());
            return 233; // 如果错误了就可以获取错误的返回结果
        }).get());

    }
}

```



## 16、JMM

>请你谈谈对Volatile的理解

Volatile 是Java 虚拟机提供的**轻量级的同步机制**

1、保证可见性

2、==不保证原子性==

3、禁止指令重排



>什么是JMM

JMM：Java内存模型，不存在的东西，是一种概念，约定



**关于JMM的一些同步约定**

1、线程解锁前，必须把共享变量(存放在主存中的变量，因为每条线程都是单独执行的，有自己的工作区间，线程在工作时会把主存中的							共享变量拷贝一份到工作区间中)刷新会主存

2、线程加锁前，必须读取主存中最新的值到工作空间

3、加锁和解锁必须是同一把锁



JMM就是线程在操作**工作内存**与**主存**的规定

**JMM中有8种约定(操作)**

![image-20210813145033622](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210813145033622.png)



![image-20210813145229051](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210813145229051.png)





**内存交互操作有8种，虚拟机实现必须保证每一个操作都是原子的，不可在分的（对于double和long类型的变量来说，load、store、read和write操作在某些平台上允许例外）**

- lock   （锁定）：作用于主内存的变量，把一个变量标识为线程独占状态
- unlock （解锁）：作用于主内存的变量，它把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定
- read  （读取）：作用于主内存变量，它把一个变量的值从主内存传输到线程的工作内存中，以便随后的load动作使用
- load   （载入）：作用于工作内存的变量，它把read操作从主存中变量放入工作内存中
- use   （使用）：作用于工作内存中的变量，它把工作内存中的变量传输给执行引擎，每当虚拟机遇到一个需要使用到变量的值，就会使用到这个指令
- assign （赋值）：作用于工作内存中的变量，它把一个从执行引擎中接受到的值放入工作内存的变量副本中
- store  （存储）：作用于主内存中的变量，它把一个从工作内存中一个变量的值传送到主内存中，以便后续的write使用
- write 　（写入）：作用于主内存中的变量，它把store操作从工作内存中得到的变量的值放入主内存的变量中

　　

**JMM对这八种指令的使用，制定了如下规则：**

- 不允许read和load、store和write操作之一单独出现。即使用了read必须load，使用了store必须write
- 不允许线程丢弃他最近的assign操作，即工作变量的数据改变了之后，必须告知主存
- 不允许一个线程将没有assign的数据从工作内存同步回主内存
- 一个新的变量必须在主内存中诞生，不允许工作内存直接使用一个未被初始化的变量。就是怼变量实施use、store操作之前，必须经过assign和load操作
- 一个变量同一时间只有一个线程能对其进行lock。多次lock后，必须执行相同次数的unlock才能解锁
- 如果对一个变量进行lock操作，会清空所有工作内存中此变量的值，在执行引擎使用这个变量前，必须重新load或assign操作初始化变量的值
- 如果一个变量没有被lock，就不能对其进行unlock操作。也不能unlock一个被其他线程锁住的变量
- 对一个变量进行unlock操作之前，必须把此变量同步回主内存

![image-20210813150140470](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210813150140470.png)



上面的模型就会存在一个问题，线程B将主存中的值改变之后，线程A里的主存的值没有变，如何让线程A也知道主存中的值变了，使用Volatile。

## 17、Volatile

>1、保证可见性

```java
package com.xiang.tvolatile;

import java.util.concurrent.TimeUnit;

public class JMMDemo {
    // 如果不加 volatile 程序就会死循环
    // 加 volatile 保证了程序的可见性
    public volatile  static int num = 0;

    public static void main(String[] args) { // main线程
        /**
         * 没有在变量上加volatile时，该线程是不知道主存中的值是改变了的
         */
        new Thread(() -> {
            while (num == 0) {

            }
        }).start();

        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        num = 1;
        System.out.println(num);
    }
}

```



>2、不保证原子性

原子性：不可分割

线程A在执行任务时，是不能被打扰的，也不能被分割，要么同时成功，要么同时失败。

```java
package com.xiang.tvolatile;

// volatile 不保证原子性
public class VDemo02 {
    // volatile 不保证原子性
    public volatile static int num = 0;

    public static void add() {
        num++;
    }

    public static void main(String[] args) {
        // 理论上是2w
        for (int i = 0; i < 20; i++) {
            new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    add();
                }
            }).start();
        }
        while (Thread.activeCount() > 2) {
            Thread.yield();
        }
        System.out.println(Thread.currentThread().getName() + " " + num);
    }
}
```

**如果不加lock，不加Synchronized怎么保证原子性**



![image-20210813152218879](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210813152218879.png)

使用原子类，解决原子性问题

![image-20210813152338710](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210813152338710.png)

```java
package com.xiang.tvolatile;

import java.util.concurrent.atomic.AtomicInteger;

// volatile 不保证原子性
public class VDemo02 {
    // volatile 不保证原子性
    // 将其改为原子类的 Integer
    public volatile static AtomicInteger num = new AtomicInteger();

    public static void add() {
//        num++; // 本身就不是一个原子操作
        num.getAndIncrement(); // +1操作，该方便底层是native方法。
    }

    public static void main(String[] args) {
        // 理论上是2w
        for (int i = 0; i < 20; i++) {
            new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    add();
                }
            }).start();
        }
        while (Thread.activeCount() > 2) {
            Thread.yield();
        }
        System.out.println(Thread.currentThread().getName() + " " + num);
    }
}

```

这些原子类的底层都是和操作系统交互的



>指令重排

什么是指令重排：**你写的程序，计算机并不是按照我们写的顺序那样去执行的。**

源代码-->编译器优化重排-->指令并行也可能会重排-->内存系统也会重排--> 执行

==**处理器在进行指令重排的时候，考虑：数据之间的依赖性！**==

```java
// 对于单条线程，指令重排不对程序造成结果的影响的情况
int x = 1; // 1
int y = 2; // 2
x = x + 5; //3
y = x * x // 4
    
我们所期望的执行顺序：1234，但是可能执行的时候会变成 2134   1324
但是不可能是 4123，因为4依赖1
```



多条线程时，指令重排对程序造成结果的影响的情况 ，下面表格中 a b x y 的默认值都是0

| 线程A | 线程 B |
| ----- | ------ |
| x=a   | y=b    |
| b=1   | a=2    |

正常结果为 x = 0, y = 0; 但是可能因为指令重排变成下面这样

| 线程A | 线程 B |
| ----- | ------ |
| b=1   | a=2    |
| x=a   | y=b    |

指令重排造成的诡异结果： x = 2; y = 1

**volatile可以避免指令重排**

在系统中有个内存屏障，它是cpu指令。

内存屏障的作用：

1、保证特定的操作的执行顺序

2、可以保证某些变量的内存可见性(volatile的可见性就是这样的）

![image-20210813161559385](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210813161559385.png)

即，加了volatile后，会在其上下加一个内存屏障，可以保证避免指令重排的现象产生

内存屏障在单例模式中使用的最多



## 18、彻底玩转单例模式

饿汉式   DCL懒汉式，深究！

>饿汉式



```java
package com.xiang.single;

// 饿汉式单例
public class Hungry {

    // 可能会造成浪费空间
    byte[] data1 = new byte[1024 * 1024];
    byte[] data2 = new byte[1024 * 1024];
    byte[] data3 = new byte[1024 * 1024];
    byte[] data4 = new byte[1024 * 1024];


    private Hungry() {

    }

    private static final Hungry HUNGRY = new Hungry();

    public static Hungry getInstance() {
        return HUNGRY;
    }
}
```



DCL懒汉式 + volatile

```java
package com.xiang.single;


// 懒汉式单例模式
public class LazyMan {
    private LazyMan() {
        System.out.println(Thread.currentThread().getName() + " OK");
    }

    private volatile static LazyMan lazyMan;

    // 双重检查锁模式的 懒汉式单列  DCL懒汉式
    public static LazyMan getInstance() {
        if (lazyMan == null) {
            synchronized (LazyMan.class) {
                if (lazyMan == null){
                    lazyMan = new LazyMan(); // 这不是一个原子性操作
                    /**
                     * 1、分配内存空间
                     * 2、执行这个构造方法，初始化对象
                     * 3、把这个对象指向这个空间
                     *
                     *  执行顺序期望的是 123
                     *  线程A执行顺序132，如果此时A没有完成构造，线程B进来了
                     *  由于此时lazyMan不为空（存储lazyMan本身的空间不为空），
                     *  就会直接返回，但是此时真正存储对象的空间中没有东西，因为还没有完成构造进行初始化，
                     *  此时就需要加volatile保证可见,即DCL+volatile保证完全的安全
                     */
                }
            }

        }
        return lazyMan;
    }

    // 单线程下，上面这样的写法是可以的
    // 但是在多线程就不可以了
    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {

            new Thread(() -> {
                System.out.println(getInstance());
            }).start();
        }
    }
}
```



枚举中的单例模式是不可以被反射破坏的

```java
package com.xiang.single;


import java.lang.reflect.Constructor;

// 枚举中的没有空参构造,即使我们显示定义了空参构造，编译也不会通过
public enum EnumSingle {
    INSTANCE;

    public EnumSingle getInstance() {
        return INSTANCE;
    }

}

class Test {
    public static void main(String[] args) throws Exception {
        EnumSingle instance1 = EnumSingle.INSTANCE;
        Constructor<EnumSingle> declaredConstructor = EnumSingle.class.getDeclaredConstructor(String.class, int.class);
        declaredConstructor.setAccessible(true);
        EnumSingle instance2 = declaredConstructor.newInstance();
        System.out.println(instance1 == instance2);

    }
}
```





## 19、深入理解CAS

>什么是CAS

```java
package com.xiang.cas;

import java.util.concurrent.atomic.AtomicInteger;

public class CASDemo {
    // CAS： compareAndSet 比较并交换
    public static void main(String[] args) {
        AtomicInteger atomicInteger = new AtomicInteger(2021);

        //public final boolean compareAndSet(int expect, int update) 第一个参数为期望，第二个为更新
        //意思就是 如果达到期望就更新，否则就不更新,CAS 是CPU的并发原语，就是cpu的指令
        System.out.println(atomicInteger.compareAndSet(2021, 2022));// true
        System.out.println(atomicInteger.get());//2022
     
        System.out.println(atomicInteger.compareAndSet(2021, 2022));// false
        System.out.println(atomicInteger.get());//2022
    }
}
```



>Unsafe类

![image-20210813194024266](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210813194024266.png)



![image-20210813194309438](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210813194309438.png)

```java
compareAndSwapInt(var1,var2,var5,var5+var4)
该函数的意思就是 如果var对象的var中的值是var5，就把这个值修改为var4+var5.

```



![image-20210813194700335](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210813194700335.png)

CAS：比较当前的工作内存中的值和主内存中的值，如果这个值是期望的，就执行操作，否则就一直循环（底层是自旋锁）

**缺点：**

1、循环会耗时，但是相对java而言还是很快，因为是内存操作

2、一次性只能保证一个共享变量的原子性

3、ABA问题



>CAS: ABA 问题（狸猫换太子）

![image-20210813201048608](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210813201048608.png)

ABA：比如有两条线程同时对某一资源进行操作，其中线程1中进行CAS(1,2)，线程2中进行了两次CAS，第一次把CAS(1,3),第二次CAS(3,1), 此时对于线程1来说 A还是1，线程1就被欺骗了 

```java
package com.xiang.cas;

import java.util.concurrent.atomic.AtomicInteger;

public class CASDemo {
    // CAS： compareAndSet 比较并交换
    public static void main(String[] args) {
        AtomicInteger atomicInteger = new AtomicInteger(2021);

        //public final boolean compareAndSet(int expect, int update) 第一个参数为期望，第二个为更新
        //意思就是 如果达到期望就更新，否则就不更新,CAS 是CPU的并发原语，就是cpu的指令

        // =====================捣乱的线程===================
        System.out.println(atomicInteger.compareAndSet(2021, 2022));
        System.out.println(atomicInteger.get());

        System.out.println(atomicInteger.compareAndSet(2022, 2021));
        System.out.println(atomicInteger.get());
        // ====================期望的线程======================
        System.out.println(atomicInteger.compareAndSet(2021, 2023));
        System.out.println(atomicInteger.get());
    }
}

```

如何解决ABA，使用带版本号的原子操作，即原子引用



## 20、原子引用

>解决ABA问题，引入原子引用，对应的思想就是乐观锁

带版本号的原子操作

```java
package com.xiang.cas;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.atomic.AtomicStampedReference;

public class CASDemo {
    // AtomicStampedReference的泛型是包装类，要注意对象的引用问题
    static AtomicStampedReference<Integer> stampedReference = new AtomicStampedReference<>(1, 1);

    public static void main(String[] args) {
//        AtomicInteger atomicInteger = new AtomicInteger(2021);
        new Thread(() -> {
            int stamp = stampedReference.getReference(); // 获取当前版本号
            System.out.println("a1==>" + stamp);
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(stampedReference.compareAndSet(1, 2,
                    stampedReference.getStamp(), stampedReference.getStamp() + 1));

            System.out.println("a2==>" + stampedReference.getStamp());

            System.out.println(stampedReference.compareAndSet(2, 1,
                    stampedReference.getStamp(), stampedReference.getStamp() + 1));

            System.out.println("a3==>" + stampedReference.getStamp());
        }, "a").start();

        new Thread(() -> {
            int stamp = stampedReference.getStamp();
            System.out.println("b1==>" + stamp);
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(stampedReference.compareAndSet(1, 6, stamp, stamp + 1));
            System.out.println("b2==>"+stampedReference.getStamp());
        }, "b").start();

    }
}
```



**Integer使用了对象缓存机制，默认范围是-128~127，推荐使用静态工厂方法valueOf获取对象实例，而不是new，,因为valueOf使用缓存，而new一定会创建新的对象分配新的内存空间;**

![image-20210813204119497](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210813204119497.png)



## 21、各种锁的理解

### 1、公平锁，非公平锁

公平锁：非常公平，不能插队，必须先来后到！

非公平锁：非常不公平，可以插队，后来的线程可以提前执行（不过是lock还是synchronized都是默认为非公平锁），保证效率

```java
 public ReentrantLock() {
        sync = new NonfairSync();
    }

 public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
```



### 2、可重入锁

可重入锁(递归锁)不管是 lock还是synchronized都是可重入锁

![image-20210813205213982](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210813205213982.png)



>synchronized 版

```java
package com.xiang.lock;

import java.util.concurrent.TimeUnit;

//synchronized
public class Demo01 {
    public static void main(String[] args) {
        Phone phone = new Phone();
        new Thread(()->{
            phone.sms();

        },"A").start();

        new Thread(()->{
            phone.sms();
        },"B").start();
    }
}

class Phone {
    public synchronized void sms() {
        System.out.println(Thread.currentThread().getName() + "sms");
        call(); // 这里也有锁
    }
    public synchronized void call(){
        System.out.println(Thread.currentThread().getName() + "call");
    }
}
```



>lock 版

```java
package com.xiang.lock;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Demo02 {
    public static void main(String[] args) {
        Phone2 phone = new Phone2();
        new Thread(()->{
            phone.sms();

        },"A").start();

        new Thread(()->{
            phone.sms();
        },"B").start();
    }
}

class Phone2 {
    Lock lock = new ReentrantLock();
    public  void sms() {
        lock.lock();// lock.lock();与lock.unlock();必须配对，不然就会死在里面
        try {
            System.out.println(Thread.currentThread().getName() + "sms");
            call(); // 这里也有锁
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
    public  void call(){
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "call");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```



### 3、自旋锁

spinlock

![image-20210813210822002](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210813210822002.png)



自定义自旋锁

```java
package com.xiang.lock;

import java.util.concurrent.atomic.AtomicReference;
// 自旋锁就是一直判断，直到满足期望的条件才解锁
public class SpinLockDemo {
    // 泛型为引用类型是 默认为null
    AtomicReference<Thread> atomicReference = new AtomicReference<>();

    // 加锁
    public void myLock(){
        Thread thread = Thread.currentThread();
        System.out.println(Thread.currentThread().getName()+"==> myLock");
        while(!atomicReference.compareAndSet(null,thread)){

        }
    }
    // 解锁
    public void myUnLock(){
        Thread thread = Thread.currentThread();
        System.out.println(Thread.currentThread().getName()+"====> myUnLock");
        atomicReference.compareAndSet(thread,null);
    }

    public static void main(String[] args) {
    }
}
```



测试

```java
package com.xiang.lock;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.ReentrantLock;

public class TestSpinLock {
    public static void main(String[] args) throws InterruptedException {
//        ReentrantLock reentrantLock = new ReentrantLock();
//        reentrantLock.lock();
//        reentrantLock.unlock();
        // 自定义的自旋锁 CAS实现
        SpinLockDemo lock = new SpinLockDemo();

        // 必须等A解锁B才能解锁
        new Thread(()->{
            lock.myLock();
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                lock.myUnLock();
            }

        },"A").start();
        TimeUnit.SECONDS.sleep(1);

        new Thread(()->{
            lock.myLock();
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                lock.myUnLock();
            }

        },"B").start();
    }
}
```



结果

![image-20210813215006250](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210813215006250.png)





### 4、死锁



>死锁是什么

![image-20210813215748349](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210813215748349.png)

死锁测试，怎么排除死锁

>解决问题

1、使用`jps-l`定位进程号

![image-20210813221716396](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210813221716396.png)

2、使用`jstack 进程号`找到死锁问题

![image-20210813222238007](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210813222238007.png)