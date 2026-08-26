# 快速入门

本文档帮助用户快速了解KQMalloc V0.22预编译动态库的使用方法、环境变量和性能验证方法。

## 安装部署

获取和解压动态库的详细步骤请参见[安装指南](./installation_guide.md)。KQMalloc V0.22提供以下动态库。

| 库文件 | 说明 |
| ------- | ---- |
| `libkqmalloc.ptag.so` | 启用指针标记特性的发布版本 |
| `libkqmalloc.ptag.debug.so` | 启用指针标记、断言和double free()检查等功能的调试版本 |
| `libkqmalloc.ptag.profiler.so` | 启用指针标记和内存分析功能的分析版本 |
| `libkqmalloc.so` | 未启用指针标记特性的发布版本 |
| `libkqmalloc.debug.so` | 未启用指针标记特性，启用断言和double free()检查等功能的调试版本 |
| `libkqmalloc.profiler.so` | 未启用指针标记特性，启用内存分析功能的分析版本 |

所有动态库均为并行分配器。分配器启动时按单线程模式初始化，检测到第二个线程后自动切换为多线程模式；该模式不可通过环境变量显式指定。未启用指针标记特性的版本适用于内核不支持指针标记的系统，或自身使用指针标记的应用（如Go、Java）。

以下命令以HIP09处理器上启用指针标记特性的发布版本为例。

```shell
kqmalloc_root=/path/to/kqmalloc-vX.Y.Z
LD_PRELOAD="$kqmalloc_root/HIP09/lib/libkqmalloc.ptag.so" ./run_your_application
```

## 环境变量

如[**表1** KQMalloc库环境变量说明](#KQMalloc库环境变量说明)所示，KQMalloc支持以下环境变量。表中按单线程模式和多线程模式区分的默认值由分配器根据运行状态自动选择。

**表1** KQMalloc库环境变量说明<a id="KQMalloc库环境变量说明"></a>

| 变量名称 | 说明 | 默认值 | 可选值 |
|---------|------|-------|-------|
| KQMALLOC_ALIGNMENT | 设置对象地址的对齐大小，单位为字节。无效值将被忽略 | 单线程模式为4；多线程模式为16 | 4、8、16、32、64；缓存行为128字节时还支持128 |
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
| KQMALLOC_EXCLUDE | 设置不使用的对象大小级别，可用于改善对齐并减少缓存未命中 | 单线程模式为"112;224;240"；多线程模式为空字符串 | 以";"分隔的对象大小级别列表 |

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

预编译交付包只包含动态库。请使用实际业务程序或客户已有的性能测试程序，对系统默认内存库和KQMalloc进行A/B测试。

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

|发布日期|修订记录|
|--|--|
|2026-09-30|第三次正式发布。<br><ul><li>优化文档结构，按[安装部署](#安装部署)、[环境变量](#环境变量)和[验证加速效果](#验证加速效果)组织文档内容。</li><li>[更新V0.22六种动态库的用途、选择原则及预加载方法。](#安装部署)</li><li>[新增V0.21、V0.22环境变量说明，并更新变量默认值、取值范围和配置示例。](#环境变量)</li><li>[将旧版本固定性能数据更新为客户业务程序的A/B性能验证方法。](#验证加速效果)</li></ul>|
|2026-06-30|第二次正式发布。<br><ul><li>[新增V0.20四种动态库的用途及预加载使用说明。](#安装部署)</li><li>[新增分级堆缓存和Quick Cache阈值环境变量，更新脏页缓存阈值说明及配置示例。](#环境变量)</li><li>[新增基于malloc_benchmark的加速效果验证与性能分析。](#验证加速效果)</li></ul>|
|2026-03-30|第一次正式发布。<br><ul><li>[新增源码编译、透明大页启用和动态库预加载说明。](#安装部署)</li><li>[新增KQMalloc环境变量及配置方法。](#环境变量)</li></ul>|
