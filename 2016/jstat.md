# Options #
jstat 命令支持两种类型的选项，常规选项和输出选项。常规选项使jstat命令显示简单的用法和版本信息。输出选项决定统计输出的内容与格式。
所有选项及其功能可能会在将来的版本中更改或删除。
## 常规选项 ##
如果指定其中一个常规选项，则不能指定任何其他选项或参数。
### -help ###
显示帮助信息
### -options ###
显示一系列静态选项。详细请见“输出选项”
## 输出选项
如果没有指定常规选项，则可以指定输出选项。输出选项确定jstat命令输出的内容和格式，由单个statOption和任何其他输出选项（-h，-t和-J）组成。statOption必须先开始。
输出格式为表格，列以空格分隔。带标题行描述列的作用。使用-h选项设置标题显示的频率。在不同选项中列名称的代表的含义是一致的。一般来说，如果两个选项提供具有相同名称的列，则两列的数据源是相同的。
使用 -t 选项可以显示时间戳列，输出的第一列为时间戳。时间戳列包含自目标 JV M启动以来经过的时间（以秒为单位）。时间戳的准确度依赖于多种因素并且在高负载系统中受制于线程延时调度而容易变动。
使用时间间隔和次数参数分别确定 jstat 命令显示输出的频率与次数。
注意：不要编写脚本来解析jstat命令的输出，因为格式可能会在将来的版本中更改。如果你已经编写了用于解析 jstat 命令的输出格式，那么在这个工具的未来版本，你需要修改脚本。
### -statOption ###
确定jstat命令显示的统计信息。以下列出可用的选项。使用“常规选项”可显示特定平台安装的选项列表。
**class**: 显示有关类加载器行为的统计信息。
**compiler**: 显示有关Java HotSpot VM即时编译器的行为的统计信息。
**gc**: 显示垃圾回收行为的统计信息。
**gccapacity**: 显示各个代的容量以及对应的可用空间。
**gccause**: 显示关于垃圾回收统计（与 -gcutil 功能相同），以及上次垃圾回收的原因。
**gcnew**: 显示新生代行为的统计信息。
**gcnewcapacity**: 显示新生代容量大小以及对应的剩余空间。
**gcold**: 显示年老代和元空间行为的统计信息。
**gcoldcapacity**: 显示年老代大小的统计数据。
**gcmetacapacity**: 显示元空间大小的统计数据。
**gcutil**: 显示一个关于垃圾收集的统计汇总。
**printcompilation**: 显示的Java HotSpot虚拟机编译方法统计。
### -h n ###
每 n 行显示一次标题行。n 是一个正整数。缺省值为 0，表示只在数据的第一行显示。
### -t ###
输出的第一列为时间戳。时间戳是该 JVM 启动开始计时。
### -JjavaOption ###
为应用传递启动参数。例如， -J-Xms48m 设置启动的内存为 48 MB。
## Stat Options and Output ##
以下总结了 jstat 命令每个选项输出列的含义。
###-class option ###
类加载统计。
**Loaded**: 加载类的数量
**Bytes**: 加载的类大小，以 KB 为单位
**Unloaded**: 卸载的类数量
**Bytes**: 卸载的类大小，以 KB 为单位
**Time**: 执行类加载和卸载操作所花费的时间
### -compiler option ###
Java HotSpot VM 即时编译统计
**Compiled**: 执行的编译任务数
**Failed**: 编译失败任务数
**Invalid**: 已失效的编译任务数
**Time**: 执行编译任务所花费的时间
**FailedType**: 上次编译失败的编译类型
**FailedMethod**: 上次失败编译的类名和方法
### -gc option ###
Garbage-collected heap statistics.
**S0C**: survivor 空间 0 当前容量（KB）
**S1C**: survivor 空间 1 当前容量（KB）
**S0U**: survivoe 空间 0 使用量（KB）
**S1U**: Survivor 空间 1 使用量（kB）
**EC**: 当前 eden 空间容量（KB）
**EU**: Eden 空间使用量（KB）
**OC**: 当前年老大容量（KB）
**OU**: 年老代空间使用量
**MC**: 元空间容量（KB）
**MU**: 元空间使用量（KB）
**CCSC**: 压缩类空间容量（KB）
**CCSU**: 压缩类空间使用量（KB）
**YGC**: 年轻代垃圾回收次数
**YGCT**: 年轻代垃圾回收总时间
**FGC**: 完全垃圾回收次数
**FGCT**: 完全垃圾回收时间
**GCT**: 总垃圾回收时间
### -gccapacity option ###
内存生产与空间容量
**NGCMN**: 年轻代最小容量（KB）
**NGCMX**: 年轻代最大容量（KB）
**NGC**: 当前年轻代容量（KB）
**S0C**: 当前 survivor 空间 0 容量（KB）
**S1C**: 当前 survivor 空间 1 容量（KB）
**EC**: 当前 eden 使用量（KB）
**OGCMN**: 最小年老代容量（KB）
**OGCMX**: 最大年老代容量（KB）
**OGC**: 当前年老代容量（KB）
**OC**: 当前年老代使用量（KB）
**MCMN**: 最小元空间容量（KB）
**MCMX**: 最大元空间容量（KB）
**MC**: 元空间容量（KB）
**CCSMN**: 压缩类空间最小容量（KB）
**CCSMX**: 压缩类空间最大容量（KB）
**CCSC**: 压缩类空间使用量（KB）
**YGC**: 年轻代 gc 次数（KB）
**FGC**: full gc 次数（KB）
### -gccause option ###
这个和 -gcutil 选项一样，显示垃圾回收统计的统计，但是包括最近一次垃圾回收的原因和（如果适用）当前垃圾收集事件。除了为-gcutil列出的列之外，此选项还添加以下列。
**LGCC**: 上次垃圾回收的原因
**GCC**: 当前垃圾回收原因
### -gcnew option ###
新生代统计
**S0C**: 当前 survivor 空间 0 容量（KB）
**S1C**: 当前 sruvivor 空间 1 容量（KB）
**S0U**: Survivor 空间 0 使用量（KB）
**S1U**: SSurvivor 空间 1 使用量（KB）
**TT**: 存活期限
**MTT**: 最大存活期限
**DSS**: 希望 survivor 大小（KB） 
**EC**: 当前 eden 空间使用量（KB）
**EU**: Eden 空间使用量（KB）
**YGC**: 年轻代 GC 次数
**YGCT**: 年轻代 GC 总时间
### -gcnewcapacity option ###
新生代空间大小统计
**NGCMN**: 最小新生代容量（KB）
**NGCMX**: 最大新生代容量（KB）
**NGC**: 当前年轻代容量（KB）
**S0CMX**: 最大 survivor 空间 0 容量（KB）
**S0C**: 当前 survivor 空间 0 容量（KB）
**S1CMX**: 最大 survivor 空间 1 容量（KB）
**S1C**: 当前 survivor 空间 1 容量（KB）
**ECMX**: 最大 eden 空间容量（KB）
**EC**: 当前 eden 空间容量（KB）
**YGC**: 年轻代 GC 次数（KB）
**FGC**: full GC 次数（KB）
### -gcold option ###
年老代和元空间统计
**MC**: 元空间容量（KB）
**MU**: 元空间使用量（KB）
**CCSC**: 压缩类空间容量（KB）
**CCSU**: 压缩类空间使用量（KB）
**OC**: 当前年老代容量（KB）
**OU**: 年老代使用量（KB）
**YGC**: 年轻代 GC 次数（KB）
**FGC**: full GC 次数（KB）
**FGCT**: full GC 总时间（KB）
**GCT**: 总垃圾回收时间（KB）
### -gcoldcapacity option ###
年老大容量统计
**OGCMN**: 最小年老大容量（KB）
**OGCMX**: 最大年老大容量（KB）
**OGC**: 当前年老代容量（KB）
**OC**: 当前年老代空间容量大小（KB）
**YGC**: 年轻代 GC 次数
**FGC**: full gc 次数
**FGCT**: full GC 总时间
**GCT**: 总垃圾回收时间
### -gcmetacapacity option ###
元空间大小统计
**MCMN**: 最小元空间容量（KB）
**MCMX**: 最大元空间容量（KB）
**MC**: 元空间容量（KB）
**CCSMN**: 压缩类空间最小容量（KB）
**CCSMX**: 压缩类空间最大容量（KB）
**YGC**: 年轻代 GC 次数
**FGC**: full gc 次数
**FGCT**: full GC 总时间
**GCT**: 总垃圾回收时间
### -gcutil option ###
垃圾回收统计
**S0**: survivor 空间 0 使用量与该空间当前容量的百分比
**S1**: survivor 空间 1 使用量与改空间当前容量的百分比
**E**: Eden 空间使用量与该空间当前容量的百分比
**O**: 年老代使用量与改空间当前容量的百分比
**M**: 元空间使用量与该空间当前容量的百分比
**CCS**: 压缩类空间使用量百分比
**YGC**: 年轻代 GC 次数
**YGCT**: 年轻代垃圾回收总时间
**FGC**: full gc 次数
**FGCT**: full GC 总时间
**GCT**: 总垃圾回收时间
### -printcompilation option ###
Java HotSpot VM 编译方法统计
Compiled: Number of compilation tasks performed by the most recently compiled method.
Size: Number of bytes of byte code of the most recently compiled method.
Type: Compilation type of the most recently compiled method.
Method: Class name and method name identifying the most recently compiled method. Class name uses slash (/) instead of dot (.) as a name space separator. Method name is the method within the specified class. The format for these two fields is consistent with the HotSpot -XX:+PrintCompilation option.
Examples
This section presents some examples of monitoring a local JVM with an lvmid of 21891.
The gcutil Option
This example attaches to lvmid 21891 and takes 7 samples at 250 millisecond intervals and displays the output as specified by the -gcutil option.
The output of this example shows that a young generation collection occurred between the third and fourth sample. The collection took 0.078 seconds and promoted objects from the eden space (E) to the old space (O), resulting in an increase of old space utilization from 66.80% to 68.19%. Before the collection, the survivor space was 97.02% utilized, but after this collection it is 91.03% utilized.
jstat -gcutil 21891 250 7
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT   
  0.00  97.02  70.31  66.80  95.52  89.14      7    0.300     0    0.000    0.300
  0.00  97.02  86.23  66.80  95.52  89.14      7    0.300     0    0.000    0.300
  0.00  97.02  96.53  66.80  95.52  89.14      7    0.300     0    0.000    0.300
 91.03   0.00   1.98  68.19  95.89  91.24      8    0.378     0    0.000    0.378
 91.03   0.00  15.82  68.19  95.89  91.24      8    0.378     0    0.000    0.378
 91.03   0.00  17.80  68.19  95.89  91.24      8    0.378     0    0.000    0.378
 91.03   0.00  17.80  68.19  95.89  91.24      8    0.378     0    0.000    0.378
