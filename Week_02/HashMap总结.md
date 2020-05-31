HashMap 笔记


### HashMap 类结构
  
    package java.util;
    public class HashMap<K,V> extends AbstractMap<K,V>
      implements Map<K,V>, Cloneable, Serializable 
      

HashMap实现了Map结构，集成AbstractMap。不同于HashTable，它允许Key和Value为Null。
HashMap和性能相关的连个参数分别是:初始化容量(initial capacity)和负载因子(load factor)

        /**
         * The default initial capacity - MUST be a power of two.
         */
        static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; //默认为16
        
        /**
         * The load factor used when none specified in constructor.
         */
        static final float DEFAULT_LOAD_FACTOR = 0.75f;//默认为0.75
          
### 初始化

      //设定负载因子
      public HashMap() {
          this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
      }
      
      //设定初始化容量和负载因子
      public HashMap(int initialCapacity, float loadFactor) {
          if (initialCapacity < 0)
              throw new IllegalArgumentException("Illegal initial capacity: " +
                                                 initialCapacity);
          if (initialCapacity > MAXIMUM_CAPACITY)
              initialCapacity = MAXIMUM_CAPACITY;
          if (loadFactor <= 0 || Float.isNaN(loadFactor))
              throw new IllegalArgumentException("Illegal load factor: " +
                                                 loadFactor);
          this.loadFactor = loadFactor;
          this.threshold = tableSizeFor(initialCapacity);
      }
      
      /**
      * 手动设定大小,并设定初始化
      */
      public HashMap(int initialCapacity) {
          this(initialCapacity, DEFAULT_LOAD_FACTOR);
      }
  
      //在put方法中初始化
      public V put(K key, V value) {
          return putVal(hash(key), key, value, false, true);
      }
      
      final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                     boolean evict) {
          Node<K,V>[] tab; Node<K,V> p; int n, i;
          if ((tab = table) == null || (n = tab.length) == 0)
              n = (tab = resize()).length;  //当table没有初始化的时候,在resize方法中初始化table
          if ((p = tab[i = (n - 1) & hash]) == null)
              tab[i] = newNode(hash, key, value, null);
          ...省略
       }
       
       
       final Node<K,V>[] resize() {
           Node<K,V>[] oldTab = table;
           int oldCap = (oldTab == null) ? 0 : oldTab.length;
           int oldThr = threshold;
           int newCap, newThr = 0;
           if (oldCap > 0) {
               if (oldCap >= MAXIMUM_CAPACITY) {
                   threshold = Integer.MAX_VALUE;
                   return oldTab;
               }
               else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                        oldCap >= DEFAULT_INITIAL_CAPACITY)
                   newThr = oldThr << 1; // double threshold
           }
           else if (oldThr > 0) // initial capacity was placed in threshold
               newCap = oldThr;
           else {               // zero initial threshold signifies using defaults
               newCap = DEFAULT_INITIAL_CAPACITY;
               newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
           }
           if (newThr == 0) {
               float ft = (float)newCap * loadFactor;
               newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                         (int)ft : Integer.MAX_VALUE);
           }
           threshold = newThr; //设定扩容阈值
           @SuppressWarnings({"rawtypes","unchecked"})
               Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
           table = newTab;//初始化容量
           
           ...省略
       }
      
      
在通常直接使用`new HashMap<>();`的情况下，是在put方法调用时才开始初始化的。


### HashMap Null 安全

      //HashTable的put方法
      public synchronized V put(K key, V value) {
          // 首先校验value不能为null
          if (value == null) {
              throw new NullPointerException();
          }
  
          Entry<?,?> tab[] = table;
          int hash = key.hashCode();//直接调用key的hashCode方法
          。。。省略
      }
      
      
      //hashMap的put方法(key,value 都没有硬性要求)
      public V put(K key, V value) {
          return putVal(hash(key), key, value, false, true);
      }
      
      //如果key为null直接返回0的hash值
      static final int hash(Object key) {
          int h;
          return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
      }
      
