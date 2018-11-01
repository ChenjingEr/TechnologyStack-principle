## ReentrantReadWriteLock 详解
>ReentrantReadWriteLock是读锁与写锁分离的一种锁。读锁可以同时被多个线程获取，写锁的获取是互斥的。读锁与写锁是互斥的。可重入

#### ReentrantReadWriteLock 源码解析

##### 属性

```
public class ReentrantReadWriteLock
        implements ReadWriteLock, java.io.Serializable {
    ...
    private final ReentrantReadWriteLock.ReadLock readerLock;
    private final ReentrantReadWriteLock.WriteLock writerLock;
    final Sync sync;
    ...
}
```

##### 获取读写锁
读锁写锁是分离的，通过不同的方法获取。
```
    public ReentrantReadWriteLock.WriteLock writeLock() { return writerLock; }
    public ReentrantReadWriteLock.ReadLock  readLock()  { return readerLock; }

```

分别分析读锁(ReadLock)/写锁(ReadLock)</br>

####ReentrantReadWriteLock.ReadLock解析
##### 读属性
```
public static class ReadLock implements Lock, java.io.Serializable {
	...
	private final Sync sync;
	...
}
```
##### lock()
```
    public void lock() {
    	//获取共享锁
        sync.acquireShared(1);
    }

     protected final int tryAcquireShared(int unused) {

        Thread current = Thread.currentThread();
        int c = getState();
        //锁已经被占用且是独占的
        if (exclusiveCount(c) != 0 &&
            getExclusiveOwnerThread() != current)
            return -1;
        //获取已经被占用的共享锁的数量
        int r = sharedCount(c);
        if (!readerShouldBlock() &&
            r < MAX_COUNT &&
            compareAndSetState(c, c + SHARED_UNIT)) {
            if (r == 0) {
                firstReader = current;
                firstReaderHoldCount = 1;
            } else if (firstReader == current) {
                firstReaderHoldCount++;
            } else {
                HoldCounter rh = cachedHoldCounter;
                if (rh == null || rh.tid != getThreadId(current))
                    cachedHoldCounter = rh = readHolds.get();
                else if (rh.count == 0)
                    readHolds.set(rh);
                rh.count++;
            }
            return 1;
        }
        return fullTryAcquireShared(current);
    }
```
在读锁获取时不可已经有写锁在工作。
