# KQMalloc（Kunpeng Quick Malloc）介绍

鲲鹏高性能内存库KQMalloc（Kunpeng Quick Malloc）是专为鲲鹏处理器设计的并行内存分配器。分配器初始化时使用单线程模式，检测到第二个线程后自动切换到多线程模式。KQMalloc通过降低内部缓存占用和缓存未命中提升应用性能。从V0.21版本开始，KQMalloc不再提供独立的单线程动态库，并提供启用指针标记特性的libkqmalloc.ptag.so，以及未启用该特性的libkqmalloc.so。

适用场景：内存的申请、释放等。

## 修订记录

|文档版本|发布日期|修改说明|
|--|--|--|
|01|2026-09-30|第一次正式发布。|
