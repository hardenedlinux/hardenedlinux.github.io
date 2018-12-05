---
layout: post
author: zerons
title: SRCINV-源码审计系统的简要介绍
summary: SRCINV的说明文档, 使用方法, 设计思路, 测试演示

categories:
---

# Contents
+ 1 - 简介
 + 1.1 - 框架
 + 1.2 - 依赖
 + 1.3 - 针对目标
+ 2 - 框架的简单使用
+ 3 - 信息收集
+ 4 - 信息解析
 + 4.1 - 信息调整
 + 4.2 - 基础信息获取
 + 4.3 - 详细信息获取
 + 4.4 - 交叉引用信息整理
 + 4.5 - 间接调用1
 + 4.6 - 间接调用2
+ 5 - 信息使用
+ 6 - 后续
+ 7 - 引用



### 1 - 简介
该文档将介绍SRCINV框架(beta)的使用, 以及设计思路, 和主要针对的目标.



##### 1.1 - 框架
SRCINV是source code investigation的缩写, 是根据经验, 对源码审计工作的代码体现.

源码(或二进制代码)是提供给研究人员查看的, 也是提供给编译器(或解释器或执行单元)的,
该框架从编译器编译源码文件的中间信息, 提取解析该项目的所有源码信息提供给研究人员使用.
由于通常的gcc插件, 看到的是当前编译单元的信息, 无法看到整个项目. 所以我们将整个项目的
每个编译单元的信息提取出来再进行整合, 便于从全局角度去观察整个项目.

该框架旨在对开源项目进行高度自动化的代码审计, 以帮助开发测试/安全研究人员, 发现项目
中可能存在的代码问题, 并提供一定程度的验证样本生成和代码补丁生成等功能.

对于每个项目, 使用一个struct src结构记录所有生成的索引信息, 包含
sibuf(用于表示编译文件的索引信息)链表,
resfile(用于表示提取的结果文件)链表,
sinodes(用于表示非局部变量/函数/类型, 分两大种类:名称索引, 位置索引. 对于文件
变量(static)/文件函数/有位置信息的类型, 使用位置索引; 对于全局变量/全局函数/没有位
置信息但是有名称的类型, 使用名称索引; 对于没有位置也没有名称的类型, 记录在sibuf中)

目录结构如下:
```
	si_core.c	主程序
	collect/	编译器插件, 信息收集
	plugins/	信息解析及使用等各种功能plugin
	output/		索引数据等文件
```



##### 1.2 - 依赖
该框架只能运行于64位GNU/Linux系统, 且需要存在`personality`系统调用以关闭进行aslr.

终端颜色显示, 不同的发行版可能会存在一些兼容问题.

其他依赖的库:
```
	clib (https://github.com/snorez/clib)
	ncurses
	readline
	libcapstone
```

需要的头文件:
```
	gcc plugin 库文件
```



##### 1.3 - 针对目标
该框架预期为尽可能多种类的开源项目进行测试. 当前该框架只针对gcc编译器编译的C项目.



