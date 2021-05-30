---
layout:         post
title:          shared library wrinkle
data:           2016-06-30
auther:         zet
mail:           zet@gmail.com
summary: 
categories:     GNU Linux-Security
---

# shared library怎样实现对象的单一定义原则
@(ELF | linker | runtime)[shared library]
        --[zet](https://github.com/fanfuqiang)

## 00 导引

前段时间在查看shared library的一段汇编代码的时候发现一个非常有趣的reloc type: 
R_386_COPY,这个类型出现在当一个main executable引用shared library里面导出的
globals的值时.然后又看到LLVM/lld里面的R_386_COPY的实现[patch](https://reviews.llvm.org/D14958)
在代码*lld/trunk/ELF/Target.cpp*: 203行有一个赋值,R_386_COPY的核心实现就是这么简
单.但是背后涉及到的细节并没有这么简单.

还有另外一个问题就是在一个main executable和各个链接的shared library里面如果取函
数地址,得到的是什么? binutils/ld和glibc/dynamic-linker如何保证全局的类型函数指针
的变量如果赋值为同一个函数,比较它们的值会相等? 这个问题粗看很简单,但是考虑如果某
个类型是函数指针的全局变量的赋值发生在一个shared library里面,取地址的函数是定义
另外一个library里或者executable里,注意定义于shared library的函数的地址是运行时才
知道的地址,binutils/ld并不知道.所以显然这个shared library里的类型是函数指针的全
局变量得到的地址并不是真实的函数地址.如果是定义于main executable里的函数地址可以
由binutils/ld确定,所以函数地址可以是函数的真实地址.另外一种特别有趣的情况是: 如
果函数定义于shared library里而全局变量却位于main executable,这个变量得到的是什么
值?这里先给出答案: 得到的是定义于shared library里的函数在main executable里PLT表
项的地址.

本文的描述将围绕这两个问题来进行.这两个问题的解决的实现代码位于glibc,所以源代码
的也来自于glibc.对于如何搭建glibc调试环境详细见我的前一篇[文章](http://hardenedlinux.org/toolchains/2016/08/25/build_debug_environment_for_dynamic_linker_of_glibc.html).

## 01 引申

根据*导引*部分给出的问题描述,其中一个很核心的问题是: 当execuatable和library取另
外的shared library里的函数地址的时候,在shared library里,因为函数的实际地址只有运
行时才能知道,函数的调用是通过PLT/GOT来进行的,所以最朴素的实现方式应该是返回函数
对应的**PLT**条目的地址,但是因为**每一个**library里面如果引用同一个函数都会有一
个独立于当前library的PLT条目,这样就会导致其实不同的library取得的相同的函数的地址
并不相同.但是做比较的时候按照ISO C标准里的定义,同一个函数或对象的地址应该是相等
的.这个问题的解决需要binutils/ld和glibc/dynamic-linker配合来解决.

> 6.5.9 Equality operators
Paragraph 6
Two pointers compare equal if and only if both are null pointers, both are 
pointers to the same object (including a pointer to an object and a subobject at
its beginning) or function
>
>--[ISO C11 standard]

## 02 示例

本文所涉及到的测试代码如下(为了缩短篇幅,删去了一些extern声明代码):

```
liba.c

```

```
// exported globals
int lib_global = 1;
int *a_ptr;
typedef void ( *func_ptr_type) ();
func_ptr_type a_fptr;

void stub () {
        return;
}

void liba_init () {
        a_ptr = &lib_global;
        a_fptr = stub;
        lib_global = 3;
}
```

```
libb.c

```

```
volatile int lib_flag = 1;
int *b_ptr;
typedef void ( *func_ptr_type) ();
func_ptr_type b_fptr;

static void libb_init () {
        b_ptr = &lib_global;
        b_fptr = stub;
} 

void libb_test () {
        libb_init();

        if (a_fptr == b_fptr)
                lib_flag = 2;
        if (a_ptr == b_ptr)
                lib_flag = 3;
}

```

```
mian.c

```

```
#include <stdio.h>

typedef void ( *func_ptr_type) ();
int *main_ptr;
func_ptr_type main_fptr;

int main () {
        main_ptr = &lib_global;
        main_fptr = stub;

        stub();
        // liba
        liba_init();
        // libb
        libb_test();

        if (main_ptr == b_ptr)
                lib_flag = 4;
        if (main_fptr == b_fptr)
                lib_flag = 5;

        puts("hello\n");
}

```

build方式为:

``` bash
gcc -m32 -fpic -g -o liba.so liba.c
gcc -m32 -fpic -g -o libb.so libb.c ./liba.so
gcc -m32 -g -o main main.c ./liba.so ./libb.so

```

## 03 问题展开

上面的测试代码是由名字为main的测试binary来驱动的.

```
        main_ptr = &lib_global;
        // 对应的汇编代码
        movl   $0x804a034,0x804a03c
 
        main_fptr = stub;
        // 对应的汇编代码
        movl   $0x8048520,0x804a040
        
        stub();
        //
        call   0x8048520 <stub@plt>

        liba_init();
        //
        call   0x8048510 <liba_init@plt> 
        
        libb_test();
        //
        call   0x8048540 <libb_test@plt>
        
        if (main_ptr == b_ptr)
        //
        mov    0x804a03c,%edx
        mov    0x804a028,%eax
        cmp    %eax,%edx
        jne    80486b2 <main+0x45>
 
                lib_flag = 4;
                //
                movl   $0x4,0x804a02c

        if (main_fptr == b_fptr)
        //
        mov    0x804a040,%edx
        mov    0x804a030,%eax
        cmp    %eax,%edx
        jne    80486cb <main+0x5e>

                lib_flag = 5;
                //
                movl   $0x5,0x804a02c

```

```bash

zet@fuck-GFW ~/dust/test $readelf -r main

Relocation section '.rel.dyn' at offset 0x484 contains 5 entries:
 Offset     Info    Type            Sym.Value  Sym. Name
08049ffc  00000506 R_386_GLOB_DAT    00000000   __gmon_start__
0804a028  00001105 R_386_COPY        0804a028   b_ptr
0804a02c  00000b05 R_386_COPY        0804a02c   lib_flag
0804a030  00001305 R_386_COPY        0804a030   b_fptr
0804a034  00000f05 R_386_COPY        0804a034   lib_global

Relocation section '.rel.plt' at offset 0x4ac contains 5 entries:
 Offset     Info    Type            Sym.Value  Sym. Name
0804a00c  00000207 R_386_JUMP_SLOT   00000000   liba_init
0804a010  00000c07 R_386_JUMP_SLOT   08048520   stub
0804a014  00000307 R_386_JUMP_SLOT   00000000   puts@GLIBC_2.0
0804a018  00000407 R_386_JUMP_SLOT   00000000   libb_test
0804a01c  00000607 R_386_JUMP_SLOT   00000000   __libc_start_main@GLIBC_2.0

```

```
Section Headers:
  [Nr] Name              Type            Addr     Off    Size   ES Flg Lk Inf Al
  [ 0]                   NULL            00000000 000000 000000 00      0   0  0
  [...]
  [ 9] .rel.dyn          REL             08048484 000484 000028 08   A  5   0  4
  [10] .rel.plt          REL             080484ac 0004ac 000028 08  AI  5  24  4
  [12] .plt              PROGBITS        08048500 000500 000060 04  AX  0   0 16
  [13] .plt.got          PROGBITS        08048560 000560 000008 00  AX  0   0  8
  [14] .text             PROGBITS        08048570 000570 0001e2 00  AX  0   0 16
  [22] .dynamic          DYNAMIC         08049f04 000f04 0000f8 08  WA  6   0  4
  [23] .got              PROGBITS        08049ffc 000ffc 000004 04  WA  0   0  4
  [24] .got.plt          PROGBITS        0804a000 001000 000020 04  WA  0   0  4
  [25] .data             PROGBITS        0804a020 001020 000008 00  WA  0   0  4
  [26] .bss              NOBITS          0804a028 001028 00001c 00  WA  0   0  4
  [...]

```
需要注意的main executable里定义或者引用的object的地址列表如下:

| identifier | address   | section |
| :--------- | ---------:| :-----: |
| main_ptr   | 0x804a03c | .bss    |
| main_fptr  | 0x804a040 | .bss    |
| lib_global | 0x804a034 | .bss    |
| stub       | 0x8048520 | .plt    |
| b_ptr      | 0x804a028 | .bss    |
| b_fptr     | 0x804a030 | .bss    |
| lib_flag   | 0x804a02c | .bss    |

在上图中main_ptr/main_fptr是定义于main executable里的object,其他的都是对shared 
library里面定义的引用,stub是我们的重点研究对象,既对其调用也取其地址,当然也是定义
于library里.由上表可以看到一个非常有趣的现象: 那就是对于定义于library里的object
的引用在当前executable里也是要在.bss里存在副本,也就是说: 这个对象在当前进程里会
有**多于一个的实体存在**.也要注意其重定位类型是: R_386_COPY.

而stub()的处理更加特殊,注意stub涉及到的重定位项.

*定义于[IA32 psABI](https://github.com/hjl-tools/x86-psABI/wiki/X86-psABI) Table 3.6: Relocation Types*里的重定位类型R_386_GLOB_DAT和
R_386_JMP_SLOT,glibc/dynamic-linker对于他们的重定位计算方法是一样的就是: 在reloc
offset处写入根据reloc找到的symbol的值.在binutils/ld产生这两种重定位类型
(R_386_GLOB_DAT/R_386_JMP_SLOT)时,区别就在于是否是函数调用.*

在main里面我们引用了library里面的几个函数,其中有liba_init和stub,但是我们对stub进
行了取地址运算.为了比较方便重新拷贝到下面.

0804a00c  00000207 R_386_JUMP_SLOT   00000000   liba_init
0804a010  00000c07 R_386_JUMP_SLOT   08048520   stub

仔细对比上面的内容,最开始的两项是reloc的offset,这两个值都是位于.got里,第二项保存
了两类信息: 在符号表里的索引和重定位类型,第三项重定位类型的可读表示来自于第二项,
第五项是符号名称很明显,第四项是整个问题的关键,按照ELF符号的定义来说这两个函数都
是当前executable的引用library里的函数,这里的符号值应该是STN_UNDEF(这个值在ELF里
是0),但是stub的符号值是0x8048520而且作为了取stub函数地址的返回值,但是注意这个值
表示的地址位于.plt里的stub@plt也就是stub函数的PLT项.

函数的PLT项主要的是为了对其进行首次调用时,为调用dynamic linker提供一个参数(重定
位项的offset).然后再压入_GLOBAL_OFFSET_TABLE_[1]以这个stack直接jmp调用
_GLOBAL_OFFSET_TABLE_[2].那么_GLOBAL_OFFSET_TABLE_是什么呢?

**_GLOBAL_OFFSET_TABLE_的初始化**

_GLOBAL_OFFSET_TABLE_表的前三项是特殊值,_GLOBAL_OFFSET_TABLE_[0]是section
.dynamic的地址,是由linker在创建.got/.plt等动态链接section的时候写入的,
_GLOBAL_OFFSET_TABLE_[1]的值是定义于glibc中的类型为指向struct link_map的指针,根
据这个指针动态连接器可以找到前期运行时所创建的所有信息._GLOBAL_OFFSET_TABLE_[2]
的值是一个函数指针: _dl_runtime_reslove(),解决所有application运行时的需要的重定
位.

_GLOBAL_OFFSET_TABLE_[1]/[2]的初始化是为什么kernel在进行do_execeve()运行一个新的
application的时候如果发现这个app有一个类型是PT_INTERP的program header则先调用这
个header的内容(对于linux来说是ld-linux.so.2也就是动态连接器)的一个很重要的理由.

_GLOBAL_OFFSET_TABLE_[1]/[2]的初始化代码位于:

```
glibc-2.23/sysdeps/i386/dl_machine.h
```

```
/* Set up the loaded object described by L so its unrelocated PLT
   entries will jump to the on-demand fixup code in dl-runtime.c.  */

static inline int __attribute__ ((unused, always_inline))
elf_machine_runtime_setup (struct link_map *l, int lazy, int profile)
{
  Elf32_Addr *got;
  extern void _dl_runtime_resolve (Elf32_Word) attribute_hidden;
  extern void _dl_runtime_profile (Elf32_Word) attribute_hidden;

  if (l->l_info[DT_JMPREL] && lazy)
    {
      /* The GOT entries for functions in the PLT have not yet been filled
	 in.  Their initial contents will arrange when called to push an
	 offset into the .rel.plt section, push _GLOBAL_OFFSET_TABLE_[1],
	 and then jump to _GLOBAL_OFFSET_TABLE[2].  */
      got = (Elf32_Addr *) D_PTR (l, l_info[DT_PLTGOT]);
      /* If a library is prelinked but we have to relocate anyway,
	 we have to be able to undo the prelinking of .got.plt.
	 The prelinker saved us here address of .plt + 0x16.  */
      if (got[1])
	{
	  l->l_mach.plt = got[1] + l->l_addr;
	  l->l_mach.gotplt = (Elf32_Addr) &got[3];
	}
      //
      // **_GLOBAL_OFFSET_TABLE_[1]的初始化**
      //
      got[1] = (Elf32_Addr) l;	/* Identify this shared object.  */

      /* The got[2] entry contains the address of a function which gets
	 called to get the address of a so far unresolved function and
	 jump to it.  The profiling extension of the dynamic linker allows
	 to intercept the calls to collect information.  In this case we
	 don't store the address in the GOT so that all future calls also
	 end in this function.  */
      if (__glibc_unlikely (profile))
	{
	  got[2] = (Elf32_Addr) &_dl_runtime_profile;

	  if (GLRO(dl_profile) != NULL
	      && _dl_name_match_p (GLRO(dl_profile), l))
	    /* This is the object we are looking for.  Say that we really
	       want profiling and the timers are started.  */
	    GL(dl_profile_map) = l;
	}
      else
	/* This function will get called to fix up the GOT entry indicated by
	   the offset on the stack, and then jump to the resolved address.  */
     
        //
        // **_GLOBAL_OFFSET_TABLE_[2]的初始化**
        //
	got[2] = (Elf32_Addr) &_dl_runtime_resolve;
    }

  return lazy;
}

```

_dl_runtim_reslove的代码是一段跟处理器有关系的汇编.


```
glibc-2.23/sysdeps/i386/dl-trampoline.S

```

```
	.text
	.globl _dl_runtime_resolve
	.type _dl_runtime_resolve, @function
	cfi_startproc
	.align 16
_dl_runtime_resolve:
	cfi_adjust_cfa_offset (8)
	pushl %eax		# Preserve registers otherwise clobbered.
	cfi_adjust_cfa_offset (4)
	pushl %ecx
	cfi_adjust_cfa_offset (4)
	pushl %edx
	cfi_adjust_cfa_offset (4)
	movl 16(%esp), %edx	# Copy args pushed by PLT in register.  Note
	movl 12(%esp), %eax	# that fixup takes its parameters in regs.
	call _dl_fixup		# Call resolver.
	popl %edx		# Get register content back.
	cfi_adjust_cfa_offset (-4)
	movl (%esp), %ecx
	movl %eax, (%esp)	# Store the function address.
	movl 4(%esp), %eax
	ret $12			# Jump to function address.
	cfi_endproc
	.size _dl_runtime_resolve, .-_dl_runtime_resolve

```
可以看出这段代码主要的工作就是处理ELF里的PLT代码压入的参数,一个是重定位项的
offset,另外一个就是dynamic linker做初始化工作时写入的_GLOBAL_OFFSET_TABLE_[1]的
值(也就是指向struct link_map指针).

*_dl_runtime_reslove处理的是运行时对于.plt及其相关的sections的重定位并不在本文的
描述范围*

## 04 Glibc/dynamic-linker的解决方案

*下面的描述将会以dl指代glibc/dynamic-linker*
其实在本文最开始提出的两个问题glibc的解决及其实现代码,位于glibc/dynamic-linker在
处理完自身的bootstrap之后在将控制权交给main executable之前.因为在executable运行
之前它自己的.got以及依赖的library的.got需要是重定位已经完成的.本文描述的函数指针
和R_386_COPY不存在函数的调用,所以重定位offset不会位于跟.plt相关的sections里面(这
些section的重定位是dl运行时也就是在main executable获得控制权之后解决的)里.

需要描述清楚dl对于本文提出的两个问题的解决方案,一个本质的问题就是当遇到重定位的
需求时,根据reloc里的保存的符号索引查找符号定义,dl查找的文件范围和顺序是什么?

**dl符号定义查找集合的构建**

在dl得到kernel给予的控制权之后首先运行的是由**_start**标识的一段汇编代码.

```
glibc-src/sysdeps/i386/dl-machine.h
```

```

.globl _start\n\
.globl _dl_start_user\n\
_start:\n\
	# Note that _dl_start gets the parameter in %eax.\n\
	movl %esp, %eax\n\
	call _dl_start\n\


```
然后进入**_dl_start()**初始化一些dl bootstrap的数据结构,并且处理dl本身的重定位,dl本
身是一个自包含没有依赖其他library的shared object,见过一些文章说是dl是staticlly 
linked,我认为并不是.

```bash

zet@fuck-GFW ~ $ldd /lib/i386-linux-gnu/ld-2.19.so
	statically linked
zet@fuck-GFW ~ $readelf -d /lib/i386-linux-gnu/ld-2.19.so

Dynamic section at offset 0x1ff30 contains 19 entries:
  Tag        Type                         Name/Value
 0x0000000e (SONAME)                     Library soname: [ld-linux.so.2]
 0x00000004 (HASH)                       0x138
 0x6ffffef5 (GNU_HASH)                   0x1f8
 0x00000005 (STRTAB)                     0x4ac
 0x00000006 (SYMTAB)                     0x2dc
 0x0000000a (STRSZ)                      406 (bytes)
 0x0000000b (SYMENT)                     16 (bytes)
 0x00000003 (PLTGOT)                     0x21000
 0x00000002 (PLTRELSZ)                   48 (bytes)
 0x00000014 (PLTREL)                     REL
 0x00000017 (JMPREL)                     0x7b4
 0x00000011 (REL)                        0x744
 0x00000012 (RELSZ)                      112 (bytes)
 0x00000013 (RELENT)                     8 (bytes)
 0x6ffffffc (VERDEF)                     0x67c
 0x6ffffffd (VERDEFNUM)                  6
 0x6ffffff0 (VERSYM)                     0x642
 0x6ffffffa (RELCOUNT)                   11
 0x00000000 (NULL)                       0x0
zet@fuck-GFW ~ $readelf -S /lib/i386-linux-gnu/ld-2.19.so | grep .dynamic
  [16] .dynamic          DYNAMIC         00020f30 01ff30 0000b8 08  WA  5   0  4

```
注意不要被ldd的输出所误导,readelf -d /lib/i386-linux-gnu/ld-2.19.so的输出里有内
容但是没有ND_NEEDED,所以dl并不是一个staticlly linked binary.

在_dl_start()里会进行dl bootstrap最重要的调用以解决dl自身的重定位.

```
  if (bootstrap_map.l_addr || ! bootstrap_map.l_info[VALIDX(DT_GNU_PRELINKED)])
    {
      /* Relocate ourselves so we can do normal function calls and
	 data access using the global offset table.  */
      //
      // 解决dl的重定位
      //
      ELF_DYNAMIC_RELOCATE (&bootstrap_map, 0, 0, 0);
    }
  bootstrap_map.l_relocated = 1;

```

然后进入**_dl_start_final()**,_dl_start_final做一些跟dl本身相关的数据赋值,然后调
用**dl_sysdep_start()**读取kernel写入辅助数组(包含kernel提供数据,比如uid和页容量
以及用户程序入口等等),然后进入**dl_main()**.

```
  start_addr = _dl_sysdep_start (arg, &dl_main);

```

**dl_main()**是真正的处理用户代码的入口.所有剩余操作都是由dl_main来驱动完成的.

```
glibc-src/elf/rtld.c
```

```
      /* Process the environment variable which control the behaviour.  */
      //
      // 这个函数处理各种影响到executable的环境变量
      //
      process_envvars (&mode);

      /* Create a link_map for the executable itself.
	 This will be what dlopen on "" returns.  */

      //
      // 创建main executable的struct link_map
      //
      main_map = _dl_new_object ((char *) "", "", lt_executable, NULL,
				 __RTLD_OPENEXEC, LM_ID_BASE);
      assert (main_map != NULL);
      main_map->l_phdr = phdr;
      main_map->l_phnum = phnum;
      main_map->l_entry = *user_entry;

```

**process_envvars()**跟dl_main()位于同一个文件里,我们感兴趣的代码主要是对于
LD_PRELOAD的处理.

```
          /* List of objects to be preloaded.  */
	  if (memcmp (envline, "PRELOAD", 7) == 0)
	    {
              //
              // 记录LD_PTRLOAD信息
              //
	      preloadlist = &envline[8];
	      break;
	    }

```

dl_main()接着的代码就是一系列的读取elf文件以及初始化library的搜索目录.注意这是
依赖library的搜索目录集合并不是本文关注的主要关注点: symbol定义的搜索集合.

library的搜索集合的构建是调用函数**_dl_init_paths()**,构建的过程跟
DT_RPATH/DT_RUNPATH以及环境变量LD_LIBRARY_PATH有关系,感兴趣的读者自己阅读相关代
码.或者*man ld.so*参考手册.

继续在dl_main()里的跟本文相关的代码的分析.

```
  /* We have two ways to specify objects to preload: via environment
     variable and via the file /etc/ld.so.preload.  The latter can also
     be used when security is enabled.  */
  assert ( *first_preload == NULL);
  //
  // 保存preload library的link_map的指针数组
  //
  struct link_map **preloads = NULL;
  unsigned int npreloads = 0;

  if (__builtin_expect (preloadlist != NULL, 0))
    {
      /* The LD_PRELOAD environment variable gives list of libraries
	 separated by white space or colons that are loaded before the
	 executable's dependencies and prepended to the global scope
	 list.  If the binary is running setuid all elements
	 containing a '/' are ignored since it is insecure.  */
      char *list = strdupa (preloadlist);
      char *p;

      HP_TIMING_NOW (start);

      /* Prevent optimizing strsep.  Speed is not important here.  */
      while ((p = (strsep) (&list, " :")) != NULL)
	if (p[0] != '\0'
	    && (__builtin_expect (! INTUSE(__libc_enable_secure), 1)
		|| strchr (p, '/') == NULL))
          //
          // do_preload()会打开每一个preload library然后构建对应的link_map结构
          // 然后保存在上面的link_map **preloads里
          //
	  npreloads += do_preload (p, main_map, "LD_PRELOAD");

      HP_TIMING_NOW (stop);
      HP_TIMING_DIFF (diff, start, stop);
      HP_TIMING_ACCUM_NT (load_time, diff);
    }

  /* There usually is no ld.so.preload file, it should only be used
     for emergencies and testing.  So the open call etc should usually
     fail.  Using access() on a non-existing file is faster than using
     open().  So we do this first.  If it succeeds we do almost twice
     the work but this does not matter, since it is not for production
     use.  */
  //
  // 正如注释所写的,一般时候并没有*/etc/ld.so.preload*文件,就算是有也不会影响本
  // 文主题的描述.所以略过处理代码.
  //
  static const char preload_file[] = "/etc/ld.so.preload";
  if (__builtin_expect (__access (preload_file, R_OK) == 0, 0))
    {
      ...
    }

  //
  // ...
  //

  /* Load all the libraries specified by DT_NEEDED entries.  If LD_PRELOAD
     specified some libraries to load, these are inserted before the actual
     dependencies in the executable's searchlist for symbol resolution.  */
  HP_TIMING_NOW (start);
  //
  // 这个函数构建symbol搜索集合的也就是本文描述内容的核心部分.
  //
  _dl_map_object_deps (main_map, preloads, npreloads, mode == trace, 0);

```

代码位置是:

```
glibc-src/elf/dl-deps.c/_dl_map_object_deps()
```
注意调用时的第一个实参main_map是用户代码main executable对应的struct link_map.

这个代码太长了,实际的实现也比较复杂.不过主要就是在已经有的link_map指针数组
*preloads*的后面再添加executable的DT_NEEDED保存的library按照出现顺序构建一个列
表.这个列表的先后顺序是有关系的.先出现的symbol搜索时会先搜索.所以不看代码也应该
推测到列表中的第一个肯定是main executable的link_map.

```
link_map **main_map->l_searchlist.r_list
        |
        |
        |
        | main_map             npreloads                  nneededs
        |      |                  |                          |
        |      |      +-------------------+      +-----------------------+
        |      |      |                   |      |                       |
        |     +--+   +--+   +--+         +--+   +--+   +--+            +--+
        +---->|  |-->|  |-->|  |-------->|  |-->|  |-->|  |----------->|  |
        +---->|  |<--|  |<--|  |<--------|  |<--|  |<--|  |<-----------|  |
        |     +--+   +--+   +--+         +--+   +--+   +-++            +--+
        |              |      |            |      |      |               |
        |              |      |            |      |      |               |
        |              |      |            |      |      |               |
        |         preload_1   |            |   needed_1  |               |
        |                  preload_2       |           needed_2          |
        |                              preload_n                    needed_n
        |
        |
preloads/neededs->l_scope[0]->r_list

```
这就是_dl_map_object_deps()函数构建的symbol查找时的binary/library的查找顺序表.由
表可知symbol定义查找时,永远都是先进入main executable文件.

将注意力继续回到dl_main(),构建了symbol的查找顺序表之后下面开始解决重定位.

```
      unsigned i = main_map->l_searchlist.r_nlist;
      while (i-- > 0)
	{
          //
          // 在一般情况下l_initfinit的和l_searchlist.r_list是一样的.也就是遍历的
          // 依旧是上面的那个查找顺序表.
          //
	  struct link_map *l = main_map->l_initfini[i];

	  /* While we are at it, help the memory handling a bit.  We have to
	     mark some data structures as allocated with the fake malloc()
	     implementation in ld.so.  */
	  struct libname_list *lnp = l->l_libname->next;

	  while (__builtin_expect (lnp != NULL, 0))
	    {
	      lnp->dont_free = 1;
	      lnp = lnp->next;
	    }
	  /* Also allocated with the fake malloc().  */
	  l->l_free_initfini = 0;

          //
          // dl_rtld_map就是dynamic-linker本身的link_map指针,在本文的情况下l应该
          // 是指向executable.
          //
	  if (l != &GL(dl_rtld_map))i
            //
            // 调用解决重定位的函数
            //
	    _dl_relocate_object (l, l->l_scope, GLRO(dl_lazy) ? RTLD_LAZY : 0,
				 consider_profiling);

	  /* Add object to slot information data if necessasy.  */
	  if (l->l_tls_blocksize != 0 && tls_init_tp_called)
	    _dl_add_to_slotinfo (l);
	}
 
```

**_dl_relocate_object()**函数是解决executable及其依赖library的重定位的入口,流程
基本就是遍历reloc table查找symbol定义根据[psABI](https://github.com/hjl-tools/x86-psABI/wiki/X86-psABI)里定义的reloc计算方法来解决重定
位.

```
glibc-src/elf/dl-reloc.c/_dl_relocate_object()
```

```
  //
  // 略去跟本文无关系的部分代码
  //

    /* This macro is used as a callback from the ELF_DYNAMIC_RELOCATE code.  */
    //
    // 这个macro是一个回调的macro,并不在本函数调用
    //
#define RESOLVE_MAP(ref, version, r_type) \
    (ELFW(ST_BIND) ((*ref)->st_info) != STB_LOCAL			      \
     ? ((__builtin_expect ((*ref) == l->l_lookup_cache.sym, 0)		      \
	 && elf_machine_type_class (r_type) == l->l_lookup_cache.type_class)  \
	? (bump_num_cache_relocations (),				      \
	   (*ref) = l->l_lookup_cache.ret,				      \
	   l->l_lookup_cache.value)					      \
	: ({ lookup_t _lr;						      \
	     int _tc = elf_machine_type_class (r_type);			      \
	     l->l_lookup_cache.type_class = _tc;			      \
	     l->l_lookup_cache.sym = (*ref);				      \
	     const struct r_found_version *v = NULL;			      \
	     if ((version) != NULL && (version)->hash != 0)		      \
	       v = (version);						      \
            //
            // 这个函数是symbol定义查找的入口,实参scope保存了上面建立的查找表
            //
	     _lr = _dl_lookup_symbol_x (strtab + (*ref)->st_name, l, (ref),   \
					scope, v, _tc,			      \
					DL_LOOKUP_ADD_DEPENDENCY, NULL);      \
	     l->l_lookup_cache.ret = (*ref);				      \
	     l->l_lookup_cache.value = _lr; }))				      \
     : l)

#include "dynamic-link.h"

    //
    // 重定位的实际入口
    //
    ELF_DYNAMIC_RELOCATE (l, lazy, consider_profiling, skip_ifunc);

```

```
glibc-src/elf/dynamic-link.h/ELF_DYNAMIC_RELOCATE
```
```
/* This can't just be an inline function because GCC is too dumb
   to inline functions containing inlines themselves.  */
# define ELF_DYNAMIC_RELOCATE(map, lazy, consider_profile, skip_ifunc) \
  do {									      \
    int edr_lazy = elf_machine_runtime_setup ((map), (lazy),		      \
					      (consider_profile));	      \
    //
    // 在i386的机器上reloc的addend就是存在于根据reloc的offset计算出来的地址处,所
    // 以只有ELF_DYNAMIC_DO_REL,ELF_DYNAMIC_DO_RELA无定义.
    //
    ELF_DYNAMIC_DO_REL ((map), edr_lazy, skip_ifunc);			      \
    ELF_DYNAMIC_DO_RELA ((map), edr_lazy, skip_ifunc);			      \
  } while (0)

//
// ELF_DYNAMIC_DO_REL的定义也存在于dynamic-link.h这个文件里
// RELOC和reloc都是跟机器是不是有单独的addend有关系的.这个宏的实际效果就是遍历
// reloc表,然后调用elf_dynamic_do_##reloc(经过宏扩张将会调用elf_dynamic_do_Rel)
// 来进行实际的重定位修正.
//
# define _ELF_DYNAMIC_DO_RELOC(RELOC, reloc, map, do_lazy, skip_ifunc, test_rel) \
  do {									      \
    struct { ElfW(Addr) start, size;					      \
	     __typeof (((ElfW(Dyn) *) 0)->d_un.d_val) nrelative; int lazy; }  \
      ranges[2] = { { 0, 0, 0, 0 }, { 0, 0, 0, 0 } };			      \
									      \
    if ((map)->l_info[DT_##RELOC])					      \
      {									      \
	ranges[0].start = D_PTR ((map), l_info[DT_##RELOC]);		      \
	ranges[0].size = (map)->l_info[DT_##RELOC##SZ]->d_un.d_val;	      \
	if (map->l_info[VERSYMIDX (DT_##RELOC##COUNT)] != NULL)		      \
	  ranges[0].nrelative						      \
	    = MIN (map->l_info[VERSYMIDX (DT_##RELOC##COUNT)]->d_un.d_val,    \
		   ranges[0].size / sizeof (ElfW(reloc)));		      \
      }									      \
    if ((map)->l_info[DT_PLTREL]					      \
	&& (!test_rel || (map)->l_info[DT_PLTREL]->d_un.d_val == DT_##RELOC)) \
      {									      \
	ElfW(Addr) start = D_PTR ((map), l_info[DT_JMPREL]);		      \
	ElfW(Addr) size = (map)->l_info[DT_PLTRELSZ]->d_un.d_val;	      \
									      \
	if (ranges[0].start + ranges[0].size == (start + size))		      \
	  ranges[0].size -= size;					      \
	if (! ELF_DURING_STARTUP && ((do_lazy) || ranges[0].size == 0))	      \
	  {								      \
	    ranges[1].start = start;					      \
	    ranges[1].size = size;					      \
	    ranges[1].lazy = (do_lazy);					      \
	  }								      \
	else								      \
	  {								      \
	    /* Combine processing the sections.  */			      \
	    ranges[0].size += size;					      \
	  }								      \
      }									      \
									      \
    if (ELF_DURING_STARTUP)						      \
      elf_dynamic_do_##reloc ((map), ranges[0].start, ranges[0].size,	      \
			      ranges[0].nrelative, 0, skip_ifunc);	      \
    else								      \
      {									      \
	int ranges_index;						      \
	for (ranges_index = 0; ranges_index < 2; ++ranges_index)	      \
	  elf_dynamic_do_##reloc ((map),				      \
				  ranges[ranges_index].start,		      \
				  ranges[ranges_index].size,		      \
				  ranges[ranges_index].nrelative,	      \
				  ranges[ranges_index].lazy,		      \
				  skip_ifunc);				      \
      }									      \
  } while (0)

```

**elf_dynamic_do_Rel()**做的最主要的事情就是调用**elf_machine_rel()**这是实际的
解决机器相关的reloc的函数.

```
glibc-src/sysdeps/i386/dl-machine.h
```

```
/* Perform the relocation specified by RELOC and SYM (which is fully resolved).
   MAP is the object containing the reloc.  */

auto inline void
__attribute ((always_inline))
elf_machine_rel (struct link_map *map, const Elf32_Rel *reloc,
		 const Elf32_Sym *sym, const struct r_found_version *version,
		 void *const reloc_addr_arg, int skip_ifunc)
{
  Elf32_Addr *const reloc_addr = reloc_addr_arg;
  const unsigned int r_type = ELF32_R_TYPE (reloc->r_info);

      //
      // RESOLVE_MAP是symbol定义查找的实际代码,决定了找到的是哪个symbol
      //
      struct link_map *sym_map = RESOLVE_MAP (&sym, version, r_type);
      Elf32_Addr value = sym_map == NULL ? 0 : sym_map->l_addr + sym->st_value;

      if (sym != NULL
	  && __builtin_expect (ELFW(ST_TYPE) (sym->st_info) == STT_GNU_IFUNC,
			       0)
	  && __builtin_expect (sym->st_shndx != SHN_UNDEF, 1)
	  && __builtin_expect (!skip_ifunc, 1))
	value = ((Elf32_Addr (*) (void)) value) ();

      switch (r_type)
	{
        //
        // 还记得前面说过psABI里面对于这两种类型的计算方式是一样的吗?
        //
	case R_386_GLOB_DAT:
	case R_386_JMP_SLOT:
	  *reloc_addr = value;
	  break;

	case R_386_PC32:
	  *reloc_addr += (value - (Elf32_Addr) reloc_addr);
	  break;
	case R_386_COPY:
	  if (sym == NULL)
	    /* This can happen in trace mode if an object could not be
	       found.  */
	    break;
	  if (__builtin_expect (sym->st_size > refsym->st_size, 0)
	      || (__builtin_expect (sym->st_size < refsym->st_size, 0)
		  && GLRO(dl_verbose)))
	    {
	      const char *strtab;

	      strtab = (const char *) D_PTR (map, l_info[DT_STRTAB]);
	      _dl_error_printf ("\
%s: Symbol `%s' has different size in shared object, consider re-linking\n",
				RTLD_PROGNAME, strtab + refsym->st_name);
	    }

          //
          // 注意这里的memcpy的调用,根据前面描述的查找和解决重定位的时候的处理次
          // 序,以及最开始readelf读取main executable给出的输出,在shared library
          // 里定义的globals在main executable里的.bss段里有一个副本,也就是说同样
          // 的一个global在运行时有两个副本存在于进程空间.只不过shared library里
          // 面的副本存在的最重要的理由就是如有初始化发生在shared library里,那么
          // 初始化至是保存在shared library的副本里的,这也就是R_386_COPY这个
          // reloc类型的本意.就是一个初始值的拷贝.根据上面的分析第一个实参应该是
          // main executable里面的地址.而value应该是symbol定义查找时找到symbol的
          // 值.有意思的是后面的对于定义于shared library里的global的操作,操作的
          // 实体都会是main executable里.bss里的实体,甚至是shared library本身的
          // 代码对本身定义的global的操作.这个模型可以工作的根本原因是shared 
          // library里对本身定义的global的操作也需要引入reloc来解决.
          //
	  memcpy (reloc_addr_arg, (void *) value,
		  MIN (sym->st_size, refsym->st_size));
	  break;

	default:
	  _dl_reloc_bad_type (map, r_type, 0);
	  break;
	}
    }
}

