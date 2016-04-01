---
layout: post
title:  "(A/T/KT) - Sanitized GNU/Linux: a new way of bug hunter in FLOSS Community"
summary: As long as there is bugs, there will be vulnerablities. As long as there are vulnerablities, there will be regular/stable/weaponized exploits. Bug hunting is one of most important issue that we've been fighting for decades in FLOSS community. Addr/thread sanitizers are very powerful weapons for bug hunters to build their own Fuzzing platform] or can be integrated into the regression testing. In either ways, FLOSS community can get benefit from it;-) 
categories: system-security
---

by citypw and an anonymous dude 


"As long as there is technology, there will be hackers. As long as there are hackers, there will be PHRACK magazine." --- The Circle of Lost Hackers on Phrack issue 64

As long as there are bugs, there will be vulnerablities. As long as there are vulnerablities, there will be regular/stable/weaponized exploits. Bug hunting is one of most important issue that we've been fighting for decades in FLOSS community. Addr/thread sanitizers are very powerful weapons for bug hunters to build their own [Fuzzing platform](http://nullcon.net/website/archives/ppt/goa-15/analyzing-chrome-crash-reports-at-scale-by-abhishek-arya.pdf) or can be integrated into the regression testing. In either ways, FLOSS community can get benefit from it;-)

