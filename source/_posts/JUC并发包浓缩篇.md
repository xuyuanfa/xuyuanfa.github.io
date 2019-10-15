---
title: JUC并发包浓缩篇
categories:
- Java
tags:
- JUC
---

- [线程池](##1)
- [同步辅助类Semaphore、CountDownLatch、CyclicBarrier、Phaser、Exchanger](##2)
- [原子变量和原子类](##3)
- [ConcurrentHashMap](##4)
- [CopyOnWrite](##5)
- [ForkJoin](##6)
- [锁](##7)
<!--more-->

<span id="#1"></span>
### 线程池
**线程池核心构造方法**
```
public ThreadPoolExecutor(
int corePoolSize,
int maximumPoolSize,
long keepAliveTime,
TimeUnit unit,
BlockingQueue<Runnable> workQueue,
ThreadFactory threadFactory,
RejectedExecutionHandler handler){...}
```

- corePoolSize：当前线程池中的线程数目小于corePoolSize，则每来一个任务，就会创建线程执行这个任务；
- workQueue：当前线程池中的线程数目>=corePoolSize，则每来一个任务，会尝试将其添加到任务缓存队列当中；
- maximumPoolSize：当workQueue有界队列满了，则每来一个任务，创建临时线程执行这个任务，当前线程池中的线程数目达到maximumPoolSize，则会采取任务拒绝策略进行处理；
- rejectedExecutionHandler：拒绝策略；
- keepAliveTime：临时线程休闲时的存活时间；
- TimeUnit：存活时间单位；
- threadFactory：线程工厂，可自定义有意义名称的线程，也可以使用默认线程工厂；


**预设线程池**

- newSingleThreadExecutor：
创建一个单线程的线程池。这个线程池只有一个线程在工作，也就是相当于单线程串行执行所有任务。如果这个唯一的线程因为异常结束，那么会有一个新的线程来替代它。此线程池保证所有任务的执行顺序按照任务的提交顺序执行。
new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>())
- newFixedThreadPool
创建固定大小的线程池。每次提交一个任务就创建一个线程，直到线程达到线程池的最大大小。线程池的大小一旦达到最大值就会保持不变，如果某个线程因为执行异常而结束，那么线程池会补充一个新线程。
new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>())
- newCachedThreadPool
创建一个可缓存的线程池。如果线程池的大小超过了处理任务所需要的线程，那么就会回收部分空闲（60秒不执行任务）的线程，当任务数增加时，此线程池又可以智能的添加新线程来处理任务。此线程池不会对线程池大小做限制，线程池大小完全依赖于操作系统（或者说JVM）能够创建的最大线程大小。
new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS, new SynchronousQueue<Runnable>())
- newScheduledThreadPool
创建一个大小无限的线程池。此线程池支持定时以及周期性执行任务的需求。
new ScheduledThreadPoolExecutor(corePoolSize, Integer.MAX_VALUE, 0, TimeUnit.NANOSECONDS, new DelayedWorkQueue())


**预定义拒绝策略**

- ThreadPoolExecutor.AbortPolicy
当添加任务出错时时，直接抛出异常
- ThreadPoolExecutor.CallerRunsPolicy
当添加任务出错时时，主线程执行加入的任务
- ThreadPoolExecutor.DiscardOldestPolicy
当添加任务出错时时，移除第一个任务，执行加入的任务
- ThreadPoolExecutor.DiscardPolicy
当添加任务出错时时，忽略不做处理


**线程存活机制**

临时线程会在指定keepAliveTime空闲后销毁；
核心线程在设置allowCoreThreadTimeOut后允许keepAliveTime空闲后销毁；
实现原理：
1. 创建工作线程，初始化第一个task任务开始执行；
2. 循环从队列拉取任务，继续执行；
3. 如是临时线程或核心线程允许超时，则调用任务队列的poll(keepAliveTime, timeUnit)方法，指定时间后取不到任务则返回空，线程跳出循环执行至完毕自动结束；
4. 如是核心线程未设置允许超时，则调用任务队列的take()方法，无任务则一直阻塞等待。


**Callable和Future**

- 单线程用法
Callable对象封装成FutureTask对象（FutureTask继承了Runnable），放入new Thread(callable)执行返回future对象，future.get()获取结果；
- 线程池用法
threadPool.submit(callable)，submit内部封装callable对象为FutureTask对象，执行返回future对象，future.get()获取结果；
threadPool.submit(runnable)，submit内部封装runnable对象为FutureTask对象，执行返回future对象，future.get()只返回null；
- 线程池多任务结果无顺序用法
把threadPool封装为ExecutorCompletionService对象（接口CompletionService），cs.submit(callable)执行结果无顺利存放在cs中的队列，cs.take().get()从队列弹出结果；

区分
- Executor，是接口，只有execute方法；
- ExecutorService，是接口，扩展了Executor；
- ThreadPoolExecutor，继承AbstractExecutorService实现了ExecutorService接口；
- Executors，封装了ThreadPoolExecutor的实现类；


<span id="#2"></span>
### 同步辅助类CountDownLatch、CyclicBarrier、Semaphore、Phaser、Exchanger

**CountDownLatch闭锁**
await()等待一组线程达到条件countDown()后，计数为0时继续执行，不可复用。

**CyclicBarrier循环栅栏**
await()一组线程相互等待，达到计数后同时执行，可以复用。

**Semaphore信号量**
acquire()申请占用一个信号，release()释放一个信号，超过信号量的申请则阻塞等待释放。

**Phaser阶段**
register()注册参与者线程，arriveAndAwaitAdvance()等待其他参与者到达；
new Phaser(1)初始化注册者，arriveAndDeregister()注销注册者；
重写onAdvance(int phase, int registeredParties) 返回true时isTerminated()为true，phase是当前阶段数（从0开始），registeredParties是当前阶段已注册数；当arriveAndAwaitAdvance()线程都达到后为一个阶段，重复进入下一阶段；
new Phaser(phaser)传入父节点实现分层，子phaser第一次register时也向父节点doRegister(1)。

**Exchanger交换器**
exchange(o)交换两个线程的数据。


<span id="#3"></span>
### 原子变量和原子类
**volatile两个语义**
a.保证此变量对所有线程的可见性；
b.禁止指令重排序优化；
- 保证原子性的规则
a.运算结果并不依赖变量的当前值，或者能够确保只有单一的线程修改变量的值；
b.变量不需要与其他的状态变量共同参与不变约束；
- 可见性指令实现
lock addl $0x0,(%esp)
addl $0x0,(%esp)把ESP寄存器的值加0；
lock前缀使本CPU的cache写入内存，该写入动作会引起别的CPU或内核无效化其cache；
通过这样的空操作，让volatile变量的修改对其他CPU立即可见。
- 指令重排序
是指CPU采用了允许将多条指令不按程序规定的顺序分开发送给各相应的电路单元处理；
lock addl $0x0,(%esp)指令把修改同步到内存时，所有之前的操作都已经执行完成，实现内存屏障的效果。

**Atomic**
类可以分成4组
1. AtomicBoolean，AtomicInteger，AtomicLong，AtomicReference
2. AtomicIntegerArray，AtomicLongArray
3. AtomicLongFieldUpdater，AtomicIntegerFieldUpdater，AtomicReferenceFieldUpdater
4. AtomicMarkableReference，AtomicStampedReference，AtomicReferenceArray

AtomicInteger 类主要利用 CAS (compare and swap) + volatile 和 native 方法来保证原子操作。


<span id="#4"></span>
### ConcurrentHashMap
**JDK7中ConcurrentHashMap结构**
- 分段锁Segment
内部类似于HashMap的结构，拥有一个Entry数组，数组中的每个元素又是一个链表；
同时又是一个ReentrantLock（Segment继承了ReentrantLock）。

- 对比HashMap
1. ConcurrentHashMap中的HashEntry相对于HashMap中的Entry有一定的差异性：HashEntry中的value以及next都被volatile修饰，这样在多线程读写过程中能够保持它们的可见性。
2. ConcurrentHashMap键值对都不能为空。
3. HashMap键值对可为空，当key为null时，存放在数组的第0个链表。

- 结构
ConcurrentHashMap-->Segment数组-->HashEntry数组-->链表-->键值对。
HashMap-->Entry数组-->链表-->键值对。

**ConcurrentHashMap双倍扩容**
元素存放数组位置：key的hash值与 HashEntry数组长度减一的与运算；
- 双倍扩容
数组长度是2的倍数，元素的key的hash值再次和新长度减一做与运算，位置取决于2倍数减一的二进制的第一个1，即决定存放在原来的数组位置，还是加一倍的位置。


**JDK8中ConcurrentHashMap结构**
抛弃了原有的 Segment 分段锁，而采用了 CAS + synchronized 来保证并发安全性。
- 结构
ConcurrentHashMap-->Node数组-->链表（或红黑树）-->键值对。
- 原理
1. 数组索引位置的链表为空时，尝试CAS写入第一个node。
2. 数组索引位置有node时，synchronized锁住第一个node。
3. 第一个node的hash值大于0时表示链表，遍历链表，key已存在则更新，不存在则写入尾巴。
4. 数量大于等于8 时转换为红黑树。

**JDK8中，HashMap特性：**
1. 添加红黑树，当链表长度大于等于8时，转换链表为红黑树；
2. 扩容优化，计算索引位置改为计算是否落在原索引，否则原索引+扩容大小，与运算2的幂次方（只有一个1），采用链表预存储元素，头部节点指向第一个元素，尾部节点向后移动，替代每次增加元素到数组的链表，思路更清晰；
3. 求数组位置，与运算取代取模运算，由于长度固定为2的幂次方，长度减1与运算；


<span id="#5"></span>
### CopyOnWrite
可以对CopyOnWrite容器进行并发的读，而不需要加锁，用于读多写少的并发场景。
Copy的是引用，而不是在堆内存复制对象。


<span id="#6"></span>
### ForkJoin
是一个执行并行任务的框架，它是把一个大任务分割成若干个小任务，再将若干个小任务的计算结果汇总起来，得到大任务的计算结果。
采用的是工作窃取算法，某个线程从其他队列中窃取任务进行执行的过程。
执行：创建ForkJoinPool，创建任务RecursiveTask，提交任务submit(task)。

    @Override
    protected Integer compute() {
        int sum = 0;
        //如果任务足够小就计算任务
        boolean canCompute = (end - start) <= threshold;
        if (canCompute) {
            for (int i = start; i <= end; i++) {
                sum += i;
            }
        } else {
            // 如果任务大于阈值，就分裂成两个子任务计算
            int middle = (start + end) / 2;
            ForkJoinTaskExample leftTask = new ForkJoinTaskExample(start, middle);
            ForkJoinTaskExample rightTask = new ForkJoinTaskExample(middle + 1, end);
            // 执行子任务
            leftTask.fork();
            rightTask.fork();
            // 等待任务执行结束合并其结果
            int leftResult = leftTask.join();
            int rightResult = rightTask.join();
            // 合并子任务
            sum = leftResult + rightResult;
        }
        return sum;
    }


<span id="#7"></span>
### 锁
[同步锁2019总结篇](#10 )






by xuyuanfa