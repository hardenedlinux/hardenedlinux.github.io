---
layout:     post
title:      NX(No-eXecute)的实现分析
date:       2016-06-01
author:     zet
summary:    在计算机安全领域一个很经典的话题就是缓冲区溢出(Buffer Overflow).缓冲区溢出一般时候伴随着攻击者的篡改堆栈里保存的返回地址,然后执行注入到stack中的shellcode,攻击者可以发挥想象力仔细编写shellcode进行下一步的攻击,直到完全控制了计算机.这种攻击之所以能够成功主要原因就是因为stack里的shellcode的可执行.所以主要的防御手段(mitigation)就是禁止stack里数据的执行(noexecstack)
categories: system-security
---

>Shawn: GNU/Linux系统级攻防在历史上曾经停留在用户空间很长的时间，经历了NX/COOKIE/PIE/ASLR/RELRO的进化后后0ldsk00l以及security "researcher"们已经无法通过用户空间触及到“上帝宝藏"(-_root_-)，sgrakkyu和twzi在Phrack Issue 64中的[Attacking the Core](http://phrack.org/archives/issues/64/6.txt)标志着这个领域正式进入了内核层面的对抗，10年过去了，在新的时代性背景下（Android/IoT/TEE），人们意识到安全应该是一个整体（again?WTH），而单纯依赖于内核层面的攻防无法解决很多老问题，传统的mitigation技术再次在某些场景化的方案中受到重视，NX（armv6中是XN）是其中之一，栈的不可执行最早是由PaX team实现的[PAGEEXEC](http://hardenedlinux.org/system-security/2015/05/25/pageexec-old.html)和[SEGEXEC](http://hardenedlinux.org/system-security/2015/05/26/segmexec.html)，后来Intel CPU在硬件上支持NX后Ingo Molnar给出了[硬件NX的第一版实现](http://redhat.com/~mingo/nx-patches/nx-2.6.7-rc2-bk2-AE)给[Fedora的用户尝鲜](http://people.redhat.com/mingo/nx-patches/QuickStart-NX.txt)，后来则进入了Linux mainline。这篇文档详细的分析了GCC/ld/kernel三个层面的NX的工作路线图。Enjoy it!


# NX(No-eXecute)的实现分析

