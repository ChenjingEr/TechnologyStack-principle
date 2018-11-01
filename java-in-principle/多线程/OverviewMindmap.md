Concurrent Overview MindMap

Concurrent Queue

1. BlockingQueue

1. 队列实现
2. Blocking 
3. 同步  

ArrayBlockingQueue
添加至队尾

put*(E) : blocking
offer(e):non-blocking

  if (count == items.length)
                return false;

同步

ReentrantLock

blocking 

while (count == items.length)
      notFull.await(); //等待不为满条件

队列实现：

 final Object[] items = this.items;
    items[putIndex] = x;
    if (++putIndex == items.length)
        putIndex = 0; 
    count++;
    notEmpty.signal(); 唤醒poll(),take()操作等待的线程


取出队头：

同步 ReentrantLock lock

blocking :

while (count == 0)
   notEmpty.await();

同步:


 E x = (E) items[takeIndex];
    items[takeIndex] = null;
    if (++takeIndex == items.length)
        takeIndex = 0;
    count--;
    if (itrs != null)
        itrs.elementDequeued();
    notFull.signal();


LinkedBlockingQueue:

插入至队尾：

1. 同步：this.putLock
2. blocking  
	while (count.get() == capacity) { capacity为空时时MAX_VALUE,所以LinkedBlocking并不是无界的
        notFull.await();
    }
3.队列实现：
 Node<E> node = new Node<E>(e); 
 last = last.next = node;
 count.getAndIncrement()
  if (c + 1 < capacity)
           notFull.signal();


取队头：
this.takeLock

 while (count.get() == 0) {
                notEmpty.await();
            }

 Node<E> h = head;
        Node<E> first = h.next;
        h.next = h; // help GC
        head = first;
        E x = first.item;
        first.item = null;



DelayQueue:无界队列，元素在一段时间之后才可以take

PriorityQueue 优先级队列

put = offer 无界队列，non-blocking
available.signal();

take()

一直等待超时才获取


ConcurrentHashMap:

1. HashMap,数据保存的结构
2. 同步

数据保存的结构

transient volatile Node<K,V>[] table;


1. put 

1. inittable

Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
table = tab = nt;

2 tabAt == null casAndSwap 无锁

3. tabAt != null synchronized(tabAt)


hashCode >= 0  

hashCode相同，key == old.key || key.equal(oldKey)
	oldValue替换
tabAt.next == null  tabAt.next = new node

要么替换，要么next

binCount >= TREEIFY_THRESHOLD

treeifyBin (tab,i)

2. get


copyOnWrite  arrayList

1. arrayList 

private transient volatile Object[] array;


2. 同步


Lock

1. 实现基本原理
2. 公平 非公平实现
3. 读 写 实现


1. AQS

lock status 维护   private volatile int state; cas
waiting list 维护  private transient volatile Node head,tail

reentrantLock:

1. lock

没被占用：
if (compareAndSetState(0, 1))
    setExclusiveOwnerThread(Thread.currentThread());

已经被占用

acquire(1)：非公平

 if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();

再次尝试获取锁，失败则添加到waiting queue


condition 

1. 在confidtion上等待:
await() 
signal()/signalAll 













