---
title: Java面试知识点
date: 2019-08-27 19:54:11
tags:
categories: java
---

## JMM

java内存模型试图屏蔽不同硬件与操作系统的内存访问差异，以实现java在不同平台都能达到一致的内存访问效果（跨平台）

处理器上的寄存器的读写速度比内存快几个数量级，为了解决这种速度矛盾，在之间加了高速缓存，这时会带来另一个问题：缓存一致性，如果多个缓存共享同一块	主内存区域，那么多个缓存的数据可能会不一致，需要一个协议来解决这个问题。

所有变量都存储在主内存中，每个线程还有自己的工作内存，工作内存存储在高速缓存或寄存器中，保存了该线程使用的变量的主内存副本拷贝。

内存间交互操作：

- read：把一个变量从主内存拷贝到工作内存中
- load：在read之后，把read得到的值放入工作内存的变量副本中
- use：把工作内存的一个变量的值传递给执行引擎
- assign：把一个从执行引擎得到的值赋给工作内存的变量
- store：把工作内存的一个变量的值传递给主内存
- write：store之后，把store得到的值放入主内存的变量中
- lock：加锁，作用于主内存的变量
- unlock：释放锁

三大特性：

- 原子性
- 可见性：volatile，synchronized，final
- 有序性

## Java虚拟机

程序计数器

本地方法栈

虚拟机栈：栈帧（局部变量表、操作数栈、常量池引用）

堆

元空间（1.8）

### 垃圾收集器

Serial收集器：单线程，新生代（复制算法，STW），老年代（标记整理算法，STW）

ParNew收集器：多线程，新生代（复制算法，STW），老年代（标记整理算法，STW）

CMS收集器：初始标记（STW），并发标记，重新标记（STW），并发清除

G1收集器：初始标记，并发标记，最终标记，筛选回收

### 常用工具

1. jps：列出正在运行的虚拟机进程
2. jstat：虚拟机状态
   `jstat -gc 1505 250 20`
3. jinfo
4. jmap
5. jhat
6. jstack
7. hsdis

## 基础

Integer缓冲池的最大值可调：-XX:AutoBoxCacheMax

### String

value不可变，final

字符串常量池(String Pool)保存着所有字符串字面量(literal strings)，这些字面量在编译时期就确定。不仅如

此，还可以使用 String 的 intern() 方法在运行过程中将字符串添加到 String Pool 中。

## 反射

java.lang.reflect：Field,Method,Constructor

## 并发

Java内存模型规定，将所有的变量都存放在主内存中，当线程使用变量时，会把主内存里面的变量复制一份到工作空间或者叫工作内存，线程读写变量时操作的是自己工作内存里的变量。

### synchronized关键字

synchronized是一种原子性内置锁，同时java中的线程与操作系统的原生线程是一一对应的，所以当阻塞一个线程时，需要从用户态切换到内核态执行阻塞操作，很耗时，synchronized使用时会导致上下文切换。synchronized会把其块内使用的变量从工作内存中清除，直接从主内存中获取，同时退出时会把对共享变量的修改刷新到主内存中。

JDK1.6的优化：对锁的实现引入了大量的优化，如偏向锁、轻量级锁、自旋锁、适应性自旋锁、锁消除、锁粗化等技术来减 少锁操作的开销。

锁主要存在四种状态，依次是:无锁状态、偏向锁状态、轻量级锁状态、重量级锁状态，他们会随着竞争的激烈而逐渐升级。注意锁可以升级不可降级，这种策略是为了提高获得锁和释放锁的效率。

### volatile关键字

当一个变量被声明为volatile时，线程在写入变量时不会把值缓存到寄存器或其他地方，而是直接刷新到主内存中，当其他线程读取该变量时，不会从工作内存中读，而是直接从主内存读

写入volatile变量 相当于 线程退出synchronized块

读取volatile变量 相当于 线程进入synchronized块

volatile只保证可见性，无法保证原子性

使用场景：

- 写入变量不依赖变量当前值时
- 读写变量值时没有加锁

### CAS操作

Compare And Set

解决线程上下文切换和重新调度开销

Unsafe类：对于var1中偏移量为var2的变量，如果其值时var4，那么将其更新为var6

