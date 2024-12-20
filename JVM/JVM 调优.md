# JVM 调优

## jps

查看当前运行的 Java 进程

## jmap

### `jmap -heap <pid>`

打印堆内存的相关信息

```shell
Attaching to process ID 16325, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.412-b08

using thread-local object allocation.
Concurrent Mark-Sweep GC

Heap Configuration:
   MinHeapFreeRatio         = 40
   MaxHeapFreeRatio         = 70
   MaxHeapSize              = 268435456 (256.0MB)
   NewSize                  = 134217728 (128.0MB)
   MaxNewSize               = 134217728 (128.0MB)
   OldSize                  = 134217728 (128.0MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 67108864 (64.0MB)
   CompressedClassSpaceSize = 159383552 (152.0MB)
   MaxMetaspaceSize         = 167772160 (160.0MB)
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
New Generation (Eden + 1 Survivor Space):
   capacity = 120848384 (115.25MB)
   used     = 47024120 (44.84569549560547MB)
   free     = 73824264 (70.40430450439453MB)
   38.911666373627305% used
Eden Space:
   capacity = 107479040 (102.5MB)
   used     = 46783160 (44.61589813232422MB)
   free     = 60695880 (57.88410186767578MB)
   43.52770549495046% used
From Space:
   capacity = 13369344 (12.75MB)
   used     = 240960 (0.22979736328125MB)
   free     = 13128384 (12.52020263671875MB)
   1.8023322610294117% used
To Space:
   capacity = 13369344 (12.75MB)
   used     = 0 (0.0MB)
   free     = 13369344 (12.75MB)
   0.0% used
concurrent mark-sweep generation:
   capacity = 134217728 (128.0MB)
   used     = 22344024 (21.308921813964844MB)
   free     = 111873704 (106.69107818603516MB)
   16.647595167160034% used

12058 interned Strings occupying 1069392 bytes.
```

## jstat

## 参考

## 待解决

- [Java中9种常见的CMS GC问题分析与解决](https://tech.meituan.com/2020/11/12/java-9-cms-gc.html)
- [新一代垃圾回收器ZGC的探索与实践](https://tech.meituan.com/2020/08/06/new-zgc-practice-in-meituan.html)
- [Java Hotspot G1 GC的一些关键技术](https://tech.meituan.com/2016/09/23/g1.html)
- [JVM调优-调优原则和原理分析](https://juejin.cn/post/7126366964494106654)
- [JVM调优-GC基本原理和调优关键分析](https://juejin.cn/post/7126918474646945800)
- [cpu打满且频繁full GC，怎么解决？](https://juejin.cn/post/7208498140160573500)
- [系统运行缓慢，CPU 100%，以及Full GC次数过多问题的排查思路](https://juejin.cn/post/6844903944276148232#heading-0)
- [面渣逆袭：JVM经典五十问，这下面试稳了！](https://mp.weixin.qq.com/s/XYsEJyIo46jXhHE1sOR_0Q)
- [一次线上OOM问题分析](https://juejin.cn/post/7205141492264976445)
- [一次大量 JVM Native 内存泄露的排查分析（64M 问题）](https://juejin.cn/post/7078624931826794503)
- [What causes the JVM to do a major garbage collection?](https://stackoverflow.com/questions/22249869/what-causes-the-jvm-to-do-a-major-garbage-collection)
- [一次线上JVM调优实践，FullGC40次/天到10天一次的优化过程](https://heapdump.cn/article/1859160)
- [什么场景触发fullGC](https://www.cnblogs.com/use-D/p/18277269)
- [Java常用命令：jps、jstack、jmap、jstat（带有实例教程）](https://www.cnblogs.com/jiaodaoniujava/p/18044895)
- [linux分析工具之top命令详解](https://www.cnblogs.com/zsql/p/11637535.html)
- [【JVM】jmap命令详解----查看JVM内存使用详情](https://www.cnblogs.com/sxdcgaq8080/p/11089664.html)
- [JVM调优-常见的调优工具](https://juejin.cn/post/7127642745690128398)
- [JVM的监控工具之jstat](https://www.cnblogs.com/cheng21553516/p/11223557.html)
- [线上项目频繁Full GC问题排查解决](https://blog.csdn.net/qq_43295093/article/details/120324120)
- [Linux使用jstat命令查看jvm的GC情况](https://blog.csdn.net/zlzlei/article/details/46471627)
- [linux分析工具之top命令详解](https://www.cnblogs.com/zsql/p/11637535.html)
