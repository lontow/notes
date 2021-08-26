# JUC

## 线程的状态

- NEW 新建
- RUNNABLE　准备就绪
- BLOCKED　阻塞（等待monitor Lock）
- WAITING　等待（不见不散）等待其他线程的某个特殊操作
- TIMED_WAITING　过时不侯
- TERMINATED　终结

## wait/sleep

1. sleep 是Thread 的静态方法，wait 是Object 的方法
2. sleep 不会释放锁，也不会占用锁。wait 会释放锁，调用的前提是当前进程占有锁
3. 都可以被 interrupted 方法中断



## 管程

monitor /锁

一种同步机制，在同一时间内，只有一个线程访问相应资源

jvm 同步基于进入/推出管程对象实现

## 用户线程/守护线程

1. 自定义线程
2. 守护线程（垃圾回收线程）

如果没有用户线程了，jvm 会终结

## Lock 接口

### synchronized

同步锁。

```java
synchronized(obj){
    
}
synchronized void func(){
    
}
```

*多线程编程步骤*

1. 创建资源类，实现属性和操作方法
   1. 判断
   2. 干活
   3. 通知
2. 创建多个线程，调用资源类的操作方法
3. 防止虚假唤醒

### Lock

#### ReentrantLock

可重入锁。

1. 需要手动上锁和释放锁。发生异常时，需要在finally unlock 
2. lock 可以响应中断

## 线程通信

### 虚假唤醒问题

if 只会判断一次，wait() 在哪儿停止，在哪儿继续运行

```java
if(number !=0){
    wait();
}

while(number!=0){
    wait();
}
```

### 定制化通信

`Condition` 类　

## 集合的线程安全

### List并发修改解决

#### Vector

#### Collections.synchronizedList

#### CopyOnWriteArrayList

​	**写时复制技术**　复制一封ArrayList用于写。然后将副本和原来的合并。

### HashSet并发修改解决

#### CopyOnWriteArraySet

### HashMap并发修改解决

#### ConcurrentHashMap

## 多线程锁

#### synchronized

1. synchronized 静态方法，锁类对象Class
2. synchronized 普通方法，锁类对象Object
3. 同步方法块，锁Synchronized 中的对象

#### 公平锁和非公平锁

导致一个线程做完所有的工作

#### 可重入锁（递归锁）

1. synchronized(隐式)
2. lock

## Callable 接口

可以得到线程的返回结果

与Runnable 的不同

1. 有返回值。
2. 可以抛出异常
3. 使用call方法

**不能直接传入Thread(),可借助FutureTask（实现了Runnable）**

### FutureTask

## 辅助类

### CountDownLatch

可以设置一个计数器(`CountDownLatch(init val)`)，`countDown()`减一。

`await()` 等待直到　计数器为0

### 循环栅栏CyclicBarrier

同步辅助类，允许一组线程互相等待。直到达到某个公共屏障点。由于在释放等待线程后可以重用，因此是循环的

### 信号灯Semaphore

计数信号量，维护了一个许可集。如有必要，在许可可用前会阻塞没一个`acquire()`,然后再获取该许可。每个`release()`添加一个许可，从而可能释放一个正在阻塞的获取者

## ReentrantReadWriteLock

读写锁，一个资源可以被多个读线程访问，但只能被一个写线程访问。但不能同时存在读写线程

### 优势

1. `synchronized`和`ReentrantLock` 读读不能共享

### 缺点

1. 造成锁饥饿
2. 读时不能写。
3. 获取写锁后，可以再获取读锁

### 锁降级

将写锁降级为读锁。不必释放写锁后，在重新获取读锁。从而保证一直持有锁。但不必一直持有重量级的写锁。

## 阻塞队列

多线程共享的队列。

### 分类

1. ArrayBlockingQueue 定长数组
2. LInkedListBlockingQueue
3. DelayQueue 优先队列实现的延迟无界阻塞队列
4. PriorityBlockingQueue 优先级排序
5. SynchronousQueue 单元素队列
6. LinkedTransferQueue 

## 线程池

### 底层原理

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler)
```



- corePoolSize  常驻线程数量
- maximumPoolSize　最大线程数量
- keepAliveTime　线程存活时间
-  unit　存活时间单位
- workQueue　工作任务的阻塞队列
- threadFactory　线程工厂
- handler　拒绝策略

### 工作流程

![image-20210810172039523](/home/lzl123/.config/Typora/typora-user-images/image-20210810172039523.png)

1. 调用　execute() 才创建线程
2. 阻塞队列满，且达到maximumPoolSize 执行拒绝策略
3. 阻塞队列满，未达到maximumPoolSize。创建新线程，新任务给新线程

***注意：***

自定义线程池，防止任务队列OOM

## FORK/JOIN

将大的任务拆分成多个子任务进行并行处理，最后将子任务结果合并成最后的计算结果。

```java
//ForkJoinPool
//ForkJoinPool
//ReCursiveTask<>

/*
    compute 1+2+3..+100
 */
class MyTask extends RecursiveTask<Integer>{
    //差值<= VALUE 计算，否则拆分
    private static final Integer VALUE=10;
    private int begin;
    private  int end;
    private int result;

    public MyTask(int begin, int end) {
        this.begin = begin;
        this.end = end;

    }

    @Override
    protected Integer compute() {
        if(end-begin>VALUE){
            int mid  =begin+ (end-begin)/2;
            MyTask task1 = new MyTask(begin,mid);
            MyTask task2 =new MyTask(mid+1,end);
            task1.fork();
            task2.fork();
            result = task1.join()+task2.join();
        }else {
            for (int i = begin; i <= end; i++) {
                result += i;
            }
        }
        return result;
    }
}
public class ForkJoinDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        MyTask task = new MyTask(1,100);
        //ForkJoinPool
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        ForkJoinTask<Integer> submit = forkJoinPool.submit(task);
        Integer result = task.get();
        System.out.println(result);
        forkJoinPool.shutdown();
    }
}
```



### CompletableFuture 异步回调
