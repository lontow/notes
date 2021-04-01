## 并发

### 线程

又叫做轻量级进程。创建线程比创建进程需要更少的资源。线程存在于进程中，每个进程至少含有一个线程。线程共享进程的资源，包括内存和打开文件等等。这使线程的效率提高，但也造成了线程间交互的问题。在Java应用程序中，会有一个main线程，这个线程能够创建其他线程。

### Thread 对象

每个线程都会和一个线程实例相联系。有两种基本的策略使用Thread:

* 直接实例化 Thread。亲自控制线程的创建和管理
* 将线程管理的任务交给 Executor.只需将task 提交给Executor

本节讲述第一种方式

#### 线程的定义和开始

##### 定义线程的方式有两种

* Runnable接口。该接口定义了一个方法run() 。包含在进程中执行的代码。通过Thread的构造函数，传递给进程。更常用，
* 继承Thread。覆盖Thread自身实现的run()

##### 线程的开始

Thread.start();

##### 使用sleep()暂停执行

Thread.sleep()可使当前的线程挂起，一段时间。这意味在挂起时间段内，处理器可以执行其他线程、甚至其他应用程序的任务。可以指定挂起时间，但是并不保证一定能挂起这么长时间。这取决与操作系统的实现。可能会被中断打断。

当一个线程在挂起过程中，如果被其他线程打断。sleep()会抛出 InterruptedException.

##### 中断

中断告诉线程它应该停止正在做的事，去做一些其他的事。如何回应中断取决于程序员。

线程可通过给想要中断的线程实例上使用interrupt 发送一个中断。

###### 中断支持

有很多方法能够抛出 InterruptedException。对这一类的中断支持，可以catch此Exception并做相应的操作。如果线程中没有此类方法，单纯的耗时任务。则需要周期性的查看Thread.interrupted() ,检查是否收到中断。

```java
//常用方法如下
if(Thread.interrupted()){
    thtow new InterruptedException();
}
```

###### 中断状态标志

`Thread.interrupt`会设置此标志，静态方法`Thread.interrupted()`会清空此设置。非静态方法`isInterrupted`可以查询状态。

##### Joins

join()方法允许一个线程等待另一个线程完成。若 t是需要等待的线程，则`t.join()`将会使当前线程等待t线程结束。同时，还可以指定等待时间，此函数以依赖于操作系统的实现，和sleep()相同，并不能保证一定会等待指定的时间。同时，接受到 InterruptedException时也会退出并抛出异常。



### 同步 Synchronization

线程之间的通信主要通过共享field 和field指向的对象引用。非常有效率但容易造成线程干扰和内存的一致性错误。同步可防止这些问题。但同步可能会造成线程争用。当两个或多个线程试图同时访问相同资源时，JVM的执行会变得非常缓慢甚至停滞。活锁和饥饿时线程争用的两种形式。

#### 线程接口

描述问题如何产生

程序中的一个操作包含多步，而且这些步之间是重叠的。当两个操作操作同一份数据是，变可能发生线程干扰。

#### 内存一致性错误

多个线程对同一份数据有不同的”认知“。大多是由时序上的原因造成的。

###### happens-before关系

既是在操作之前，前面对相同数据的操作完全发生了。

#### 同步方法

java中有两种同步的模式：

* 方法同步
* 语句同步

这两种同步方式的同步粒度不同。本节介绍方法同步。

使用synchronized 关键字

```java
public synchronized void foo(){
}
```

这会产生两个效果，第一，当一个线程调用foo()方法时，其他想要调用相同对象上的foo()方法是不允许的。第二，同步方法推出时，保证方法内的语句都符合 happens-before关系。



***注意：***构造函数不能使用synchronized 。因为在对象被创建的时候，只有创建它的线程访问它。当构造一个对象时，一定要保证构造完成后，才能被其他线程看到。例如，向`List<E>`添加元素是时，一定要保证其添加完成后才能被其他线程看到。

final域由于在初始化之后无法更改，因而可以放在非同步方法中。

#### 隐式锁和同步

##### 内置锁

内置锁又叫监视锁。每个对象都有一个内置锁。习惯上，一个线程想要排外的，一致的访问一个对象的域，那么他需要在访问之前获得该对象的内置锁。并在，访问完毕后，释放内置锁。

当一个线程获得内置锁后，其他的线程不能获得该对象的内置锁。

##### 同步方法中的内置锁

当一个线程调用同步方法时，便会自动获得该对象上的内置锁，并在返回时，释放。

