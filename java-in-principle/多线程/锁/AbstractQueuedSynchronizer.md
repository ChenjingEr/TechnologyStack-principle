## AbstractQueuedSynchronizer 详解
>AbstractQueuedSynchronizer（AQS）队列同步器是Java中实现各种锁，同步工具的基本工具。AQS维护一个int变量表示同步状态，一个FIFO队列表示线程等待 </br>
AQS定义模板方法实现同步状态的获取，中断获取，释放等操作。锁，同步工具聚合AQS，重写同步器的其他方式实现类。<br>
等待队列的head是获取同步同步状态成功的节点

#### AbstractQueuedSynchronizer 源码详解
##### 数据结构
```
public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer
    implements java.io.Serializable {
    	...
    	//等待队列头
        private transient volatile Node head;
        //等待队列尾
        private transient volatile Node tail;
        //同步状态
        private volatile int state;
    	...
    }
```
##### Node结构
```
static final class Node {
 	static final Node SHARED = new Node();	
 	static final Node EXCLUSIVE = null;
 	volatile int waitStatus;
 	volatile Node prev;
 	volatile Node next;
 	volatile Thread thread;
 	Node nextWaiter;
}
```

##### 状态相关方法
```
    protected final int getState() {
        return state;
    }

    protected final void setState(int newState) {
        state = newState;
    }

    protected final boolean compareAndSetState(int expect, int update) {
        return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
    }
```

##### 添加到等待队列且自旋等待获取
```
    private Node addWaiter(Node mode) {
        //新建一个Node，mode表示被独占或共享
        Node node = new Node(Thread.currentThread(), mode);
        //尝试快速设置尾节点
        Node pred = tail;
        if (pred != null) {
            node.prev = pred;
            //CAS设置tail
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
        enq(node);
        return node;
    }

    private Node enq(final Node node) {
        for (;;) {
            Node t = tail;
            if (t == null) { // Must initialize
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
                node.prev = t; 
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
    }
```
新建一个node,先尝试插入到尾节点后面。如果失败就自旋，直到添加成功为止。
```
    final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```
自旋尝试获取同步状态，成功则返回。





shouldParkAfterFailedAcquire()检测是否需要阻塞当前线程(根据前一个节点的waitStatus),删除取消状态的线程
```
    private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        int ws = pred.waitStatus;
        //前一个节点是SINGNAL,则当前节点需要等待
        if (ws == Node.SIGNAL)
            return true;
        //ws>0 表示线程被中断或超时，删除等待线程
        if (ws > 0) {
            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {
            //设置等待状态
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        return false;
    }
```
```
    private final boolean parkAndCheckInterrupt() {
        LockSupport.park(this);
        return Thread.interrupted(); 
    }
```
如果需要阻塞，使用LockSupport.park。

---
####独占式获取与释放
##### AQS模板方法之acquire(int arg)独占式获取同步状态
```
    public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
```
先尝试获取锁，成功则返回。不成功添加到等待队列，然后循环等待直到获取成功。


##### AQS模板方法之acquireInterruptibly(int arg),独占式获取锁，可响应中断
```
    public final void acquireInterruptibly(int arg)
            throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        if (!tryAcquire(arg))
            doAcquireInterruptibly(arg);
    }

    private void doAcquireInterruptibly(int arg)
        throws InterruptedException {
        //添加到等待队列中
        final Node node = addWaiter(Node.EXCLUSIVE);
        boolean failed = true;
        try {
            for (;;) {
                //只有当前节点的pre是head,才可以尝试获取锁。
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return;
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    //如果线程调用了中断，抛出异常，实现中断
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                //中断后，取消等待
                cancelAcquire(node);
        }
    }
```
先判断线程是否被中断，如果中断抛出InterruptedException异常</br>
尝试获取锁，成功直接返回。失败执行doAcquireInterruptibly。</br>
doAcquireInterruptiblu先添加到等待队列中，在自旋中，只有当前节点的pre是head才尝试获取锁(因为头节点是获取同步状态成功的节点，头节点释放锁后唤醒下一个节点)，


#####AQS模板方法之tryAcquireNanos(int arg, long nanosTimeout),超时等待
```
    public final boolean tryAcquireNanos(int arg, long nanosTimeout)
            throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        return tryAcquire(arg) ||
            doAcquireNanos(arg, nanosTimeout);
    }

     private boolean doAcquireNanos(int arg, long nanosTimeout)
            throws InterruptedException {
        if (nanosTimeout <= 0L)
            return false;
        final long deadline = System.nanoTime() + nanosTimeout;
        //添加一个独占节点到等待队列
        final Node node = addWaiter(Node.EXCLUSIVE);
        boolean failed = true;
        try {
            //自旋尝试获取锁，检测超时
            for (;;) {
                //尝试获取锁，获取成功返回true
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return true;
                }
                nanosTimeout = deadline - System.nanoTime();
                if (nanosTimeout <= 0L)
                    return false;
                //大于spinForTimeoutThreshold时间间隔就park一段时间
                if (shouldParkAfterFailedAcquire(p, node) &&
                    nanosTimeout > spinForTimeoutThreshold)
                    LockSupport.parkNanos(this, nanosTimeout);
                //线程中断，抛出异常
                if (Thread.interrupted())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```
先尝试获取锁，获取失败执行doAcquireNanos()方法</br>
doAcquireNanos()除了尝试获取锁同时记录超时，超过时间没有成功，返回false

##### AQS模板方法之release(int arg)释放锁
```
    public final boolean release(int arg) {
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }

    private void unparkSuccessor(Node node) {
        int ws = node.waitStatus;
        if (ws < 0)
            compareAndSetWaitStatus(node, ws, 0);
        Node s = node.next;
        if (s == null || s.waitStatus > 0) {
            s = null;
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t;
        }
        if (s != null)
            //唤醒head之后的节点
            LockSupport.unpark(s.thread);
    }
```

####共享获取 
##### AQS模板方法之acquireShared(int arg)获取共享锁
```
    public final void acquireShared(int arg) {
        //先尝试获取共享锁
        if (tryAcquireShared(arg) < 0)
            doAcquireShared(arg);
    }

    private void doAcquireShared(int arg) {
        //添加一个共享节点到队列
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                final Node p = node.predecessor();
                if (p == head) {
                    int r = tryAcquireShared(arg);
                    if (r >= 0) {
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        if (interrupted)
                            selfInterrupt();
                        failed = false;
                        return;
                    }
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```
---
####释放


---
#####AQS总结