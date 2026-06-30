# Installation Guide

## Verified Environments

To use KQMalloc smoothly and securely, ensure that your environment is one of the verified environments.

| OS | CPU | Compiler |
| ------------------------- | --------- | ---------------------------- |
| openEuler 20.03 LTS SP1 | Kunpeng 920 series | GCC 7.3.0 / BiSheng 4.2.0 |
| openEuler 22.03 LTS SP3 | Kunpeng 920 series | GCC 10.3.1 / BiSheng 4.2.0 |
| openEuler 24.03 LTS SP3 | Kunpeng 950 | GCC 12.3.1 / Clang 14.0.6 |

Download a required openEuler version from [Download Center](https://www.openeuler.openatom.cn/en/download/?archive=true).

>![](public_sys-resources/icon-notice.gif) **NOTICE**
>
>- KQMalloc supports only Kunpeng processors and will exit abnormally for any other processors. If a program dynamically loads KQMalloc, the program exits when KQMalloc is linked and an undefined result will be generated.
>- KQMalloc does not support GCC 7.3.0 or earlier.
>- KQMalloc is a primitive library whose computing process involves memory read, write, and allocation. KQMalloc does not provide or release an OS, or take responsibility for the OS security. You need to install an OS by yourself and perform security hardening, such as installing only necessary applications or uninstalling unnecessary applications that have been installed.

## Usage Method

Before using KQMalloc, obtain the KQMalloc binary compressed file from the [official website](https://repo.boostkit.osinfra.cn/boostcore/kqmalloc/release/v0.19-beta0/).

Taking the 0.19 version as an example, you can obtain the KQMalloc dynamic library after decompressing the file.

 ```shell
 tar -xzvf kqmalloc-v0.19-beta0.tar.gz
 ```

### Enabling Transparent Huge Pages

Before running the application, it is recommended to confirm the system page size and enable the Transparent Huge Pages (THP) feature.

```shell
getconf PAGESIZE
echo always > /sys/kernel/mm/transparent_hugepage/enabled
echo madvise > /sys/kernel/mm/transparent_hugepage/defrag
```

If THP have been successfully enabled on the system, the following status will be displayed:

```text
[root@localhost ~]# cat /sys/kernel/mm/transparent_hugepage/enabled
[always] madvise never
[root@localhost ~]# cat /sys/kernel/mm/transparent_hugepage/defrag
always defer defer+madvise [madvise] never
```

### Using KQMalloc

KQMalloc takes effect simply by using the preloading method, which optimizes memory-related functions in the standard library.
Please select the files under the corresponding directory based on the Kunpeng processor model you use.
Among them, `libkqmalloc.so` is the single-threaded version of the library; `libkqmallocmt.so` is the multi-threaded version of the library; `libkqmallocmt.debug.so` is the debugging version with assertions, `double free()` checks, and other features enabled; `libkqmallocmt.profiler.so` is the profiling version with memory profiling features enabled, and the profiling files will be saved in the `/tmp/kqmalloc_<process_name>CPU<cpu_id>.txt` file.

Taking the multi-threaded scenario as an example:

```shell
export LD_PRELOAD=/<path to KQMalloc>/build/HIP12/lib/libkqmallocmt.so
```

Similarly, you can add the path where the dynamic library is located to the GCC compilation options to link the dynamic library file that needs to be used.

### Verifying KQMalloc

After setting the `LD_PRELOAD` environment variable or the `rpath` compilation path, you can use the `ldd` command to check whether the KQMalloc library is successfully linked. The normal linking result is as follows:

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
