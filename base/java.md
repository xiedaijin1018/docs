# Java


#### 锁
* 原子性: 被加锁的代码，对多个共享资源的操作，同一时间只有一个线程获取到锁，保证代码块是原子性
* 可见性: 当锁释放的时候，会将保证对共享变量的操作结果写回内存，因此变量修改结果对其他线程是可见的
* 有序性: 由于处理器和编译器重排序，所以多线程操作时是无序的，但单线程中遵循as-if-serial原则，不能禁止重排序，但执行结果是不可以变的。加锁代码块由于锁的独占性，只有一个线程获取到锁，执行结果不变，从而能认为线程执行是有序的
* 可重入锁：代表持有锁的线程，在重复获取锁的时候，无需阻塞，能继续获取锁
* 公平锁和非公平锁：对于已有线程在获取锁，即存在等待获取锁的线程等待队列。此时，对于公平锁，其他线程正在获取锁时，直接加入线程等待队列，让之前的线程等待队列先获取锁；非公平锁，则先尝试获取锁，获取成功直接返回，不成功才加入线程等待队列

#### volatile
特性
* 可见性
* 禁止重排序

volatile没有原子性，而锁没有禁止重排序功能

#### synchronized
特性: 原子，可见，有序，可重入，非公平

实现原理(对象头，markword，monitor)
经研究发现大部分时候加锁不一定会触发竞争，但每次加锁都去切换内核态很费时，所有设计了在各种情况下锁的实现方式
* 无锁
* 偏向锁：只有一个线程访问的时候(通过cas获取偏向锁)
* 轻量级锁：当锁是偏向锁的时候来了另一个线程竞争，锁就升级成轻量级锁，新来的线程自旋等待，消耗cpu
* 重量级锁：自旋多次失败后，锁升级为重量级锁

#### LockSupport
通过操作系统的pthread_cond_wait和pthread_cond_signal实现

#### AbstractQueueSynchronizer
通过cas，链表队列，LockSupport的park，unpark来实现

队列中的线程调用park，获得锁的线程退出时会unpark头节点

#### reentranlock
使用AbstractQueueSynchronizer(AQS)实现，支持公平锁和非公平锁

一般用法
```
    ReentrantLock lock = new ReentrantLock();
    lock.lock();
    try{
        // TODO
    }catch(Exception e){
        
    }finally{
        lock.unlock();
    }
```

#### cas
原理：使用操作系统的cas(汇编指令，lock cmpxchg a,b,c)

优点：
* 没有锁，并发量不高下提高效率
* 减少上下文切换

缺点：
* 如果失败会一直尝试，对cpu消耗大
* 只能保证一个变量的原子操作
* ABA问题，就是B线程把值改了一次然后又改回来，对A来说以为值是不变的(解决方案，使用AtomicStampedReference，带有时间戳)

#### wait，notify
一般用法
```
    synchronized (obj) {
        while (<condition does not hold>)
            obj.wait();
        ... // Perform action appropriate to condition
    }
```

底层通过操作系统的pthread_cond_wait和pthread_cond_signal实现

#### 并发工具类
* CountDownLatch：让多个线程等待一系列在其他线程的操作完成
* CyclicBarrier：等待多个线程到达等待区
* Exchanger：用于两个线程等待对方互换数据
* Phaser：加强版的CyclicBarrier和CountDownLatch，用于多阶段的协调等待
* Semaphore：用来控制能拿到资源的线程数量

#### ThreadPoolExecutor和ForkJoinPool
ForkJoinPool使用工作窃取算法，适用于任务可拆分成很多小任务的场景，在子任务中可以通过fork来创建子任务
处理流程
1. 每个Worker线程利用它自己的任务队列维护可执行任务；
2. 任务队列是一种双端队列，支持LIFO的push和pop操作，也支持FIFO的take操作；
3. 任务fork的子任务，只会push到它所在线程（调用fork方法的线程）的队列；
4. 工作线程既可以使用LIFO通过pop处理自己队列中的任务，也可以FIFO通过poll处理自己队列中的任务，具体取决于构造线程池时的asyncMode参数；
5. 当工作线程自己队列中没有待处理任务时，它尝试去随机读取（窃取）其它任务队列的base端的任务；
6. 当线程进入join操作，它也会去处理其它工作线程的队列中的任务（自己的已经处理完了），直到目标任务完成（通过isDone方法）；
7. 当一个工作线程没有任务了，并且尝试从其它队列窃取也失败了，它让出资源（通过使用yields, sleeps或者其它优先级调整）并且随后会再次激活，直到所有工作线程都空闲了——此时，它们都阻塞在等待另一个顶层线程的调用。

核心就是在执行任务的时候通过fork创建出子任务，并通过join去看子任务是否完成，没完成就去看自己是不是可以去完成它，如果不行就awaitJoin，可能去偷取一个子任务来执行

#### ThreadPoolExecutor
处理流程
1. 默认情况下，创建完线程池后并不会立即创建线程, 而是等到有任务提交时才会创建线程来进行处理。（除非调用prestartCoreThread或prestartAllCoreThreads方法）
2. 当线程数小于核心线程数时，每提交一个任务就创建一个线程来执行，即使当前有线程处于空闲状态，直到当前线程数达到核心线程数。
3. 当前线程数达到核心线程数时，如果这个时候还提交任务，这些任务会被放到工作队列里，等到线程处理完了手头的任务后，会来工作队列中取任务处理。
4. 当前线程数达到核心线程数并且工作队列也满了，如果这个时候还提交任务，则会继续创建线程来处理，直到线程数达到最大线程数。
5. 当前线程数达到最大线程数并且队列也满了，如果这个时候还提交任务，则会触发饱和策略。
6. 如果某个线程的控线时间超过了keepAliveTime，那么将被标记为可回收的，并且当前线程池的当前大小超过了核心线程数时，这个线程将被终止。

工作队列
* ArrayBlockingQueue
* LinkedBlockingQueue
* SynchronousQueue
* PriorityBlockingQueue
* DelayedWorkQueue

拒绝策略
* AbortPolicy
* DiscardPolicy
* DiscardOldestPolicy
* CallerRunsPolicy

#### threadLocak原理
ThreadLocal本身只是一个key，真正的值存储在Thread的ThreadLocalMap里面。ThreadLocalMap的entry是一个弱引用

内存泄漏: 就算ThreadLocal被回收后，ThreadLocalMap里的entry只是key变成null，value和entry都还存在，直到下次调用ThreadLocalMap的set，get，remove方法才会清除掉这个entry，或者等线程结束

解决内存泄漏方法：调用ThreadLocal的remove方法

#### 强引用，弱引用，软引用，虚引用
* 强引用(StrongReference)：最普遍的引用，垃圾回收收不了，没有内存就oom
* 软引用(SoftReference): 内存足够就不回收，内存不足就回收，用来实现本地缓存
* 弱引用(WeakReference)：不管内存是否足够，只要扫描到就回收
* 虚引用(PhantomReference)：任何时候都会被回收