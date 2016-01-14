---
layout: post
title:  "Hardening your desktop: Linux Mint with PaX/Grsecurity "
date:   2016-01-10 03:54:45
summary: We just celebrated another new year a couple of days ago, which means it's 2016 already. A another new year usually just brings us to another fight. FOSS is still our fortress, as always.
categories: system-security
---

By citypw

--[ CONTENTS

0. About this doc

1. Build and install customized kernel with PaX/Grsecurity patch

2. PaX flags: paxctl-ng & pax-bites

3. Kernel tuning

4. Networking

5. Sandboxing: seccomp

6. Crypto

   6.1 Entropy

   6.2 Daily bread

##--[ 0. About this documentation
We just celebrated another new year a couple of days ago, which means it's 2016 already. A another new year usually just brings us to another fight. FOSS is still our fortress, as always.

* People may quite excited about we are getting one step closer to the Singularity. 
* People may praise about how great of technological evolution will be.
* People may just stuck in some shitty places and get drunk and going out of nowhere.
* People may..........well, it's none of my fuc\*ing business.

I just reviewed the current status of GNU/Linux distros. Sadly to say, only a few distros( Aphine, Gentoo, etc) shipped PaX/Grsecurity by default. Even worse if you take a glance at the GNU/Linux desktop field. The GUI is looking better than ever before. But the sense of security is almost like a decade ago. This is what make me ticks, to write this doc. I'm trying to show you there's a pretty easy way to do the whole process of hardening your desktop/laptop. I choose Linux Mint 17 as exmaple for two reasons.

1, I'm a Linux Mint user since Linut Mint 15. I personally like MATE and I hate GNOME3;-)

2, Linux Mint is the highest rank GNU/Linux distro in past 5 years( at least) according to [DistroWatch](http://distrowatch.com/).


##--[ 1. Build and install customized kernel with PaX/Grsecurity patch

I assumed you have Linux Mint 17 installed on your laptop or PC already. The 1st thing to do is install some packages will be used in the building of PaX/Grsecurity kernel:
<pre>
sudo apt-get update && sudo apt-get upgrade

sudo apt-get install build-essential libncurses5-dev gcc-4.8-plugin-dev libssl-dev
</pre>

Then, download [linux kernel 4.3.3](https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.3.3.tar.xz) and PaX/Grsecurity [test patch for 4.3.3](https://github.com/hardenedlinux/hardenedlinux_profiles/raw/master/linux-mint_desktop/grsecurity-3.1-4.3.3-201601051958.patch)( note: for any commercial purpose use, I'm highly recommend you to use [PaX/Grsecurity stable patch](https://grsecurity.net/announce.php)). Then, you could use [my config](https://github.com/hardenedlinux/hardenedlinux_profiles/raw/master/linux-mint_desktop/grsec-4.3.3-for-linux-mint-17.config) to build kernel:
<pre>
cd kernel-src
patch -p1 < ../grsecurity-patch
cp grsec-4.3.3-for-linux-mint-17.config .config
make -j4 deb-pkg
</pre>

Then you should see some .deb files and you can use dpkg to install your new kernel:
<pre>
dpkg -i *.deb
</pre>

Now you should have multiple kernel options. Comment two lines in /etc/default/grub if you want to choose which kernel to bootup:
<pre>
#GRUB_HIDDEN_TIMEOUT=0
#GRUB_HIDDEN_TIMEOUT_QUIET=true
</pre>

And run:
<pre>
update-grub2
</pre>

Before reboot, plz make sure you install some popular applications:
<pre>
sudo apt-get install chromium-browser vlc firefox amarok
</pre>


##--[ 2. PaX flags: paxctl-ng & pax-bites

[PaX flags](https://en.wikibooks.org/wiki/Grsecurity/Appendix/PaX_Flags) is provided by PaX/Grsecurity to tell the kernel which mitigations should be used in the specific binary. More flags are enabled, more security you gains. But some binaries doesn't work with some PaX flags, e.g: JIT can't work with MPROTECT. For the sake of making PaX/Grsecurity work with Linux Mint, we'll have to disable some mitigations in some binaries. I wrote a tool is called [pax-bites](https://github.com/hardenedlinux/pax-bites), which utilize paxctl-ng to add/delete PaX flags. The config file of pax-bites is very easy to write. The format is like this:
<pre>
file_path;flags
</pre>

For example, I'm using [this config](https://github.com/hardenedlinux/hardenedlinux_profiles/raw/master/linux-mint_desktop/pax_flags_mint17.config):
<pre>
cat pax_flags_mint17.config 
/usr/lib/jvm/java-7-openjdk-amd64/jre/bin/java;m
/usr/bin/python2.7;m
/usr/bin/python3.4;m
/usr/lib/mdm/mdmwebkit;m
/usr/lib/firefox/firefox;m
/usr/lib/firefox/plugin-container;m
/usr/lib/thunderbird/thunderbird;m
/usr/lib/chromium-browser/chromium-browser;m
/usr/bin/amarok;m
/usr/bin/pulseaudio;m
</pre>

The very limited number of applications on Linut Mint GNU/Linux have been tested by us( I and other members of Hardenedlinux community). If you find some applications won't work correctly caused by PaX flags, you can write your own config( plz let us know) or just file a bug by our [github repo](https://github.com/hardenedlinux/hardenedlinux_profiles).


##--[ 3. Kernel tuning

The current [sysctl.conf](https://github.com/hardenedlinux/hardenedlinux_profiles/raw/master/linux-mint_desktop/sysctl.conf). We will do more work on this. Any detail about security-related kernel tuning, you might want to check [Debian security checklist](http://hardenedlinux.org/system-security/2015/06/09/debian-security-chklist.html).


##--[ 4. Networking

Iptables is a eay-to-use stateful firewall. It's very important to let your desktop/laptop have it. [Some policies](https://github.com/hardenedlinux/hardenedlinux_profiles/blob/master/linux-mint_desktop/iptables_mint17.sh) has been tested. You can use it directly. Any feedback or improvement are always welcome.

You might be interested in [how iptables/netfilter works](https://raw.githubusercontent.com/citypw/DNFWAH/master/2/d2_0x06_Hacking_the_wholism_of_linux_net.txt).

##--[ 5. Sandbox: Seccomp-bpf based implementation

The sandboxing technolog has a very long history of evolution journey in past decades. Here we're just going through [Seccomp based implementation](http://ekoparty.org/archive/2013/charlas/Sandboxing%20Linux%20code%20to%20mitigate%20exploitation%20%28Or-%20How%20to%20ship%20a%20secure%20operating%20system%20that%20includes%20third-party%20code%29.pdf). [Seccomp](https://wiki.mozilla.org/Security/Sandbox/Seccomp) is being merged into vanilla kernel since 2.6.12. Seccomp can not support complicated task because it whitelisted a few system calls for computing-only task. Anyway, it's not what we talk about here. We only care about Seccomp's extension: [Seccomp-bpf](https://www.kernel.org/doc/Documentation/prctl/seccomp_filter.txt).

There are two approaches to sandboxing the application you choose:

* You can add Seccomp-bpf polices into the application by tweaking the polices via prctl(). IMOHO, it's quite like pledge() system call in latest OpenBSD. Here's [some examples](https://outflux.net/teach-seccomp/) from Kees Cook( Yeah, that Kees who helped out to brings the real sense of security into kernel upstream;-))

* Use a external wrapper program to exec the application with Seccomp-bpf filtering polices.

The 1st approach is developer-only. Because you'll have to modify the source code add those polices into it. Otherwise, I don't think this gonna happen in GNU/Linux community. OpenBSD community are trying add [pledge()](http://www.openbsd.org/cgi-bin/man.cgi/OpenBSD-current/man2/pledge.2) into diverse applications but...the people who cares about security from OpenBSD community were called "[bunch of masturbating monkeys](http://article.gmane.org/gmane.linux.kernel/706950)" by Linus Torvalds....

The 2nd approach seems quite flexible. Here are two options:

* [Playpen](https://github.com/thestinger/playpen), developed by Daniel Micay.

* [Firejail](https://firejail.wordpress.com/), developed/maintained by a group of people.

I choose firejail, because the community provides a bunch of sandboxing polices for [some popular applications](http://forums.linuxmint.com/viewtopic.php?f=42&t=202735)( firefox, vlc, chromium, etc). It's almost like off-the-shell stuff. You can download the latest version of [firejail](http://sourceforge.net/projects/firejail/files/firejail/) and install it by package or compile it from source code. Note that the default Seccomp-bpf policies doesn't have any system call whitelists, which mean it is a sandbox without filtering any syscalls basically. So [write your own one](https://l3net.wordpress.com/2015/04/13/firejail-seccomp-guide/) if you want.


##--[ 6. Crypto

Crypto engineering has been playing a very important role in past decade. System security & crypto engineering are like twins. One can't live without another. Use crypto without system hardening is like building your infrastructre on sand. Only hardening the system without using crypto is like archer's holding a fancy Elf silver-bow but doesn't have any arrows at all. Don't forget the goal of infosec is to protect the fuc*ing information in 3 basic dimensions: C(onfidentiality), I(ntegrity) and A(vailability).

##----[ 6.1 Entropy

Entropy is matter to private/session keys. It'd be a big deal if you're using GNU/Linux as server. Desktop won't be have much trouble about it. PaX/Grsecurity increased entropy for the kernel somehow. You may still want to install a daemon to ensuring the system always has enough entrop. Let the cryptographers worries about blocking/non-blocking issues on [random/urandom](http://www.2uo.de/myths-about-urandom/). All you need to know is [Haveged](http://www.issihosts.com/haveged/history.html) will be very helpful to your server/desktop. You can [install it via](https://www.digitalocean.com/community/tutorials/how-to-setup-additional-entropy-for-cloud-servers-using-haveged):

<pre>
Installing haveged on Debian/Ubuntu

You can easily install haveged on Debian and Ubuntu by running the following command:

# apt-get install haveged

Should this package not be available in your default repositories, you will need to compile from source (see below)

Once you have the package installed, you can simply edit the configuration file located in /etc/default/haveged, ensuring the following options are set (usually already the default options):

DAEMON_ARGS="-w 1024"

Finally, just make sure it's configured to start on boot:

# update-rc.d haveged defaults
</pre>

##----[ 6.2 Daily bread

* [Pidgin](https://pidgin.im/) + OTR
<pre>
sudo apt-get install pidgin pidgin-otr
</pre>


* [Tor broswer](https://www.torproject.org/)

It can bring you anonymity. Download it [here](https://www.torproject.org/download/download-easy.html.en).

* [GnuPG](https://www.gnupg.org/)

Encryption/Decryption email with GPG is not a bad idea. You may also want to try [opmsg](https://github.com/stealth/opmsg).
