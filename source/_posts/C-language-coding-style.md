---
title: C language coding style
date: 2017-09-13 22:50:08
categories: 重拾 C 语言
tags: C
---

本文描述的编码规范是个人比较喜欢的一些方式，基本上在个人项目中会优先采用，特此记录。
我比较喜欢 GNU/Linux 项目中的编码规范，所以本文中随处可见它们的影子~

## 如何选择合适的 Style
我个人一般是按照如下的优先级顺序来选择合适的编码规范：
1. 当前项目的编码规范
2. 组织的编码规范
3. 系统平台的编码规范 (比如 Windows 平台和 Linux 平台下 C 的编码规范是有所不同的)
4. 本文要描述的规范，也是个人比较喜欢的一些方式

## 命名方式
lowercase except enum members.

## include guard
Some orgnization doesn't suggest include guards for non-public header files.
I guess this is due to possible name confliction of MACROs.
Cause `_XX_H_` may be reserved for some situation, I take `XX_H_` for granted.
foo.h
```
#ifndef FOO_H_
#define FOO_H_

#endif
```

## constant variables
learned from google, use `k_` as prefix of constant variables.
```
const int32_t k_max_thread_count = 10;
```

## enum
- each member is in uppercase.
- enum name is in lowercase.
- do not use typedef on enum definition.

## each definition per line.
```
int32_t a;
char c;
```

## `*` close to var
```
int32_t *p; // good
int32_t* p; // bad
```


## type-cast
preserve blank before casting and var.
```
int32_t *p = (int32_t *) malloc(...);
```

## a function without parameters must use `(void)`
```
int32_t test(void)
{
    ...
}
```

## if statement
```
a = 1;
if (xxxx) { // bad

}
```
```
a = 1;

if (xxx) { // good

}
```
```
a = 1;

if (xxx) {

}
b = 1; // bad
```
```
a = 1;

if (xxx) {

}

b = 1; // good
```