@(mitigation)[NX|gcc|binutils|kernel]
        --[zet](https://github.com/fanfuqiang)

## 00 导引

以下的代码分析仅限**linux kernel/gcc/GNU binutils-as/GNU binutils-ld/ELF**.

在计算机安全领域一个很经典的话题就是缓冲区溢出(Buffer Overflow).缓冲区溢出一般时
候伴随着攻击者的篡改堆栈里保存的返回地址,然后执行注入到stack中的shellcode,攻击者
可以发挥想象力仔细编写shellcode进行下一步的攻击,直到完全控制了计算机.这种攻击之
所以能够成功主要原因就是因为stack里的shellcode的可执行.所以主要的防御手段
(mitigation)就是禁止stack里数据的执行(noexecstack).

Noexecstack的实现主要出现在两个地方: compiler-assembler-linker(这里表示一个生成
binary的过程: 编译->汇编->链接器)里和kernel里.在compiler-assembler-linker里的实
现基本上的纯粹的软件实现,结果是在elf的一个stack的section里置位不可以执行.但是捕
获违反stack不可执行这个问题是在kernel里.

在kernel里的实现,随着处理器在(页模式)paging处理过程中涉及到功能寄存器中引入
No-eXecute的配置位,所以实际上kernel在实现NX的时候是在相关的寄存器里置NX的位,在
CPU操作的时候由硬件来做是否可以执行的检查.

本文的描述描述顺序是先描述NX在gcc/binutils里的实现,然后再描述在kernel里的实现.

*本文的分析对应的**gcc**版本是6.1.0,**binutils**版本是2.26,**linux kernel**的版本是4.6*


## 01 NX在gcc/binutils里面的实现

在gcc/ld里面有NX相关的选择,gcc/ld都是-z execstack/noexecstack,在gcc 6.1 manual里
跟-z相关的内容如下:

>3.14 Options for Linking:
-z keyword
-z is passed directly on to the linker along with the keyword keyword. See the 
section in the documentation of your linker for permitted values and their 
meanings.
--[gcc 6.1 manual](https://gcc.gnu.org/onlinedocs/gcc-6.1.0/gcc/Link-Options.html#Link-Options)

也就是说gcc将参数-z execstack/noexecstack直接传给了ld(linker).

```
gcc -### -z execstack test.c

```

```
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/lib/gcc/x86_64-linux-gnu/4.8/lto-wrapper
Target: x86_64-linux-gnu
 /usr/lib/gcc/x86_64-linux-gnu/4.8/cc1 -quiet -imultiarch x86_64-linux-gnu 
test.c -quiet -dumpbase test.c "-mtune=generic" "-march=x86-64" -auxbase test 
-fstack-protector -Wformat -Wformat-security -o /tmp/ccgX6EXC.s

 // 调用as
 as --64 -o /tmp/ccVl7H5u.o /tmp/ccgX6EXC.s

 // 调用collect2
 /usr/lib/gcc/x86_64-linux-gnu/4.8/collect2 "--sysroot=/" --build-id 
--eh-frame-hdr -m elf_x86_64 "--hash-style=gnu" --as-needed -dynamic-linker 
/lib64/ld-linux-x86-64.so.2 -z relro 
// 传入的参数
-z execstack 
/usr/lib/gcc/x86_64-linux-gnu/4.8/../../../x86_64-linux-gnu/crt1.o 
/usr/lib/gcc/x86_64-linux-gnu/4.8/../../../x86_64-linux-gnu/crti.o 
/usr/lib/gcc/x86_64-linux-gnu/4.8/crtbegin.o -L/usr/lib/gcc/x86_64-linux-gnu/4.8
 -L/usr/lib/gcc/x86_64-linux-gnu/4.8/../../../x86_64-linux-gnu 
-L/usr/lib/gcc/x86_64-linux-gnu/4.8/../../../../lib -L/lib/x86_64-linux-gnu 
-L/lib/../lib -L/usr/lib/x86_64-linux-gnu -L/usr/lib/../lib 
-L/usr/lib/gcc/x86_64-linux-gnu/4.8/../../.. /tmp/ccVl7H5u.o -lgcc --as-needed
-lgcc_s --no-as-needed -lc -lgcc --as-needed -lgcc_s --no-as-needed 
/usr/lib/gcc/x86_64-linux-gnu/4.8/crtend.o 
/usr/lib/gcc/x86_64-linux-gnu/4.8/../../../x86_64-linux-gnu/crtn.o
```

接下来将会详细描述gcc将-z execstack这一参数怎样传给ld(linker)以及这一参数对生成
的ELF文件产生怎样的影响,最后将会分析这样的影响是如何导致stack被执行在kernel里捕
获的.


### NX在gcc里的处理

由上面的**gcc -###**输出可知道**gcc**只是一个外壳管理程序,严格来说是一个
**driver**,根据传入的参数来控制各个**compile/assemble/link**过程.在**gcc**实现里
**compile**过程由**cc1**来完成,**assemble**由**GNU as**完成,**link**由
**GNU ld**来完成(GNU社区里有一个备选的链接器:*gold*).

当**gcc**遇到**-z execstack**这样的选项时的处理代码如下:

*在我的机器上gcc的目录是$HOME/github/gcc*

```
export $SRC=$HOME/github/gcc

```

```
// $SRC/gcc/gcc-main.c
// gcc driver的入口代码
int
main (int argc, char **argv)
{
  driver d (false, /* can_finalize */
	    false); /* debug */

  return d.main (argc, argv);
}

// src/gcc/gcc.c
/* driver::main is implemented as a series of driver:: method calls.  */
int
driver::main (int argc, char **argv)
{
  bool early_exit;
	
  set_progname (argv[0]);  // 对调用gcc时指定的名字做处理,删掉前面的目录名,不需
  要过多研究expand_at_files (&argc, &argv); // 对参数做一些扩展处理
  decode_argv (argc, const_cast <const char **> (argv)); // 下面分析
  // 下面的这些函数都是在做gcc的常规处理,跟本文主题关系不大
  global_initializations ();
  build_multilib_strings ();
  set_up_specs ();
  putenv_COLLECT_GCC (argv[0]);
  maybe_putenv_COLLECT_LTO_WRAPPER ();
  maybe_putenv_OFFLOAD_TARGETS ();
  handle_unrecognized_options ();

  if (!maybe_print_and_exit ())
    return 0;

  early_exit = prepare_infiles ();
  if (early_exit)
    return get_exit_code ();
  
  do_spec_on_infiles (); // 这里会调用cc1和as
  maybe_run_linker (argv[0]); // 这里会调用ld
  final_actions ();
  return get_exit_code ();
}

```


由上面的代码注释可知,我们需要研究3个main入口里的函数,下面依次进行分析.由于代码过
多,下面的分析会删掉跟本文主题无关的代码,函数调用关系由**->**表示.如果不做申明,源
代码位于*src/gcc/gcc.c*这个文件里.

在分析之前需要说清楚另外一个问题,那就是**-z**这个参数的定义问题.

```
cd $HOME/github/gcc
mkdir build
cd build
../configure --prefix="$HOME/bin" --disable-nls --enable-languages=c,c++
make -j8
make install

```


编译完成之后,会在*build/gcc*里有两个跟本文的主题有关的文件: 
**options.c**/**options.h**.这两个文件跟gcc的编译选项处理有很大关系,这两个文件的
生成是几个*$SRC/gcc*里的*awk*脚本共同作用的结果.

```
// Makefile.in
// 注意这里的输入是$(ALL_OPT_FILES)

optionlist: s-options ; @true
s-options: $(ALL_OPT_FILES) Makefile $(srcdir)/opt-gather.awk
	$(AWK) -f $(srcdir)/opt-gather.awk $(ALL_OPT_FILES) > tmp-optionlist
	$(SHELL) $(srcdir)/../move-if-change tmp-optionlist optionlist
	$(STAMP) s-options
options.c: optionlist $(srcdir)/opt-functions.awk $(srcdir)/opt-read.awk \
    $(srcdir)/optc-gen.awk
	$(AWK) -f $(srcdir)/opt-functions.awk -f $(srcdir)/opt-read.awk \
	       -f $(srcdir)/optc-gen.awk \
	       -v header_name="config.h system.h coretypes.h options.h tm.h" < $< > $@	       
options.h: s-options-h ; @true
s-options-h: optionlist $(srcdir)/opt-functions.awk $(srcdir)/opt-read.awk \
    $(srcdir)/opth-gen.awk
	$(AWK) -f $(srcdir)/opt-functions.awk -f $(srcdir)/opt-read.awk \
	       -f $(srcdir)/opth-gen.awk \
	       < $< > tmp-options.h
	$(SHELL) $(srcdir)/../move-if-change tmp-options.h options.h
	$(STAMP) $@

// options.h/options.c输入文件,也就时生成gcc选项处理代码的配置文件.加选项只需要
// 修改这些配置文件,很方便.
# All option source files
ALL_OPT_FILES=$(lang_opt_files) $(extra_opt_files)

// 编译选项配置文件
lang_opt_files=@lang_opt_files@ $(srcdir)/c-family/c.opt $(srcdir)/common.opt

```

由上面的代码可以知道编译选项的生成过程是输入配置文件,然后awk脚本处理配置文件,然
后输出options.h/options.c

```
// $SRC/gcc/common.opt里关于-z的内容如下:
// 注意z底下的三个配置,Driver表示这是一个driver处理的选项(考虑一些debug的配置选
项),Joined/Separate表示-z与跟-z本身相对于的参数(在本文中当然是指execstack/
noexecstack)之间需不需要空白符隔开.

```

```
z   
Driver Joined Separate

```

相应的生成代码是:

```
OPT_z = 1251, 		/* -z */

```


接着进行**gcc**对参数处理的分析


```
// 处理调用gcc时的参数存入decoded_options数组里
decode_argv()->decode_cmdline_options_to_array()->decode_cmdline_option()

static unsigned int
decode_cmdline_option (const char **argv, unsigned int lang_mask,
		       struct cl_decoded_option *decoded)
{
  // awk处理参数配置文件时会将这些参数存进一个数组里cl_options[]
  // 这里argv[0] + 1的值是'z',由此来找到-z在cl_options数组里的索引值
  opt_index = find_opt (argv[0] + 1, lang_mask);
  // const struct cl_option *option
  option = &cl_options[opt_index];
  // -z在配置文件里的定义是Joined Separate,所以会进入这个代码块
  if (joined_arg_flag)
    {
      // 注意下面的+1,arg的值会是"-z"里z之后的下一个字符,是'\0'
      arg = argv[extra_args] + cl_options[opt_index].opt_len + 1 + adjust_len;
      //cl_missing_ok表示-z后面不接参数是否可以,显然是不行
      if (*arg == '\0' && !option->cl_missing_ok)
	{
	  // -z的另外一个配置:Separate
	  if (separate_arg_flag)
	    {
	      // 这里arg的值应该是"execstack"
	      arg = argv[extra_args + 1];
	      result = extra_args + 2;
	      if (arg == NULL)
		result = extra_args + 1;
	      else
		have_separate_arg = true;
	    }
	  else
	      /* Missing argument.  */
	      arg = NULL;
	}
    }
    // 删掉无关代码
    // 这个结构是要传会给调用者的
    decoded->opt_index = opt_index; // OPT_z在cl_options[]里的索引
    decoded->arg = arg;  // "execstack"
    decoded->value = value;
    decoded->errors = errors;
    decoded->warn_message = warn_message;
    // 后面的代码会处理别的参数,在本文的例子里就是待编译的文件:test.c

```

*gcc*处理完参数之后会进行对输入文件的各种处理.当然在上面的分析中可知输入文件也是
被处理参数的代码处理的,只不过*decoded->opt_index*表示这就是输入文件.

*gcc driver*对*cc1/as/ld*的调用都是通过一个**spec**文件来进行的,也是一种配置文件.
**spec**配置的语法定义于
[gcc manual: 3.19 Specifying Subprocesses and the Switches to Pass to Them]
(https://gcc.gnu.org/onlinedocs/gcc/Spec-Files.html),
与本文涉及到的比较重要的语法如下:
>%a
处理as的相关调用. 默认的spec文件叫: asm
>
>%A
处理as相关的调用,默认的spec文件是: asm_final
>
>%(name) 
类似于宏替换,将之前定义的name在这里展开
>
>%{S}
当选项S给出时,用-S替换S,注意这里的S是元字符
>
>%{S:X}
对X进行替换操作,当选项-S给出时
>
>%{!S:X}
对X进行替换操作,当选项-S没有给出时

*gcc driver*对可以调用的子工具的存储在一个统一的数组里.其中*compiler->spec*就是
相应工具默认的调用参数.

```

/* The default list of file name suffixes and their compilation specs.  */
static const struct compiler default_compilers[] =
{
  // 只留下部分代表性的数据
  {".cc", "#C++", 0, 0, 0}, {".cxx", "#C++", 0, 0, 0},
  {".cpp", "#C++", 0, 0, 0}, {".cp", "#C++", 0, 0, 0},
  {".c++", "#C++", 0, 0, 0}, {".C", "#C++", 0, 0, 0},
  {".CPP", "#C++", 0, 0, 0}, {".ii", "#C++", 0, 0, 0},
  {".ads", "#Ada", 0, 0, 0}, {".adb", "#Ada", 0, 0, 0},
  {".go", "#Go", 0, 1, 0},
  /* Next come the entries for C.  */
  {".c", "@c", 0, 0, 1},
  // 这里是cc1的spec文件
  {"@c",
     "%{E|M|MM:%(trad_capable_cpp) %(cpp_options) %(cpp_debug_options)}\
      %{!E:%{!M:%{!MM:\
          %{traditional:\
      %eGNU C no longer supports -traditional without -E}\
      %{save-temps*|traditional-cpp|no-integrated-cpp:%(trad_capable_cpp) \
	  %(cpp_options) -o %{save-temps*:%b.i} %{!save-temps*:%g.i} \n\
	    cc1 -fpreprocessed %{save-temps*:%b.i} %{!save-temps*:%g.i} \
	  %(cc1_options)}\
      %{!save-temps*:%{!traditional-cpp:%{!no-integrated-cpp:\
	  cc1 %(cpp_unique_options) %(cc1_options)}}}\
	  // 注意这里!fsyntax-only:%(invoke_as)表示,如果-fsyntax-only没有指定,那
	  // 么就调用as(invoke_as),invoka_as    将会在这里在这里展开,invoke_as定
	  // 义见下面.
      %{!fsyntax-only:%(invoke_as)}}}}", 0, 0, 1},
      {"-",
      "%{!E:%e-E or -x required when input is from standard input}\
      %(trad_capable_cpp) %(cpp_options) %(cpp_debug_options)", 0, 0, 0},
  // 当gcc -S时
  {".s", "@assembler", 0, 0, 0},
  {"@assembler",
   "%{!M:%{!MM:%{!E:%{!S:as %(asm_debug) %(asm_options) %i %A }}}}", 0, 0, 0},

#include "specs.h"
  /* Mark end of table.  */
  {0, 0, 0, 0, 0}
};

// 当gcc编译完输入文件之后,调用as时的driver spec定义.
static const char *invoke_as =
#ifdef AS_NEEDS_DASH_FOR_PIPED_INPUT
"%{!fwpa*:\
   %{fcompare-debug=*|fdump-final-insns=*:%:compare-debug-dump-opt()}\
   // 在下面可以看到很明显的as调用.
   %{!S:-o %|.s |\n as %(asm_options) %|.s %A }\
  }";
