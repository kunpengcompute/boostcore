# Quick Start

<!-- md-trans-meta sourceCommit=f32aeaef843daccbe95304cc9ebc8c24ba63a286 translatedAt=2026-11-18T03:11:28.074Z pushedAt=2026-11-19T07:14:11.850Z -->

This document helps users quickly understand how to use the KQMalloc V0.22 precompiled dynamic libraries, their environment variables, and performance verification methods.

## Installation and Deployment

For detailed steps on obtaining and extracting the dynamic libraries, see the [Installation Guide](./installation_guide.md). KQMalloc V0.22 provides the following dynamic libraries.

| Library File | Description |
| ------- | ------ |
| libkqmalloc.ptag.so | Release version with the pointer tagging feature enabled |
| libkqmalloc.ptag.debug.so | Debug version with pointer tagging, assertions, and double free() checking enabled |
| libkqmalloc.ptag.profiler.so | Profiler version with pointer tagging and memory profiling enabled |
| libkqmalloc.so | Release version with the pointer tagging feature disabled |
| libkqmalloc.debug.so | Debug version with the pointer tagging feature disabled, with assertions and double free() checking enabled |
| libkqmalloc.profiler.so | Profiler version with the pointer tagging feature disabled, with memory profiling enabled |

All dynamic libraries are parallel allocators. When the allocator starts, it initializes in single-threaded mode and automatically switches to multi-threaded mode after a second thread is detected; this mode cannot be explicitly specified through an environment variable. Versions with the pointer tagging feature disabled are suitable for systems whose kernels do not support pointer tagging, or for apps that use pointer tagging themselves (such as Go and Java).

The following commands use the release version with the pointer tagging feature enabled as an example.

```shell
kqmalloc_root=/path/to/kqmalloc-vX.Y.Z
LD_PRELOAD="$kqmalloc_root/HIP09/lib/libkqmalloc.ptag.so" ./run_your_application
```

## Test Program Introduction

`malloc_benchmark` is a memory allocation test program delivered in the package. It simulates memory allocation and deallocation workloads through different thread collaboration modes, object size distributions, and memory access patterns. The program outputs the run time, speed, and memory usage, and can be used with `LD_PRELOAD` to test either the system default memory library or KQMalloc.

The test program organizes workloads in units of composite objects, each containing 7 independently allocated components. `-objects` controls the number of composite objects, and `-size_min` and `-size_max` control the size of each component in bytes. A complete test consists of several rounds (epochs), each containing several stages (ages), and each stage executes a specified number of iterations.

