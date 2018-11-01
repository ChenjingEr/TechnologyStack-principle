#### BlockingQueue 详解

BlockingQueue是实现了FIFO的队列，且提供阻塞访问，也就是当队列为空，读操作阻塞；当队列(有限)为满，写操作阻塞。

BlockingQueue API中几个方法比较:

插入操作:
add(e):插入队列，成功返回true,失败抛出异常
offer(e):插入队列，成功返回true,失败返回false
put(e):插入队列，一直阻塞到插入成功(有位置可以插入)
offer(e,timeout):插入队列，失败不立即返回false,在timeout的时间范围内等待。

读取操作:
remove(e)：元素如果存在，移除元素，成功返回true。
take():阻塞等待直到有元素可以读取(读取并移除)
poll(time, unit):阻塞一段时间直到有元素可以读取(读取并移除)

