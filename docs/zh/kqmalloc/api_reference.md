# API参考

## 函数说明

KQMalloc是专为鲲鹏处理器设计的内存分配器，且特别适配优化Linux操作系统4KB和64KB系统页、48bits内存虚拟地址的系统场景。KQMalloc使用透明大页等技术，可最大限度地减少页表缓冲未命中，最大限度地减少缓存碎片，最大限度地减少缓存别名。此分配器支持单线程和多线程应用场景，可极大地提升应用性能。该库当前已优化的函数有malloc、calloc、realloc、free。

## 函数定义

### malloc

**函数功能<a name="section4942716451"></a>**

内存分配。

**函数定义<a name="section14649194274512"></a>**

```c
void *malloc(size_t size);
```

**参数说明<a name="zh-cn_topic_0000001201850423_section177211056192019"></a>**

|参数名|描述|取值范围|输入/输出|
|--|--|--|--|
|size|申请内存的字节数。|非负数|输入|

**返回值<a name="zh-cn_topic_0000001201850423_section104851617202119"></a>**

- 成功：返回指向分配内存的指针。
- 失败：返回空指针。

**示例<a name="zh-cn_topic_0000001201850423_section1292577112312"></a>**

1. 编辑示例代码。

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

2. 编译。
   
   ```shell
   gcc -o run_your_application run_your_application.c -L/<path to KQMalloc>/build/HIP09/lib/ -lkqmallocmt -Wl,-rpath=/<path to KQMalloc>/build/HIP09/lib/
   ```

3. 运行。

   ```shell
   ./run_your_application
   ```

   运行结果：

   ```text
   malloc pointer address: ffff97ae8174
   ```

4. 验证是否使用KQMalloc。

   ```shell
   ldd ./run_your_application
   ```

   运行结果：

   ```text
       linux-vdso.so.1 (0x0000ffff85520000)
       libkqmallocmt.so => /home/kqmalloc/build/HIP09/lib/libkqmallocmt.so (0x0000ffff6dac0000)
       libc.so => /usr/lib64/libc.so.6 (0x0000ffff6d910000)
       /lib/ld-linux-aarch64.so.1 (0x0000ffff854e3000)
   ```

   出现上述输出，即表示KQMalloc已成功加载并使能。

### calloc

**函数功能<a name="section1371791614611"></a>**

连续内存块分配并且初始化内存为0。

**函数定义<a name="section42042834613"></a>**

```c
void *calloc(size_t number_of_objects, size_t size_of_object);
```

**参数说明<a name="zh-cn_topic_0000001201850423_section177211056192019"></a>**

|参数名|描述|取值范围|输入/输出|
|--|--|--|--|
|number_of_objects|申请内存块的数量。|非负数|输入|
|size_of_object|申请内存块的字节数。|非负数|输入|

**返回值<a name="zh-cn_topic_0000001201850423_section104851617202119"></a>**

- 成功：返回指向分配内存的指针。
- 失败：返回空指针。

**示例<a name="zh-cn_topic_0000001201850423_section1292577112312"></a>**

1. 编辑示例代码
   
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

2. 编译。
   
   ```shell
   gcc -o run_your_application run_your_application.c -L/<path to KQMalloc>/build/HIP09/lib/ -lkqmallocmt -Wl,-rpath=/<path to KQMalloc>/build/HIP09/lib/
   ```

3. 运行。
   
   ```shell
   ./run_your_application
   ```
   
   运行结果：

   ```text
   calloc pointer address: ffffafe4c690
   ```

### realloc

**函数功能<a name="section816364604611"></a>**

内存重新分配并将旧内存中数据拷贝至新内存中。

**函数定义<a name="section82574577467"></a>**

```c
void *realloc(void *old_pointer, size_t new_size);
```

**参数说明<a name="zh-cn_topic_0000001201850423_section177211056192019"></a>**

|参数名|描述|取值范围|输入/输出|
|--|--|--|--|
|old_pointer|指向需重新分配的旧内存指针。|非空|输入|
|new_size|重新申请内存的字节数。|非负数|输入|

**返回值<a name="zh-cn_topic_0000001201850423_section104851617202119"></a>**

- 成功：返回指向新分配内存的指针。
- 失败：返回空指针。

**示例<a name="zh-cn_topic_0000001201850423_section1292577112312"></a>**

1. 编辑示例代码。

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

2. 编译。

   ```shell
   gcc -o run_your_application run_your_application.c -L/<path to KQMalloc>/build/HIP09/lib/ -lkqmallocmt -Wl,-rpath=/<path to KQMalloc>/build/HIP09/lib/
   ```

3. 运行。

   ```shell
   ./run_your_application
   ```

   运行结果：

   ```text
   old_pointer address: ffff921c8174
   old_pointer test data: 0 1 2 3 4 5 6 7 8 9
   new_pointer address: ffff921c87ec
   new_pointer test data: 0 1 2 3 4 5 6 7 8 9
   ```

### free

**函数功能<a name="section137634205477"></a>**

释放已分配内存。

**函数定义<a name="section29611292474"></a>**

```c
void free(void *p);
```

**参数说明<a name="zh-cn_topic_0000001201850423_section177211056192019"></a>**

|参数名|描述|取值范围|输入/输出|
|--|--|--|--|
|p|指向需释放内存的指针。|非空|输入|

**示例<a name="zh-cn_topic_0000001201850423_section1292577112312"></a>**

同malloc函数示例。

>![](public_sys-resources/icon-note.gif) **说明：** 
>以上结果仅供参考，以实际运行结果为准。
