# API Reference

<!-- md-trans-meta sourceCommit=9fa61b173fd7611c29025519a909923fe1a3bf15 translatedAt=2026-11-18T03:11:28.074Z pushedAt=2026-11-19T07:14:11.850Z -->

This document describes the KQMalloc memory allocator API, covering the definitions, parameter descriptions, return values, and examples of the malloc, calloc, realloc, and free functions. It is intended for developers who use KQMalloc for memory allocation on Kunpeng processors.

## KQMalloc Introduction

KQMalloc is a memory allocator designed for Kunpeng processors and is suitable for Linux environments with 4 KB or 64 KB system page size and 48-bit virtual memory addresses. KQMalloc uses technologies such as transparent huge pages to reduce TLB misses, cache fragmentation, and cache aliasing. This document describes the malloc, calloc, realloc, and free functions in detail.

The following uses the V0.22 precompiled `libkqmalloc.ptag.so` in the processor directory as an example. Replace `/path/to/kqmalloc-vX.Y.Z` with the actual decompression directory.

## Function Introduction

### malloc

**Function Description**

Allocates memory.

**Function Definition**

```c
void *malloc(size_t size);
```

**Parameter Description**

| Parameter | Type | Description | Value Range | Input/Output |
| ------ | ---- | ---- | -------- | --------- |
| size | size_t | Number of bytes to allocate. | 0 to SIZE_MAX; the actual allocatable size is limited by the implementation and available memory. | Input |

**Usage Constraints**

- When `size` is 0, the behavior is implementation-defined. The function may return NULL or a non-null pointer that cannot be used to access an object; a non-null return value must still be released by calling `free()`.
- The returned pointer must be checked against NULL before use, and only the memory within the requested size range can be accessed.
- The alignment of object addresses is affected by the `KQMALLOC_ALIGNMENT` and `KQMALLOC_C11_COMPLIANT_PROCESSES` configurations. For details, see the environment variable descriptions in [Quick Start](./quick_start.md). Before use, ensure that the configurations meet the alignment requirements of the target type. To explicitly specify the alignment, use `aligned_alloc()` or `posix_memalign()` and comply with the parameter constraints of the corresponding interface. Do not assume that a normal allocation is aligned to the cache line or page.

**Return Value**

- Success: A pointer to the allocated memory is returned.
- Failure: A null pointer is returned.

**Example**

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

Save the example code above, and then follow the steps below to compile and run the malloc function example.

1. Perform the compilation.

    ```shell
    kqmalloc_root=/path/to/kqmalloc-vX.Y.Z
    gcc -o run_your_application run_your_application.c \
        -L"$kqmalloc_root/HIP09/lib" \
        -lkqmalloc.ptag \
        -Wl,-rpath="$kqmalloc_root/HIP09/lib"
    ```

2. Run your application.

    ```shell
    ./run_your_application
    ```

    The run result is as follows.

    ```output
    malloc pointer address: 0xffff97ae8174
    ```

3. Check whether KQMalloc is used.

    ```shell
    ldd ./run_your_application
    ```

    The verification result should contain the KQMalloc dynamic library, as shown in the following example.

    ```output
    libkqmalloc.ptag.so => /path/to/kqmalloc-vX.Y.Z/HIP09/lib/libkqmalloc.ptag.so (0x0000ffff6dac0000)
    ```

### calloc

**Function Description**

Allocates contiguous memory blocks and initializes them to 0.

**Function Definition**

```c
void *calloc(size_t number_of_objects, size_t size_of_object);
```

**Parameter Description**

| Parameter | Type | Description | Value Range | Input/Output |
| ------ | ---- | ---- | -------- | --------- |
| number_of_objects | size_t | Number of memory blocks to allocate. | 0 to SIZE_MAX, and the product of this value and `size_of_object` must not overflow `size_t` | Input |
| size_of_object | size_t | Number of bytes per memory block. | 0 to SIZE_MAX, and the product of this value and `number_of_objects` must not overflow `size_t` | Input |

**Usage Constraints**

- The total number of requested bytes is `number_of_objects * size_of_object`. The actual allocatable size is limited by the implementation and available memory. If the product overflows, the allocation fails and NULL is returned.
- When either parameter is 0, the behavior is implementation-defined. The function may return NULL or a non-null pointer that cannot be used to access an object; a non-null return value must still be released by calling `free()`.
- The returned pointer must be checked against NULL before use, and only the memory within the total number of requested bytes can be accessed. The alignment requirements are the same as those of `malloc()`.