### 2 - 框架的简单使用
框架集成了一些基础命令以供使用(可查看 --[ 5 - 信息使用 节的演示视频):
```
	si_core:	主程序
	help:		使用帮助, 查看当前可使用的命令
	exit/quit:	退出程序
	load_plugin:	加载plugin
	unload_plugin:	卸载plugin
	reload_plugin:	重新加载plugin
	list_plugin:	显示当前所有可供使用的plugin
	do_make:	按照内核编译模式编译指定项目
	do_sh:		执行终端命令
	set_plugin_dir: 设置plugin目录, 默认为plugins/
	showlog:	显示日志信息, 日志文件位于plugins/log.txt
	load_srcfile:	加载索引信息文件
	set_srcfile:	设置索引信息路径
	getinfo:	获取收集的项目信息的索引信息
	staticchk:	静态检测
	itersn:		输出所有的索引信息

	SRCINV> help
	========= USAGE INFO =========
	help:
				Show this help message
	exit:
				Exit this main process
	quit:
				Exit this main process
	load_plugin:
		(plugin_path) [plugin_args...]
	unload_plugin:
		(id|path)	Unload specific plugin
	reload_plugin:
		(id|path) [plugin_args]	Reload target plugin
	list_plugin:
				Show current loaded plugins
	do_make:
		(c|cpp|...) (sodir) (projectdir) (outfile) [extras]
				Build target project, make sure Makefile has EXTRA_CFLAGS
	do_sh:
				Execute bash command
	set_plugin_dir:
		(plugin_dir)	Set the plugin directory, the original still count
	showlog:
		Show current log messages
	load_srcfile:
		[srcfile]
	set_srcfile:
		(srcfile_name)
	getinfo:
		(res_path) (is_builtin) (linux_kernel?) (step)
			Get information of resfile, for step
				0 Get all information
				1 Get base information
				2 Get detail information
				3 Get xrefs information
				4 Get indirect call information
				5 Check if all GIMPLE_CALL are set
	staticchk:
		Run registered static check methods
	itersn:
		[output_path]	Traversal all sinodes to stderr/file
	sn_load:
		[id]		most for test cases
	========= USAGE END =========

	SRCINV> list_plugin
	0	0	loaded		fuzz
	>>>> /home/zerons/workspace/todo/srcinv/plugins/fuzz.so
	1	0	loaded		sibuf
	>>>> /home/zerons/workspace/todo/srcinv/plugins/sibuf.so
	2	1	loaded		staticchk
	>>>> /home/zerons/workspace/todo/srcinv/plugins/staticchk.so
	3	0	unload		(null)
	>>>> /home/zerons/workspace/todo/srcinv/plugins/test.so
	4	1	loaded		sinode
	>>>> /home/zerons/workspace/todo/srcinv/plugins/sinode.so
	5	7	loaded		getinfo
	>>>> /home/zerons/workspace/todo/srcinv/plugins/getinfo.so
	6	1	loaded		debuild
	>>>> /home/zerons/workspace/todo/srcinv/plugins/debuild.so
	7	0	loaded		sn_load
	>>>> /home/zerons/workspace/todo/srcinv/plugins/sn_load.so
	8	0	loaded		uninit
	>>>> /home/zerons/workspace/todo/srcinv/plugins/uninit.so
	9	1	loaded		resfile
	>>>> /home/zerons/workspace/todo/srcinv/plugins/resfile.so
	10	4	loaded		src
	>>>> /home/zerons/workspace/todo/srcinv/plugins/src.so
	11	1	loaded		gen_sample
	>>>> /home/zerons/workspace/todo/srcinv/plugins/gensample.so
	12	0	loaded		c
	>>>> /home/zerons/workspace/todo/srcinv/plugins/c.so
	13	0	loaded		itersn
	>>>> /home/zerons/workspace/todo/srcinv/plugins/itersn.so
	14	1	loaded		utils
	>>>> /home/zerons/workspace/todo/srcinv/plugins/utils.so

	list_plugin显示当前可供使用的plugin. 显示的信息包含:
	序号	引用计数	是否加载	plugin名称	plugin路径
```
更多的指令详细用法, 请参考项目仓库doc/commands.md文件.
框架的使用分为三个阶段, 信息收集, 信息解析, 信息使用. 下面将依次介绍.



### 3 - 信息收集
注意: 信息收集之前, 需要删除之前的结果文件.

对单个文件, 可以使用gcc的-fplugin, -fplugin-arg参数编译, 对使用make进行编译的项目,
确保Makefile提供了类似EXTRA_CFLAGS的编译参数, 此时可使用
`make EXTRA_CFLAGS+='-fplugin=/.../x.so -fplugin-arg-x-output=/.../...'` 编译

信息收集, 是使用编译器提供的接口, 提取源码编译时的中间信息, 进而对整个项目的信息
进行汇总. 此功能实现在collect目录下, 独立于主程序, 需要在项目编译时使用. 本质为
编译器插件.

关于GCC插件的使用, 该框架处理的是low-level GIMPLE形式的语句, 它的形成过程是:
```
	前端语言分析源码文件, 生成前端语言的AST表示
	将前端语言的AST表示转换成GIMPLE中间表示
```
然后GCC对GIMPLE中间表示进行处理, 这些过程叫pass. 包括从高级GIMPLE转换成低级GIMPLE(框
架使用的), IPA处理, GIMPLE优化, 最终由GIMPLE转换成RTL等.
Pass根据处理的对象及功能的不同, 分为四大类: GIMPLE_PASS, RTL_PASS, SIMPLE_IPA_PASS,
IPA_PASS. 其中GIMPLE_PASS以GIMPLE中间表示为处理对象, RTL_PASS的处理对象为RTL中间表示,
SIMPLE_IPA_PASS和IPA_PASS处理对象也是GIMPLE中间表示, 但功能主要是过程间分析(IPA,
Inter-Procedural Analysis). 例如如下几个PASS(新版本GCC有些pass可能不存在了):
```
	all_lowering_passes
	|--->	useless		[GIMPLE_PASS]
	|--->	mudflap1	[GIMPLE_PASS]
	|--->	omplower	[GIMPLE_PASS]
	|--->	lower		[GIMPLE_PASS]
	|--->	ehopt		[GIMPLE_PASS]
	|--->	eh		[GIMPLE_PASS]
	|--->	cfg		[GIMPLE_PASS]
	...
	all_ipa_passes
	|--->	visibility	[SIMPLE_IPA_PASS]
	...
```
对源码中每个定义的函数, 会先执行一遍all_lowering_passes中的处理过程, 也就是说, 在处理
到cfg pass的时候, 有些函数并未进行lower处理.
更多的GCC插件信息, 可以参考refs[1] refs[3], 或者查询GCC源码.

当前实现的针对c源码文件的信息提取, 是gcc插件, 在加载时会检测当前编译的文件名称, 匹
配文件路径(主要针对内核源码结构进行的检测), 注册回调函数, 在cfg pass执行之前调用回
调函数. 在PLUGIN_ALL_IPA_PASSES_START执行时将数据写入文件. 文件大小使用PAGE_SIZE对齐.

这里我们不能使用PLUGIN_PRE_GENERICIZE来处理每个tree_function_decl, 原因在于, gcc在
这个处理阶段, 是流式的, 可能一个函数调用的函数还并未定义. 比如
```
	static void test_func0(void);
	static void test_func1(void)
	{
		test_func0();
	}

	static void test_func0(void)
	{
		/* test_func0 body */
	}
```
这种情况, 处理test_func1时, 会跟入test_func0, 而此时的test_func0的tree_function_decl
结构体并未进行完整的初始化.

当gcc完成整个源码文件的扫描工作, 所有的数据会被串起来(TREE_CHAIN), 我们需要尽可能的
避免数据的重复, 并保证数据的完整性.

当调用该pass的处理程序时, 处理对象为struct function, 其成员decl指向包含这个对象的
tree_function_decl. 这个过程会将tree_function_decl->saved_tree成员(为函数体)保留的
信息转换成GIMPLE语句保存到tree_function_decl->f->gimple_body中. 此时, 由于所有的函数
已被串起来, 我们在跟踪tree_function_decl的时候, 需要判断这个结构是否已经完成AST->
GIMPLE的转换, 如果还存在saved_tree, 不处理这个函数, 后续转换这个函数时会完成该函数
的信息收集.

对于tree_var_decl(变量), 我们着重需要记录的是非局部变量的信息. 在gcc的实现中, 函数
is_global_var的实现(gcc/gcc/tree.h)如下:
```
	static inline bool is_global_var (const_tree t)
	{
		return (TREE_STATIC(t) || DECL_EXTERNAL(t));
	}
```
TREE_STATIC对于变量来说, 表示这个函数是否使用静态存储区, 如果使用, 表明这个变量is
global variable. DECL_EXTERNAL表示该变量是否是外部引用. 当前实现的c.cc在此基础上添
加了一个检测:
```
	if (is_global_var(node) &&
			((!DECL_CONTEXT(node)) ||
			 (TREE_CODE(DECL_CONTEXT(node)) == TRANSLATION_UNIT_DECL))) {
			objs[start].is_global_var = 1;
		}
```
DECL_CONTEXT为空或者为TRANSLATION_UNIT_DECL时, 表示该变量为函数外变量.

另外, 需要记录每个对象的指针, 每个位置信息等.

该collect/c.cc在linux kernel 4.14.x上的测试显示, make vmlinux -j9耗时大致为20分钟,
提取出的信息文件为19.4G.



### 4 - 信息解析
解析是框架需要实现的主要功能之一, 也是最复杂的. 我们需要完成提取的项目信息的整理和
分类, 构建索引信息, 方便后续的使用.

该实现主要在plugin/getinfo.c中, 其对每个编译文件的信息进行检测, 查看该文件编译类
型(enum si_lang_type), 查找是否有对应的注册函数(struct lang_ops), 并依次调用.

针对大型项目的解析, 为了有更好的体验, 框架使用ncurses来提供进度条提示(框架编译时使
用make ver=release).

信息解析过程, 需要将提取信息文件(称为resfile)加载到内存. 而在linux kernel 4.14.x中
的测试, 生成的resfile达到19G之多, 无法一次加载到内存中, 且后续的信息使用亦需要这些
信息, 解决方案如下:
```
	自定义进程的内存布局. 在RESFILE_BUF_START位置开始加载resfile文件, 对每个编
	译文件生成一个sibuf结构体, 记录文件的加载位置, 加载大小, 以及加载的数据在
	文件中的偏移等信息.

	当加载到内存的文件大小超过RESFILE_BUF_SIZE时, 调用munmap取消之前的加载, 继
	续加载后续的数据.

	当需要读取之前已经munmap的数据时, 只需要调用resfile__resfile_load函数, 将
	sibuf对应的文件数据加载到适当的内存位置, 即可直接进行读写.

	内存布局如下图:
	0x0			--- 0x400000		NULL pages
	0x400000		--- 0x403000		si_core指令
	0x602000		--- 0x603000		si_core数据
	0x603000		--- 0x605000		si_core数据
	0x605000		--- 0x647000		heap
	SRC_BUF_START		--- RESFILE_BUF_START	索引信息区域
	RESFILE_BUF_START	--- 0x????????		resfile加载区域
	0x700000000000		--- 0x7fffffffffff	线程 动态库 plugins 进程栈

	当SRC_BUF_START为0x100000000, RESFILE_BUF_START为0x1000000000时, 索引信息最
	高可达到64G, resfile可处理高达1024G(末端到达0x100 0000 0000)的数据文件.
```



##### 4.1 - 信息调整
信息收集阶段生成的文件, 因为包含了很多指针, si_core进程并不能直接读写其中的信息, 需
要先完成适当的指针转换. 此过程需要与collect/x.cc对应. 名为PHASE1前半部分.

读取函数信息, 对其中包含的指针数据修改之后, 依次对每个对象进行访问并调整指针数据. 同
时, 对location结构, 由于location_t只有4字节大小, 设置其为sibuf->payload的偏移位置,
于是`*(expanded_location *)(sibuf->payload + loc)`即可获取需要的位置信息.

当所有对象调整完毕, 即完成PHASE1, 设置完文件状态信息, 返回.



##### 4.2 - 基础信息获取
PHASE1后半部分, 提取每个文件的基础信息: 有哪些定义了的函数, 非局部变量, 类型.

函数分TYPE_FUNC_GLOBAL和TYPE_FUNC_STATIC,
非局部变量分TYPE_VAR_GLOBAL和TYPE_VAR_STATIC,
类型分TYPE_TYPE_LOC和TYPE_TYPE_NAME.

如果类型为TYPE_FUNC_GLOBAL/TYPE_VAR_GLOBAL/TYPE_TYPE_NAME, 为名称索引
如果类型为TYPE_FUNC_STATIC/TYPE_VAR_STATIC/TYPE_TYPE_LOC, 为位置索引
如果类型为TYPE_NONE, 该节点为tree_type_non_common, 放入sibuf->type_nodes中.

首先查询当前sinodes中, 是否有重复的节点, 如果存在, 为位置索引时则进行下次循环, 为名
称索引时需要检测名称冲突, 比如对于TYPE_*_GLOBAL, 存在weak symbol.

生成新的sinode, 根据当前得到的位置 名称等信息, 完成初始化.



##### 4.3 - 详细信息获取
PHASE2, 获取每个sinode节点的详细信息.

对于类型, 需要获取该类型指向的类型, 或者类型的大小, 类型的成员等信息.
对于变量, 当前只获取了变量是什么类型.
对于函数, 首先获取返回值类型, 然后处理参数列表(以var_node_list表示), 然后获取函数体(
以code_path表示).

关于函数体的表示, 以label为分隔点, label之后的语句为一个code_path, 直到另一个label或者到GIMPLE_SWITCH/GIMPLE_GOTO/GIMPLE_COND/含nl的GIMPLE_ASM语句. 同时会检测该函数中是否有不可到达的语句, 比如:
```
	static int test_func(void)
	{
		int err = 0;
		if (err)
			return 1;
		else
			return 0;
		return 0;		/* not reachable */
	}
```



##### 4.4 - 交叉引用信息整理
PHASE3, 处理非局部变量的初始化值, 设置每个变量的可能值(possible_value_list).

对每个函数(除了直接调用)和变量的使用位置(use_at_list)进行标记. 并获取直接调用信息.

每个函数的调用均由GIMPLE_CALL语句表示, 其第一个操作数为返回值, 第二个操作数可能为函
数地址, 也可能为VAR_DECL/PARM_DECL等, 后面的操作数为函数的实际参数. 直接调用即为第二
个操作数为函数地址的情形. 记录的函数使用位置是除第二个操作数外的所有引用位置.

PHASE3暂时还未通过linux kernel 4.14.x的vmlinux的resfile检测, 存在一些未考虑到的情形.



##### 4.5 - 间接调用1
PHASE4, 处理被标记了的函数, 即表示该函数存在除直接调用之外的引用情形.
如果引用语句为GIMPLE_ASSIGN(赋值语句), 获取左值的var_node, 添加possible_value.



##### 4.6 - 间接调用2
PHASE5, 处理GIMPLE_CALL语句的第二个参数为VAR_DECL/PARM_DECL的情形.

如果为VAR_DECL情形, 获取变量的var_node, 查看possible_value_list, 添加调用关系;
然后查看该变量的use_at_list, 检测对其的赋值, 然后递归跟踪.
例如:
```
	static void test_func0(void);
	static void test_func1(void)
	{
		void (*testf)(void);
		testf = test_func0;
		testf();
	}
```
或:
```
	static void test_func0(void);
	struct test_a {
		int a;
		void (*b)(void);
	};
	static struct test_a static_a = {
		.a = 1,
		.b = test_func0,
	};
	static void test_func1(void)
	{
		static_a.b();
	}
```
可以得到test_func1调用了test_func0的结果.

暂时未提供处理PARM_DECL调用情形的功能.



### 5 - 信息使用
对索引信息的使用, 可以有多种可能. 由于我们收集的信息是lower-level形式的GIMPLE语句,
所以编写的框架plugin处理的对象也是lower GIMPLE语句.

这里以未初始化变量引用的某种情形进行简单说明
例如如下代码:
```
	static void test_func(int flag)
	{
		int need_free;
		char *buf;
		if (flag) {
			buf = (char *)malloc(0x10);
			need_free = 1;
		}

		/* do something here */

		if (need_free)
			free(buf);
	}
```
plugins/uninit.cc显示了如何检测这种形式的问题.
该plugin会对所有的函数依次进行检测, 首先, 调用utils__gen_code_path获取该函数所有可能的执行流, 然后:
```
	获取该函数使用的一个局部变量(不包含函数内的static变量)
	遍历所有执行流, 查看该局部变量的第一个引用位置
	如果第一个引用位置的操作是读取该变量, 则存在未初始化变量引用的情形.
```
查看检测[视频](https://www.youtube.com/watch?v=anNoHjrYqVc)



### 6 - 后续
+ 对linux kernel生成的resfile的完整支持
 + 完全通过所有解析步骤
 + .s/.S文件中包含的符号提取, 完善调用链.
 + 对添加了内核防护的版本的辨识
 + ...
+ 变量等数据的跨函数的追踪
+ 标记数据引用位置的操作, 比如读还是写, 或者取地址等
+ 对用户层应用的解析的完善, 比如很多符号都是外部库的
+ 跨函数的所有执行流的生成
+ 执行流之间的依赖关系, 比如sys_read函数的调用需要sys_open的输出
+ 验证样本的自动生成
+ 某些形式的代码问题的补丁自动生成
+ 其他编程语言的支持
+ ...

欢迎各种建议以及新的思路, 当前, 提交请求(Push Request)也是极好的.



### 7 - 引用
  [0] [项目地址, 仓库会在后续创建](https://github.com/snorez/srcinv/)

  [1] [GNU Compiler Collection Internals](https://gcc.gnu.org/onlinedocs/gcc-6.4.0/gccint.pdf)

  [2] [GCC source code](http://mirrors.concertpass.com/gcc/releases/gcc-6.4.0/gcc-6.4.0.tar.gz)

  [3] [深入分析GCC](https://www.amazon.cn/dp/B06XCPZFKD)

  [4] [gcc plugins for linux kernel](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/scripts/gcc-plugins?h=v4.14.85)
