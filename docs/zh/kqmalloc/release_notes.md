# 版本说明书

## 版本配套说明

### 产品版本信息

<a name="table62675726"></a>

<table><tbody><tr id="row41561572"><th class="firstcol" valign="top" width="42.17%" id="mcps1.1.3.1.1"><p id="p11044137"><a name="p11044137"></a><a name="p11044137"></a>产品名称</p>
</th>
<td class="cellrowborder" valign="top" width="57.830000000000005%" headers="mcps1.1.3.1.1 "><p id="p15977216939713"><a name="p15977216993713"></a><a name="p1597721693713"></a>Kunpeng BoostKit</p>
</td>
</tr>
<tr id="row924726251"><th class="firs9tcol" valign="top" width="42.17%" id="mcps19.1.3.2.1"><p id="p566699300"><a name="p566699300"></a><a name="p566699300"></a>产品版本</p>
</th>
<td class="cellrowborder" valign="top" width="57.830000000000005%" headers="mcps1.1.3.2.1 "><p id="p11923034"><a name="p11923034"></a><a name="p11923034"></a><span id="text14311218114"><a name="text14311218114"></a><a name="text14311218114"></a>26.2.RC1</span></p>
</td>
</tr>
<tr id="row19308119171892"><th class="firstcol" valign="top" width="42.17%" id="mcps1.1.3.3.1"><p id="p2030912172097"><a name="p2030912172097"></a><a name="p2030912172097"></a>软件名称</p>
</th>
<td class="cellrowborder" valign="top" width="57.830000000000005%" headers="mcps1.1.3.3.1 "><p id="p1730912179911"><a name="p1730912179911"></a><a name="p17309912179911"></a><span id="text171910917111119"><a name="text17191017111119"></a><a name="text17191017111119"></a>kqmalloc</span></p>
</td>
</tr>
<tr id="row199308111718"><th class="firstcol" valign="top" width="42.17%" id="mcps1.1.3.3.1"><p id="p20390912172097"><a name="p2030912172097"></a><a name="p2030912172097"></a>软件版本</p>
</th>
<td class="cellrowborder" valign="top" width="57.830000000000005%" headers="mcps1.1.3.3.1 "><p id="p17309121799911"><a name="p1730912179911"></a><a name="p1730912179911"></a><span id="text171919017111119"><a name="text17191017111119"></a><a name="text17191017111119"></a>V0.22</span></p>
</td>
</tr>
</tbody>
</table>

### 操作系统、编译器和CPU配套说明

| 操作系统 | CPU类型 | 编译器 |
| ------------------------- | --------- | ------------------ |
| openEuler 20.03 LTS SP1 | 鲲鹏处理器 | GCC 7.3.0 / BiSheng 4.2.0 |
| openEuler 22.03 LTS SP3 | 鲲鹏处理器 | GCC 10.3.1 / BiSheng 4.2.0 |
| openEuler 24.03 LTS SP3 | 鲲鹏处理器 | GCC 12.3.1 / clang 14.0.6 |

## 版本使用注意事项

版本详细使用注意事项请参见《[安装指南](./installation_guide.md)》。

## V0.22

### 更新说明

- 大幅提升aligned_alloc()、memalign()、posix_memalign()、valloc()和pvalloc()系列函数的性能。
- 新增KQMALLOC_ALIGNED_LOCAL_THRESHOLD、KQMALLOC_LOCAL_CACHE_LOW和KQMALLOC_ALIGNED_LOCAL_CACHE_LOW环境变量。

**修改特性<a name="zh-cn_topic_0000002518399190_section16450949161512"></a>**

无

**删除特性<a name="zh-cn_topic_0000002518399190_section9218125814159"></a>**

无

### 已解决的问题

无

### 遗留问题

无

## 版本配套文档

### V0.22版本配套文档

|文档名称|内容简介|交付形式|
|--|---|---|
|《版本说明书》|本文档提供kqmalloc的版本发布信息。|开源仓|
|《安装指南》|本文档提供kqmalloc的安装使用指导。|开源仓|
|《快速入门》|本文档提供kqmalloc推理优化特性快速入门指导。|开源仓|
|《API参考》|本文档提供kqmalloc推理优化特性API使用说明。|开源仓|

### 获取文档的方法