We are inspired by the [work of Hanno Böck](https://fosdem.org/2016/schedule/event/csafecode/).

Most of Gentoo installation STEPS in this article are **COPY**   from [Gentoo Handbook](https://wiki.gentoo.org/wiki/Handbook:AMD64)[1]


Use links show as below to download gentoo LiveCD:   
Current-install-iso:   
[http://distfiles.gentoo.org/releases/amd64/autobuilds/current-install-amd64-minimal](http://distfiles.gentoo.org/releases/amd64/autobuilds/current-install-amd64-minimal/)   
After booing into liveCD, we can start to prepare for install system.


**Creating the partitions(GPT)：**
{% highlight html %}
parted -a optimal /dev/sda   
{% endhighlight %}
Use parted to configure *sda*
{% highlight html %}
(parted)mklabel gpt
{% endhighlight %}
Setting the GPT label
{% highlight html %}
(parted)unit mib
(parted)mkpart primary 1 3
(parted)name 1 grub
(parted)set 1 bios_grub on
{% endhighlight %}
Creating a partition start from 1MB and end at 3MB used by GRUB2BOOTLOADER
{% highlight html %}
(parted) mkpart primary 3 131
(parted) name 2 boot
{% endhighlight %}
Creating BOOT partition (128MB)
{% highlight html %}
(parted) mkpart primary 131 1024
(parted) name 3 swap
{% endhighlight %}
Creating swap partition
{% highlight html %}
(parted) mkpart primary 1024 -1
(parted) name 4 rootfs
{% endhighlight %}
Creating remaining disk as ROOTFS
{% highlight html %}
(parted) set 2 boot on
(parted) quit

{% endhighlight %}
**Creating the partitions(MBR):**
{% highlight html %}
livecd ~ # fdisk -t dos /dev/sda

Welcome to fdisk (util-linux 2.26.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-104857599, default 2048): 2048
Last sector, +sectors or +size{K,M,G,T,P} (2048-104857599, default 104857599):
+2M

Created a new partition 1 of type 'Linux' and of size 2 MiB.

Command (m for help): n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (2-4, default 2): 2
First sector (6144-104857599, default 6144): 
Last sector, +sectors or +size{K,M,G,T,P} (6144-104857599, default 104857599):
+128M

Created a new partition 2 of type 'Linux' and of size 128 MiB.

Command (m for help): p
Disk /dev/sda: 50 GiB, 53687091200 bytes, 104857600 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xcc7ca523

Device     Boot Start    End Sectors  Size Id Type
/dev/sda1        2048   6143    4096    2M 83 Linux
/dev/sda2        6144 268287  262144  128M 83 Linux

Command (m for help): a
Partition number (1,2, default 2): 2

The bootable flag on partition 2 is enabled now.

Command (m for help): p
Disk /dev/sda: 50 GiB, 53687091200 bytes, 104857600 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xcc7ca523

Device     Boot Start    End Sectors  Size Id Type
/dev/sda1        2048   6143    4096    2M 83 Linux
/dev/sda2  *     6144 268287  262144  128M 83 Linux

Command (m for help): n
Partition type
   p   primary (2 primary, 0 extended, 2 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (3,4, default 3): 
First sector (268288-104857599, default 268288):       
Last sector, +sectors or +size{K,M,G,T,P} (268288-104857599, default
104857599): +1024M

Created a new partition 3 of type 'Linux' and of size 1 GiB.

Command (m for help): t
Partition number (1-3, default 3):3
Partition type (type L to list all types):82

Command (m for help): n
Partition type
   p   primary (3 primary, 0 extended, 1 free)
   e   extended (container for logical partitions)
Select (default e): p

Changed type of partition 'Linux' to 'Linux swap / Solaris'.

Selected partition 4
First sector (2365440-104857599, default 2365440): 
Last sector, +sectors or +size{K,M,G,T,P} (2365440-104857599, default
104857599): 

Created a new partition 4 of type 'Linux' and of size 48.9 GiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
{% endhighlight %}


**Creating file systems：**
{% highlight html %}
mkfs.ext2 /dev/sda2
mkfs.ext4 /dev/sda4
{% endhighlight %}
Formatting **sda2** and **sda4** in ext2 and ext4
{% highlight html %}
mkswap /dev/sda3
{% endhighlight %}
Formatting **sda3** as swap partition
{% highlight html %}
swapon /dev/sda3
{% endhighlight %}

**Mounting:**   

After all Creating partition and formatting, now we can mount those partitions. Be aware of mounting partition we should creating a directories first.
{% highlight html %}
mount /dev/sda4 /mnt/gentoo/
mkdir /mnt/gentoo/boot
mount /dev/sda2 /mnt/gentoo/boot
{% endhighlight %}


**Installing Stage3:**

We cloud use **links** to download a stage tarball by surf to the gentoo mirror list. 
{% highlight html %}
links https://www.gentoo.org/downloads/mirrors
{% endhighlight %}

Chosing a proper(close by) mirror and enter the **releases/amd64/autobuilds/current-stage3-amd64/** directory.

Chose **stage3\-amd64\-\<releases\>.tar.bz2**/**stage3\-amd64\-\<releases\>.tar.bz2.DIGESTS**/**stage3\-amd64\-\<releases\>.tar.bz2.DIGESTS.asc** to download

> .CONTENTS file that contains a list of all files inside the stage tarball   
>
> .DIGESTS file that contains checksums of the stage file, in different algorithms   
>
> .DIGESTS.asc file that, like the .DIGESTS file, contains checksums of the stage file in different algorithms, but is also cryptographically signed to ensure it is provided by the Gentoo project
> 
> —— from Gentoo Handbook

**Validate Checksum：**

{% highlight html %}
cat stage3-amd64-<releases>.tar.bz2.DIGESTS
{% endhighlight %}

{% highlight html %}
openssl dgst -r -sha512 stage3-amd64-<releases>.tar.bz2
openssl dgst -r -whirlpool stage3-amd64-<release>.tar.bz2
{% endhighlight %}
Compare the output of these commands with the value registered in the .DIGESTS(.asc) files. The values need to match, otherwise the downloaded file might be corrupt (or the digests file is).   

{% highlight html %}
gpg --keyserver hkps.pool.sks-keyservers.net --recv-keys 0xBB572E0E2D182910
#From https://www.gentoo.org/downloads/signatures/
gpg --verify stage3-amd64-<release>.tar.bz2.DIGESTS.asc
{% endhighlight %}
Using gpg to make sure the checksums have not been tampered with.

**Unpacking the stage tarball**   

{% highlight html %}
tar xvjpf stage3-<release>.tar.bz2 --xattrs
{% endhighlight %}

**Configuring compile options**:

{% highlight html %}
vi /mnt/gentoo/etc/portage/make.conf
{% endhighlight %}

{% highlight html %}
CFLAGS="-march=native -O2 -pipe"
MAKEOPTS="-j8" #Depends on your Processor
{% endhighlight %}   

**CFLAGS and CXXFLAGS**   

> The CFLAGS and CXXFLAGS variables define the optimization flags for the GCC C and C++ compiler respectively. Although those are defined generally here, for maximum performance one would need to optimize these flags for each program separately. The reason for this is because every program is different. However, this is not manageable, hence the definition of these flags in the make.conf file.
> 
> In make.conf one should define the optimization flags that will make the system the most responsive generally. Don't place experimental settings in this variable; too much optimization can make programs behave bad (crash, or even worse, malfunction).
>
> —— from Gentoo Handbook   

**Chosing proper mirror**

{% highlight html %}
mirrorselect -i -o >> /mnt/gentoo/etc/portage/make.conf
mkdir /mnt/gentoo/etc/portage/repos.conf
cp /mnt/gentoo/usr/share/portage/config/repos.conf /mnt/gentoo/etc/portage/repos.conf/gentoo.conf
vi /mnt/gentoo/etc/portage/repos.conf/gentoo.conf
{% endhighlight %}   

cat gentoo.conf
{% highlight html %}
[gentoo]
location = /usr/portage
sync-type = rsync
sync-uri = rsync://rsync.gentoo.org/gentoo-portage
auto-sync = yes
{% endhighlight %}   

**Copy Nameserver info:**    

{% highlight html %}
cp -L /etc/resolv.conf /mnt/gentoo/etc/
{% endhighlight %}   

**Mounting the necessary filesystems**

{% highlight html %}
mount -t proc proc /mnt/gentoo/proc
mount --rbind /sys /mnt/gentoo/sys
mount --make-rslave /mnt/gentoo/sys
mount --rbind /dev /mnt/gentoo/dev
mount --make-rslave /mnt/gentoo/dev
{% endhighlight %}   

**Entering the chroot environment**
{% highlight html %}
chroot /mnt/gentoo /bin/bash
source /etc/profile
export PS1="(chroot) $PS1"
{% endhighlight %}   

**Update**
{% highlight html %}
emerge-webrsync
{% endhighlight %}   



{% highlight html %}
(chroot) livecd / # eselect profile list
Available profile symlink targets:
  [1]   default/linux/amd64/13.0 *
  [2]   default/linux/amd64/13.0/selinux
  [3]   default/linux/amd64/13.0/desktop
  [4]   default/linux/amd64/13.0/desktop/gnome
  [5]   default/linux/amd64/13.0/desktop/gnome/systemd
  [6]   default/linux/amd64/13.0/desktop/kde
  [7]   default/linux/amd64/13.0/desktop/kde/systemd
  [8]   default/linux/amd64/13.0/desktop/plasma
  [9]   default/linux/amd64/13.0/desktop/plasma/systemd
  [10]  default/linux/amd64/13.0/developer
  [11]  default/linux/amd64/13.0/no-multilib
  [12]  default/linux/amd64/13.0/systemd
  [13]  default/linux/amd64/13.0/x32
  [14]  hardened/linux/amd64
  [15]  hardened/linux/amd64/selinux
  [16]  hardened/linux/amd64/no-multilib
  [17]  hardened/linux/amd64/no-multilib/selinux
  [18]  hardened/linux/amd64/x32
  [19]  hardened/linux/musl/amd64
  [20]  hardened/linux/musl/amd64/x32
  [21]  default/linux/uclibc/amd64
  [22]  hardened/linux/uclibc/amd64
(chroot) livecd / # eselect profile set 12
{% endhighlight %}   


**Configuring Timezone**

{% highlight html %}
echo "Asia/Shanghai" > /etc/timezone
emerge --config sys-libs/timezone-data
{% endhighlight %}   

**Install gcc-5.3**

{% highlight html %}
emerge -av =gcc-5.3.0

The following keyword changes are necessary to proceed:
 (see "package.accept_keywords" in the portage(5) man page for more details)
# required by =gcc-5.3.0 (argument)
=sys-devel/gcc-5.3.0 ~amd64

Would you like to add these changes to your config files? [Yes/No]
{% endhighlight %}   

Update Configuration file

{% highlight html %}
dispatch-conf
{% endhighlight %}   

{% highlight html %}
--- /tmp/tmpjosdvwsz/0  2016-03-29 05:44:01.780036771 +0000
+++ /etc/portage/._cfg0000_package.accept_keywords      2016-03-29
05:43:15.840036346 +0000
@@ -1 +1,2 @@
-/dev/null
+# required by =gcc-5.3.0 (argument)
+=sys-devel/gcc-5.3.0 ~amd64

>> (1 of 1) -- /etc/portage/package.accept_keywords
>> q quit, h help, n next, e edit-new, z zap-new, u use-new
   m merge, t toggle-merge, l look-merge: 
{% endhighlight %}   

Run again
{% highlight html %}
emerge -av =gcc-5.3.0
{% endhighlight %}   


Change GCC default version

{% highlight html %}
(chroot) livecd / # gcc-config -l
 [1] x86_64-pc-linux-gnu-4.9.3 *
 [2] x86_64-pc-linux-gnu-5.3.0
(chroot) livecd / # gcc-config 2
{% endhighlight %}   

**Install Kernel**

Download kernel source
{% highlight html %}
emerge -av sys-kernel/gentoo-sources
{% endhighlight %}   
Checking Kernel Version
{% highlight html %}
(chroot) livecd linux # eselect kernel list
Available kernel symlink targets:
  [1]   linux-4.1.15-gentoo-r1 *
{% endhighlight %}   

Configuring Kernel Options

{% highlight html %}
cd /usr/src/linux
make menuconfig
{% endhighlight %}   

Enable KASan
{% highlight html %}
[*] KASan: runtime memory debugger 
    Instrumentation type (Inline instrumentation)  --->
        ( ) Outline instrumentation
        (X) Inline instrumentation #This options required GCC 5.0+

<*>   Intel ESB, ICH, PIIX3, PIIX4 PATA/SATA support 

[*]   Fusion MPT logging facility
       <*>   Fusion MPT ScsiHost drivers for SPI  
       <*>   Fusion MPT ScsiHost drivers for SAS  
       (128) Maximum number of scatter gather entries (16 - 128)
       <*>   Fusion MPT misc device (ioctl) driver  
       [*]   Fusion MPT logging facility  
{% endhighlight %}   

Compiling kernel

{% highlight html %}
make -j9 && make modules_install &&make install
{% endhighlight %}   

Generating initramfs
{% highlight html %}
emerge -av genkernel
genkernel initramfs
{% endhighlight %}   

Install Firmware
{% highlight html %}
emerge --ask sys-kernel/linux-firmware
{% endhighlight %}   

**Configure fstab**

{% highlight html %}
# NOTE: If your BOOT partition is ReiserFS, add the notail option to opts.
/dev/sda2               /boot           ext2            noauto,noatime  0 2
/dev/sda4               /               ext4            noatime         0 1
/dev/sda3               none            swap            sw              0 0
/dev/cdrom              /mnt/cdrom      auto            noauto,ro       0 0
/dev/fd0                /mnt/floppy     auto            noauto          0 0
{% endhighlight %}   


**Configure Network:**

{% highlight html %}
emerge net-misc/netifrc dhcp dhcpd

vim /etc/conf.d/net
###
config_eth0="dhcp"
###
{% endhighlight %}   

Automatically start networking at boot

{% highlight html %}
cd /etc/init.d
ln -s net.lo net.eth0
rc-update add net.eth0 default
{% endhighlight %}   

**Update SYSTEM**

{% highlight html %}
emerge -avuDN @world
{% endhighlight %}   

**Setting root password**

{% highlight html %}
passwd root
{% endhighlight %}   


**Install BOOTLOADER**

In this section, I going to use GRUB2 as my bootloader

{% highlight html %}
emerge -av grub
grub2-install /dev/sda
vim /etc/default/grub
######################add line show as below.
GRUB_CMDLINE_LINUX="rootfstype=ext4 init=/usr/lib/systemd/systemd"
######################
grub2-mkconfig -o /boot/grub/grub.cfg
{% endhighlight %}   

**Exiting chroot environment** 

{% highlight html %}
exit
{% endhighlight %}   

#reboot

After reboot, we could simply use dhcp automatically connect to Internet

{% highlight html %}
dhclient 
{% endhighlight %}   

Adding address sanitizer FLAGS into **/etc/portage/make.conf**
{% highlight html %}
CFLAGS="-march=native  -O2 -pipe -fsanitize=address"
CXXFLAGS="-march=native  -O2 -pipe -fsanitize=address"
{% endhighlight %}   

HINT:Address sanitizer and kernel address sanitizer are incompatible with thread sanitizer. If you want to use thread sanitizer, you can build another system to enble -fsanitize=thread separately.

Clear global variable to avoid **configure** error
{% highlight html %}
export LIBS=
export CFLAGS=
export CXXFLAGS=
{% endhighlight %}   

rebuild whole system (exclude gcc and glibc)

{% highlight html %}
emerge -e world --exclude=gcc --exclude=glibc
{% endhighlight %}   
