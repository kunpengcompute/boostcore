# 安装指南

## 已验证环境

为保证您可以顺利安全地使用KQMalloc，请确保所使用的环境信息在已验证环境范围内。

| 操作系统 | CPU类型 | 编译器 |
| ------------------------- | --------- | ------------------ |
| openEuler 20.03 LTS SP1 | 鲲鹏920系列处理器 | GCC 7.3.0 / 毕昇4.2.0 |
| openEuler 22.03 LTS SP3 | 鲲鹏920系列处理器 | GCC 10.3.1 / 毕昇4.2.0 |
| openEuler 24.03 LTS SP3 | 鲲鹏950处理器 | GCC 12.3.1 / Clang 14.0.6 |

openEuler操作系统[获取链接](https://www.openeuler.openatom.cn/zh/download/?archive=true)。

>![](public_sys-resources/icon-notice.gif) **须知：**
>
>- KQMalloc目前只支持鲲鹏处理器，其他处理器使用KQMalloc时会异常退出。如果以动态的方式加载KQMalloc库，程序会在链接KQMalloc时退出，造成未定义的结果。
>- KQMalloc版本不支持GCC 7.3.0以下版本。
>- KQMalloc为底层原语库，计算流程涉及内存读写、分配等操作。KQMalloc不提供和也不发布操作系统，操作系统须用户自行安装，KQMalloc不承担操作系统的安全责任，用户需要结合自身应用对操作系统安全加固，包括不安装或者剔除不必要的应用等。

## 使用方法

使用KQMalloc之前，请先从[官方网站](https://repo.boostkit.osinfra.cn/boostcore/kqmalloc/release/v0.19-beta0/)获取KQMalloc的二进制压缩文件。

以0.19版本的KQMalloc动态库为例，用户解压缩该文件后即可获取到KQMalloc动态库。

 ```shell
 tar -xzvf kqmalloc-v0.19-beta0.tar.gz
 ```

### 启用透明大页

在运行应用前，建议先确认系统页大小并开启透明大页功能。

```shell
getconf PAGESIZE
echo always > /sys/kernel/mm/transparent_hugepage/enabled
echo madvise > /sys/kernel/mm/transparent_hugepage/defrag
```

若系统已成功开启透明大页，将显示如下所示状态。

```text
[root@localhost ~]# cat /sys/kernel/mm/transparent_hugepage/enabled
[always] madvise never
[root@localhost ~]# cat /sys/kernel/mm/transparent_hugepage/defrag
always defer defer+madvise [madvise] never
```

### 使用KQMalloc

KQMalloc通过优化标准库中的内存相关函数，只需使用预加载方式即可生效。
请根据所使用的鲲鹏处理器型号（HIP08, HIP09, HIP10A, HIP11, HIP12）选择对应的目录下的文件。
其中libkqmalloc.so是该库的单线程版本；libkqmallocmt.so是该库的多线程版本；libkqmallocmt.debug.so是启用了断言、double free()检查等功能的调试版本；libkqmallocmt.profiler.so是启用了内存分析功能的分析版本，分析文件会保存在/tmp/kqmalloc_<进程名>CPU *<cpu_id>*.txt文件中。

以鲲鹏HIP12的多线程场景为例：

```shell
export LD_PRELOAD=/<path to KQMalloc>/build/HIP12/lib/libkqmallocmt.so
```

同样，可以在GCC编译选项中添加动态库所在路径，链接需要使用的动态库文件。
例如，若需要使用适配HIP12的KQMalloc请在编译时添加：**-L/<KQMalloc路径>/build/HIP12/lib/ -lkqmallocmt -Wl,-rpath=/<KQMalloc路径>/build/HIP12/lib/** 。

### 验证KQMalloc

设置**LD_PRELOAD**环境变量或**rpath**编译路径后，可使用**ldd**命令检查是否成功链接KQMalloc库。正常链接结果如下。

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
