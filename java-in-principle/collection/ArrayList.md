## List 之 ArrayList

#### 官方Doc

Resizable-array implementation(大小可调整列表实现) <br>
The size, isEmpty, get, set, iterator, and listIterator operations run in constant time. The add operation runs in amortized constant time, that is, adding n elements requires O(n) time. (利于随机读，顺序插入，不利于在某个位置插入)
permits all elements, including null（可以放任何元素）<br>
This class is roughly equivalent to Vector, except that it is unsynchronized.(与Vector的区别在于是否同步)
its capacity grows automatically(自增长)
List list = Collections.synchronizedList(new ArrayList(...));(本身非同步是实现)
fail-fast

#### 源码实现

```
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable {}
```
*  列表实现
```
private static final int DEFAULT_CAPACITY = 10; //默认大小
transient Object[] elementData; // non-private to simplify nested class access
```
底层是数组实现

*  随机读
```
public E get(int index) {
    //检查的是index>=size的情况，index <0 JVM报异常
    rangeCheck(index);
    return elementData(index);
}

E elementData(int index) {
    return (E) elementData[index];
}
```
随机读，通过索引读数组对应的值。O(1)

*  顺序插入，可插入null
```
 public boolean add(E e) {
    //每次都会有一次check，小了自动扩容
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    //在size位置上放入e
    elementData[size++] = e;
    return true;
}
```
没有null的check,可以插入null值 <br>
check，容量不够自动grow() <br>
放在最后1个位置，之后size++）(size实际有的data数量，elementData.length是容量)

*  随机插入, 可插入null
```
 public void add(int index, E element) {
    rangeCheckForAdd(index);

    ensureCapacityInternal(size + 1);  // Increments modCount!!
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    elementData[index] = element;
    size++;
}
```
*  可调整大小
```
private void grow(int minCapacity) {
   
    int oldCapacity = elementData.length;
    //容量增加1.5倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    //复制
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```