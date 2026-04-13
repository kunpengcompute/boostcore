# 安装指南<a name="ZH-CN_TOPIC_0000002550002161"></a>
旧版本的KQMalloc位于KSL软件安装包[BoostKit-ksl\_2.5.3.zip](https://www.hikunpeng.com/developer/boostkit/library/system?subtab=AVX2KI)，用户解压缩zip文件可获取到RPM安装包。
新版本的KQMalloc已开源，可以在[源码仓](https://gitcode.com/boostkit/kqmalloc/)中查看发行版。

## 已验证环境<a name="ZH-CN_TOPIC_0000002518522356"></a>

为保证您可以顺利安全地使用KQMalloc，请确保所使用的环境信息在已验证环境范围内。

| 操作系统                    | CPU类型   | 编译器                        |
|-------------------------|---------|----------------------------|
| openEuler 20.03 LTS SP1 | 鲲鹏处理器   | GCC 7.3.0 / BiSheng 4.2.0  |
| openEuler 22.03 LTS SP3 | 鲲鹏处理器   | GCC 10.3.1 / BiSheng 4.2.0 |
| openEuler 24.03 LTS SP3 | 鲲鹏处理器   | GCC 12.3.1 / clang 14.0.6  |

openEuler操作系统[获取链接](https://www.openeuler.openatom.cn/zh/download/?archive=true)。

>![](public_sys-resources/icon-notice.gif) **须知：** 
>-   KQMalloc目前只支持鲲鹏处理器，其他处理器使用KQMalloc时会异常退出。如果以动态的方式加载KQMalloc库，程序会在链接KQMalloc时退出，造成未定义的结果。
>-   KQMalloc版本不支持GCC 7.3.0以下版本。
>-   KQMalloc为底层原语库，计算流程涉及内存读写、分配等操作。KQMalloc不提供和发布操作系统，操作系统须用户自行安装，KQMalloc不承担操作系统的安全责任，用户需要结合自身应用对操作系统安全加固，包括不安装或者剔除不必要的应用等。

## 获取软件包<a name="ZH-CN_TOPIC_0000002518522366"></a>

安装KQMalloc之前，请先从官方网站获取软件包，再进行软件包校验，确保与官方网站上的原始软件包一致。

1. 从gitcode上获取KQMalloc软件安装包[KQMalloc-xxxx.zip](https://gitcode.com/boostkit/kqmalloc)，用户解压缩zip文件可获取到RPM安装包。
   
   上述命令中涉及的**_xxxx_**代表版本号。

    >![](public_sys-resources/icon-note.gif) **说明：**
    >使用软件包前请先阅读《[鲲鹏应用使能套件BoostKit用户许可协议 2.0](https://www.hikunpeng.com/zh/legal/developer/boostkit/software/protocol)》，如确认继续使用，则默认同意协议的条款和条件。

2. <a name="zh-cn_topic_0000001087649782_li178091993411"></a>从[华为企业业务网站](https://support.huawei.com/enterprise/zh/tool/pgp-verify-TL1000000054)获取校验工具和校验方法。

## 安装<a name="ZH-CN_TOPIC_0000002518682168"></a>

本文提供RPM包安装KQMalloc的方法，请在安装后对KQMalloc进行验证。

**安装步骤<a name="section39201312169"></a>**

1. 为阻止缓冲区溢出攻击，建议在安装KQMalloc前开启使用ASLR（Address Space Layout Randomization）。

    >![](public_sys-resources/icon-note.gif) **说明：** 
    >ASLR技术通过对堆、栈、共享库映射等线性区布局的随机化，增加攻击者预测目的地址的难度，防止攻击者直接定位攻击代码位置。该技术可作用于堆、栈、内存映射区（mmap基址、shared libraries、vdso页）。

    ```shell
    echo 2 >/proc/sys/kernel/randomize_va_space
    ```

2. 获取到KQMalloc软件包后，解压后得到二进制RPM包。
3. 使用root用户安装KQMalloc。

    ```shell
    rpm -ivh KQMalloc-xxxx.rpm
    ```

    安装结束后，会自动在环境变量LD\_LIBRARY\_PATH中添加lib文件夹所在目录即"/usr/local/kqmalloc/lib"。

**安装后验证<a name="section193111321616"></a>**

1. 执行**source**命令或重新登录终端使环境变量生效。

    ```shell
    source /etc/profile
    ```

2. 查看环境变量LD\_LIBRARY\_PATH是否包含的安装路径"/usr/local/kqmalloc/lib"。

    ```shell
    env | grep LD_LIBRARY_PATH
    ```

    回显结果包含安装路径，说明安装成功，如下所示。

    ```text
    LD_LIBRARY_PATH=/usr/local/kqmalloc/lib:
    ```

    安装成功后在安装路径（默认路径是"/usr/local/kqmalloc"）下生成相应文件，其中，"include"文件夹包含子库的头文件，"lib"文件夹包含了系统库的动态库文件。

3. 使用时，请在GCC编译选项中添加动态库所在路径，链接需要使用的动态库文件，编译后用**ldd**命令检查程序依赖库是否准确链接。
4. 若需要使用KQMalloc请在编译时添加：**-L/<path to KQMalloc>/build/HIP09/lib/ -lkqmallocmt -Wl,-rpath=/<path to KQMalloc>/build/HIP09/lib/**
  
## 卸载<a name="ZH-CN_TOPIC_0000002518522398"></a>

若不再需要使用KQMalloc，可卸载KQMalloc。卸载KQMalloc将会影响您正在执行的业务流，建议先停止正在执行的业务流再进行卸载操作。

1. 执行**rpm -e**命令卸载RPM安装包。

    ```shell
    rpm -e kqmalloc
    ```

2. 查看"/usr/local/kqmalloc"目录，确认安装目录"/usr/local/kqmalloc"已被删除。

    ```shell
    ll /usr/local/kqmalloc
    ```

    回显信息中显示找不到该目录，则安装目录删除成功。

3. 确认"/etc/profile"文件中已删除"/usr/local/kqmalloc"相关环境变量。

    ```shell
    cat /etc/profile
    ```
