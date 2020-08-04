## 特性

### doublewrite

为了解决在数据库宕机时，InnoDB存储引擎可能正在写入某个页且只写了一部分，这种情况下会出现写失效的问题，尽管可以通过重做日志恢复，但如果页本身发生损坏，则无法进行重做。这时InnoDB引入两次写（doublewrite）来解决这个问题。

doublewrite由两部分组成：

1、doublewrite buffer

2、共享表空间的两个区（连续的128个页）

在刷新缓冲池中的脏页时，首先将页复制到doublewrite buffer中，然后通过doublewrite buffer分两次顺序地写入共享表空间中的物理磁盘上，然后再同步到数据文件中。这样如果发生意外，则可以从共享表空间当中恢复数据。

![img](https://static.dingtalk.com/media/lALPDe7sw0WVrR3NAhjNA5c_919_536.png_827x10000.jpg?bizType=report) 

## 存储结构

### 行溢出

InnoDB单行最大限制为65535字节，不包括TEXT、Blob。

InnoDB存储引擎页大小为16KB，即16384字节，如何存放65535字节呢？

一般情况下，InnoDB存储引擎的数据都是存放在页类型为B-tree node中，但是当发生行溢出时，数据存放在页类型为Uncompress BLOB页中。

原因：存储引擎表是索引组织的，即B+tree的结构，这样每个页中至少应该有两条行记录（否则失去B+tree的意义，变成链表了）。