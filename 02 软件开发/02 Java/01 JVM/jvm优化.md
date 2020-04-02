# JVM调优

## 调优基础概念

>1. 吞吐量：用户代码时间 / (用户代码时间 + 垃圾回收时间)
>2. 响应时间：STW越短，响应时间约好

所谓调优，首先确定，追求啥？吞吐量优先，还是响应时间优先？还是在满足一定的响应时间的情况下，要求达到多大的吞吐量。。。

## 什么是调优

1. 根据需求进行JVM规划和预调优
2. 优化运行JVM运行环境（慢、卡顿）
3. 解决JVM运行过程中出现的各种问题（OOM）

## JVM常用命令行参数

* JVM命令行参考：https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html

* HotSpot参数分类

  >标准：-开头，所有的HotSpot都支持
  >
  >非标准：-X开头，特定版本HotSpot支持特定命令
  >
  >不稳定：-XX开头，下个版本可能取消

* -XX相关参数查看
  1. -XX:+PrintCommandLineFlags：打印启动时真正使用的命令行参数
  2. -XX:+PrintFlagsFinal 最终参数值，可以查看java所有参数
  3. -XX:+PrintFlagsInitial 默认参数值
  4. java -XX:+PrintFlagsFinal | grep xxx 找到对应的参数
  5. java -XX:+PrintFlagsFinal -version | grep GC
  6. java -XX:+PrintFlagsFinal -version | wc -l