The following table describes common command-line parameters and their default values. The default values come from the initialization settings in `malloc_benchmark.c`; the default values of `-objects` and `-iterations` vary with `-mode`, and explicitly specifying the corresponding parameters overrides the default values. The values in the **Description** column are examples. For the first run, you can use the [minimal test case](#minimal-test-case) below.

| Variable Name | Description | Default Value |
| ---- | ---- | ---- |
| -threads | Number of test threads. For example, `320` means using 320 threads. | 1 |
| -mode | Thread collaboration mode.<br>`0`: All-to-all mode, where threads share an object pool, simulating cross-thread allocation and deallocation.<br>`1`: Producer/consumer mode, where with an even number of threads, half of the threads allocate and half deallocate; with an odd number of threads, every thread can both allocate and deallocate.<br>`2`: Independent thread mode, where each thread operates only on its own object partition. | 0 |
| -touch | Memory access method after allocation.<br>`0`: Do not proactively access the allocated memory.<br>`1`: Access memory in 4096-byte steps.<br>`2`: Fill memory via `memset()`.<br>`3`: Allocate zero-initialized memory via `calloc()`.<br>`4`: Access memory in 64-byte steps.<br>`5`: Access memory in 65,536-byte steps.<br>`6`: Memory verification mode; write verification data after allocation, and read and check it before freeing. | 1 |
| -objects | Upper limit of the number of composite objects. For example, `4194304` means a maximum of 4,194,304 composite objects; in independent thread mode, the number should be at least equal to the number of threads. | `-mode 0`: 1000000<br>`-mode 1`: 10000<br>`-mode 2`: 1000000 |
| -size_min | Lower limit of the allocation size, in bytes. For example, `8` means a minimum of 8 bytes. | 8 |
| -size_max | Upper limit of the allocation size, in bytes. For example, `16777216` means a maximum of 16 MiB. | 33554432 (32 MiB) |
| -epochs | Number of complete test rounds. For example, `1` means executing 1 test round. | 1 |
| -ages | Number of stages per test round. For example, `100` means each epoch contains 100 ages. | 64 |
| -iterations | Number of iterations per thread per stage. For example, `500000` means each thread executes 500,000 iterations per age; one iteration involves the 7 components of a composite object. | `-mode 0`: 250000<br>`-mode 1`: 400000<br>`-mode 2`: 750000 |
| -temperature | Controls the distribution of object size classes. For example, `0.11` means a size distribution temperature of 0.11. Smaller values favor small objects, and larger values increase the proportion of large objects. This does not refer to CPU temperature. | 0.09 |
| -profiler | Sets whether to enable the built-in allocation performance statistics of the test program.<br>`0`: Disabled<br>`1`: Enabled; outputs the counts, time, and latency statistics of allocation and deallocation by size class. | 0 |
| -trim | Interval (in stages) for calling `malloc_trim(0)`.<br>`0`: Not called<br>`12`: Called at the end of every 12th age in each epoch (the 12th, 24th, 36th, ...); the actual reclamation behavior depends on the memory library used. | 0 |
| -purge | Probability of freeing existing objects at the start of each stage.<br>`0.00`: No additional freeing<br>`0.42`: Frees allocated composite objects with a probability of 42%, with random fluctuation in the actual ratio. The value range is 0 to 1. This parameter does not take effect in producer/consumer mode. | 0.00 |
| -aligned | Allocation size threshold for using `aligned_alloc()`, in bytes.<br>`0`: Not used<br>`65536`: Components no larger than 65,536 bytes use `aligned_alloc()`, and larger components use `malloc()`. The alignment value is determined by the natural alignment rule of each size class, not uniformly aligned to 65,536 bytes. This parameter does not take effect when `-touch 3` is used. | 0 |
| -affinity | Sets whether the test program proactively binds CPUs.<br>`1`: Sets CPU affinity for worker threads in multi-threaded tests.<br>**`0`: Does not proactively set CPU affinity; threads inherit the existing affinity constraints of the process and are scheduled by the OS.** | 1 (Linux) |
| -imbalance | Sets whether to output detailed information about inter-thread imbalance.<br>`0`: Does not output per-thread details; multi-threaded tests still output a summary.<br>`1`: Outputs per-thread details in multi-threaded tests to help compare the run time of worker threads. | 0 |

## Minimal Test Case

The delivery package contains the memory allocation test program `HIP08/bin/malloc_benchmark`. The following uses the HIP08 version as an example to verify dynamic library loading and program running with a single thread and a small test scale. Select the corresponding directory based on your actual processor model; systems or applications that do not support pointer tagging should use `libkqmalloc.so`.

1. Set the decompression directory and check whether the dynamic library can be loaded.

    ```shell
    kqmalloc_root=/path/to/kqmalloc-vX.Y.Z
    LD_PRELOAD="$kqmalloc_root/HIP08/lib/libkqmalloc.ptag.so" \
        ldd "$kqmalloc_root/HIP08/bin/malloc_benchmark"
    ```

    The output should contain the actual path of `HIP08/lib/libkqmalloc.ptag.so`, without any `not found` or preloading failure message.

2. Preload KQMalloc and run the test program.

    ```shell
    LD_PRELOAD="$kqmalloc_root/HIP08/lib/libkqmalloc.ptag.so" \
        "$kqmalloc_root/HIP08/bin/malloc_benchmark" \
        -threads 1 -mode 2 -objects 1000 \
        -size_min 8 -size_max 1024 \
        -epochs 1 -ages 4 -iterations 100000 -affinity 0
    echo $?
    ```

    This example uses 1 thread and the independent thread mode (`-mode 2`), with a maximum of 1000 composite objects (each containing 7 components) and object sizes ranging from 8 to 1,024 bytes. The test executes 1 epoch, each epoch contains 4 ages, and each age executes 100,000 iterations. `-affinity 0` means the test program does not proactively set CPU affinity.

3. Check the test results.

    The following is an excerpt of the actual output obtained on 2026-10-12 on openEuler 24.03 LTS SP3 in the 4 KB system page environment, using the existing HIP08 test program and dynamic library (the remote source code version is 0.21). The environment, parameter, and temperature information is omitted.

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

    `echo $?` should output `0`, indicating that the program exited normally. `Seconds` is the cumulative run time; `Speed` is the number of million component iterations per second counted by the test program; `Virtual` and `Resident` are virtual memory and resident memory, respectively, converted in units of 1024³ bytes and output as GB. The `Mode: 2` at the end summarizes the elapsed time, resident memory, and speed of this test.

The actual elapsed time, memory, and speed vary with the environment and do not need to exactly match the example values. This example is used to verify basic operation; to evaluate acceleration effects, see [Verifying the Acceleration Effect](#verifying-the-acceleration-effect).

## Environment Variables

KQMalloc supports the following environment variables, as listed in [Table 1 KQMalloc environment variables](#kqmalloc-environment-variables). In the table, the default values distinguished by single-threaded mode and multi-threaded mode are automatically selected by the allocator based on its running state.

**Table 1** KQMalloc environment variables<a id="kqmalloc-environment-variables"></a>

| Variable Name | Description | Default Value | Value Range |
| --------- | ------ | ------- | ------- |
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
| KQMALLOC_EXCLUDE | Sets the object size classes to exclude. The values are the size classes divided internally by the allocator by object size (in bytes), which can improve alignment and reduce cache misses. | "112;224;240" in single-threaded mode; an empty string in multi-threaded mode | A semicolon-separated list of byte counts; an empty string means no size class is excluded. |

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

The precompiled delivery package contains dynamic libraries and the test program. You can first use the preceding [minimal test case](#minimal-test-case) to verify basic running. To evaluate the acceleration effect for actual services, use the actual business program or the customer's existing performance test program to perform A/B testing on the system default memory library and KQMalloc.

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

| Version | Date | Description |
| --- | --- | --- |
| 01 | 2026-09-30 | This is the first official release. |
