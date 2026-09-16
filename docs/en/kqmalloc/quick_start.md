# Quick Start

This document helps users quickly understand how to use the KQMalloc V0.22 precompiled dynamic libraries, their environment variables, and performance verification methods.

## Installation and Deployment

For detailed steps on obtaining and extracting the dynamic libraries, see the [Installation Guide](./installation_guide.md). KQMalloc V0.22 provides the following dynamic libraries.

| Library File| Description|
| ------- | ------ |
| `libkqmalloc.ptag.so` | Release version with the pointer tagging feature enabled |
| `libkqmalloc.ptag.debug.so` | Debug version with pointer tagging, assertions, and double free() checking enabled |
| `libkqmalloc.ptag.profiler.so` | Profiler version with pointer tagging and memory profiling enabled |
| `libkqmalloc.so` | Release version with the pointer tagging feature disabled |
| `libkqmalloc.debug.so` | Debug version with the pointer tagging feature disabled, with assertions and double free() checking enabled |
| `libkqmalloc.profiler.so` | Profiler version with the pointer tagging feature disabled, with memory profiling enabled |

All dynamic libraries are parallel allocators. When the allocator starts, it initializes in single-threaded mode and automatically switches to multi-threaded mode after a second thread is detected; this mode cannot be explicitly specified through an environment variable. Versions with the pointer tagging feature disabled are suitable for systems whose kernels do not support pointer tagging, or for apps that use pointer tagging themselves (such as Go and Java).

The following commands use the release version with the pointer tagging feature enabled as an example.

```shell
kqmalloc_root=/path/to/kqmalloc-vX.Y.Z
LD_PRELOAD="$kqmalloc_root/HIP09/lib/libkqmalloc.ptag.so" ./run_your_application
```

## Environment Variables

