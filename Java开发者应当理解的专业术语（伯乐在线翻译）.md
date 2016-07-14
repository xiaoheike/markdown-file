# JVM Buzzwords Java developers should understand #
This article will share with you a few JVM “buzzwords” that are important for Java developers to understand and remember before performing any JVM performance and garbage collection tuning. A few tips are also provided including some high level performance tuning best practices at the end of the article. Further recommendations regarding the Oracle HotSpot concurrent GC collectors such as CMS and G1 will be explored in future articles.
Before reading any further, I recommend that you first get familiar with the [JVM verbose GC logs](http://javaeesupportpatterns.blogspot.com/2011/10/verbosegc-output-tutorial-java-7.html). Acquiring this JVM data analysis skill is essential, especially when combined with more advanced APM technologies.
# Java开发者应当理解的专业术语 #
本文将和你分享一些 `JVM` “专业术语”，程序员在对 `JVM` 做任何的性能和垃圾回收调整之前理解和记住这些“专业术语”是非常重要的。在文章的最后还提供一些小技巧，包括一些高性能微调的最佳实践。关于 `Oracle HotSpot` 的并发垃圾回收器，例如 `CMS` 和 `G1` 的进一步建议将在后续文章中探讨。
在深入阅读之前，我建议你先熟悉《[JVM 详细 GC 输出日志](http://javaeesupportpatterns.blogspot.com/2011/10/verbosegc-output-tutorial-java-7.html)》。获得这个 `JVM` 数据分析技能是至关重要的，特别是当与更加复杂的 `APM` 技术结合使用的时候。

## JVM Buzzwords ##
- Allocation Rate	Java objects allocated to the YoungGen space, a.k.a. “short-lived’ objects.
- Promotion Rate	Java objects promoted from the YoungGen to the OldGen space.
- LIVE Data	        Java objects sitting in the OldGen space, a.k.a. “long-lived’ objects.
- Stop-the-world Collection	   Garbage collections such as Full GC and causing a temporary suspension of your application threads until completed.

## JVM 专业术语 ##
- 分配率（Allocation Rate）    Java 对象被分配到年轻代内存空间，又名 “短暂”对象
- 转换率（Promotion Rate）     Java 对象从年轻代转移到年老代
- 存活数据（LIVE Data）        被分配在年老代内存空间的 Java 对象，又名 “长存”对象
- 暂停应用程序运行进行垃圾回收（Stop-the-world Collection）     垃圾回收，例如完全垃圾回收，造成你的应用程序线程短暂性挂起直到垃圾回收完成

## First Things First: JVM GC Logs ##
- Provides out-of-the-box fine-grained details on the Java heap and GC activity.
- Use tools such as [GCMV](http://javaeesupportpatterns.blogspot.com/2013/06/java-gc-tool-from-ibm-tutorial-on-jcg.html) (GC Memory Visualizer) in order to assess your JVM pause time and memory allocation rate vs. sizing the generations by hand.

## 第一件事情：JVM 垃圾回收日志 ##
- 提供有关 Java 堆栈和垃圾回收活动黑盒内的详细细节
- 使用工具，例如 《[GCMV](http://javaeesupportpatterns.blogspot.com/2013/06/java-gc-tool-from-ibm-tutorial-on-jcg.html)（垃圾回收内存可视化）》，用于评估 `JVM` 暂停时间和内存分配率，以及可以手工调整代内存大小（译者注：这里的“代”指 `JVM` 中的年轻代，年老代以及永久代）。

![JVM_buzzwords_YG](http://jbcdn2.b0.upaiyun.com/2015/09/9d57cec162befcfc75ffc58d11920e44.png)

## Allocation & Promotion Rates ##
- It is important to keep track of your application allocation and promotion rates for optimal GC performance.
- Keep the GCAdaptiveSizePolicy active, as part of the JVM ergonomics. Tune by hand only if required.
## 分配和转换率 ##
- 跟踪程序的分配率和转换率对于垃圾回收性能优化是非常重要的。
- `GCAdaptiveSizePolicy` 参数是 `JVM` 不可分割的一部分，保持该参数有效。仅在必要时手动调整。

![JVM_YG_allocation_rate](http://jbcdn2.b0.upaiyun.com/2015/09/1eac821e1d2133cec9ae4f626ad435a7.png)


![JVM_YG_allocation_rate](http://jbcdn2.b0.upaiyun.com/2015/09/fe190b1ec4505dde9d9d3fdcb423922e.png)
## LIVE Data Calculation ##
- Your live application data corresponds to the OldGen occupancy after a Full GC.
- It is essential that your OldGen capacity is big enough to hold your live data comfortably and to limit the frequency of major collections and impact on your application load throughput.
**Recommendation**: as a starting point, tune your Java Heap size in order to achieve an OldGen footprint or occupancy after Full GC of about 50%, allowing a sufficient buffer for certain higher load scenarios (fail-over, spikes, busy business periods…).
- ***Hot Spot****:watch for OldGen memory leaks!
- What is a memory leak in Java? Constant increase of the LIVE data over time…
## 存活数据计算##
- 一次完全垃圾回收之后应用程序依旧存活的数据被分配到年老代。
- 你的年老代容量要足够大是至关重要的，为了能够轻易容纳你的应用程序存活数据，也为了能够限制应用程序主要垃圾回收的频率，还为了能够减轻对应用程序负载吞吐量的影响。

**建议**：刚开始，调整 Java 堆大小，以达到一次完全垃圾回收之后年老代占用或者大约占用 50% 的内存，使得高负载情况（故障转移，高峰，业务繁忙时段）能够提供做够大的缓存。
- **要点**：注意年老代的内存溢出！
- 在 Java 中什么是内存溢出？随着时间的推移存活数据不断增加……

![JVM_LIVE_data](http://jbcdn2.b0.upaiyun.com/2015/09/b967e23052ec2085e86f5ec700e81ead.png)

## LIVE Data Deep Dive ##
- JVM GC logs are great…but how you can inspect your live data?
- Java Heap Histogram snapshots and heap dump analysis are powerful and proven approaches to better understand your application live data.
- Java profiler solutions and tools such as [Oracle Java Mission Control](http://www.oracle.com/technetwork/java/javaseproducts/mission-control/java-mission-control-1998576.html), Java Visual VM provide advanced features for deep Java heap inspection and profiling, including tracking of your application memory allocations.
## 深入存活数据 ##
- `JVM` 的垃圾回收日志很棒……但是你如何检查你的存活数据呢？
- `Java` 堆直方图快照和堆转储分析是强大而成熟的方法，能够更好地了解您的应用程序的存活数据。
- `Java` 剖析解决方案和工具，例如 [Oracle Java Mission Control](http://www.oracle.com/technetwork/java/javaseproducts/mission-control/java-mission-control-1998576.html)，Java Visual VM 为 Java 堆的检查和分析提供了高级特性，包括应用程序内存分配的跟踪。

![Java_Visual_VM_histogram](http://jbcdn2.b0.upaiyun.com/2015/09/59f15856b2eb14814cd5bdd6fec2a6cf.png)

## Stop-the-world Collections: GC Overhead ##
- YoungGen collections are less expensive but be careful with excessive allocation rate.
- It is recommended to initially size (JVM default) the YoungGen at 1/3 of the heap size.
- Remember: both YoungGen and OldGen collections are stop-the-world events!
- PermGen and Metaspace (JDK 1.8+) are collected during a Full GC, thus it is important to keep track of the Class meta data footprint and GC frequency. 
## Stop-the-world Collections: GC Overhead ##
- 年轻代垃圾回收性能消耗低，但是要注意不要超过分配率
- 建议初始分配年轻代大小（`JVM` 默认大小）为对大小的 1/3  
- 记住：年轻代和年老代垃圾回收两者都需要停止程序运行的时间。
- 永久代和元空间（JDK 1.8+）在完全垃圾回收时进行，因此跟踪类元数据和垃圾回收的频率是非常重要的。

![JVM_GC_Overhead](http://jbcdn2.b0.upaiyun.com/2015/09/ff14a75d301d6503106aea11b69d6f4e.png)
![JVM_YG_overhead](http://jbcdn2.b0.upaiyun.com/2015/09/37583150107f38244085c38958ffe2c2.png)


## Final Words & Recommendations ##
**Best Practices**
- Optimal Java Performance is not just about Java…explore all angles.
- Always rely on facts instead of guesswork.
- Focus on global tuning items first vs. premature fine-grained optimizations.
- Perform Performance & Load Testing when applicable.
- Take advantage of proven tools and troubleshooting techniques available.

**To Avoid**
- There are dozens of possible JVM parameters: don’t over-tune your JVM!
- You always fear what you don’t understand: good application knowledge > no fear  > better tuning recommendations.
- Never assume that your application performance is optimal.
- Don’t try to fix all problems at once, implement tuning incrementally.
- Don’t get confused and keep focus on the root cause of performance problems as opposed to the symptoms.
- Excessive trial and error approach: symptom of guesswork.

## 最后的话语和建议 ##
**最佳实践**
- 优化 `Java` 性能不只是关于 `Java`……探索各个角度。
- 始终依靠事实，而不是猜测。
- 首要集中注意力在全局微调而不是过早的细粒度优化。
- 适当时进行性能测试和负载测试。
- 利用可用的可靠工具和故障排除技巧。

**避免**
- 可能有几十个 `JVM` 参数，不要过度调整你的 `JVM`！
- 你总是怕你不明白的东西：良好的应用知识 > 无惧 > 更好的优化建议。
- 不要假设你的应用程序的性能是最佳的。
- 不要试图一次性解决所有问题，逐步实现调整。
- 不要混淆并专注于导致性能问题的根源而不是症状。
- 过多试错法：猜测症状。

Reference:	[JVM Buzzwords Java developers should understand](http://javaeesupportpatterns.blogspot.tw/2015/07/jvm-buzzwords-java-developers-should.html) from our JCG partner Pierre Hugues Charbonneau at the Java EE Support Patterns blog.

参考：JCG 伙伴 Pierre Hugues Charbonneau 发表在 **Java EE Support Patterns** 的博客 [JVM Buzzwords Java developers should understand](http://javaeesupportpatterns.blogspot.tw/2015/07/jvm-buzzwords-java-developers-should.html)。 