```
根据上面的代码里的描述,symbol定义的查找找到的symbol才是对reloc影响最大的步骤.
**RESOLVE_MAP**作为symbol定义查找的入口,其定义已经在出现在了我们上面的
_dl_relocate_object()函数里.其中很重要的是对于*_tc*的计算和函数
**_dl_lookup_symbol_x()**的调用.

```
//
// RESOLVE_MAP对于elf_machine_type_class的调用,入参是reloc类型.
//
int _tc = elf_machine_type_class(r_type)

```

```
glibc-src/sysdeps/generic/ldsodefs.h
```

```
#define ELF_RTYPE_CLASS_PLT 1
#ifndef DL_NO_COPY_RELOCS
//
// 在i386的机器上这个值肯定是2
//
# define ELF_RTYPE_CLASS_COPY 2
#else
# define ELF_RTYPE_CLASS_COPY 0
#endif

```

```
glibc-src/sysdeps/i386/dl-machine.h
```

```

# define elf_machine_type_class(type) \
  ((((type) == R_386_JMP_SLOT || (type) == R_386_TLS_DTPMOD32		      \
     || (type) == R_386_TLS_DTPOFF32 || (type) == R_386_TLS_TPOFF32	      \
     || (type) == R_386_TLS_TPOFF || (type) == R_386_TLS_DESC)		      \
    * ELF_RTYPE_CLASS_PLT)						      \
   | (((type) == R_386_COPY) * ELF_RTYPE_CLASS_COPY))

