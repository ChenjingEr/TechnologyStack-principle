##Condition 详解
>Condition配合ReentrantLock可以实现等待/通知的机制。
await()在await()前需要先获取锁。进入await()线程释放锁，进入等待状态直到被中断或者被signle()/singleAll()
single()/single()唤醒在Condition等待的线程，执行之前需要获取锁。

####Condition 源码解析
>Condtion只是一个接口。主要分析在AQS中Condtion的实现，ConditionObject。
分析过程中主要解决。await()如何释放锁？如何等待？如何被唤醒？唤醒后如何继续执行等问题

##### ConditionObject的属性
```
 public class ConditionObject implements Condition, java.io.Serializable {
 	...
 	private transient Node firstWaiter;
 	private transient Node lastWaiter;
 	...
 }
```
ConditionObject维护一个等待的队列。

##### ConditionObject.await() 进入等待
```
    public final void await() throws InterruptedException {
    	//响应中断
        if (Thread.interrupted())
            throw new InterruptedException();
        //新建一个节点，添加到Condition的等待队列上
        Node node = addConditionWaiter();
        //释放所有获取的锁
        int savedState = fullyRelease(node);
        int interruptMode = 0;
        //一直循环查节点时候进入同步队列，保持等待
        while (!isOnSyncQueue(node)) {
        	//阻塞当前线程
            LockSupport.park(this);
            //检测中断，如果线程被中断，会将node转入同步队列
            if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                break;
        }
        //重新获取同步状态（锁），THROW_IE表示线程被中断
        if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
            interruptMode = REINTERRUPT;
        //nextWaiter表示下一个等待的节点
        if (node.nextWaiter != null) // clean up if cancelled
            unlinkCancelledWaiters();
        if (interruptMode != 0)
            reportInterruptAfterWait(interruptMode);
    }
```
await()释放锁通过fullyRelease(node),通过一直循环检测当前的节点是否在同步队列中来实现一直等待。响应中断是通过中断状态检测状态来break循环。重新获取锁acquireQueued(node, savedState)，直到成功（隐式获取锁，当前在await()park, 获取成功则继续await()下执行）

##### ConditionObject.signal()
```
    public final void signal() {
    	//需要是锁的获得者
        if (!isHeldExclusively())
            throw new IllegalMonitorStateException();
        Node first = firstWaiter;
        if (first != null)
            doSignal(first);
    }

    private void doSignal(Node first) {
        do {
            if ( (firstWaiter = first.nextWaiter) == null)
                lastWaiter = null;
            first.nextWaiter = null;
        } while (!transferForSignal(first) &&
                 (first = firstWaiter) != null);
    }

    final boolean transferForSignal(Node node) {
    	//设置waitStatus = 0
        if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
            return false;
        //添加到队列
        Node p = enq(node);
        int ws = p.waitStatus;
        if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
            // unpark
            LockSupport.unpark(node.thread);
        return true;
    }
```