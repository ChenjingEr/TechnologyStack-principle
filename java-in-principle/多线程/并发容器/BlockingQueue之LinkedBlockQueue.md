## LinkedBlockingQueue 详解
>LinkedBlockQueue是一个链表的队列实现。默认情况下容量为Integer.MAX_VALUE，如果put时容量没有超出这个值，
不会阻塞。

#### LinkedBlockingQueue 源码解析
##### 数据结构
```
public class LinkedBlockingQueue<E> extends AbstractQueue<E>
        implements BlockingQueue<E>, java.io.Serializable {
        	...
        	//头节点
        	transient Node<E> head;
        	//尾节点
        	private transient Node<E> last;
        	//保存的元素数量
        	private final AtomicInteger count = new AtomicInteger();
        	//读取相关锁
        	private final ReentrantLock takeLock = new ReentrantLock();
        	private final Condition notEmpty = takeLock.newCondition();
        	//添加相关锁
        	private final ReentrantLock putLock = new ReentrantLock();
        	private final Condition notFull = putLock.newCondition();
        	...

        }
```
LinkedBlockingQueue用了两个锁分别对应读写，也就是LinkedBlockingQueue可以支持同时的读，写。</br>
元素用Node保存，Node结构如下
```
    static class Node<E> {
        E item;
        Node<E> next;
        Node(E x) { item = x; }
    }
```

##### LinkedBlockingQueue.put()
```
    public void put(E e) throws InterruptedException {
        if (e == null) throw new NullPointerException();
        int c = -1;
        Node<E> node = new Node<E>(e);
        final ReentrantLock putLock = this.putLock;
        final AtomicInteger count = this.count;
        putLock.lockInterruptibly();
        try {
  
            while (count.get() == capacity) {
                notFull.await();
            }
            enqueue(node);
            c = count.getAndIncrement();
            if (c + 1 < capacity)
                notFull.signal();
        } finally {
            putLock.unlock();
        }
        if (c == 0)
            signalNotEmpty();
    }

    private void enqueue(Node<E> node) {
        
        last = last.next = node;
    }

    private void signalNotEmpty() {
        final ReentrantLock takeLock = this.takeLock;
        takeLock.lock();
        try {
            notEmpty.signal();
        } finally {
            takeLock.unlock();
        }
    }
```
添加的元素不可以为空</br>
新的元素新建一个Node</br>
用putLock控制多线程访问,链表中元素的数量=capacity时,线程一直阻塞。当其他的线程put成功且c + 1 < capacity，会唤醒</br>
新元素添加到队列的队尾，添加到队尾的顺序时，设置新元素为原来队尾的next元素，再更新last指向新建的Node。


##### LinkedBlockingQueue.take()
```
    public E take() throws InterruptedException {
        E x;
        int c = -1;
        final AtomicInteger count = this.count;
        final ReentrantLock takeLock = this.takeLock;
        takeLock.lockInterruptibly();
        try {
            while (count.get() == 0) {
                notEmpty.await();
            }
            x = dequeue();
            c = count.getAndDecrement();
            if (c > 1)
                notEmpty.signal();
        } finally {
            takeLock.unlock();
        }
        if (c == capacity)
            signalNotFull();
        return x;
    }

    private E dequeue() {
        // assert takeLock.isHeldByCurrentThread();
        // assert head.item == null;
        Node<E> h = head;
        Node<E> first = h.next;
        h.next = h; // help GC
        head = first;
        E x = first.item;
        first.item = null;
        return x;
    }

    private void signalNotFull() {
        final ReentrantLock putLock = this.putLock;
        putLock.lock();
        try {
            notFull.signal();
        } finally {
            putLock.unlock();
        }
    }
```

