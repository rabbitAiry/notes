# Java进阶

## b. 并发

### #15 并发基础知识
##### 15.1 线程基本概念
##### 15.2 理解synchronized    
##### 15.3 线程的基本协作机制
##### 15.4 线程的中断

### #16 并发包的基石

>   Java并发工具包

##### 16.1 原子变量和CAS

>   使用原子变量改善使用synchronized的简单任务
>
>   AtomicBoolean \AtomicInteger \AtomicLong \AtomicReference \ ...

-   原子变量，主要介绍AtomicInteger

    -   提供一些原子方式实现组合操作的方法，如：`getAndSet(int newValue)`、`getAndAdd(int delta)`、`addAndGet(int delta)`

    -   组合操作的方法实现依赖于`compareAndSet(int expect, int update)`，简称CAS

        -   expect为当前值，update为当前值更新后的结果
        -   调用CAS方法进行更新，如果更新失败，说明数据被别的线程更改了，则再去取最新值并尝试更新直到成功为止

        ```java
        // 组合操作方法的实现
        private volatile int value;
        
        //...
        
        public final int incrementAndGet(){
        	for(;;){
                int current = get();
                int next = current+1;
                if(compareAndSet(current, next)){
                    return next;
                }
            }
        }
        ```

    -   乐观锁

        -   synchronized是悲观锁，代表一种阻塞式算法，有上下文切换开销
        -   原子变量是乐观锁，使用CAS进行冲突检测，更新逻辑是非阻塞式的

    -   CAS的实现：硬件层次上直接支持CAS指令，可以看作是计算机的基本操作（unsafe类）

-   ABA问题：另一个线程将当前值A修改为B后，又重新修改为A，但是线程的CAS操作无法分辨当前值发生过变化

    -   通过引用时间戳解决


##### 16.2 显式锁

-   显式锁接口Lock

    ```java
    public interface lock(){
    	void lock();		// 获取锁，该方法会阻塞直到成功
    	void lockInterruptibly() throws InterruptedException;
    	boolean tryLock();	// 尝试获取锁，不阻塞
    	boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
    	void unlock();		// 释放锁
    	Condition newCondition();
    }
    ```

-   可重入锁ReentrantLock
    -   lock/unlock实现了与synchronized一样的语义

        -   可重入、可以解决竞态条件问题（保证公平）、可以保证内存可见性

        -   在ReentrantLock的构造时，通过将fair参数指定为true以保证公平，使得等待锁的队列先到先得

            ```java
            public ReentrantLock()				// 默认不公平
            public ReentrantLock(boolean fair)
            ```

        -   一定要记得调用unlock

    -   使用tryLock避免死锁

        -   案例：账号A、B都有锁，账号A转账给账号B的同时，账号B也在转账给账号A
        -   使用`tryLock()`尝试获取锁，不能获取则放弃本次操作，释放自身所有锁

        ```java
        boolean tryTransfer(Account from,Account to,int val){
        	if(from.tryLock()){
                try{
        			if(to.tryLock()){
                        try{
                        	if(from.getMoney()>=money){
                        		from.reduce(money);
                        		to.add(money);
                    		}else{
                                throw new NoMoneyException()
                            }
                            return true;
                        }finally{
                            to.unlock();
                        }
                    }
                }finally{
                    from.unlock();
                }
            }
            return false;
        }
        ```

        ```java
        void transfer(Account from,Account to,int val) throws NoMoneyException{
            boolean success = false;
            do{
                sucess = tryTransfer(from, to, val);
                if(!success){
                    Thread.yield();
                }
            }while(!success);
        }
        ```

