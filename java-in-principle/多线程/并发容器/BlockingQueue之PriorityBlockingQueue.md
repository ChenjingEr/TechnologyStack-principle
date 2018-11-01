## PriorityBlockingQueue 详解
> PriorityBlockingQueue根据元素的优先级存取元素，PriorityBlockingQueue无界，则保存的时候不会Block。没有提供比较器使用的是默认排序。

#### PriorityBlockingQueue 源码解析
##### 数据结构
```
public class PriorityBlockingQueue<E> extends AbstractQueue<E>
    implements BlockingQueue<E>, java.io.Serializable {
    	...
    	private transient Object[] queue;
    	//根据比较器确定优先级
    	private transient Comparator<? super E> comparator;
		private final ReentrantLock lock;
		private final Condition notEmpty;
    	...
    }
```