---
title: Build Android Kernel of ARM64 on OS X
date: 2017-02-10 10:35:41
tags: 
  - Android kernel
  - OSX
---

### ‘elf.h’ file not found
拷贝 elf.h (二选一，两个都行)
- [github](https://gist.github.com/mlafeldt/3885346)
- 修改`/Volumes/android/aosp/external/elfutils/libelf/elf.h`
遇到了 `features.h 文件未找到的错误`，就将 `#include <features.h>` 一行注释掉。 
将 `elf.h` 拷贝到 `/usr/include` 或者 `/usr/local/include` 中，推荐放在后者，放在前者的话系统升级时会覆盖。*目前发现 `libelf`不需要*。 

### arm-linux-androideabi-gcc: error: unrecognized command line option '-mgeneral-regs-only’
这个错误是我的交叉编译环境错了，我写了一个脚本来设置了。 
```
#!/bin/sh
export AOSP_HOME=/Volumes/android/aosp
export ARCH=arm64
export CROSS_COMPILE=aarch64-linux-android-
export PATH=$AOSP_HOME/prebuilts/gcc/darwin-x86/aarch64/aarch64-linux-android-4.9/bin:$PATH

make flounder_defconfig
```

### 'vdso_offset_sigtramp' undeclared (first use in this function)
>In file included from arch/arm64/kernel/signal.c:36:0: 
arch/arm64/kernel/signal.c: In function 'setup_return': 
/Volumes/android/tegra/arch/arm64/include/asm/vdso.h:34:11: error: 'vdso_offset_sigtramp' undeclared (first use in this function) 
  (void *)(vdso_offset_##name - VDSO_LBASE + (unsigned long)(base)); \

1.提前更改`arch/arm64/kernel/vdso/gen_vdso_offsets.sh`脚本 
将如下一行
```
's/^\([0-9a-fA-F]*\) . VDSO_\([a-zA-Z0-9_]*\)$/\#define vdso_offset_\2\t0x\1/p'
```
更改为：
```
's/^\([0-9a-fA-F]*\) . VDSO_\([a-zA-Z0-9_]*\)$/\#define vdso_offset_\2 0x\1/p'
```
也就是，`"\t"` to `" "(whitespace)`
ps: 如果已经执行过`make`了遇到了该错误，可以执行`make clean`清理生成的h文件，再重新编译 

2.查看该源码文件，找到对应的头文件`vdso-offsets.h` 
该头文件是由上述脚本生成的，如果不想去改脚本，或者发现此时脚本已经没有效果(如果不会重新生成改头文件的话，确实没效果了已经)，就直接更改错误的头文件吧。 
```
#include <generated/vdso-offsets.h>

#define VDSO_SYMBOL(base, name)   \
({   \
(void *)(vdso_offset_##name - VDSO_LBASE + (unsigned long)(base)); \
})
```
更改`kernel`目录下的`include/generated/vdso-offsets.h`
将其中仅有的一行更改 
```
#define vdso_offset_sigtrampt0x04e0
```
====>>>
``` 
#define vdso_offset_sigtramp 0x04e0
```
### fatal error: dt-bindings/gpio/tegra-gpio.h: No such file or directory
>In file included from arch/arm64/boot/dts/tegra132-flounder-xaxb.dts:3:0: 
arch/arm64/boot/dts/tegra132-flounder-generic.dtsi:1:41: fatal error: dt-bindings/gpio/tegra-gpio.h: No such file or directory 
>`#`include `<`dt-bindings/gpio/tegra-gpio.h`>`
>compilation terminated. 
>make[1]: *** [arch/arm64/boot/dts/tegra132-flounder-xaxb.dtb] Error 1 
>make: *** [dtbs] Error 2

发现该文件的第一句 `#include <dt-bindings/gpio/tegra-gpio.h>`正是报错的地方 
一般这些都是相对`include`路径的相对路径，通常需要找到 `kernel` 源码中该模块的`include`路径 `arch/arm64/boot/dts/include`
(先查找了 `kernel` 根目录下的 `include` 文件夹，发现没有要找的文件，是否这个应该优先找本模块下的？) 
```
$ cd arch/arm64/boot/dts/include
$ ls -ali dt-bindings 
1451034 -rw-r--r--  1 ice  admin  34  2 20 22:31 dt-bindings 
$ cat dt-bindings 
../../../../../include/dt-bindings
```
到`../../../../../include/dt-bindings`
查看发现了我们需要的头文件，所以猜测此处文件本应该是一个 symlink 
动手解决之！ 
在当前的目录下： 
```
$ ln -s /Volumes/android/tegra/include/dt-bindings dt-bindings 
$ ls -ali dt-bindings 
1502515 lrwxr-xr-x  1 ice  admin    42  2 20 23:41 dt-bindings -> /Volumes/android/tegra/include/dt-bindings
```
继续编译。 
```
69 warnings generated. 
HOSTLD  scripts/mod/modpost 
ld: warning: PIE disabled. Absolute addressing (perhaps -mdynamic-no-	  pic) not allowed in code signed PIE, but used in _devtable_ptr374 from 	scripts/mod/file2alias.o. To fix this warning, don't compile with -	mdynamic-no-pic or link with -Wl,-no_pie 
CHK     include/generated/compile.h 
CHK     kernel/config_data.h 
DTC     arch/arm64/boot/dts/tegra132-flounder-xaxb.dtb 
DTC     arch/arm64/boot/dts/tegra132-flounder-xc.dtb 
DTC     arch/arm64/boot/dts/tegra132-flounder-xdxepvt.dtb 
DTC     arch/arm64/boot/dts/tegra132-flounder_lte-xaxbxcxdpvt.dtb 
OBJCOPY arch/arm64/boot/Image 
GZIP    arch/arm64/boot/Image.gz 
DTC     arch/arm64/boot/dts/tegra132-flounder-xaxb.dtb 
DTC     arch/arm64/boot/dts/tegra132-flounder-xc.dtb 
DTC     arch/arm64/boot/dts/tegra132-flounder-xdxepvt.dtb 
DTC     arch/arm64/boot/dts/tegra132-flounder_lte-xaxbxcxdpvt.dtb 
CAT     arch/arm64/boot/Image.gz-dtb
```
至此，内核编译已经结束。

### 后续
参照[官方链接](https://source.android.com/source/building-kernels.html)的说明去你本机的目录中去找相应的预编译好的内核。 
针对我的`Nexus 9`而言：
Device   | Binary Location
-------  | ---------------------------
volantis | device/htc/flounder-kernel

查看你的`<AOSP>/device/<vendor>/xx-kernel`下显示的文件后缀名，将其备份(防止我们编译的内核启动不了) 
将编译好的内核中具有同样扩展名的那个文件拷贝到该目录下。 
对于`Nexus 9`，拷贝`arch/arm64/boot/Image.gz-dtb `

``` shell
cd $AOSP_HOME 
source build/envSetup.sh 
lunch aosp_flounder-userdebug 
make bootimage 
```
生成`bootimage`
```
[100% 2/2] Target boot image: out/target/product/flounder/boot.img
```