-   显式锁ReentrantLock实现原理
    -   LockSupport

        ```java
        // 基本方法
        park()				// 使得当前线程放弃CPU，进入等待状态
        parkNanos(long Nanos)
        parkUntil(long deadline)
        unpark(Thread thread)	// 恢
        ```

        ```java
        // temp线程示例
        Temp temp = new Temp(){
            @Override
            public void run() {
                LockSupport.park();
                System.out.println("exit");
            }
        };
        temp.start();
        Thread.sleep(1000);
        LockSupport.unpark(temp);
        ```

        -   `park()`与`yield()`不同，后者只是告诉操作系统可以先让其他线程运行，而自己依然是可运行状态，而前者会放弃调度资格，使线程进入WAITING状态
        -   `park()`是响应中断的，当有中断发生时，返回
        -   parkNanos：等待的最长时间
        -   parkUntil：指定最长等待到什么时候
        -   与CAS一样，LockSupport调用了操作系统的API

    -   AbstractQueued-Synchronizer抽象类，简称AQS

        -   简化了并发工具的实现
        -   AQS封装了一个状态，给子类提供了查询和设置状态的方法

-   ReetrantLock的实现，内部使用AQS

    -   3个内部类，1个Sync成员
        -   3个内部类：抽象类Sync、NonfairSync是构造参数fair为false时使用的类、FairSync是fair为true时使用的类
        -   1个Sync成员：用于根据参数fair指向对应的Sync成员
    -   lock的实现
        -   ReentrantLock使用State表示是否被锁和持有数量，如果当前未被锁定，则立即获得锁
        -   否则调用AQS的`acquire(1)`获得锁
        -   如果还是不能获得锁，则调用`acquireQueued(addWaiter(Node.EXCLUSIVE), arg)`。其主体是是循环，如果能获得锁，则返回；否则调用`LockSupport.park`放弃CPU，等待被唤醒，检查自己是否是第一个等待的线程，若否则继续等
    -   保证公平让活跃线程得不到锁，加入等待状态，引起频繁的上下文切换

-   与synchronized对比

    -   synchronized代表一种声明式编程思维，而显式锁代表一种命令式编程思维
    -   synchronized更简单，且能被编译器和JVM优化，满足需求时应该首选


##### 16.3 显式条件

-   用法
-   生产者/消费者模式
-   原理

[TOC]

### #17 并发容器

##### 17.1 写时复制的List和Set

-   CopyOnWriteArrayList/Set写时复制的List和Set
    -   线程安全的，可被多个线程并发访问
    -   其迭代器不支持修改操作，但也不会抛出ConcurrentModificationException
    -   以原子方式支持一些复合操作
    -   迭代时不需要加锁
    
-   CopyOnWriteArrayList

    -   两个原子方法

        ```java
        // 不存在才添加，如果添加了，返回true
        public boolean addIfAbsent(E e)
        // 批量添加c中的非重复元素，不存在才添加，返回实际添加的个数
        public int addAllAbsent(Collection<? extends E> c)
        ```

    -   CopyOnWriteArrayList内部也是一个数组，但这个数组是以原子方式被整体更新的

        -   每次修改操作，都会新建一个数组，复制原数组的内容到新数组，在新数组上进行修改，然后以原子方式设置内部的数组引用
        -   数组内容是只读的
        -   性能低，不适用与数组很大且修改频繁的场景。适用于写操作发生次数少的场景

-   CopyOnWriteArraySet：内部通过CopyOnWriteArrayList实现

##### 17.2 ConcurrentHashMap

-   并发版本的HashMap
    -   并发安全
    -   直接支持一些原子复合操作
    -   支持高并发，读操作完全并行，写操作支持一定程度的并行
    -   与同步容器Collections.synchronizedMap相比，迭代不用加锁，不会抛出ConcurrentModificationException
    -   弱一致性
-   并发安全
-   原子复合操作
-   高并发的基本机制
-   迭代安全
-   弱一致性：在遍历过程中，map内部元素可能发生变化。如果在未遍历过的部分，迭代器不会反映出来

##### 17.3 基于跳表的Map和Set

-   Java并发包中与TreeMap/TreeSet对应的并发版本是ConcurrentSkipListMap和ConcurrentSkipListSet
-   基于跳表的Map和Set
    -   没有使用锁，所有操作都是无阻塞的，所有操作都可以并行，包括写，多线程可以同时写
    -   弱一致性
    -   支持一些原子复合操作

##### 17.4 并发队列

