---
layout:         post
title:          Build debug environment for the dynamic linker of Glibc
data:           2016-08-25
auther:         zet
mail:           zet@gmail.com
summary:        Describe how to build debug environment for the dynamic linker of Glibc and some analysis detials
categories:     Toolchains
---

# Build debug environment for the dynamic linker of Glibc
@(Toolchains)[Glibc|dynamic-linker|gdb|gcc]

## 00 Prologue

Recently my work need some research about the dynamic linker of Glibc, when I
use gdb from the PLT table of executable or shared library into the Glibc source
in a general way will see this:

```bash
[...]
   |0xf7ff04b3      mov    0x10(%esp),%edx                                                                                                         |
  >|0xf7ff04b7      mov    0xc(%esp),%eax                                                                                                          |
   |0xf7ff04bb      call   0xf7fea080     
[...] 
(gdb) si
[...]
0x0804852b in stub@plt ()
0x08048500 in ?? ()
0xf7ff04b0 in ?? () from /lib/ld-linux.so.2
0xf7ff04b3 in ?? () from /lib/ld-linux.so.2
[...]
0xf7ff04b7 in ?? () from /lib/ld-linux.so.2
[...]

```
So, how can i overcome this?

## 01 Analysis

I can deduce must be miss debug symbol information from the assembly code of the
top half of above list, and also miss lineno table for gdb from the output of 
command window of gdb.

Because I have successful experience of build debug environment(../configure 
CFLAGS='-Og' ,etc) for GCC/Binutils, so first I build a new imaginary debuggable
Glibc from source.

```bash
gcc -o hello_world ... \
-Wl,--rpath=/new/glibc \
-Wl,--dynamic-linker=/new/glibc/lib/ld-linux.so.2 \
hello_world.c

```
When gdb into Glibc it still not works. so sad.

I thought it must be configure/makefile/linker scripts has stripped the debug 
information of Glibc or just generated the debug information gdb and me can not
find them. Glibc has a horrible coding style and complicate build system, I have
no time research it in detials.

So, must has some simple solution.

## 02 Solution

Back to the problem itself after above analysis, I konw gdb need debug symbol
information and debug lineno table. So download the extra bubug information of
Glibc. Gdb can deal with this - debuging information in separate files. Also Gdb
can deal with extral souce code - specifying source diretories.

```bash
#Download the debug symbols for libc:

zet@fuck-GFW ~ $sudo apt-get install libc6-dbg

#And if you're on an x64 system debugging x86 code:

zet@fuck-GFW ~ $sudo apt-get install libc6:i386
zet@fuck-GFW ~ $sudo apt-get install libc6-dbg:i386

#Make sure you can see this file.
zet@fuck-GFW ~ $ls -al /usr/lib/debug/
total 52
drwxr-xr-x   4 root root  4096 Nov 29  2015 .
drwxr-xr-x 188 root root 36864 Aug 12 18:50 ..
drwxr-xr-x   4 root root  4096 Aug 23 19:38 lib
drwxr-xr-x   3 root root  4096 Nov 29  2015 usr
zet@fuck-GFW ~ $ls -al /usr/lib/debug/lib/
total 16
drwxr-xr-x 4 root root 4096 Aug 23 19:38 .
drwxr-xr-x 4 root root 4096 Nov 29  2015 ..
drwxr-xr-x 2 root root 4096 Aug 23 19:38 i386-linux-gnu
drwxr-xr-x 2 root root 4096 Jun  3 14:15 x86_64-linux-gnu

```

After get the Glibc debug information, only left the Glibc source code. Thanks
for Ubuntu package manager system, get source code is simple.

```bash
#apt-get source will download the source in current directory.

zet@fuck-GFW ~ $mkdir -v glibc/source/code/you/want/download
zet@fuck-GFW ~ $cd glibc/source/code/you/want/download
zet@fuck-GFW ~ $apt-get source libc6

```

After this you can see a directory named: eglibc-x.x which contans the Glibc
source.

Firt we must into gdb command environmet. After the gcc compile and output the 
binary and we need the binary has a Program Header: INTERP:

```bash
zet@fuck-GFW ~ $readelf -l hello_world | grep INTERP
  INTERP         0x0000000000000238 0x0000000000400238 0x0000000000400238

```
The simplest way is let the test code calling shared library. Well the most 
convenient method just calling the standard libc. 

So write this simple test code.

```vim
/* hello_world.c */

#include <stdio.h>

int main () {
        puts("hello, world\n");
}

```

