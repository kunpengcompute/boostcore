# Installation Guide

This document describes how to obtain, use, and remove the KQMalloc V0.22 precompiled dynamic library. KQMalloc is delivered as a compressed package, and the dynamic library contained in it can be used directly after decompression.

## Verified Environments

To use KQMalloc smoothly and securely, ensure that your environment is one of the verified environments.

| Operating System | CPU Type | Compiler |
| ------------------------- | --------- | ------------------ |
| openEuler 20.03 LTS SP1 | Kunpeng processor | GCC 7.3.0 / BiSheng 4.2.0 |
| openEuler 22.03 LTS SP3 | Kunpeng processor | GCC 10.3.1 / BiSheng 4.2.0 |
| openEuler 24.03 LTS SP3 | Kunpeng processor | GCC 12.3.1 / Clang 14.0.6 |

Download a required openEuler version from [Download Center](https://www.openeuler.openatom.cn/en/download/?archive=true).

>![icon-note](public_sys-resources/icon-notice.gif) **NOTICE**
>
>- KQMalloc currently supports only Kunpeng processors. When KQMalloc is used on other processors, it exits abnormally. If KQMalloc is loaded dynamically, the program exits when linking KQMalloc, causing undefined results.
>- KQMalloc does not support GCC versions earlier than 7.3.0.
>- KQMalloc is a primitive library, and its computing process involves operations such as memory read/write and allocation. KQMalloc does not provide or release an operating system, which must be installed by the user. Users need to harden the operating system security based on their own apps, including not installing or removing unnecessary apps.

## Obtaining the Software Package

1. Go to the [KQMalloc release directory](https://repo.boostkit.osinfra.cn/boostcore/kqmalloc/release/) and select the subdirectory of the required version.

2. Download the `kqmalloc-vX.Y.Z.tar.gz` package from the subdirectory, where `X.Y.Z` indicates the version number. If the download page provides a checksum file, verify the package integrity as instructed on the page.

3. Decompress the package.

    ```shell
    tar -xzf kqmalloc-vX.Y.Z.tar.gz
    ```

The decompressed directory is organized by processor models. The `lib` subdirectory of each processor directory contains the following dynamic libraries.

| Library File | Description |
| ------- | ---- |
| `libkqmalloc.ptag.so` | Release version with the pointer tagging feature enabled |
| `libkqmalloc.ptag.debug.so` | Debug version with the pointer tagging feature, assertions, and double free() checks enabled |
| `libkqmalloc.ptag.profiler.so` | Profiler version with the pointer tagging feature and memory profiling enabled |
| `libkqmalloc.so` | Release version with the pointer tagging feature disabled |
| `libkqmalloc.debug.so` | Debug version with the pointer tagging feature disabled and assertions and double free() checks enabled |
| `libkqmalloc.profiler.so` | Profiler version with the pointer tagging feature disabled and memory profiling enabled |

All the dynamic libraries above are parallel allocators. When the allocator starts, it is initialized in single-threaded mode and automatically switches to multi-threaded mode after a second thread is detected. Versions with the pointer tagging feature disabled are suitable for systems whose kernels do not support pointer tagging, or for apps that use pointer tagging themselves (such as Go and Java). Memory profiling files are saved in `/tmp/kqmalloc_<process_name>_CPU<cpu_id>_<ticks>.txt`.

## Enabling Transparent Huge Pages

Before running the app, it is recommended to first check the system page size and enable Transparent Huge Pages (THP).

```shell
getconf PAGESIZE
echo always > /sys/kernel/mm/transparent_hugepage/enabled
echo madvise > /sys/kernel/mm/transparent_hugepage/defrag
```

If THP has been successfully enabled, the following status is displayed.

```output
[root@localhost ~]# cat /sys/kernel/mm/transparent_hugepage/enabled
[always] madvise never
[root@localhost ~]# cat /sys/kernel/mm/transparent_hugepage/defrag
always defer defer+madvise [madvise] never
```

## Using KQMalloc

KQMalloc optimizes performance by replacing the memory allocation functions in the standard library. It is recommended to preload the dynamic library using `LD_PRELOAD`, which requires no modification to the app code.

The following usage methods are not ordered by priority. Select one based on the Kunpeng processor model and pointer tagging support.

- Take the release version with the pointer tagging feature enabled as an example.

    ```shell
    kqmalloc_root=/path/to/kqmalloc-vX.Y.Z
    LD_PRELOAD="$kqmalloc_root/HIP09/lib/libkqmalloc.ptag.so" ./run_your_application
    ```

- If the system or app does not support pointer tagging, use `libkqmalloc.so`.

    ```shell
    kqmalloc_root=/path/to/kqmalloc-vX.Y.Z
    LD_PRELOAD="$kqmalloc_root/HIP09/lib/libkqmalloc.so" ./run_your_application
    ```

- The dynamic library can also be linked when compiling the app.

    ```shell
    kqmalloc_root=/path/to/kqmalloc-vX.Y.Z
    gcc -o run_your_application run_your_application.c \
        -L"$kqmalloc_root/HIP09/lib" \
        -lkqmalloc.ptag \
        -Wl,-rpath="$kqmalloc_root/HIP09/lib"
    ```

## Verifying KQMalloc

After `LD_PRELOAD` is set, the `ldd` command can be executed in the same environment to check whether the KQMalloc dynamic library has been loaded correctly.

```shell
kqmalloc_root=/path/to/kqmalloc-vX.Y.Z
LD_PRELOAD="$kqmalloc_root/HIP09/lib/libkqmalloc.ptag.so" ldd ./run_your_application
```

The normal output should include the KQMalloc dynamic library, as shown in the following example.

```output
libkqmalloc.ptag.so => /path/to/kqmalloc-vX.Y.Z/HIP09/lib/libkqmalloc.ptag.so (0x0000ffff6dac0000)
```

## Removing KQMalloc

1. Stop the apps that use KQMalloc.

2. Remove the `LD_PRELOAD` or dynamic linking configuration of the apps.

3. If the dynamic library is no longer needed, delete the extracted KQMalloc version directory.

## Change History

|Document Version|Date|Description|
|--|--|--|
|01|2026-09-30|This is the first official release.|