**Return Value**

- Success: A pointer to the allocated memory is returned.
- Failure: A null pointer is returned.

**Example**

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

Save the example code above, and then follow the steps below to compile and run the calloc function example.

1. Perform the compilation.

    ```shell
    kqmalloc_root=/path/to/kqmalloc-vX.Y.Z
    gcc -o run_your_application run_your_application.c \
        -L"$kqmalloc_root/HIP09/lib" \
        -lkqmalloc.ptag \
        -Wl,-rpath="$kqmalloc_root/HIP09/lib"
    ```

2. Run your application.

    ```shell
    ./run_your_application
    ```

    The run result is as follows.

    ```output
    calloc pointer address: 0xffffafe4c690
    ```

### realloc

**Function Description**

Reallocates memory and preserves as much of the data in the old memory as fits in the new memory.

**Function Definition**

```c
void *realloc(void *old_pointer, size_t new_size);
```

**Parameter Description**

| Parameter | Type | Description | Value Range | Input/Output |
| ------ | ---- | ---- | -------- | --------- |
| old_pointer | void * | Pointer to the old memory block to be reallocated; if it is NULL, this is equivalent to calling `malloc(new_size)`. | NULL, or the start pointer of a memory block returned by a compatible memory allocation interface and not yet freed | Input |
| new_size | size_t | Number of bytes to allocate for the reallocation. | 0 to SIZE_MAX; the actual allocatable size is limited by the implementation and available memory. | Input |

**Usage Constraints**

- `old_pointer` must not be a pointer that has been freed, an address inside a memory block, or the address of an object on the stack, and must not be used across incompatible allocators.
- When `new_size` is 0 and `old_pointer` is a valid non-NULL pointer, KQMalloc frees the original memory and returns NULL; the caller must not free or access the memory again. When `old_pointer` is NULL, this is equivalent to calling `malloc(new_size)`. For portability, explicitly call `free()` to free memory, and pass a non-zero value when resizing.
- Use a temporary pointer to receive the return value, and check it against NULL before use. When `new_size` is not 0 and the allocation fails, the original memory remains valid; on success, use only the returned pointer and stop using the old pointer and its aliases.
- On success, only the data of the smaller of the original size and `new_size` is retained; the extended portion is uninitialized. The alignment requirements are the same as those of `malloc()`, and no additionally specified alignment is guaranteed to be retained.

**Return Value**

- Success: A pointer to the reallocated memory is returned.
- Failure (`new_size` is not 0): A null pointer is returned, and the original memory remains valid.
- `new_size` is 0: If `old_pointer` is not NULL, a null pointer is returned, and the original memory has been freed. If `old_pointer` is NULL, the return value is the same as that of malloc(0).

**Example**

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

Save the example code above, and then follow the steps below to compile and run the realloc function example.

1. Perform the compilation.

    ```shell
    kqmalloc_root=/path/to/kqmalloc-vX.Y.Z
    gcc -o run_your_application run_your_application.c \
        -L"$kqmalloc_root/HIP09/lib" \
        -lkqmalloc.ptag \
        -Wl,-rpath="$kqmalloc_root/HIP09/lib"
    ```

2. Run your application.

    ```shell
    ./run_your_application
    ```

    The run result is as follows.

    ```output
    old_pointer address: 0xffff921c8174
    new_pointer address: 0xffff921c87ec
    new_pointer test data: 0 1 2 3 4 5 6 7 8 9
    ```

### free

**Function Description**

Frees the allocated memory.

**Function Definition**

```c
void free(void *p);
```

**Parameter Description**

| Parameter | Type | Description | Value Range | Input/Output |
| ------ | ---- | ---- | -------- | --------- |
| p | void * | Pointer to the memory to be freed; if it is NULL, no operation is performed. | NULL, or the start pointer of a memory block returned by a compatible memory allocation interface and not yet freed | Input |

**Usage Constraints**

- `p` must be compatible with the allocator used for allocation. Do not pass the address of an object on the stack, an address inside a memory block, or a pointer that has been freed.
- Do not free the same memory twice. After freeing, the original memory must not be accessed through the pointer or its aliases. Setting the current pointer to NULL does not set other aliases to NULL.

**Example**

See the example of the malloc function.

>![icon-note](public_sys-resources/icon-note.gif) **NOTE:**
>The pointer addresses in the examples are subject to the actual run results.

## Change History

| Version | Date | Description |
| --- | --- | --- |
| 01 | 2026-09-30 | This is the first official release. |
