# Quick Start

This document helps you quickly understand the usage and performance verification of KQMalloc.

## Installation and Deployment

For detailed steps on obtaining and using the dynamic library, refer to the [Installation Guide](./installation_guide.md). The compilation artifacts are shown in the table below.

| Library File| Description|
| ------- | ------ |
| libkqmalloc.so | Single-threaded dynamic library|
| libkqmallocmt.so | Multi-threaded dynamic library|
| libkqmallocmt.debug.so | Debugging version, with assertions and double free check enabled|
| libkqmallocmt.profiler.so | Profiling version, with memory profiling enabled|

## Environment Variables

[**Table 1** Environment variables of KQMalloc](#kqmalloc-environment-variables) lists the environment variables that may need to be configured for using KQMalloc. Set the environment variables as required.

**Table 1** Environment variables of KQMalloc<a id="kqmalloc-environment-variables"></a>

| Variable| Description| Default Value| Value Range|
| --------- | ------ | ------- | -------- |
| KQMALLOC_ALIGNMENT | Sets the memory alignment size, in bytes.| 4 (incompatible with the C11 standard)| 4, 8, 16, 32, 64, 128 |
| KQMALLOC_C11_COMPLIANT_PROCESSES | Sets the list of processes compatible with the C11 standard. The memory alignment size of these processes is 16 bytes by default.| Empty string| List of process names separated by colons (:), for example, **java8:java15**|
| KQMALLOC_SINGLE_THREADED | Sets the thread mode of the program.| The value is empty for the normal version (automatic detection) and **0** for the slim version.| **0**: single-threaded mode <br> **1**: multi-threaded mode|
| KQMALLOC_SLAB_CACHE_RATIO | Sets the maximum percentage of the slab size that can be cached to improve performance.| 20 | 0 to 100|
| KQMALLOC_LOCAL_CACHE_RATIO | Sets the percentage of the local cache size of a thread.| 100 (single-threaded); 75 (multi-threaded)| 0 to 100|
| KQMALLOC_USE_THP | Sets whether to use transparent huge pages.| 1 | **1**: Enabled <br> **0**: Disabled <br> **-1**: Not specified|
| KQMALLOC_NUMA_AWARE | Sets the NUMA awareness mode.| 0 (single-threaded); 2 (multi-threaded)| **0**: NUMA awareness disabled <br> **1**: NUMA node-based management <br> **2**: CPU socket-based management|
| KQMALLOC_TINY_THRESHOLD | Sets the microobject size threshold, in bytes.| 65536+CACHE_LINE_SIZE (single-threaded); 512 (multi-threaded)| 512 to 65536+CACHE_LINE_SIZE|
| KQMALLOC_PROFILER_SECONDS | Sets the memory profiling interval, in seconds (valid only when the memory profiler is enabled).| 60.0 | 0 to 86400.0|
| KQMALLOC_PROFILER_EXPONENT | Sets the memory profiling interval growth rate (valid only when the memory profiler is enabled).| 0.05 | 0-0.25 |
| KQMALLOC_HEAP_CACHE_SIZE | Sets the maximum size of the heap space that can be cached, in megabytes.| 512 (single-threaded); 0 (multi-threaded)| ≥ 0|
| KQMALLOC_HEAP_CACHE_RATIO | Sets the maximum percentage of the heap space that can be cached.| 75 (single-threaded); 100 (multi-threaded)| 0 to 100|
| KQMALLOC_DIRTY_THRESHOLD | Sets the maximum number of dirty pages in each heap.| 256 | ≥ 0|
| KQMALLOC_TINY_HEAP_CACHE_SIZE | Functions the same as KQMALLOC_HEAP_CACHE_SIZE, but only controls the tiny object heap (the upper limit of allocation size is approximately 1,024 bytes). | 512 (single-threaded), 0 (multi-threaded) | ≥ 0 |
| KQMALLOC_SMALL_HEAP_CACHE_SIZE | Functions the same as KQMALLOC_HEAP_CACHE_SIZE, but only acts on the small object heap (up to approximately 128 KB). | 512 (single-threaded), 0 (multi-threaded) | ≥ 0 |
| KQMALLOC_MIDDLE_HEAP_CACHE_SIZE | Functions the same as KQMALLOC_HEAP_CACHE_SIZE, but only controls the middle object heap (the upper limit of memory is approximately 128 MB). Objects allocated via `aligned_alloc()` or `calloc()` are not controlled by this environment variable. | 512 (single-threaded), 0 (multi-threaded) | ≥ 0 |
| KQMALLOC_QUICK_CACHE_THRESHOLD | Controls the unconditional quick caching of middle-sized objects within the quick heap (object allocation range: 128 KB to 64 MB). | 1050624 (CACHE_LINE_SIZE=128), 1049600 (CACHE_LINE_SIZE=64) | ≥ 0 |

### Environment Variable Configuration Example

The following uses `KQMALLOC_ALIGNMENT` as an example:

1. Open the **/etc/profile** file.

    ```shell
    vi /etc/profile
    ```

2. Press **i** to enter the insert mode and add the following content to the **/etc/profile** file:

    ```shell
    export KQMALLOC_ALIGNMENT=16
    ```

3. Run the **source** command for the environment variable to take effect.

    ```shell
    source /etc/profile
    ```

## Verifying the Acceleration Effect

The following uses the malloc_benchmark test case as an example to describe how to verify the performance improvement of KQMalloc.

### Test Environment

- Processor: Kunpeng 920
- Number of threads: 320
- Kernel page table size: 64 KB
- KQMalloc version: v0.19.1
- OS version: openEuler 24.03 (LTS-SP1)

### Test Procedure

Use the default system memory library and KQMalloc memory library for the tests.

**Run a test using the default system memory library.**

 ```shell
 bin/malloc_benchmark -mode 1 -threads 320
 ```

**Run a test with the KQMalloc memory library enabled.**

 ```shell
 LD_PRELOAD=/<path to KQMalloc>/lib/libkqmallocmt.so bin/malloc_benchmark -mode 1 -threads 320
 ```

### Result Comparison

Compare the running time (`Time`), average memory usage (`Average RSS`), and maximum memory usage (`Max RSS`) to verify the performance improvement of KQMalloc.

**Results when the default memory library is used:**

```text
Mode: 1, Time: 259.727 seconds, Average RSS: 4.507 GB, Max RSS: 4.794 GB.
```

**Results after KQMalloc is enabled:**

```text
Mode: 1, Time: 68.805 seconds, Average RSS: 0.627 GB, Max RSS: 0.720 GB.
```

### Performance Analysis

| Metric| KQMalloc | Default Memory Library| Improvement (Percentage)|
| ------ | ---------- | -------------- | --------- |
| Time | 68.805 seconds| 259.727 seconds| Improved by 73.5%|
| Average RSS | 0.627 GB | 4.507 GB | Reduced by 86.1%|
| Max RSS | 0.720 GB | 4.794 GB | Reduced by 85.0%|

The test results show that KQMalloc significantly reduces the running time and memory usage.
