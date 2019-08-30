# exosip_windows_build

- 使用 Win10 + Mingw + MSYS-1.0.11 编译
  - [Mingw + MSYS 安装和配置](https://www.zhaokuangshi.cn/zh/2019/03/04/443/)
  - 点击 MSYS 图标使用 Mingw + MSYS 环境
  - 解压源代码 tar.gz 文件夹，命名为 exosip, osip, c-ares

- 编译顺序 c-ares, osip, exosip

## 编译 c-ares

```sh
./configure
# checking types of args and return type for recvfrom...到这里暂停
# [Waiting a few more minutes helped](https://github.com/c-ares/c-ares/issues/129)
# 注释 c-ares/ares_iphlpapi.h 中关于 `_SOCKET_ADDRESS` 的定义
make
make install
```

## 开启线程库

- [参考](https://stackoverflow.com/questions/13871609/error-when-cross-compiling-libosip2-with-mingw32-win32)
- [mingW + pthread](http://jeremybai.github.io/blog/2014/08/20/pthread)
  - 1 将dll\x86下的pthreadGC2.dll和pthreadGCE2.dll拷贝到MinGW的bin文件夹下；
  - 2 将include文件夹下的pthread.h、sched.h和semaphore.h拷贝到MinGW的include文件夹下；
  - 3 还有将lib\x86下的libpthreadGC2.a和libpthreadGCE2.a拷贝到MinGW的lib下面。

## 编译 osip

```sh
cd osip
./configure LIBS="-lpthreadGC2"
```

- 将 osip/include/osip2/internal.h 76 行的 `#include <stdio.h>` 移到文件顶部，并在该文件顶部增加 `#define HAVE_PTHREAD_WIN32`
- 将 osip/include/osipparser2/internal.h 87 行的 `#include <stdio.h>` 移到文件顶部
- 修改 osip/src/osip2/osip_time.c 文件 110 行如下

```c++
static int
_osip_gettimeofday_realtime (struct timeval *tp, void *tz)
{
  // FILETIME lSystemTimeAsFileTime;
  // LONGLONG ll_now;

  // GetSystemTimeAsFileTime (&lSystemTimeAsFileTime);
  // ll_now = (LONGLONG) lSystemTimeAsFileTime.dwLowDateTime + ((LONGLONG) (lSystemTimeAsFileTime.dwHighDateTime) << 32LL);
  // ll_now = ll_now / 10;         /* in us */
  // tp->tv_sec = (long) ll_now / 1000000;
  // tp->tv_usec = (long) ll_now % 1000000;
  // return 0;
  return gettimeofday(tp, tz);
}
```

```sh
make
make install
```

## 编译 exosip

```sh
cd exosip
./configure LIBS="-lpthreadGC2"
```

- 将 exosip/src/eXosip2.h 66 行的 `#include <stdio.h>` 移到文件顶部, 并将 68 行的 `#define HAVE_SYS_STAT_H` 注释
- 在 exosip/src/eXutils.c 和 exosip/src/jpipe.c 文件顶部增加 `#include<winerror.h>`
- 注释 c:/msys/1.0/local/include/osip2/osip_condv.h 61 行定义的 `timespec` 结构体, 如下

```c++
// c:/mingw/include/time.h
struct timespec
{ /* Period is sum of tv_sec + tv_nsec; while 32-bits is sufficient
   * to accommodate tv_nsec, we use 64-bit __time64_t for tv_sec, to
   * ensure that we have a sufficiently large field to accommodate
   * Microsoft's ambiguous __time32_t vs. __time64_t representation
   * of time_t; we may resolve this ambiguity locally, by casting a
   * pointer to a struct timespec to point to an identically sized
   * struct __mingw32_timespec, which is defined below.
   */
  __time64_t	  tv_sec;	/* seconds; accept 32 or 64 bits */
  __int32  	  tv_nsec;	/* nanoseconds */
};

// c:/msys/1.0/local/include/osip2/osip_condv.h 61 行
  struct timespec {
    long tv_sec;
    long tv_nsec;
  };
```

```sh
make
make insall
cd /usr/local
cp -r bin lib include /f/Git/exosip-win/pkg
```