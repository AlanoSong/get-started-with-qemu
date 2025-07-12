# Get started with QEMU

[中文](./README-zh.md)

## 1. Why This Project?
- As a beginner to QEMU, I found the latest 10.x version overly complex, making source code reading challenging
- However, learning from older versions (e.g., 0.x/1.x/2.x) is much easier.
- Thus, I aim to recompile several legacy versions on Ubuntu 20.04 with gdb debugging support
- Future plans include supporting compile and debugging in Visual Studio on Windows

## 2. Project Plan
- [ ] Support compile and gdb debugging for version **0.1.6**
- [x] Support compile and gdb debugging for version **1.0.1**
- [ ] Support compile and gdb debugging for version **2.0.0**

## 3. Compile Environment
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

## 4. Compile Steps
### 4.1 Install Build Tools
```
sudo apt install build-essential git wget libglib2.0-dev libsdl1.2-dev zlib1g-dev libpixman-1-dev autoconf libtool pkg-config
```
### 4.2 Attempt Compile
```
./configure --target-list=x86_64-softmmu
make -j$(nproc)
```
### 4.3 Fix Compile Errors (Example: QEMU-1.0.1)
- **Error 1**: Missing `librt` symbol
```
/usr/bin/ld: ivshmem.o: undefined reference to symbol 'shm_open@@GLIBC_2.2.5'
/usr/bin/ld: /lib/x86_64-linux-gnu/librt.so.1: error adding symbols: DSO missing from command line
collect2: error: ld returned 1 exit status
make[1]: *** [Makefile:417: qemu-system-x86_64] Error 1
make: *** [Makefile:89: subdir-x86_64-softmmu] Error 2
```
- **Solution**: Add `-lrt` to `config-host.mk`
```
LIBS+=-pthread -lgthread-2.0 -pthread -lglib-2.0 -lrt
```
- **Error 2**: Undefined references to `minor`, `major`, `makedev`
```
/usr/bin/ld: ../libhw64/9pfs/virtio-9p.o: in function `stat_to_v9stat':
/xxx/qemu-1.0.1/hw/9pfs/virtio-9p.c:1029: undefined reference to `minor'
/usr/bin/ld: /xxx/qemu-1.0.1/hw/9pfs/virtio-9p.c:1029: undefined reference to `major'
/usr/bin/ld: ../libhw64/9pfs/virtio-9p.o: in function `v9fs_create':
/xxx/qemu-1.0.1/hw/9pfs/virtio-9p.c:2225: undefined reference to `makedev'
/usr/bin/ld: ../libhw64/9pfs/virtio-9p.o: in function `v9fs_mknod':
/xxx/qemu-1.0.1/hw/9pfs/virtio-9p.c:2856: undefined reference to `makedev'
collect2: error: ld returned 1 exit status
make[1]: *** [Makefile:417: qemu-system-x86_64] Error 1
make: *** [Makefile:89: subdir-x86_64-softmmu] Error 2
```
- **Solution**: Add `#include <sys/sysmacros.h>` to `hw/9pfs/virtio-9p.c`
```
#include <glib.h>
#include <glib/gprintf.h>
#include <sys/sysmacros.h>
```
### 4.4 Launch QEMU Linux Image
- QEMU images are located in the bin directory
```
./qemu-system-x86_64 -m 4096 -hda /xxx/linux-0.2.img
```

## 5. QEMU Version Downloads
- https://download.qemu.org/