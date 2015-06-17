---
layout: post
title:  "Debian GNU/Linux security checklist and hardening"
date:   2015-06-09 22:48:45
categories: jekyll update
---

project STIG-4-Debian will be soonn....

[Debian GNU/Linux security checklist and hardening](https://raw.githubusercontent.com/citypw/DNFWAH/master/5/d5_0x02_DNFWAH_debian_gnu-linux_security_chklist_hardening.txt)

--[ CONTENTS

0. About this doc

1. Security updates

2. Vulnerability Assessment

   2.1 GCC mitigation

   2.2 0ld sch00l \*nix file auditing

   2.3 GNU/Linux's auditd

   2.4 T00ls

3. Kernel security

   3.1 Apparmor

   3.2 SELinux

   3.3 Mempo kernel

       3.3.1 PaX\/Grsecurity

4. SSL/TLS Checklist

   4.1 Ciphersuites in Apache2/Nginx

   4.2 OpenSSH

       4.2.1 OpenSSH in post-prism era

5. Web security

   5.1 Web server( Apache/Nginx?)

   5.2 WAF( Web Application Firewall)

6. Security standard

   6.1 STIGs for Debian

7. Reference


##--[ 0. About this documentation

GNU/Linux already become one of most important fundamental element in
\*modern\* IT platform. Almost every important applications heavily rely
on the core component of GNU system: GCC, Glibc and linux
kernel. GNU/Linux is totally free/libre and open source software(
FLOSS). Many people thinks free/libre and open source software is
secure because its open to many eyes. Yes, that's true. According to
[Coverity's report](http://developers.slashdot.org/story/14/04/16/2021227/code-quality-open-source-vs-proprietary).

The source code quality of FLOSS project are better than closed
software systems. But the FLOSS is not unbreakable. This documentation
is going to discuss something we should know about GNU/Linux security
operations. These examples in this doc has been tested only on Debian
GNU/Linux 7.5.


##--[ 1. Security update

Follow the minimal installation principle: Debian is providing mini
installation iso.

To check which packages need security updates:

\#sudo apt-get upgrade -s | grep -i security


##--[ 2. Vulnerability Assessment

Know your GNU/Linux system as your \*enemy\* does. Your enmey might hide
in the shadow and watch and learn the ways you've been using the
system. As a defender, some philosophical ideas( thanks to Bruce
Schneier) should be [kept in mind](https://www.schneier.com/essays/archives/2000/04/the_process_of_secur.html)

-----------------------------------------------------------------------
Security is NOT:

* Security is NOT installing a firewall

* Security is NOT a Product or Service

* Security is Not a Product; It's a Process


Security is:

* Security is a Process, Methodology, Costs, Policies and People

* Security is only as good as your "weakest link"

* Security is 24x7x365 ... constantly ongoing .. never ending

-----------------------------------------------------------------------

A security system is only as strong as its weakest
link. Defense-in-depth seems the only option we have. You should be
the best professional paranoia and also need a proper threat
model. "Who's gonna attack your system" would be daily bread for your
mind;-)


###----[ 2.1 GCC mitigation

setuid binaries are highly risks if the program had the bug that can
be exploitable.  The setuid binaries should be protected under GCC's
mitigation.  We only examine [4 mitigation options here](http://phrack.org/archives/issues/67/13.txt) ( some GCC mitigation
description from one Phrack paper, thanks pi3..dude, did I owe you a
beer?)

\*) NX

This feature can prevent shellcode execution on the stack.  This
mechanism can be implemented by hardware or software emulation.

In GCC's options, [NX](http://en.wikipedia.org/wiki/NX_bit) is enable by default. If you want to turn it off,
use "-z execstack".

Check method: 
<pre>
shawn@shawn-fortress ~ $ readelf -l a.out | grep GNU_STACK 
  GNU_STACK      0x000000 0x0000000000000000 0x0000000000000000 0x000000 0x000000 RW  0x8
</pre>

\*) Stack canaries (canaries of the death)

This is a compiler mechanism, in contrast to previously kernel-based
described techniques. When a function is called, the code inserted by
the compiler in its prologue stores a special value (the so-called
cookie) on the stack before the metadata. This value is a kind of
defender of sensitive data. During the epilogue the stack value is
compared with the original one and if they are not the same then a
memory corruption must have occurred. The program is then killed and
this situation is reported in the system logs. Details about technical
implementation and little arm race between protection and bypassing
protection in this area will be explained further.

GCC options:
-fno-stack-protector,  do not add any canary onto any functions

-fstack-protector, only add the canary onto a few functions in compile
 time

-fstack-protector-all , add the canary onto all functions, be cautions
 about this one. It'd be triggered the heavily performance hit.

-fstack-protector-strong, add the canary onto those functions, which
 the stack buffers would be used. This is a smart one. But its only
 supported by GCC 4.9.x. Kees Cook shared a very [good writing here](http://www.outflux.net/blog/archives/2014/01/27/fstack-protector-strong/).


Check the symbols in an elf file: 
<pre>
#readelf -s ./a.out | grep stack_chk_fail 
     3: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __stack_chk_fail@GLIBC_2.4 (3) 
    52: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __stack_chk_fail@@GLIBC_2 
</pre>

\*) RELRO 

RELocation Read-Only. 

Turn it on: -z norelro 
Turn it off: -z now 

Check elf header to find partial-RELRO: 
<pre>
shawn@shawn-fortress ~ $ readelf -l a.out | grep GNU_RELRO 
  GNU_RELRO      0x0000000000000e28 0x0000000000600e28 0x0000000000600e28 

Check elf's dynamic sections to find fully-RELRO: 
shawn@shawn-fortress ~ $ readelf -d a.out | grep BIND_NOW 
 0x0000000000000018 (BIND_NOW)
</pre>


\*) PIE

