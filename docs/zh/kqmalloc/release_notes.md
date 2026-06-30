# 版本说明书

## V0.20

- 单线程性能大幅提升。
- 重新设计堆标记缓冲区：原三维数组[NUMA 节点][秩][存储单元]改为一维数组[偏移量 + 存储单元]。
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
- 库的64K和2MB页大小的二进制版本已合并为单一二进制版本。
- 移除实验性的HBM支持。
- 优化多线程模式下KQMALLOC_ALIGNMENT的默认值（新值恢复为16）。
- 重新启用KQMALLOC_HEAP_CACHE_SIZE和KQMALLOC_HEAP_CACHE_RATIO。
- 修复超额订阅模式下，因新建slab/block分配而非等待已锁定资源导致内存增长的问题。
- 修复首次调用free()函数（此前未调用malloc()）时可能使用未初始化线程本地分配器的问题。
- 改进嵌入式内存分析器。
- 改进库的调试版本（新增重复free()检查和内存保护机制）。
- 降低精简版分配器的线程本地存储（TLS）内存消耗。
- 发布包中新增库的二进制版本，总计7个：libkqmalloc.so、
  libkqmallocmt.so、libkqmallocmt.slim.so、libkqmallocmt.debug.so、libkqmallocmt.slim.debug.so、
  libkqmallocmt.profiler.so、libkqmallocmt.slim.profiler.so。
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
- 修复`make bin`命令对cmake的依赖（简化CentOS 7下的构建流程）。
- 改进压力测试，覆盖更多对齐方式和分配大小。
- 新增对gcc 14版本的支持。

## V0.10

- 并行分配器针对"生产者-消费者"模式优化，重新设计缓存机制。
- 修复HIP09处理器下`make bin`构建选项的问题。
- 修复HIP11处理器支持：移除自定义memcpy & memset函数，修正断言逻辑。
- CI流程完全迁入代码仓库，包含毕昇4和CentOS 7配置。

## V0.9

- 引入环境变量KQMALLOC_ALIGNMENT，默认值为16。支持4、8、16、32、64、128字节的对齐粒度。
- 修复巨型对象重分配的bug。
- 针对FloorPlan OMP基准测试进行精细调优，现在性能优于jemalloc。
- 微小对象性能优化：无标记堆（untagged heaps）的最大数量提升至40。
- 启用透明超大页（4MB、8MB、16MB……1GB）。
- 实现新的巨型对象分配算法：将munmap()大部分替换为madvise(MADV_DONTNEED)。
- 通过环境变量KQMALLOC_SINGLE_THREADED=1优化单线程模式。
- 新增鲲鹏HIP11处理器支持。
- 因gcc在HIP09处理器下使用-O3 & -Ofast编译时出现错误，开始从gcc 10.3.1_2024.08迁移至BiShengCompiler-4.0.0。
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
- 修复编译警告和代码风格问题。
- 更好地支持OE平台。
