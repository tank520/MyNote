# GC

![alt 垃圾收集器](https://github.com/tank520/MyNote/blob/master/02%20%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/02%20Java/01%20JVM/%E5%9E%83%E5%9C%BE%E6%94%B6%E9%9B%86%E5%99%A8.png?raw=true)

## 垃圾收集算法

### Mark-Sweep

标记清除算法，优点：快速；缺点：碎片化

### Copying

复制算法，优点：性能高；缺点：内存浪费

### Mark-Compact

标记整理算法，优点：不会产生内存碎片，适合老年代大对象存储；缺点：算法复杂，涉及对象的移动

### 三色标记

Incremental Update  --- CMS

黑色  ---自己标记完成，成员变量标记完成

灰色  ---自己标记完成，成员变量没有标记完成

白色  ---没有标记的

### SATB(Snapshot at the begining)

G1

### Colored Pointers

ZGC、Shenandoah

### 算法实现

#### 枚举根节点

使用OopMap存放对象之间的引用关系。缺点：所需空间大

#### 安全点

线程暂停下来的一个点。

抢先式中断和主动式中断

缺点：线程可能处于sleep，无法响应中断

#### 安全区域

安全点的扩展，某段区域引用不会发生变化。

### Minor GC与Major GC

#### 新生代 GC（Minor GC）

​		指发生在新生代的垃圾收集动作，因为 Java 对象大多都具备朝生夕灭的特性，所以 Minor GC 非常频繁，一般回收速度也比较快。

##### 	触发机制

			* 当年轻代满时就会触发Minor GC，这里的年轻代满指的是Eden代满，Survivor满不会引发GC。

#### 老年代 GC（Major GC）

​		指发生在老年代的 GC，出现了 Major GC，经常会伴随至少一次的 Minor GC（但非绝对的，在 ParallelScavenge 收集器的收集策略里 就有直接进行 Major GC 的策略选择过程） 。MajorGC 的速度一般会比 Minor GC 慢 10倍以上。

##### 	触发机制

		- 当年老代满时会引发Major GC。

#### Full GC

​		Major GC 是清理OldGen。 Full GC 是清理整个堆空间—包括年轻代和永久代。

​	触发机制

 * System.gc

 * 老年代空间不足

 * 方法区空间不足

 * promotion failed (年代晋升失败,比如eden区的存活对象晋升到S区放不下，又尝试直接晋升到Old区又放不下，那么Promotion Failed,会触发FullGC)

 * CMS的Concurrent-Mode-Failure 由于CMS回收过程中主要分为四步:
   1.CMS initial mark 2.CMS Concurrent mark 3.CMS remark 4.CMS Concurrent sweep。

   在2中gc线程与用户线程同时执行，那么用户线程依旧可能同时产生垃圾，
   如果这个垃圾较多无法放入预留的空间就会产生CMS-Mode-Failure，
   切换为SerialOld单线程做mark-sweep-compact。

* 新生代晋升的平均大小大于老年代的剩余空间 （为了避免新生代晋升到老年代失败）。

## 收集器

Serial  ---单线程，STW，复制算法，适用几十M内存

Serial Old  ---单线程，STW，标记-整理算法

ParNew  ---多线程版本的Serial，STW，适用几G内存

Parallel Scavenge  ---多线程并行，复制算法，可设置吞吐量，适用几十G内存

CMS  ---多线程并行，减少STW，标记-清除算法，适用几十G内存

G1 --- 多线程并行，适用上百G内存，物理上不分区，逻辑上分区，以Region为单位回收

>三色标记 + SATB

ZGC、Shenandoah --- 4T，物理、逻辑都不分代

>ColoredPointer(颜色指针、着色指针)

Epsilon --- 啥也不干（调试、确认不用GC参与）