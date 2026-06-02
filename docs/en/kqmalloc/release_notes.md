# Release Notes

## V0.19

- Significantly improved parallel processing capabilities, capable of managing thousands of threads and TB-level memory.
- Implemented **fork()** support based on the **pthread_atfork()** framework.
- Implemented the **malloc_trim()** function.
- Redesigned the implementation of **aligned_alloc()** to support an alignment granularity of up to 128 MB.
- Implemented the environment variables **KQMALLOC_DIRTY_THRESHOLD**, **KQMALLOC_PROFILER_SECONDS**, and **KQMALLOC_PROFILER_EXPONENT**.
- Redesigned the implementation of humongous object allocation.
- Merged the binary versions of the library with 64 KB and 2 MB page sizes into a single binary version.
- Removed experimental HBM support.
- Optimized the default value of **KQMALLOC_ALIGNMENT** in multi-threaded mode (new value restored to **16**).
- Re-enabled **KQMALLOC_HEAP_CACHE_SIZE** and **KQMALLOC_HEAP_CACHE_RATIO**.
- Fixed an issue where memory growth occurred due to creating new slabs/blocks instead of waiting for locked resources in oversubscription mode.
- Fixed an issue where the first call to **free()** (without having called **malloc()** before) might use an uninitialized thread-local allocator.
- Improved the embedded memory profiler.
- Improved the debug version of the library (added duplicate **free()** checks and memory protection mechanisms).
- Reduced the thread-local storage (TLS) memory consumption of the slim allocator.
- Added seven new binary versions of the library in the release package: **libkqmalloc.so**,
  **libkqmallocmt.so**, **libkqmallocmt.slim.so**, **libkqmallocmt.debug.so**, **libkqmallocmt.slim.debug.so**,
  **libkqmallocmt.profiler.so**, and **libkqmallocmt.slim.profiler.so**.
- Improved the **malloc** benchmark tests.
- Removed the environment variables **KQMALLOC_HUGE_THRESHOLD** and **KQMALLOC_USE_HBM**.

## V0.18

- Optimized support for Kunpeng 920 processors.
- Added a slim allocator for applications with small stack sizes.
- Performed multiple optimizations in memory consumption and performance.
- Removed undocumented environment variables **KQMALLOC_HEAP_CACHE_SIZE** and **KQMALLOC_HEAP_CACHE_RATIO**.
- Fixed bugs related to relaxed memory ordering synchronization.
- Fixed bugs related to using the allocator in a terminated thread context.
- Supported BiSheng Compiler 5.
- Added more diagnostic functions, benchmarks, and test optimizations.

## V0.17

- Implemented the **KQMALLOC_LOCAL_CACHE_RATIO** environment variable.
- Implemented the **KQMALLOC_TINY_THRESHOLD** environment variable.
- Optimized the default value of **KQMALLOC_ALIGNMENT** in multi-threaded mode (new value: **64**).
- Optimized the default value of **KQMALLOC_HEAP_CACHE_SIZE** in multi-threaded mode (new value: **0**).
- Optimized the default value of **KQMALLOC_HEAP_CACHE_RATIO** in multi-threaded mode (new value: **0**).

## V0.16

- Implemented the **KQMALLOC_HUGE_THRESHOLD** environment variable.

## V0.15

- Implemented topology awareness based on the processor package level.
- Set the default value of **KQMALLOC_NUMA_AWARE** to **2** (NUMA awareness based on the processor package level).

## V0.14

- Enabled 64-byte cache line size for Kunpeng 920 processors.
- Implemented **nontemporal_malloc()** and **nontemporal_free()** calls for allocating long-lifetime objects.
- Slightly optimized the internal allocator structure.
- Introduced the **USE_ALLOCATOR_BY_VALUE** macro to control TLS behavior for integration with the Great Fiber library.

## V0.13

- Fixed a bug in the **free()** function that caused SIGSEGV in certain cases.
- Implemented the **KQMALLOC_NUMA_AWARE** environment variable. Setting **KQMALLOC_NUMA_AWARE=0** disables NUMA awareness of the library.

## V0.12