```java
// var1:对象内存位置，var2:变量的偏移量,var4：变量预期值，var6：变量新值
public final native boolean compareAndSwapLong(Object var1, long var2, long var4, long var6);
```

存在ABA问题：线程1尝试将X的值从A更新到B，但是线程2将X的值从A更新到B之后，又更新到A，此时的A已不是最开始的A，但是线程1无法感知。（原因：存在环形状态，解决方案：状态只能朝一个方向转换-AtomicStampedReference）

### Java指令重排序

多线程下会有问题，用volatile声明变量解决

写volatile变量时，不会把写之前的操作重排到写之后；读volatile变量时，不会把读之后的操作重排到读之前

### 伪共享

CPU与主内存之间存在一级或者多级高速缓冲存储器（Cache），集成到CPU内部

Cache内部按行存储，每一行叫做Cache行，大小一般为2的指数次字节

存放到Cache行的是内存块而不是单个变量，所以一行会有很多变量（目标变量地址相近的变量会一同加载到一行）

问题：多个线程操作一行内的多个变量时，会导致其他线程的缓存失效，频繁访问主内存

解决方案：利用冗余变量填充一行

```java
//jdk7以上使用此方法(jdk7的某个版本oracle对伪共享做了优化)
public final static class NewVolatileLong
{
  public volatile long value = 0L;
  public long p1, p2, p3, p4, p5, p6;
}

// jdk7以下使用此方法
public final static class OldVolatileLong
{
  public long p1, p2, p3, p4, p5, p6, p7; // cache line padding
  public volatile long value = 0L;
  public long p8, p9, p10, p11, p12, p13, p14; // cache line padding
}
// jdk8,需要在jvm启动时设置-XX:-RestrictContended才会生效。
@sun.misc.Contended
public final static class VolatileLong {
    public volatile long value = 0L;
    //public long p1, p2, p3, p4, p5, p6;
}
```

### 可重入锁

自己可以再次获取自己的内部锁。比如一个线程获得了某个对象的锁，此时 这个对象锁还没有释放，当其再次想要获取这个对象的锁的时候还是可以获取的，如果不可锁重入的话，就会造成死 锁。同一个线程每次获取锁，锁的计数器都自增1，所以要等到锁的计数器下降为0时才能释放锁。

### ReentrantLock

可重入锁，Lock的实现类

```java
// 非公平锁
public ReentrantLock() {
  sync = new NonfairSync();
}
// 公平锁
public ReentrantLock(boolean fair) {
  sync = fair ? new FairSync() : new NonfairSync();
}

// 获取锁
final boolean nonfairTryAcquire(int acquires) {
  final Thread current = Thread.currentThread();
  int c = getState();
  // 如果锁没被其他线程占用
  if (c == 0) {
    if (compareAndSetState(0, acquires)) {
      setExclusiveOwnerThread(current);
      return true;
    }
  }
  else if (current == getExclusiveOwnerThread()) {
    // 重入
    int nextc = c + acquires;
    if (nextc < 0) // overflow
      throw new Error("Maximum lock count exceeded");
    setState(nextc);
    return true;
  }
  return false;
}

// 释放锁
protected final boolean tryRelease(int releases) {
  int c = getState() - releases;
  if (Thread.currentThread() != getExclusiveOwnerThread())
    throw new IllegalMonitorStateException();
  boolean free = false;
  if (c == 0) {
    free = true;
    setExclusiveOwnerThread(null);
  }
  // 锁未被全部释放
  setState(c);
  return free;
}
```

### 锁优化

自旋锁：互斥同步进入阻塞状态的开销都很大，应该尽量避免。在许多应用中，共享数据的锁定状态只会持续很短的一段时

间。自旋锁的思想是让一个线程在请求一个共享数据的锁时执行忙循环(自旋)一段时间，如果在这段时间内能获得

锁，就可以避免进入阻塞状态。

锁消除：锁消除是指对于被检测出不可能存在竞争的共享数据的锁进行消除。（逃逸分析）

锁粗化：如果一系列的连续操作都对同一个对象反复加锁和解锁，频繁的加锁操作就会导致性能损耗。

轻量级锁