Repeat the Column Header String
This example attaches to lvmid 21891 and takes samples at 250 millisecond intervals and displays the output as specified by -gcnew option. In addition, it uses the -h3 option to output the column header after every 3 lines of data.
In addition to showing the repeating header string, this example shows that between the second and third samples, a young GC occurred. Its duration was 0.001 seconds. The collection found enough active data that the survivor space 0 utilization (S0U) would have exceeded the desired survivor Size (DSS). As a result, objects were promoted to the old generation (not visible in this output), and the tenuring threshold (TT) was lowered from 31 to 2.
Another collection occurs between the fifth and sixth samples. This collection found very few survivors and returned the tenuring threshold to 31.
jstat -gcnew -h3 21891 250
 S0C    S1C    S0U    S1U   TT MTT  DSS      EC       EU     YGC     YGCT
  64.0   64.0    0.0   31.7 31  31   32.0    512.0    178.6    249    0.203
  64.0   64.0    0.0   31.7 31  31   32.0    512.0    355.5    249    0.203
  64.0   64.0   35.4    0.0  2  31   32.0    512.0     21.9    250    0.204
 S0C    S1C    S0U    S1U   TT MTT  DSS      EC       EU     YGC     YGCT
  64.0   64.0   35.4    0.0  2  31   32.0    512.0    245.9    250    0.204
  64.0   64.0   35.4    0.0  2  31   32.0    512.0    421.1    250    0.204
  64.0   64.0    0.0   19.0 31  31   32.0    512.0     84.4    251    0.204
 S0C    S1C    S0U    S1U   TT MTT  DSS      EC       EU     YGC     YGCT
  64.0   64.0    0.0   19.0 31  31   32.0    512.0    306.7    251    0.204
