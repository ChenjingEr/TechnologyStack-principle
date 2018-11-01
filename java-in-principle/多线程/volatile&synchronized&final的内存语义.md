## volatile的内存语义

*  volatile特性：
    -  提供一种免锁的机制。没有加锁，不会使线程阻塞。
    -  volatile变量的更新对其他线程可见
    -  (2种说法)volatile不会被存入缓存(或寄存器)或其他处理器不可见的地方 | volatile的更新会发信息给读取线程，缓存失效，要求重新read,load。都是保证其可见性。
    -  volaile标识这个变量不允许指令重排序
* volatile的内存语义:
    - volatile的happens-before规则，一个线程读volatile变量，一定能够见到前一个线程对该变量的更新(可见性)
    - 线程对volatile变量的写立马刷新到主内存。普通变量不一定立马刷新，会批量刷新(对主线占用的次数，以及重复写对主线占用的优化)
    - 对volatile的读类似进入同步代码块，对volatile的写类似走出同步代码块、
* volatile最佳实践：
    - 对变量的写入不依赖当前变量的值，或确保只有单线程更新该变量(⭐)
    - 该变量不会与其他变量一起纳入不变性条件
    - 在访问变量的时候不需要加锁
* volatile局限性(不可代替synchronized的原因):
    - volatile不确保对一个变量的复合操作是原子的。也就是 
```
    private static volitale int i = 0;

    ....

    i++ //这个过程不是原子的
```

i++对应的字节码指令包括
```
getstatic 此时保证是最新的
iconst_1  不保证最新，操作的可能以及是旧的值
iadd      不保证最新，操作的可能是旧的值
putstatic 更新了之后刷回内存，但是已经覆盖了旧的值
```

这个过程中不保证原子性，在读取后操作的过程有其他的线程在修改该变量。这个时候需要有同步。
    
## synchronized的内存语义

* synchrnozed 特性：
    - 锁，同一时间仅有一个线程获取到锁，进入临界区
    - 会导致没有获取到锁的线程阻塞(Blocked)
* synchrnozed 内存语义
synchronized保证操作的原子性以及可见性，编译器在synchronzed的开始以及结尾处插入monitorenter,monitorexist指令实现在访问过程中没有其他的线程可以操作更改变量，保证原子性。在退出monitorexist属性缓存保证可见性。仅有一个线程可以进入monitorenter，monitorexist之间的代码，不存在多线程的并发，保证顺序性(重排序也是需要遵循串行语义)。 <br>

## final的内存语义

* final特性：

造成线程不安全的原因是对一个变量的更新过程没有同步(所有线程只读一个共享变量就不会出现这个问题)，那么有这个一个想法就是让这个变量不可变(没有更新操作)就是线程安全的。 <br>
"不可变对象一定是线程安全的"这只是个粗糙的说法。不可变类型的线程安全需要满足:

* 对象创建后其状态不可改变
* 对象的所有域都是final类型
* 对象是正确创建的(在构造期间没有this逸出)

* final重排序规则：
    - 构造函数对final域的写入，与随后把这个构造对象的引用复制给一个引用变量，不可重排序
    - 构造函数对final引用的对象的成员写入，域所有在构造函数把这个构造函数对象的引用赋值给其他引用变量，不可重排序(保证拿到的引用访问的final域都是已经正确初始化过了)
    - 初次读取final域的对象的引用，与随后初次读取这个final域，这两个操作不能重排序
```
public class FinalExample{ 
    int i;
    final int j;
    static FinalExample obj;

    public FinalExample() {
        i = 1;
        j = 2;   //final域写入,final重排序规则:JVM禁止域写入重排序到构造函数之外。在对象引用为任意线程可见之前，保证正确初始化。
    }

    public static void writer() {
        obj = new FinalExample();   //写线程A执行
    }

    public static void reader() { 
        FinalExample object = obj; //读对象引用
        int a = object.i;    //读对象应用的final域。与上一个操作不可重排序
        int b = object.j;
    }
}
```

* final的逸出:在构造函数中将this发布出去。
```
public class StaticExample{

    private final int i;
    private StaticExample ex;

    public StaticExample() {
        i = 1;  //保证final域的引用赋值在final初始化之后，
        ex = this; //这里还是有可能发生重排序导致发布在初始化之后
    }
}
```