-   Java并发包提供的几种队列
    -   无锁非阻塞并发队列
    -   普通阻塞队列
    -   优先级阻塞队列
    -   延时阻塞队列
    -   其他

[TOC]

### #18 异步任务执行服务

##### 18.1 基本概念和原理

-   Runnable和Callable：表示要执行的异步任务

    -   Runnable没有返回结果，而Callable有
    -   Ruunable不会抛出异常，而Callable会

-   Executor和ExecutorService：表示执行服务

    -   Executors是其工厂类
    -   Executor表示最简单的执行任务

    ```java
    public interface Executor{
    	void execute(Runnable command);
    }
    ```

    -   ExecutorService扩展了Executor，定义了更多服务
        -   返回Future后，表示任务已提交，不代表已执行
        -   通过Future可以查询异步任务状态、获取最终结果、取消任务等等

    ```java
    public interface ExecutorService extends Executor{
        <T> Future<T> submit(Callable<T> task);
        <T> Future<T> submit(Runnable task, T result);
        Future<?> submit(Runnable task);
        // ...其他方法
    }
    ```

-   Future：表示异步任务的结果

    -   get方法用于返回异步任务最终的结果。如果任务还未执行完成，会阻塞等待
    -   get方法2可以限定阻塞等待的时间，若超时任务还未结束，则抛出TimeoutException
    -   cancel方法用于取消异步任务，如果任务已完成/取消/不能被取消，则返回false

    ```java
    public interface Future<V>{
    	boolean cancel(boolean mayInterruptIfRunning);
        boolean isCancelled();
        boolean isDone();
        V get() throws InterruptedException, ExecutionException;
        V get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException;
    }
    ```

-   基本实现原理

    -   AbstractExecutorService：ExecutorService的抽象实现类
    -   FutureTask

##### 18.2 线程池

-   线程池里面有若干线程，它们的目的就是执行提交给线程池的任务，执行完一个任务后不退出，而是继续等待或执行新任务
    -   优点
        -   重用线程，避免线程创建的开销
        -   任务过多时，通过排队避免创建过多线程，减少系统资源消耗和竞争，确保任务有序完成

    -   实现类：ThreadPoolExecutor
    -   任务队列：保存待执行任务
    -   工作者线程：主体是一个循环，从任务队列中接受任务并执行
-   线程池大小
    -   参数
        -   corePoolSize：核心线程个数
        -   maximumPoolSize：最大线程个数
        -   keepAliveTime和unit：空闲线程存活时间
    -   线程的数量会动态变化，但不会超过maximumPoolSize
    -   创建一个线程池后，不会立即创建任何线程
    -   有新任务到来时，如果当前线程个数小于corePoolSize，则会创建一个新线程来执行任务。即使其他线程是空闲的
    -   如果线程数量大于corePoolSize，则会尝试排队（尝试，而不是阻塞等待）
        -   如果队列满了，则先检查线程个数是否达到了maximumPoolSize，并创建线程
    -   keepAliveTime：超时后线程被终止
-   任务队列：类型为BlockingQueue
    -   如果使用无界队列，则线程个数最多只能达到corePoolSize，参数maximumPoolSize失去意义
    -   如果使用SynchronousQueue时，因为其没有实际存储元素的空间，所以当尝试排队时有空闲线程才能入队成功
-   任务拒绝策略
    -   队列已满、线程个数达到最大值时提交任务，会触发任务拒绝策略
    -   默认情况下，此时提交任务会抛出异常RejectedExecutionException
    -   ThreadPoolExecutor实现了4种处理方式
        -   ThreadPoolExecutor.AbortPolicy：默认方式
        -   ThreadPoolExecutor.DiscardPolicy：静默处理，忽略新任务，不抛出异常，也不执行
        -   ThreadPoolExecutor.DiscardOldestPolicy：将等待时间最长的任务扔掉，然后自己排队
        -   ThreadPoolExecutor.CallerRunsPolicy：在任务提交者线程中执行任务，而不是交给线程池中的线程执行

    -   拒绝策略只有在队列有界，且maximumPoolSize有限的情况下才会触发
