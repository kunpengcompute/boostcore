# 快速入门<a name="ZH-CN_TOPIC_0000002550312025"></a>

本文档帮助用户快速了解KQMalloc的基本概念、安装部署、使用方法及性能验证。

## 安装部署<a name="ZH-CN_TOPIC_0000002518522366"></a>

编译源码并构建动态库的详细步骤请参考[README.md](../../README.md)文档的第6章。

KQMalloc提供以下编译产物：

| 库文件 | 说明 |
|-------|------|
| libkqmalloc.so | 单线程版本动态库 |
| libkqmallocmt.so | 多线程版本动态库 |
| libkqmallocmt.debug.so | 调试版本，启用断言和double free检查功能 |
| libkqmallocmt.profiler.so | 分析版本，启用内存分析功能 |

## 使用方式<a name="ZH-CN_TOPIC_0000002550001997"></a>

### 启用透明大页

在使用KQMalloc前，建议先开启系统透明大页功能，以获得更好的性能表现：

```shell
echo always > /sys/kernel/mm/transparent_hugepage/enabled
echo madvise > /sys/kernel/mm/transparent_hugepage/defrag
```

![](figures/zh-cn_image_0000002518522456.png)

### 以预加载方式使用

KQMalloc通过优化标准库中的内存分配函数（如malloc、free、calloc、realloc等）实现性能提升。用户只需通过动态库预加载方式即可使KQMalloc生效，无需修改应用程序代码。

以多线程场景、鲲鹏HIP09处理器为例：

```shell
LD_PRELOAD=/<path to KQMalloc>/HIP09/lib/libkqmallocmt.so ./run_your_application
```

>![](public_sys-resources/icon-note.gif) **说明：**
>KQMalloc也支持动态链接到应用程序中，但不建议采用此方式，因为可能导致KQMalloc库加载不生效。

## 环境变量<a name="ZH-CN_TOPIC_0000002518682302"></a>

本章节介绍KQMalloc库支持的环境变量，用户可根据实际需求进行配置。

**表 1** KQMalloc库环境变量说明<a id="KQMalloc库环境变量说明"></a>

| 变量名称 | 说明 | 默认值 | 可选值 |
|---------|------|-------|--------|
| KQMALLOC_ALIGNMENT | 设置内存对齐大小，单位为字节 | 4（不兼容C11标准） | 4; 8; 16; 32; 64; 128 |
| KQMALLOC_C11_COMPLIANT_PROCESSES | 设置兼容C11标准的进程列表，列表中的进程内存对齐大小默认设为16字节 | 空字符串 | 用":"分隔的进程名称列表，如：java8:java15 |
| KQMALLOC_SINGLE_THREADED | 设置程序的线程模式 | 常规版本为空（自动检测），slim版本为0 | 0：单线程模式；1：多线程模式 |
| KQMALLOC_SLAB_CACHE_RATIO | 设置为提高性能而缓存的slab最大大小占比 | 20 | 0-100 |
| KQMALLOC_LOCAL_CACHE_RATIO | 设置线程本地缓存大小占比 | 100（单线程），75（多线程） | 0-100 |
| KQMALLOC_USE_THP | 设置是否使用透明大页 | 1 | 1：使用透明大页；0：不使用透明大页；-1：不指定 |
| KQMALLOC_NUMA_AWARE | 设置NUMA感知模式 | 0（单线程），2（多线程） | 0：禁用NUMA感知；1：以NUMA节点粒度管理；2：以CPU插槽粒度管理 |
| KQMALLOC_TINY_THRESHOLD | 设置微对象大小阈值，单位为字节 | 65536+CACHE_LINE_SIZE（单线程），512（多线程） | 512-65536+CACHE_LINE_SIZE |
| KQMALLOC_PROFILER_SECONDS | 设置内存分析时间间隔，单位为秒（仅在启用内存分析器时有效） | 60.0 | 0-86400.0 |
| KQMALLOC_PROFILER_EXPONENT | 设置内存分析时间间隔增长率（仅在启用内存分析器时有效） | 0.05 | 0-0.25 |
| KQMALLOC_HEAP_CACHE_SIZE | 设置可缓存的堆余量空间最大大小，单位为兆字节 | 512（单线程），0（多线程） | 大于等于0 |
| KQMALLOC_HEAP_CACHE_RATIO | 设置可缓存的堆余量空间最大占比 | 75（单线程），100（多线程） | 0-100 |
| KQMALLOC_DIRTY_THRESHOLD | 设置每个堆中"脏页"数量上限 | 256 | 大于等于0 |

### 环境变量配置示例

以配置KQMALLOC_ALIGNMENT为例：

1. 打开"/etc/profile"文件。

    ```shell
    vi /etc/profile
    ```

2. 按"i"进入编辑模式，在"/etc/profile"文件中增加以下内容。

    ```shell
    export KQMALLOC_ALIGNMENT=16
    ```

3. 执行**source**命令使环境变量生效。

    ```shell
    source /etc/profile
    ```

## 验证加速效果<a name="ZH-CN_TOPIC_0000002518682302"></a>

本章节以malloc_benchmark测试用例为例，说明如何验证KQMalloc的性能提升效果。

### 测试环境

- 处理器：鲲鹏HIP09
- 线程数：320
- 内核页表大小：64K
- KQMalloc版本号：v0.19.1
- 操作系统版本：openEuler 24.03 (LTS-SP1)

### 测试步骤

1. 运行启用KQMalloc内存库的测试：

    ```shell
    LD_PRELOAD=/<path to KQMalloc>/lib/libkqmallocmt.so bin/malloc_benchmark -mode 1 -threads 320
    ```

2. 运行使用系统默认内存库的测试：

    ```shell
    bin/malloc_benchmark -mode 1 -threads 320
    ```

### 结果对比

通过比较运行时间、平均内存占用（Average RSS）和最大内存占用（Max RSS）三项指标，验证KQMalloc的性能提升效果。

**启用KQMalloc后的运行结果**：

```text
Mode: 1, Time: 68.805 seconds, Average RSS: 0.627 GB, Max RSS: 0.720 GB.
```

**使用系统默认内存库的运行结果**：

```text
Mode: 1, Time: 259.727 seconds, Average RSS: 4.507 GB, Max RSS: 4.794 GB.
```

### 性能分析

| 指标 | KQMalloc | 系统默认内存库 | 提升比例 |
|------|----------|--------------|---------|
| 运行时间 | 68.805秒 | 259.727秒 | 提升73.5% |
| 平均内存占用 | 0.627 GB | 4.507 GB | 降低86.1% |
| 最大内存占用 | 0.720 GB | 4.794 GB | 降低85.0% |

从测试结果可以看出，KQMalloc在运行时间和内存占用方面均有显著优化效果。
