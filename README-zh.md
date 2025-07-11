# QEMU入门

## 1. 为什么要建这个工程？
- 作为QEMU的初学者，发现最新的10.x版本工程过于复杂，阅读源码很吃力
- 但是从老版本入手，例如0.x/1.x/2.x版本，学起来会容易很多
- 因此尝试在ubuntu20.04上，重新编译几个老版本，并支持gdb调试
- 后续会尝试在windows环境下，支持Visual Studio编译调试环境

## 2. 工程计划
- [ ] 支持0.1.6版本编译，并能通过gdb调试
- [x] 支持1.0.1版本编译，并能通过gdb调试
- [ ] 支持2.0.0版本编译，并能通过gdb调试

## 3. 编译环境
```
we@xxx:~$ uname -a
Linux xxx 6.6.87.2-microsoft-standard-WSL2 #1 SMP PREEMPT_DYNAMIC Thu Jun  5 18:30:46 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux

we@xxx:~$ gcc --version
gcc (Ubuntu 5.4.0-6ubuntu1~16.04.12) 5.4.0 20160609

we@xxx:~$ g++ --version
g++ (Ubuntu 5.4.0-6ubuntu1~16.04.12) 5.4.0 20160609

we@xxx:~$ python --version
Python 2.7.18
```

## 4. 编译启动步骤
### 4.1 安装编译工具
```
sudo apt install build-essential git wget libglib2.0-dev libsdl1.2-dev zlib1g-dev libpixman-1-dev autoconf libtool pkg-config
```

### 4.2 尝试编译
```
./configure --target-list=x86_64-softmmu
make -j$(nproc)
```

### 4.3 修复编译报错（以QEMU-1.0.1版本为例）
- 编译报错
```
/usr/bin/ld: ivshmem.o: undefined reference to symbol 'shm_open@@GLIBC_2.2.5'
/usr/bin/ld: /lib/x86_64-linux-gnu/librt.so.1: error adding symbols: DSO missing from command line
collect2: error: ld returned 1 exit status
make[1]: *** [Makefile:417: qemu-system-x86_64] Error 1
make: *** [Makefile:89: subdir-x86_64-softmmu] Error 2
```
- 在`config-host.mk`添加链接库`-lrt`
```
LIBS+=-pthread -lgthread-2.0 -pthread -lglib-2.0 -lrt
```
- 编译报错
```
/usr/bin/ld: ../libhw64/9pfs/virtio-9p.o: in function `stat_to_v9stat':
/xxx/qemu-1.0.1/hw/9pfs/virtio-9p.c:1029: undefined reference to `minor'
/usr/bin/ld: /home/we/learn-qemu/qemu-1.0.1/hw/9pfs/virtio-9p.c:1029: undefined reference to `major'
/usr/bin/ld: ../libhw64/9pfs/virtio-9p.o: in function `v9fs_create':
/xxx/qemu-1.0.1/hw/9pfs/virtio-9p.c:2225: undefined reference to `makedev'
/usr/bin/ld: ../libhw64/9pfs/virtio-9p.o: in function `v9fs_mknod':
/xxx/qemu-1.0.1/hw/9pfs/virtio-9p.c:2856: undefined reference to `makedev'
collect2: error: ld returned 1 exit status
make[1]: *** [Makefile:417: qemu-system-x86_64] Error 1
make: *** [Makefile:89: subdir-x86_64-softmmu] Error 2
```
- 在`hw/9pfs/virtio-9p.c`中添加头文件`#include <sys/sysmacros.h>`
```
#include <glib.h>
#include <glib/gprintf.h>
#include <sys/sysmacros.h>
```

## 5. QEMU版本下载
- https://download.qemu.org/