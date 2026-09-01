# BoostCore Basic Acceleration

## Latest News

- \[2026-03-30\]: Added protobuf and sonic-cpp to the system library.

- \[2025-12-30\]: Added KACC_JSON to the system library and KACC_cropto to the encryption and decryption library.

- \[2025-06-30\]: Added the Kunpeng Unified Acceleration Framework (KUAF).

- \[2024-02-22\]: Released Kunpeng Accelerator Engine (KAE) 2.0.

## Project Introduction

BoostCore Basic Acceleration provides acceleration software packages for base software (OS and system software) in mainstream application scenarios. For example, it includes KAE and kpglibc from KSL.

## Feature Overview

This section briefly describes all basic acceleration features included in the BoostCore Basic Acceleration project and provides the repository addresses for these features.

### KAE

| Sub-library | Description |
| :--- | :--- |
| [KAE](https://gitcode.com/boostkit/KAE) | KAE is a hardware acceleration solution based on the Kunpeng processors. It includes the KAE encryption and decryption and the KAE decompression modules, which are used to accelerate SSL/TLS applications and data compression, respectively. Therefore, it can significantly reduce processor consumption and improve processor efficiency. In addition, KAE shields the internal implementation details from the application layer. You can quickly migrate services by using standard OpenSSL and zlib interfaces. |
| [KUAF](https://gitcode.com/boostkit/KUAF) | KUAF is a hardware acceleration scheduling framework. |

<!--
### HAAS

| Sub-library| Description|
| :--- | :--- |
| HASS (to be open-sourced)| Hardware Affinitive Acceleration Service (HAAS) is a hardware affinity acceleration access framework developed by Kunpeng, designed for big data and search/recommendation AI applications. HAAS extracts fundamental operators from the Kunpeng BoostKit Big Data components, specifically OmniOperatorJIT and OmniStream, and uses SVE and NEON instructions for vectorization optimization. This process helps create Kunpeng instruction affinity and primitive library to enhance performance. HAAS is applicable to the new Kunpeng 920 processor model, supporting 128-bit wide NEON instructions and 256-bit wide SVE instructions. |
-->

### KZL (Compression Library)

| Sub-library | Description |
| :--- | :--- |
| [zstd](https://gitcode.com/boostkit/zstd) | Patch repository based on upstream community source code with Kunpeng optimization and adaptation. |
| [Snappy](https://gitcode.com/boostkit/snappy) | Patch repository based on upstream community source code with Kunpeng optimization and adaptation. |
| [LZ4](https://gitcode.com/boostkit/lz4) | Patch repository based on upstream community source code with Kunpeng optimization and adaptation. |
| [zlib](https://gitcode.com/boostkit/zlib) | Patch repository based on upstream community source code with Kunpeng optimization and adaptation. |
<!--| KZstar (to be open-sourced)| Multi-threaded compression library that complies with the zstd protocol.|-->
<!--| Gzip | Align right|-->

<!--
### KCL (Encryption & Decryption Library)

| Sub-library | Description |
| :--- | :--- |
| KACC_crypto | Align right |

-->

### KSL (System Library)

| Sub-library | Description |
| :--- | :--- |
| [kpglibc](https://gitcode.com/boostkit/kpglibc) | kpglibc is a repository contributed by Kunpeng to the glibc open-source community. It leverages a vectorized instruction set to enhance the performance of string, memory, and time operation functions for the new Kunpeng 920 processor model and Kunpeng 950 processor. The version optimized with SVE is recommended for better performance. The function interfaces remain consistent with the open-source glibc, and internal input parameter validation is not performed. The validity of input parameters must be ensured by the service that calls the interfaces. |
| [protobuf](https://gitcode.com/boostkit/protobuf) | Patch repository based on upstream community source code with Kunpeng optimization and adaptation. |
| [sonic-cpp](https://gitcode.com/boostkit/sonic-cpp) | Patch repository based on upstream community source code with Kunpeng optimization and adaptation. |
| [KQMalloc](./docs/en/kqmalloc/README_EN.md) (provides precompiled dynamic libraries and supporting documentation) | KQMalloc is a parallel memory allocator designed specifically for Kunpeng processors, adapted to system scenarios with 4 KB and 64 KB system pages and 48-bit virtual memory addresses under the Linux operating system. The allocator runs in single-threaded mode during initialization and automatically switches to multi-threaded mode after detecting a second thread. KQMalloc uses technologies such as transparent huge pages to reduce TLB misses, cache fragmentation, and cache aliasing issues. |
| [AVX2KI](https://gitcode.com/boostkit/avx2ki) | A library for porting AVX intrinsics to Kunpeng instructions. |

<!--
| HTL (to be open-sourced) | Patch repository based on upstream community source code with Kunpeng optimization and adaptation. |
| KSL_ASN1 | Content |
| BiSheng JDK acceleration library | Content |
| KACC_JSON | Content |
| HAF | Content |
| KVSIP | Content |
-->

### KMML (Media Library)

| Sub-library | Description |
| :--- | :--- |
| [HMPP Documentation](https://gitcode.com/boostkit/boostmedia/blob/master/docs/hmpp/README_EN.md) (Only the documentation repository is provided. The code repository is to be open-sourced.) | HMPP (Hyper Media Performance Primitives) is an acceleration library developed on the Kunpeng processor hardware platform. It covers image processing, color conversion, filtering, transform, and geometry, and provides high-performance function interfaces for computer vision operations, vector operations, statistics, signal filtering, signal transform, and fixed-precision operations through vector instruction sets. |
| [KVCL Documentation](https://gitcode.com/boostkit/boostmedia/blob/master/docs/kvcl/README_EN.md) (Only the documentation repository is provided. The code repository is to be open-sourced.) | It optimizes common basic operators for video encoding by using vectorized instructions, and fully utilizes vectorization in combination with hardware features to improve operator performance. |

### KAIL (AI Library)

| Sub-library | Description |
| :--- | :--- |
| [Kunpeng AI Operator Library Documentation](https://gitcode.com/boostkit/boostsra/blob/master/docs/zh/kdnn/README.md) (Only the documentation repository is provided. The code repository is to be open-sourced.) | Based on the microarchitecture features of the Kunpeng processor, Kunpeng Deep Neural Network Library (KDNN) enhances the performance of core DNN operators through vectorization, assembly-level optimizations, and algorithmic improvements. |

### KSAL (Storage Library)

| Sub-library | Description |
| :--- | :--- |
| [ISA-L](https://gitcode.com/boostkit/isa-l) | ISA-L is an open-source high-performance storage algorithm library. On the basis of the open-source version, we further optimize erasure coding and cyclic redundancy check (CRC) by using NEON/SVE instructions, memory prefetch, and other means. |
| [KAE-enabled SPDK](https://gitcode.com/boostkit/spdk) | The bdev device of SPDK serves as a virtual device layer that connects to various underlying device types (virtual and physical devices). By enabling compression and encryption/decryption in bdev devices, it can support all SPDK devices. |

<!--
### Others

| Sub-library | Description |
| :--- | :--- |
| [Brotli4j](https://gitcode.com/boostkit/Brotli4j) | Patch repository based on upstream community source code with Kunpeng optimization and adaptation. |
| [folly](https://gitcode.com/boostkit/folly) | This project is an optimization repository for the folly asynchronous network I/O framework, focusing on and optimizing the use of io_uring. This optimization solution adopts a hybrid mode, in which read operations use the multishot mode of io_uring to reduce system calls, while write operations fall back to the native send system call, which is better at handling continuous writes, to avoid the latency that io_uring may introduce in order-preserving scenarios. |
| [gosericesopt](https://gitcode.com/boostkit/msgp) | Patch repository based on upstream community source code with Kunpeng optimization and adaptation. |
| [gozstd](https://gitcode.com/boostkit/msgp) | Patch repository based on upstream community source code with Kunpeng optimization and adaptation. |
| [json-iterato](https://gitcode.com/boostkit/msgp) | Patch repository based on upstream community source code with Kunpeng optimization and adaptation. |
| [gosericesopt](https://gitcode.com/boostkit/msgp) | Patch repository based on upstream community source code with Kunpeng optimization and adaptation. |
| [KDADK-APPID](https://gitcode.com/boostkit/KDADK-APPID) | The network flow classification library within the KDADK. It provides core capabilities including flow aggregation, protocol detection, feature computation, an AI engine, and a training toolkit. |
| [KDADK-TOOLS](https://gitcode.com/boostkit/KDADK-TOOLS) | A toolkit for network traffic analysis and real-time encrypted traffic detection, designed to address the limitations of traditional DPI solutions. It provides example implementations of feature extraction and inference interfaces, along with a complete toolchain including traffic capture, feature filtering, model training, and adaptive labeling. All components are optimized for Kunpeng. |
| [kitex](https://gitcode.com/boostkit/msgp) | Patch repository based on upstream community source code with Kunpeng optimization and adaptation. |
| [msgp](https://gitcode.com/boostkit/msgp) | Patch repository based on upstream community source code with Kunpeng optimization and adaptation. |
| [suricata](https://gitcode.com/boostkit/suricata) | Patch repository based on upstream community source code with Kunpeng optimization and adaptation. |

### Public Configuration Repository

| Sub-library | Description |
| :--- | :--- |
| [AcclibBenchmark](https://gitcode.com/boostkit/AcclibBenchmark) | Stores performance test scripts for acceleration libraries. |
| [AcclibSkills](https://gitcode.com/boostkit/AcclibSkills) | Shares skills that work well during the development and debugging of acceleration libraries. |

-->

## About the Community

This part provides the introduction and guidance to the public modules, such as the community governance architecture, SIG operations regulations, contribution, email subscription, and social media contact information.

## Contributions, Suggestions, and Discussions

We welcome your contributions to the community. If you have any questions/suggestions or want to provide feedback on feature requirements and bug reports, you can submit [issues](https://gitcode.com/boostkit/boostcore/issues/create/choose). For details, see the [contribution guideline](https://gitcode.com/boostkit/community/blob/master/docs/contributor/contributing.md). You are also welcome to share insights in the [Discussions](https://gitcode.com/boostkit/community/discussions). Thank you for your support.

## LICENSE

This project is licensed under Apache License 2.0. For details, see [LICENSE](https://gitcode.com/boostkit/cloud-virtual/blob/master/LICENSE).

The documents of this project are licensed under CC-BY 4.0. For details, see [LICENSE](docs/en/kqmalloc/LICENSE).
