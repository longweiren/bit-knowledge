###jstat###

*jstat 主要用来监控JVM进程的资源和性能。本文用的是`Java HotSpot(TM) Client VM`1.8*

> jstat option vmid [interval] [count] 
> interval jstat采集数据间隔时间
> count    jstat打印的记录行数
> -h[n]    间隔多少行打印一次header
> -t       打印采集的时间
> 使用`jps`可以查看JVM进程的vmid

```
AnaTool.java
public class AnaTool {
    public static void main(String[] args) {
        // 让程序一直运行，便于我们分析进程
        while(true) {
            System.out.println(1);
            try {
                Thread.currentThread().sleep(1000);
            } catch (Exception e) {

            }
        }
    }
}

// 编译后运行 java -cp . AnaTool
```
使用 `jps`查看JVM进程id
```
E:\>jps
372 Jps
3916 AnaTool
```

**可选的 options**
> class              统计分析 class loader的行为
> compiler           统计分析 jit 编译器的行为
> gc                 统计分析堆GC行为
> gccapacity         统计分析分代容量及相关的空间
> gccause            GC统计摘要(与gcutil相似)
> gcnew              统计分析新生代的行为
> gcnewcapacity      统计分析新生代相关空间的大小
> gcold              统计分析老生代 ~~(和持久带的行为)~~
> gcoldcapacity      统计分析老生代的大小
> ~~gcpermcapacity     统计分析持久代的大小~~
> gcmetacapacity     统计分析metaspace大小
> gcutil             GC统计摘要
> printcompilation   打印HotSpot汇编方法统计信息


**jstat -class pid**
---
> 查看进程加载的 class 的数量、占用空间及时间信息

```
E:\>jstat -class 3916
Loaded  Bytes  Unloaded  Bytes     Time
   415  443.6        0     0.0     0.09

// 加载了415个class(包括我们自己编写的和直接引用到的Class),在permgen空间占用 443 bytes。

Loaded   加载的 class 数量
Bytes    加载的 class 大小
Unloaded 卸载的 class 数量
Bytes    卸载的 class 大小
Time     加载卸载 class 所用的时间
```

**jstat -compiler pid**
---
> 查看JVM实时编译的 class 的数量、占用空间及时间信息

```
E:\>jstat -compiler 3916
Compiled Failed Invalid   Time   FailedType FailedMethod
       8      0       0   0.00          0

// JVM实例在运行过程中会对静态编译的方法或类进行优化(如指令重排等)。上述指令就是查看实时优化的情况。

Compiled     执行实时编译的任务数量
Failed       实时编译失败的数量
Invalid      无效的实时编译任务数量
Time         实时编译任务执行所耗时间
FailedType   最近一次编译失败的编译类型
FailedMethod 最近一次编译失败的类名和方法
```

**jstat -gc pid**
---
```
E:\>jstat -gc 3916
 S0C    S1C    S0U  S1U   EC      EU     OC       OU   MC     MU
 512.0  512.0  0.0  0.0   4416.0  852.7  10944.0  0.0  2240.0 492.9

 CCSC   CCSU   YGC  YGCT    FGC   FGCT   GCT
 0.0    0.0    0    0.000   0     0.000  0.000
```

> S0C  S0区当前容量(KB)
> S1C  S1区当前容量
> S0U  S0区已使用的容量
> S1U  S1区已使用的容量
> EC   Eden区当前容量
> EU   Eden区已使用量
> OC   Old区当前容量
> OU   Eden区已使用量
> PC   permanent 空间当前容量
> PU   permanent 空间已使用容量
> YGC  Young区 GC 次数
> YGCT Young区 GC 使用的时间
> FGC  Full GC 次数
> FGCT Full GC 使用的时间
> GCT  总的 GC 耗时

**jstat -gccapacity pid**
---

```
E:\>jstat -gccapacity 3916
 NGCMN   NGCMX    NGC     S0C    S1C    EC      OGCMN    OGCMX     OGC
 5440.0  87360.0  5440.0  512.0  512.0  4416.0  10944.0  174784.0  10944.0

 OC       MCMN  MCMX    MC     CCSMN   CCSMX   CCSC   YGC  FGC
 10944.0  0.0   4480.0  240.0  0.0     0.0     0.0    0    0
```

> 查看堆空间情况
> NGCMN  最小 new generation 容量
> NGCMX  最大 new generation 容量
> NGC    当前 new generation 容量
> OGCMN  最小 old generation 容量
> OGCMX  最大 old generation 容量
> OGC    当前 old generation 容量
> MCMN   最小 metaspace 容量
> MCMX   最大 metaspace 容量
> MC     metaspace 容量
> CCSMN  最小的压缩后 class space 容量
> CCSMX  最大的压缩后 class space 容量
> CCSC   压缩后 class space 容量(compressed class space capacity)

