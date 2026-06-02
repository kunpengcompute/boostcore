# Installation Guide

KQMalloc of the new version is open-sourced. You can view the release version in the [source code repository](https://repo.boostkit.osinfra.cn/boostcore/kqmalloc/release/v0.19-beta0/).

## Verified Environments

To use KQMalloc smoothly and securely, ensure that your environment is one of the verified environments.

| OS                   | CPU  | Compiler                       |
|-------------------------|---------|----------------------------|
| openEuler 20.03 LTS SP1 | Kunpeng processor  | GCC 7.3.0 / BiSheng 4.2.0  |
| openEuler 22.03 LTS SP3 | Kunpeng processor  | GCC 10.3.1 / BiSheng 4.2.0 |
| openEuler 24.03 LTS SP3 | Kunpeng 950 processor  | GCC 12.3.1 / clang 14.0.6  |

Download a required openEuler version from [Download Center](https://www.openeuler.openatom.cn/en/download/?archive=true).

>![](public_sys-resources/icon-notice.gif) **NOTICE:**
>
>- KQMalloc supports only Kunpeng processors and will exit abnormally for any other processors. If a program dynamically loads KQMalloc, the program exits when KQMalloc is linked and an undefined result will be generated.
>- KQMalloc does not support GCC 7.3.0 or earlier.
>- KQMalloc is a primitive library whose computing process involves memory read, write, and allocation. KQMalloc does not provide or release an OS, or take responsibility for the OS security. You need to install an OS by yourself and perform security hardening, such as installing only necessary applications or uninstalling unnecessary applications that have been installed.

## Obtaining the Software Package

Before installing KQMalloc, obtain the software package from the official website and verify the software package to ensure that it is consistent with the original software package on the website.

Obtain the KQMalloc software installation package [KQMalloc-xxxx.zip](https://repo.boostkit.osinfra.cn/boostcore/kqmalloc/release/v0.19-beta0/) from GitCode. Decompress the ZIP file to obtain the RPM installation package.

   **_xxxx_** indicates the version number.

## Installing KQMalloc

This section describes how to install KQMalloc using an RPM package and verify KQMalloc after the installation.

**Installation Procedure<a name="section39201312169"></a>**

1. To prevent buffer overflow attacks, enable address space layout randomization (ASLR) before installing KQMalloc.

    >![](public_sys-resources/icon-note.gif) **NOTE:**
    >ASLR randomizes the layout of linear areas such as the heap, stack, and shared library mapping to make it more difficult for attackers to predict target addresses and locate code. This technology can be applied to heaps, stacks, and memory mapping areas (mmap base addresses, shared libraries, and vDSO pages).

    ```shell
    echo 2 >/proc/sys/kernel/randomize_va_space
    ```

2. Obtain the KQMalloc software package and decompress it to obtain the binary RPM package.
3. Install KQMalloc as the **root** user.

    ```shell
    rpm -ivh KQMalloc-xxxx.rpm
    ```

    After the installation, the system automatically adds the path to the **lib** folder, that is, **/usr/local/kqmalloc/lib**, to the environment variable **LD\_LIBRARY\_PATH**.

**Verification<a name="section193111321616"></a>**

1. Run the **source** command or log in to the terminal again for the environment variable to take effect.

    ```shell
    source /etc/profile
    ```

2. Check whether the environment variable **LD\_LIBRARY\_PATH** contains the installation path **/usr/local/kqmalloc/lib**.

    ```shell
    env | grep LD_LIBRARY_PATH
    ```

    If the command output contains the installation path, the installation is successful.

    ```text
    LD_LIBRARY_PATH=/usr/local/kqmalloc/lib:
    ```

    After the installation, the corresponding files are generated in the installation path (the default path is **/usr/local/kqmalloc**). The **include** folder contains the header files of the sub-libraries, the **lib** folder contains the dynamic library files of the system library.

3. Add the path of the dynamic library to the GCC compilation option to link the required dynamic library file. After compilation, run the **ldd** command to check that the dependency library of the program is correctly linked.
4. To use KQMalloc, add **-L/<path to KQMalloc>/build/HIP09/lib/ -lkqmallocmt -Wl,-rpath=/<path to KQMalloc>/build/HIP09/lib/** during compilation.
  
## Uninstalling KQMalloc

If you do not require KQMalloc, uninstall it. During uninstallation, service flows that are being executed are affected. You are advised to stop the service flows before uninstalling KQMalloc.

1. Run the **rpm -e** command to uninstall the RPM package.

    ```shell
    rpm -e kqmalloc
    ```

2. Verify that the **/usr/local/kqmalloc** directory is deleted.

    ```shell
    ll /usr/local/kqmalloc
    ```

    If the command output indicates that the directory cannot be found, the installation directory is deleted successfully.

3. Confirm that the **/usr/local/kqmalloc** environment variables are deleted from the **/etc/profile** file.

    ```shell
    cat /etc/profile
    ```
