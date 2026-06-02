# API Reference

## Functions

KQMalloc is a memory allocator designed for Kunpeng processors and is suitable for Linux environments especially with 4 KB or 64 KB system page size and 48-bit virtual memory addresses. KQMalloc uses techniques such as transparent huge pages to minimize translation lookaside buffer (TLB) misses, reduce cache fragmentation, and mitigate cache aliases. This allocator supports both single-threaded and multi-threaded applications, significantly improving performance. It provides optimized functions, including malloc, calloc, realloc, and free.

## Function Description

### malloc

**Function Usage<a name="section4942716451"></a>**

Allocates memory.

**Function Syntax<a name="section14649194274512"></a>**

```c
void *malloc(size_t size);
```

**Parameters<a name="en-us_topic_0000001201850423_section177211056192019"></a>**

|Parameter|Description|Value Range|Input/Output|
|--|--|--|--|
|size|Number of bytes of the requested memory.|The value cannot be negative.|Input|

**Return Value<a name="en-us_topic_0000001201850423_section104851617202119"></a>**

- Success: A pointer to the allocated memory is returned.
- Failure: A null pointer is returned.

**Example<a name="en-us_topic_0000001201850423_section1292577112312"></a>**

Edit the sample code.

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>

#define KQMALLOC_TEST_LEN 10

void MallocExample()
{
    int8_t *res = (int8_t *)malloc(KQMALLOC_TEST_LEN);
    if (res == NULL) {
        printf("res is null\n");
    } else {
        printf("malloc pointer address: %lx\n", res);
        free(res);
    }
}

int main(void) {
    MallocExample();
    return 0;
}
```

Perform the compilation.

```shell
gcc -o run_your_application run_your_application.c -L/<path to KQMalloc>/build/HIP09/lib/ -lkqmallocmt -Wl,-rpath=/<path to KQMalloc>/build/HIP09/lib/
```

Run your application.

```shell
./run_your_application
```

Output:

```text
malloc pointer address: ffff97ae8174
```

Check whether kqmalloc is used.

```shell
ldd ./run_your_application
```

Output:

```text
    linux-vdso.so.1 (0x0000ffff85520000)
    libkqmallocmt.so => /home/kqmalloc/build/HIP09/lib/libkqmallocmt.so (0x0000ffff6dac0000)
    libc.so => /usr/lib64/libc.so.6 (0x0000ffff6d910000)
    /lib/ld-linux-aarch64.so.1 (0x0000ffff854e3000)
```

If the preceding information is displayed, KQMalloc is successfully loaded and enabled.

### calloc

**Function Usage<a name="section1371791614611"></a>**

Allocates contiguous memory blocks and initializes the memory blocks to 0.

**Function Syntax<a name="section42042834613"></a>**

```c
void *calloc(size_t number_of_objects, size_t size_of_object);
```

**Parameters<a name="en-us_topic_0000001201850423_section177211056192019"></a>**

|Parameter|Description|Value Range|Input/Output|
|--|--|--|--|
|number_of_objects|Number of requested memory blocks.|The value cannot be negative.|Input|
|size_of_object|Number of bytes of the requested memory blocks.|The value cannot be negative.|Input|

**Return Value<a name="en-us_topic_0000001201850423_section104851617202119"></a>**

- Success: A pointer to the allocated memory is returned.
- Failure: A null pointer is returned.

**Example<a name="en-us_topic_0000001201850423_section1292577112312"></a>**

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>

#define KQMALLOC_TEST_LEN 10

void CallocExample()
{
    int8_t *res = (int8_t *)calloc(KQMALLOC_TEST_LEN, KQMALLOC_TEST_LEN);
    if (res == NULL) {
        printf("res is null\n");
    } else {
        printf("calloc pointer address: %lx\n", res);
        free(res);
    }
}

int main(void) {
    CallocExample();
    return 0;
}
```

Perform the compilation.

```shell
gcc -o run_your_application run_your_application.c -L/<path to KQMalloc>/build/HIP09/lib/ -lkqmallocmt -Wl,-rpath=/<path to KQMalloc>/build/HIP09/lib/
```

Run your application.

```shell
./run_your_application
```

Output:

```text
calloc pointer address: ffffafe4c690
```

### realloc

**Function Usage<a name="section816364604611"></a>**

Reallocates memory and copies data in the old memory to the new memory.

**Function Syntax<a name="section82574577467"></a>**

```c
void *realloc(void *old_pointer, size_t new_size);
```

**Parameters<a name="en-us_topic_0000001201850423_section177211056192019"></a>**

|Parameter|Description|Value Range|Input/Output|
|--|--|--|--|
|old_pointer|Pointer to the old memory to be reallocated.|The value cannot be NULL.|Input|
|new_size|Number of bytes of the reallocated memory.|The value cannot be negative.|Input|

**Return Value<a name="en-us_topic_0000001201850423_section104851617202119"></a>**

- Success: A pointer to the reallocated memory is returned.
- Failure: A null pointer is returned.

**Example<a name="en-us_topic_0000001201850423_section1292577112312"></a>**

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>

#define KQMALLOC_TEST_LEN 10

void ReallocExample()
{
    int8_t *old_pointer = (int8_t *)malloc(KQMALLOC_TEST_LEN * sizeof(int8_t));
    if (old_pointer == NULL) {
        printf("old_pointer is null\n");
        return;
    } else {
        printf("old_pointer address: %lx\n", old_pointer);
        for (int8_t i = 0; i < KQMALLOC_TEST_LEN; ++i) {
            old_pointer[i] = i;
        }
        printf("old_pointer test data:");
        for (int8_t i = 0; i < KQMALLOC_TEST_LEN; ++i) {
            printf(" %d", old_pointer[i]);
        }
        printf("\n");
    }

    int8_t *new_pointer = (int8_t *)realloc(old_pointer, 2 * KQMALLOC_TEST_LEN * sizeof(int8_t));
    if (new_pointer == NULL) {
        printf("new_pointer is null\n");
        free(old_pointer);
        return;
    } else {
        printf("new_pointer address: %lx\n", new_pointer);
        printf("new_pointer test data:");
        for (int8_t i = 0; i < KQMALLOC_TEST_LEN; ++i) {
            printf(" %d", new_pointer[i]);
        }
        printf("\n");
    }
    free(old_pointer);
    free(new_pointer);
}

int main(void) {
    ReallocExample();
    return 0;
}
```

Perform the compilation.

```shell
gcc -o run_your_application run_your_application.c -L/<path to KQMalloc>/build/HIP09/lib/ -lkqmallocmt -Wl,-rpath=/<path to KQMalloc>/build/HIP09/lib/
```

Run your application.

```shell
./run_your_application
```

Output:

```text
old_pointer address: ffff921c8174
old_pointer test data: 0 1 2 3 4 5 6 7 8 9
new_pointer address: ffff921c87ec
new_pointer test data: 0 1 2 3 4 5 6 7 8 9
```

### free

**Function Usage<a name="section137634205477"></a>**

Frees the allocated memory.

**Function Syntax<a name="section29611292474"></a>**

```c
void free(void *p);
```

**Parameters<a name="en-us_topic_0000001201850423_section177211056192019"></a>**

|Parameter|Description|Value Range|Input/Output|
|--|--|--|--|
|p|Pointer to the memory to be freed.|The value cannot be NULL.|Input|

**Example<a name="en-us_topic_0000001201850423_section1292577112312"></a>**

See the example of the malloc function.

>![](public_sys-resources/icon-note.gif) **NOTE:**
>The version number and compile time are subject to the running results in your environment. The preceding results are for reference only.
