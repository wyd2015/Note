# Java — 三种阻塞队列，四种拒绝策略，五种线程池
```java
// 基础参数
int corePoolSize = 2;
int maximumPoolSize = 5;
int keepAliveTime = 5;
TimeUnit unit = TimeUnit.SECONDS;

// 阻塞队列
BlockingQueue<Runnable> workQueue = null;
workQueue = new ArrayBlockingQueue<>(5); // 基于数组的队列，有界
workQueue = new LinkedBlockingQueue<>(); // 基于链表的队列，无界
workQueue = new SynchronousQueue<>(); // 无缓冲的等待队列，无界

// 拒绝策略
RejectedExecutionHandler rejected = null;
rejected = new ThreadPoolExecutor.AbortPolicy(); // 默认，队列满了丢任务、抛异常
rejected = new ThreadPoolExecutor.DiscardPolicy(); // 队列满了，丢任务、不抛异常
rejected = new ThreadPoolExecutor.DiscardOldestPolicy(); // 将最早进入队列的任务删除，之后再尝试加入队列
rejected = new ThreadPoolExecutor.CallerRunsPolicy(); // 如果添加到线程池失败，那么主线程会自己去执行该任务

// 五种线程池
ExecutorService threadPool = null;
threadPool = Executors.newCachedThreadPool(); // 有缓冲的线程池，线程数由JVM控制
threadPool = Executors.newFixedThreadPool(3); // 固定大小的线程池
threadPool = Executors.newScheduledThreadPool(2); // 
threadPool = Executors.newSingleThreadExecutor(); // 单线程的线程池，只有一个线程在工作
threadPool = new ThreadPoolExecutor(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, rejected); // 默认线程池，可控制参数比较多
```