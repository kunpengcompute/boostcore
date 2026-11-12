# 快速入门

本文档帮助用户快速了解KQMalloc V0.22预编译动态库的使用方法、环境变量和性能验证方法。

## 安装部署

获取和解压动态库的详细步骤请参见[安装指南](./installation_guide.md)。KQMalloc V0.22提供以下动态库。

| 库文件 | 说明 |
| ------- | ---- |
| libkqmalloc.ptag.so | 启用指针标记特性的发布版本 |
| libkqmalloc.ptag.debug.so | 启用指针标记、断言和double free()检查等功能的调试版本 |
| libkqmalloc.ptag.profiler.so | 启用指针标记和内存分析功能的分析版本 |
| libkqmalloc.so | 未启用指针标记特性的发布版本 |
| libkqmalloc.debug.so | 未启用指针标记特性，启用断言和double free()检查等功能的调试版本 |
| libkqmalloc.profiler.so | 未启用指针标记特性，启用内存分析功能的分析版本 |

所有动态库均为并行分配器。分配器启动时按单线程模式初始化，检测到第二个线程后自动切换为多线程模式；该模式不可通过环境变量显式指定。未启用指针标记特性的版本适用于内核不支持指针标记的系统，或自身使用指针标记的应用（如Go、Java）。

以下命令以HIP09处理器上启用指针标记特性的发布版本为例。

```shell
kqmalloc_root=/path/to/kqmalloc-vX.Y.Z
LD_PRELOAD="$kqmalloc_root/HIP09/lib/libkqmalloc.ptag.so" ./run_your_application
```

## 测试程序介绍

`malloc_benchmark`是交付包中的内存分配测试程序，可通过不同的线程协作模式、对象大小分布和内存访问方式，模拟内存分配与释放负载。程序输出运行时间、速度和内存占用，可配合`LD_PRELOAD`测试系统默认内存库或KQMalloc。

测试程序以复合对象为单位组织负载，每个复合对象包含7个独立分配的分量。`-objects`控制复合对象数量，`-size_min`和`-size_max`控制各分量的大小，单位为字节。一次完整测试由若干轮（epoch）组成，每轮包含若干阶段（age），每个阶段按指定迭代次数执行。

