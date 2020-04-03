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

## 调优，从规划开始

* 步骤

  1. 熟悉业务场景（没有最好的垃圾回收器，只有最合适的垃圾回收器）

     1. 响应时间、停顿时间【CMS G1 ZGC】（需要给用户作响应）
     2. 吞吐量= 用户时间 / (用户时间 + GC时间)【PS】

  2. 选择回收器组合

     * -XX:+UseSerialGC = Serial New (DefNew) + Serial Old

       小型程序。默认情况下不会是这种选项，HotSpot会根据计算及配置和JDK版本自动选择收集器

     * -XX:+UseParNewGC = ParNew + SerialOld

       这个组合已经很少用（在某些版本中已经废弃）

     * -XX:+UseConcMarkSweepGC = ParNew + CMS + Serial Old

     * -XX:+UseParallelGC = Parallel Scavenge + Parallel Old (1.8默认) 【PS + SerialOld】

     * -XX:+UseParallelOldGC = Parallel Scavenge + Parallel Old

     * -XX:+UseG1GC = G1

  3. 计算内存需求

  4. 选定CPU（越高越好）

  5. 设定年代大小、升级年龄

  6. 设定日志参数

     ```java
     -Xloggc:/opt/xxx/logs/xxx-xxx-gc-%t.log -XX:UseGcLogFileRotation -XX:NumberOfGcLogFiles=5 -XX:GCLogFileSize=20M -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCCause
     ```

  7. 观察日志情况

## 优化环境

1. 有一个50万PV的资料类网站（从磁盘提取文档到内存）原服务器32位，1.5G
   的堆，用户反馈网站比较缓慢，因此公司决定升级，新的服务器为64位，16G
   的堆内存，结果用户反馈卡顿十分严重，反而比以前效率更低了
   1. 为什么原网站慢?
      很多用户浏览数据，很多数据load到内存，内存不足，频繁GC，STW长，响应时间变慢
   2. 为什么会更卡顿？
      内存越大，FGC时间越长
   3. 咋办？
      PS -> PN + CMS 或者 G1
2. 系统CPU经常100%，如何调优？(面试高频)
   CPU100%那么一定有线程在占用系统资源，找出哪个进程cpu高（top）
   该进程中的哪个线程cpu高（top -Hp）
   导出该线程的堆栈 (jstack)
   查找哪个方法（栈帧）消耗时间 (jstack)
   工作线程占比高 | 垃圾回收线程占比高
3. 系统内存飙高，如何查找问题？（面试高频）
   导出堆内存 (jmap)
   分析 (jhat jvisualvm mat jprofiler ... )
4. 如何监控JVM
   jstat jvisualvm jprofiler arthas top...

## GC常用参数

* -Xmn -Xms -Xmx -Xss
  年轻代 最小堆 最大堆 栈空间
* -XX:+UseTLAB
  使用TLAB，默认打开
* -XX:+PrintTLAB
  打印TLAB的使用情况
* -XX:TLABSize
  设置TLAB大小
* -XX:+DisableExplictGC
  System.gc()不管用 ，FGC
* -XX:+PrintGC
* -XX:+PrintGCDetails
* -XX:+PrintHeapAtGC
* -XX:+PrintGCTimeStamps
* -XX:+PrintGCApplicationConcurrentTime (低)
  打印应用程序时间
* -XX:+PrintGCApplicationStoppedTime （低）
  打印暂停时长
* -XX:+PrintReferenceGC （重要性低）
  记录回收了多少种不同引用类型的引用
* -verbose:class
  类加载详细过程
* -XX:+PrintVMOptions
* -XX:+PrintFlagsFinal -XX:+PrintFlagsInitial
  必须会用
* -Xloggc:opt/log/gc.log
* -XX:MaxTenuringThreshold
  升代年龄，最大值15
* 锁自旋次数 -XX:PreBlockSpin 热点代码检测参数-XX:CompileThreshold 逃逸分析 标量替换 ...
  这些不建议设置

## Parallel常用参数

- -XX:SurvivorRatio
- -XX:PreTenureSizeThreshold
  大对象到底多大
- -XX:MaxTenuringThreshold
- -XX:+ParallelGCThreads
  并行收集器的线程数，同样适用于CMS，一般设为和CPU核数相同
- -XX:+UseAdaptiveSizePolicy
  自动选择各区大小比例

## CMS常用参数

- -XX:+UseConcMarkSweepGC
- -XX:ParallelCMSThreads
  CMS线程数量
- -XX:CMSInitiatingOccupancyFraction
  使用多少比例的老年代后开始CMS收集，默认是68%(近似值)，如果频繁发生SerialOld卡顿，应该调小，（频繁CMS回收）
- -XX:+UseCMSCompactAtFullCollection
  在FGC时进行压缩
- -XX:CMSFullGCsBeforeCompaction
  多少次FGC之后进行压缩
- -XX:+CMSClassUnloadingEnabled
- -XX:CMSInitiatingPermOccupancyFraction
  达到什么比例时进行Perm回收
- GCTimeRatio
  设置GC时间占用程序运行时间的百分比
- -XX:MaxGCPauseMillis
  停顿时间，是一个建议时间，GC会尝试用各种手段达到这个时间，比如减小年轻代
  G1常用参数
- -XX:+UseG1GC
- -XX:MaxGCPauseMillis
  建议值，G1会尝试调整Young区的块数来达到这个值
- -XX:GCPauseIntervalMillis
  GC的间隔时间
- -XX:+G1HeapRegionSize
  分区大小，建议逐渐增大该值，1 2 4 8 16 32。
  随着size增加，垃圾的存活时间更长，GC间隔更长，但每次GC的时间也会更长
  ZGC做了改进（动态区块大小）
- G1NewSizePercent
  新生代最小比例，默认为5%
- G1MaxNewSizePercent
  新生代最大比例，默认为60%
- GCTimeRatio
  GC时间建议比例，G1会根据这个值调整堆空间
- ConcGCThreads
  线程数量
- InitiatingHeapOccupancyPercent
  启动G1的堆空间占用比例