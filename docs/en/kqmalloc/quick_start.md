# Quick Start

This document helps you quickly understand the basic concepts, installation and deployment, usage, and performance verification of KQMalloc.

## Installation and Deployment

For details about how to compile the source code and build a dynamic library, see [Installation Guide](./installation_guide.md).

KQMalloc provides the following compilation files:

| Library File| Description|
|-------|------|
| libkqmalloc.so | Single-threaded dynamic library|
| libkqmallocmt.so | Multi-threaded dynamic library|
| libkqmallocmt.debug.so | Debugging version, with assertions and double free check enabled|
| libkqmallocmt.profiler.so | Profiling version, with memory profiling enabled|

## Usage Description

### Enabling Transparent Huge Pages

Before using KQMalloc, you are advised to enable transparent huge pages of the system to achieve better performance.

```shell
echo always > /sys/kernel/mm/transparent_hugepage/enabled
echo madvise > /sys/kernel/mm/transparent_hugepage/defrag
```

### Using KQMalloc in Preloading Mode

KQMalloc optimizes memory allocation functions (such as malloc, free, calloc, and realloc) in standard libraries. You only need to preload it without modifying the application code.

The following uses the version for multi-threaded applications and the Kunpeng 920 processor as an example:

```shell
LD_PRELOAD=/<path to KQMalloc>/HIP09/lib/libkqmallocmt.so ./run_your_application
```

>![](public_sys-resources/icon-note.gif) **NOTE:**
>KQMalloc can also be dynamically loaded to applications. However, this method is not recommended because KQMalloc may not take effect after being loaded.

## Environment Variables

This section describes the environment variables that may need to be configured for using KQMalloc. Set the environment variables as required.

**Table 1** Environment variables of KQMalloc<a id="KQMalloc-environment-variables"></a>

| Variable| Description| Default Value| Value Range|
|---------|------|-------|--------|
| KQMALLOC_ALIGNMENT | Sets the memory alignment size, in bytes.| 4 (incompatible with the C11 standard)| 4; 8; 16; 32; 64; 128 |
| KQMALLOC_C11_COMPLIANT_PROCESSES | Sets the list of processes compatible with the C11 standard. The memory alignment size of these processes is 16 bytes by default.| Empty string| List of process names separated by colons (:), for example, **java8:java15**|
| KQMALLOC_SINGLE_THREADED | Sets the thread mode of the program.| The value is empty for the normal version (automatic detection) and **0** for the slim version.| **0**: single-threaded mode; **1**: multi-threaded mode|
| KQMALLOC_SLAB_CACHE_RATIO | Sets the maximum percentage of the slab size that can be cached to improve performance.| 20 | 0 to 100|
| KQMALLOC_LOCAL_CACHE_RATIO | Sets the percentage of the local cache size of a thread.| 100 (single-threaded); 75 (multi-threaded)| 0 to 100|
| KQMALLOC_USE_THP | Sets whether to use transparent huge pages.| 1 | **1**: Enabled; **0**: Disabled; **-1**: Not specified|
| KQMALLOC_NUMA_AWARE | Sets the NUMA awareness mode.| 0 (single-threaded); 2 (multi-threaded)| **0**: NUMA awareness disabled; **1**: NUMA node-based management; **2**: CPU socket-based management|
| KQMALLOC_TINY_THRESHOLD | Sets the microobject size threshold, in bytes.| 65536+CACHE_LINE_SIZE (single-threaded); 512 (multi-threaded)| 512 to 65536+CACHE_LINE_SIZE|
| KQMALLOC_PROFILER_SECONDS | Sets the memory profiling interval, in seconds (valid only when the memory profiler is enabled).| 60.0 | 0 to 86400.0|
| KQMALLOC_PROFILER_EXPONENT | Sets the memory profiling interval growth rate (valid only when the memory profiler is enabled).| 0.05 | 0-0.25 |
| KQMALLOC_HEAP_CACHE_SIZE | Sets the maximum size of the heap space that can be cached, in megabytes.| 512 (single-threaded); 0 (multi-threaded)| ≥ 0|
| KQMALLOC_HEAP_CACHE_RATIO | Sets the maximum percentage of the heap space that can be cached.| 75 (single-threaded); 100 (multi-threaded)| 0 to 100|
| KQMALLOC_DIRTY_THRESHOLD | Sets the maximum number of dirty pages in each heap.| 256 | ≥ 0|

### Environment Variable Configuration Example

The following uses KQMALLOC_ALIGNMENT as an example:

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

- Processor: New Kunpeng 920 processor model
- Number of threads: 320
- Kernel page table size: 64 KB
- KQMalloc version: v0.19.1
- OS version: openEuler 24.03 (LTS-SP1)

### Test Procedure

1. Run the test with the KQMalloc memory library enabled.

    ```shell
    LD_PRELOAD=/<path to KQMalloc>/lib/libkqmallocmt.so bin/malloc_benchmark -mode 1 -threads 320
    ```

2. Run the test with the default memory library.

    ```shell
    bin/malloc_benchmark -mode 1 -threads 320
    ```

### Result Comparison

Compare the runtime, average memory usage (Average RSS), and maximum memory usage (Max RSS) to verify the performance improvement of KQMalloc.

**Results after KQMalloc is enabled:**

```text
Mode: 1, Time: 68.805 seconds, Average RSS: 0.627 GB, Max RSS: 0.720 GB.
```

**Results when the default memory library is used:**

```text
Mode: 1, Time: 259.727 seconds, Average RSS: 4.507 GB, Max RSS: 4.794 GB.
```

### Performance Profiling

| Metric| KQMalloc | Default Memory Library| Improvement (Percentage)|
|------|----------|--------------|---------|
| Runtime| 68.805 seconds| 259.727 seconds| Improved by 73.5%|
| Average Memory Usage| 0.627 GB | 4.507 GB | Reduced by 86.1%|
| Maximum Memory Usage| 0.720 GB | 4.794 GB | Reduced by 85.0%|

The test results show that KQMalloc significantly improves the runtime and reduces memory usage.
