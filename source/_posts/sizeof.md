---
title: About sizeof
date: 2017-09-11 10:03:04
categories: 重拾 C 语言
tags: C
---

`sizeof` **只能引用自动分配的变量**，而不是一个指针可能指向的数据。
自动分配的或者静态的数组，都是可以使用 `sizeof` 来运算的。
因为sizeof是在编译期确定的，所以手动 `malloc` 的指针是 `sizeof` 无法处理的。
<!-- more -->

### Sample
``` C
#include <stdio.h> 
int a[10];

static void print_pointer_size(int data[]);

int main(void) 
{ 
    int *p = malloc(sizeof(int) * 10); 
    printf("%u, %u\n", sizeof(a), sizeof(p));
    print_pointer_size(a);
    free(p);
    return 0;
}

void print_pointer_size(int data[])
{
    printf("[%s] size of data: %u\n", __FUNC__, sizeof(data));
}
```

### 运行
``` 
valgrind --leak-check=full ./test
==5666== Memcheck, a memory error detector
==5666== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
==5666== Using Valgrind-3.13.0 and LibVEX; rerun with -h for copyright info
==5666== Command: ./test
==5666== 
[main] size of array a: 40, size of allocated pointer: 8
[print_pointer_size] size of data: 8
==5666== 
==5666== HEAP SUMMARY:
==5666==     in use at exit: 0 bytes in 0 blocks
==5666==   total heap usage: 2 allocs, 2 frees, 1,064 bytes allocated
==5666== 
==5666== All heap blocks were freed -- no leaks are possible
==5666== 
==5666== For counts of detected and suppressed errors, rerun with: -v
==5666== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)
```