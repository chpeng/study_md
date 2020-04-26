# java 线程池

## 创建线程池

![image-20200411184036548](Java%E7%BA%BF%E7%A8%8B%E6%B1%A0.assets/image-20200411184036548.png)

1. 使用 Executors 工具类创建

   1. newSingleThreadExecutor 、newFixedThreadPool 和 newScheduledThreadPool () 都是使用的无解队列(BlockingQueue)，容易造成OOM

   2. newCachedThreadPool  maximumPoolSize是integer的最大值，会无线创建线程


2. 通过new ThreadPoolExecutor（）创建

   1. int corePoolSize

      线城池中长期存活的核心线程数

   2. int maximumPoolSize

      线程中最大工作线程的数量

   3. long keepAliveTime

      非核心线程在空闲时存活的时间

   4. TimeUnit unit

      非核心线程空闲时，存活时间的单位

   5. BlockingQueue<Runnable> workQueue

      核心线程都在工作时，新产生的任务队列

   6. ThreadFactory threadFactory

      生成线程的工厂

   7. RejectedExecutionHandler handler

      当队列满了，线程池中没有可用线程时的拒绝策略

## 线程池工作原理

1. 在创建线程池时，等待提交任务

2. 当调用 executor() 时,线程池会做如下判断

   1. 判断正在执行的线程数是否小于corePoolSize，如果小于，则立即执行任务
   2. 正在执行的线程数大于corePoolSize,则将这个任务放入队列
   3. 如果队列已经满了，且正在执行的线程数小于maximumPoolSize，则创建非核心线程，立即执行这个任务
   4. 如果队列满了，且正在执行的线程数 >= maximumPoolSize，那么线程池怎么启动拒绝策略执行

3. 当线程完成任务时，则从队列中取下一个任务执行

4. 当队列没有没有可执行任务是，查过keepAliveTime事件就会将非核心线程停掉

   ![image-20200411184128785](Java%E7%BA%BF%E7%A8%8B%E6%B1%A0.assets/image-20200411184128785.png)

## 线程池的拒绝策略

1. AbortPolicy 当队列满时，则直接抛出异常
2. CallerRunsPolicy 当队列满时，不会抛出错误，会将任务返回给调用者
3. DiscardOldestPolicy 去掉队列中等待时间最长的任务
4. DiscardPolicy 直接丢弃任务

## 线程池数量配置

* 如果是CPU密集型计算，则尽量减少线程数，从而减少线程之间切换，数量一般就是CUP数量+1 一个线程的线程池
* 如果是IO型计算，则需要配置多的线程，以为io密集型阻塞事件较长，需要更多的线程去处理，数量一般设置成  `CUP核心数 / (1-阻塞系数)`



