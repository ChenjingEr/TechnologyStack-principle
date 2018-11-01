#### 线程复用

> 线程复用，避免大量线程的创建和销毁的开销，避免创建大量的线程占用内存。
 
---
###### 1.1 Executor涉及到的类

&nbsp; **Executor中涉及到任务，任务的执行，执行的结果3个方面的内容**</br>
&nbsp; **任务：**用于定义被执行的任务是什么。包括Runable接口，Callable接口。Runnable没有返回值，不可throw异常，Callable可以有返回值，也可以有异常</br>

&nbsp; Runnable 接口
```
	public abstract void run();
```
&nbsp;Callable 接口
```
	V call() throws Exception;
```
	
&nbsp;   **任务执行:**</br>
&nbsp;&nbsp; interface Executor 定义了一个执行实现了Runnable的接口execute。ExecuteService扩展了Executor，定义了执行Callable接口的submitz执行方法，还定义了一些与Pool生命周期相关的一些方法。ThreadPoolExecutor是具体实现

&nbsp;   **任务的结果:**TODO
异步执行的结果用Future / FutureTask 表示

###### 2.1 默认线程池
+ newFixedThreadPool
+ newCachedThreadPool
+ newSingleThreadPool
+ newScheduleThreadPool

###### 3.1 线程池生命周期

* java.util.concurrent.ExecutorService.shutdown();
* java.util.concurrent.ExecutorService.shutdownNow()
* java.util.concurrent.ExecutorService.awaitTermination(long, TimeUnit)
* java.util.concurrent.ExecutorService.isShutdown()
* java.util.concurrent.ExecutorService.isTerminated()

###### 4.1 自定义线程池
1. size: corePoolSize, maxPoolSize
2. workQueue：BlockingQueue<E>
	+ 有界队列 ArrayBlockingQueue
	+ 无界队列 LinkedBlockingQueue 
	+ 优先级队列 PriorityBlockingQueue
	+ 立即提交队列  SynchronousQueue
3. ThreadFactory
4. RejectPolicy
	+ AbortPolicy
	+ DiscardPolicy
	+ DiscardOldestPolicy
	+ CallerRunsPolicy

###### 5.1 线程池的大小（性能）
###### 6.1 扩展ThreadPoolExecutor
* java.util.concurrent.ThreadPoolExecutor.beforeExecute(Thread, Runnable)
* java.util.concurrent.ThreadPoolExecutor.afterExecute(Runnable, Throwable)
* java.util.concurrent.ThreadPoolExecutor.terminated()