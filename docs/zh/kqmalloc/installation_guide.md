# 安装指南

本文介绍如何获取、使用和移除KQMalloc V0.22预编译动态库。KQMalloc以压缩包形式交付，解压后可直接使用其中的动态库。

## 已验证环境

为保证您可以顺利安全地使用KQMalloc，请确保所使用的环境信息在已验证环境范围内。

| 操作系统 | CPU类型 | 编译器 |
| ------------------------- | --------- | ------------------ |
| openEuler 20.03 LTS SP1 | 鲲鹏处理器 | GCC 7.3.0 / BiSheng 4.2.0 |
| openEuler 22.03 LTS SP3 | 鲲鹏处理器 | GCC 10.3.1 / BiSheng 4.2.0 |
| openEuler 24.03 LTS SP3 | 鲲鹏处理器 | GCC 12.3.1 / clang 14.0.6 |

openEuler操作系统[获取链接](https://www.openeuler.openatom.cn/zh/download/?archive=true)。

>![](public_sys-resources/icon-notice.gif) **须知：**
>
>- KQMalloc目前只支持鲲鹏处理器，其他处理器使用KQMalloc时会异常退出。如果以动态方式加载KQMalloc库，程序会在链接KQMalloc时退出，造成未定义的结果。
>- KQMalloc不支持GCC 7.3.0以下版本。
>- KQMalloc为底层原语库，计算流程涉及内存读写、分配等操作。KQMalloc不提供和发布操作系统，操作系统须用户自行安装。用户需要结合自身应用完成操作系统安全加固，包括不安装或者移除不必要的应用等。

## 获取软件包

1. 进入[KQMalloc发布目录](https://repo.boostkit.osinfra.cn/boostcore/kqmalloc/release/)，选择所需版本的子目录。
2. 下载版本子目录中的`kqmalloc-vX.Y.Z.tar.gz`压缩包，其中`X.Y.Z`表示版本号。如果下载页面提供校验文件，请按页面说明校验软件包完整性。
3. 解压压缩包。

    ```shell
    tar -xzf kqmalloc-vX.Y.Z.tar.gz
    ```

解压目录按HIP08、HIP09、HIP10A、HIP11和HIP12处理器型号组织。每个处理器目录的`lib`子目录包含以下动态库：

| 库文件 | 说明 |
| ------- | ---- |
| `libkqmalloc.ptag.so` | 启用指针标记特性的发布版本 |
| `libkqmalloc.ptag.debug.so` | 启用指针标记、断言和double free()检查等功能的调试版本 |
| `libkqmalloc.ptag.profiler.so` | 启用指针标记和内存分析功能的分析版本 |
| `libkqmalloc.so` | 未启用指针标记特性的发布版本 |
| `libkqmalloc.debug.so` | 未启用指针标记特性，启用断言和double free()检查等功能的调试版本 |
| `libkqmalloc.profiler.so` | 未启用指针标记特性，启用内存分析功能的分析版本 |

以上动态库均为并行分配器。分配器启动时按单线程模式初始化，检测到第二个线程后自动切换为多线程模式。未启用指针标记特性的版本适用于内核不支持指针标记的系统，或自身使用指针标记的应用（如Go、Java）。内存分析文件保存在`/tmp/kqmalloc_<进程名>_CPU<cpu_id>_<ticks>.txt`。

## 启用透明大页

在运行应用前，建议先确认系统页大小并开启透明大页功能。

```shell
getconf PAGESIZE
echo always > /sys/kernel/mm/transparent_hugepage/enabled
echo madvise > /sys/kernel/mm/transparent_hugepage/defrag
```

若系统已成功开启透明大页，将显示如下状态。

```text
[root@localhost ~]# cat /sys/kernel/mm/transparent_hugepage/enabled
[always] madvise never
[root@localhost ~]# cat /sys/kernel/mm/transparent_hugepage/defrag
always defer defer+madvise [madvise] never
```

## 使用KQMalloc

KQMalloc通过替换标准库中的内存分配函数实现性能优化。建议使用`LD_PRELOAD`预加载动态库，无需修改应用程序代码。

请根据鲲鹏处理器型号选择对应目录。以HIP09处理器上启用指针标记特性的发布版本为例：

```shell
kqmalloc_root=/path/to/kqmalloc-vX.Y.Z
LD_PRELOAD="$kqmalloc_root/HIP09/lib/libkqmalloc.ptag.so" ./run_your_application
```

如果系统或应用不支持指针标记，请使用`libkqmalloc.so`：

```shell
kqmalloc_root=/path/to/kqmalloc-vX.Y.Z
LD_PRELOAD="$kqmalloc_root/HIP09/lib/libkqmalloc.so" ./run_your_application
```

也可以在编译应用程序时链接动态库：

```shell
kqmalloc_root=/path/to/kqmalloc-vX.Y.Z
gcc -o run_your_application run_your_application.c \
    -L"$kqmalloc_root/HIP09/lib" \
    -lkqmalloc.ptag \
    -Wl,-rpath="$kqmalloc_root/HIP09/lib"
```

## 验证KQMalloc

设置`LD_PRELOAD`后，可在相同环境下执行`ldd`命令检查KQMalloc动态库是否已正确加载：

```shell
kqmalloc_root=/path/to/kqmalloc-vX.Y.Z
LD_PRELOAD="$kqmalloc_root/HIP09/lib/libkqmalloc.ptag.so" ldd ./run_your_application
```

正常输出中应包含KQMalloc动态库，例如：

```text
libkqmalloc.ptag.so => /path/to/kqmalloc-vX.Y.Z/HIP09/lib/libkqmalloc.ptag.so (0x0000ffff6dac0000)
```

## 移除KQMalloc

1. 停止使用KQMalloc的应用程序。
2. 取消应用程序的`LD_PRELOAD`或动态链接配置。
3. 如果不再需要保留动态库，删除解压得到的KQMalloc版本目录。
