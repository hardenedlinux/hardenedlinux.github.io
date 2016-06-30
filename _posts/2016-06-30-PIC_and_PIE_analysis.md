---
layout:
title:
data:
auther:         zet
mail:           
summary:
categories:
---

#Position Independent Code(PIC) and Position Independent Executable(PIE)
@(mitgation)[PIC|PIE|gcc|binutils]

##导引

> In computing, position-independent code (PIC) or position-independent 
executable (PIE) is a body of machine code that, being placed somewhere in the
primary memory, executes properly regardless of its absolute address.
--[Wikipedia](https://en.wikipedia.org/wiki/Position-independent_code)

现代的linux/ELF系统可以随机化shared library的加载地址,这种技术叫做: Address
Space Layout Randomization或者ASLR. shared library肯定是PIC,也就是说可以被记载
在任意地址,而且可以在各个kernel的进程之间共享已经加载入RAM的代码段.加载地址的随
机化使依赖固定地址的攻击(比如buffer overflow)变得难以进行.

尽管可以随机化shared library的加载地址,但是ELF可执行文件在由linker处理的时候被
分配了一个固定的入口地址(在i386下是一个略大于0x08048000的值),怎样就提供了攻击者
一个地址范围.如果把分配入口的权限的给了kernel,那么kernel就可以给main executable
一个随机的入口地址,而linker需要生成Position Independent Executable或PIE,这样很
大程度上可以解决一大类的安全问题.

生成PIE的过程需要通过gcc并给gcc相应的参数来进行,下面主要分析涉及到的参数以及这
些参数的组合引发的结果,过程中再引入gcc/linker的实现相关的描述.在后期会有一篇文
章来做关于relocation type的分析.

首先注意一个问题,PIC是通过GOT/PLT/dynamic linker来实现的,后面会有文章来分析
dynamic linker.本文也不会详细描述GOT以及PLT的实现方式,因为已经有很好这样的分析
[文章]存在.
(http://eli.thegreenplace.net/2011/11/03/position-independent-code-pic-in-shared-libraries)
阅读本文之前建议先了解一些**GOT/PLT**的实现方式.

*出于篇幅以及文章的清晰程度的考虑,本文的描述仅限于i386/ELF32/gcc/gas/GNU ld/
linux kernel.现在通用的ELF 1.2标准出版于1995/05,ASLR在linux kernel里第一个版本
是从2.6.12. 由此可见要想测试本文描述的原理需要的平台软件版本根本不需要太新.后
面可能会有x86_64版本的补充.*

##生成PIC/PIE的gcc/ld参数分析

分析参数之前需要注意另外一个问题,解决应用程序可以在可变地址执行的两种模式:

- load-time relocation
- PIC

其实PIE在两种模式下都可以.后面会解释在两种模式下的二进制级别的差别.

与PIC/PIE相关的参数主要有以下6个.

###gcc code generation options

  -fpic, -fPIC, -fpie, -fPIE

总体来说如果指定了这些参数那么生成PIC模式的object,如果没有指定那么就生成
load-time relocation模式的object.

上面这4个参数分为两类,-fpic/-fPIC和-fpie/-fPIE,对于i386来说这每一类中的两个参数
完全等价,而且这两类参数都是指导编译器生成PIC模式的汇编代码(也就是会有@PLT/
@GOTOFF这样的汇编伪指令).

当指定了-fpie/-fPIE时gcc也会置位-fpic/-fPIC.
```
src/gcc/gcc.c/finish_options()
```
```
  if (!opts->x_flag_opts_finished)
    {
      if (opts->x_flag_pie)
	opts->x_flag_pic = opts->x_flag_pie;
      if (opts->x_flag_pic && !opts->x_flag_pie)
	opts->x_flag_shlib = 1;
      opts->x_flag_opts_finished = true;
    }
```
其中x_flag_pie对应于-fpie/-fPIE, x_flag_pic对应-fpie/-fPIE.

```
src/gcc/gcc.c/cancel_option()
```
```
/* Return true if NEXT_OPT_IDX cancels OPT_IDX.  Return false if the
   next one is the same as ORIG_NEXT_OPT_IDX.  */

static bool
cancel_option (int opt_idx, int next_opt_idx, int orig_next_opt_idx)
{
  /* An option can be canceled by the same option or an option with
     Negative.  */
  if (cl_options [next_opt_idx].neg_index == opt_idx)
    return true;

  if (cl_options [next_opt_idx].neg_index != orig_next_opt_idx)
    return cancel_option (opt_idx, cl_options [next_opt_idx].neg_index,
			  orig_next_opt_idx);

  return false;
}
```
其实在gcc内部实现上这四个参数的有一个索引(neg_index用于重叠意思参数重复指定时的
删除到只剩下一个)分别指向另外一个参数,然后这四个参数的指向正好形成一个环路.也就
是说这四个参数没有任何区别.

但是在ld社区见过另外一种说法:

```
-fpic与-fpie的差别很细微,当使用-fpie时编译器知道当前的编译会生成一个PIC模式的
main executable(也就是有main入口的可执行文件),这样对于内部定义的global符号,就不
要考虑全局符号介入(global symbol interpose)的问题,对于这样的globals直接产生
PC-relative方式的代码而不需要通过GOT/PLT.
```
但是据我对gcc源代码的阅读以及测试,并没有发现四个参数有什么不同. 一个有趣的思考
就是当gcc编译main executable时是知道的, 那么也就是说指定-fpic时也知道这是一个
main executable,那么-fpic与-fpie就肯定是没有区别的.(TODO)这里下一个版本的文章会
给出一个确切的答案.

对于上面的描述需要解释一下全局符号介入.
全局符号介入这个问题主要是因为ELF规定当main executable调用shared library时,同名
全局符号的定义,main executable覆盖shared library.而在static link的情况下,同名符
号的重复定义是非法的.还有就是链接器启动时候需要集合各个elf文件的符号表,生成一个
总的符号表的集合.在确定是否将一个符号加入总符号表时,会查找是否已经有同名符号存
在,如果有则忽略符号的加入.各个elf文件的处理顺序是根据ELF里面的_DYNAMIC[]里d_tag
是DT_NEEDED的元素组成当前elf的依赖链,这样所有涉及到的elf及其依赖链会形成一个图.
连接器按照依赖的满足性(当依赖elf全部已经遍历时就可以遍历本elf)来遍历图.

###ld options

-shared, -pie

因为这两个选项是对linker起作用,所以对最后生成的elf(main executable/library)文件
的影响,取决于是否指定了-fpic/-fPIC/-fpie/-fPIE(对应于load-time relocation和PIC
模式的object)这些参数.

*处理load-time relocation的object

这两个参数有很多相似的地方,-shared和-pie都指导ld处理load-time relocation的代码.
这种代码对引用的符号的relocation type只有三种: R_386_32/R_386_PC32/
R_386_RELATIVE,这三种类型的重定位目标都可以位于.text(代码段).这样如果linker解决
了重定位的问题,那么这个.text里面的数据已经经过了修改,那么这个.text就不可能在多
个kernel中运行的进程之间共享,相对于shared library来说这样对整个系统的RAM很浪费.
但是相应地相对于shared library有一个优点,那就是因为启动快,因为基本上动态连接器
不需要reslove什么符号,而且代码中R_386_32/R_386_RELATIVE方式的重定位目标已经被修
改成了绝对地址,相对于需要运行时全部做间接调用的shared library来说少了很多个指令
周期.

这两个参数最大的不同就是-pie生成的是可以执行的代码.

```bash
zet@fuck-GFW ~/dust/lib/test $gcc -m32 -pie -o pie test.c
zet@fuck-GFW ~/dust/lib/test $gcc -m32 -shared -o shared test.c
zet@fuck-GFW ~/dust/lib/test $readelf -S pie
There are 31 section headers, starting at offset 0x17fc:
Section Headers:
  [Nr] Name              Type            Addr     Off    Size   ES Flg Lk Inf Al
  [ 1] .interp           PROGBITS        00000154 000154 000013 00   A  0   0  1
        [删除不相关的项]

zet@fuck-GFW ~/dust/lib/test $readelf -p1 pie
String dump of section '.interp':
  [     0]  /lib/ld-linux.so.2
```

对比上面的输出可以看出,-pie的代码输出多了一个很重要的section: .interp.kernel就
是根据这个section来找到dynamic linker的,因为pie和shared都是需要重定位的代码,没
有这个section就找不到dynamic linker也就是解决不了重定位当然不可以执行了.

*处理PIC模式的object

PIC模式的object和-shared/-pie配合是manual给出的标准调用.只不过-shared生成了
shared library而且-pie生成了PIC模式的PIE.

根据上面的分析可以想象另外一种假设,如果load-time relocation然后由linker转换为
PIC模式的PIE或者shared library是否可行?

因为GNU ld并没有这样的实现方式,所以这里只是给出分析:

在包含relocation entry的文件里,各个section之间以当前文件和外部library的offset关
系,都是通过relocation entry来联系的,也就是说relocation entry是连接所有有引用关
系唯一的一条线索,如果想调整当前文件的section之间的位置或者多个文件之间的section
位置(linker在进行给几个文件的相似属性的section的合并时会调整section的位置),只要
保证调整之后的所有(linker也会进行各种relocation section的合并)relocation entry
的各个field引用到的symbol和offset处的值和调整之前的一样,就是正确的.

linker的权限很大,注意section的属性R(读)W(写)根本不是针对linker的,这些权限最终会
由于section合并进入segment里,然后kernel执行的main executable的时候来进行检查.
linker会添加一些section(比如.got/.plt)和修改一些指令(比如上面提到的对R_386_32重
定位类型的决议.linker处理时当分配了main executable的入口地址,便可以分配这些引用
到的variable的地址了,linker就可以将variable的地址写入引用这些variable的指令处).
这样linker甚至可以修改load-time relocation方式的三种重定位类型到PIC方式,因为只
要确保修改前后重定位表相关的所有信息一致就可以.

下面给出一个小例子,然后都生成PIE,但是第一种是load-time relocation模式的PIE,第二
种是PIC模式的PIE.然后对比分析两种模式下PIE的区别.

```vim
#include<stdio.h>

int main() {
        printf("address is : %p\n", main);
        
        return 42;
}
```
load-time relocation模式的PIE
```bash
zet@fuck-GFW ~/dust/lib/test $gcc -m32 -pie -o pie test.c

zet@fuck-GFW ~/dust/lib/test $readelf -r pie

Relocation section '.rel.dyn' at offset 0x3c4 contains 12 entries:
 Offset     Info    Type            Sym.Value  Sym. Name
        [删除一些无关项]
00000618  00000c01 R_386_32          0000060b   main
00000624  00000202 R_386_PC32        00000000   printf@GLIBC_2.0

Relocation section '.rel.plt' at offset 0x424 contains 2 entries:
 Offset     Info    Type            Sym.Value  Sym. Name
0000200c  00000207 R_386_JUMP_SLOT   00000000   printf@GLIBC_2.0
00002010  00000507 R_386_JUMP_SLOT   00000000   __libc_start_main@GLIBC_2.0
```
注意上面对printf的重定位模式是R_PC32,这是load-time relocation中相对于当前PC的重
定位方式.linker重定位完成之后是要修改指令段的.

```
zet@fuck-GFW ~/dust/lib/test $gcc -m32 -pie -o pie test.c
zet@fuck-GFW ~/dust/lib/test $./pie
address is : 0xf771765f
zet@fuck-GFW ~/dust/lib/test $./pie
address is : 0xf774865f
zet@fuck-GFW ~/dust/lib/test $./pie
address is : 0xf77de65f
zet@fuck-GFW ~/dust/lib/test $./pie
address is : 0xf773565f
```
```gdb

(gdb) disas/r
Dump of assembler code for function main:
   0x5655560b <+0>:	55	push   %ebp
   0x5655560c <+1>:	89 e5	mov    %esp,%ebp
   0x5655560e <+3>:	83 e4 f0	and    $0xfffffff0,%esp
   0x56555611 <+6>:	83 ec 10	sub    $0x10,%esp
   0x56555614 <+9>:	c7 44 24 04 0b 56 55 56	movl   $0x5655560b,0x4(%esp)
   0x5655561c <+17>:	c7 04 24 c0 56 55 56	movl   $0x565556c0,(%esp)
=> 0x56555623 <+24>:	e8 e8 6d 8f a1	call   0xf7e4c410 <printf>
   0x56555628 <+29>:	b8 2a 00 00 00	mov    $0x2a,%eax
   0x5655562d <+34>:	c9	leave  
   0x5655562e <+35>:	c3	ret    
End of assembler dump.
(gdb) info symbol 0xf7e4c410
printf in section .text of /lib/i386-linux-gnu/libc.so.6
```
上面是运行时的情况,对printf的调用变成了一个相对地址,而且这个相对地址是跟libc加
载入进程空间之后的起始地址有关系也跟当面的main executable加载入kernel时分配的地
址有关系.这就是典型的load-time relocation.会改变代码段的值.

PIC模式的PIE

```bash
zet@fuck-GFW ~/dust/lib/test $gcc -m32 -fpic -pie -o pie test.c
zet@fuck-GFW ~/dust/lib/test $readelf -r pie

Relocation section '.rel.plt' at offset 0x40c contains 2 entries:
 Offset     Info    Type            Sym.Value  Sym. Name
0000200c  00000207 R_386_JUMP_SLOT   00000000   printf@GLIBC_2.0
00002010  00000507 R_386_JUMP_SLOT   00000000   __libc_start_main@GLIBC_2.0
```
```gdb
(gdb) disas
Dump of assembler code for function main:
   0x565555eb <+0>:	push   %ebp
   0x565555ec <+1>:	mov    %esp,%ebp
   0x565555ee <+3>:	push   %ebx
   0x565555ef <+4>:	and    $0xfffffff0,%esp
   0x565555f2 <+7>:	sub    $0x10,%esp
   0x565555f5 <+10>:	call   0x565554c0 <__x86.get_pc_thunk.bx>
   0x565555fa <+15>:	add    $0x1a06,%ebx
   0x56555600 <+21>:	lea    -0x1a15(%ebx),%eax
   0x56555606 <+27>:	mov    %eax,0x4(%esp)
   0x5655560a <+31>:	lea    -0x1940(%ebx),%eax
   0x56555610 <+37>:	mov    %eax,(%esp)
=> 0x56555613 <+40>:	call   0x56555450 <printf@plt>
   0x56555618 <+45>:	mov    $0x2a,%eax
   0x5655561d <+50>:	mov    -0x4(%ebp),%ebx
   0x56555620 <+53>:	leave  
   0x56555621 <+54>:	ret    
End of assembler dump.
(gdb) info symbol 0x56555450
printf@plt in section .plt of /home/zet/dust/lib/test/pie
```
查看gdb最后一行的输出,结合PIC的实现方式,可以看出这就是典型PIC代码运行方式,因为
当前的main executable的.text和.plt的相对地址是不变的,而.text调用的只是相对于
.plt某项,这个值也不变,所以不想要改变.text只需要更改.plt里面的内容,.plt这个
section相对于.text来说很小.所以如果kernel有多个进程执行同一个main executable,那
么这些多个进程之间是可以共享同一份在RAM里的.text的,这就是shared library为什么节
省系统RAM的原因.

##总结

对上面的描述做一个总结:

调用gcc时添加-fpic/-fPIC/-fpie/-fPIE会生成PIC模式的object,如果不添加任何一个则
生成load-time relocation的object.

调用gcc时添加-shared/-pie(这两个选项不会影响编译器会直接传给linker),如果有-pie
那么会生成PIE.如果指定的是-shared那么根据object的情况生成shared librery(编译器
生成的是PIC模式的object)或者非shared的library(编译器没有指定上面的四个选项中的
任意一个).

本文描述了PIC与PIE的区别与联系,以及与他们相关的六个参数.本文编写的过程中忽略了
部分linker相关的细节,GOT/PLT相关的部分可以参考导引部分推荐的那篇文章.至于文中涉
及到的relocation type会有下一篇文章来描述,至于dynamic linker也会有后面的文章来
描述.