下表说明常用命令行参数及其默认值。默认值来自`malloc_benchmark.c`中的初始化设置；`-objects`和`-iterations`的默认值随`-mode`变化，显式指定对应参数时会覆盖默认值。说明列中的数值为取值示例，首次运行可使用后面的[最小用例](#最小用例)。

| 变量名称 | 说明 | 默认值 |
| ---- | ---- | ---- |
| -threads | 设置测试线程数。例如`320`表示使用320个线程 | 1 |
| -mode | 设置线程协作模式。<br>`0`：All-To-All模式，线程共享对象池，模拟跨线程分配与释放<br>`1`：生产者/消费者模式，偶数线程时由一半线程分配、一半线程释放；奇数线程时各线程均可分配和释放<br>`2`：独立线程模式，各线程仅操作自己的对象分区 | 0 |
| -touch | 设置分配后的内存访问方式。<br>`0`：不主动访问分配的内存<br>`1`：按4096字节步长访问内存<br>`2`：通过`memset()`填充内存<br>`3`：通过`calloc()`分配零初始化内存<br>`4`：按64字节步长访问内存<br>`5`：按65536字节步长访问内存<br>`6`：内存校验模式，分配后写入校验数据，释放前读取并检查 | 1 |
| -objects | 设置复合对象数量上限。例如`4194304`表示最多4194304个复合对象；独立线程模式下，数量应至少等于线程数 | `-mode 0`：1000000<br>`-mode 1`：10000<br>`-mode 2`：1000000 |
| -size_min | 设置分配大小下限，单位为字节。例如`8`表示最小8字节 | 8 |
| -size_max | 设置分配大小上限，单位为字节。例如`16777216`表示最大16MiB | 33554432（32MiB） |
| -epochs | 设置完整测试轮数。例如`1`表示执行1轮测试 | 1 |
| -ages | 设置每轮测试的阶段数。例如`100`表示每个epoch包含100个age | 64 |
| -iterations | 设置每个线程在每个阶段的迭代次数。例如`500000`表示每个线程在每个age执行500000次迭代；一次迭代涉及复合对象的7个分量 | `-mode 0`：250000<br>`-mode 1`：400000<br>`-mode 2`：750000 |
| -temperature | 控制对象大小级别的分布。例如`0.11`表示大小分布温度为0.11。值较小时偏向小对象，值较大时大对象比例增加；不表示CPU温度 | 0.09 |
| -profiler | 设置是否启用测试程序内置的分配性能统计。<br>`0`：关闭<br>`1`：开启，按大小级别输出分配与释放次数、耗时及延迟统计 | 0 |
| -trim | 设置调用`malloc_trim(0)`的阶段间隔。<br>`0`：不调用<br>`12`：在每个epoch的第12、24、36等age结束时调用；实际回收行为取决于所用内存库 | 0 |
| -purge | 设置每个阶段开始时释放已有对象的概率。<br>`0.00`：不额外释放<br>`0.42`：对已分配的复合对象按42%的概率释放，实际比例会有随机波动；取值范围为0至1，在生产者/消费者模式下不生效 | 0.00 |
| -aligned | 设置使用`aligned_alloc()`的分配大小阈值，单位为字节。<br>`0`：不使用<br>`65536`：大小不超过65536字节的分量使用`aligned_alloc()`，更大的分量使用`malloc()`；按大小级别的自然对齐规则确定对齐值，并非统一对齐到65536字节。`-touch 3`时该参数不生效 | 0 |
| -affinity | 设置测试程序是否主动绑定CPU。<br>`1`：为多线程测试的工作线程设置CPU亲和性<br>**`0`：不主动设置CPU亲和性，线程继承进程已有的亲和性限制，由操作系统调度** | 1（Linux） |
| -imbalance | 设置是否输出线程间不均衡的详细信息。<br>`0`：不输出各线程明细，多线程测试仍输出汇总<br>`1`：在多线程测试中输出各线程明细，帮助比较工作线程的运行时间 | 0 |

## 最小用例

交付包包含内存分配测试程序`HIP08/bin/malloc_benchmark`。以下以HIP08版本为例，使用单线程和较小的测试规模验证动态库加载及程序运行。请根据实际处理器型号选择对应目录；不支持指针标记的系统或应用应使用`libkqmalloc.so`。

1. 设置解压目录，并检查动态库是否能够加载。

    ```shell
    kqmalloc_root=/path/to/kqmalloc-vX.Y.Z
    LD_PRELOAD="$kqmalloc_root/HIP08/lib/libkqmalloc.ptag.so" \
        ldd "$kqmalloc_root/HIP08/bin/malloc_benchmark"
    ```

    输出应包含`HIP08/lib/libkqmalloc.ptag.so`的实际路径，且没有`not found`或预加载失败提示。

2. 预加载KQMalloc并运行测试程序。

    ```shell
    LD_PRELOAD="$kqmalloc_root/HIP08/lib/libkqmalloc.ptag.so" \
        "$kqmalloc_root/HIP08/bin/malloc_benchmark" \
        -threads 1 -mode 2 -objects 1000 \
        -size_min 8 -size_max 1024 \
        -epochs 1 -ages 4 -iterations 100000 -affinity 0
    echo $?
    ```

    本例使用1个线程、独立线程模式（`-mode 2`），最多1000个复合对象（每个包含7个分量），对象大小为8至1024字节。测试执行1个epoch，每个epoch包含4个age，每个age执行100000次迭代。`-affinity 0`表示测试程序不主动设置CPU亲和性。

3. 检查测试结果。

    以下为2026-10-12在openEuler 24.03 LTS SP3、4KB系统页环境下，使用已有HIP08测试程序及动态库（远端源码版本标识为0.21）得到的实测输出节选，省略了环境、参数和温度信息。

    ```output
        Sequence      Seconds        Speed      Virtual     Resident
               0        0.001        0.000        1.539        0.019
               1        0.001        0.000        2.039        0.021
               2        0.017       45.992        2.039        0.032
               3        0.032       47.129        2.039        0.032
               4        0.046       47.297        2.039        0.032
               5        0.061       47.188        2.039        0.032
               6        0.061        0.000        2.039        0.032

    Mode: 2, Time:  0.061 seconds, Average RSS:  0.032 GB, Max RSS:  0.032 GB. Average speed: 46.901, Max speed: 47.297.
    ```

    `echo $?`应输出`0`，表示程序正常退出。`Seconds`为累计运行时间；`Speed`为测试程序统计的每秒百万次对象分量迭代数；`Virtual`和`Resident`分别为虚拟内存和常驻内存，按1024³字节换算，输出标记为GB。末尾`Mode: 2`汇总本次测试的耗时、常驻内存和速度。

实际耗时、内存和速度会随环境变化，无需与示例数值完全一致。本例用于验证基本运行；评估加速效果请参见[验证加速效果](#验证加速效果)。

## 环境变量

如[表1 KQMalloc库环境变量说明](#KQMalloc库环境变量说明)所示，KQMalloc支持以下环境变量。表中按单线程模式和多线程模式区分的默认值由分配器根据运行状态自动选择。

**表1** KQMalloc库环境变量说明<a id="KQMalloc库环境变量说明"></a>

| 变量名称 | 说明 | 默认值 | 可选值 |
|---------|------|-------|-------|
| KQMALLOC_ALIGNMENT | 设置对象地址的对齐大小，单位为字节。无效值将被忽略 | 单线程模式为4；多线程模式为16 | 4、8、16、32、64；缓存行是128字节时还支持128 |
| KQMALLOC_C11_COMPLIANT_PROCESSES | 设置强制使用C11兼容的16字节指针对齐的进程列表，并影响KQMALLOC_ALIGNMENT的默认值 | java | 以":"分隔的进程名列表，如java8:java15 |
| KQMALLOC_SLAB_CACHE_RATIO | 设置分配器为提升性能可缓存的slab最大空间比例 | 20 | 0-100 |
| KQMALLOC_LOCAL_CACHE_RATIO | 设置每个大小级别的线程本地微小对象和小对象缓存的相对大小，可增大或减小本地缓存 | 100.0 | 0.0-1000.0 |
| KQMALLOC_LOCAL_CACHE_MIN | 设置每个大小级别的线程本地微小对象和小对象缓存的最小物理容量，在KQMALLOC_LOCAL_CACHE_RATIO之后生效 | 常规模式为2；超额订阅模式为0 | 0-1000 |
| KQMALLOC_LOCAL_CACHE_MAX | 设置每个大小级别的线程本地微小对象和小对象缓存的最大物理容量，在KQMALLOC_LOCAL_CACHE_RATIO之后生效 | 1000 | 0-10000 |
| KQMALLOC_LOCAL_CACHE_LOW | 设置常规对象线程本地缓存的动态最小容量。增大该值可能降低延迟，但会增加内存占用 | 0 | 0-1000 |
| KQMALLOC_ALIGNED_LOCAL_CACHE_LOW | 设置对齐对象线程本地缓存的动态最小容量。增大该值可能降低延迟，但会增加内存占用 | 0 | 0-1000 |
| KQMALLOC_USE_THP | 设置是否通过madvise()显式控制透明大页；-1表示不调用madvise() | 4KB系统页和2MB大页场景为1；其他场景为0 | -1、0、1 |
| KQMALLOC_NUMA_AWARE | 设置NUMA感知模式 | 2 | 0：禁用；1：NUMA节点粒度；2：CPU插槽粒度；3：将系统NUMA节点轮询映射到4个内部堆组 |
| KQMALLOC_TINY_THRESHOLD | 设置微小对象大小阈值，单位为字节 | 单线程模式为4096；多线程模式为512 | 512-4096 |
| KQMALLOC_LOCAL_THRESHOLD | 设置可在线程本地缓存的对象大小阈值，单位为字节；设置为0可禁用此类线程本地缓存 | 512 | 0-262208（64字节缓存行）；0-262272（128字节缓存行） |
| KQMALLOC_ALIGNED_LOCAL_THRESHOLD | 功能同KQMALLOC_LOCAL_THRESHOLD，但仅作用于aligned_alloc()、memalign()、posix_memalign()、valloc()和pvalloc()分配的对齐对象；设置为0可禁用此类线程本地缓存 | 缓存行大小（64或128） | 0-524288 |
| KQMALLOC_PROFILER_SECONDS | 设置内存分析时间间隔，单位为秒，仅在启用内存分析器时有效 | 60.0 | 0.001-86400.0 |
| KQMALLOC_PROFILER_EXPONENT | 设置随程序运行时间指数增长的内存分析间隔比例，仅在启用内存分析器时有效 | 0.05 | 0.0-0.25 |
| KQMALLOC_HEAP_CACHE_SIZE | 设置分配器可缓存的堆头余量最大值，单位为兆字节，并受KQMALLOC_HEAP_CACHE_RATIO限制 | 单线程模式为900；多线程模式为0 | 大于等于0 |
| KQMALLOC_TINY_HEAP_CACHE_SIZE | 功能同KQMALLOC_HEAP_CACHE_SIZE，但仅作用于微小对象堆 | 与KQMALLOC_HEAP_CACHE_SIZE相同 | 大于等于0 |
| KQMALLOC_SMALL_HEAP_CACHE_SIZE | 功能同KQMALLOC_HEAP_CACHE_SIZE，但仅作用于小对象堆（最大约256KB） | 与KQMALLOC_HEAP_CACHE_SIZE相同 | 大于等于0 |
| KQMALLOC_MIDDLE_HEAP_CACHE_SIZE | 功能同KQMALLOC_HEAP_CACHE_SIZE，但仅作用于中等对象堆（最大约64MB）；aligned_alloc()和calloc()分配的对象可能不受其影响 | 与KQMALLOC_HEAP_CACHE_SIZE相同 | 大于等于0 |
| KQMALLOC_HEAP_CACHE_RATIO | 设置可缓存的堆头余量占堆峰值大小的最大比例，并受KQMALLOC_HEAP_CACHE_SIZE限制 | 100 | 0-100 |
| KQMALLOC_DIRTY_THRESHOLD | 设置每个堆中脏页缓存大小上限，单位为兆字节；不包括KQMALLOC_HEAP_CACHE_SIZE控制的堆头余量 | 1024 | 大于等于2 |
| KQMALLOC_QUICK_CACHE_THRESHOLD | 设置FAST堆内中等对象（约256KB至64MB）的无条件快速缓存阈值，单位为字节 | 缓存行为128字节时为1050624；缓存行为64字节时为1049600 | 大于等于0 |
| KQMALLOC_EXCLUDE | 设置不使用的对象大小级别，取值为分配器内部按对象大小划分的档位（单位为字节），可用于改善对齐并减少缓存未命中 | 单线程模式为"112;224;240"；多线程模式为空字符串 | 以";"分隔的字节数列表，空字符串表示不排除任何级别 |


### 环境变量配置示例

环境变量可以仅对当前应用生效。以下示例将对象地址对齐大小设置为16字节。

```shell
kqmalloc_root=/path/to/kqmalloc-vX.Y.Z
KQMALLOC_ALIGNMENT=16 \
LD_PRELOAD="$kqmalloc_root/HIP09/lib/libkqmalloc.ptag.so" \
./run_your_application
```

如果需要长期生效，也可以将环境变量写入应用程序的启动脚本或服务配置文件。

## 验证加速效果

预编译交付包包含动态库和测试程序，可先使用上述[最小用例](#最小用例)验证基本运行。评估实际业务的加速效果时，请使用实际业务程序或客户已有的性能测试程序，对系统默认内存库和KQMalloc进行A/B测试。

### 测试步骤

1. 确保两次测试使用相同的硬件、操作系统、输入数据、线程绑定方式、系统负载和应用配置。
2. 使用系统默认内存库运行应用，并记录耗时和最大常驻内存。

    ```shell
    /usr/bin/time -v -o system-default.time ./run_your_application
    ```

3. 在相同条件下预加载KQMalloc并再次运行。

    ```shell
    kqmalloc_root=/path/to/kqmalloc-vX.Y.Z
    /usr/bin/time -v -o kqmalloc.time \
        env LD_PRELOAD="$kqmalloc_root/HIP09/lib/libkqmalloc.ptag.so" \
        ./run_your_application
    ```

4. 比较两次输出中的`Elapsed (wall clock) time`和`Maximum resident set size`。建议多次运行并取中位数，以降低系统抖动的影响。

运行时间提升比例可按以下公式计算。

```text
(系统默认内存库运行时间 - KQMalloc运行时间) / 系统默认内存库运行时间 × 100%
```

实际结果受处理器、操作系统、业务负载、线程绑定方式和环境变量配置等因素影响，请以客户环境实测数据为准。

## 修订记录

|文档版本|发布日期|修改说明|
|--|--|--|
|01|2026-09-30|第一次正式发布。|
