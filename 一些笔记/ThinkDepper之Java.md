1. collection

1. List -> ArrayList/LinkedList/Vector
  List特点: 1. 元素可以重复 2. 顺序是插入的顺序 3. 可以插入null  
ArrayList -> 
    -> ArrayList额外特点: 1. RandomAccess 可以随机访问

    ->数据结构
        -> Object[] elementData -> elementData.size() 是实际容量，没有指定，初始化默认是10
        -> int size -> 是已经有的数据大小
    -> add()
        -> ensureCapacityInternal(size + 1) -> 这里可能会grow()
        ->  elementData[size++] = e;
    -> get()
        -> rangeCheck(index); 只是检测上限， 下线是JVM的数据越界
        -> return elementData(index);
    -> grow(minCapacity);
        -> 默认增长到1.5倍，如果minCapacity > 1.5倍 newCapacity = minCapacity
        -> Arrays.copyOf(elementData, newCapacity);
    -> 序列化 TODO

LinkedList -> 
    -> 数据结构
        -> transient Node<E> first;
        -> transient Node<E> last;
        -> transient int size = 0;
            -> Node  -> 双向的节点
                -> E item; 
                -> Node<E> next;
                -> Node<E> prev;
    -> add()
        -> linkLast(e);
    -> get() -> node()会以O(n/2)的性能去获取一个结点 -> 如果索引值大于链表大小的一半，那么将从尾结点开始遍历 ->使用空间（双向链表）来换取时间。

HashMap -> 
    -> k,v 存储，o(1)的查找效率
    -> 面试trap -> 1.设置初始的capacity，需要考虑避免rehash 0.75x16 = 12; 2. loadFactory的大小设置作用；3.hash算法考虑

    ->数据结构
        -> transient Node<K,V>[] table;
            ->  static class Node<K,V> implements Map.Entry<K,V>
                -> final int hash;
                -> final K key;
                -> V value;
                -> Node<K,V> next;
    -> hash(key) -> hash算法
        -> return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16); 
    -> V put(K key, V value) 
        -> table 初始化判断
        -> tab[i] == null
            -> tab[i] = newNode();
        -> 链表或tree (TreeNode)
    -> resize()

2.

