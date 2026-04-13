
# Kunpeng Quick Memory Allocator
Memory allocation library

## Description
Parallel Kunpeng Quick Memory Allocator optimized for Kunpeng processors.
The libkqmalloc.so is a single threaded version of the library.
The libkqmallocmt.so is a multi threaded version of the library.
The libkqmallocmt.debug.so is a version of the allocator with enabled assertions, double free() checks and so on.
The libkqmallocmt.profiler.so is a version of the allocator with enabled memory profiler.

## Building generic version for Linux
There are several targets defined in Makefile to build allocator. Use
```shell
make clean build
```
to build using cmake.

## Building Kunpeng libraries for Linux
There are several "kunpeng" targets defined in Makefile to build allocator for Kunpeng processors. Use
```shell
make clean kunpeng
```
to build using Makefile directly. The result is not installable using make install.

## Installing
```shell
make install
```
If you want to change install prefix:
```shell
make install prefix=/your/directory
```

## Using
Just link against libkqmallocmt.so or preload it like:
```shell
LD_PRELOAD=/<path to KQMalloc>/lib/libkqmallocmt.so ./run_your_application
```

## Environment variables
### KQMALLOC_ALIGNMENT
This environment variable controls the alignment of object addresses.
In systems with 64-byte cache line size, the following alignment values are acceptable: 4, 8, 16, 32, 64.
In systems with 128-byte cache line size, address alignment by 128 is additionally acceptable.
If an incorrect value is set to this variable, it is ignored.
For the single-threaded mode, the default value is 4.
For the multi-threaded mode, the default value is 16.
If the threading mode is not specified explicitly, but the process name is included
in the list of C11-compliant processes KQMALLOC_C11_COMPLIANT_PROCESSES
then the default value is 16 (C11-compliant).
If the threading mode is not specified explicitly, and the process name is not listed in
the C11 compliant processes, the allocator tries to use 4-byte alignment
until the second thread appears.
When the second thread appears, the allocator rewrites the size class translation table
so that the effective alignment is 16
(but the objects already allocated by the first thread may not be aligned to 16).
Alignment to 4 bytes means the following:
objects with size less than or equal to 48 bytes are allocated at the addresses aligned to no worse than 4;
all other objects are allocated at addresses aligned to no worse than 16.
Alignment to 8 bytes means the following:
objects with size less than or equal to 48 bytes are allocated at the addresses aligned to no worse than 8;
all other objects are allocated at addresses aligned to no worse than 16.
Alignment to 16 bytes means the following:
objects with size less than or equal to 8 bytes are allocated at the addresses aligned to no worse than 8;
all other objects are allocated at addresses aligned to no worse than 16.
Alignment to 32 bytes means the following:
objects with size less than or equal to 8 bytes are allocated at the addresses aligned to no worse than 8;
objects with size less than or equal to 16 bytes are allocated at the addresses aligned to no worse than 16;
all other objects are allocated at addresses aligned to no worse than 32.
Alignment to 64 bytes means the following:
objects with size less than or equal to 8 bytes are allocated at the addresses aligned to no worse than 8;
objects with size less than or equal to 16 bytes are allocated at the addresses aligned to no worse than 16;
objects with size less than or equal to 32 bytes are allocated at the addresses aligned to no worse than 32 bytes;
all other objects are allocated at addresses aligned to no worse than 64.

- Allowed values: 4, 8, 16, 32, 64, (128).
- Default: 4 (single-threaded mode KQMALLOC_SINGLE_THREADED=1, and unspecified threading mode)
- Default: 16 (multi-threaded mode KQMALLOC_SINGLE_THREADED=0)
- Example: `export KQMALLOC_ALIGNMENT=4`

### KQMALLOC_C11_COMPLIANT_PROCESSES
This environment variable defines a list of process names, which forced to use C11 compliant pointer alignment to sizeof(long double)=16.
The default value is "java".
Additional process names can be added by separating them with a colon ":", for example, "java:jvm".
This environment variable affects the default value of the KQMALLOC_ALIGNMENT variable.
By default, KQMalloc does not use C11 compliant 16-byte alignment because
actually it is not beneficial to anyone:
old style single-threaded applications (written in 32-bit computer epoch) usually high benefit from 4-byte alignment,
and modern massive multi-threaded applications with hundreds of threads on several NUMA-nodes
high benefit from at least 64-byte alignment to avoid false cache line sharing.
By default, KQMalloc library initialized to single threaded mode until the second thread appeared,
when the second thread appeared the library re-initialized to multi-threaded mode.
There are applications that do not take care of critical objects address alignment on their own
and just crash with BUS ERROR if the allocator does not align the addresses of objects to 16 bytes
(in compliance with the C11 standard).
Well known such process is "java", this is the reason of this environment variable.

- Default: java
- Example: `export KQMALLOC_C11_COMPLIANT_PROCESSES="java8:java12:java15`

