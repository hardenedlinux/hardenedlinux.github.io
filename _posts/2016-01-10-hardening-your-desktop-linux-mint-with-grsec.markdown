---
layout: post
title:  "Hardening your desktop: Linux Mint with PaX/Grsecurity "
date:   2016-01-10 03:54:45
summary: We just celebrated another new year a couple of days ago, which means it's 2016 already. A another new year usually just brings us to another fight. FOSS is still our fortress, as always.
categories: system-security
---

--[ CONTENTS

0. About this doc

1. Build and install customized kernel with PaX/Grsecurity patch

2. PaX flags: paxctl-ng & pax-bites

3. Kernel tuning

4. Networking

5. Sandboxing: seccomp


##--[ 0. About this documentation
We just celebrated another new year a couple of days ago, which means it's 2016 already. A another new year usually just brings us to another fight. FOSS is still our fortress, as always.

* People may quite excited about we are getting one step closer to the Singularity. 
* People may praise about how great of technological evolution will be.
* People may just stuck in some shitty places and get drunk and going out of nowhere.
* People may..........well, it's none of my fuc\*ing business.

I just reviewed the current status of GNU/Linux distros. Sadly to say, only a few distros( Aphine, Gentoo, etc) shipped PaX/Grsecurity by default. Even worse if you take a glance at the GNU/Linux desktop field. The GUI is looking better than ever before. But the sense of security is almost like a decade ago. This is what make me ticks, to write this doc. I'm trying to show you there's a pretty easy way to do the whole process of hardening your desktop/laptop. I choose Linux Mint 17 as exmaple for two reasons.

1, I'm a Linux Mint user since Linut Mint 15. I personally like MATE and I hate GNOME3;-)
2, Linux Mint is the highest rank GNU/Linux distro in past 5 years( at least) according to (DistroWatch)[http://distrowatch.com/].


##--[ 1. Build and install customized kernel with PaX/Grsecurity patch

I assumed you have Linux Mint 17 installed on your laptop or PC already. The 1st thing to do is install some packages will be used in the building of PaX/Grsecurity kernel:
<pre>
sudo apt-get update && sudo apt-get upgrade
sudo apt-get install build-essential libncurses5-dev gcc-4.8-plugin-dev libssl-dev
</pre>

Download [linux kernel 4.3.3](https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.3.3.tar.xz) and PaX/Grsecurity [test patch for 4.3.3](https://github.com/hardenedlinux/hardenedlinux_profiles/raw/master/linux-mint_desktop/grsecurity-3.1-4.3.3-201601051958.patch)( note: for any commercial purpose use, I'm highly recommend you to use [PaX/Grsecurity stable patch](https://grsecurity.net/announce.php)). Then, you could use [my config](https://github.com/hardenedlinux/hardenedlinux_profiles/raw/master/linux-mint_desktop/grsec-4.3.3-for-linux-mint-17.config) to build kernel:
<pre>
cd kernel-src
patch -p1 < ../grsecurity-patch
cp grsec-4.3.3-for-linux-mint-17.config .
make -j4 deb-pkg
</pre>

Then you should see some .deb files and you can use dpkg to install your new kernel:
<pre>
dpkg -i \*.deb
</pre>

Now you should have multiple kernel options. Comment two lines in /etc/default/grub if you want to choose which kernel to bootup:
<pre>
#GRUB_HIDDEN_TIMEOUT=0
#GRUB_HIDDEN_TIMEOUT_QUIET=true
<pre>

And run:
<pre>
update-grub2
</pre>

Before reboot, plz make sure you install some popular applications:
<pre>
sudo apt-get install chromium-browser vlc firefox amarok
</pre>


##--[ 2. PaX flags: paxctl-ng & pax-bites

[PaX flags](https://en.wikibooks.org/wiki/Grsecurity/Appendix/PaX_Flags) is provided by PaX/Grsecurity to tell the kernel which mitigations should be used in the specific binary. More falgs are active, more security you gains. But some binaries are not working with some PaX flags, e.g: JIT can't work with MPROTECT. For the sake of making PaX/Grsecurity work with Linux Mint, we'll have to disable some mitigations in some binaries. I wrote a tool is called [pax-bites](https://github.com/hardenedlinux/pax-bites), which utilize paxctl-ng to add/delete PaX flags. The config file of pax-bites is very easy to write. The format is like this:
<pre>
file\_path;flags
</pre>

For example, I'm using (this config)[https://github.com/hardenedlinux/hardenedlinux_profiles/raw/master/linux-mint_desktop/pax_flags_mint17.config):
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

 I( and other members of Hardenedlinux community) have tested some popular applications. If you find some applications won't work correctly caused by PaX/Grsecurity, you can write your own config( plz let us know) or just file a bug by our github.


##--[ 3. Kernel tuning

The current [sysctl.conf](https://github.com/hardenedlinux/hardenedlinux_profiles/raw/master/linux-mint_desktop/sysctl.conf). We will do more work on this. Any detail about security-related kernel tuning, you might want to check [Debian security checklist](http://hardenedlinux.org/system-security/2015/06/09/debian-security-chklist.html).


##--[ 4. Networking

##--[ 5. Sandbox: seccomp