当调用静态同步方法时，将获得与该方法所在类相关联的Class对象的内置锁。

##### 语句同步

控制同步粒度，增加并发性。

```java
synchronized(this){
    ......
        ......
}
```

上述语句，会获得this上的内置锁。

```java
Object lock1=  new Object();
synchronized(lock1){
    ......
        ......
}
```

上述语句，会获得this上的内置锁。

##### 可重入同步

一个线程不能获得被别的线程持有的锁。但可获得被自己持有的锁。这便是可重入同步。例如一个对象中有多个同步方法，或者在同步方法中，调用同一对象上的另一个同步方法。如果没有可重入同步，则线程可能会因为无法获得被自己持有锁而等待甚至造成死锁。

#### 原子访问

原子操作，既是一系列可高效完成的不会被打断的操作，操作结果要么是完全成功，要么和操作之前的状态一致。

java 本身保证的原子操作：

* 对变量引用和大部分元类型（除了long 和double ）的读写是原子的
* 对所有由 volatile(包括long 和double)修饰的变量的读写是原子的

原子操作不能被重叠，所以不用担心线程干扰。但是并不能解决内存一致性问题。volatile可以始终保持最新的内容，减少内存一致性问题。

***注意：***变量保持最新同时也可能造成更改丢失。例如，线程A写变量v->v1,然后线程B也改变了v->v2.此时，若线程A接着读v.则读到了v2 造成一致性错误。

使用原子访问变量比使用同步代码更高效，但同时也更需要注意内存一致性错误。

### 活跃性

并发程序被及时执行的能力称为活跃性。本节讨论两个活跃性问题：

* 死锁
* 活锁和饥饿

#### 死锁

表示多个线程永远相互等待。

#### 饥饿

线程不能正常访问共享资源，因而不能继续进行下去。例如，有耗时任务，一致被执行，则其他想要获得占用资源锁的线程永远被阻塞。

#### 活锁

一个线程经常响应另一个线程的操作。如果另一个线程的操作也是对另一个线程的操作的响应，那么可能会产生livelock。例如：

```
start： 
	P1 lock A 
	P2 lock B 
	P1 lock B fail context switch 
	P2 lock A fail context switch 
	P1 release A 
	P2 release B 
	goto start
```

两人在走廊里相遇，互相挡着，一人向左避，另一人向右避。两人依旧互相挡着。

### Guarded Blocks

此节讨论线程间的合作，其中最常见的合作方式是 Guarded Blocks。既是当某些条件成立时，才能继续进行 block 中的操作。

最常见的方式时，使用Object.wait 挂起当前线程。当另一个线程发出某个特定事件的通知后，此函数才会返回。

```java
public synchronized void guardedJoy() {
    // 对每个特定的事件只循环一次
    while(!joy) {
        try {
            wait();
        } catch (InterruptedException e) {}
    }
    System.out.println("Joy and efficiency have been achieved!");
}
```

***注意：***总是在循环里使用wait().不要假设发生的中断就是你等待的特殊条件，也不要假设条件仍然成立。

使用synchronized 可以使调用者获得相应对象上的内置锁。

当调用wait() 时，线程会释放锁并挂起。一段时间后，另一个线程重新获得该对象上的锁。并调用notifyAll()通知，所有等待该锁的线程。

```java
public synchronized notifyJoy() {
    joy = true;
    notifyAll();
}
```

第二个进程释放锁后的一段时间，第一个线程获得锁并从wait()返回

***注意：***notify()可以通知另一个线程，但不能指定线程。因而常常被用于有很多执行相似任务线程的应用，因为唤醒那个线程无所谓。

#### 生产者——消费者模式

共享对象——一系列的信息：

```java
public class Drop {
    // Message sent from producer
    // to consumer.
    private String message;
    // True if consumer should wait
    // for producer to send message,
    // false if producer should wait for
    // consumer to retrieve message.
    private boolean empty = true;

    public synchronized String take() {
        // Wait until message is
        // available.
        while (empty) {
            try {
                wait();
            } catch (InterruptedException e) {}
        }
        // Toggle status.
        empty = true;
        // Notify producer that
        // status has changed.
        notifyAll();
        return message;
    }

    public synchronized void put(String message) {
        // Wait until message has
        // been retrieved.
        while (!empty) {
            try { 
                wait();
            } catch (InterruptedException e) {}
        }
        // Toggle status.
        empty = false;
        // Store message.
        this.message = message;
        // Notify consumer that status
        // has changed.
        notifyAll();
    }
}
```

