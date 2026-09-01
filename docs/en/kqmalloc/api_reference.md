# API Reference

## Functions

KQMalloc is a memory allocator designed for Kunpeng processors and is suitable for Linux environments with 4 KB or 64 KB system page size and 48-bit virtual memory addresses. KQMalloc uses technologies such as transparent huge pages to reduce TLB misses, cache fragmentation, and cache aliasing. This document describes the malloc, calloc, realloc, and free functions in detail.

The following examples use the V0.22 precompiled `libkqmalloc.ptag.so` in the processor directory as an example. Replace `/path/to/kqmalloc-vX.Y.Z` with the actual decompression directory.

## Function Definitions

### malloc

**Function Description**

Allocates memory.

**Function Definition**

```c
void *malloc(size_t size);
```

**Parameter Description**

|Parameter|Description|Value Range|Input/Output|
|--|--|--|--|
|size|Number of bytes of the requested memory.|The value cannot be negative.|Input|

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

Compile and run the malloc function example by following these steps.

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

    ```text
    malloc pointer address: 0xffff97ae8174
    ```

3. Check whether KQMalloc is used.

    ```shell
    ldd ./run_your_application
    ```

    The verification result should contain the KQMalloc dynamic library, as shown in the following example.

    ```text
    libkqmalloc.ptag.so => /path/to/kqmalloc-vX.Y.Z/HIP09/lib/libkqmalloc.ptag.so (0x0000ffff6dac0000)
    ```

### calloc

**Function Description**

Allocates contiguous memory blocks and initializes the memory blocks to 0.

**Function Definition**

```c
void *calloc(size_t number_of_objects, size_t size_of_object);
```

**Parameter Description**

|Parameter|Description|Value Range|Input/Output|
|--|--|--|--|
|number_of_objects|Number of requested memory blocks.|The value cannot be negative.|Input|
|size_of_object|Number of bytes of a single memory block.|The value cannot be negative.|Input|

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

Compile and run the calloc function example by following these steps.

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

    ```text
    calloc pointer address: 0xffffafe4c690
    ```

### realloc

**Function Description**

Reallocates memory and preserves the data in the old memory that can fit into the new memory.

**Function Definition**

```c
void *realloc(void *old_pointer, size_t new_size);
```

**Parameter Description**

|Parameter|Description|Value Range|Input/Output|
|--|--|--|--|
|old_pointer|Pointer to the old memory to be reallocated. If it is NULL, this is equivalent to calling malloc(new_size).|NULL or a valid pointer|Input|
|new_size|Number of bytes of the reallocated memory.|The value cannot be negative.|Input|

**Return Value**

- Success: A pointer to the reallocated memory is returned.

- Failure: A null pointer is returned, and the original memory remains valid.

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

Compile and run the realloc function example by following these steps.

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

    ```text
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

|Parameter|Description|Value Range|Input/Output|
|--|--|--|--|
|p|Pointer to the memory to be freed. If it is NULL, no operation is performed.|NULL or a valid pointer|Input|

**Example**

See the example of the malloc function.

>![](public_sys-resources/icon-note.gif) **NOTE**
>The pointer addresses in the examples are subject to the actual run results.

## Change History

|Document Version|Date|Description|
|--|--|--|
|01|2026-09-30|This is the third official release.|
