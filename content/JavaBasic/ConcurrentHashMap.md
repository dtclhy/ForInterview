# 1.7和1.8区别

在 JDK1.7 的实现上，ConrruentHashMap 由一个个 Segment 组成，简单来说， ConcurrentHashMap 是一个 Segment 数组，它通过继承 ReentrantLock 来进行加锁，通过每次锁住一个 segment 来保证每个 segment 内的操作的线程安全性从而实现全局线程安全。

在 JDK1.7 的实现上

1. 取消了 segment 分段设计，直接使用 Node 数组来保存数据，并且采用 Node 数组元素作为锁来实现每一行数据进行加锁来进一步减少并发冲突的概率
2. 将原本数组+单向链表的数据结构变更为了数组+单向链表+红黑树的结构。采用单向列表方式，那么查询某个节点的时间复杂度就变为 O(n); 因此对于 队列长度超过 8 的列表，JDK1.8 采用了红黑树的结构，那么查询的时间复杂度就会降低到 O(logN),可以提升查找的性能。

# Put源码

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {        
  if (key == null || value == null) 
    throw new NullPointerException();  
    //计算hash值       
  int hash = spread(key.hashCode());       
  int binCount = 0;  //用来记录链表的长度      
  for (Node<K,V>[] tab = table;;) {         
    Node<K,V> f; int n, i, fh;           
    if (tab == null || (n = tab.length) == 0)              
      tab = initTable(); //如果数组为空，则进行数组初始化           
    else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {                
      if (casTabAt(tab, i, null,new Node<K,V>(hash, key, value, null)))                    
         //cas 将新的值封装成 node 插入
        break;                               
    }            
    //
    else if ((fh = f.hash) == MOVED)                
      tab = helpTransfer(tab, f);            
    else {               
      V oldVal = null;              
      synchronized (f) {                
        if (tabAt(tab, i) == f) {                        
          ////头结点的hash值大于0，说明是链表
          if (fh >= 0) {                      
            binCount = 1;                        
            for (Node<K,V> e = f;; ++binCount) {                          
              K ek;                               
              if (e.hash == hash &&((ek = e.key) == key ||(ek != null && key.equals(ek)))) {                           
                oldVal = e.val;                           
                if (!onlyIfAbsent)                               
                  e.val = value;                          
                break;                            
              }
              Node<K,V> pred = e;
              //一直遍历到链表的最末端，直接把新的值加入到链表的最后面
              if ((e = e.next) == null) {                    
                pred.next = new Node<K,V>(hash, key,value, null);                         
                break;                         
              }                       
            }                
          }   
          //如果当前的 f 节点是一颗红黑树
          else if (f instanceof TreeBin) {                  
            Node<K,V> p;                   
            binCount = 2;                        
            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,value)) != null) {                        
              oldVal = p.val;                              
              if (!onlyIfAbsent)                                    
                p.val = value;                         
            }                   
          }                
        }              
      }             
      if (binCount != 0) {
        ////如果链表长度已经达到临界值 8 
        ////tab 的长度是不是小于 64， 如果是，则执行扩容
        //否则，将当前链表转化为红黑树结构存储
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
```

## 数组初始化

```java
private final Node<K,V>[] initTable() {     
  Node<K,V>[] tab; int sc;     
  while ((tab = table) == null || tab.length == 0) {          
    if ((sc = sizeCtl) < 0)              
      Thread.yield(); // 被其他线程抢占了初始化的操作,则直接让出自己的 CPU        
    else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) { 
      //通过 cas 操作，将 sizeCtl 替换为-1，标识当前线程抢占到了初始化资格
      try {            
        if ((tab = table) == null || tab.length == 0) {              
          int n = (sc > 0) ? sc : DEFAULT_CAPACITY;            
          @SuppressWarnings("unchecked")             
          Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];  
          //将这个数组赋值给table
          table = tab = nt; 
          //计算下次扩容的大小
          sc = n - (n >>> 2);           
        }         
      } finally {
        //设置sizeCtl为sc
        sizeCtl = sc;      
      }    
      break;      
    }   
  }   
  return tab;   
}
```

sizeCtl 这个标志是在 Node 数组初始化或者扩容的时候的一个控制位标识，负数代表正在进行初化或者扩容操作。

-1 代表正在初始化。-N 代表有 N-1个线程正在进行扩容操作

0 标识 Node 数组还没有被初始化，正数代表初始化或者下一次扩容的大小

## addCount

 addCount 来增加 ConcurrentHashMap 中的元素个数， 并且还会可能触发扩容操作。

ConcurrentHashMap 是采用 CounterCell 数组来记录元素个数的。

首先判断 counterCells 是否为空，如果为空，就通过 cas 操作尝试修改 baseCount 变量，对这个变量进行原子累加操作。在没有竞争的情况下，仍然采用 baseCount 来记录元素个数。如果 cas 失败说明存在竞争，这个时候不能再采用 baseCount 来累加，而是通过CounterCell 来记录。

CounterCell 数组的每个元素，都存储一个元素个数，而实际我们调用size 方法就是通过这个循环累加来得到的。

## transfer 扩容

  判断是否需要扩容，也就是当更新后的键值对总数 baseCount >= 阈值 sizeCtl 时，进行
rehash，这里面会有两个逻辑。
1. 如果当前正在处于扩容阶段，则当前线程会加入并且协助扩容
 2. 如果当前没有在扩容，则直接触发扩容操作

ConcurrentHashMap 并没有直接加锁，而是采用 CAS 实现无锁的并发同步策略，最精华 的部分是它可以利用多线程来进行协同扩容。它把 Node 数组当作多个线程之间共享的任务队列，然后通过维护一个指针来划 分每个线程锁负责的区间，每个线程通过区间逆向遍历来实现扩容，一个已经迁移完的 bucket 会被替换为一个 ForwardingNode 节点，标记当前 bucket 已经被其他线程迁移完了。

把 Node 数组进行拆分，让每个线程处理自己的区域，假设 table 数组总长度是 64，默认情况下，那么每个线程可以分到 16 个 bucket。 然后每个线程处理的范围，按照倒序来做迁移
 通过 for 自循环处理每个槽位中的链表元素，默认 advace 为真，通过 CAS 设置 transferIndex 属性值，并初始化 i 和 bound 值，i 指当前处理的槽位序号，bound 指需要处理的槽位边界， 先处理槽位 31 的节点; (bound,i) =(16,31) 从 31 的位置往前推动。

通过分配好迁移的区间之后，开始对数据进行迁移。对数组该节点位置加锁，开始处理数组该位置的迁移工作。ConcurrentHashMap 在做链表迁移时，会用高低位来实现。通过 fn&n 可以把这个链表中的元素分为两类，将低位的链表放在 i 位置也就是不动，将高位链表放在 i+n 位置。

# Get源码

```java
//会发现源码中没有一处加了锁
public V get(Object key) {
    Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
    int h = spread(key.hashCode()); //计算hash
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (e = tabAt(tab, (n - 1) & h)) != null) {//读取首节点的Node元素
        if ((eh = e.hash) == h) { //如果该节点就是首节点就返回
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                return e.val;
        }
        //hash值为负值表示正在扩容，这个时候查的是ForwardingNode的find方法来定位到nextTable来
        //eh=-1，说明该节点是一个ForwardingNode，正在迁移，此时调用ForwardingNode的find方法去nextTable里找。
        //eh=-2，说明该节点是一个TreeBin，此时调用TreeBin的find方法遍历红黑树，由于红黑树有可能正在旋转变色，所以find里会有读写锁。
        //eh>=0，说明该节点下挂的是一个链表，直接遍历该链表即可。
        else if (eh < 0)
            return (p = e.find(h, key)) != null ? p.val : null;
        while ((e = e.next) != null) {//既不是首节点也不是ForwardingNode，那就往下遍历
            if (e.hash == h &&
                ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }
    return null;
}
```

- get操作全程不需要加锁是因为Node的成员val是用volatile修饰的和数组用volatile修饰没有关系。
- 数组用volatile修饰主要是保证在数组扩容的时候保证可见性。

