# 计算机原理

## CPU

* 单核

* 多核CPU

  提高并行处理能力

* 多CPU

  分布式计算

* 单核多线程

  并发

* 多核多线程

  并行

## MESI Cache一致性协议

Cache Line（缓存行）四种状态

1. Modified（修改的）
2. Exclusive（独占的）
3. Shared（共享的）
4. Invalid（失效的）

缓存锁实现之一，有些无法被缓存或跨多个缓存行的数据依然必须使用总线锁。