KQMalloc supports the following environment variables in [**Table 1** KQMalloc environment variables](#kqmalloc-environment-variables). In the table, the default values distinguished by single-threaded mode and multi-threaded mode are automatically selected by the allocator based on its running state.

**Table 1** KQMalloc environment variables<a id="kqmalloc-environment-variables"></a>

| Variable Name | Description | Default Value | Value Range |
|---------|------|-------|-------|
| KQMALLOC_ALIGNMENT | Sets the alignment size of object addresses, in bytes. Invalid values are ignored. | 4 in single-threaded mode; 16 in multi-threaded mode | 4, 8, 16, 32, 64; 128 is also supported when the cache line size is 128 bytes. |
| KQMALLOC_C11_COMPLIANT_PROCESSES | Sets the list of processes forced to use C11-compliant 16-byte pointer alignment, and affects the default value of KQMALLOC_ALIGNMENT. | java | A list of process names separated by ":", such as java8:java15 |
| KQMALLOC_SLAB_CACHE_RATIO | Sets the maximum ratio of slab space that the allocator can cache to improve performance. | 20 | 0-100 |
| KQMALLOC_LOCAL_CACHE_RATIO | Sets the relative size of the thread-local cache for tiny and small objects at each size class. This can be used to increase or decrease the local cache capacity. | 100.0 | 0.0-1000.0 |
| KQMALLOC_LOCAL_CACHE_MIN | Sets the minimum physical capacity of the thread-local cache for tiny and small objects at each size class. It takes effect after KQMALLOC_LOCAL_CACHE_RATIO. | 2 in normal mode; 0 in oversubscription mode | 0-1000 |
| KQMALLOC_LOCAL_CACHE_MAX | Sets the maximum physical capacity of the thread-local cache for tiny and small objects at each size class. It takes effect after KQMALLOC_LOCAL_CACHE_RATIO. | 1000 | 0-10000 |
| KQMALLOC_LOCAL_CACHE_LOW | Sets the dynamic minimum capacity of the thread-local cache for regular objects. Increasing this value may reduce latency but increases memory usage. | 0 | 0-1000 |
| KQMALLOC_ALIGNED_LOCAL_CACHE_LOW | Sets the dynamic minimum capacity of the thread-local cache for aligned objects. Increasing this value may reduce latency but increases memory usage. | 0 | 0-1000 |
| KQMALLOC_USE_THP | Sets whether to explicitly control transparent huge pages through madvise(); -1 means madvise() is not called. | 1 for the 4 KB system page and 2 MB huge page scenarios; 0 for other scenarios | -1, 0, 1 |
| KQMALLOC_NUMA_AWARE | Sets the NUMA-aware mode. | 2 | 0: disabled; 1: NUMA node granularity; 2: CPU socket granularity; 3: Map system NUMA nodes to 4 internal heap groups in a round-robin manner |
| KQMALLOC_TINY_THRESHOLD | Sets the threshold for tiny object size, in bytes. | 4096 in single-threaded mode; 512 in multi-threaded mode | 512-4096 |
| KQMALLOC_LOCAL_THRESHOLD | Sets the threshold for the object size that can be cached in the thread-local cache, in bytes; setting it to 0 disables this type of thread-local cache. | 512 | 0-262208 (64-byte cache line); 0-262272 (128-byte cache line) |
| KQMALLOC_ALIGNED_LOCAL_THRESHOLD | Same functionality as KQMALLOC_LOCAL_THRESHOLD, but applies only to aligned objects allocated by aligned_alloc(), memalign(), posix_memalign(), valloc(), and pvalloc(); setting it to 0 disables this type of thread-local cache. | Cache line size (64 or 128) | 0-524288 |
| KQMALLOC_PROFILER_SECONDS | Sets the memory profiling interval, in seconds, effective only when the memory profiler is enabled. | 60.0 | 0.001-86400.0 |
| KQMALLOC_PROFILER_EXPONENT | Sets the exponential growth rate of the memory profiling interval over program runtime, effective only when the memory profiler is enabled. | 0.05 | 0.0-0.25 |
| KQMALLOC_HEAP_CACHE_SIZE | Sets the maximum heap header margin that the allocator can cache, in megabytes, and is limited by KQMALLOC_HEAP_CACHE_RATIO. | 900 in single-threaded mode; 0 in multi-threaded mode | Greater than or equal to 0 |
| KQMALLOC_TINY_HEAP_CACHE_SIZE | Same functionality as KQMALLOC_HEAP_CACHE_SIZE, but applies only to tiny object heaps. | Same as KQMALLOC_HEAP_CACHE_SIZE | Greater than or equal to 0 |
| KQMALLOC_SMALL_HEAP_CACHE_SIZE | Same functionality as KQMALLOC_HEAP_CACHE_SIZE, but applies only to small object heaps (up to about 256 KB). | Same as KQMALLOC_HEAP_CACHE_SIZE | Greater than or equal to 0 |
| KQMALLOC_MIDDLE_HEAP_CACHE_SIZE | Same functionality as KQMALLOC_HEAP_CACHE_SIZE, but applies only to middle object heaps (up to about 64 MB); objects allocated by aligned_alloc() and calloc() may not be affected by it. | Same as KQMALLOC_HEAP_CACHE_SIZE | Greater than or equal to 0 |
| KQMALLOC_HEAP_CACHE_RATIO | Sets the maximum ratio of the cacheable heap header margin to the peak heap size, and is limited by KQMALLOC_HEAP_CACHE_SIZE. | 100 | 0-100 |
| KQMALLOC_DIRTY_THRESHOLD | Sets the upper limit of the dirty page cache size in each heap, in megabytes; it does not include the heap header margin controlled by KQMALLOC_HEAP_CACHE_SIZE. | 1024 | Greater than or equal to 2 |
| KQMALLOC_QUICK_CACHE_THRESHOLD | Sets the unconditional quick cache threshold for middle objects (about 256 KB to 64 MB) in the FAST heap, in bytes. | 1050624 for 128-byte cache line; 1049600 for 64-byte cache line | Greater than or equal to 0 |
| KQMALLOC_EXCLUDE | Sets the object size classes that are not used, which can improve alignment and reduce cache misses. | "112;224;240" in single-threaded mode; an empty string in multi-threaded mode | A list of object size classes separated by ";" |

### Environment Variable Configuration Example

Environment variables can take effect only for the current app. The following example sets the object address alignment size to 16 bytes.

```shell
kqmalloc_root=/path/to/kqmalloc-vX.Y.Z
KQMALLOC_ALIGNMENT=16 \
LD_PRELOAD="$kqmalloc_root/HIP09/lib/libkqmalloc.ptag.so" \
./run_your_application
```

If long-term effect is required, the environment variables can also be written into the app's startup script or service configuration file.

## Verifying the Acceleration Effect

The precompiled delivery package contains only the dynamic libraries. Use the actual business program or the customer's existing performance test program to perform A/B testing on the system default memory library and KQMalloc.

### Test Procedure

1. Ensure that the two tests use the same hardware, operating system, input data, thread binding method, system load, and app configuration.

2. Run the app using the system default memory library, and record the elapsed time and maximum resident set size.

    ```shell
    /usr/bin/time -v -o system-default.time ./run_your_application
    ```

3. Preload KQMalloc under the same conditions and run again.

    ```shell
    kqmalloc_root=/path/to/kqmalloc-vX.Y.Z
    /usr/bin/time -v -o kqmalloc.time \
        env LD_PRELOAD="$kqmalloc_root/HIP09/lib/libkqmalloc.ptag.so" \
        ./run_your_application
    ```

4. Compare the `Elapsed (wall clock) time` and `Maximum resident set size` in the two outputs. It is recommended to run multiple times and take the median to reduce the impact of system jitter.

The running time improvement ratio can be calculated using the following formula.

```text
(System default library runtime - KQMalloc runtime) / System default library runtime × 100%
```

Results depend on CPU, OS, workload, thread binding, and environment variable configuration. Always verify with your own environment measurements.

## Change History

|Document Version|Date|Description|
|--|--|--|
|01|2026-09-30|This is the first official release.|
