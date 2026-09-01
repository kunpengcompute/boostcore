# Introduction<a name="EN-US_TOPIC_0000002552340101"></a>

KQMalloc (Kunpeng Quick Malloc) is a parallel memory allocator designed specifically for Kunpeng processors. The allocator uses single-threaded mode during initialization and automatically switches to multi-threaded mode after detecting a second thread. KQMalloc improves app performance by reducing internal cache usage and cache misses. Starting from V0.21, KQMalloc no longer provides a standalone single-threaded dynamic library, and instead provides **libkqmalloc.ptag.so** with the pointer tagging feature enabled, as well as **libkqmalloc.so** with this feature disabled.

Application scenarios: memory allocation and deallocation

## Change History

|Document Version|Date|Description|
|--|--|--|
|01|2026-09-30|This is the third official release.|
