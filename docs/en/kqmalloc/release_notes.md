# Release Notes

## V0.22

- Achieved a significant improvement in the performance of the aligned_alloc(), memalign(), posix_memalign(), valloc(), and pvalloc() functions.

- Added the KQMALLOC_ALIGNED_LOCAL_THRESHOLD, KQMALLOC_LOCAL_CACHE_LOW, and KQMALLOC_ALIGNED_LOCAL_CACHE_LOW environment variables.

## V0.21

- The standalone single-threaded dynamic library is no longer provided.

- Implemented the pointer tagging feature.

- Adjusted the binary library naming: **libkqmalloc.so** is now the parallel version of KQMalloc; **libkqmalloc.ptag.so** is the parallel version of KQMalloc with the pointer tagging feature enabled.

- Added the KQMALLOC_EXCLUDE, KQMALLOC_LOCAL_CACHE_MIN, KQMALLOC_LOCAL_CACHE_MAX, and KQMALLOC_LOCAL_THRESHOLD environment variables.

- KQMALLOC_LOCAL_CACHE_RATIO can be used not only to reduce the local cache, but also to increase it.

- Removed the KQMALLOC_SINGLE_THREADED environment variable.

- Implemented the free_sized(), free_aligned_sized(), valloc(), and pvalloc() standard functions.

- Added support for 512 MB transparent huge pages.

- Added a round-robin mode to KQMALLOC_NUMA_AWARE.

## V0.20

- Significantly improved single-threaded library performance.

- Redesigned the heap tag buffer: the original three-dimensional array [NUMA node][rank][storage unit] was changed to a one-dimensional array [offset + storage unit].

- Optimized the high-frequency allocation logic for large objects up to 64 MB, adapting to MySQL-like workloads.

- Significantly reduced the memory footprint of the producer-consumer benchmark without compromising performance.

- Added new environment variables: KQMALLOC_TINY_HEAP_CACHE_SIZE, KQMALLOC_SMALL_HEAP_CACHE_SIZE, KQMALLOC_MIDDLE_HEAP_CACHE_SIZE, and KQMALLOC_QUICK_CACHE_THRESHOLD.

- Changed the meaning of KQMALLOC_DIRTY_THRESHOLD from the upper limit on the number of dirty pages to the upper limit on the dirty page cache size, in megabytes. The default value was changed from 256 to 1024, and the minimum effective value is 2.

- Achieved a significant improvement in the GREAT memory allocation benchmark score.

## V0.19

- Significantly improved parallel processing capabilities, capable of managing thousands of threads and TB-level memory.

- Implemented fork() support based on the pthread_atfork() framework.

- Implemented the malloc_trim() function.

- Redesigned the aligned_alloc() implementation to support alignment granularity up to 128 MB.

- Implemented the KQMALLOC_DIRTY_THRESHOLD, KQMALLOC_PROFILER_SECONDS, and KQMALLOC_PROFILER_EXPONENT environment variables.

- Redesigned the huge object allocation implementation.

- Merged the binary versions of the library with 64 KB and 2 MB page sizes into a single binary version.

- Removed experimental HBM support.

- Optimized the default value of KQMALLOC_ALIGNMENT in multi-threaded mode (the new value is restored to 16).

- Re-enabled KQMALLOC_HEAP_CACHE_SIZE and KQMALLOC_HEAP_CACHE_RATIO.

- Fixed the issue of memory growth caused by allocating new slabs/blocks instead of waiting for locked resources in oversubscription mode.

- Fixed the issue where the first call to free() (without a prior call to malloc()) could use an uninitialized thread-local allocator.

- Improved the embedded memory profiler.

- Improved the debug version of the library (added duplicate free() checking and memory protection mechanisms).

- Reduced thread-local storage (TLS) memory consumption, so the slim allocator is no longer needed.

- Removed the slim allocator.

- The release package contains a total of 4 binary libraries: **libkqmalloc.so**, **libkqmallocmt.so**, **libkqmallocmt.debug.so**, and **libkqmallocmt.profiler.so**.

- Improved the malloc benchmark.

- Removed the KQMALLOC_HUGE_THRESHOLD and KQMALLOC_USE_HBM environment variables.

## V0.18

- Optimized support for Kunpeng 920 processors.

- Added a slim allocator for apps with small stack sizes.

- Performed multiple optimizations in memory consumption and performance.

- Removed the undocumented KQMALLOC_HEAP_CACHE_SIZE and KQMALLOC_HEAP_CACHE_RATIO.

- Fixed a bug related to relaxed memory ordering synchronization.

- Fixed a bug where the allocator was used in the context of a terminated thread.

- Added support for BiSheng Compiler 5.

- Added more diagnostics, benchmarks, and test optimizations.

## V0.17

- Implemented the KQMALLOC_LOCAL_CACHE_RATIO environment variable.

- Implemented the KQMALLOC_TINY_THRESHOLD environment variable.

- Optimized the default value of KQMALLOC_ALIGNMENT in multi-threaded mode (new value is 64).

