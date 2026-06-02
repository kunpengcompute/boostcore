# BoostCore Basic Acceleration

## Latest Updates

- \[2026-03-30\]: Added **protobuf** and **sonic-cpp** to the system library.
<!-- \[2025-12-30\]: Added **KACC_JSON** to the system library and **KACC_cropto** to the encryption and decryption library.-->
- \[2025-06-30\]: Added the Kunpeng Unified Acceleration Framework (KUAF).
- \[2024-02-22\]: Released Kunpeng Accelerator Engine (KAE) 2.0.

## Project Introduction

BoostCore Basic Acceleration provides acceleration software packages for base software (OS and system software) in mainstream application scenarios. For example, it includes KAE and kpglibc from KSL.

## Feature Overview

This section briefly describes all basic acceleration features included in the BoostCore Basic Acceleration project and provides the repository addresses for these features.

### Kunpeng Accelerator Engine (KAE)

| Sub-library| Description|
| :--- | :--- |
| [KAE](https://gitcode.com/boostkit/KAE) | Kunpeng Accelerator Engine (KAE) is a hardware acceleration solution based on the Kunpeng processor. It includes the KAE encryption and decryption and the KAE decompression modules, which are used to accelerate SSL/TLS applications and data compression, respectively. Therefore, it can significantly reduce processor consumption and improve processor efficiency. In addition, KAE shields the internal implementation details from the application layer. You can quickly migrate services by using standard OpenSSL and zlib interfaces.|
| [KUAF](https://gitcode.com/boostkit/KUAF) | Kunpeng Unified Acceleration Framework (KUAF) is a hardware acceleration scheduling framework.|

<!--
### HAAS

| Sub-library| Description|
| :--- | :--- |
| HASS (to be open-sourced)| Hardware Affinitive Acceleration Service (HAAS) is a hardware affinity acceleration access framework independently developed by Kunpeng, designed for big data and search/recommendation AI applications. HAAS extracts fundamental operators from the Kunpeng BoostKit Big Data components, specifically OmniOperatorJIT and OmniStream, and uses SVE and NEON instructions for vectorization optimization. This process helps create a Kunpeng instruction set and primitive library to enhance performance. HAAS is applicable to the new Kunpeng 920 processor model, supporting 128-bit wide NEON instructions and 256-bit wide SVE instructions.|
-->

### KZL (Compression Library)

| Sub-library| Description|
| :--- | :--- |
| [zstd](https://gitcode.com/boostkit/zstd) | A patch repository based on upstream community source code for Kunpeng optimization and adaptation.|
| [Snappy](https://gitcode.com/boostkit/snappy) | A patch repository based on upstream community source code for Kunpeng optimization and adaptation.|
| [LZ4](https://gitcode.com/boostkit/lz4) | A patch repository based on upstream community source code for Kunpeng optimization and adaptation.|
| [zlib](https://gitcode.com/boostkit/zlib) | A patch repository based on upstream community source code for Kunpeng optimization and adaptation.|
<!--| Kzstar (to be open-sourced)| Multi-thread compression library that complies with the ZSTD protocol.|-->
<!--| Gzip | Align right|-->

<!--
### KCL (Encryption & Decryption Library)

| Sub-library| Description|
| :--- | :--- |
| KACC_crypto | Align right|
-->

### KSL (System Library)

| Sub-library| Description|
| :--- | :--- |
|[kpglibc](https://gitcode.com/boostkit/kpglibc) | kpglibc is a repository contributed by Huawei Kunpeng to the glibc open-source community. It leverages a vectorized instruction set to enhance the performance of string, memory, and time operation functions for the new Kunpeng 920 processor model and Kunpeng 950 processor. The version optimized with SVE is recommended for better performance. The function interfaces remain consistent with the open-source glibc, and internal input parameter validation is not performed. The validity of input parameters must be ensured by the service that calls the interfaces.|
| [protobuf](https://gitcode.com/boostkit/protobuf) | A patch repository based on upstream community source code for Kunpeng optimization and adaptation.|
| [sonic-cpp](https://gitcode.com/boostkit/sonic-cpp) | A patch repository based on upstream community source code for Kunpeng optimization and adaptation.|
| [KQMalloc](./docs/en/kqmalloc/README_EN.md) (Only the documentation repository path is provided. The code repository is to be open-sourced.)| KQMalloc is a memory allocator designed for Kunpeng 920 series processors and is suitable for Linux environments with 4 KB or 64 KB system page size and 48-bit virtual memory addresses. KQMalloc uses techniques such as transparent huge pages to minimize translation lookaside buffer (TLB) misses, reduce cache fragmentation, and mitigate cache aliases. This allocator supports both single-threaded and multi-threaded applications, significantly improving performance.|
| [AVX2KI](https://gitcode.com/boostkit/avx2ki) | A library for porting AVX intrinsics to Kunpeng instructions.|
<!--
| HTL (to be open-sourced) | A patch repository based on upstream community source code for Kunpeng optimization and adaptation. |
| KSL_ASN1 | Content |
| BiSheng JDK acceleration library | Content |
| KACC_JSON | Content |
| HAF | Content |
| KVSIP | Content |
-->

<!--
### KMML (Media Library)

| Sub-library| Description|
| :--- | :--- |
| [HMPP documentation](https://gitcode.com/boostkit/boostmedia/blob/master/docs/hmpp/README.md) | Content|
| [KVCL documentation](https://gitcode.com/boostkit/boostmedia/blob/master/docs/kvcl/README.md)  | Content|
-->

### KAIL (Kunpeng AI Library)

| Sub-library| Description|
| :--- | :--- |
| [KAIL documentation](https://gitcode.com/boostkit/boostsra/blob/master/docs/en/kdnn/README.md) (Only the documentation repository path is provided. The code repository is to be open-sourced.) | Based on the microarchitecture features of the Kunpeng processor, Kunpeng Deep Neural Network Library (KDNN) enhances the performance of core DNN operators through vectorization, assembly-level optimizations, and algorithmic improvements. |

<!--
### KSAL (Storage Library)

| Sub-library| Description|
| :--- | :--- |
| KAIL| KDNN | Content|
| KSAL| KVCL | Content|
| KMML| KVCL | Content|
-->

<!--
### Other

| Sub-library| Description|
| :--- | :--- |
| [Brotli4j](https://gitcode.com/boostkit/Brotli4j) | A patch repository based on upstream community source code for Kunpeng optimization and adaptation.|
| [folly](https://gitcode.com/boostkit/folly) | This is an optimization repository for the folly asynchronous network I/O framework, focusing on optimizing the use of io_uring. The optimization solution adopts a hybrid mode: read operations use the multishot mode of io_uring to reduce system calls, while write operations fall back to the native send system call, which is better at handling continuous writes, to avoid potential latency introduced by io_uring in ordered scenarios.|
| [gosericesopt](https://gitcode.com/boostkit/msgp) | A patch repository based on upstream community source code for Kunpeng optimization and adaptation.|
| [gozstd](https://gitcode.com/boostkit/msgp) | A patch repository based on upstream community source code for Kunpeng optimization and adaptation.|
| [json-iterato](https://gitcode.com/boostkit/msgp) | A patch repository based on upstream community source code for Kunpeng optimization and adaptation.|
| [gosericesopt](https://gitcode.com/boostkit/msgp) | A patch repository based on upstream community source code for Kunpeng optimization and adaptation.|
| [KDADK-APPID](https://gitcode.com/boostkit/KDADK-APPID) | A network flow classification library of Kunpeng BoostKit for Intelligent Data Analytics (IDA). It includes flow aggregation, protocol detection, feature calculation, an AI engine, and a training tool package.|
| [KDADK-TOOLS](https://gitcode.com/boostkit/KDADK-TOOLS) | It is optimized for Kunpeng features and can analyze network traffic and detect encrypted traffic in real time. This addresses the limitations of conventional rule-based data distribution solutions that cannot analyze encrypted traffic. It provides examples of using feature extraction and inference APIs, and tools for traffic collection, feature filtering, model training, and adaptive labeling.|
| [kitex](https://gitcode.com/boostkit/msgp) | A patch repository based on upstream community source code for Kunpeng optimization and adaptation.|
| [msgp](https://gitcode.com/boostkit/msgp) | A patch repository based on upstream community source code for Kunpeng optimization and adaptation.|
| [suricata](https://gitcode.com/boostkit/suricata) | A patch repository based on upstream community source code for Kunpeng optimization and adaptation.|

### Public Configuration Repository

| Sub-library| Description|
| :--- | :--- |
| [AcclibBenchmark](https://gitcode.com/boostkit/AcclibBenchmark) | Used for storing performance testing scripts for accelerator cards.|
| [AcclibSkills](https://gitcode.com/boostkit/AcclibSkills) | Used for sharing effective techniques or skills applied during the debugging and development of the acceleration library.|
-->

## About the Community

This part provides the introduction and guidance to the public modules, such as the community governance architecture, SIG operations regulations, contribution, email subscription, and social media contact information.

## Contributions, Suggestions, and Discussions

We welcome your contributions to the community. If you have any questions/suggestions or want to provide feedback on feature requirements and bug reports, you can submit [issues](https://gitcode.com/boostkit/community/blob/master/docs/contributor/issue-submit.md). For details, see the [contribution guideline](https://gitcode.com/boostkit/community/blob/master/docs/contributor/contributing.md). You are also welcome to share insights in the [Discussions](https://gitcode.com/boostkit/community/discussions). Thank you for your support.

## License

This project is licensed under Apache License 2.0. For details, see [LICENSE](https://gitcode.com/boostkit/cloud-virtual/blob/master/LICENSE).

The documents of this project are licensed under CC-BY 4.0. For details, see [LICENSE](docs/LICENSE).