-   线程工厂ThreadFactory
-   关于核心线程的特殊配置
    -   线程个数小于等于corePoolSize时，称这些线程为核心线程
    -   核心线程不会因为空闲而终止，keepAliveTime参数默认不适用于核心线程
-   工厂类Executors可以方便地创建一些预配置的线程池
    -   FixedThreadPool
        - 线程数量固定的线程池，只有核心线程
        - 任务队列也无大小限制
        - 因为线程不会被回收，所以能够更快地响应外界请求
    -   CachedThreadPool
        - 线程数量不定的线程池。只有非核心线程，且最大线程数为Integer.MAX_VALUE（相对于无限大）
        - 所有的空闲线程都会有超时机制，默认为60秒
        - 其队列相对于一个空集合，这会导致所有的任务都会被立即执行（SynchronousQueue）
        - 适用于执行大量的耗时较少的任务
    -   ScheduledThreadPool
        - 核心线程数量是固定的，而非核心线程数是没有限制的
        - 当非核心线程闲置时会被立即回收
        - 适用于执行定时任务和具有固定周期的重复任务
    -   SingleThreadExecutor
        - 只有一个核心线程，确保所有的任务都在该同一个线程中按顺序执行，使得在这些任务之间不需要处理线程同步的问题

-   线程池中的死锁

##### 18.3 定时任务及其陷阱

-   Java中两种实现定时任务的方式
    -   Timer和TimerTask
    -   Java并发包的ScheduledExecutorService
-   Timer和TimerTask
    -   使用
    -   原理
        -   Timer内部主要由任务队列和Timer线程两部分组成。一个Timer对象只有一个Timer线程
        -   Timer线程主体是一个循环，不断从队伍中获取或等待任务
        -   只有完成一个任务后才执行下一个
    -   异常处理
        -   在执行任何任务中遇到抛出异常，Timer线程就会退出，从而所有定时任务都会被取消
        -   Timer会因此抛出异常
-   ScheduledExecutorService
    -   原理
        -   基于线程池实现，有多个线程可以执行定时任务
        -   任务队列是一个无界的优先级队列
    -   异常处理
        -   TaskB抛出异常并不影响TaskA，但ScheduledExecutorSerivce不会因此抛出异常





[TOC]

### #19 同步和协作工具类

>   19.1-19.4皆是基于AQS实现

##### 19.1 读写锁ReentrantReadWriteLock

-   在读操作中可以并行，只要操作中包括写就不可以并行
-   分为读锁和写锁

##### 19.2 信号量Semaphore

>   使用场景：对不同等级的账户，限制不同的最大并发访问数

-   可以限制对资源的并发访问数

##### 19.3 倒计时门栓CountDownLatch

-   倒计时变为0后，门栓打开，等待的所有线程都可以通过
-   打开后就不能关上了，所以CountDownLatch是一次性的

##### 19.4 循环栅栏CyclicBarrier

-   适用于并行迭代计算，每个线程负责一部分计算，然后再栅栏处等待其他线程完成。所有线程到齐后，交换数据和计算结果，再进行下一次迭代

##### 19.5 理解ThreadLocal

-   线程本地变量ThreadLocal
    -   线程本地变量：每个线程都有一个变量的独有拷贝，这个变量不会被其他线程所更改
    -   initialValue：初始值。threadLocal设置值前或删除值后都会为当前值
-   使用场景
    -   日期处理
        -   日期和时间的操作不是线程安全的
        -   通过ThreadLocal，每个线程使用自己的DateFormat，这样解决了线程安全问题
    -   随机数
        -   使用ThreadLocal减少竞争
        -   Random是线程安全的，但是如果并发访问激烈的话，性能会下降。所以Java提供了类ThreadLocalRandom，通过静态方法current获取对象
    -   上下文信息：保存线程执行过程的全局信息
-   原理
    -   每个线程都有一个ThreadLocalMap，对于每个ThreadLocal对象，调用其get/set实际上就是以ThreadLocal对象为键读写当前线程的Map
    -   ThreadLocalMap的键类型为是WeakReference\<ThreadLocal>

### #20 并发总结