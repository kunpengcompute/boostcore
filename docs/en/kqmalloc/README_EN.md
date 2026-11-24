# KQMalloc Introduction

<!-- md-trans-meta sourceCommit=34063ca090d649978df53902795da9b82ec8a75a translatedAt=2026-11-18T03:11:28.074Z pushedAt=2026-11-19T07:14:11.850Z -->

[简体中文](../../zh/kqmalloc/README.md)|English

This document introduces the KQMalloc parallel memory allocator, covering feature introduction, release notes, installation and deployment, and quick start. It is intended for users who allocate and free memory on Linux.

## Latest News

- [2026-09-30]: Released KQMalloc V0.22, which significantly improves the performance of aligned allocation functions such as aligned_alloc(), as well as valloc() and pvalloc(), for Kunpeng processors; added an aligned-object local cache threshold and two types of local cache low-watermark configurations to reduce allocation overhead.
- [2026-06-30]: Released KQMalloc V0.20, which significantly improves single-threaded and GREAT memory allocation benchmark performance, optimizes high-frequency allocation of large objects up to 64 MB, and reduces memory usage in producer-consumer scenarios; refactored the heap tag buffer, added hierarchical heap caches and Quick Cache threshold configuration, and optimized the dirty page cache threshold configuration.
- [2026-03-30]: Released KQMalloc V0.19, which significantly improves parallel processing capability and supports thousands of threads and TB-level memory; added support for fork(), malloc_trim(), and aligned allocation up to 128 MB; refactored huge object allocation, improved dirty page thresholds, memory analysis, and debugging capabilities, and unified the binary versions for 64 KB and 2 MB page sizes.

## Project Introduction

This document applies to KQMalloc V0.22.

The Kunpeng high-performance memory library KQMalloc (Kunpeng Quick Malloc) is a parallel memory allocator designed specifically for Kunpeng processors, adapted and optimized for Linux operating system scenarios with 4 KB and 64 KB system page sizes and 48-bit virtual memory addresses.
The allocator runs in single-threaded mode during initialization and automatically switches to multi-threaded mode after detecting a second thread. Starting from V0.21, KQMalloc no longer releases a standalone single-threaded dynamic library.
KQMalloc uses technologies such as transparent huge pages to minimize TLB misses, reduce cache fragmentation, and mitigate cache aliasing issues.

Applicable scenarios: memory allocation and deallocation

## Feature Introduction

**Table 1** KQMalloc features

| Feature | Description |
| ------------ | ------------ |
| Multi-level lock-free cache fast path | Each thread has its own independent local cache. Small-object allocation requires fewer than 10 instructions and no locking throughout the entire process. On a cache miss, allocation falls back level by level to the Quick Cache (a lock-free cache using CAS atomic operations) and the Fast Cache (a shared cache with adaptive capacity). |
| In-place resize of large objects | Large-object realloc adjusts the size in place through virtual memory commit/decommit, avoiding the allocation of new memory and subsequent copying. |
| Fine-grained size classes and low-fragmentation heap selection | Objects are classified into more than 100 size classes. Small objects use a 4-byte step and large objects use a cache-line step, so each allocation matches the closest slot. |
| Bidirectional heap dirty-page reclamation | Tail clean: Release the excess pages at the tail using MADV_DONTNEED when heap residency exceeds the threshold.<br>Compression: Reclaim the virtual address space at the tail.<br>Body clean: Sort and merge the freed middle regions to extend them towards adjacent free cells, aligning them to page boundaries before uniformly releasing them. |

## Release Notes

The release notes for the KQMalloc memory allocator provide detailed descriptions of each KQMalloc version. For details, see [Release Notes](./release_notes.md).

## Installation and Deployment

The installation guide for the KQMalloc memory allocator, including how to obtain the dynamic libraries and how to use them. For details, see [Installation Guide](./installation_guide.md).

## Quick Start

The quick start guide for the KQMalloc memory allocator, including environment variables and acceleration examples. For details, see [Quick Start](./quick_start.md).

## Learning Documents

| Document Name | Description |
| ------------ | ------------ |
| [Release Notes](./release_notes.md) | Records feature updates, bug fixes, and known limitations of each version. |
| [Quick Start](./quick_start.md) | Introduces the basic concepts, installation and deployment, usage, and performance verification of KQMalloc to help users get started quickly. |
| [Installation Guide](./installation_guide.md) | Describes in detail the environment requirements, acquisition of precompiled dynamic libraries, usage, verification, and removal methods of KQMalloc. |
| [API Reference](./api_reference.md) | Provides detailed descriptions and usage examples of functions such as malloc, calloc, realloc, and free. |

## Issue Feedback

If you encounter any problems or have suggestions during use, or need to report feature requirements and bugs, you can submit [issues](https://gitcode.com/boostkit/boostcore/issues/create/choose) to contact us.

## Disclaimer

This directory provides the supporting documentation for the KQMalloc precompiled dynamic libraries. Actual functionality, compatibility, and performance are subject to the delivered version and runtime environment in use. Please closely follow version updates and security notices, and complete operating system security hardening in accordance with your business scenarios.

## License

The documents of this project are licensed under CC-BY 4.0. For details, see [LICENSE](LICENSE).

## Acknowledgments

Thank you for every PR from the community. We welcome your contributions to KQMalloc!