- Optimized the default value of KQMALLOC_HEAP_CACHE_SIZE in multi-threaded mode (new value is 0).

- Optimized the default value of KQMALLOC_HEAP_CACHE_RATIO in multi-threaded mode (new value is 0).

## V0.16

- Implemented the KQMALLOC_HUGE_THRESHOLD environment variable.

## V0.15

- Implemented topology awareness at processor package-level granularity.

- Set the default value of KQMALLOC_NUMA_AWARE to 2 (NUMA awareness based on processor package-level granularity).

## V0.14

- Enabled 64-byte cache line size for Kunpeng 920 processors.

- Implemented nontemporal_malloc() and nontemporal_free() calls for allocating long-lived objects.

- Slightly optimized the internal allocator structure.

- Introduced the USE_ALLOCATOR_BY_VALUE macro to control TLS behavior for integration with the Great Fiber library.

## V0.13

- Fixed a bug in the free() function that caused SIGSEGV under certain circumstances.

- Implemented the KQMALLOC_NUMA_AWARE environment variable. Setting KQMALLOC_NUMA_AWARE=0 disables the library's NUMA awareness capability.

## V0.12

- Optimized the KQMALLOC_ALIGNMENT environment variable: the new default value is KQMALLOC_ALIGNMENT=4 (not compliant with the C11 standard).

- Implemented the KQMALLOC_C11_COMPLIANT_PROCESSES environment variable to accommodate the new default value of KQMALLOC_ALIGNMENT.

  This variable is a colon-separated (":") list of process names, and the processes in the list need to comply with the C11 standard alignment. For processes in this list, the default value KQMALLOC_ALIGNMENT=16 is implicitly applied unless explicitly specified. The default value is KQMALLOC_C11_COMPLIANT_PROCESSES=java.

- Improved the single-threaded performance of parallel KQMalloc in the SPEC CPU 2017 benchmark in the out-of-the-box scenario.

  (That is, the "no environment variable" usage scenario).

- Optimized the KQMALLOC_USE_THP environment variable: KQMALLOC_USE_THP=-1 indicates that madvise() is not used to set MADV_HUGEPAGE or MADV_NOHUGEPAGE.

- (Experimental) Implemented the experimental environment variable KQMALLOC_USE_HBM=1 (not documented in README.md and subject to change in the future).

  This variable allows memory allocation on NUMA nodes 16-31. The target system must have the **libnuma.so** library installed to support dynamic loading.

## V0.11

- Added full support for NUMA awareness by the allocator. In earlier versions, NUMA awareness is supported only for objects larger than 128 KB.

- Implemented the KQMALLOC_USE_THP environment variable to disable/enable transparent huge pages (THP). The default value is KQMALLOC_USE_THP=1.

- Fixed an integer overflow bug (allocating "-1" bytes with malloc, calloc, or realloc).

- Fixed a bug where the allocator was used after thread destruction (638.imagick_s test case).

- Improved the stress test to cover more alignment modes and allocation sizes.

- Added support for GCC 14.

## V0.10

- Optimized the parallel allocator for the "producer-consumer" pattern and redesigned the caching mechanism.

- Fixed issues with the support for the 920 processor: removed custom memcpy and memset functions and corrected assertion logic.

## V0.9

- Introduced the environment variable KQMALLOC_ALIGNMENT, with a default value of 16. Alignment granularities of 4, 8, 16, 32, 64, and 128 bytes are supported.

- Fixed a bug in reallocation of huge objects.

- Performed fine-tuning for the FloorPlan OMP benchmark, resulting in better performance than jemalloc.

- Optimized the performance of tiny and small objects: increased the maximum number of untagged heaps to 40.

- Enabled transparent huge pages (4 MB, 8 MB, 16 MB, ..., 1 GB).

- Implemented a new huge object allocation algorithm: replaced most munmap() calls with madvise(MADV_DONTNEED).

- Optimized single-threaded mode via the environment variable KQMALLOC_SINGLE_THREADED=1.

- Added support for a new Kunpeng processor model.

- Implemented the environment variable KQMALLOC_HEAP_CACHE_SIZE with cache optimization.

- Adjusted the benchmark to touch allocated memory, estimate resident memory, and add memory barriers to measure operation latency (rather than throughput).

## V0.8

- Added support for NUMA awareness (only for objects larger than 128 KB).

- Introduced the untagged heaps algorithm.

- Reduced memory consumption.

- Replaced munmap() of heap memory with madvise(MADV_DONTNEED).

- MADV_FREE is no longer used.

- Added the KQMALLOC_ALIGNMENT=4 environment variable for SPEC tests (8 is also supported, with a default value of 16).

- Improved parallel tests.

## V0.7

- Released the beta version of the parallel allocator, with test cases.

## V0.6

- Performed code alignment for exported functions to improve performance stability.

## V0.5

- Significantly improved the test cases.

- Incorporated code cleanliness feedback.

- Renamed to KQMalloc.

- Added better support for the OE platform.
