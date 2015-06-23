---
layout: post
title:  "Gentoo系统安装"
date:   2015-06-23 
categories: ciphergateway update

---

--[  目录

1. 准备工作
2. 组建RAID1
3. 安装Gentoo系统
   3.1 livecd下配置安装环境
   3.2 Gentoo安装及编译
   3.3 软件环境安装
4. Gentoo内核加固
   4.1 加固选项设置
   4.2 编译gentoo hardened内核

##--[ 1、准备工作

  用U盘启动引导安装gentoo系统，将gentoo系统的镜像文件livedvd-amd64-multilib-20140826.iso拷贝到已经格式化过的格式为fat32的U盘中；

U盘镜像下载链接：[http://mirrors.se.kernel.org/gentoo//releases/amd64/20140826/livedvd-amd64-multilib-20140826.iso](http://mirrors.se.kernel.org/gentoo//releases/amd64/20140826/livedvd-amd64-multilib-20140826.iso "下载链接")

##--[ 2、组建RAID1

1）Advanced Mode-->高级-->cpu设置，将Intel虚拟技术设为开启；

![](http://i.imgur.com/ljVLrxi.jpg)

2）Advanced Mode-->高级-->PCH存储设置，将SATA模式选择为RAID；

![](http://i.imgur.com/K6rDKtM.jpg)

3）保存设置并重启，按Ctrl+I(分别按不是同时按)，进入RAID配置界面，将两块3T机械硬盘组建为RAID1;

![](http://i.imgur.com/XjgyVIQ.jpg) 

##--[ 3、安装Gentoo系统

###----[ 3.1 livecd下配置安装环境

1）服务器上插入U盘，配置root账户，开启sshd服务:
　
![](http://i.imgur.com/LUPazv2.png)

2）使用gentoo livecd中自带的分区工具gdisk为系统分区，机械盘分两区，固态盘分五区(实际使用过程中使用cfdisk，gparted也可，我使用给gdisk的原因是这次在对raid盘进行分区时使用gparted造成了gpt与mbr分区表同时存在的问题，使用cfdisk将整个分区识别为了一个名为gpt的主分区)，以下是两块硬盘的分区结果：

![](http://i.imgur.com/LytBlwM.png)

![](http://i.imgur.com/pJ1S2K2.png)

3）利用secureCRT登陆系统shell(链接前先要确认主机的ip，并确保两个主机处于同一网段，如果只有一台电脑，也可以在该主机上直接操作)：

![](http://i.imgur.com/y5eo8Qh.png)

4）制作文件系统(格式化分区，默认格式成了ext4分区，如有需要，也可以格式成其他格式的分区)：

        mkfs.ext4 /dev/sda3
        mkfs.ext4 /dev/sda4
        mkfs.ext4 /dev/sda5

![](http://i.imgur.com/6x8T18i.jpg)
设置swap分区

        mkswap /dev/sda2
![](http://i.imgur.com/FX6HKRi.png)
格式化其他分区
        mkfs.fat /dev/sda1
        mkfs.ext4 /dev/md126p1
        mkfs.ext4 /dev/md126p2

![](http://i.imgur.com/wVi2cPS.jpg)

5）挂载分区（将两块硬盘以目录树的形式挂载在/mnt）(分区用途/dev/sda1 efi分区，才用mbr引导的可忽略此份区，/dev/sda3 tmp分区，/den/sda4 根分区，/dev/sda5 home分区)：

     mount /dev/sda4/mnt/gentoo
     mkdir -p/mnt/gentoo/boot/efi
     mount /dev/sda1/mnt/gentoo/boot/efi
     mount -p/mnt/gentoo/home
     mount /dev/sda5/mnt/gentoo/home
     mkdir /mnt/gentoo/tmp
     mount /dev/sda3/mnt/gentoo/tmp

6）设置系统时间(注意 日期格式为MMddhhmmyyyy)：


    date 061915262015

7）下载系统编译包portage-latest.tar.bz2和stage3-amd64-20150618.tar.bz2到/mnt/gentoo目录下(注意，镜像源我们选择的是阿里云的镜像，使用者可自行更换，下载stage3包时要选择日期最近的)：

   `wget http://mirrors.aliyun.com/gentoo/releases/snapshots/current/portage-latest.tar.bz2`

![](http://i.imgur.com/1YOrwhV.png)

   ` wget http://mirrors.ustc.edu.cn/gentoo/releases/amd64/autobuilds/current-stage3-amd64/stage3-amd64-20150618.tar.bz`

![](http://i.imgur.com/gNpBpCX.png)

8）解压stage3和portage两个文件：

    tar -jxvf stage3-amd64-20150618.tar.bz2

注: stage3-amd64-20131010.tar.bz2解压的文件是Gentoo的目录结构，所以要解压到临时的系统目录下,即/mnt/gentoo，方便后面进行chroot

    tar -jxvf portage-latest.tar.bz2 -C /mnt/gentoo/usr

注: portage-latest.tar.bz2解压的文件为系统软件目录结构,需要解压到/mnt/gentoo/usr目录下

###----[ 3.2 Gentoo安装及编译

1）切换系统到/dev/sda3根分区上并更新系统环境变量：

    mount -t proc none /mnt/gentoo/proc
    mount -o bind /dev /mnt/gentoo/dev
    mount -t sysfs sys /mnt/gentoo/sys
    chroot /mnt/gentoo /bin/bash
    env-update
    >> Regenerating /etc/ld.so.cache...
    source /etc/profile
    export PS1="(chroot) $PS1"

2）设置时区：

    cp /usr/share/Asia/Shanghai /etc/localtime

3）设置主机名：

    sed -i -e's/hostname.*/hostname="shenhua"/' /etc/conf.d/hostname

4）修改镜像源 并且设置编译参数为8核编译(注意，编译参数在设置为核心数加1时理论上也可获得最优效果)：

![](http://i.imgur.com/9452UEB.png)

5）设置DNS(通常情况下设置默认dns即可，但此步缺失会造成dns无法解析，原因为chroot过程中dns信息未被转移)：

    echo "nameserver 10.1.2.1" >> /etc/resolv.conf

6）安装内核源码：

    emerge gentoo-sources

7）安装自动编译内核工具genkernel：

    emerge genkernel

8）复制安装光盘的配置文件到genkernel搜索配置文件的默认位置(注意32位用户目录为/usr/share/arch/x8/,此配置文件livecd所使用的配置文件)：

    zcat /proc/config.gz > /usr/share/arch/x86_64/kernel-config

9）编译内核：

     genkernel all

10）修改fstab：

![](http://i.imgur.com/Olr6tbb.png)

11）配置网络：

　（1）生成软连接(注意，eno1为网络接口名，不同电脑可能不同，请在chroot前用ifconfig命令确认)：
    
       ln -s /etc/init.d/net.lo /etc/init.d/net.eno1
	
　（2）创建网络配置文件：

     vi /etc/conf.d/net

 　在空文件中写入(注意，如果是静态ip请采用静态配置)：
     # DHCP
     config_eno1=( "dhcp" )

12）设置网卡开机自启动：

    rc-update add net.eno1 default

13）设置GRUB引导：

   (1) 修改配置文件：在etc/portage/make.conf中添加GRUB_PLATFORMS="efi-64"(mbr引导可忽略此步骤)

   (2) 安装GRUB  

    emerge --ask sys-boot/grub:2
    emerge -av sys-boot/os-prober
    grub2-install --target=x86_64-efi
    grub2-mkconfig -o /boot/grub2/grub.cfg

   (3) 重起电脑，确认grub引导是否安装成功，如果成功再进行一下步骤

###----[ 3.3 软件环境安装

1）安装xrog：
 
     在 /etc/portage/make.conf 中添加 INPUT_DEVICES="evdev synaptics" 
     emerge --ask --verbose --pretend x11-base/xorg-drivers
     emerge --ask x11-base/xorg-server
     cat /etc/portage/package.use/._cfg0000_iputils > /etc/portage/package.use/iputils
     emerge twm
     emerge xclock
     emerge xterm

　　备注：twm xclock xterm原本是xrog软件包的一部分，在版本更新后从xrog中分离出来 需要单独安装

2）编译kde桌面环境 
　　备注：编译其他环境 请参照wiki.gentoo.org openrc启动的kde对应的编号是6 其他桌面环境请自行更换

    eselect profile list
    eselect profile set 6
    emerge --ask kde-apps/kdebase-meta
    emerge --ask kde-base/kde4-l10n
    在/etc/portage/make.conf中添加 LINGUAS="de"

##--[  4、Gentoo内核加固

###----[ 4.1　加固选项设置

　　编辑/etc/ssh/ssh_config文件，在里面加入：

		# 1, known_hosts stores server's signature, so hash the host name:	
		HashKnownHosts yes
		#2, SSH protocl version 1 is not secure:
		Protocol 2
		#3, If you don't use X11 forwarding, plz disable it"
		X11Forwarding no
		#4, Disable rhosts:
		IgnoreRhosts yes
		#5, Not allow empty password:
		PermitEmptyPasswords no
		#6, Maxisum tries:
		MaxAuthTries 5
		#7, Now allow root login:
		PermitRootLogin no
		#(Optional)
		#1, disable password auth, enable pubkey auth:
		PubkeyAuthentication yes
		PasswordAuthentication no
		#2，Allow or deny users/groups
		#AllowGroups, AllowUsers, DenyUsers, DenyGroups

　　根据自己的实际要求在改写最后一行 改为自己的用户组合用户名

###----[ 4.2　编译gentoo hardened内核

1）下载加固内核源码

	emerge --ask sys-kernel/hardened-sources

2）配置加固内核 参照：

　　[https://wiki.gentoo.org/wiki/Hardened/PaX_Quickstart#Building_a_PaX_Kernel](https://wiki.gentoo.org/wiki/Hardened/PaX_Quickstart#Building_a_PaX_Kernel)

　　[https://wiki.gentoo.org/wiki/Hardened/Grsecurity2_Quickstart](https://wiki.gentoo.org/wiki/Hardened/Grsecurity2_Quickstart)

3）编译 

	$ make

4）安装
	
	# make install

5）选择内核

	# eselect kernel list

6）重启使用新内核即可
	 