## ReentrantLock
>ReentrantLock是可重入排他锁。 

##### 实现锁的原理：

1. 维护一个lock status,是否已经被某个线程获取，获取了几次(可重入) <br>
2. 维护一个waiting list,等待获取锁的列表，当占用的锁被释放，在等待列表中获取一个正在等待的节点唤醒让它取得锁继续运行 <br>

    -- 另外的功能：锁的非公平与公平实现，超时等待锁，等待中断...

ReentrantLock借助用AQS(AbstractQueuedSynchronizer)实现。 <br>
lock statu 维护: private volatile int state;<br>
waiting list 维护：private transient volatile Node head,tail;<br>

##### 加锁lock() (非公平)
```
final void lock() {
    //直接尝试cas,成功则说明获取成功，设置当前的锁占用线程
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
    //已经被占用执行逻辑，acquire()是AQS的模板方法
        ，acquire(1);
}

//AQS
public final void acquire(int arg) {
    //再尝试获取一次锁，失败则加入等待队列中等待
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```
##### 获取锁失败，加入等待列表直到获取成功(非中断方式获取)
```
//一直等待直到成功获取锁
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) { // 一直在尝试获取锁，直到成功
            final Node p = node.predecessor();
            //head表示正在持有的锁的节点，仅有head.next节点才可以获取锁
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            //当p.waitStatus = SINGLE,说明当前节点需要被park。LockSupport.park(this);实现挂起
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

##### 释放锁 unlock()
```
 public void unlock() {
    //release是AQS的模板方法
    sync.release(1);
}

public final boolean release(int arg) {
    //尝试释放锁，直到所有的锁都被释放了才是真正释放(state = 0),仅有Thread.currentThread() != getExclusiveOwnerThread()才可以释放锁。tryRelease没有加任何锁，lock(),unlock()happens-before规则
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            //waitStatus没有被cancle，就LockSupport.unpark(s.thread);
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```