### KQMALLOC_SINGLE_THREADED
This environment variable controls whether to initialize the allocator
for a single-threaded application or for a multi-threaded one.
It is the most sensitive environment variable.
Applicable values are 0 or 1.
The default value is not set (empty).
An empty value (the environment variable is not set) indicates
that the allocator considers the application to be single-threaded
until the second thread appears. In this case the allocator will try to
use some single threaded optimizations.
The explicit value 1 indicates the guarantee from an application
to the allocator that the application will use the allocator from
only one thread or will take care of synchronization itself.
In this case the allocator can use 100% single threaded optimizations.
If the value is set to 1 but the second thread appears then the allocator rises SIG ABORT
(assertion about single threaded mode has been violated).
The explicit value 0 hints that the application is pure multi-threaded
therefore the allocator must not even try to use some single threaded
optimizations at start up.
In this case the allocator initializes into multi-threaded mode
directly without attempts to use any single threaded optimizations.
It is highly recommended to use KQMALLOC_SINGLE_THREADED explicitly,
in this case only you will get the maximum performance.

- Allowed values: 0, 1
- Default: (the default value is not set)
- Example: `export KQMALLOC_SINGLE_THREADED=0`

### KQMALLOC_SLAB_CACHE_RATIO
Tiny objects allocated at slabs.
This environment variable controls the maximum slab size in percentage that an allocator may cache to improve performance.
Applicable values are from 0 to 100.
Default value is 20. It means that to improve performance,
a fully filled slab will not be again considered suitable for new allocations until it has freed 20% of space.
Large values of this variable cause a large resident memory consumption,
but can speed up the application.
On the contrary, small values of this variable lead to economical use of resident memory,
but can slow down the application. For maximum memory savings, set this variable to 0.

- Min: 0
- Max: 100
- Default: 20
- Example: `export KQMALLOC_SLAB_CACHE_RATIO=0`

### KQMALLOC_LOCAL_CACHE_RATIO
Each thread has a local tiny and small object cache.
The maximum size of the thread local cache is defined at a compile time
and cannot be increased during program execution.
However, it can be reduced.
To control the effective size of the thread local small object cache,
KQMALLOC_LOCAL_CACHE_RATIO is used,
which takes values from 0 to 100
and indicates the percentage of the maximum cache size that is allowed to be used.
Default value is 100 percent.

- Min: 0
- Max: 100
- Default: 100
- Example: `export KQMALLOC_LOCAL_CACHE_RATIO=40`

### KQMALLOC_USE_THP
This environment variable controls whether the allocator should use transparent huge pages or not.
Applicable values are: -1, 0, 1.
The default value is 1.
The value 1 means that the allocator will use transparent huge pages by explicit calls madvise() with flag MADV_HUGEPAGE.
The value 0 means that the allocator will not use transparent huge pages by explicit calls madvise() with flag MADV_NOHUGEPAGE.
So, this variable allows to disable transparent huge page usage even if they are enabled in the system as "always"
(/sys/kernel/mm/transparent_hugepage/enabled).
The value -1 means that the allocator will not call madvise() at all, in this case transparent huge page usage
depends on system preferences. There is no clear answer to the question whether it is useful to use transparent huge pages.
As a rule, if all threads in an application are pinned to its CPU (or at least to its NUMA-nodes),
then using THP may be useful. Otherwise, if threads of an application are migrating between NUMA-nodes
and have ill-advised memory access, the use of THP may not be useful.
To figure out if THP is useful for your application, you may need to conduct an experiment.
The KQMALLOC_USE_THP is valid only to the systems with 4096 bytes system page size and 2 MB transparent huge page size.

- Default: 1
- Example: `export KQMALLOC_USE_THP=0`

### KQMALLOC_NUMA_AWARE
This environment variable controls whether the allocator should take NUMA-nodes into account.
Applicable values are 0, 1, 2.
The value 0 means the NUMA awareness is disabled.
The value 1 means the NUMA awareness is enabled with NUMA-node level granulation.
The value 2 means the NUMA awareness is enabled with processor's socket (package) level granulation.
In the multi-threaded mode, the default value is 2 (in general, it is a tradeoff between performance and memory consumption).
In the single-threaded mode the value is always 0 and cannot be changed.
Ideally, all application threads should be pinned to the corresponding CPUs before the first allocator usage
and never migrate between NUMA-nodes (if enabled NUMA-node granulation)
or between processor's sockets (if enabled processor's socket level granulation).
The allocator is able to track the migration of threads between NUMA-nodes and adapt to the new configuration,
but this adaptation may be slow.
If application threads migrate between NUMA-nodes after allocating objects on the previous NUMA-node,
this may lead to performance degradation of such application.
In particular, for the unoptimized applications where threads often migrate between NUMA nodes,
it may be useful to completely disable the allocator's NUMA-awareness (at least it reduces memory consumption).
Also, it may be useful to disable NUMA-awareness to reduce memory consumption for memory-footprint-critical applications,
but it may lead to sensitive performance degradation.

