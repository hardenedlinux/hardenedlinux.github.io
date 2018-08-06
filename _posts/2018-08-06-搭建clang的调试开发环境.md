---
layout:         post
title:          搭建clang/LLVM的调试开发环境
data:           2017-08-12
auther:         zet
mail:           zet@tya.email
---


# 搭建clang/LLVM的调试开发环境
@(LLVM)[clang/LLVM]
        --[zet](https://github.com/fanfuqiang)


## 00 导引

对于某个软件的开发搭建调试环境应该是最基础的一步，但是据说很多人都卡在了这一步而
放弃。

最近因为跟着某低温老师学习讨论LLVM的pass相关的东西，又把LLVM捡起来重新编译了一个
可以调试的版本。再加上吴老板的建议所以写篇文章来给后人指路。

如果你自认为自己是一个调试环境都搭不起来的沙雕，那么这篇文章就是写给你看的。xD

## 01 细节

Linux下的系统编译就三板斧，configure/make/make install。configure是最有技术含量
的。下面我以clang/LLVM为例写一下我面对新系统编译的处理和思考。

因为我只想参与跟低温老师针对LLVM pass的讨论，所以我找了一个比较老版本的LLVM，再
加上因为microblaze target的最后一个仍在LLVM里的版本是3.3，所以我的调试环境版本就
是clang/LLVM 3.3。为什么是microblaze？因为简单。

下载clang/LLVM的代码解压，官网下载llvm代码解压之后应该是一个名字是llvm-3.3.src的
文件夹，将解压后的clang源代码文件夹放入llvm-3.3.src/tools里并且改名为clang。至于
为什么要这样做，是因为LLVM/tools/CMakeLists.txt里有这样一行：

```
add_llvm_external_project(clang)

```
接着进入llvm-3.3.src这个目录。

```
# 创建一个新的编译目录并进入
mkdir build && cd build
# 第一步，configure
configure --prefix=/home/zet/bin/llvm-33 --enable-debug-runtime --enable-debug-symbols
 --enable-keep-symbols --enable-threads="no" --enable-pthreads="no"
 --enable-shared="no" --enable-targets=host,cpp,mblaze
# 第二步， make
make CFLAGS="-g3 -O0" CXXFLAGS="-g3 -O0"
# 第三步，install
sudo make install
```
解释一下上面的参数和来源，打开llvm-3.3.src/configure文件，搜索enable会看到一堆的
可选参数及其选择之后的影响。debug和symbols的那三个选项很明显。--enable-threads/
--enable-pthreads这两个选项看解释不清楚是build的时候多线程还是生成的binary使用多
线程,不过我猜测是后者，因为不加这两个参数的时候我遇到过libthread_db的assert内部
错误，触发了gdb的内部错误，gdb的代码我不懂，也不想深究。再就是我只是想研究LLVM的
pass细节。--enable-shared的意思就是编译为shared library。shared有一个问题就是会
使调试困难，gdb里面查看bt的结果如果是shared会看到ld-linux.so这种类型的东西，因为
shared本身就是运行时才load进进程空间的，需要调用glibc的loader，如果你想看到
ld-linux.so的东西，会是另外一个话题，你得下载libc6的debug符号以及源代码，使用gdb
的*set debug-file-directory*和*set directory*，你可以阅读gdb手册了解一下。至于最
后一个关于target的参数为什么这么指定，你可以搜索configure这个文件里的targets这个
关键字，从里面找相关内容看，查询configure和阅读手册是非常重要的手段。

因为照着这个configure应该是一个static的clang,所以最后的链接阶段会极度占用内存。
吃内存的软件最好退出，比如浏览器。

测试的时候有两个非常好用的参数*--verbose/-save-temps*，会把所有的中间文件存到当
前目录，也会打印出clang内部添加的参数和调用其他binary的细节。

```
# gdb后面加入编译出的可以调试的clang
gdb /home/zet/bin/llvm-33/clang
# 将--verbose输出的参数加到args后面
(gdb) set args XXXXXXXXXXXXX
(gdb) b main
```
至此应该是一个搭好的调试环境了。

## 02 后记

上面的基本上就是我所遇到一个编译器的时候所有操作。推荐大家没事干的时候多读手册。
因为工作的关系最近两年基本上都是在做gcc，对比LLVM/gcc的pass来看，gcc真的好清晰，
但是作为一个library来说LLVM无疑是成功的。在2016年的时候LLVM meeting san jose回来
跟北邮的刘正阳(他是那年的LLVM student code项目参与者之一)机场聊天讨论到为什么
LLVM的pass可以这么乱，他说他问过某个编译器厂商的CTO这个问题，那位CTO的解释是LLVM
要搞library这种模式就会这样。后来我想是不是pass之间耦合度太低的问题导致的？

*live long and prosper.*
