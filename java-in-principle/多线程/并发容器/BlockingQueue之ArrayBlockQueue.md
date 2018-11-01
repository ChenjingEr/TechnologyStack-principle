## ArrayBlockingQueue 详解
>ArrayBlockQueue是一个数组阻塞队列实现。队列满时，put()阻塞，add() / offer()返回false。队列为空时，take()阻塞，peek() / poll() 返回null。

#### ArrayBlockingQueue 源码解析
##### 数据结构
```
public class ArrayBlockingQueue<E> extends AbstractQueue<E>
        implements BlockingQueue<E>, java.io.Serializable {
        	...
        	//用数组存储元素
        	final Object[] items;
        	final ReentrantLock lock;
        	// take等待条件
			private final Condition notEmpty;
			//put等待条件
			private final Condition notFull;
        	...
        }

```
用可重入锁保证同步，Condition保证等待条件

##### ArrayBlockQueue.put()方法（队列满时，阻塞线程）
```
    public void put(E e) throws InterruptedException {
        checkNotNull(e);
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == items.length)
                notFull.await();
            enqueue(e);
        } finally {
            lock.unlock();
        }
    }

    private void enqueue(E x) {
        // assert lock.getHoldCount() == 1;
        // assert items[putIndex] == null;
        final Object[] items = this.items;
        items[putIndex] = x;
        if (++putIndex == items.length)
            putIndex = 0;
        count++;
        notEmpty.signal();
    }
```
put元素不可可为空</br>
count == items.length队列满时，线程在 notFull上等待。</br>
队列有空位，添加元素到最后，且唤醒消费者</br>

##### ArrayBlockQueue.take()方法（队列空时，阻塞线程）
```
    public E take() throws InterruptedException {
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == 0)
                notEmpty.await();
            return dequeue();
        } finally {
            lock.unlock();
        }
    }

    private E dequeue() {

        final Object[] items = this.items;
        @SuppressWarnings("unchecked")
        E x = (E) items[takeIndex];
        items[takeIndex] = null;
        if (++takeIndex == items.length)
            takeIndex = 0;
        count--;
        if (itrs != null)
            itrs.elementDequeued();
        notFull.signal();
        return x;
    }   
```

count == 0,线程阻塞等待</br>
否则取出队头元素。队头通过takeIndex记录位置。items[takeIndex] = null;置空该位置表示删除。更新count。然后唤醒在notFull上等待的一个线程

##### ArrayBlockQueue.offer()方法（不阻塞添加）
```
    public boolean offer(E e) {
        checkNotNull(e);
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            if (count == items.length)
                return false;
            else {
                enqueue(e);
                return true;
            }
        } finally {
            lock.unlock();
        }
    }
```
offer,队列满时，return false. 否者添加到队尾

##### ArrayBlockQueue.poll()方法（不阻塞获取并删除队头）
```
    public E poll() {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            return (count == 0) ? null : dequeue();
        } finally {
            lock.unlock();
        }
    }
```