- Default: 0 (single-threaded mode)
- Default: 2 (multi-threaded mode)
- Example: `KQMALLOC_NUMA_AWARE=0`

### KQMALLOC_TINY_THRESHOLD
Tiny object size threshold in bytes.
For the single-threaded mode, the default value is (64 * 1024 + CACHE_LINE_SIZE) bytes.
For the multi-threaded mode, the default value is 512 bytes.
This environment variable is high performance sensitive especially
for applications with massive cross-socket (cross-NUMA) communication.
The motivation is that for performance reasons tiny objects are placed
in the local cache of the thread that freed them,
while non-tiny objects, which are created in another NUMA node,
are immediately returned back to their NUMA node cache or heap.
It directly affects tradeoff between memory consumption and performance:
on the massive multi-threaded applications with hundreds of threads
on several NUMA-nodes the lowest values may increase performance
and the highest values may decrease performance.

- Default: 65536 + CACHE_LINE_SIZE (single-threaded mode)
- Default: 512 (multi-threaded mode)
- Example: `KQMALLOC_TINY_THRESHOLD=512`

### KQMALLOC_PROFILER_SECONDS
Valid only if the memory profiler is enabled.
Profiling interval in seconds. Default is 60.0 seconds.
Must be greater than zero. Must be less than or equal to 60*60*24 = 86'400 seconds. 
Regardless of the set value, the profiler file is written for the first time after ~2 seconds.
To significantly reduce the size of the profile files, you can set this variable to, for example, KQMALLOC_PROFILER_SECONDS=1000 seconds.

- Min: 0+
- Max: 86400.0
- Default: 60.0
- Example: `KQMALLOC_PROFILER_SECONDS=1000`

### KQMALLOC_PROFILER_EXPONENT
Valid only if the memory profiler is enabled.
As the program runs, the profiling interval increases exponentially.
The default profiling interval value is 0.05 of total elapsed time (i.e. 5%), but not less than KQMALLOC_PROFILER_SECONDS.
Must be greater than zero. Must be less than or equal to 0.25.

- Min: 0+
- Max: 0.25
- Default: 0.05
- Example: `KQMALLOC_PROFILER_EXPONENT=0.10`

### KQMALLOC_HEAP_CACHE_SIZE
This environment variable controls the maximum size in megabytes for the margin from the heap head that can be cached by the allocator
(but not greater than KQMALLOC_HEAP_CACHE_RATIO of heap peak size).
This memory is considered as the free-memory from the application point of view, but this memory
is not returned back to the system due to performance considerations.
Default value is 512 megabytes per each heap in the single-threaded mode and 0 megabytes per each heap in the multi-threaded mode.
Large values of this variable cause a large resident memory consumption,
but can speed up the application. On the contrary, small values of this variable lead to economical use of resident memory,
but can slow down the application.
For maximum memory savings, set this variable to 0.

- Default: 512 (single-threaded mode)
- Default: 0 (multi-threaded mode)
- Example: `KQMALLOC_HEAP_CACHE_SIZE=0`

### KQMALLOC_HEAP_CACHE_RATIO
This environment variable controls the maximum size in percentage of heap peak size
for the margin from the heap head that can be cached by the allocator, but not greater than KQMALLOC_HEAP_CACHE_SIZE megabytes.
Applicable values are from 0 to 100.
Default value is 75 percent in the single-threaded mode and 100 percent in the multi-threaded mode.
Large values of this variable cause a large resident memory consumption,
but can speed up the application.
On the contrary, small values of this variable lead to economical use of resident memory,
but can slow down the application.
For maximum memory savings, set this variable to 0.

- Default: 75 (single-threaded mode)
- Default: 100 (multi-threaded mode)
- Example: `KQMALLOC_HEAP_CACHE_RATIO=0`

### KQMALLOC_DIRTY_THRESHOLD
This variable sets the upper limit on the number of so-called dirty pages in each heap in megabytes
(the heap head margin controlled by KQMALLOC_HEAP_CACHE_SIZE is not taken into account).
The default value is 256 megabytes. The number of dirty pages in each heap is adaptively set to
no more than one standard deviation from the average memory usage, and no more than KQMALLOC_DIRTY_THRESHOLD megabytes.
Small values of KQMALLOC_DIRTY_THRESHOLD lead to slightly less memory usage, but significantly degrade performance.
Large values of KQMALLOC_DIRTY_THRESHOLD lead to slightly more memory usage, but significantly improve performance.
It is safe to set this variable to a large value if the standard deviation of the average memory consumption is small.

- Default: 256
- Example: `KQMALLOC_DIRTY_THRESHOLD=1024`