- Optimized the **KQMALLOC_ALIGNMENT** environment variable. The new default value is **KQMALLOC_ALIGNMENT=4** (which does not compliant with the C11 standard).
- Implemented the **KQMALLOC_C11_COMPLIANT_PROCESSES** environment variable to adapt to the new default value of **KQMALLOC_ALIGNMENT**.
  This variable is a colon-separated (:) list of process names that require C11-compliant alignment.
  For processes in this list, unless explicitly specified, the default value of **KQMALLOC_ALIGNMENT=16** will be implicitly applied.
  The default value is **KQMALLOC_C11_COMPLIANT_PROCESSES=java**.
- Improved the single-threaded performance of parallel KQMalloc in the SPEC CPU 2017 benchmark in out-of-the-box scenarios.
  (that is, the scenarios where no environment variable is used).
- Optimized the **KQMALLOC_USE_THP** environment variable: **KQMALLOC_USE_THP=-1** indicates that **madvise()** is not used to set **MADV_HUGEPAGE** or **MADV_NOHUGEPAGE**.
- (Experimental) Enabled HBM for HIP11. Implemented the experimental environment variable **KQMALLOC_USE_HBM=1** (not documented in README_EN.md and subject to change in the future).
  This variable allows memory allocation on NUMA nodes 16 to 31. The target system must have the **libnuma.so** library installed to support dynamic loading.

## V0.11

- Added full support for NUMA awareness by the allocator. In earlier versions, NUMA awareness is supported only for objects larger than 128 KB.
- Implemented the **KQMALLOC_USE_THP** environment variable to enable or disable transparent huge pages (THP). The default value is **KQMALLOC_USE_THP=1**.
- Fixed an integer overflow bug (when using **malloc**, **calloc**, or **realloc** to allocate **-1** bytes).
- Fixed a bug where the allocator was used after thread destruction (in the 638.imagick_s test case).
- Fixed the dependency of the`make bin` command on CMake, simplifying the build process on CentOS 7.
- Improved stress testing to cover more alignment methods and allocation sizes.
- Added support for GCC 14.

## V0.10

- Optimized the parallel allocator for the "producer-consumer" pattern, redesigning the caching mechanism.
- Fixed issues with the `make bin` build option on the 920 processor.
- Fixed issues with the support for the 920 processor: removed custom **memcpy** and **memset** functions and corrected assertion logic.
- Fully migrated the CI process into the code repository, including configurations for BiSheng Compiler 4 and CentOS 7.

## V0.9

- Introduced the environment variable **KQMALLOC_ALIGNMENT** with a default value of 16. The value can be 4, 8, 16, 32, 64, or 128 bytes.
- Fixed a bug in reallocating humongous objects.
- Performed fine-tuning for the FloorPlan OMP benchmark, resulting in better performance than jemalloc.
- Optimized the performance of small objects: increased the maximum number of untagged heaps to 40.
- Enabled transparent huge pages (4 MB, 8 MB, 16 MB, ..., 1 GB).
- Implemented a new humongous object allocation algorithm: replaced most uses of **munmap()** with **madvise(MADV_DONTNEED)**.
- Optimized the single-threaded mode using the environment variable **KQMALLOC_SINGLE_THREADED=1**.
- Added support for Kunpeng 920 processors.
- Started migrating from GCC 10.3.1_2024.08 to BiShengCompiler-4.0.0 due to errors encountered when compiling with **-O3** and **-Ofast** on the 920 processor using GCC.
- Implemented a cache-optimized environment variable **KQMALLOC_HEAP_CACHE_SIZE**.
- Adjusted benchmarks: touched allocated memory, estimated resident memory, added memory barriers to measure operation latency (rather than throughput).

## V0.8

- Added support for NUMA awareness (only for objects larger than 128 KB).
- Introduced the untagged heaps algorithm.
- Reduced memory consumption.
- Replaced **munmap()** of the heap memory with **madvise(MADV_DONTNEED)**.
- No longer using **MADV_FREE**.
- Added the **KQMALLOC_ALIGNMENT=4** environment variable for SPEC tests (The value 8 is also supported, and the default value is 16).
- Improved parallel testing.

## V0.7

- Released the beta version of the parallel allocator, along with test cases.

## V0.6

- Performed code alignment for exported functions to improve performance stability.

## V0.5

- Greatly improved test cases.
- Integrated feedback on code cleanliness.
- Rename the library **KQMalloc**.
- Fixed compilation warnings and code style issues.
- Added better support for the OE platform.