偏向锁：偏向锁的思想是偏向于让第一个获取锁对象的线程，这个线程在之后获取该锁就不再需要进行同步操作，甚至连 CAS 操作也不再需要。

## 集合 

### HashMap

#### put流程

1. 若table未初始化，则初始化
2. 若定位到的槽`i=(n - 1) & hash`为空，则直接写入
3. 若与当前key相同，则直接覆盖旧值
   若当前节点为红黑树，则调用putTreeVal设值
   若当前节点为链表，遍历是否存在要插入的key，遍历到尾部后，决定是否要转为红黑树（长度大于8）
4. 如果达到阈值（threshold），则进行扩容

#### 扩容流程

1. 设置新容量与新阈值
   - 若原来容量大于0，并且达到MAXIMUM_CAPACITY（1 << 30），则设值阈值为Integer.MAX_VALUE，并取消扩容；否则新容量为原来的2倍
   - 若原来的threshold大于0，则新容量直接设置为threshold
   - 否则新容量设置为DEFAULT_INITIAL_CAPACITY（16），新threshold设置为DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY
2. 创建新数组
   - 遍历旧数组，将当前node赋给e，清空当前节点，利用4个指针完成扩容（无需重新计算hash，利用e.hash & oldCap是否为0来判断新数组的下标）：loHead存储为0的头节点，loTail不断延伸为0的链表；hiHead存储为1的头节点，hiTail不断延伸为1的链表
   - 最后loHead放入newTab[j]，hiHead放入newTab[j + oldCap]

Hash：(key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16)  ，  (n - 1) & hash

```java
int threshold;             // 所能容纳的key-value对极限 
final float loadFactor;    // 负载因子
int modCount;  
int size;  
```



解决哈希表的冲突-开放地址法和链地址法

并发问题---resize时会出现环形链表，1.8后大大降低了概率

### ConcurrentHashMap

#### put流程

1. 判空
2. 计算hash，调用spread计算`hash=(h ^(h >>>16))& HASH_BITS`；
3. 遍历table：
   - 若table为空，则初始化（判断sizeCtl小于0，代表其他线程正在操作，让出CPU，否则CAS更新sizectl为-1）
   - 若定位（volatile）到的槽`i=(n - 1) & hash`为空，则CAS写入数据
   - 若当前槽hash为-1，就是为ForwardingNode，代表其他线程正在扩容，帮助扩容
   - 否则出现哈希碰撞，利用synchronized同步寻找value，分为链表和红黑树两种情况
   - 若链表长度为8，则treeifyBin转树（Note：若length<64,直接tryPresize,两倍table.length;不转树）
4. addCount增加数量

#### addCount：

1. CAS更新baseCount
2. 检验是否需要扩容，若sizeCtl小于0，代表正在扩容，则sizeCtl+1同时辅助扩容transfer(tab, nt)，否则开始扩容transfer(tab, null)，完毕后计算数量

