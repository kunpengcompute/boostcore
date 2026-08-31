# BoostCore基础加速

## 最新消息

- \[2026.03.30 \]：系统库新增protobuf、sonic-cpp。
- \[2025.12.30 \]：系统库新增KACC_JSON；加解密库新增KACC_cropto。-->
- \[2025.06.30 \]：新增KUAF鲲鹏统一加速框架。
- \[2024.02.22 \]：KAE鲲鹏加速引擎发布2.0版本。

## 项目介绍

BoostCore基础加速面向主流应用场景，提供OS及系统软件等基础软件层的加速软件包。例如：KAE硬件加速引擎、KSL系统库的kpglibc。

## 特性介绍

介绍BoostCore基础加速项目包含的所有基础加速特性的简要介绍，并提供特性仓的地址。

### KAE 鲲鹏加速引擎

| 子库 | 描述 |
| :--- | :--- |
| [KAE](https://gitcode.com/boostkit/KAE) | 鲲鹏加速引擎（KAE）是基于鲲鹏处理器提供的硬件加速解决方案，包含了 KAE 加解密和 KAE 解压缩两个模块，分别用于加速 SSL/TLS 应用和数据压缩，可以显著降低处理器消耗，提高处理器效率。此外，加速引擎对应用层屏蔽了其内部实现细节，用户通过 OpenSSL、zlib标准接口即可以实现快速迁移现有业务。 |
| [KUAF](https://gitcode.com/boostkit/KUAF) | 鲲鹏统一加速框架（KUAF）硬件加速调度框架. |

<!--
### HAAS硬件亲和算子

| 子库 | 描述 |
| :--- | :--- |
| HASS（待开源） | HAAS（Hardware Affinitive Acceleration Service）是鲲鹏自研的硬件亲和加速访问框架，对接大数据和搜推AI使用。HAAS通过从鲲鹏BoostKit大数据OmniOperatorJIT和OmniStream中提取基础算子并利用SVE/NEIN进行向量化优化，打造鲲鹏指令清河和原语库。HAAS使用于鲲鹏920新型号处理器，支持NEON指令（128位宽）和SVE指令（256位宽）。 |
-->

### KZL 压缩库

| 子库 | 描述 |
| :--- | :--- |
| [ZSTD](https://gitcode.com/boostkit/zstd) | 基于上游社区源码进行鲲鹏优化适配的补丁仓。 |
| [Snappy](https://gitcode.com/boostkit/snappy) | 基于上游社区源码进行鲲鹏优化适配的补丁仓。 |
| [LZ4](https://gitcode.com/boostkit/lz4) | 基于上游社区源码进行鲲鹏优化适配的补丁仓。 |
| [zlib](https://gitcode.com/boostkit/zlib) | 基于上游社区源码进行鲲鹏优化适配的补丁仓。 |
<!--| Kzstar（待开源） | 满足ZSTD协议的多线程压缩库。 |-->
<!--| Gzip | 右对齐 |-->

<!--
### KCL 加解密库

| 子库 | 描述 |
| :--- | :--- |
| KACC_crypto | 右对齐 |
-->

### KSL 系统库

| 子库 | 描述 |
| :--- | :--- |
|[kpglibc](https://gitcode.com/boostkit/kpglibc) | kpglibc是鲲鹏参与glibc开源社区的仓库，基于华为鲲鹏处理器利用向量化指令集对glibc中的字符串操作函数、内存操作函数和时间操作函数进行性能优化，适用于鲲鹏920新型号处理器、950处理器，推荐使用SVE优化的版本获得更好的性能。函数接口与开源glibc保持一致，接口内部不做完整入参校验，入参合法性及合理性由调用方业务来保证。 |
| [protobuf](https://gitcode.com/boostkit/protobuf) | 基于上游社区源码进行鲲鹏优化适配的补丁仓。 |
| [sonic-cpp](https://gitcode.com/boostkit/sonic-cpp) | 基于上游社区源码进行鲲鹏优化适配的补丁仓。 |
| [KQMalloc](./docs/zh/kqmalloc/README.md)（提供预编译动态库及配套文档） | KQMalloc是专为鲲鹏处理器设计的并行内存分配器，适配Linux操作系统下4KB和64KB系统页、48位内存虚拟地址的系统场景。分配器初始化时按单线程模式运行，检测到第二个线程后自动切换到多线程模式。KQMalloc使用透明大页等技术，可减少页表缓冲未命中、缓存碎片和缓存别名问题。 |
| [AVX2KI](https://gitcode.com/boostkit/avx2ki) | AVX intrinsic向鲲鹏指令集迁移库。 |

<!--
| HTL（待开源） | 基于上游社区源码进行鲲鹏优化适配的补丁仓。 |
| KSL_ASN1  | 内容 |
| 毕昇JDK加速库 | 内容 |
| KACC_JSON | 内容 |
| HAF | 内容 |
| KVSIP | 内容 |
-->

### KMML 媒体库

| 子库 | 描述 |
| :--- | :--- |
| [HMPP文档](https://gitcode.com/boostkit/boostmedia/blob/master/docs/hmpp/README.md) （仅提供文档仓，代码仓待开源） | HMPP（Hyper Media Performance Primitives，鲲鹏超媒体性能库）是在鲲鹏处理器硬件平台基础上开发的加速库，涉及图像处理、颜色转换、滤波、变换、几何，为计算机视觉运算、向量运算、统计、信号滤波、信号变换和固定精度运算等，通过向量指令集对业务功能提供高性能加速函数接口。 |
| [KVCL文档](https://gitcode.com/boostkit/boostmedia/blob/master/docs/kvcl/README.md) （仅提供文档仓，代码仓待开源） | 采用向量化指令对视频编码通用基础算子进行优化，结合硬件特性充分利用向量化指令加速，提升算子性能。 |

### KAIL AI算子库

| 子库 | 描述 |
| :--- | :--- |
| [鲲鹏AI算子库文档](https://gitcode.com/boostkit/boostsra/blob/master/docs/zh/kdnn/README.md) （仅提供文档仓，代码仓待开源） | Kunpeng Deep Neural Network Library，鲲鹏深度神经网络算子库，结合鲲鹏处理器微架构特性，通过向量化、汇编、算法优化等手段，提升DNN核心算子性能。 |

### KSAL 存储库

| 子库 | 描述 |
| :--- | :--- |
| [ISA-L](https://gitcode.com/boostkit/isa-l) | ISA-L是开源的高性能存储算法库，在开源基础上，通过使用NEON/SVE指令指令、内存预期等手段进一步纠删码、优化循环冗余检查（CRC）。 |
| [KAE使能SPDK](https://gitcode.com/boostkit/spdk) | SPDK的BDEV设备作为虚拟设备层对接底层多种设备类型（虚拟设备、物理设备），通过在BDEV设备中使能压缩和加解密能够支持所有SPDK设备。 |

<!--
### 其他

| 子库 | 描述 |
| :--- | :--- |
| [Brotli4j](https://gitcode.com/boostkit/Brotli4j) | 基于上游社区源码进行鲲鹏优化适配的补丁仓。 |
| [folly](https://gitcode.com/boostkit/folly) | 本项目是针对folly异步网络io框架的优化仓库，聚焦并优化io_uring的使用。本优化方案采用混合模式，即读操作使用io_uring的multishot模式以减少系统调用，而写操作回退到使用更擅长处理连续写的原生send系统调用，以规避io_uring在保序场景下可能引入的延迟。 |
| [gosericesopt](https://gitcode.com/boostkit/msgp) | 基于上游社区源码进行鲲鹏优化适配的补丁仓。 |
| [gozstd](https://gitcode.com/boostkit/msgp) | 基于上游社区源码进行鲲鹏优化适配的补丁仓。 |
| [json-iterato](https://gitcode.com/boostkit/msgp) | 基于上游社区源码进行鲲鹏优化适配的补丁仓。 |
| [gosericesopt](https://gitcode.com/boostkit/msgp) | 基于上游社区源码进行鲲鹏优化适配的补丁仓。 |
| [KDADK-APPID](https://gitcode.com/boostkit/KDADK-APPID) | 鲲鹏数据分析开发套件的网络流分类库，包含流汇聚、协议检测、特征计算、AI引擎和训练工具包。 |
| [KDADK-TOOLS](https://gitcode.com/boostkit/KDADK-TOOLS) | 用户可借助该项目实现网络流量分析及加密流量实时检测，弥补传统DPI方案短板。它提供特征提取与推理接口示例，包含流量采集、特征过滤、模型训练及自适应打标签工具，针对鲲鹏特性优化。 |
| [kitex](https://gitcode.com/boostkit/msgp) | 基于上游社区源码进行鲲鹏优化适配的补丁仓。 |
| [msgp](https://gitcode.com/boostkit/msgp) | 基于上游社区源码进行鲲鹏优化适配的补丁仓。 |
| [suricata](https://gitcode.com/boostkit/suricata) | 基于上游社区源码进行鲲鹏优化适配的补丁仓。 |

### 公共配置仓

| 子库 | 描述 |
| :--- | :--- |
| [AcclibBenchmark](https://gitcode.com/boostkit/AcclibBenchmark) | 用于存放加速卡领域性能测试脚本。 |
| [AcclibSkills](https://gitcode.com/boostkit/AcclibSkills) | 用于分享加速库在调试开发过程中，应用效果较好的skills。 |
-->

## 关于社区

提供社区治理架构、SIG组织运作章程、参与贡献、邮件订阅、社媒联系方式等公共模块内容简介和指引。

## 贡献、建议与交流

欢迎大家为社区做贡献，如果使用过程中有任何问题/建议，或者需要反馈特性需求和bug报告，可以提交[Issues](https://gitcode.com/boostkit/boostcore/issues/create/choose)联系我们，具体贡献方法可参考[这里](https://gitcode.com/boostkit/community/blob/master/docs/contributor/contributing.md)。同时也欢迎大家在[讨论专区](https://gitcode.com/boostkit/community/discussions)展开讨论交流。感谢您的支持。

## LICENSE

本项目采用Apache License 2.0许可证。详见[LICENSE](https://gitcode.com/boostkit/cloud-virtual/blob/master/LICENSE)文件。

本项目的文档适用CC-BY 4.0许可证，具体请参见[LICENSE](docs/zh/kqmalloc/LICENSE)文件。
