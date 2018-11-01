## ConcurrentHashMap 详解(二)

#### ConcurrentHashMap 源码解析 JDK 1.8
##### *putVal() 中操作解析：

+ 初始化
```
    private final Node<K,V>[] initTable() {
        Node<K,V>[] tab; int sc;
        while ((tab = table) == null || tab.length == 0) {
            if ((sc = sizeCtl) < 0)
                Thread.yield(); // lost initialization race; just spin
            else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                try {
                    if ((tab = table) == null || tab.length == 0) {
                        int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                        @SuppressWarnings("unchecked")
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                        table = tab = nt;
                        sc = n - (n >>> 2);
                    }
                } finally {
                    sizeCtl = sc;
                }
                break;
            }
        }
        return tab;
    }
```
Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n]; 初始化，n默认值是DEFAULT_CAPACITY。</br>
sizeCtl 是初始化和调整大小的控制器。当sizeCtl < 0表示正在初始化或者resize,-1表示初始化。当sizeCtl=0时，使用默认大小初始化。初始化之后，保存的是下一个需要调整的大小（Threshold)(当n=16, n - (n >>> 2) = 12)</br>
sizeCtl 是**volatile**,所以修改对其他线程可见
+ addCount TODO