#else
"%{!fwpa*:\
   %{fcompare-debug=*|fdump-final-insns=*:%:compare-debug-dump-opt()}\
   %{!S:-o %|.s |\n as %(asm_options) %m.s %A }\
  }";
#endif

```

接着将会是*gcc driver*调用相应的工具程序处理输入源文件.由上面的*spec*可以看到默
认的*cc1/as*调用以输出汇编代码.

```
/* 处理输入源文件,根据相应的工具程序的spec输出汇编代码*/
void
driver::do_spec_on_infiles () const
{
  size_t i;

  for (i = 0; (int) i < n_infiles; i++)
    {
      // 根绝输入文件的后缀查找编译器,就是找到上面的default_compilers[]里面的一个
      input_file_compiler
          = lookup_compiler (infiles[i].name, input_filename_length,
			   infiles[i].language);

      if (input_file_compiler) {
	  if (input_file_compiler->spec[0] == '#')
	    ;
	  else {
	      int value;
	      // 根据spec文件来调用相应的工具程序,这里会输出汇编
	      value = do_spec (input_file_compiler->spec);
	      infiles[i].compiled = true;
	  }
	}    
}

```

最后将是*gcc*调用*linker*来处理汇编代码,在这里本文将会研究*-z execstack*的传递过程.

```
driver::maybe_run_linker() -> do_spec(link_command_spec);

#define link_command_spec LINK_COMMAND_SPEC
#define LINK_COMMAND_SPEC "\
%{!fsyntax-only:%{!c:%{!M:%{!MM:%{!E:%{!S:\
    // linker在这里定义为collect2,其实只是一个GNU ld的包装
    %(linker) " \
    LINK_PLUGIN_SPEC \
    "%{flto|flto=*:%<fcompare-debug*} \
    %{flto} %{fno-lto} %{flto=*} %l " LINK_PIE_SPEC \
    "%{fuse-ld=*:-fuse-ld=%*} " LINK_COMPRESS_DEBUG_SPEC \
    "%X %{o*} %{e*} %{N} %{n} %{r}\
    // 这里第4个就是本文关注的-z选项
    %{s} %{t} %{u*} %{z} %{Z} %{!nostdlib:%{!nostartfiles:%S}} \
    %{static:} %{L*} %(mfwrap) %(link_libgcc) " \
    VTABLE_VERIFICATION_SPEC " " SANITIZER_EARLY_SPEC " %o " CHKP_SPEC " \
    %{fopenacc|fopenmp|%:gt(%{ftree-parallelize-loops=*:%*} 1):\
	%:include(libgomp.spec)%(link_gomp)}\
    %{fcilkplus:%:include(libcilkrts.spec)%(link_cilkrts)}\
    %{fgnu-tm:%:include(libitm.spec)%(link_itm)}\
    %(mflib) " STACK_SPLIT_SPEC "\
    %{fprofile-arcs|fprofile-generate*|coverage:-lgcov} " SANITIZER_SPEC " \
    %{!nostdlib:%{!nodefaultlibs:%(link_ssp) %(link_gcc_c_sequence)}}\
    %{!nostdlib:%{!nostartfiles:%E}} %{T*}  \n%(post_link) }}}}}}"

```

接着本文将会分析当遇到*LINK_COMMAND_SPEC*里的*%{z}*时进行的操作.

```
do_spec()->do_spec_2()->do_spec_1()

```

```
static int
do_spec_1 (const char *spec, int inswitch, const char *soft_matched_part) {
  // 在这里本文关注的spec将会是%{z}
  const char *p = spec;
  while ((c = *p++))
    switch (inswitch ? 'a' : c) {
      case '%':
	    switch (c = *p++)
		  case '{':
		    p = handle_braces (p);
		    break;

```

```
do_spec_1()->handle_braces()

```

```
static const char *
handle_braces (const char *p) {
  // 标记"z}"的起始和结束
  atom = p;
  while (ISIDNUM (*p) || *p == '-' || *p == '+' || *p == '='
		 || *p == ',' || *p == '.' || *p == '@')
    p++;
    end_atom = p;
    // p当前的值应该是'}'
    switch (*p) {
	case '&': case '}':
        /** 
        struct switchstr {
          const char *part1;
          const char **args;
          unsigned int live_cond;
          bool known;
          bool validated;
          bool ordering;
        };
        struct switchstr switches[];
        在这里的时候switches[]里面存储的是调用gcc时候的参数,其中有一项是{"z", &"execstack",}
        根据'z'在switches[]里面置part1是"z"的这一项的ordering为1 */
	  
	mark_matching_switches (atom, end_atom, a_is_starred);
	if (*p == '}')
	  process_marked_switches ();
	break;
    }  
}

```

```
do_spec_1()->handle_braces()->process_marked_switches()

```

```
static inline void
process_marked_switches (void) {
  int i;

  for (i = 0; i < n_switches; i++)
    // 根据上面的ordering的标记调用give_switch (i, 0)
    if (switches[i].ordering == 1) {
	  switches[i].ordering = 0;
	  give_switch (i, 0);
    }
}

```

```
do_spec_1()->handle_braces()->process_marked_switches()->give_switch()

```

```
static void
give_switch (int switchnum, int omit_first_word) {
  if (!omit_first_word) {
      do_spec_1 ("-", 0, NULL);
      // 这里的part1是"z",这个函数最终的处理会将"z"压入一个类似于C++ STL vertor
      // 的容器argbuf里
      do_spec_1 (switches[switchnum].part1, 1, NULL);
  }

  if (switches[switchnum].args != 0) {
      const char **p;
      for (p = switches[switchnum].args; *p; p++) {
	    const char *arg = *p;
	    // 这里arg的值将会是"execstack",这个函数会将execstack压入argbuf里,到
	    // 这里argbuf的值已经是"z execstack"了,由上面的link_command_spec定义
	    // 中的%{z}和spec文件的相关语义,gcc最终对linker的调用将会是:
	    // collect2 -z execstack ... 这个样子的
	    do_spec_1 (arg, 1, NULL);
	  }
    }
    // ...
}

```

### NX在ld里面的处理

下面将会分析*linker*遇到*-z execstack*时进行怎样的处理.对生成的ELF文件产生怎样的
影响.

```
在binutils/include/elf/common.h里与execstack/noexecstack相关的定义如下:

#define PF_X		(1 << 0)	/* Segment is executable */
#define PF_W		(1 << 1)	/* Segment is writable */
#define PF_R		(1 << 2)	/* Segment is readable */

```

由于篇幅所限,下面仅仅分析当*ld*被调用时与*-z execstack*相关的代码.

```
	/**
	bfd_link_info里分别有两个位域: 
	unsigned int execstack: 1;
    	unsigned int noexecstack: 1; */
	struct bfd_link_info link_info;
// GUN linker的入口函数
int
main (int argc, char **argv) {
  // 给bfd_link_info赋一些默认值
  // 处理参数,不过-z execstack的处理代码是在binutils里架构相关的结构里定义的
  parse_args (argc, argv);
  // 做一些分配地址之前的准备工作
  lang_process ();
  // 生成一个elf文件
  ldwrite ();
  // ...
}

```

```
main()->parse_args()->ldemul_handle_option()->ld_emulation->handle_option()

```

```
/** ld_emulation是一个跟架构相关的结构,binutils里面根据后端的不同分开定义是为了
移植和feature的方便,在项目里是很常见的工程设计.*/

// EMULATION_NAME是一个类似于elf_x86_64这样的名字
static bfd_boolean
gld${EMULATION_NAME}_handle_option (int optc) {
  switch (optc) {
    case 'z':
    if (strcmp (optarg, "execstack") == 0) {
      // 在link_info里保存置相关的位
	  link_info.execstack = TRUE;
	  link_info.noexecstack = FALSE;
	} else if (strcmp (optarg, "noexecstack") == 0) {
	  link_info.noexecstack = TRUE;
	  link_info.execstack = FALSE;
	}
	// ...
  }
}

```

```
main()->lang_process ()->ldemul_before_allocation()->ld_emulation->before_allocation()

```

```
// ld_emulation的初始化定义为:
struct ld_emulation_xfer_struct ld_${EMULATION_NAME}_emulation =
{
  gld${EMULATION_NAME}_before_parse,
  syslib_default,
  hll_default,
  after_parse_default,
  after_open_default,
  after_allocation_default,
  set_output_arch_default,
  ldemul_default_target,
  // ld_emulation->before_allocation()的调用会是这个函数
  gld${EMULATION_NAME}_before_allocation,
  //...
};

```

```
gld${EMULATION_NAME}_before_allocation()->bfd_elf_size_dynamic_sections()

```

```
bfd_boolean
bfd_elf_size_dynamic_sections (bfd *output_bfd,
			       const char *soname,
			       const char *rpath,
			       const char *filter_shlib,
			       const char * const *auxiliary_filters,
			       struct bfd_link_info *info,
			       asection **sinterpptr,
			       struct bfd_elf_version_tree *verdefs) {
  
  // 根据参数的分析结果,也就是bfd_link_info结构中的execstack/noexecstack来置位
  // stack_flags
  if (info->execstack)
    elf_tdata (output_bfd)->stack_flags = PF_R | PF_W | PF_X;
  else if (info->noexecstack)
    elf_tdata (output_bfd)->stack_flags = PF_R | PF_W;
  // ...
}

```

*ld_write()*将会是*linker*的最后一步,正确执行完将会得到一个目标文件(比如说*ELF*
格式的可执行文件)

```
main->ld_write()->bfd_final_link()->bfd_elf_final_link()
->_bfd_elf_compute_section_file_positions()
->assign_file_positions_except_relocs()->assign_file_positions_for_segments()
->map_sections_to_segments()

```

```
// 这个结构描述section到segment的对应关系
struct elf_segment_map
{
  /* Next program segment.  */
  struct elf_segment_map *next;
  unsigned long p_type;
  unsigned long p_flags;
  bfd_vma p_paddr;
  unsigned int p_flags_valid : 1;
  unsigned int p_paddr_valid : 1;
  unsigned int includes_filehdr : 1;
  unsigned int includes_phdrs : 1;
  /* 这个segment包含的section数目*/
  unsigned int count;
  /* Sections*/
  asection *sections[1];
};

map_sections_to_segments() {
  // 删掉跟本文无关的代码
  struct elf_segment_map *m;
  if (elf_tdata (abfd)->stack_flags) {
    amt = sizeof (struct elf_segment_map);
    m = bfd_zalloc (abfd, amt);
    if (m == NULL)
	  goto error_return;
    m->next = NULL;
    m->p_type = PT_GNU_STACK;
    /** 根据stack_flags来置位segment的p_flags,最终这个值就是ELF文件的
    Program Header里的p_flag的值,在ELF1.2标准里定义的可选值是: 
    PF_X 0x1
    PF_W 0x2
    PF_R 0x4
    */
    m->p_flags = elf_tdata (abfd)->stack_flags;
    m->p_flags_valid = 1;

    *pm = m;
    pm = &m->next;
  }
}

```

### NX在kernel里的捕获
上面已经介绍了*gcc/ld*里对*-z execstack*的处理,总共的影响就是在ELF文件里对应的
*program header*里置相关的位.下面将会描述在ELF文件里这样的置位前提下,如果违反了
访问规则,*kernel*如何捕获非法访问.

一般来说*kernel*执行一个binary的时候会进行下面的代码调用链:

```
do_execve()->search_binary_handler()->linux_binfmt.load_binary()
// elf文件的情况下load_binary()实际是就是调用load_elf_binary()
static struct linux_binfmt elf_format = {
	.module		= THIS_MODULE,
	.load_binary	= load_elf_binary,
	.load_shlib	= load_elf_library,
	.core_dump	= elf_core_dump,
	.min_coredump	= ELF_EXEC_PAGESIZE,
};

```

```
static int load_elf_binary(struct linux_binprm *bprm, struct pt_regs *regs) {
  int executable_stack = EXSTACK_DEFAULT;
  // bprm->buf里存储的就是elf文件的二进制流,读入elf header
  loc->elf_ex = *((struct elfhdr *)bprm->buf);
  // e_phnum表示program header的数目,这里分配的存储是为了容纳elf里的
  // program header在进程地址空间里
  size = loc->elf_ex.e_phnum * sizeof(struct elf_phdr);
  retval = -ENOMEM;
  elf_phdata = kmalloc(size, GFP_KERNEL);
  if (!elf_phdata)
    goto out;
  // 读入elf文件program header进入地址空间
  retval = kernel_read(bprm->file, loc->elf_ex.e_phoff,
			     (char *)elf_phdata, size);
  // ...
  for (i = 0; i < loc->elf_ex.e_phnum; i++, elf_ppnt++)
    if (elf_ppnt->p_type == PT_GNU_STACK) {
      // 在ld里置位的p_flags
        if (elf_ppnt->p_flags & PF_X)
	  executable_stack = EXSTACK_ENABLE_X;
	else
	  executable_stack = EXSTACK_DISABLE_X;
        break;
    }
	//...
  retval = setup_arg_pages(bprm, randomize_stack_top(STACK_TOP),
				 executable_stack);

```

```
load_elf_binary()->setup_arg_pages()

```

```
// 处理加载的elf对应的进程的初始stack对应的vm_area_struct
int setup_arg_pages(struct linux_binprm *bprm,
		    unsigned long stack_top,
		    int executable_stack) {	
	struct mm_struct *mm = current->mm;
	struct vm_area_struct *vma = bprm->vma;
	unsigned long vm_flags;
    	// ...
    	if (unlikely(executable_stack == EXSTACK_ENABLE_X))
		vm_flags |= VM_EXEC;
	else if (executable_stack == EXSTACK_DISABLE_X)
		vm_flags &= ~VM_EXEC;
	vm_flags |= mm->def_flags;
	vm_flags |= VM_STACK_INCOMPLETE_SETUP;
	// vm_flags里的位会加入到vma里
    	ret = mprotect_fixup(vma, &prev, vma->vm_start, vma->vm_end,
			vm_flags);
	// ...

```

注意上面建立的*vm_area_struct*只是存在于进程的虚拟地址空间里.并没有映射实际的RAM,
当这个ELF对*stack*进行访问时就会进入*page fault*,处理代码就是*do_page_fault()*

```
void do_page_fault(struct pt_regs *regs, unsigned long error_code) {
	// 忽略掉跟本文讨论无关的一系列kernel的检查过程
    	// 对于我们刚刚映射的stack VMA来说会执行到这里
good_area:
	// access_error()会对vma进行常规检查
	if (unlikely(access_error(error_code, vma))) {
		bad_area_access_error(regs, error_code, address);
		return;
	}
	// ...
}

```

```
do_page_fault()->access_error()

```

```
static inline int
access_error(unsigned long error_code, struct vm_area_struct *vma)
{
	if (error_code & PF_WRITE) {
	  /* write, present and write, not present: */
	    if (unlikely(!(vma->vm_flags & VM_WRITE)))
	      return 1;
	    return 0;
	}
	/* read, present: */
	if (unlikely(error_code & PF_PROT))
		return 1;
	// 假如调用gcc编译elf文件时给出的参数是-z noexecstack,那么stack 
	// vma->flags的VM_EXEC位肯定是清除了的.如果是这种情况,那么access_error()
	// 返回1,do_page_fault()也会返回上一级,对应的必将是kernel的报错.
	if (unlikely(!(vma->vm_flags & (VM_READ | VM_EXEC | VM_WRITE))))
	  return 1;

	return 0;
}

```

### NX软件实现小结
总结一下前面的内容,就是当调用*gcc -z execstack test.c*时,gcc将参数打包处理传给ld,
由于参数的影响,ld会在生成的ELF文件stack对应的program header里置位p_flags的PF_X值,
当ELF文件执行时,由于RAM需要分配就会触发page fault,然后处理do_page_fault()函数里
调用access_error()以捕获到stack的执行权限错误.


## 02 NX在kernel/CPU里的实现:

NX在CPU里面的实现跟硬件有很大的关系.所以下面的描述先从硬件相关的寄存器开始描述,
然后进行kernel层面的描述.

### NX相关的寄存器

>Intel® 64 and IA-32 Architectures Developer's Manual - System Programming Guide
>2.2.1  Extended Feature Enable Register(EFER)
IA32_EFER MSR提供了一些IA32e模式相关的使能配置位,还有另外一些位是跟page-access权
限相关的.
typedef struct IA32_EFER {
	long SYSCALL_Enable : 1;	// Enables SYSCALL/SYSRET instructions in 64-bit mode
	long Reserved : 7;		// Reserved
	long IA-32e_Mode_Enable : 1;	// Enables IA-32e mode operation
	long Reserved : 1;	        // Reserved
	long IA-32e_Mode_Active : 1;	// Indicates IA-32e mode is active when set.
	long Execute_Disable_Bit_Enable : 1 // Enables page access restriction by preventing instruction 
					    // fetches from PAE pages with the XD bit set.
	                                    // 我们感兴趣的这一位,也就是第12位,这一位也叫作NXE
	long : 0;
} IA32_EFER;

>IA32_EFER.NXE仅仅对PAE和IA-32e模式起作用(因为只有PAE/IA-32e模式下的paging单元(页
表项/页目录表项)是64位的).如果IA32_EFER.NXE = 1,从某一线性地址处的指令预取将会被
禁止,即使这一线性地址处的数据访问是允许的.

>如果CPUID.80000001H:EDX.NX [bit 20] = 1, IA32_EFER.NXE才能够被设置为1,不支持
CPUID.80000001H的处理器IA32_EFER.NXE不能被设置为1.

>4.4.2  Linear-Address Translation with PAE Paging 
在PAE paging中,如果IA32_EFER.NXE = 0且PDE/ PTE的P是1, 则XD(63位)是保留的.

>(PAE/PTE).63 (XD)跟IA32_EFER.NXE的功能是类似的,只不过存在于页表寄存器/页目录表
寄存器的最高位.

由上面的内容可以知道,要想在MMU层面使用NX,首先需要检测
CPUID.80000001H:EDX.NX [bit 20]是否为1,如果是1进行IA32_EFER.NXE的置位使能,然后按
照需在PAE/PTE里使能第63位(XD).

### NX在kernel里的实现

// 在arch/x86/mm/Setup_nx.c有如下的代码:

```
static int disable_nx;
/*
 * noexec = on|off
 *
 * Control non-executable mappings for processes.
 *
 * on      Enable
 * off     Disable
 */
static int __init noexec_setup(char *str)
{
	if (!str)
		return -EINVAL;
	if (!strncmp(str, "on", 2)) {
		disable_nx = 0;
	} else if (!strncmp(str, "off", 3)) {
		disable_nx = 1;
	}
	x86_configure_nx();
	return 0;
}
// 注册到kernel的启动组件里,可以在boot参数里配置是否启用noexec
early_param("noexec", noexec_setup);

void x86_configure_nx(void)
{
	if (boot_cpu_has(X86_FEATURE_NX) && !disable_nx)
		__supported_pte_mask |= _PAGE_NX;
	else
		__supported_pte_mask &= ~_PAGE_NX;
}

// __supported_pte_mask的初始定义
pteval_t __supported_pte_mask __read_mostly = ~0;

// _PAGE_NX的定义与PAE/PTE的第63位(XD)对应
#if defined(CONFIG_X86_64) || defined(CONFIG_X86_PAE)
#define _PAGE_NX	1 << 63

```

MMU实现NX的在三层的paging结构中是类似的.下面的描述以PTE为代表来进行.

```
#define mk_pte(page, pgprot)   pfn_pte(page_to_pfn(page), (pgprot))
// 创建一个能进入PTE的项,对于本文的NX描述来说,这个值肯定是64位的.
static inline pte_t pfn_pte(unsigned long page_nr, pgprot_t pgprot)
{
	return __pte(((phys_addr_t)page_nr << PAGE_SHIFT) |
		     massage_pgprot(pgprot));
}

static inline pgprotval_t massage_pgprot(pgprot_t pgprot)
{
	pgprotval_t protval = pgprot_val(pgprot);
	if (protval & _PAGE_PRESENT)
	    // 这里对每一个实际作用的访问(_PAGE_PRESENT置位),pte_t(最终是要写入
	    // PTE项的)的值的产生都要经过__supported_pte_mask,这个变量里按需存储
	    // 了是否使用_PAGE_NX的信息.最终的pte_t的值会写入PTE项.
	    protval &= __supported_pte_mask;

	return protval;
}

```

写入PAT/PTE之后,就是CPU自己的操作了,paging之前CPU会检查相应的置位,以决定是否预取
指令.


## 03 总结

上面我们详细描述了NX在整个系统层的实现细节.在系统安全领域由于stack作为一个可以写
的存储区所以很容易作为攻击者的目标,stack overflow作为经典而且古老的攻击方式给
stack植入shellcode然后因为stack的可执行性,让攻击的门槛非常低.后来引入了canary的
机制,但是canary容易被bypass,真正解决这个问题就是NX的引入,所以NX其实是stack 
overflow攻击的最重要的解决方案.

*live long and prosper*