**jstat -gcnew pid**
---

```
E:\>jstat -gcnew 3916
 S0C    S1C    S0U  S1U  TT  MTT  DSS   EC       EU     YGC  YGCT
 512.0  512.0  0.0  0.0  15  15   0.0   4416.0   852.7  0    0.000
```

> Young = Eden + S0 + S1
> 
> 查看new对象的信息。
> TT   对象存活次数(Tenuring Threshold)。当Young对象年龄超过TT时会移到Old区
> MTT  最大存活次数限制(Max Tenuring Threshold)
> DSS  期望的存活区大小(Desired survivor size)


**jstat -gcnewcapacity pid**
---

```
E:\>jstat -gcnewcapacity 3916
 NGCMN    NGCMX    NGC     S0CMX   S0C    S1CMX   S1C    ECMX    EC     
 5440.0   87360.0  5440.0  8704.0  512.0  8704.0  512.0  69952.0 4416.0

 YGC   FGC
   0     0
```

> 查看new对象的信息及堆空间情况
> S0CMX  最大的 S0 区容量
> S1CMX  最大的 S1 区容量
> ECMX   最大的 Eden 区容量

**jstat -gcold pid**
---

```
E:\>jstat -gcold 3916
 MC      MU     CCSC   CCSU   OC       OU    YGC  FGC  FGCT    GCT
 2240.0  492.9  0.0    0.0    10944.0  0.0   0    0    0.000   0.000
```

> 查看old对象的信息
> MU    metaspace 已使用容量
> CCSU  已使用的压缩后 class space 容量

**jstat -gcoldcapacity pid**
---

```
E:\>jstat -gcoldcapacity 3916
 OGCMN     OGCMX      OGC       OC       YGC   FGC  FGCT    GCT
 10944.0   174784.0   10944.0   10944.0  0     0    0.000   0.000
```

> 查看old对象的信息及堆空间情况

**jstat -util pid**
---

```
E:\>jstat -gcutil 3916
 S0     S1    E      O     M      CCS  YGC  YGCT   FGC  FGCT   GCT
 0.00   0.00  19.31  0.00  22.00  -    0    0.000  0    0.000  0.000
```

> E   eden  space 使用率(百分比表示)
> O   old   space 使用率(百分比表示)
> M   meta  space 使用率(百分比表示)
> CCS class space 使用率(百分比表示)

**jstat -gcmetacapacity pid**
---

```
E:\workspace\gitbook\java-code>jstat -gcmetacapacity 3916
 MCMN  MCMX    MC      CCSMN  CCSMX  CCSC  YGC  FGC  FGCT   GCT
 0.0   4480.0  2240.0  0.0    0.0    0.0   0    0    0.000  0.000
```

> 统计分析metaspace大小




>
> S0C    S0区当前容量(KB)
> S1C    S1区当前容量
> S0U    S0区已使用的容量
> S1U    S1区已使用的容量
> S0CMX  最大的 S0 区容量
> S1CMX  最大的 S1 区容量
> 
> E      Eden区使用率(百分比表示)
> EC     Eden区当前容量
> EU     Eden区已使用量
> ECMX   最大的 Eden 区容量
> 
> TT     对象存活次数(Tenuring Threshold)。
> MTT    最大存活次数限制(Max Tenuring Threshold)
> DSS    期望的存活区大小(Desired survivor size)
> 
> NGC    当前 new generation 容量
> NGCMN  最小 new generation 容量
> NGCMX  最大 new generation 容量
> 
> O      Old区使用率(百分比表示)
> OC     Old区当前容量
> OU     Old区已使用量
> OGC    当前 old generation 容量
> OGCMN  最小 old generation 容量
> OGCMX  最大 old generation 容量
> 
> PC     permanent 空间当前容量
> PU     permanent 空间已使用容量
> 
> M      meta space 使用率(百分比表示)
> MC     meta space 容量
> MU     meta space 已使用容量
> MCMN   最小 meta space 容量
> MCMX   最大 metaspace 容量
> 
> CCS    class space 使用率(百分比表示)
> CCSC   压缩后 class space 容量(compressed class space capacity)
> CCSU   已使用的压缩后 class space 容量
> CCSMN  最小的压缩后 class space 容量
> CCSMX  最大的压缩后 class space 容量
> 
> YGC    Young区 GC 次数
> YGCT   Young区 GC 使用的时间
> FGC    Full GC 次数
> FGCT   Full GC 使用的时间
> GCT    总的 GC 耗时
