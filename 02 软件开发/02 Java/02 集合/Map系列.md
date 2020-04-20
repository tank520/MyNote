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

## LinkedHashMap

​		

## ConcurrentHashMap