生产者线程：

```java
import java.util.Random;

public class Producer implements Runnable {
    private Drop drop;

    public Producer(Drop drop) {
        this.drop = drop;
    }

    public void run() {
        String importantInfo[] = {
            "Mares eat oats",
            "Does eat oats",
            "Little lambs eat ivy",
            "A kid will eat ivy too"
        };
        Random random = new Random();

        for (int i = 0;
             i < importantInfo.length;
             i++) {
            drop.put(importantInfo[i]);
            try {
                Thread.sleep(random.nextInt(5000));//模仿现实情况，随机等待
            } catch (InterruptedException e) {}
        }
        drop.put("DONE");//生产完毕
    }
}
```

消费者线程：

```java
import java.util.Random;

public class Consumer implements Runnable {
    private Drop drop;

    public Consumer(Drop drop) {
        this.drop = drop;
    }

    public void run() {
        Random random = new Random();
        for (String message = drop.take();
             ! message.equals("DONE");
             message = drop.take()) {
            System.out.format("MESSAGE RECEIVED: %s%n", message);
            try {
                Thread.sleep(random.nextInt(5000));
            } catch (InterruptedException e) {}
        }
    }
}
```

主线程：

```java
public class ProducerConsumerExample {
    public static void main(String[] args) {
        Drop drop = new Drop();
        (new Thread(new Producer(drop))).start();
        (new Thread(new Consumer(drop))).start();
    }
}
```

### Immutable Objects(不可变对象)

在构造之后其状态不可再变的对象。使用不可变对象有助于实现简单可靠的编码。可以免疫线程干扰和不一致状态。

通常认为不可变对象的创建的代价非常大，实际并非安全如此。而且，可以在垃圾回收和维护可变对象的消耗中找补回来。

定义不可变对象的简单规则：

1. 不提供“setter”方法。
2. 所有的域声明为 final 和 private。
3. 不允许子类重写方法。可以使用final 修饰类。也可以使构造函数私有化并通过工厂方式构造实例
4. 如果包含指向可变对象的引用。不允许这些对象改变
   1. 不提供修改可变对象的方法
   2. 不共享指向可变对象的引用。不向外部或者可变对象存储引用。如果必要，创建副本，把引用保存到副本。相似的，创建内部可变对象副本并返回副本。

### 高级并发对象

> jdk 1.5 之后

新的并发数据结构：

* Lock objects锁对象
* Executors 提供线程池，管理线程
* Concurrent collections 更容易管理大集合数据，能够极大减少同步需求
* atomic variables 原子变量，最小的同步，帮助避免内存不一致。
* ThreadLocalRandom（JDK7）多线程伪随机数

#### Lock Objects

Lock 很像同步代码块中使用的隐式锁。同一时间，只有一个线程持有同一对象上的锁。通过相关联的Condition 对象支持wait/notify机制。

Lock objects的最大优势是，tryLock。既是在锁不能获得时，立即或者超时返回。在获得锁之前，lockInterruptibly 方法在收到另一个线程的中断时，能够放弃。

#### Executors

##### Executor interfaces

* Executor 支持启动任务的接口
* ExecutorService  Executor的子接口。管理任务和其自身的生命周期
* ScheduledExecutorService ExecutorService的子接口，支持未来或者周期性执行任务

###### Executor

只用一个简单的 execute()执行函数定义。接受Runnable

###### ExecutorService

支持 execute() 和 submit()。submit() 接受Runnable ,同时接受Callable.Callabel对象支持任务返回值。submit 返回Future对象，Future对象可以提取Callable对象的返回值，并能够管理Callable和Runnable任务。

ExecutorService也支持提交大量的Callable 集合。同时提供管理Executor的方法。为了支持立即关闭，需要任务正确处理interrupt。

###### ScheduledExecutorService

增加 schedule，可以延时执行执行Runnable或者Callable任务。

增加 scheduleAtFixedRate，scheduleWithFixedDelay周期性执行任务

##### Thread Pools

大量Executor的实现使用线程池，线程池中包含工作线程。工作线程独立于Runnable和Callable存在。常常用于执行多任务。使用线程池可以大大减少线程创建的工作量。

一种常用的线程池使固定数目线程池。顾名思义，维护一个固定数量线程的线程池。任务被提交到内部队列，队列里保存多于线程数量的额外任务。

