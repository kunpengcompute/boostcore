# API参考

本文档介绍KQMalloc内存分配器API参考，涵盖malloc、calloc、realloc和free函数的定义、参数说明、返回值与示例，适用于在鲲鹏处理器上使用KQMalloc进行内存分配的开发者。

## KQMalloc介绍

KQMalloc是专为鲲鹏处理器设计的内存分配器，适配Linux操作系统4KB和64KB系统页、48位内存虚拟地址的系统场景。KQMalloc使用透明大页等技术减少页表缓冲未命中、缓存碎片和缓存别名。本文档详细介绍malloc、calloc、realloc和free函数。

以下示例以HIP09处理器目录中的V0.22预编译`libkqmalloc.ptag.so`为例。请将`/path/to/kqmalloc-vX.Y.Z`替换为实际解压目录。

## 函数介绍

### malloc

**函数功能**

分配内存。

**函数定义**

```c
void *malloc(size_t size);
```

**参数说明**

| 参数名 | 类型 | 参数说明 | 取值范围 | 输入/输出 |
| ------ | ---- | ---- | -------- | --------- |
| size | size_t | 申请内存的字节数。 | 0至SIZE_MAX，实际可分配大小受实现和可用内存限制 | 输入 |

**使用约束**

- `size`为0时，行为由具体实现决定，可能返回NULL或不可用于访问对象的非空指针；非空返回值仍需调用`free()`释放。
- 返回指针必须判空后再使用，只能访问申请大小范围内的内存。
- 对象地址的对齐受`KQMALLOC_ALIGNMENT`和`KQMALLOC_C11_COMPLIANT_PROCESSES`配置影响，详见《[快速入门](./quick_start.md)》的环境变量说明。使用前应确保配置满足目标类型的对齐要求；需要显式指定对齐时，应使用`aligned_alloc()`或`posix_memalign()`并满足对应接口的参数约束，不应假定普通分配会按缓存行或页面对齐。

**返回值**

- 成功：返回指向分配内存的指针。
- 失败：返回空指针。

**示例**

```c
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>

#define KQMALLOC_TEST_LEN 10

void MallocExample(void)
{
    int8_t *res = (int8_t *)malloc(KQMALLOC_TEST_LEN);
    if (res == NULL) {
        printf("res is null\n");
        return;
    }

    printf("malloc pointer address: %p\n", (void *)res);
    free(res);
}

int main(void)
{
    MallocExample();
    return 0;
}
```

将上述示例代码保存，再按照以下步骤编译并运行malloc函数示例。

1. 编译。

    ```shell
    kqmalloc_root=/path/to/kqmalloc-vX.Y.Z
    gcc -o run_your_application run_your_application.c \
        -L"$kqmalloc_root/HIP09/lib" \
        -lkqmalloc.ptag \
        -Wl,-rpath="$kqmalloc_root/HIP09/lib"
    ```

2. 运行。

    ```shell
    ./run_your_application
    ```

    运行结果如下。

    ```output
    malloc pointer address: 0xffff97ae8174
    ```

3. 验证是否使用KQMalloc。

    ```shell
    ldd ./run_your_application
    ```

    验证结果应包含KQMalloc动态库，示例如下。

    ```output
    libkqmalloc.ptag.so => /path/to/kqmalloc-vX.Y.Z/HIP09/lib/libkqmalloc.ptag.so (0x0000ffff6dac0000)
    ```

### calloc

**函数功能**

分配连续内存块并将其初始化为0。

**函数定义**

```c
void *calloc(size_t number_of_objects, size_t size_of_object);
```

**参数说明**

| 参数名 | 类型 | 参数说明 | 取值范围 | 输入/输出 |
| ------ | ---- | ---- | -------- | --------- |
| number_of_objects | size_t | 申请内存块的数量。 | 0至SIZE_MAX，且与size_of_object的乘积不得溢出size_t | 输入 |
| size_of_object | size_t | 单个内存块的字节数。 | 0至SIZE_MAX，且与number_of_objects的乘积不得溢出size_t | 输入 |

**使用约束**

- 总申请字节数为`number_of_objects * size_of_object`，实际可分配大小受实现和可用内存限制；乘积溢出时分配失败，返回NULL。
- 任一参数为0时，行为由具体实现决定，可能返回NULL或不可用于访问对象的非空指针；非空返回值仍需调用`free()`释放。
- 返回指针必须判空后再使用，只能访问总申请字节数范围内的内存；对齐要求与`malloc()`相同。

**返回值**

- 成功：返回指向分配内存的指针。
- 失败：返回空指针。

**示例**

```c
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>

#define KQMALLOC_TEST_LEN 10

void CallocExample(void)
{
    int8_t *res = (int8_t *)calloc(KQMALLOC_TEST_LEN, sizeof(int8_t));
    if (res == NULL) {
        printf("res is null\n");
        return;
    }

    printf("calloc pointer address: %p\n", (void *)res);
    free(res);
}

int main(void)
{
    CallocExample();
    return 0;
}
```