```

根据上面的代码和本文需要研究的reloc类型得到下面的对应关系:

|          reloc type              | _tc的值  |
| -------------------------------- | -------- |
| R_386_JMP_SLOT/TLS相关reloc type |    1     |
| R_386_COPY                       |    2     |
| others                           |    0     |

接着考察**_dl_lookup_symbol_x()**其实这个函数跟本文研究相关的也就是调用位于同一个文
件的的另外一个函数**do_lookup_x()**.

```
glibc-src/elf/dl-lookup.c/do_lookup_x()
```

```
static int
__attribute_noinline__
do_lookup_x (const char *undef_name, uint_fast32_t new_hash,
	     unsigned long int *old_hash, const ElfW(Sym) *ref,
	     struct sym_val *result, struct r_scope_elem *scope, size_t i,
	     const struct r_found_version *const version, int flags,
	     struct link_map *skip, int type_class, struct link_map *undef_map)
{
  size_t n = scope->r_nlist;
  //
  // 这个就是前文重点描述的建立的那个symbol定义的搜索列表.
  //
  struct link_map **list = scope->r_list;

  do
    {
      const ElfW(Sym) *versioned_sym = NULL;

      const struct link_map *map = list[i]->l_real;

      /* Here come the eixtra test needed for `_dl_lookup_symbol_skip'.  */
      //
      // skip是需要跳过的link_map
      //
      if (map == skip)
	continue;

      /* Don't search the executable when resolving a copy reloc.  */
      //
      // 这里是本文研究的R_386_COPY的相关的重点,type_class就是前面列表里的的_tc
      // 的值,只有在main executable里map->l_type的值才会是lt_executable,代表是
      // executable,另外的两个可能值是: lt_library和lt_loaded,前面表示是needed 
      // library后者表示运行时额外加载的shared object.
      // 那么注意这里的判断条件,如果重定位类型是R_386_COPY并且当前的link_map是
      // main executable那么跳过当前link_map,回想我们前面描述的symbol定义的查找
      // 列表,main executable永远是位于第一个,而且除了R_386_COPY之外dl是确实想要
      // 找到位于main executable里面的这个位于.bss段里的symbol的.但是如果是
      // R_386_COPY的symbol定义是出现在shared library里面而且有初始值,那么这种情
      // 况下dl因为要知道symbol的初始值,所以想要找到的位于shared library里的
      // symbol.
      //
      if ((type_class & ELF_RTYPE_CLASS_COPY) && map->l_type == lt_executable)
	continue;

      /* Do not look into objects which are going to be removed.  */
      if (map->l_removed)
	continue;

      /* If the hash table is empty there is nothing to do here.  */
      if (map->l_nbuckets == 0)
	continue;

      /* The tables for this map.  */
      const ElfW(Sym) *symtab = (const void *) D_PTR (map, l_info[DT_SYMTAB]);
      const char *strtab = (const void *) D_PTR (map, l_info[DT_STRTAB]);


      /* Nested routine to check whether the symbol matches.  */
      //
      // 当前函数do_lookup_x()的主要功能就是遍历前文描述的那个symbol定义查找表里
      // 的每一个elf的动态符号表然后,然后调用下面的这个内嵌函数check_match()来检
      // 查合法性.所以对于本文的研究来说这个函数的至关重要的.
      //
      const ElfW(Sym) *
      __attribute_noinline__
      check_match (const ElfW(Sym) *sym)
      {
	unsigned int stt = ELFW(ST_TYPE) (sym->st_info);
	assert (ELF_RTYPE_CLASS_PLT == 1);
        //
        // stub函数在main executable里的重定位表项的值
        // 0804a010  00000c07 R_386_JUMP_SLOT   08048520   stub
        //
        // 查看定义在shared library里的函数stub在main executable的符号表的值:
        // 12: 08048520     0 FUNC    GLOBAL DEFAULT  UND stub
        //
        // 注意上面的symbol值8048520是main executable里的stub@plt的值.也要注意在
        // 取函数指针的shared library里这个重定位类型是: R_386_GLOB_DAT.上面列出
        // 的重定位是位于exxcutable的.plt里面的根本不是现在解决的重定位,是运行时
        // 由_dl_runtime_reslove()来解决的.
        //
        // 这个判断非常重要,还记得文章的最开始说过的*stub*函数的特殊吗?stub是定义
        // 在shared library里的函数在main executable里面有引用,但是在executable
        // 里面的重定位表里面stub的那一项的symbol是有值的,因为那里我们还取了stub
        // 的函数地址,这是一个非常隐晦的dl实现上的trick,而且一切的解决源头就是这
        // 里.事实上dl确实是想要main executable里的这个symbol,因为ISO C标准里要
        // 求的的是指针比较的相等性,而并不是要求指针就是实际的函数的地址.在这里
        // 如果能够保证所有的(executable和library)里的指针的值都是0x8048520那么
        // 就够了.而且dl里确实是这样做的.
        // 那么这里可能会引发另外一个问题: 实际发生函数调用的时候怎么办?
        // 答案就是: 如果有实际函数调用的发生,在一般的lazy binding的情况下,将会
        // 调用保存在_GLOBAL_OFFSET_TABLE_[2]的函数_dl_runtime_resolve()来解决重
        // 定位问题.已经不在本文的描述范围之内.本文的描述是在dl bootrap之后但是
        // 在main executable实际获得控制权之前.
        // 回到当前代码,我们知道对于main executable里面的符号而言,sym->st_value
        // 的值是: 8048520,type_class的值0(对应于_tc输出值表里的others).
        //
	if (__builtin_expect ((sym->st_value == 0 /* No value.  */
			       && stt != STT_TLS)
			      || (type_class & (sym->st_shndx == SHN_UNDEF)),
			      0))
	  return NULL;

	/* Ignore all but STT_NOTYPE, STT_OBJECT, STT_FUNC,
	   STT_COMMON, STT_TLS, and STT_GNU_IFUNC since these are no
	   code/data definitions.  */
#define ALLOWED_STT \
	((1 << STT_NOTYPE) | (1 << STT_OBJECT) | (1 << STT_FUNC) \
	 | (1 << STT_COMMON) | (1 << STT_TLS) | (1 << STT_GNU_IFUNC))
	if (__builtin_expect (((1 << stt) & ALLOWED_STT) == 0, 0))
	  return NULL;

	if (sym != ref && strcmp (strtab + sym->st_name, undef_name))
	  /* Not the symbol we are looking for.  */
	  return NULL;
        
        //
        // symbol版本信息的检查
        //
	const ElfW(Half) *verstab = map->l_versyms;
	if (version != NULL)
	  {
	    if (__builtin_expect (verstab == NULL, 0))
	      {
		/* We need a versioned symbol but haven't found any.  If
		   this is the object which is referenced in the verneed
		   entry it is a bug in the library since a symbol must
		   not simply disappear.

		   It would also be a bug in the object since it means that
		   the list of required versions is incomplete and so the
		   tests in dl-version.c haven't found a problem.*/
		assert (version->filename == NULL
			|| ! _dl_name_match_p (version->filename, map));

		/* Otherwise we accept the symbol.  */
	      }
	    else
	      {
		/* We can match the version information or use the
		   default one if it is not hidden.  */
		ElfW(Half) ndx = verstab[symidx] & 0x7fff;
		if ((map->l_versions[ndx].hash != version->hash
		     || strcmp (map->l_versions[ndx].name, version->name))
		    && (version->hidden || map->l_versions[ndx].hash
			|| (verstab[symidx] & 0x8000)))
		  /* It's not the version we want.  */
		  return NULL;
	      }
	  }
	else
	  {
	    if (verstab != NULL)
	      {
		if ((verstab[symidx] & 0x7fff)
		    >= ((flags & DL_LOOKUP_RETURN_NEWEST) ? 2 : 3))
		  {
		    /* Don't accept hidden symbols.  */
		    if ((verstab[symidx] & 0x8000) == 0
			&& num_versions++ == 0)
		      /* No version so far.  */
		      versioned_sym = sym;

		    return NULL;
		  }
	      }
	  }

	/* There cannot be another entry for this symbol so stop here.  */
	return sym;
      }

        //
        // 后面的代码逻辑就是检查每一个symbol定义查找表的link_map的动态符号表,然
        // 后调用chenk_match()来检查.
        //
```

根据上面的代码分析及其代码里添加的注释,应该解释清楚了本文最开始提出的两个问题,其
实核心就是symbol查找表的建立和检查过程.

## 05 总结

本文由两个问题的解决进而分析了glibc/dynamic-linker的bootstrap之后到控制权交由
executable之前的最重要的dl的代码,本文分析的两个问题基本上的dl实现里最trick
最精巧的两个实现细节.而且经过对涉及到的代码和dl架构的描述相信读者已经对dl的实现
有了进一步的认识.

也请关注作者的另外的GCC/binutils的文章.

*live long and prosper*