### HashMap 存储结构
      
      //由node数组组成的table,hash就是来定位数组的node位置
      transient Node<K,V>[] table;
      
      //Node的链表行Node,当hash冲突为8以内时，使用链表结构
      static class Node<K,V> implements Map.Entry<K,V> {
          final int hash;
          final K key;
          V value;
          Node<K,V> next;
      }
      
      //Node红黑树结构,当hash冲突为8以上时，使用红黑树
      static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
          TreeNode<K,V> parent;  // red-black tree links
          TreeNode<K,V> left;
          TreeNode<K,V> right;
          TreeNode<K,V> prev;    // needed to unlink next upon deletion
          boolean red;
              
      }
      
HashMap的存储结构是Node数组组成的table，key的hash就是用来定位key在table的位置。jdk1.8之前hashmap采用数组加链表
的结构，在1.8之后当链表的长度达到了8以上时，替换为红黑树提高效率。
      
      
### HashMap put方法  

      public V put(K key, V value) {
          return putVal(hash(key), key, value, false, true);
      }
      
      //如果key为null直接返回0的hash值
      static final int hash(Object key) {
          int h;
          return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);//调用key的hashCode方法
      }
      
      final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                     boolean evict) {
          Node<K,V>[] tab; Node<K,V> p; int n, i;
          //如果table为null时进行初始化
          if ((tab = table) == null || (n = tab.length) == 0)
              n = (tab = resize()).length;
          if ((p = tab[i = (n - 1) & hash]) == null)  //如果key hash之后的数据位置为null，说明还没有值
              tab[i] = newNode(hash, key, value, null);//将put的值构建为Node
          else {
              Node<K,V> e; K k;
              //如果hash,key 都一样,就直接替换
              if (p.hash == hash &&
                  ((k = p.key) == key || (key != null && key.equals(k))))
                  e = p;
              //如果是红黑树,就调用红黑树的方法
              else if (p instanceof TreeNode)
                  e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
              //如果不是红黑树,那就是链表处理
              else {
                  for (int binCount = 0; ; ++binCount) {
                      if ((e = p.next) == null) {
                          //找到链表中的next节点，并赋值
                          p.next = newNode(hash, key, value, null);
                          //TREEIFY_THRESHOLD 为8 ,当超过了8就调用treeifyBin构建红黑树
                          if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                              treeifyBin(tab, hash);
                          break;
                      }
                      //如果找到了hash一样,key也一样的node，则break在外层处理
                      if (e.hash == hash &&
                          ((k = e.key) == key || (key != null && key.equals(k))))
                          break;
                      p = e;
                  }
              }
              //如果原始位置有值,就根据条件替换
              if (e != null) { // existing mapping for key
                  V oldValue = e.value;
                  if (!onlyIfAbsent || oldValue == null)
                      e.value = value;
                  afterNodeAccess(e);
                  return oldValue;
              }
          }
          ++modCount;
          if (++size > threshold)
              resize();
          afterNodeInsertion(evict);
          return null;
      }
      
### HashMap put方法  

      public V get(Object key) {
          Node<K,V> e;
          return (e = getNode(hash(key), key)) == null ? null : e.value;
      }
  
      final Node<K,V> getNode(int hash, Object key) {
          Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
          if ((tab = table) != null && (n = tab.length) > 0 &&
              (first = tab[(n - 1) & hash]) != null) {
              
              //如果通过hash找到了第一个节点的hash值和key相同,说明就是要找的值
              if (first.hash == hash && // always check first node
                  ((k = first.key) == key || (key != null && key.equals(k))))
                  return first;
                  
              //如果hash之后,查找的key和存储的key对不上
              if ((e = first.next) != null) {
                  //如果是红黑树就调用树结构查找
                  if (first instanceof TreeNode)
                      return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                      
                  //链表的话就调用next持续对比
                  do {
                      if (e.hash == hash &&
                          ((k = e.key) == key || (key != null && key.equals(k))))
                          return e;
                  } while ((e = e.next) != null);
              }
          }
          return null;
      }