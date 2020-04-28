# Map

![](https://github.com/tank520/MyNote/blob/master/02%20%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/02%20Java/02%20%E9%9B%86%E5%90%88/Map%E5%AE%B6%E6%97%8F%E7%B1%BB%E5%9B%BE.jpg?raw=true)

| 集合              | Key          | Value        | 有序 | 线程安全   |
| ----------------- | ------------ | ------------ | ---- | ---------- |
| HashMap           | 可以为null   | 可以为null   | 无序 | 线程不安全 |
| HashTable         | 不可以为null | 不可以为null | 无序 | 线程安全   |
| TreeMap           | 不可以为null | 可以为null   | 有序 | 线程不安全 |
| LinkedHashMap     | 可以为null   | 可以为null   | 有序 | 线程不安全 |
| ConcurrentHashMap | 不可以为null | 不可以为null | 无序 | 线程安全   |

## HashMap	

| 功能         | JDK 1.7                                               | JDK 1.8                               |
| ------------ | ----------------------------------------------------- | ------------------------------------- |
| Node插入方式 | 头插法<br/>（缺点：多线程环境下扩容时会导致循环死链） | 尾插法                                |
| 存储结构     | 数组 + 链表的形式                                     | 链表中Node个数达到8个时，转化为红黑树 |



## HashTable

​	方法都加了synchronized，因此是线程安全的。

## LinkedHashMap

​		自定了Entry，增加before、after指针，实现有序插入和访问。

​		特性

- removeEldestEntry

  删除策略，默认返回的false，如果固定大小，采用的是LRU算法

## TreeMap

​		使用SortedMap对key进行排序，通过红黑树算法实现。

## ConcurrentHashMap

### 1.7版本

​		使用Segment + 自定义HashEntry实现，通过Segment使用可重入锁来对主体进行分段锁，提高并发效率，由于HashEntry使用volatile修饰，所以get不需要额外的加锁，获取map的size时，有两种方式：

1. 不加锁而多次计算size，若前后两次结果一致则表示准确（最多3次）
2. 如果两次结果不同，就对所有的Segment加锁来计算size

### 1.8版本

​		使用数组 + 链表 + 红黑树实现，通过CAS和Synchronized实现线程安全。

​		为什么使用Synchronized代替ReentrantLock？

1. 1.8里面锁的是链表或红黑树的头节点而不是一整个链表，将锁细化了
2. Synchronized和ReentrantLock他们的开销差距是在释放锁时唤醒线程的数量，Synchronized是唤醒锁池里所有的线程+刚好来访问的线程，而ReentrantLock则是当前线程后进来的第一个线程 + 刚好来访问的线程
3. 如果是线程并发量不大的情况下，那么Synchronized因为自旋锁、偏向锁、轻量级锁的原因，不用将等待线程挂起，偏向锁甚至不用自旋，所以在这种情况下要比ReentrantLock高效