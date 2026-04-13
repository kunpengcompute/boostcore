# 1 项目介绍
KQMalloc是专为鲲鹏处理器设计的内存分配器，针对Linux操作系统下4KB和64KB系统页大小、48位内存虚拟地址的系统场景进行了适配与优化。
KQMalloc使用透明大页等技术，可最大限度地减少页表缓冲未命中、降低缓存碎片和缓存别名问题。
该分配器支持单线程和多线程应用场景，可极大地提升应用性能。

# 2 特性介绍
**表1** KQMalloc特性介绍
|  特性名称 | 特性介绍  |
| ------------ | ------------ |
|  多级无锁缓存快速路径| 每个线程拥有独立的本地缓存，小对象分配仅需 10 条以内指令，全程无需加锁。未命中时逐级回退到 Quick Cache（CAS 原子操作的无锁缓存）和 Fast Cache（自适应容量的共享缓存）分配。 |
|  大页（THP）惰性释放与原地扩缩| 释放内存时使用 MADV_FREE 惰性回收，标记页面可回收但不立即归还，减少系统调用。大对象 realloc 通过虚拟内存提交/取消提交（commit/decommit）原地调整大小，避免分配新内存再拷贝。 |
|  精细的大小分级与低碎片堆选择| 将对象按大小划分为 100+ 级别，小对象以 4 字节为步进、大对象以缓存行为步进，每次分配匹配最接近的槽位。 |
|  双向堆脏页回收| 尾部裁剪（tail clean）：当堆驻留超过阈值，通过 MADV_DONTNEED 释放尾部多余页面；堆压缩（compression）回收尾部虚拟地址空间；中部回收（body clean）：对释放后的中间区域排序合并，向相邻空闲 cell 扩展以凑齐页面对齐边界后统一释放。 |

# 3 目录结构
关键目录如下，详细目录介绍参见[目录结构](docs/zh/dir_structure.md)

# 4 版本说明

参考[版本说明](docs/zh/release_notes.md)

# 5 环境兼容性信息
**表2** KQMalloc已验证环境
|  操作系统 | CPU 类型  | 编译器 |
| ------------ | ------------ | ------------ |
|  openEuler 20.03 LTS SP1| 鲲鹏处理器  | GCC 7.3.0 / BiSheng 4.2.0 |
|  openEuler 22.03 LTS SP3| 鲲鹏处理器  | GCC 10.3.1 / BiSheng 4.2.0 |
|  openEuler 24.03 LTS SP3| 鲲鹏处理器  | GCC 12.3.1 / clang 14.0.6 |

# 6 安装部署
## 6.1 启用透明大页
在运行应用前，建议先确认系统页大小并开启透明大页功能：
```shell
getconf PAGESIZE
echo always > /sys/kernel/mm/transparent_hugepage/enabled
echo madvise > /sys/kernel/mm/transparent_hugepage/defrag
```
若系统已成功开启透明大页，将显示如下所示状态：
```text
[root@localhost ~]# cat /sys/kernel/mm/transparent_hugepage/enabled
[always] madvise never
[root@localhost ~]# cat /sys/kernel/mm/transparent_hugepage/defrag
always defer defer+madvise [madvise] never
```

## 6.2 使用KQMalloc
KQMalloc通过优化标准库中的内存相关函数，只需使用预加载方式即可生效。
请根据所使用的鲲鹏处理器型号(HIP08, HIP09, HIP10A, HIP11, HIP12)选择对应的目录下的文件。
其中libkqmalloc.so是该库的单线程版本；libkqmallocmt.so是该库的多线程版本；libkqmallocmt.debug.so是启用了断言、double free()检查等功能的调试版本；libkqmallocmt.profiler.so是启用了内存分析功能的分析版本，分析文件会保存在/tmp/kqmalloc_<进程名>_CPU<cpu_id>_<ticks>.txt文件中。

以鲲鹏HIP12的多线程场景为例：

```shell
export LD_PRELOAD=/<path to KQMalloc>/build/HIP12/lib/libkqmallocmt.so
```

## 6.3 安装后验证
设置LD_PRELOAD环境变量后，可使用ldd命令检查是否成功链接KQMalloc库。正常链接结果如下：
```text
[root@localhost mmpm]# ldd test
        linux-vdso.so.1 (0x0000ffff9ee6a000)
        /home/kqmalloc/build/HIP12/lib/libkqmallocmt.so (0x0000ffff9b400000)
        libmmpm.so => not found
        libstdc++.so.6 => /usr/lib64/libstdc++.so.6 (0x0000ffff9b200000)
        libm.so.6 => /usr/lib64/libm.so.6 (0x0000ffff9b150000)
        libgcc_s.so.1 => /usr/lib64/libgcc_s.so.1 (0x0000ffff9b110000)
        libc.so.6 => /usr/lib64/libc.so.6 (0x0000ffff9af60000)
        /lib/ld-linux-aarch64.so.1 (0x0000ffff9ee2d000)
```

# 7 快速入门
KQMalloc内存分配器的快速入门，包括快速安装、使用方式和环境变量等，具体请参见[快速入门](docs/zh/quick_start.md)。

# 8 学习文档

**表3** KQMalloc学习资源

| 学习资源类别 | 学习资源名称 | 学习资源简介 |
| ------------ | ------------ | ------------ |
| 文档 | [快速入门](quick_start.md) | 介绍KQMalloc的基本概念、安装部署、使用方法及性能验证，帮助用户快速上手。 |
| 文档 | [安装指南](installation_guide.md) | 详细介绍KQMalloc的环境要求、软件包获取、安装步骤及卸载方法。 |
| 文档 | [API参考](api_reference.md) | 提供malloc、calloc、realloc、free等函数的详细说明和使用示例。 |
| 文档 | [版本说明](release_notes.md) | 记录各版本的功能更新、问题修复和已知限制等信息。 |
| 文档 | [简介](introduction.md) | 概述KQMalloc的设计目标、适用场景和核心特性。 |

# 9 贡献声明
欢迎大家为社区做贡献，如果使用过程中有任何问题/建议，或者需要反馈特性需求和bug报告，可以提交[issues](https://gitcode.com/boostkit/kqmalloc/issues/create/choose)联系我们，具体贡献方法可参考[这里](https://gitcode.com/boostkit/community/blob/master/docs/contributor/contributing.md)。

# 10 免责声明
此代码仓计划参与KQMalloc软件开源，仅作为KQMalloc功能扩展或性能提升，编码风格遵照原生开源软件，继承原生开源软件安全设计，不破坏原生开源软件设计及编码风格和方式，软件的任何漏洞与安全问题，均由相应的上游社区根据其漏洞和安全响应机制解决。请密切关注上游社区发布的通知和版本更新。鲲鹏计算社区对软件的漏洞及安全问题不承担任何责任。

# 11 License
本项目的文档适用CC-BY 4.0许可证，具体请参见[LICENSE](LICENSE)文件。