您可以通过访问[开源仓](https://gitcode.com/boostkit/boostcore/blob/master/docs/zh/kqmalloc/README.md)浏览和获取相关文档。

## V0.21

- 不再提供独立的单线程动态库。
- 实现指针标记（pointer tag）特性。
- 调整二进制库命名：libkqmalloc.so现为并行版KQMalloc；libkqmalloc.ptag.so为启用指针标记特性的并行版KQMalloc。
- 新增KQMALLOC_EXCLUDE、KQMALLOC_LOCAL_CACHE_MIN、KQMALLOC_LOCAL_CACHE_MAX和KQMALLOC_LOCAL_THRESHOLD环境变量。
- KQMALLOC_LOCAL_CACHE_RATIO不仅可用于减小本地缓存，也可用于增大本地缓存。
- 移除KQMALLOC_SINGLE_THREADED环境变量。
- 实现free_sized()、free_aligned_sized()、valloc()和pvalloc()标准函数。
- 支持512MB透明大页。
- KQMALLOC_NUMA_AWARE新增轮询（round-robin）模式。

## V0.20

- 单线程性能大幅提升。
- 重新设计堆标记缓冲区：原三维数组(NUMA 节点)(秩)(存储单元)改为一维数组(偏移量 + 存储单元)。
- 优化最大64MB的大对象高频分配逻辑，适配类MySQL业务负载场景。
- 在不损失性能的前提下，显著降低生产者-消费者基准测试的内存占用。
- 新增环境变量配置项：KQMALLOC_TINY_HEAP_CACHE_SIZE、KQMALLOC_SMALL_HEAP_CACHE_SIZE、KQMALLOC_MIDDLE_HEAP_CACHE_SIZE、KQMALLOC_QUICK_CACHE_THRESHOLD。
- KQMALLOC_DIRTY_THRESHOLD含义由脏页数量上限调整为脏页缓存大小上限，单位为兆字节，默认值由256调整为1024，最低生效值为2。
- 大幅提升GREAT内存分配基准测试跑分性能。

## V0.19

- 大幅提升并行处理能力，可管理数千线程和TB级内存。
- 启用鲲鹏HIP10A处理器支持。
- 基于pthread_atfork()框架实现了fork()支持。
- 实现malloc_trim()函数。
- 重新设计aligned_alloc()实现，现在支持最高128MB的对齐粒度。
- 实现KQMALLOC_DIRTY_THRESHOLD、KQMALLOC_PROFILER_SECONDS、KQMALLOC_PROFILER_EXPONENT环境变量。
- 重新设计巨型对象分配实现。
- 库的处理器二进制版本重命名为HIP08、HIP09、HIP10A、HIP11和HIP12。
- 库的64K和2MB页大小的二进制版本已合并为单一二进制版本。
- 移除实验性的HBM支持。
- 优化多线程模式下KQMALLOC_ALIGNMENT的默认值（新值恢复为16）。
- 重新启用KQMALLOC_HEAP_CACHE_SIZE和KQMALLOC_HEAP_CACHE_RATIO。
- 修复超额订阅模式下，因新建slab/block分配而非等待已锁定资源导致内存增长的问题。
- 修复首次调用free()函数（此前未调用malloc()）时可能使用未初始化线程本地分配器的问题。
- 改进嵌入式内存分析器。
- 改进库的调试版本（新增重复free()检查和内存保护机制）。
- 降低线程本地存储（TLS）内存消耗，因此不再需要精简版分配器。
- 移除精简版分配器。
- 发布包中的二进制库总计4个：libkqmalloc.so、libkqmallocmt.so、libkqmallocmt.debug.so和libkqmallocmt.profiler.so。
- 改进malloc基准测试。
- 移除KQMALLOC_HUGE_THRESHOLD、KQMALLOC_USE_HBM环境变量。

## V0.18

- 启用鲲鹏HIP12处理器支持，优化HIP11处理器支持。
- 为小栈大小的应用添加精简版分配器。
- 内存消耗和性能方面的多项优化。
- 撤销未文档化的KQMALLOC_HEAP_CACHE_SIZE和KQMALLOC_HEAP_CACHE_RATIO。
- 修复宽松内存序同步相关的bug。
- 修复在已终止线程上下文使用分配器的bug。
- 支持毕昇5编译器。
- 新增更多诊断功能、基准测试及测试优化。

## V0.17

- 实现KQMALLOC_LOCAL_CACHE_RATIO环境变量。
- 实现KQMALLOC_TINY_THRESHOLD环境变量。
- 优化多线程模式下KQMALLOC_ALIGNMENT的默认值（新值为64）。
- 优化多线程模式下KQMALLOC_HEAP_CACHE_SIZE的默认值（新值为0）。
- 优化多线程模式下KQMALLOC_HEAP_CACHE_RATIO的默认值（新值为0）。

## V0.16

- 实现KQMALLOC_HUGE_THRESHOLD环境变量。

## V0.15

- 实现基于处理器插槽（package）级粒度的拓扑感知能力。
- KQMALLOC_NUMA_AWARE的默认值设为2（基于处理器插槽级粒度的NUMA感知）。

## V0.14

- 为鲲鹏HIP11和HIP12启用64字节缓存行大小。
- 实现nontemporal_malloc()、nontemporal_free()调用，用于分配长生命周期对象。
- 内部分配器结构的小幅优化。
- 引入USE_ALLOCATOR_BY_VALUE宏，用于控制TLS行为，以适配Great Fiber库集成。

## V0.13

- 修复free()函数中在某些情况下导致SIGSEGV的bug。
- 实现KQMALLOC_NUMA_AWARE环境变量。设置KQMALLOC_NUMA_AWARE=0可禁用库的NUMA感知能力。

## V0.12

- 优化KQMALLOC_ALIGNMENT环境变量：新默认值为KQMALLOC_ALIGNMENT=4（不符合C11标准）。
- 实现KQMALLOC_C11_COMPLIANT_PROCESSES环境变量，以适配KQMALLOC_ALIGNMENT的新默认值。
  该变量是冒号分隔（":"）的进程名列表，列表内进程需要符合C11标准的对齐方式。
  对于此列表中的进程，除非显式指定，否则会隐式应用KQMALLOC_ALIGNMENT=16的默认值。
  默认值为KQMALLOC_C11_COMPLIANT_PROCESSES=java。
- 并行KQMalloc在SPEC CPU 2017基准测试中的单线程性能在开箱即用场景下得到提升
  （即"无环境变量"使用场景）。
- 优化KQMALLOC_USE_THP环境变量：KQMALLOC_USE_THP=-1表示不使用madvise()设置MADV_HUGEPAGE或MADV_NOHUGEPAGE。
- （实验性）为HIP11启用HBM。实现实验性环境变量KQMALLOC_USE_HBM=1（README.md中未文档化，未来可能变更）。
  该变量允许在16-31号NUMA节点分配内存。目标系统必须安装libnuma.so库以支持动态加载。

## V0.11

- 分配器现在100%支持NUMA感知。此前版本仅对大于128KB的对象实现NUMA支持。
- 实现KQMALLOC_USE_THP环境变量，用于禁用/启用透明大页（THP）。默认值为KQMALLOC_USE_THP=1。
- 修复整数溢出bug（使用malloc、calloc、realloc分配"-1"字节）。
- 修复线程销毁后使用分配器的bug（638.imagick_s测试用例）。
- 改进压力测试，覆盖更多对齐方式和分配大小。
- 新增对gcc 14版本的支持。

## V0.10

- 并行分配器针对"生产者-消费者"模式优化，重新设计缓存机制。
- 修复HIP11处理器支持：移除自定义memcpy & memset函数，修正断言逻辑。

## V0.9

- 引入环境变量KQMALLOC_ALIGNMENT，默认值为16。支持4、8、16、32、64、128字节的对齐粒度。
- 修复巨型对象重分配的bug。
- 针对FloorPlan OMP基准测试进行精细调优，现在性能优于jemalloc。
- 微小对象性能优化：无标记堆（untagged heaps）的最大数量提升至40。
- 启用透明超大页（4MB、8MB、16MB……1GB）。
- 实现新的巨型对象分配算法：将munmap()大部分替换为madvise(MADV_DONTNEED)。
- 通过环境变量KQMALLOC_SINGLE_THREADED=1优化单线程模式。
- 新增鲲鹏HIP11处理器支持。
- 实现带缓存优化的环境变量KQMALLOC_HEAP_CACHE_SIZE。
- 调整基准测试：触达已分配内存、估算常驻内存、添加内存屏障以测量操作延迟（而非吞吐量）。

## V0.8

- NUMA感知（仅针对>128KB的对象）。
- 引入无标记堆（untagged heaps）算法。
- 降低内存消耗。
- 将堆内存的munmap()替换为madvise(MADV_DONTNEED)。
- 不再使用MADV_FREE。
- 为SPEC测试新增KQMALLOC_ALIGNMENT=4环境变量（也支持8，默认值为16）。
- 改进并行测试。

## V0.7

- 并行分配器beta版本发布，附带测试用例。

## V0.6

- 导出函数的代码对齐（提升性能稳定性）。

## V0.5

- 大幅改进测试用例。
- 整合代码整洁性反馈。
- 重命名为KQMalloc。
- 更好地支持OE平台。
