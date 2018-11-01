ThreadPoolExecutor

数据结构

private final BlockingQueue<Runnable> workQueue; //等待执行的队列
private final ReentrantLock mainLock = new ReentrantLock(); //
private final HashSet<Worker> workers = new HashSet<Worker>(); 工作线程
private final Condition termination = mainLock.newCondition();
private volatile int corePoolSize;
private volatile int maximumPoolSize;


private static final int RUNNING    = -1 << COUNT_BITS;
private static final int SHUTDOWN   =  0 << COUNT_BITS;
private static final int STOP       =  1 << COUNT_BITS;
private static final int TIDYING    =  2 << COUNT_BITS;
private static final int TERMINATED =  3 << COUNT_BITS;


execute

什么时候执行？ 任务队列怎么执行?

1. 空任务检查

2. 工作线程是否小于核心工作线程？
if (workerCountOf(c) < corePoolSize) {
    if (addWorker(command, true))
        return;
    c = ctl.get();
}

a. 取出工作线程数量  return c & CAPACITY(29个1);
b. boolean addWorker(command, true)  // 添加新工作线程   如果正在工作的线程 < corePoolSize ,则添加工作线程提交任务 。 （返回false的情况）

3. 等待队列是否满了？ 添加队列

4. addWorker 正在工作的线程小于maxPoolSize 

5. RejectedExecutionHandler.rejectedExecution

添加一个新的工作线程 addWorker

addWorker(Runnable firstTask, boolean core)

1. boolean core 表示当前线程数量是否小于corePoolSize
2. 线程池的状态检测
3. 如果当前工作线程大于corePoolSize/maxPoolSize(取决于boolean core)，return false;
4. compareAndIncrementWorkerCount(c) 工作线程+1（cas）
5. 添加 worker 
	1. new 一个worker， worker的线程是通过ThreadFactory得到的
	2. lock 中添加线程 workers.add(w); 添加成功 通过 t.start启动work（是启动work本身而不是启动execute(runnable进来的任务)） return true

一个Woker是一个AQS 以及 一个Runnable

runWorker

//扩展的方法
beforeExecute()
task.run()
//扩展的方法
afterExecute()

//task 和 workQueue都执行完， 当前线程会怎样？
processWorkerExit()
 先remote worker 
 再添加一个 runnable==null的wokder  (执行完之后最大的工作线程 == maxPoolSize)

workers.remove(w);
tryTerminate()





1. 任务的执行  execute 
2. 工作线程  worker , 是一个AQS，也是一个Runnable。 execute的t.start 回调的是worker的 run . 工作线程执行当前的任务或从队列里取一个任务（getTask()）执行。
	getTask()当获取一个Runnable的逻辑
					    if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
                decrementWorkerCount();
                return null;
            }
					boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;
                    Runnable r = timed  ？workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                    workQueue.take(); 

3. 生命周期 
	shutdown()
	1. 池状态set SHUTDOWN
	2. interruptIdleWorkers(); 设置线程终端 t.interrupt();
	3. tryTerminate()  尝试终止线程池
		1. 线程池的状态 == RUNNING  || 线程池的状态 < TIDYING || (线程池的状态 == SHUTDOWN && 等待任务队列为空) return  //workQueue的队列完成
		2. 线程池的状态 == SHUTDOWN && 等待任务队列为空 terminated()(空方法)，线程池的状态 == TIDYING   // workQueue的队列完成后，状态->TIDYING 执行terminated()之后 线程池的状态 == TERMINATED  

	shutdownNow() 如何让线程立即停止？
	1. 线程池的状态 == STOP
	2. tryTerminate()

	awaitTermination(long timeout, TimeUnit unit)
	1. 在定义的时间内 for(;;) 查询线程池的状态是否是 TERMINATED



4. 工作线程的存活时间

当工作线程执行完毕，且任务队列没有任务后，工作线程的数量
1. 用无界任务队列且没有shutDown()/shutDownNow(), countOfWorks = corePoolSize
2. 用无界任务队列且调用了shutDown()/shutDownNow(), countOfWorks = 0
3. 用有界任务队列，没有调用shutDown()/shudDownNow()， countOfWorks = corePoolSize
4. 用有界任务队列且调用了shutDown()/shutDownNow(), countOfWorks = 0

5. 拒绝策略 
RejectedExecutionHandler
CallerRunsPolicy
AbortPolicy
DiscardPolicy
DiscardOldestPolicy

submit(Callable task)

1. 任务如何执行？
	1. 任务转换成Runnable 
	newTaskFor(task)  将一个Callable task 转换成一个 RunnableFuture的task
		
		public interface RunnableFuture<V> extends Runnable, Future<V>
    RunnableFuture 是一个Runnable 也是一个 Future（代表异步执行的结果）
    2. execute（task）
2. 结果怎么获取？
	返回一个Future，实现类是FutureTask
	FutureTask 中run 调用 task.call,完成后setResult, LockSupport.unpark waitNote
	get() 线程成为一个waitNode, LockSupport.park



1. 线程池总结
  任务的表示？
  任务的执行？
  任务执行的结果？
  执行完之后线程的各种参数是什么？
  线程的生命周期方法？
  awaitToTerminate 方法