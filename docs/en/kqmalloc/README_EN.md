# KQMalloc

## Project Introduction

KQMalloc is a memory allocator designed for Kunpeng processors and is suitable for Linux environments with 4 KB or 64 KB system page size and 48-bit virtual memory addresses.
KQMalloc uses techniques such as transparent huge pages to minimize translation lookaside buffer (TLB) misses, reduce cache fragmentation, and mitigate cache aliases.
This allocator supports both single-threaded and multi-threaded applications, significantly improving performance.

Application scenarios: memory allocation and deallocation

## Description

**Table 1** KQMalloc features

|  Feature| Description |
| ------------ | ------------ |
|  Multi-level lock-free cache fast path| Each thread has its own independent local cache. Small object allocation requires no more than 10 instructions and involves no locking throughout the process. When a miss occurs, it falls back to allocating from the Quick Cache (a lock-free cache using atomic CAS operations) and the Fast Cache (an adaptive-capacity shared cache).|
|  Lazy release and in-place resize of Transparent Huge Pages (THP)| When releasing memory, use MADV_FREE for lazy reclamation, marking pages as reclaimable but not immediately returning them, which reduces system calls. For large objects, realloc adjusts the size in place through virtual memory commit/decommit, avoiding the need to allocate new memory and copy data.|
|  Fine-grained size classes and low-fragmentation heap selection| Objects are divided into more than 100 size classes. Small objects are classified with a 4-byte step, and large objects are classified with a cache line step. During each allocation, the closest matching slot is selected.|
|  Bidirectional heap dirty page reclaiming| **Tail Clean**: When the resident memory of the heap exceeds a threshold, release excess tail pages using **MADV_DONTNEED**. <br>**Heap Compression**: Reclaim the tail virtual address space. <br>**Body Clean**: Sort and merge the freed middle regions to extend them towards adjacent free cells, aligning them to page boundaries. These aligned regions can then be uniformly released, reducing memory fragmentation.|

## Release Notes

For details about the description of the KQMalloc memory allocator of each version, see [Release Notes](./release_notes.md).

## Installation and Deployment

The installation guide of KQMalloc provides instructions for obtaining and using the dynamic library. For details, see [Installation Guide](./installation_guide.md).

## Quick Start

The quick start guide of KQMalloc provides instructions for environment variables and acceleration examples. For details, see [Quick Start](./quick_start.md).

## Document

| Name| Description|
| ------------ | ------------ |
| [Release Notes](./release_notes.md)| Provides the feature updates, resolved issues, and known restrictions of each version.|
| [Quick Start](./quick_start.md)| Describes the basic concepts, installation, usage, and performance verification of KQMalloc.|
| [Installation Guide](./installation_guide.md)| Describes the environment requirements, software package obtaining, installation procedure, and uninstallation method of KQMalloc.|
| [API Reference](./api_reference.md)| Provides the detailed descriptions and usage examples of functions such as malloc, calloc, realloc, and free.|

## Contribution Statement

We welcome your contributions to the community. If you have any questions/suggestions or want to provide feedback on feature requirements and bug reports, you can submit issues. For details, see the contribution guideline.

## Disclaimer

This code repository contributes to the KQMalloc open-source project solely for performance optimization. It strictly adheres to the coding style and methods, as well as security design of the native open-source software. Any vulnerability and security issues of the software shall be resolved by the corresponding upstream communities according to their response mechanisms. Please pay attention to the notifications and version updates released by the upstream communities. The Kunpeng computing community does not assume any responsibility for software vulnerabilities and security issues.

## License

The documents of this project are licensed under CC-BY 4.0. For details, see [LICENSE](LICENSE).
