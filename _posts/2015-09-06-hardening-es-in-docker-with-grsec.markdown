---
layout: post
title:  "社区最佳实践：基于PaX/Grsecurity & STIG & Sheild针对es的Docker场景化加固"
date:   September 6, 2015 10:42 PM
summary: 随着Container技术的逐步成熟，Docker的应用也越来越广泛，越来越多的重要业务依赖于Docker运行环境，针对场景化的加固是基于GNU/Linux系统重要工作，本文基于Docker里运行es为应用来进行场景化加固的探讨关于PaX/Grsecurity，STIG-4-Debian以及Sheild的实践。
categories: system-security

---

By: 炼石网络CipherGateway

# Hardening es(in docker) with PaX/Grsecurity & STIG & Shield

**文档说明**：

*基础平台*：此实践文档使用[PaX/Grsecurity](http://https://en.wikibooks.org/wiki/Grsecurity/Appendix/Grsecurity_and_PaX_Configuration_Options#Disallow_access_to_overly-permissive_IPC_objects)完成Linux内核级别的加固，并采用美国DISA组织的STIG脚本对Debian GNU/Linux发行版进行安全扫描，以使Debian GNU/Linux发行版安全达到一定的高度。

*业务方面*：以在Docker下运行的ElasticSearc服务为例，结合ES的[Shield](http://https://www.elastic.co/guide/en/shield/current/index.html)插件对ES进行业务级别的加固。


**基本步骤如下：**

1、最小化安装Debian8 GNU/Linux

2、使用PaX/Grsecurity加固

3、使用STIG验证

4、使用Shield对ElasticSearch进行业务级别的加固

5、完善物理机RBAC系统，达到最小权限要求

-----------------

## 1、最小化安装Debian8 GNU/Linux系统

 1）在服务器上插入U盘，连接好网线，配置网络：

![](http://i.imgur.com/Y8ZBvBJ.png)

 2）为硬盘分区：

![](http://i.imgur.com/omVXCcF.png)

 3）软件环境安装：

![](http://i.imgur.com/GV5UIz9.png)


![](http://i.imgur.com/sHqiWQv.png)

4）系统安装完成：

![](http://i.imgur.com/zBfEUEQ.png)

### 加固前准备

1）在root下打开/etc/apt/sources.list，确定是否有以下三个debian的源，若没有自行添加,并且注释或删掉其他的软件源，尽量避免使用非社区源：
 
    nano /etc/apt/sources.list
    deb http://ftp.debian.org/debian jessie main contrib
    deb http://security.debian.org/jessie/updates main contrib
    deb http://ftp.debian.org/debian jessie-updates main contrib
	
	apt-get update


2）安装sudo

	apt-get install sudo
	
修改sudoers文件，添加一句（等效于添加当前用户到sudo用户组，或者直接使用root用户可忽略此步）:   

     nano /etc/sudoers
     add "user   ALL=(ALL:ALL)ALL"

3）安装make

	sudo apt-get install make

------------------------

## 2、使用PaX/Grsecurity加固

这次加固主要用到的是一个增强内核安全的工具Grsecurity，和SELinux以及Apparmor一样，是用来控制文件访问权限的安全工具。

1）Grsecurity需要打包到内核，所以需要重新编译内核，由于grsec长期支持的内核版本是3.14.48，而debian8的内核版本是3.16，所以我们要将内核版本降到3.14.48，查看内核版本命令:

    cat /proc/version
	
2）到kernel.org下载相应的目标内核版本源代码:

     wget https://kernel.org/pub/linux/kernel/v3.x/linux-3.14.48.tar.xz

3)到Grsecurity下载相应的patch，注意必须要和内核版本一致(先到http://grsecurity.net/download.php查看“grsecurity - stable kernel patch”和“paxctld - PaX flags maintenance daemon - binary packages”的版本):

     wget http://grsecurity.net/stable/grsecurity-3.1-3.14.48-201507261203.patch
     wget http://grsecurity.net/paxctld/paxctld_1.0-2_amd64.deb


4）编译前安装支持插件的gcc：

     apt-get install libncurses* kernel-package build-essential
     gcc --version
     apt-get install gcc-`gcc --version`-plugin-dev（gcc --version用大版本即可 例如4.9而不是4.9.2）

5）解压缩内核包和补丁包，给内核打patch:

    xz -d linux-3.14.48.tar.xz
    tar -xvf linux-3.14.48.tar
    cd linux-3.14.48/
    patch -p1 < ../grsecurity-3.1-3.14.48-201507261203.patch
	
6）配置内核：

    make menuconfig

### 开始配置Grsecurity选项

*Note：该文档的配置选项仅测试在Debian8 GNU/Linux中。（业务：启动docker，并在其中运行ElasticSearch服务）*

关于Grsecurity具体的配置选项请参照：[https://en.wikibooks.org/wiki/Grsecurity/Appendix/Grsecurity_and_PaX_Configuration_Options](https://en.wikibooks.org/wiki/Grsecurity/Appendix/Grsecurity_and_PaX_Configuration_Options)

7)开启Grsecurity支持
 
执行命令后会出现配置窗口：

	在配置界面按下键盘选择"security options"

![](http://i.imgur.com/HGIQswO.png)

	security options --> Grsecurity

![](http://i.imgur.com/CYR2XEe.png)

	按空格选上Grsecurity(NEW)
  
![](http://i.imgur.com/RW83QSo.png)

![](http://i.imgur.com/w85poOM.png)

8）选择automatic后Grsecurity将自动根据通用标准形成一系列配置，但仍可在
Customer Configuration中进行更细致的加固措施的划分，二者效果会叠加。

	Configuration Method -->选择 automatic

![](http://i.imgur.com/GYO41kc.png)

9）优先选项选择Security优先

	Required Priority-->Security

![](http://ww2.sinaimg.cn/large/d8c6c741jw1evo9ufrg99j20ke0cajth.jpg)

10）去掉mprotect限制。因为Grsecurity默认禁止mprotect功能，而这会使JVM无法运行

	Security Options -->Grsecurity -->customize configuration -->PAX -->Nonexcutable pages-->disable  restrict mprotect

![](http://ww4.sinaimg.cn/large/d8c6c741jw1ev1140ill5j20ll0dqwhl.jpg)
![](http://ww1.sinaimg.cn/large/d8c6c741jw1ev115fx6koj20lr0dogog.jpg)
![](http://ww4.sinaimg.cn/large/d8c6c741jw1ev1166ekopj20lq0dg77k.jpg)

11）为了docker能正常运行，去掉chroot下'chmod+s'和'mknod'的限制

	Security Options -->Grsecurity -->customize configuration -->FileSystem Protections-->Deny (f)chmod +s && Deny mknod

![](http://ww4.sinaimg.cn/large/d8c6c741jw1evo9ux6odcj20ke0cewhz.jpg)

12)去除USB限制（看业务需求）

	Security Options -->Grsecurity -->customize configuration -->Physical Protections-->Deny new USB connections after toggle

![](http://ww2.sinaimg.cn/large/d8c6c741jw1evo9vgclhoj20kh0cadil.jpg)

13)其余选项保持默认，Save之后退出内核配置窗口。

14)完成之后执行编译内核命令，这一步需要较长时间

    make deb-pkg -j'cpu核心数+1' 

15）编译之后安装：

    cd ..
    dpkg -i linux*.deb 
    dpkg -i paxctld_1.0-2_amd64.deb

16）安装完成重启即可，重启时选择新内核

    paxctld -d
    sudo reboot

在启动界面选择高级选项

![](http://ww4.sinaimg.cn/large/d8c6c741jw1evrj57c193j20iv0dwwgu.jpg)

选择已加固的内核

![](http://ww3.sinaimg.cn/large/d8c6c741jw1evrj6wg06dj20ii0dxad7.jpg)

17) 重启之后

    su - root
    paxctld -d
	exit

18)安装[gradm](https://en.wikibooks.org/wiki/Grsecurity/The_Administration_Utility)工具

*这是RBAC系统的管理工具，使用它可以配置/etc/policy规则文件来定义每个用户的权限，也可通过gradm的学习模式来自动生成policy文件。*

**Note：安装gradm之前必须保证在已有Grsecurity加固的内核上，并且系统有（lex或flex）和（byacc或bison）**

下载相应的组件：

	apt-get install flex
	apt-get install bison

下载gradm：（或者到Grsecurity官网下载最新版）

	wget http://grsecurity.net/stable/gradm-3.1-201507191652.tar.gz

开始安装：

	tar zxvf gradm-3.1-201507191652.tar.gz
	cd gradm
	make nopam (无PAM支持）
	sudo make install

安装结束后，要求输入gradm的管理密码，注意不要与root密码相同，否则降低了Grsecurity的安全防护。
另外，将会在/sbin下安装gradm和grlearn程序。并在/etc/grsec目录下生成learn_config和policy两个文件。

*关于使用gradm对系统权限进行细分将在下面提及。*

19)	卸载旧内核(*Note：dpkg确定内核版本号，remove掉相应的内核deb包，请替换后再执行命令*)
	
	dpkg -l | grep linux-headers（若没有3.16的headers则可略去下面的apt-get remove linux-headers-''）
	dpkg -l | grep linux-image
	sudo apt-get remove linux-headers-'' linux-image-''(注意：这一步使用前两个命令得到的信息替换''里的内容）

此时再重启计算机会发现现在只有Linux 3.14.48-grsec的内核了。
	
20） 卸载编译工具链

	sudo apt-get remove kernel-package build-essential
	sudo apt-get remove libncursesada-dbg  libncursesada3-dev libncursesada-doc libncurses5-dbg libncurses5-dev libncursesw5-dbg  libncurses-gst libncursesw5-dev libncursesada3
	sudo apt-get autoremove

至此Grsecurity的第一步加固完成。若以后在系统运行时需要修改某些特性，可在/etc/sysctl.conf中对相应的条目进行修改。具体条目对应的选项请参看[https://en.wikibooks.org/wiki/Grsecurity/Appendix/Grsecurity_and_PaX_Configuration_Options](https://en.wikibooks.org/wiki/Grsecurity/Appendix/Grsecurity_and_PaX_Configuration_Options)

------------

## 3、使用STIG验证

*STIG（安全技术实现指南）是由DISA为了IT安全态势给DoD(美国国防部）提供的一套防御指南。*

**NOTE：STIG是一组庞大的集合，且针对OS的部分官方只针对RHEL。这里的STIG指的是h4rdenedzer0(http://hardenedlinux.org/about/) 的stig-4-debian项目**

1） 将github中STIG的脚本文件克隆到本地：

	git clone https://github.com/hardenedlinux/STIG-4-Debian.git
	cd STIG-4-Debian

2） 使用方法

	usage: check.sh [options]

	  -c    Output Log with catable colors
	  -s    Perform STIG checking with NORMAL output log
	  -v    Show version
	  -h    Show this message

3） 执行脚本

	sudo ./check.sh -s

4) 脚本对内核执行完毕后我们可以看到在/var/log下生成了一个100K左右名为STIG-Checking-2015-??-??.log的日志，即为STIG对内核进行安全扫描的结果。

其中，日志中为FAIL的条目表示未达到STIG安全标准的项目，PASS的项目为通过STIG验证的项目。我们接下来要做的工作就是通过修改内核选项尽可能地使所有FAIL的条目变成PASS。

5）示例：

![](http://ww3.sinaimg.cn/large/d8c6c741jw1evrrl7g6obj20m60dk409.jpg)

在上图中可看到一个FAIL的条目是"The /etc/shadow file must be group-owned by root",Vulnerability是该条目的漏洞分析，Fix text是解决方法。

为了解决该问题，执行Fix text中提示的命令：

	chgrp root /etc/shadow

修改后：

![](http://ww1.sinaimg.cn/large/d8c6c741jw1evrs5ykn7bj20kn018t8m.jpg)

可以看到，按Fix text修改后该条目通过STIG验证。

6）STIG总结

STIG是由DISA为了IT安全态势给美国国防部提供的一套安全指南，据说DoD的应用在上线前都会使用这个指南进行安全强化。在日志内容上看来也的确有不少值得借鉴之处。

综上，尽可能把其余的FAIL条目按照STIG标准进行修改达到PASS，使系统安全到达更高一个层次。

------------

以上即为Linux内核加固的实践内容。接下来是结合ElasticSearch进行业务层面的加固，具体部分包括：

* 在docker下运行ES，并使用Shield对ES进行加固————安全认证及RBAC系统

* 在物理机上配置RBAC系统，达到系统最小权限————主要为gradm工具的使用

## 4、使用Shield对ElasticSearch进行业务级别的加固

### 介绍Shield

Shield是一个为了使ElasticSearch更安全而产生的一个插件。在Shield的安全加固中包括安全认证机制，RBAC系统，IP过滤和系统审计。

下面的加固将通过配置安全认证机制和RBAC系统来演示加固流程。

    ### 配置ElasticSearch

    1） 安装docker

        apt-get install curl
        curl -sSL https://get.docker.com | sh

    验证docker是否安装成功：`$ docker`

    2）启动docker

        sudo service docker start

### 安装Shield

1) 安装需求:

* 安装了Java7或更新版本
* Shield plugin必须在集群的每个节点都安装，并且每个节点安装完后都要重启。

2) 启动ES的每个节点：（oaDataNode1等是自定义的节点名称）

    sudo service docker start
    sudo weave launch
    sudo weave start 10.0.0.1/24 oaDataNode1
    sudo weave start 10.0.0.2/24 oaDataNode2
    sudo weave start 10.0.0.3/24 oaDataNode3
    sudo weave start 10.0.0.4/24 oaDataNode4
    sudo weave expose 10.0.0.254/24

3) 首先进入Docker：

	sudo docker exec -it oaDataNode1 /bin/bash

4) 进入ES的安装目录下：

	cd /usr/es/elasticsearch-1.7.0/bin

5) 安装ElasticSearch许可插件和Shield插件：：

	./plugin -i elasticsearch/license/latest
    ./plugin -i elasticsearch/shield/latest

6）重启ElasticSearch服务

    sudo weave stop
    sudo docker stop oaDataNode2 oaDataNode1 oaDataNode3 oaDataNode4
    sudo weave launch
    sudo weave start 10.0.0.1/24 oaDataNode1
    sudo weave start 10.0.0.2/24 oaDataNode2
    sudo weave start 10.0.0.3/24 oaDataNode3
    sudo weave start 10.0.0.4/24 oaDataNode4
    sudo weave expose 10.0.0.254/24

### 安全认证模块

1) 新建一个ElasticSearch管理员账户

	./esusers useradd es_admin -r admin(新建一个名为es_admin的用户，角色为admin)

现在可以尝试着用RESTFUL API来访问ElasticSearch，会发现访问被拒绝

    curl -XGET 'http://localhost:9200/'

在请求上加上用户名和密码即可：

    curl -u es_admin -XGET 'http://localhost:9200/'

由此可知，使用Shield新建ES账户后，可以为ES添加严格个安全认证机制。

### RBAC模块

#### 知识准备

Shield实现了RBAC权限管理系统。在RBAC系统下，所有的行为都会被默认限制，所有用户的权限都与“角色”联系在一起，而每个“角色”代表了一组允许的行为。

##### 1）Roles, Permissions and Privileges的概念

* Privileges：代表一组用户在ElasticSearch中允许执行的行为。例如：是否能运行query就是一种Privileges。

* Permissions：一组结合一个或多个“安全对象（Secured Object）”的Privileges。

>在ElasticSearch中有两种Secured Object：
>>Cluster：Cluster permissions提供了集群范围内的管理、侦听权限。

>>Index：Index permissions提供了数据通道，包括在集群中特定索引的管理和侦听。

* roles：一组命名的Permission的组合。例如：你可以定义一个“日志管理员”的role，他允许对名字为“logs-*”的索引做任何操作，但对其他文件却没有任何权限。

*Note:作为ES的管理员，你需要定义一些角色，并将每个用户分配到这些角色上。*

##### 2) 定义roles

Roles在ES_HOME/config/shield中的roles.yml文件中定义，其中的每个条目都定义了唯一一个role和相应的permission。

**Note：roles.yml文件在每个节点中独立管理。所以在拥有多个节点的集群中，所有节点的该文件都需要修改。**

roles.yml文件的格式如下：

    # All cluster rights
    # All operations on all indices
    admin:
      cluster: all
      indices:
        '*': all

    # Monitoring cluster privileges
    # All operations on all indices
    power_user:
      cluster: monitor
      indices:
        '*': all

    # Only read operations on indices named events_*
    events_user:
      indices:
        'events_*': read

具体的Privileges请参见：[https://www.elastic.co/guide/en/shield/current/reference.html#ref-actions-list](http://)

##### 3） 使用esusers工具管理用户

esusers工具主要控制在ES_HOME/config/shield/的两个文件————users和users_roles。这两个文件存储着所有esuser域的信息，并且会在ES启动时被shield读取。

**Note：roles.yml文件在每个节点中独立管理。所以在拥有多个节点的集群中，所有节点的该文件都需要修改。**

* users文件：

>users文件储存着所有用户和他们的密码。每个条目是每个用户的名字和密码的hash值。

* users_roles文件：

>users_roles存储的是每个用户相联系的roles

###### A、添加用户

    esusers useradd <username>
    esusers useradd <username> -r <comma-separated list of role names> 使用-r参数定义用户的roles，roles间用逗号隔开

ES在启动时会读取用户和角色信息，应该使用开启ES的用户使用useradd添加用户。

###### B、打印用户信息

    esusers list
    esusers list <username> 这个命令打印指定用户信息

###### C、修改用户密码

	esusers passwd <username>

###### D、分配用户角色

	esusers roles <username> -a <roles> -r <roles>
	-a指定添加，-r指定移除

###### E、删除用户

	esusers userdel <username>

#### RBAC加固实践

1） 修改roles.yml文件

    cd /usr/es/elasticsearch-1.7.0/config/shield
    vim roles.yml

在文件的末尾加入如下内容：

    Data_user: #Data_user 角色定义
      indices:  #index部分
        '*': crub  #指定Data_user可以读写所有indices

    Server_user:  #Serve_user 角色定义
      cluster: cluster:admin/nodes/restart, cluster:admin/nodes/shutdown

2） 添加两个用户user_Data和user_Server

    esusers useradd user_data -r Data_user
    esusers useradd user_server -r Server_user

这时，用不同的用户登陆ES将可以进行不同的操作：

* user_data：读写ES的所有文件

* user_server:启动或关闭ES服务

## 5、完善物理机RBAC系统，达到最小权限要求

### 知识准备

之前讲解了ElasticSearch上的RBAC系统的配置。在Grsecurity中也提供了RBAC权限控制系统，而且比ES中的划分更细致更严格。

管理物理机中的系统主要依靠gradm工具。使用`gradm --help`获得gradm的用法：

    # gradm --help
    gradm 3.1
    grsecurity RBAC administration and policy analysis utility

    Usage: gradm [option] ... 

    Examples:
            gradm -P
            gradm -F -L /etc/grsec/learning.logs -O /etc/grsec/policy
    Options:
            -E, --enable    Enable the grsecurity RBAC system
            -D, --disable   Disable the grsecurity RBAC system
            ...

/etc/grsec/policy文件是RBAC系统的核心。在RBAC启动后，它会根据该文件中所写定的策略对系统进行严格的访问控制。

#### policy中的三个概念role, subject, object

同Shield的RBAC相似，Grsecurity的RBAC也具有三个关于角色和权限的概念。

* role（角色）：如“role admin sA“，“role default ”，”role root uG“，表明有三个角色，分别是admin , default, root

>Roles: Users and groups on the system
>在policy中声明即可以定义角色，语法为”role < role name> < parameter>

>grsecurity中的角色分为用户角色，组角色，缺省角色（default），还有个管理员角色。定义不同角色需要有不同的参数。

> NOTE：grsecurity的角色与用户是一对一的。假设有tester用户，属于test组，那么tester用户登录后，会先在policy文件中匹配名为tester用户角色，如果没有，就会去匹配叫test的组角色，如果还没有，那么tester用户进入系统后的角色会是default（default角色在配置文件里面定义）。

>也就是说，tester用户登录后，不是tester用户角色，就是test组角色，要么就是default缺省角色，不可能进入其他policy中定义的角色。

* subject（程序），每个subject中首先定义了一个可执行程序（注意，一个subject中只定义一个可执行程序，要对多个可执行程序进行定义的话，就需要多个subject），或者更准确一点，定义了一个运行在系统中的进程。然后后面跟着一系列的object，用来规定当前这个进程的权限。每个角色定义的后面都可以跟一个或多个subject

>Subjects: Processes and directories
>参数：
>>h 这个进程是隐藏的，只能够被具有v模式的进程看到；
v 具有这个模式的进程拥有察看隐藏进程的能力；
p 进程是受保护的，这种模式的进程只能被具有k模式的进程杀死；
k 具有这个模式的进程可以杀死处于保护模式（p）的进程；
l 为这个进程打开学习模式；
o 撤销ACL继承

* object（对象）：每个subject中都会有若干的object，表示每个进程都有若干个操作对象，这些操作对象一般来说都是一些目录，文件等等，用来规定当前这个subject中的进程对这些文件拥有哪些权限

>Objects: Files and PaX flags
>参数：
>>r 这个对象可以打开阅读；
  w 这个对象可以打开写或者添加；
  o 这个对象可以打开添加；
  h 这个对象是隐藏的；
  i 这个模式只用于二进制可执行文件。当这个对象被执行时，它继承所在主体进程的访问控制列表；
  x: 这个模式代表文件可执行

**NOTE：配置文件中关于subject和object的所有路径都必须是绝对路径**

#### 例：

    role root uG
    subject /
    subject /usr/bin/ssh
    /etc/ssh/ssh_config r
    表示/usr/bin/ssh这个进程对/etc/ssh/ssh_config这个文件有读权限（r）。更进一步讲，这个subject位于角色root后面，所以这两行策略的含义就是以root角色运行ssh时，ssh进程对/etc/ssh/ssh_config有读权限。该策略只定义在了root角色中，对于其他角色不起作用。

缺省ACL

    注意：在每一个角色中，都必须有一个subject /，表示一个缺省的ACL，如果没有这个缺省的ACL，grsecurity启动时会报错同时启动失败。当以一个角色登录系统后，如果要执行的可执行程序没有被某个subject定义，那么，该程序就会采用subject /中的缺省定义。例如root角色，没有对cat命令进行定义，所以以root角色执行cat命令时，ACL系统就会参照subject /中的定义来控制cat进程对文件的访问。

### 部署RBAC系统

Grsecurity的policy文件极其繁大复杂，通过手工几乎不可能完成一个具有安全效力的规则文件。所幸Grsecurity为RBAC系统提供了学习机制，即可以通过你在学习机制下进行的操作自动配置policy文件。

Grsecurity的学习模式分为全系统学习模式跟基于角色的学习模式，在这里我们使用全系统学习模式:

1） 开启全系统学习模式

	sudo gradm -F -L /etc/grsec/learning.logs

![](http://ww3.sinaimg.cn/large/d8c6c741jw1evsx0lb4fij20pn02u0ta.jpg)

如图所示，RBAC学习模式开启成功。

**从现在开始，gradm将记录下你的一切操作。**

>*NOTE：*初次启动时RBAC可能会报出一些ERROR，原因是默认的policy文件没有配置完全：如图，该ERROR为未给shutdown用户设置密码。按照提示执行`gradm -P shutdown`并设置密码即可。

![](http://ww1.sinaimg.cn/large/d8c6c741jw1evswtm71ylj20q904gmxv.jpg)

2) 进行一些系统运行时正常的必要的操作，并重复至少4遍，以使gradm将操作记录下来

*NOTE：不要在学习模式下进行进行管理员操作，如果需要此类操作，先切换到RBAC管理员角色：*

	gradm -a admin

*在高危操作结束后，记得从管理员角色退出：`gradm -u`*

**3） 以针对ElasticSearch业务的加固为例，对RBAC进行配置：**

*业务分析：*

* 在docker中的ES内设置两个用户，一个只允许对其中数据进行读写操作，一个只允许在docker内部重启ES服务；这一步已在前文Shield的加固中实现。

* 在docker外设置一个本地用户，其除了拥有对本地系统的管理权限之外，还应对运行着ES业务的docker拥有启停等操作的权限；但不应拥有进入docker内部的权限。

在全系统学习模式下，对普通用户执行如下命令：

	启动docker:
    sudo service docker start
    sudo weave launch
    sudo weave start 10.0.0.1/24 oaDataNode1
    sudo weave start 10.0.0.2/24 oaDataNode2
    sudo weave start 10.0.0.3/24 oaDataNode3
    sudo weave start 10.0.0.4/24 oaDataNode4
    sudo weave expose 10.0.0.254/24

    关闭docker：
    sudo weave stop
	sudo docker stop oaDataNode2 oaDataNode1 oaDataNode3 oaDataNode4

重复执行至少4遍。

4）关闭RBAC

	gradm -D

这一步是必要的，因为它强制将学习模式的缓存写入磁盘。如果没有关闭gradm就生成policy，可能会导致学习记录的丢失。

此时查看learning.logs会看见里面有很多命令。

4） 生成policy文件

	gradm -F -L /etc/grsec/learning.logs -O /etc/grsec/policy

**NOTE：执行该命令前应该确保学习模式已经记录了足够多安全的操作，否则再次启动RBAC时将导致无法管理系统。**

# 结语

该文档通过使用Grsecurity和Shield对系统及ES进行了一系列的加固，虽然所做的所有措施只针对与ES有关的业务，但也能大体描绘出一个最小权限系统加固的轮廓。在其他不同的环境下依然能够按照这个思路对系统进行安全加固。

Grsecurity和Shield都是无比强大的加固武器，通过STIG检测的系统更是拥有更高的安全性。文档只是粗勒地描绘出一套加固思路，更多安全措施请参考Grsecurity和Shield的官方文档。

# Reference

1） Grsecurity官方文档：[https://en.wikibooks.org/wiki/Grsecurity/Appendix/Grsecurity_and_PaX_Configuration_Options#Address_Space_Layout_Randomization](http://)

2） gradm官方文档：[https://en.wikibooks.org/wiki/Grsecurity/The_Administration_Utility](http://)

3） Shield官方文档：[https://www.elastic.co/guide/en/shield/current/index.html](http://)

4） hardenedlinux社区：[http://hardenedlinux.org/](http://)