Include a Time Stamp for Each Sample
This example attaches to lvmid 21891 and takes 3 samples at 250 millisecond intervals. The -t option is used to generate a time stamp for each sample in the first column.
The Timestamp column reports the elapsed time in seconds since the start of the target JVM. In addition, the -gcoldcapacity output shows the old generation capacity (OGC) and the old space capacity (OC) increasing as the heap expands to meet allocation or promotion demands. The old generation capacity (OGC) has grown from 11,696 kB to 13,820 kB after the eighty-first full garbage collection (FGC). The maximum capacity of the generation (and space) is 60,544 kB (OGCMX), so it still has room to expand.
Timestamp      OGCMN    OGCMX     OGC       OC       YGC   FGC    FGCT    GCT
          150.1   1408.0  60544.0  11696.0  11696.0   194    80    2.874   3.799
          150.4   1408.0  60544.0  13820.0  13820.0   194    81    2.938   3.863
          150.7   1408.0  60544.0  13820.0  13820.0   194    81    2.938   3.863
Monitor Instrumentation for a Remote JVM
This example attaches to lvmid 40496 on the system named remote.domain using the -gcutil option, with samples taken every second indefinitely.
The lvmid is combined with the name of the remote host to construct a vmid of 40496@remote.domain. This vmid results in the use of the rmi protocol to communicate to the default jstatd server on the remote host. The jstatd server is located using the rmiregistry command on remote.domain that is bound to the default port of the rmiregistry command (port 1099).
jstat -gcutil 40496@remote.domain 1000
... output omitted

