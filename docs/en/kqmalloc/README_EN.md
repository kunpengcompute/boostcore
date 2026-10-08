# KQMalloc Introduction

English|[简体中文](../../zh/kqmalloc/README.md)

## Latest News

- [2026-09-30]: Released KQMalloc V0.22, which significantly improves the performance of aligned allocation functions such as aligned_alloc(), as well as valloc() and pvalloc(), for Kunpeng 930; added an aligned-object local cache threshold and two types of local cache low-watermark configurations to reduce allocation overhead.

## Project Introduction

This document applies to KQMalloc V0.22.

KQMalloc is a parallel memory allocator designed specifically for Kunpeng processors, adapted and optimized for system scenarios with 4 KB and 64 KB system page sizes and 48-bit memory virtual addresses under the Linux operating system.
The allocator runs in single-threaded mode during initialization and automatically switches to multi-threaded mode after a second thread is detected. Starting from V0.21, KQMalloc no longer provides a separate single-threaded dynamic library.
KQMalloc uses technologies such as transparent huge pages to minimize TLB misses and reduce cache fragmentation and cache aliasing issues.

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

| Resource Name | Resource Introduction |
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

Thank you for every PR from the community. We welcome your contributions to kqmalloc!
