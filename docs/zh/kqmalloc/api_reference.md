# API参考

## 函数说明

KQMalloc是专为鲲鹏处理器设计的内存分配器，适配Linux操作系统4KB和64KB系统页、48位内存虚拟地址的系统场景。KQMalloc使用透明大页等技术减少页表缓冲未命中、缓存碎片和缓存别名。本文档详细介绍malloc、calloc、realloc和free函数。

以下示例以HIP09处理器目录中的V0.22预编译`libkqmalloc.ptag.so`为例。请将`/path/to/kqmalloc-vX.Y.Z`替换为实际解压目录。

## 函数定义

### malloc

**函数功能**

分配内存。

**函数定义**

```c
void *malloc(size_t size);
```

**参数说明**

| 参数名 | 参数说明 | 取值范围 | 输入/输出 |
| ------ | ---- | -------- | --------- |
| size | 申请内存的字节数。 | 非负数 | 输入 |

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

按照以下步骤编译并运行malloc函数示例。

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

    ```text
    malloc pointer address: 0xffff97ae8174
    ```

3. 验证是否使用KQMalloc。

    ```shell
    ldd ./run_your_application
    ```

    验证结果应包含KQMalloc动态库，示例如下。

    ```text
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

| 参数名 | 参数说明 | 取值范围 | 输入/输出 |
| ------ | ---- | -------- | --------- |
| number_of_objects | 申请内存块的数量。 | 非负数 | 输入 |
| size_of_object | 单个内存块的字节数。 | 非负数 | 输入 |

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

按照以下步骤编译并运行calloc函数示例。

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

    ```text
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

| 参数名 | 参数说明 | 取值范围 | 输入/输出 |
| ------ | ---- | -------- | --------- |
| old_pointer | 指向需重新分配的旧内存；为NULL时等价于调用malloc(new_size)。 | NULL或有效指针 | 输入 |
| new_size | 重新申请内存的字节数。 | 非负数 | 输入 |

**返回值**

- 成功：返回指向重新分配内存的指针。
- 失败：返回空指针，原内存仍然有效。

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

按照以下步骤编译并运行realloc函数示例。

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

    ```text
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

| 参数名 | 参数说明 | 取值范围 | 输入/输出 |
| ------ | ---- | -------- | --------- |
| p | 指向需释放内存的指针；为NULL时不执行任何操作。 | NULL或有效指针 | 输入 |

**示例**

参见malloc函数示例。

>![](public_sys-resources/icon-note.gif) **说明：**
>示例中的指针地址以实际运行结果为准。

## 修订记录

| 文档版本|发布日期 | 修改说明 |
| ------| --------------------|---------|
| 01| 2026-09-30 | 第一次正式发布。<br><ul><li>[优化文档结构，统一函数说明、参数、返回值和示例格式。](#函数定义)</li><li>[更新V0.22预编译`libkqmalloc.ptag.so`的编译链接及加载验证示例。](#malloc)</li><li>完善[realloc()](#realloc)和[free()](#free)的参数边界及内存释放说明，修正指针打印和示例代码。</li></ul> |