固定数目线程池最大的优点是，限制了系统资源的消耗。

创建固定数量线程池的方式，可以使用 Executors.newFixedThreadPool 工厂模式。同时也提供了：

* newCachedThreadPool 创建表一个可扩展的线程池。
* newSingleThreadExecutor 创建一个单线程Executor
* 创建ScheduledExecutorService版本的线程池

同时可以使用ThreadPoolExecutor 或者ScheduledThreadPoolExecutor 自行创建。

##### Fork/Join

实现ExecutorService以提供对多处理器的支持。使用work-stealing算法。运行完的线程可偷取其他繁忙线程的任务。

整个Fork/Join框架的中心是 ForkJoinPool类，是AbstractExecutorService 类的扩展。实现了work-stealing算法，并可以执行ForkJoinTask。

###### 基本用法

将大任务拆成小任务，根据任务的大小，判断是否可以直接运行。伪代码如下:

```
if (my portion of the work is small enough)
  do the work directly
else
  split my work into two pieces
  invoke the two pieces and wait for the results
```

将上述代码封装到ForkJoinTask子类中，如 RecursiveTask(可以返回结果)或者 RecursiveAction。

然后，创建一个代表所有任务的对象，并使用ForkJoinPool.invoke()调用。

###### 图片模糊实例

源图片使用一个Integer数组表示，每个Integer代表一个像素的颜色。目标模糊图像也是这种格式。

实现图片模糊的方法是，对源图片的每一个像素的颜色使用它周围的像素进行平均。由于图片数组可能很大。所以考虑使用fork/join框架。

```java
public class ForkBlur extends RecursiveAction {
    private int[] mSource;
    private int mStart;//每一部分的起始
    private int mLength;//每一部分的长度
    private int[] mDestination;
  
    // Processing window size; should be odd.
    private int mBlurWidth = 15;
  
    public ForkBlur(int[] src, int start, int length, int[] dst) {
        mSource = src;
        mStart = start;
        mLength = length;
        mDestination = dst;
    }

    protected void computeDirectly() {
        int sidePixels = (mBlurWidth - 1) / 2;
        for (int index = mStart; index < mStart + mLength; index++) {
            // Calculate average.
            float rt = 0, gt = 0, bt = 0;
            for (int mi = -sidePixels; mi <= sidePixels; mi++) {
                int mindex = Math.min(Math.max(mi + index, 0),
                                    mSource.length - 1);
                int pixel = mSource[mindex];
                rt += (float)((pixel & 0x00ff0000) >> 16)
                      / mBlurWidth;
                gt += (float)((pixel & 0x0000ff00) >>  8)
                      / mBlurWidth;
                bt += (float)((pixel & 0x000000ff) >>  0)
                      / mBlurWidth;
            }
          
            // Reassemble destination pixel.
            int dpixel = (0xff000000     ) |
                   (((int)rt) << 16) |
                   (((int)gt) <<  8) |
                   (((int)bt) <<  0);
            mDestination[index] = dpixel;
        }
    }
  
  ...
```

实现抽象的compute()方法，该方法要么直接模糊要么分成更小的任务。

```java
protected static int sThreshold = 100000;//划分成更小任务的界限

protected void compute() {
    if (mLength < sThreshold) {
        computeDirectly();
        return;
    }
    
    int split = mLength / 2;
    //分成两个更小的任务
    invokeAll(new ForkBlur(mSource, mStart, split, mDestination),
              new ForkBlur(mSource, mStart + split, mLength - split,
                           mDestination));
}
```

设置 ForkJoinPool:

```java
// source image pixels are in src
// destination image pixels are in dst
ForkBlur fb = new ForkBlur(src, 0, src.length, dst);
ForkJoinPool pool = new ForkJoinPool();
pool.invoke(fb);
```

###### 使用Fork/Join的标准库

* java.util.Arrays.parallelSort()
* java.util.streams

#### ConcurrentCollections

1. BlockingQueue 定义一个队列。当向满队列添加或者向空队列取数据。将会block或者timeout
2. ConcurrentMap 接口额外定义了一些原子操作。
3. ConcurrentNavigableMap接口。支持模糊匹配。

#### 原子变量

所有的原子变量方法都是原子的。

#### 并发随机数

```java
int r = ThreadLocalRandom.current() .nextInt(4, 77);
```

### See Also

java-tutorial https://docs.oracle.com/javase/tutorial/essential/concurrency/threadlocalrandom.html