Everything is ok, Now we need the gdb konw where is the debug information and
where is the source code.

**Specifying the debug information for gdb**

>set debug-file-directory directories

>Set the directories which gdb searches for separate debugging information files
to directory. Multiple path components can be set concatenating them by a path 
separator.
>
--[gdb official manual](https://sourceware.org/gdb/onlinedocs/gdb/Separate-Debug-Files.html#index-set-debug_002dfile_002ddirectory-1237)

```bash
zet@fuck-GFW ~ $gcc -g -o hello_world hello_world.c
zet@fuck-GFW ~ $gdb ./hello_world
(gdb) b main
(gdb) set debug-file-directory /usr/lib/debug/

#Force the gdb reload the debug information
(gdb) r
#Continue this, until you can see gdb jump through the PLT table to the Glibc 
#source
(gdb) si

#Gdb command window output like this(I use the gdb tui mode):
>|0x7ffff7df04a8 <_dl_runtime_resolve+8 at ../sysdeps/x86_64/dl-trampoline.S:37>  mov    %rcx,0x8(%rsp)                                            |
 |0x7ffff7df04ad <_dl_runtime_resolve+13 at ../sysdeps/x86_64/dl-trampoline.S:38> mov    %rdx,0x10(%rsp)                                           |
 |0x7ffff7df04b2 <_dl_runtime_resolve+18 at ../sysdeps/x86_64/dl-trampoline.S:39> mov    %rsi,0x18(%rsp)                                           |
 |0x7ffff7df04b7 <_dl_runtime_resolve+23 at ../sysdeps/x86_64/dl-trampoline.S:40> mov    %rdi,0x20(%rsp)                                           |
 |0x7ffff7df04bc <_dl_runtime_resolve+28 at ../sysdeps/x86_64/dl-trampoline.S:41> mov    %r8,0x28(%rsp)                                            |
 |0x7ffff7df04c1 <_dl_runtime_resolve+33 at ../sysdeps/x86_64/dl-trampoline.S:42> mov    %r9,0x30(%rsp)                                            |
 |0x7ffff7df04c6 <_dl_runtime_resolve+38 at ../sysdeps/x86_64/dl-trampoline.S:43> mov    0x40(%rsp),%rsi                                           |
 |0x7ffff7df04cb <_dl_runtime_resolve+43 at ../sysdeps/x86_6yy4/dl-trampoline.S:44> mov    0x38(%rsp),%rdi                                         |
 |0x7ffff7df04d0 <_dl_runtime_resolve+48 at ../sysdeps/x86_64/dl-trampoline.S:45> callq  0x7ffff7de9430 <_dl_fixup at ../elf/dl-runtime.c:66>      |

```

**Specifying the source code for gdb**

>directory dirname ...

>dir dirname ...

>Add directory dirname to the front of the source path. Several directory names
may be given to this command, separated by ‘:’ (‘;’ on MS-DOS and MS-Windows, 
where ‘:’ usually appears as part of absolute file names) or whitespace. You may
specify a directory that is already in the source path; this moves it forward,
so gdb searches it sooner.

>You can use the string ‘$cdir’ to refer to the compilation directory (if one is
recorded), and ‘$cwd’ to refer to the current working directory. ‘$cwd’ is not 
the same as ‘.’—the former tracks the current working directory as it changes 
during your gdb session, while the latter is immediately expanded to the current
directory at the time you add an entry to the source path.  
>
--[gdb official manual](https://sourceware.org/gdb/onlinedocs/gdb/Source-Path.html#index-dir-550)

Pay attention to the above output list after we have force gdb reload the debug
information. The gdb is first into parent directory and then *sysdeps/*. So
specify the source code directory for gdb must follow this rules.

```bash
 |0x7ffff7df04c1 <_dl_runtime_resolve+33 at ../sysdeps/x86_64/dl-trampoline.S:42> mov    %r9,0x30(%rsp)                                                      |

```

```gdb
#Any sub-directory of eglibc-x.x is fine.
(gdb) dir /glibc/source/code/you/want/download/eglibc-x.x/elf
#Add the break point of entry point of dynamic-linker.
#Alternative you can create a watch point of GOT item for the called shared
# library function of the test binary.
(gdb) b _start
(gdb) r

```
Until now, you must can debug the dynamic-linker of Glibc.

## 03 Epilogue

All of the content of this article described is only my persional experiment, 
but depend on my analysis I think you can deal with your maybe special problem.

Good luck.

*live long and prosper*