```java
// 存储节点数据，扩容时总是2的指数次方
transient volatile Node<K,V>[] table;
// 扩容时新生成的数组，是table大小的两倍
private transient volatile Node<K,V>[] nextTable;
// 默认为0，-1代表table正在初始化，-N代表N-1个线程正在执行扩容操作
// 其余情况：1.如果table未初始化，表示table需要初始化的大小 2.如果table初始化完成，代表table容量，默认为table大小的0.75倍，
private transient volatile int sizeCtl;
static class Node<K,V> implements Map.Entry<K,V> {
  final int hash;
  final K key;
  volatile V val;
  volatile Node<K,V> next;
}
// 
static final class ForwardingNode<K,V> extends Node<K,V> {
  final Node<K,V>[] nextTable;
  ForwardingNode(Node<K,V>[] tab) {
    super(MOVED, null, null, null);
    this.nextTable = tab;
  }
}

// 此方法会将大小构造成2的指数次方
private static final int tableSizeFor(int c) {
  int n = c - 1;
  n |= n >>> 1;
  n |= n >>> 2;
  n |= n >>> 4;
  n |= n >>> 8;
  n |= n >>> 16;
  return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}

// table初始化操作会延迟到第一次put，此方法保证并发时不会初始化多次
private final Node<K,V>[] initTable() {
  Node<K,V>[] tab; int sc;
  while ((tab = table) == null || tab.length == 0) {
    // 代表其他线程正在操作，让出CPU
    if ((sc = sizeCtl) < 0)
      Thread.yield(); // lost initialization race; just spin
    // CAS修改sc
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

final V putVal(K key, V value, boolean onlyIfAbsent) {
  if (key == null || value == null) throw new NullPointerException();
  int hash = spread(key.hashCode());
  int binCount = 0;
  for (Node<K,V>[] tab = table;;) {
    Node<K,V> f; int n, i, fh;
    if (tab == null || (n = tab.length) == 0)
      tab = initTable();
    else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
      if (casTabAt(tab, i, null,
                   new Node<K,V>(hash, key, value, null)))
        break;                   // no lock when adding to empty bin
    }
    else if ((fh = f.hash) == MOVED)
      tab = helpTransfer(tab, f);
    else {
      V oldVal = null;
      synchronized (f) {
        if (tabAt(tab, i) == f) {
          if (fh >= 0) {
            binCount = 1;
            for (Node<K,V> e = f;; ++binCount) {
              K ek;
              if (e.hash == hash &&
                  ((ek = e.key) == key ||
                   (ek != null && key.equals(ek)))) {
                oldVal = e.val;
                if (!onlyIfAbsent)
                  e.val = value;
                break;
              }
              Node<K,V> pred = e;
              if ((e = e.next) == null) {
                pred.next = new Node<K,V>(hash, key,
                                          value, null);
                break;
              }
            }
          }
          else if (f instanceof TreeBin) {
            Node<K,V> p;
            binCount = 2;
            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                  value)) != null) {
              oldVal = p.val;
              if (!onlyIfAbsent)
                p.val = value;
            }
          }
        }
      }
      if (binCount != 0) {
        if (binCount >= TREEIFY_THRESHOLD)
          treeifyBin(tab, i);
        if (oldVal != null)
          return oldVal;
        break;
      }
    }
  }
  addCount(1L, binCount);
  return null;
}
static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
  return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
}
static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                    Node<K,V> c, Node<K,V> v) {
  return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
}
```

## 线程池

Executors：工厂类

ThreadPoolExecutor：

- corePoolSize：核心线程数
- maximumPoolSize：最大线程数
- keepAliveTime：线程空闲时的存活时间
- unit：keepAliveTime单位
- workQueue：保存等待被执行任务的阻塞队列：
  1. ArrayBlockingQueue：基于数组的有界阻塞队列，FIFO
  2. LinkedBlockingQueue：基于链表的阻塞队列，FIFO，吞吐量高于ArrayBlockingQueue
  3. SynchornousQueue：不存储元素的阻塞队列，每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量高于LinkedBlockingQueue
  4. PriorityBlockingQueue：具有优先级的无界阻塞队列

### 四种线程池

**newFixedThreadPool**：corePoolSize=maximumPoolSize，利用LinkedBlockingQueue做阻塞队列，当线程没有可执行任务时，也不会释放线程

**newCachedThreadPool**：存活时间60s，最大线程数可达Integer.MAX_VALUE，使用SynchornousQueue作为阻塞队列，注意控制并发数

**newSingleThreadExecutor**：初始化只有一个线程，如果该线程异常结束后，会重新创建一个新线程继续执行任务，可以保证任务的顺序执行，使用LinkedBlockingQueue作为阻塞队列

**newScheduledThreadPool**：在指定的时间内周期性的执行所提交的任务

除newScheduledThreadPool，其他都基于ThreadPoolExecutor实现

### 线程状态

```java
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
private static final int COUNT_BITS = Integer.SIZE - 3;
private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

// runState is stored in the high-order bits
private static final int RUNNING    = -1 << COUNT_BITS;
private static final int SHUTDOWN   =  0 << COUNT_BITS;
private static final int STOP       =  1 << COUNT_BITS;
private static final int TIDYING    =  2 << COUNT_BITS;
private static final int TERMINATED =  3 << COUNT_BITS;
```

## Spring

### IOC

控制反转，一种设计思想，是将对象创建及生命周期的管理移交给Spring容器来做。