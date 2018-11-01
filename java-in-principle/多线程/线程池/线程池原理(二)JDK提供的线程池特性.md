## 线程池原理(二) 线程池工厂提供的线程特性

Executors是线程池工厂，通过不同的方法获取不同类型的线程池 <br>

* **newFixedThreadPool**：持有固定工作线程的线程池。corePoolSize = maximumPoolSize, blockingQueue= new LinkedBlockingQueue<Runnable>()
* **newSingleThreadExecutor**:仅有一个工作线程的线程池。corePoolSize = maximumPoolSize = 1, blockingQueue=new LinkedBlockingQueue<Runnable>()。
* **newCachedThreadPool**：按需创建工作线程,线程会缓存一段时间。corePoolSize = 0; maximumPoolSize = Integer.MAX_VALUE,keepAliveTime = 60L,TimeUnit=TimeUnit.SECONDS,blockingQueue=new SynchronousQueue<Runnable>()
* **newScheduledThreadPool**:周期性执行线程池