[PIE](http://en.wikipedia.org/wiki/Position-independent_code) enforces every process's code segment is mmap()'d, it begins at a
different base address at each execution of the application.

Note: mmap()' is always used no matter what the type of the executable
is (PIE vs. non-PIE). For non-PIE binaries the kernel uses an internal
flag equivalent to MMAP_FIXED when mapping program headers.

GCC option: -pie, it only work for sec mitigation when kernel enables
ASLR.
<pre>
no PIE: 
shawn@shawn-fortress ~ $ readelf -h a.out | grep "Type:[[:space:]]*EXEC" 
  Type:                              EXEC (Executable file) 

PIE: 
shawn@shawn-fortress ~ $ readelf -h a.out | grep "Type:[[:space:]]*DYN" 
  Type:                              DYN (Shared object file)
</pre>

These exploit mitigations provided by GCC will definitely increase the
cost of attackers. We all did believed so...until shit happened( as
always?). Hector Marco [released a method](http://cybersecurity.upv.es/attacks/offset2lib/offset2lib.html) that can bypass NX/ASLR/PIE/CANARY mitigations locally/remotely
easily. After these years of debating and bragging about how secure of
GNU/Linux is/was and we finally ended up in\*One mem infoleak can rule
the fuc\*ing GNU/Linux\*!!! Damn, PaX/Grsecurity will be our last hope
again, like a decade ago...........


###----[ 2.2 0ld sch00l \*nix file auditing

There are a bunch of files that could be exploited by attackers in the
specific scene. Fortunately, FOSS( Free & Open Source) community is
providing a lot of methods for the security audit work. They should be
a defender's daily bread, which being part of defense-in-depth model.

WildCards is a powerful feature in UNIX-like platform, but it can be
[exploited](http://www.defensecode.com/public/DefenseCode_Unix_WildCards_Gone_Wild.txt) by attackers:

<pre>find / -path /proc -prune  -name "-*"</pre>


World-writable file audit:

<pre>find / -path /proc -prune -o -perm -2 ! -type l -ls</pre>


World-readable file audit, correct permission: chmod 640 /var/log/:

<pre>find /var/log -perm -o=r ! -type l</pre>


Check if files were orphange:

<pre>find / -path /proc -prune -o -nouser -o -nogroup</pre>


List avaiable users, be cautions about who's the "user":
<pre>egrep -v '.*:\*|:\!' /etc/shadow | awk -F: '{print $1}'</pre>


Check which files belong to whom. Then delete the user correctly:

<pre>userdel -r account</pre>

<pre>find / -path /proc -prune -o -user account -ls</pre>


List which users are unavailable:

<pre>grep -v ':x:' /etc/passwd</pre>


List expired passwords:

<pre>cat /etc/shadow | cut -d: -f 1,2 | grep '!'</pre>


The correct permission should be 644 at least. 600 would be better:

<pre>ls -l /boot</pre>


Files with suid or sgid flags:

<pre>find / -xdev -user root \( -perm -4000 -o -perm -2000 \)</pre>


Check if some stupid mistakes has been made( Thanks to Tim Brown):

<pre>objdump -x $i | grep -i path</pre>


Note: The main stream GNU/Linux distro( Debian, Gentoo, OpenSUSE,
CentOS) won't have big chance to do stupid things, but it's worth to
look at GNU/Linux platform with 3-rd party applications. Some
commercial applications may do something stuipid like this one:

http://lists.openwall.net/bugtraq/2014/06/04/5


----[ 2.3 GNU/Linux's auditd

One particular scene is that some m41wares( or human attackers) might
be interested in change some file's metadata for some \*interesting\*
reasons and then change it back to the original. Let's show time:

Install auditd and make sure its on boot startup:
<pre>#apt-get install auditd
#update-rc.d auditd enable</pre>


Config file:

<pre>/etc/audit/auditd.conf</pre>


Store log file:

<pre>log_file = /var/log/audit/audit.log</pre>


Add one policy to /etc/audit/audit.rules:

<pre>-w /home/shawn/change-test -p wa -k  identify</pre>


Use this test program to change the permission：

#include <stdio\.h>
#include <sys/stat\.h>
#include <stdlib\.h>

int main(int argc, char *argv[])
{
	struct stat sb;

	if( stat(argv[1], &sb) == -1){
	    perror("stat");
		exit(EXIT_FAILURE);
		}

		if( chmod(argv[1], sb.st_mode) == -1)
		{
			perror("stat");
				exit(EXIT_FAILURE);
				}
				return ;
}

<pre>
shawn@shawn-fortress ~ $ gcc change.c
shawn@shawn-fortress ~ $ touch change-test
shawn@shawn-fortress ~ $ ./a.out change-test
</pre>

The date of  "Modify" and "Change" should be different:
shawn@shawn-fortress ~ $ stat change-test

Check \*who\* did it:

shawn@shawn-fortress ~ $ ausearch -i -k identify


###----[ 2.4 T00ls

NMAP/OpenVAS/lynis/rkhunter/chkrootkit/metasploit/volatality/etc


--[ 3. Kernel security

* Anti-DoS related:

** SYN cookies is a syn flood attack protection, the default is enable( 1)：
<pre>
net.ipv4.tcp_syncookies = 1
/proc/sys/net/ipv4/tcp_syncookies

(optional)，if your kernel support SYNPROXY：
iptables -t raw -A PREROUTING -i eth0 -p tcp --dport 80 --syn -j NOTRACK
iptables -A INPUT -i eth0 -p tcp --dport 80 -m state UNTRACKED,INVALID \
	 -j SYNPROXY --sack-perm --timestamp --mss 1480 --wscale 7 --ecn

echo 0 > /proc/sys/net/netfilter/nf_conntrack_tcp_loose

Note: SYNPROXY has been added into vanilla kernel in 3.13.
</pre>


** TCP FIN-WAIT-2 status lifetime, it'd be an DoS attack risk if
the value is too big. It'd be cause remote machine doesn't have enough
time to close the connection if the value is too small. Default is 60(
seconds). 15 is better, you think? [Further reading](http://benohead.com/tcp-about-fin_wait_2-time_wait-and-close_wait/):

<pre>
net.ipv4.tcp_fin_timeout = 15
/proc/sys/net/ipv4/tcp_fin_timeout
</pre>


** SYN queue length, the bigger value can handle more connections, the
  default is 1024:

<pre>
net.ipv4.tcp_max_syn_backlog = 8192
/proc/sys/net/ipv4/tcp_max_syn_backlog
</pre>


** Device queue length, this value should be bigger than syn queue?
   The default is 1000

<pre>
net.core.netdev_max_backlog = 16384
/proc/sys/net/core/netdev_max_backlog
</pre>


** listen()'s backlog, the default is 128:

<pre>
net.core.somaxconn = 4096
/proc/sys/net/core/somaxconn
</pre>


** TIME_WAIT status TCP connections, the system will empty the
   connection if the number is exceed the value, 

<pre>
net.ipv4.tcp_max_tw_buckets = 65535
/proc/sys/net/ipv4/tcp_max_tw_buckets
</pre>


** TIME-WAIT status can be reuse, the default is disable( 0):

<pre>
net.ipv4.tcp_tw_reuse = 1
/proc/sys/net/ipv4/tcp_tw_reuse
</pre>


** fast recycle of TIME-WAIT status connection, the default is disable( 0): 

<pre>
net.ipv4.tcp_tw_recycle = 1
/proc/sys/net/ipv4/tcp_tw_recycle
</pre>


** TCP KEEPALIVE probe frequency,the default is 7,200 seconds:

<pre>
net.ipv4.tcp_keepalive_time = 300
/proc/sys/net/ipv4/tcp_keepalive_time
</pre>


** TCP KEEPALIVE probe packets, the default is 9:

<pre>
net.ipv4.tcp_keepalive_probes = 3
/proc/sys/net/ipv4/tcp_keepalive_probes
</pre>


** how many times of SYN and SYN+ACK can be re-transimit, the default is 5:

<pre>
net.ipv4.tcp_syn_retries = 3
/proc/sys/net/ipv4/tcp_syn_retries

net.ipv4.tcp_synack_retries = 3 
/proc/sys/net/ipv4/tcp_synack_retries
</pre>


** the bigger value of TCP ORPHAN would prevent simple DoS attack,
   each ORPHAN cost 64KB memory, so 65535 is about 4GB:

<pre>
net.ipv4.tcp_max_orphans = 65536
/proc/sys/net/ipv4/tcp_max_orphans
</pre>


** How many pages( 4KB each page in x86) can be used in TCP connection:

<pre>
net.ipv4.tcp_mem = 131072 196608 262144
/proc/sys/net/ipv4/tcp_mem
</pre>

Be careful about this one, it'd be triggered OOM if the TCP connection
consume all pages.


** The maximum send and receive window, you can set 64MB for a 10G NIC:

<pre>
net.core.rmem_max = 67108864
/proc/sys/net/core/rmem_max

net.core.wmem_max = 67108864
/proc/sys/net/core/wmem_max
</pre>


** Each TCP connection's read buffer( X bytes):

<pre>
net.ipv4.tcp_rmem = 4096 8192 16777216( 4096 87380 33554432)
/proc/sys/net/ipv4/tcp_rmem

net.ipv4.tcp_wmem = 4096 8192 16777216( 4096 65536 33554432)
/proc/sys/net/ipv4/tcp_wmem
</pre>

If default paging 8kb * 2 = 16kb/connection, 4GB memory can be used for:
(4 * 1024 * 1024) / 16 = 262144

Oracle DB server's [best practice](http://www.dba-oracle.com/t_linux_networking_kernel_parameters.htm)


* Networking

Ref:
https://www.suse.com/documentation/sles11/singlehtml/book_hardening/book_hardening.html

** Source Routing is used to specify a path or route through the
network from source to destination. This feature can be used by
network people for diagnosing problems. However, if an intruder was
able to send a source routed packet into the network, then he could
intercept the replies and your server might not know that it's not
communicating with a trusted server.

<pre>
net.ipv4.conf.all.accept_source_route = 0
/proc/sys/net/ipv4/conf/all/accept_source_route
</pre>


** ICMP redirects are used by routers to tell the server that there is
   a better path to other networks than the one chosen by the
   server. However, an intruder could potentially use ICMP redirect
   packets to alter the hosts's routing table by causing traffic to
   use a path you didn't intend.

<pre>
net.ipv4.conf.all.accept_redirects = 0
/proc/sys/net/ipv4/conf/all/accept_redirects
</pre>


** Turn it off if this is not a router：

<pre>
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0

/proc/sys/net/ipv4/conf/all/send_redirects
/proc/sys/net/ipv4/conf/default/send_redirects
</pre>


** IP spoofing protection, the default is disabled( 0)：
<pre>
net.ipv4.conf.all.rp_filter = 1
/proc/sys/net/ipv4/conf/all/rp_filter
</pre>


** If you want to ignore all ICMP package, you can enable it. The
   default is disabled( 0):

<pre>
net.ipv4.icmp_echo_ignore_all = 1
/proc/sys/net/ipv4/icmp_echo_ignore_all
</pre>


** Ignore ICMP broadcast, the default is enabled( 1):

<pre>
net.ipv4.icmp_echo_ignore_broadcasts = 1
/proc/sys/net/ipv4/icmp_echo_ignore_broadcasts
</pre>


** To alert you about bad message, the default is disabled( 1):

<pre>
net.ipv4.icmp_ignore_bogus_error_responses = 1
/proc/sys/net/ipv4/icmp_ignore_bogus_error_responses
</pre>


** To turn on logging for Spoofed Packets, Source Routed Packets, and
   Redirect Packets, the default is disabled( 0):

<pre>
/proc/sys/net/ipv4/conf/all/log_martians
net.ipv4.conf.all.log_martians = 1
</pre>


Exploit mitigation
--------------------------------------------------------------
** Randomize the memory area. 0: disable ASLR. 1: Randomize the stack,
   VDSO page, shared memory regions. 2: (1) + also randomize the data
   segment.

<pre>
kernel.randomize_va_space=2
/proc/sys/kernel/randomize_va_space
</pre>


** Not allow the user to read kernel address symbol tables:
<pre>
kernel.kptr_restrict=1
/proc/sys/kernel/kptr_restrict
</pre>


** the minimal memory map address, 65536 is good at least:
<pre>
vm.mmap_min_addr=65536
/proc/sys/vm/mmap_min_addr
</pre>

** Now allow the debugger trace the process by ptrace. The current
   Debian stable kernel doesn't have this feature. 0: All process can
   be debugged. 1: Only one father process can be debugged. 2: Only
   *root* can do debug( with CAP_SYS_PTRACE) 3: no process can be
   debugged
<pre>
kernel.yama.ptrace_scope = 2
/proc/sys/kernel/yama/ptrace_scope
</pre>


###----[ 3.1 Apparmor

Why Apparmor? It's easy to deploy. More importantly, it's easy to
audit the polices. Everyone can write your own MAC/RBAC
policy. Debian/OpenSuSE shipped with Apparmor by default.

Install Apparmor and MAC polices from community:
\#apt-get install -y apparmor-profiles apparmor

Check the status:
\#aa-status


###----[ 3.2 SELinux

S0rry. I barely use SELinux for reasons. The 1st one is I don't trust
NSA, even the source code is GPL'ed. NSA is professional about
*implant* backdoors, which would be very hard to audit. People has
been discussing it for years:

[NSA SELinux](https://www.schneier.com/blog/archives/2008/04/nsas_linux.html)

[NSA has inserted its code into Android](http://www.zerohedge.com/news/2013-07-09/nsa-has-inserted-its-code-android-os-bugging-three-quarters-all-smartphones)

[NSA linux/android kernel](http://www.eteknix.com/nsa-has-code-running-in-the-linux-kernel-and-android/)


Sebastian Krahmer found a [exploitable bug](https://github.com/stealth/troubleshooter) from SELinux recently. It looks like a backdoor more than a "vulnerablity", isn't it?;-)
Another important reason people don't like SELinux because it's hard
to use and cause other application troubles from time to time. [Stop
disabling SELinux](http://stopdisablingselinux.com/) movement won't work in the near future:

S0rry, Mr.Walsh. It's nothing personal:-)


###----[ 3.3 Mempo kernel

""⌘ Mempo project aims to provide most secure and yet comfortable
out-of-the-box Desktop and Server computer, for professionals,
business, journalists, and every-day users avoiding PRISM-like
spying. ⌘"


[Mempo](https://wiki.debian.org/Mempo) is a FLOSS project for protect user's digital freedom. Let the
massive surveillance cry;-)

The Debian Mempo repo is not working for the internet user now and
it'll be back soon. So I'll write how to use PaX/Grsecurity from
offical Mempo repository on Debian.


###----[ 3.3.1 PaX/Grsecurity

PaX/Grsecurity is the cutting-edge kernel protection in past 14
years. But they don't have the credit what they supposed to
have. Almost every main stream OS kernel security mechanism has
influenced by PaX/Grsecurity in past decade. Lionel tells a [little story](http://www.openwall.com/lists/oss-security/2014/12/06/14) about PaX/Grsecurity better than myself:

PaX/Grsecurity treat the kernel security as a whole. They've been
inventing many innovations( SEGMEXEC, PAGEEXEC, MPROTECT, UDEREF,
RANDSTRUCT*, etc), while hardening the kernel in source code level(
make important *struct* read-only, etc). PaX/Grsecurity is one of most
respected 0ld sch00l hacker community. The main contributor( Spender)
was even been through a [very badly economic situation](http://developers.slashdot.org/story/04/05/31/1949241/end-of-development-for-grsecurity-announced) back in 2004 

Thanks to the G0d of techn0logy, PaX/Grsecurity is still alive.... I
personally agree with some ideas:

-----------------------------------------------------------------------
"The "better than none" point of view is actually a nice way to false
sense of security for those who don't know better. We got
better-than-none apparmor, selinux, tomoyo, some poorly maintained and
crippled ports of grsec features or alikes, namespaces and containers,
rootkit-friendly LSM, the dumb and useless kernel version of SSP,
etc. What's the sum of all this shit for end users? False sense of
security..."

"without Grsecurity/PaX, linux security is like monkey can never
perform a regular masturbation cu'z lacking of giant pennis;-)"
-----------------------------------------------------------------------

Too many better-than-none product or solutions, which only makes you
feel safe, maybe for a while. Feel safe is not equal to
secure. Fuc*ing cargo cult shitty security only makes things worse.

I'll show you how to install PaX/Grsecurity manually( still waiting
Mempo back online):

** [Download kernel](https://www.kernel.org/pub/linux/kernel/v3.x/) ( Pick one)：
https://www.kernel.org/pub/linux/kernel/v3.x/

** Download [PaX/Grsecurity patch](https://grsecurity.net/download.php) ( you can download the latest version
   from )：

** Decompress the kernel and patch the kernel with grsecurity:
<pre>
xz -d linux-*.tar.xz
tar xvf linux-*.tar
cd linux-*/
patch -p1 < ../grsecurity-*.patch
</pre>

** Do "make menuconfig" to customize your kernel, or you can use [my test config](https://raw.githubusercontent.com/citypw/citypw-SCFE/master/security/apparmor_test/debian-7.4-linux-3.14.1-grsec.config)


** Compile
<pre>
make -j7 deb-pkg
</pre>

** Install the new kernel
<pre>
dpkg -i ../*.deb
</pre>


##--[ 4. Crypto

“Encryption works. Properly implemented strong crypto systems are one
of the few things that you can rely on. Unfortunately, endpoint
security is so terrifically weak that NSA can frequently find ways
around it.”   --- Edward Snowden

Damn, we should treat the crypto engineering very carefully. Because
it may be the last outter-heaven we have;-)

----[ 4.1 SSL/TLS Checklist

[Very good writing](http://www.exploresecurity.com/wp-content/uploads/custom/SSL_manual_cheatsheet.html)

SSL/TLS has been through BEAST/CRIME/LUCKY-13/HEARTBLEED/POODLE in
past few years. and it's already become one of hottest topic in cyber
security. There are a set of vulnerable protocols and ciphersuites are
worth to do audit. There are a few open source tools would make your
audit work easier. Try this one:

<pre>
#apt-get install sslscan


SSLv2 should be disabled:
openssl s_client -ssl2 -connect www.google.com:443

OpenSSL 1.0 no longer support SSLv2. So you can use GnuTLS do the
check:

gnutls-cli -d 5 -p 443 --priority "NORMAL:-VERS-TLS1.2:-VERS-TLS1.1:-VERS-TLS1.0:-VERS-SSL3.0" www.google.com
</pre>

FREAK:
<pre>
openssl s_client -cipher EXPORT -connect www.google.com:443
</pre>

If succeed, it's risk to FREAK.


----[ 4.2 OpenSSH

Config file：/etc/ssh/ssh_config

<pre>
1, known_hosts stores server's signature, so hash the host name:

HashKnownHosts yes

2, SSH protocl version 1 is not secure:

Protocol 2

3, If you don't use X11 forwarding, plz disable it"

X11Forwarding no

4, Disable rhosts:

IgnoreRhosts yes

5, Not allow empty password:

PermitEmptyPasswords no

6, Maxisum tries:

MaxAuthTries 5

7, Now allow root login:

PermitRootLogin no


(Optional)
1, disable password auth, enable pubkey auth:

PubkeyAuthentication yes

PasswordAuthentication no


2，Allow or deny users/groups

AllowGroups, AllowUsers, DenyUsers, DenyGroups
</pre>


###------[ 4.2.1 OpenSSH in post-prism era

Well, plz [read this](https://stribika.github.io/2015/01/04/secure-secure-shell.html)


###----[ 4.2 Ciphersuites in Apache2/Nginx

The explanation is [here](https://hynek.me/articles/hardening-your-web-servers-ssl-ciphers/)

Apache:
SSLProtocol ALL -SSLv2 -SSLv3
SSLHonorCipherOrder On
SSLCipherSuite ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:ECDH+3DES:DH+3DES:RSA+AESGCM:RSA+AES:RSA+3DES:!aNULL:!MD5:!DSS

Nginx:
ssl_prefer_server_ciphers On;
ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
ssl_ciphers ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:ECDH+3DES:DH+3DES:RSA+AESGCM:RSA+AES:RSA+3DES:!aNULL:!MD5:!DSS


--[ 5. Web security

You're not reading this article for learning pentest, are you? Let's
just consider the fence of defender's;-)

OWASP code review:
https://www.owasp.org/images/2/2e/OWASP_Code_Review_Guide-V1_1.pdf
https://www.owasp.org/index.php/OWASP_Code_review_V2_Project

OWASP testing guide:
https://www.owasp.org/images/5/52/OWASP_Testing_Guide_v4.pdf
https://www.owasp.org/index.php/OWASP_Guide_Project


----[ 5.1 Web server( Apache/Nginx ?)

ServerRoot
<pre>
Web server's root path. Default is "/etc/httpd". It's import to keep
track of its permission. Recommend: Do not allow none-root user has
the permission to modify it.

chown root:root /etc/httpd
chmod 754 /etc/httpd
</pre>

Timeout
<pre>
Lifetime per session. Default is 60 seconds. Set the lower value for
mitigating DoS attack. Recommend: 15 <= X <= 30
</pre>


KeepAlive
<pre>
Persistent session. Default is “Off”. Recommend: On
</pre>

User
<pre>
Decide which user Apache work process running as. Recommend: nobody
</pre>

Group
<pre>
Decide which group Apache work process running as. Recommend: nobody
</pre>

Blacklist/whitelist IP/networks
<pre>
Order Deny,Allow
Deny from all
Allow from 176.16.0.0/16

Or by IP:

Order Deny,Allow
Deny from all
Allow from 127.0.0.1
</pre>

Blacklist/whitelist web contents
<pre>
It can prevent malicious attack via web content.

< Directory />
Order Deny,Allow # Default is Allow
Deny from all  # Deny all contents
< /Directory>
</pre>

Options FollowSymLinks
<pre>
Do not list any other files if the visited file don't exit.
</pre>

Hide info
<pre>
ServerSignature:
* Off, do not provide any information
* On, provide Apache infomation

ServerTokens:
* Full, exposure all information
* Prod, only provide server name
</pre>

Limit risky HTTP methods
<pre>
< Directory />
< LimitExcept GET POST>
Deny from all
< /LimitExcept>
< /Directory>

PUT/DELETE/etc methods won't be available.
</pre>

MinSpareServers
<pre>
Minimal spare processes. Default is 5. Recommend: 32.
</pre>

MaxSpareServers
<pre>
Maximum spare processes. Default is 20. Recommend: 64.
</pre>

ServerLimit
<pre>
The number that MaxClients can't not exceed.
</pre>

MaxClients
<pre>
The maximum number of working processes. Default is 256. Recommend:
8192.

MaxClients = (RAM available to Apache) / (RAM per Apache process)

RAM per Apache process:
#ps -ylC httpd --sort:rss

Example:
64GB physical memory * 0.8 = RAM available to Apache
HTML <= 5MB per process, PHP <= 15MB per processe
php: MaxClients = 65536MB / 15MB =4369
</pre>

MaxRequestsPerChild
<pre>
Limit on the number of requests that an individual child server will
handle during its life. Default is 4000. Recommend: 1500.
</pre>

----[ 5.2 WAF

ModSecurity is an open source, cross-platform web application firewall
(WAF) module. Known as the "Swiss Army Knife" of WAFs, it enables web
application defenders to gain visibility into HTTP(S) traffic and
provides a power rules language and API to implement advanced
protections. The web malicious signatures( including OWASP Top 10) are
maintained by ModSecurity community. You can [deploy it on Debian](https://www.digitalocean.com/community/tutorials/how-to-set-up-mod_security-with-apache-on-debian-ubuntu).

Anti-DoS: [mod_evasive](https://www.linode.com/docs/websites/apache-tips-and-tricks/modevasive-on-apache)

About anti DoS solution, I personally don't get used to
mod_evasive. Iptables would be much easier to maintain, eg:
<pre>
iptables -I INPUT -p tcp -m multiport --dports 80,443 -i eth0 -m state --state NEW -m recent --set
iptables -I INPUT -p tcp -m multiport --dports 80,443 -i eth0 -m state --state NEW -m recent --update --seconds 30 --hitcount 5 -j DROP
</pre>

--[ 6. Security standard

Well, there are a bunch of crazy security standards in the
planet. Some are compliance in some contries. FIPS-140-2/3, CC( EAL 7?
damn, it'd be an incarnation of the organge book;-)), PCI-DSS are very
popular terms you might hear from your security consultant. But... due
to lack of engineering implementation, these crazy( & creepy?)
security standards are not our concerns here.


----[ 6.1 STIGs for Debian

Once there's history, there's story about offense & defense. Once
we've heard fascinating stories from Mr.Sn0wden about how NSA fuck the
world, there should be some open information about how BIG-BROTHER do
the defense. [STIGs](http://iase.disa.mil/stigs/Pages/index.aspx) is one of them.

I think I'm not the right person to write this section.....


--[ 7. Reference

[1] Back To The Future: Unix Wildcards Gone Wild
    http://www.defensecode.com/public/DefenseCode_Unix_WildCards_Gone_Wild.txt

[2] SYNPROXY
    http://lwn.net/Articles/563151/

[3] DDoS protection Using Netfilter/iptables
    http://people.netfilter.org/hawk/presentations/devconf2014/iptables-ddos-mitigation_JesperBrouer.pdf

[4] INTERNET PROTOCOL
    http://tools.ietf.org/html/rfc791

[5] A simple TCP spoofing attack
    http://www.citi.umich.edu/u/provos/papers/secnet-spoof.txt

[6] ICMP Attacks Illustrated
    http://www.sans.org/reading-room/whitepapers/threats/icmp-attacks-illustrated-477

[7] SUSE Linux Enterprise Server 11 SP3 - Security and Hardening
    https://www.suse.com/documentation/sles11/singlehtml/book_hardening/book_hardening.html

[8] Securing Debian Manual 
    https://www.debian.org/doc/manuals/securing-debian-howto/

[9] A Brief Introduction to auditd
    http://security.blogoverflow.com/2013/01/a-brief-introduction-to-auditd/

[10] Apparmor RBAC
     http://wiki.apparmor.net/index.php/Pam_apparmor_example

[11] Hardening PHP from php.ini
     http://www.madirish.net/199

[12] CVE-2014-0196 exploit
http://bugfuzz.com/stuff/cve-2014-0196-md.c

[13] Secure Secure Shell
https://stribika.github.io/2015/01/04/secure-secure-shell.html

[14] STIGs
     http://iase.disa.mil/stigs/Pages/index.aspx
     http://iase.disa.mil/stigs/os/unix-linux/Pages/index.aspx