将上述示例代码保存，再按照以下步骤编译并运行calloc函数示例。

1. 编译。

    ```shell
    kqmalloc_root=/path/to/kqmalloc-vX.Y.Z
    gcc -o run_your_application run_your_application.c \
        -L"$kqmalloc_root/HIP09/lib" \
        -lkqmalloc.ptag \
        -Wl,-rpath="$kqmalloc_root/HIP09/lib"
    ```

2. 运行。

    ```shell
    ./run_your_application
    ```

    运行结果如下。

    ```output
    calloc pointer address: 0xffffafe4c690
    ```

### realloc

**函数功能**

重新分配内存，并保留旧内存中可容纳于新内存的数据。

**函数定义**

```c
void *realloc(void *old_pointer, size_t new_size);
```

**参数说明**

| 参数名 | 类型 | 参数说明 | 取值范围 | 输入/输出 |
| ------ | ---- | ---- | -------- | --------- |
| old_pointer | void * | 指向需重新分配的旧内存；为NULL时等价于调用malloc(new_size)。 | NULL，或兼容内存分配接口返回且尚未释放的内存块起始指针 | 输入 |
| new_size | size_t | 重新申请内存的字节数。 | 0至SIZE_MAX，实际可分配大小受实现和可用内存限制 | 输入 |

**使用约束**

- `old_pointer`不得为已释放的指针、内存块内部地址或栈上对象的地址，不得跨不兼容分配器使用。
- `new_size`为0时，行为由具体实现决定，原内存可能被释放；不能仅凭返回NULL判定原内存仍然有效。为避免歧义，释放内存应显式调用`free()`，调整大小时应传入非零值。
- 使用临时指针接收返回值，并判空后再使用。`new_size`非0且分配失败时，原内存仍然有效；成功后应仅使用返回指针，不再使用旧指针及其别名。
- 成功后仅保留原大小与`new_size`中较小字节数的数据，扩展部分未初始化；对齐要求与`malloc()`相同，不保证保留额外指定的对齐。

**返回值**

- 成功：返回指向重新分配内存的指针。
- 失败（new_size非0）：返回空指针，原内存仍然有效。new_size为0时参见上述使用约束。

**示例**

```c
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>

#define KQMALLOC_TEST_LEN 10

void ReallocExample(void)
{
    int8_t *old_pointer = (int8_t *)malloc(KQMALLOC_TEST_LEN * sizeof(int8_t));
    if (old_pointer == NULL) {
        printf("old_pointer is null\n");
        return;
    }

    printf("old_pointer address: %p\n", (void *)old_pointer);
    for (int8_t i = 0; i < KQMALLOC_TEST_LEN; ++i) {
        old_pointer[i] = i;
    }

    int8_t *new_pointer = (int8_t *)realloc(
        old_pointer, 2 * KQMALLOC_TEST_LEN * sizeof(int8_t));
    if (new_pointer == NULL) {
        printf("new_pointer is null\n");
        free(old_pointer);
        return;
    }

    printf("new_pointer address: %p\n", (void *)new_pointer);
    printf("new_pointer test data:");
    for (int8_t i = 0; i < KQMALLOC_TEST_LEN; ++i) {
        printf(" %d", new_pointer[i]);
    }
    printf("\n");
    free(new_pointer);
}

int main(void)
{
    ReallocExample();
    return 0;
}
```

将上述示例代码保存，再按照以下步骤编译并运行realloc函数示例。

1. 编译。

    ```shell
    kqmalloc_root=/path/to/kqmalloc-vX.Y.Z
    gcc -o run_your_application run_your_application.c \
        -L"$kqmalloc_root/HIP09/lib" \
        -lkqmalloc.ptag \
        -Wl,-rpath="$kqmalloc_root/HIP09/lib"
    ```

2. 运行。

    ```shell
    ./run_your_application
    ```

    运行结果如下。

    ```output
    old_pointer address: 0xffff921c8174
    new_pointer address: 0xffff921c87ec
    new_pointer test data: 0 1 2 3 4 5 6 7 8 9
    ```

### free

**函数功能**

释放已分配内存。

**函数定义**

```c
void free(void *p);
```

**参数说明**

| 参数名 | 类型 | 参数说明 | 取值范围 | 输入/输出 |
| ------ | ---- | ---- | -------- | --------- |
| p | void * | 指向需释放内存的指针；为NULL时不执行任何操作。 | NULL，或兼容内存分配接口返回且尚未释放的内存块起始指针 | 输入 |

**使用约束**

- `p`必须与分配时的分配器兼容，不得传入栈上对象的地址、内存块内部地址或已释放的指针。
- 不得重复释放；释放后不得再通过该指针或其别名访问原内存。将当前指针置为NULL不会将其他别名同步置为NULL。

**示例**

参见malloc函数示例。

>![表示说明的图片](public_sys-resources/icon-note.gif) **说明：**
>示例中的指针地址以实际运行结果为准。

## 修订记录

| 文档版本|发布日期 | 修改说明 |
| ------| -------|---------|
| 01| 2026-09-30 | 第一次正式发布。|
