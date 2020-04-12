## 系统底层如何实现数据一致性

1. MESI

2. 锁总线

## 系统底层如何保证有序性

1. 内存屏障sfence、mfense、lfense等系统原语

2. 锁总线

## CAS

​		CAS算法由底层通过汇编指令“lock cmpxchg”来进行操作，“cmpxchg”本身不是原子操作，但是通过“lock”锁定一个北桥信息就可以实现原子操作（不是锁总线的方式）。

## Synchronized

实现原理：

1. java代码层面直接使用synchronized
2. 字节码使用moniterEnter和monitorExit实现
3. 执行过程中进行锁的升级（偏向锁 -> 轻量级锁 -> 重量级锁）
4. 系统底层通过指令“lock cmpxchg”实现

## Volatile

实现原理：

1. 源码层面：volatile

2. 字节码层面：ACC_VOLATILE

3. JVM的内存屏障

   屏障两边的指令不可以重排，保证有序！

4. hotspot实现

   使用“lock addl”指令