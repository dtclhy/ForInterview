

# Put源码

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,boolean evict) {     
  Node<K,V>[] tab; Node<K,V> p; int n, i;    
  //初始化      
  if ((tab = table) == null || (n = tab.length) == 0)       
    n = (tab = resize()).length; 
  //判断该位置是否为空
  if ((p = tab[i = (n - 1) & hash]) == null)        
    tab[i] = newNode(hash, key, value, null);     
  else {  
    //e为临时节点，判断是否覆盖
    Node<K,V> e; K k; 
    //覆盖原来节点
    if (p.hash == hash &&((k = p.key) == key || (key != null && key.equals(k))))         
      e = p;  
    //树节点
    else if (p instanceof TreeNode)          
      e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);        
    else {  
      //链表情况
      for (int binCount = 0; ; ++binCount) {             
        if ((e = p.next) == null) {                    
          p.next = newNode(hash, key, value, null); 
           //触发树化操作
          if (binCount >= TREEIFY_THRESHOLD - 1) //                          
            treeifyBin(tab, hash);                  
          break;                
        }                    
        if (e.hash == hash &&((k = e.key) == key || (key != null && key.equals(k))))                        
          break;                    
        p = e;                
      }            
    }           
    if (e != null) {                
      V oldValue = e.value;               
      if (!onlyIfAbsent || oldValue == null)                   
        e.value = value;               
      afterNodeAccess(e);              
      return oldValue;           
    }        
  }        
  ++modCount;   
  //判断是否扩容
  //hashmap中的元素个数超过数组大小*loadFactor
  if (++size > threshold)           
    resize();        
  afterNodeInsertion(evict);      
  return null;   
}
```

## hash计算

hash = key.hashCode()的高16位和低16位异或的方式

hash值尽可能分散

数组的位置：(n-1) & hash

数组的大小必须是2的n次幂，保证最后为1111，与操作计算时才取决于hash值

# resize扩容

原位置只有一个元素，直接hash之后放置

原位置是红黑树，split(tree)，会判断是否小于6，转链表

原位置是链表，则通过判断`e.hash & oldCap) == 0`,==0则放入原位置，!=0则移动到    <原位置+原容量>

容量扩大，左移一位

# remove()方法

原位置是红黑树时，会根据下面条件动态判断，节点数小于6可能转化。

```java
root == null|| (movable&& (root.right == null|| (rl = root.left) == null
            || rl.left == null)
```

# Get()方法

get(Object key)流程，通过传入的key通过hash()算法得到hash值，在通过(n - 1) & hash找到数组下标，如果数组下标所对应的node值正好key一样就返回，否则找到node.next找到下一个节点，看是否是treenNode，如果是，遍历红黑树找到对应node，如果不是遍历链表找到node。